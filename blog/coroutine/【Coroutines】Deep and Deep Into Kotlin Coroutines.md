##### Structure of Coroutine Framwork

coroutine framwork consist of two parts

- Basic Coroutine Library, which is naturally integrated in kotlin language
- Coroutine Application Framwork, which is organized to simplify advanced usage of coroutines
- The first part is really simple, while the next part is larger, and offered by `kotlinx-coroutines-core` library

in this chapter, we will talk about the first part, that is, how coroutines really work at underlying level

##### Nature of Coroutine

coroutine is a code scope that has ability to suspend and resume execution

if a function has a `suspend` modifier, it is running in coroutine scope, can share the ability of coroutine

in fact, `suspend function` implictly invole an object with Continuation class, that is how it interact with coroutines

##### Coroutine States

- NotDecided, created by not suspended
- Suspended, suspend temporally
- Resumed, resume from suspended state, may be running or completed

##### Composition of Basic Coroutine Library

these classed are important for understanding what coroutines really do

- CoroutineSingletons : coroutine state enumerations
- Result : save suspend function result and error, also can used to save coroutine states
- Continuation : have many functional roles, depend on concrete subclass, start coroutine, coroutine body, coroutine callback

let us take a first look at basic coroutine code frist, then step deeply into Continuation subclassed

##### Basic Coroutines Usage

```kotlin
fun main() {
    val completeContinuation = object : Continuation<Int> {
        override val context = EmptyCoroutineContext
        override fun resumeWith(result: Result<Int>) {
            println(result.getOrThrow()) // ④
        }
    }
    val safeContinuation = suspend {
        100 // ③
    }.createCoroutine(completeContinuation) // ①
    safeContinuation.resume(Unit) // ②
}
```

```kotlin
suspend fun main() {
    val result = suspendCoroutine { continuation ->
        continuation.resume(100) // ②
    } // ①
    println(result) // ③
}
```

demo 1 show how to create and start a coroutine

demo 2 show how to suspend and resume current coroutine

##### Continuation Subclasses

- BaseContinuationImpl  : abstract class, implement ability of resume
- SuspendLambda : coroutine body, do actual work
- SafeContinuation : check coroutine state, if suspended resume it by delegate, else save result and return
- CompletedContinuation : handle coroutine result callback
- RunSuspend : start a coroutine using main function as coroutine body (suspend block)

`resumeWith(result)` has various effect in different implementations

- SafeContinuation : start coroutine
- SuspendLambda : execute coroutine body
- CompletedContinuation : handle coroutine callback

##### Analysis of Demo 1

```kotlin
fun main() {
    val completeContinuation = object : Continuation<Int> {
        override val context = EmptyCoroutineContext
        override fun resumeWith(result: Result<Int>) {
            println(result.getOrThrow()) // ④
        }
    }
    val safeContinuation = suspend {
        100 // ③
    }.createCoroutine(completeContinuation) // ①
    safeContinuation.resume(Unit) // ②
}
```

- ① create coroutine from suspend block, and enter suspended state, then return a SafeContinuation instance

- ② start coroutine, as SafeContinuation do not handle actual work, param is void
- suspend block will be compiled to a SuspendLambda instance
- SafeContinuation use SuspendLambda as a delegate object, deliver actual work to SuspendLambda
- ③ SuspendLambda execute block code by `invokeSuspend` , and get execution result
- ④ SuspendLambda deliver result to CompletedContinuation (completion)
- if completion is not CompletedContinuation, regard completion as a delegate continuation, and repeat wroks above
- then a new cycle of circulation begins, forms a hierachical coroutine system

##### How Does Continuation Resume from Suspended State 

let us see `BaseContinuationImpl#resumeWith`

this is the quintessence of how coroutine works, shuttling between suspended and resumed states

```kotlin
public final override fun resumeWith(result: Result<Any?>) {
    var current = this
    var param = result
    while (true) {
        with(current) {
            val outcome: Result<Any?> = try {
                val outcome = invokeSuspend(param)
                if (outcome === COROUTINE_SUSPENDED) return
                Result.success(outcome)
            } catch (exception: Throwable) {
                Result.failure(exception)
            }
            val completion = completion!!
            if (completion is BaseContinuationImpl) {
                current = completion
                param = outcome
            } else {
                completion.resumeWith(outcome)
                return
            }
        }
    }
}
```

we can get these conclusions from above

- child coroutine can resume parent coroutine, through looped assignment of `current`
- when completion is instance of `BaseContinuationImpl`, means it is not only a callback, but also the parent coroutine which launch it
- result of `invokeSuspend` can be used as input param for parent coroutine
- if current coroutine suspend again during `invokeSuspend` , function returned, wait next calling of `resumeWith`

##### What Does SafeContinuation Do

