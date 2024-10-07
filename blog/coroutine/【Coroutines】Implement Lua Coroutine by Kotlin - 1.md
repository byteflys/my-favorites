##### What We Will Do in This Chapter

we will achieve a lua-like coroutine framework

further more, we will learn couroutine design theroy on these sides

- CoroutineScope
- ContinuationInterceptor
- CoroutineDispatcher
- ThreadSafety

##### Expected

we want a coroutine framework like this style

```kotlin
package x.coroutine

suspend fun main() {

    val producer = GlobalScope.launch<Unit, Int>(Dispatchers.new()) {
        for (i in 1..3)
          yield(i)
        return@launch 0
    }

    val consumer = GlobalScope.launch<Int, Unit>(Dispatchers.new()) {
        for (i in 1..3)
          yield(Unit)
        return@launch Unit
    }

    while (!producer.completed() && !consumer.completed()) {
        val param1 = producer.resume(Unit)
        val param2 = consumer.resume(param1)
    }
}
```

it looks like simple, but not easy to implement, when suspend feature is required

let us clarify the workflow first

- create a producer coroutine, used to generate data
- create a comsumer coroutine, used to consume data
- both producer and consumer will not work immediately, until coroutine is resumed
- while-loop block resumes producer, then use the result of producer as param, to resume consumer
- main function will suspend, after resume producer/consumer, until result is yielded
- main function works in a implicit coroutine scope, that is why the while-loop block can suspend
- when result is yielded, producer/consumer will suspend itself, and resume main function

##### Code Design

- there are three coroutines totaly, main coroutine is implicitly created by `suspend main`
- producer/consumer coroutine can be created by `createCoroutine` api
- main coroutine will suspend when call resume, this feature can be implemented by `suspendCoroutine` api
- each coroutine can receive a param value on resume, and return a result value by yield

- result yielded by producer, is the resume param of consumer

- specially, when producer/consumer reach block end, it will call `CompletionContinuation.resumeWith` instead of `Coroutine.resume` , to resume main coroutine
- one coroutine must holder another coroutine which resume it, in order to resume back on suspend

next, let us start implementing lua-style coroutine

##### WriteableCoroutine

receive param and yield result

``````kotlin
interface WriteableCoroutine<P, R> {

    var parameter: P?

    suspend fun yield(result: R): P
}
``````

##### ReadableCoroutine

offer param and read result

``````kotlin
interface ReadableCoroutine<P, R> {

    suspend fun resume(parameter: P): R

    fun completed(): Boolean
}
``````

##### Status

hold coroutine status and continuation remaining to resume coroutine

``````kotlin
sealed class Status {
    internal class Created(val continuation: Continuation<Unit>) : Status()
    internal class Suspended<P>(val continuation: Continuation<P>) : Status()
    internal class Resumed<R>(val continuation: Continuation<R>) : Status()
    internal data object Completed : Status()
}
``````

##### CoroutineScope

contains a set of coroutine contexts, which can control coroutine's workflow

`GlobalScope` is the default `CoroutineScope` , do nothing to coroutines

``````kotlin
interface CoroutineScope {
    var coroutineContext: CoroutineContext
}

object GlobalScope : CoroutineScope {
    override var coroutineContext: CoroutineContext = EmptyCoroutineContext
}
``````

##### Dispatchers

use `Executor` and `Interceptor` to specify which thread coroutines work on

`Dispatcher` is a subclass of `ContinuationInterceptor` , also a subclass of `CoroutineContext`

```kotlin
package x.coroutine

import java.util.concurrent.Executor
import java.util.concurrent.Executors
import kotlin.coroutines.Continuation
import kotlin.coroutines.ContinuationInterceptor

// theory of Dispatcher & Interceptor
object Dispatchers {

    private val currentExecutor = Executor { runnable -> runnable.run() }

    private val newExecutor = Executors.newCachedThreadPool()

    private val dataExecutor = Executors.newSingleThreadExecutor()

    private val networkExecutor = Executors.newScheduledThreadPool(10)

    fun current() = Dispatcher(currentExecutor)

    fun new() = Dispatcher(newExecutor)

    fun data() = Dispatcher(dataExecutor)

    fun network() = Dispatcher(networkExecutor)

    fun sequence() = Dispatcher(Executors.newSingleThreadExecutor())
}

// delegate continuation work to another continuation
// which supports thread schedule
class Dispatcher(
    val executor: Executor? = null
) : ContinuationInterceptor {

    override val key = ContinuationInterceptor

    override fun <T> interceptContinuation(continuation: Continuation<T>): Continuation<T> {
        return DispatcherContinuation(continuation, executor)
    }
}

// continuation that supports thread schedule
class DispatcherContinuation<T>(
    val continuation: Continuation<T>,
    val executor: Executor? = null
) : Continuation<T> by continuation {

    override fun resumeWith(result: Result<T>) {
        if (executor == null) {
            return continuation.resumeWith(result)
        }
        executor.execute {
            continuation.resumeWith(result)
        }
    }
}
```

