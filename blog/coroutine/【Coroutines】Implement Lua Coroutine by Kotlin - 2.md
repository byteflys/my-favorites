##### Symmetric Coroutines

in last blog, we have talked about how to implement lua-style coroutine

there are two kinds of coroutines, symmetric and non-symmetric

- symmetric

  when coroutine suspend or complete, execution go back to the point resume it

  that means coroutines have a hierarchical relation of calling

- non-symmetric

  each coroutine is independent, coroutine can specify where to go when suspend

##### Non-Symmetric Coroutine Sample

like this, implemented in last blog

``````kotlin
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
``````

##### Symmetric Coroutine Sample

which we will talk about soon in this blog

``````kotlin
package x.coroutine

suspend fun main() {
    lateinit var coroutine1: SymmetricCoroutine<String>
    lateinit var coroutine2: SymmetricCoroutine<String>
    lateinit var coroutine3: SymmetricCoroutine<String>
    coroutine1 = createSymmetric {
        println("parameter ${getParameter()}")
        transfer(coroutine3, "d")
    }
    coroutine2 = createSymmetric {
        transfer(coroutine1, "c")
    }
    coroutine3 = createSymmetric {
        println("symmetric start")
        transfer(coroutine2, "b")
        transfer(coroutine1, "e")
    }
    val main = launchSymmetric(coroutine3, "a")
    coroutine1.clean()
    coroutine2.clean()
    coroutine3.clean()
    println("symmetric end")
}
``````

each coroutine can randomly goto another coroutine, with a input param carried

##### How to Implement Symmetric Coroutines

kotlin built-in coroutine is the non-symmetric one

but we can implement symmetric coroutines through non-symmetric ones

obviously `transfer` is the core api that we need to implement

 `transfer` suspend current coroutine, and resume another coroutine, with a yielded param carried

this point is same to `non-symmetric coroutines` 

the difference is,  `symmetric coroutine` will never go back to previous coroutine

##### Wonderful Tricks

if we create a implicit `main coroutine`

when `coroutine a` want to transfer to `coroutine b`

it can deliver `coroutine b` and `resume parameter` to `main coroutine` 

then let the `main coroutine` resume `coroutine b`

that is, a suspend, return back to main, then main resume b

now, it is totally same with the `non-symmetric coroutines` 

the yield result is `target coroutine + resume param`

##### Code Design

`SymmetricCoroutine` hold a `Coroutine` object, that responsible for execution schedule

when `main coroutine` calls `transfer` , it will resume `work coroutine` and wait for its result

when `work coroutine` calls `transfer` , it will yield a `TransferContext` object as result, then resume  `main coroutine`

`TransferContext` is composed of `next coroutine` object and a `coroutine resume parameter`

when  `main coroutine` received the `TransferContext` as a result, it will transfer the  `next coroutine` again

```kotlin
internal val coroutine: CoroutineImpl<T, TransferContext<*>?> = CoroutineImpl(context) {
    block()
    return@CoroutineImpl null
}
```

```kotlin
data class TransferContext<T>(
    val coroutine: SymmetricCoroutine<T>,
    val parameter: T?
)
```

```kotlin
private tailrec suspend fun <R> transferInner(other: SymmetricCoroutine<R>, param: Any?) {
    if (!isMain) {
        val transferContext = TransferContext(other, param as R)
        coroutine.yield(transferContext)
        return
    }
    if (!other.isMain()) {
        val impl = other as SymmetricCoroutineImpl<R>
        val transferContext = impl.coroutine.resume(param as R)
        transferContext?.let {
            transferInner(it.coroutine, it.parameter)
        }
    }
}
```

these are core codes, while the remains are auxiliary, just to fulfill details and offer easy-to-use apis

##### Tail Recursion Optimization

we notice that, all transfer work in work coroutines

are actually implemented by recursive execution of `MainCoroutine.transfer` , until all work coroutines finished

when work coroutines works for a long time, calling stack of main coroutine will become bigger and bigger

eventually caused `StackOverflowError` error

kotlin offers a `tailrec` keyword to optimize recursive execution

the theroy of  `tailrec` is, use while instead of recursion, to avoid stack size increase

##### Full Sources

```kotlin
package x.coroutine

suspend fun main() {
    lateinit var coroutine1: SymmetricCoroutine<String>
    lateinit var coroutine2: SymmetricCoroutine<String>
    lateinit var coroutine3: SymmetricCoroutine<String>
    coroutine1 = createSymmetric {
        println("parameter ${getParameter()}")
        transfer(coroutine3, "d")
    }
    coroutine2 = createSymmetric {
        transfer(coroutine1, "c")
    }
    coroutine3 = createSymmetric {
        println("symmetric start")
        transfer(coroutine2, "b")
        transfer(coroutine1, "e")
    }
    val main = launchSymmetric(coroutine3, "a")
    coroutine1.clean()
    coroutine2.clean()
    coroutine3.clean()
    println("symmetric end")
}
```

```kotlin
package x.coroutine

import kotlin.coroutines.EmptyCoroutineContext

interface SymmetricCoroutine<T> {

    fun isMain(): Boolean

    suspend fun clean()
}

interface SymmetricCoroutineScope<T> {

    fun getParameter(): T

    suspend fun <R> transfer(other: SymmetricCoroutine<R>, param: R)
}

data class TransferContext<T>(
    val coroutine: SymmetricCoroutine<T>,
    val parameter: T?
)

fun <T> createSymmetric(
    block: suspend SymmetricCoroutineScope<T>.() -> Unit
): SymmetricCoroutine<T> {
    return SymmetricCoroutineImpl(EmptyCoroutineContext, block)
}

suspend fun <T> launchSymmetric(
    symmetric: SymmetricCoroutine<T>, param: T
): SymmetricCoroutine<Unit> {
    val main = SymmetricCoroutineImpl<Unit>(EmptyCoroutineContext) {
        transfer(symmetric, param)
    }
    main.isMain = true
    main.coroutine.resume(Unit)
    return main
}
```

```kotlin
package x.coroutine

import kotlin.coroutines.CoroutineContext

internal class SymmetricCoroutineImpl<T>(
    context: CoroutineContext,
    block: suspend SymmetricCoroutineScope<T>.() -> Unit
) : SymmetricCoroutine<T>, SymmetricCoroutineScope<T> {

    internal var isMain = false

    internal val coroutine: CoroutineImpl<T, TransferContext<*>?> = CoroutineImpl(context) {
        block()
        return@CoroutineImpl null
    }

    override fun isMain() = isMain

    override fun getParameter(): T {
        return coroutine.parameter!!
    }

    override suspend fun <R> transfer(other: SymmetricCoroutine<R>, param: R) = transferInner(other, param)

    private tailrec suspend fun <R> transferInner(other: SymmetricCoroutine<R>, param: Any?) {
        if (!isMain) {
            val transferContext = TransferContext(other, param as R)
            coroutine.yield(transferContext)
            return
        }
        if (!other.isMain()) {
            val impl = other as SymmetricCoroutineImpl<R>
            val transferContext = impl.coroutine.resume(param as R)
            transferContext?.let {
                transferInner(it.coroutine, it.parameter)
            }
        }
    }

    override suspend fun clean() {
        while (!coroutine.completed()) {
            coroutine.resume(getParameter())
        }
    }
}
```