```kotlin
public actual override fun resumeWith(result: Result<T>) {
    while (true) {
        val current = this.result
        when {
            current === UNDECIDED -> 
                if (RESULT.compareAndSet(this, UNDECIDED, result.value)) return
            current === COROUTINE_SUSPENDED -> 
                if (RESULT.compareAndSet(this, COROUTINE_SUSPENDED, RESUMED)) {
                    delegate.resumeWith(result)
                    return
                }
            else ->
          			throw IllegalStateException("Already Resumed")
        }
    }
}
```

- if coroutine is not suspended, directly set result
- if coroutine is suspended, resume it through delegate
- if coroutine is resumed, throw error

##### What Does CompletedContinuation Do

```kotlin
override fun resumeWith(result: Result<Any?>) {
    error("This continuation is already complete")
}
```

do nothing and throw error by default

we must override it by ourself, it is just a callback, not responsible for resume actually

##### What Does InvokeSuspend Do

```kotlin
try {
    val outcome = invokeSuspend(param)
    if (outcome === COROUTINE_SUSPENDED) return
    Result.success(outcome)
} catch (exception: Throwable) {
    Result.failure(exception)
}
```

`invokeSuspend` execute code from suspend block

if the block directly run out all lines, return block result

if the block suspended on half way, return coroutine state of `COROUTINE_SUSPENDED`

##### What Does CreateCoroutine Do

```kotlin
public fun <T> (suspend () -> T).createCoroutine(
    completion: Continuation<T>
): Continuation<Unit> = SafeContinuation(createCoroutineUnintercepted(completion), COROUTINE_SUSPENDED)
```

create a coroutine, and set its initial state to supsended

two continuations are created here, `SuspendLambda` and `SafeContinuation`

their initial states are all `COROUTINE_SUSPENDED`

##### When to Suspend and Resume

there are two typical ways to suspend and resume a coroutine

- Thread Scheduling : 

  suspend current code, start a thread to handle async work, resume at end of thread

- Loop Queue : 

  suspend current code, submit a task to handle async work, resume at end of current task

suspend and resume may happen in same thread, if we use the Loop Queue way

##### Analysis of Demo 2

we are already clear that, how to start, how to resume, how to callback

but how to suspend during a current code chain, is still a mystery, let us continue

```kotlin
suspend fun main() {
    val result = suspendCoroutine { continuation ->
        continuation.resume(100) // ②
    } // ①
    println(result) // ③
}
```

`suspend main` can be regard as this

```kotlin
val mainBlock: suspend () -> Unit
val completion: Continuation
mainBlock.createCoroutine(completion).resume(Unit)
completion.await()
```

the compiler tricked us with a magic, `suspend main` is actually the `suspend block` that used to create coroutine

`suspend main` was running within a hidden method, which was the truly `main`

##### Nature of Suspend Main

implementation of `suspend main` can be found in `RunSuspend.kt`

```kotlin
internal fun runSuspend(block: suspend () -> Unit) {
    val run = RunSuspend()
    block.startCoroutine(run)
    run.await()
}
```

```kotlin
private class RunSuspend : Continuation<Unit> {

    var result: Result<Unit>? = null

    override fun resumeWith(result: Result<Unit>) = synchronized(this) {
        this.result = result
        notifyAll()
    }

    fun await() = synchronized(this) {
        while (true) {
            when (val result = this.result) {
                null -> wait()
                else -> {
                    result.getOrThrow()
                    return
                }
            }
        }
    }
}
```

when this code is compiled to jvm bytecode, its running process can be like this

- JVM launch main function of main Class
- main function create a RunSuspend object, as coroutine completion callback
- main function use code in `suspend main` as `suspend block` to create and start a coroutine
- main function call completion.await, as result is null, it will wait for lock
- coroutine resumed, completion.resumeWith is called
- completion save result and call notifyAll
- lock is woken up, completion.await returns, program ends

##### What Does SuspendCoroutine Do

```kotlin
suspend fun main() {
    val result = suspendCoroutine { continuation ->
        continuation.resume(100) // ②
    } // ①
    println(result) // ③
}
```

① `suspendCoroutine` suspend current coroutine, and start a async code block

② a `Continuation` object is given to resume coroutine, when async work is finished

as `suspendCoroutine` is designed to suspend coroutine, it must run in coroutine scope first

③ as current coroutine is suspended, step 3 will be executed after step 2

if code in step 2 is removed, step 3 will never reached, as coroutine is suspened forever

##### Conclusions

- there is no class called Coroutine
- coroutine present a continuous code block that can suspend and resume
- suspend and resume abilities are implemented through Continuation class
- suspend keyword is not naturally supported by jvm bytecode
- suspend and resume code are converted into jvm bytecode through kotlin compiler
- suspend and resume location can be inferred by saving and restoring coroutine calling stack frame

##### Trailer

the most roles in kotlin coroutines are :

`Continuation class` `suspend block` `createCoroutine` `suspendCoroutine` `resume function`

other concepts and apis are all extended from these ones, they are application framworks, not basic facilities

next, we will talk about how to implement `CoroutineStackFrame Suspend and Resume` in other topics