##### Create Lua-Style Coroutine

```GlobalScope``` offer the `scope-related contexts`

while `LaunchContext` offer the `user-defined contexts`

add them up to get a `CombinedContext` as the final `CoroutineContext`

in this program, ```GlobalScope``` do nothing, just display role `CoroutineScope` acts

``````kotlin
fun <P, R> GlobalScope.launch(
    context: CoroutineContext,
    block: suspend WriteableCoroutine<P, R>.() -> R
): ReadableCoroutine<P, R> {
    val context = coroutineContext + context
    return CoroutineImpl(context, block)
}
``````

##### CoroutineImpl

this is the final implementation of all interfaces above

``````kotlin
package x.coroutine

import java.util.concurrent.atomic.AtomicReference
import kotlin.coroutines.*

internal class CoroutineImpl<P, R>(
    override var context: CoroutineContext,
    block: suspend WriteableCoroutine<P, R>.() -> R
) : Continuation<R>, WriteableCoroutine<P, R>, ReadableCoroutine<P, R> {

    override var parameter: P? = null

    private val status: AtomicReference<Status>

    init {
        val continuation = block.createCoroutine(this, this)
        status = AtomicReference(Status.Created(continuation))
    }

    override fun completed() = status.get() is Status.Completed

    override suspend fun yield(result: R): P = suspendCoroutine { continuation ->
        val previous = status.getAndUpdate {
            when (it) {
                is Status.Resumed<*> -> Status.Suspended(continuation)
                else -> throw IllegalStateException()
            }
        }
        (previous as Status.Resumed<R>).continuation.resume(result)
    }

    override suspend fun resume(param: P): R = suspendCoroutine { continuation ->
        val previous = status.getAndUpdate {
            when (it) {
                is Status.Created,
                is Status.Suspended<*> -> {
                    parameter = param
                    Status.Resumed(continuation)
                }
                else -> throw IllegalStateException()
            }
        }
        when (previous) {
            is Status.Created -> previous.continuation.resume(Unit)
            is Status.Suspended<*> -> (previous as Status.Suspended<P>).continuation.resume(param)
            else -> {}
        }
    }

    override fun resumeWith(result: Result<R>) {
        val previous = status.getAndUpdate {
            when (it) {
                is Status.Resumed<*> -> Status.Completed
                else -> throw IllegalStateException()
            }
        }
        (previous as Status.Resumed<R>).continuation.resumeWith(result)
    }
}
``````

##### Running Test

let's add some annotations, to see wheter the workflow is right

``````kotlin
package x.coroutine

suspend fun main() {

    printThreadId("main.start")

    val producer = GlobalScope.launch<Unit, Int>(Dispatchers.new()) {
        for (i in 1..3) {
            printThreadId("producer.yieldWith $i")
            yield(i)
        }
        return@launch 0
    }

    val consumer = GlobalScope.launch<Int, Unit>(Dispatchers.new()) {
        for (i in 1..3) {
            println("$parameter consumed")
            printThreadId("consumer.yieldWith Unit")
            yield(Unit)
        }
        return@launch Unit
    }

    while (!producer.completed() && !consumer.completed()) {
        printThreadId("producer.resumeWith Unit")
        val param1 = producer.resume(Unit)
        printThreadId("consumer.resumeWith $param1")
        val param2 = consumer.resume(param1)
    }

    printThreadId("main.end")
}

fun printThreadId(tag: String) {
    val tid = Thread.currentThread().id
    println("$tag <tid=$tid>")
}
``````

``````xaml
main.start <tid=1>
producer.resumeWith Unit <tid=1>
producer.yieldWith 1 <tid=23>
consumer.resumeWith 1 <tid=23>
1 consumed
consumer.yieldWith Unit <tid=24>
producer.resumeWith Unit <tid=24>
producer.yieldWith 2 <tid=23>
consumer.resumeWith 2 <tid=23>
2 consumed
consumer.yieldWith Unit <tid=24>
producer.resumeWith Unit <tid=24>
producer.yieldWith 3 <tid=23>
consumer.resumeWith 3 <tid=23>
3 consumed
consumer.yieldWith Unit <tid=24>
producer.resumeWith Unit <tid=24>
consumer.resumeWith 0 <tid=23>
main.end <tid=24>
``````

perfect, all as expected !
