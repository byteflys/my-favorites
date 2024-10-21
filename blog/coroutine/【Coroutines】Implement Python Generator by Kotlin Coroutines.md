##### Expected

a generator object that can generate sequence objects

and only generate object when consumer object request for it

```kotlin
fun main() {
    val generator = generator {
        yield(100)
        yield(200)
        yield(300)
    }
    while (generator.hasNext()) {
        val value = generator.await()
        println(value)
    }
}
```

##### Implementation

```kotlin
package com.code.kotlin

import kotlin.coroutines.*

@RestrictsSuspension
interface GeneratorScope<T> {

    suspend fun yield(value: T)
}

interface GeneratorIterator<T> : Iterator<T> {

    fun await(): T
}

internal interface Generator<T> : GeneratorScope<T>, GeneratorIterator<T>, Continuation<Any>

internal sealed class GeneratorState<T> {
    class WAITING<T>(val continuation: Continuation<Unit>) : GeneratorState<T>()
    class READY<T>(val continuation: Continuation<Unit>, val value: T) : GeneratorState<T>()
    class COMPLETED<T> : GeneratorState<T>()
}

internal class GeneratorImpl<T> : Generator<T> {

    override val context: CoroutineContext = EmptyCoroutineContext

    internal lateinit var state: GeneratorState<T>

    override suspend fun yield(value: T) {
        suspendCoroutine { continuation ->
            when (val _state = state) {
                is GeneratorState.WAITING -> {
                    state = GeneratorState.READY(continuation, value)
                }
                is GeneratorState.READY,
                is GeneratorState.COMPLETED -> throw IllegalStateException()
            }
        }
    }

    override fun resumeWith(result: Result<Any>) {
        state = GeneratorState.COMPLETED()
    }

    private fun resume() {
        when (val _state = state) {
            is GeneratorState.WAITING -> {
                _state.continuation.resume(Unit)
            }
            else -> {}
        }
    }

    override fun hasNext(): Boolean {
        resume()
        return state !is GeneratorState.COMPLETED
    }

    override fun next(): T {
        when (val _state = state) {
            is GeneratorState.READY -> {
                state = GeneratorState.WAITING(_state.continuation)
                return _state.value
            }
            is GeneratorState.WAITING,
            is GeneratorState.COMPLETED -> throw IllegalStateException()
        }
    }

    override fun await() = next()
}

fun <T> generator(
    block: suspend GeneratorScope<T>.() -> Unit
): GeneratorIterator<T> {
    val generator = GeneratorImpl<T>()
    val continuation = block.createCoroutine(generator, generator)
    generator.state = GeneratorState.WAITING(continuation)
    return generator
}
```

