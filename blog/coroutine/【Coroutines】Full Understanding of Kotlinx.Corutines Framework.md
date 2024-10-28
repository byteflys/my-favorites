##### What is Corutines

corutines is a modern way of  async programming

a corutine is a async code block  that supports suspend and resume

typically the code block looks like a synchronous block, but actually executed in async way

##### Difference between Corutine and Thread

both corutine and thread can achieve async works

for this point, they are the same, corutine can be regard as light-weight thread

but in fact, corutine is more a programming pattern, it cares how to write code easily

while thread is a hardware concept, it cares how cpu really works in concurrent situation

multiple corutines can run on same thread, also can run on different threads

corutines emphase how to write async code, while threads emphase the actual performance

corutines depend on threads, but it simplify traditional async programming with thread apis

##### Fast Usage

`launch` start a corutine to handle async work

`step 2` will output before `step 1` , because coroutine will not block code in host

```kotlin
import kotlinx.coroutines.*

suspend fun main() {
    GlobalScope.launch {
        delay(1000L)
        println("step 1")
    }
    println("step 2")
  delay(999*1000L)
}
```

`async` start a corutine that have a return value

the return value which called `deferred` , can be used in other corutine

`step 1` will output before `step 2` this time, as corutine 2 will wait result of corutine 1

`step 3` will still output before `step 1` 

```kotlin
import kotlinx.coroutines.*

suspend fun main() {
    val deferred = GlobalScope.async {
        delay(5000L)
        println("step 1")
        return@async 100
    }
    GlobalScope.launch {
        println("step 2 ${deferred.await()}")
    }
    println("step 3")
    delay(99000L)
}
```

##### Suspend Function

from demos above, we can see a keyword called `suspend`

`suspend` means that this function can hang up, and continue handling work in another place

`suspend` function can only be called from coroutine block, or other `suspend` functions

because only coroutine has the ability to suspend and resume, normal functions was not able to do this

behavior contrary to `suspend` is `resume` , which is often hidden under the mask of coroutines framework

##### Advanced Usage of Coroutine

there are many variant forms to use coroutines

you can launch a coroutine using `kotlinx.coroutines` library like this

```kotlin
import kotlinx.coroutines.*

suspend fun main() {
    val parentJob = Job()
    val scope = CoroutineScope(parentJob)
    val dispatcher = Dispatchers.Default
    val start = CoroutineStart.LAZY
    val job = scope.launch(dispatcher, start) {
        println("coroutine by launch")
    }
    job.start()
    delay(999*1000L)
}
```

##### Coroutine Essentials

a coroutine contains may composed of many essentials

now, let's introduce them one by one, please take patience here

- CoroutineContext : all essentials that determine how coroutine works
- CoroutineScope : offer a scope of current coroutine, determine which coroutine you are writting in
- CoroutineDispatcher : decide which thread coroutine work on
- CoroutineStart : decide when to execute coroutine block
- CoroutineExceptionHandler : how to handle exception when error occurs
- ContinuationInterceptor : intercept current coroutine, and create a new coroutine based on current
- most common implementation of ContinuationInterceptor is CoroutineDispatcher
- Job : present the task that coroutine handles, can be cancelled manually
- CoroutineName : give a name for current coroutine

##### CoroutineContext

CoroutineContext is the base class of most coroutine essential instances

coroutine context can be a single essential, or a collection of multiple essentials

context can be a standalone essential, also can hold some child essentials

likes a data struct below

```kotlin
class Context : Map<Key, Context>
```

a coroutine context can be added to another context, then form a new context contains both of them

if key is not contained in current context, add it, otherwise, replace it

```kotlin
val newCoroutineContext = currentContext + contextItem
```

coroutine essentials can be obtained from context through specific key

```kotlin
val job = context[Job]
val name = context[CoroutineName]
val dispather = context[CoroutineDispatcher]
val interceptor = context[ContinuationInterceptor]
val errorHandler = context[CoroutineExceptionHandler]
```

`CoroutineContext` has a direct child interface called `CoroutineContext.Element`

most classes are directly inherited from `Element` but not `Context`

they are actually the same thing

but `Element` emphas the class is intented to resolve specific requirement, not act as a collection

##### CoroutineScope

coroutine scope hold a coroutine context object named `coroutineContext`

`coroutineContext` holds all essentials for current coroutine

design intents of `CoroutineScope` can cover two aspects

- hide internal functions of actual CoroutineImpl
- offer a block scope object to control current coroutine

coroutine scope can be cancelled, when scope is cancelled, coroutine is cancelled

in fact,  coroutine scope cancell coroutine by delegate object obtained from `coroutineContext[Job]`

##### Predefined CoroutineScope

- GlobalScope

  an empty scope, cannot be cancelled, always available

- LifecycleScope

  bind with android LifecycleOwner, when lifecycle is destroyed, scope will be cancelled

- ViewModelScope

  bind with android ViewModel, when ViewModel is cancelled, scope will be cancelled

##### Predefined Dispatchers

- Dispatchers.Default

  post to default thread pool, designed to handle computation work

- Dispatchers.IO

  post to default thread pool, designed to handle computation work

- Dispatchers.Main

  post to ui thread, designed to handle ui work, implementation depend on platform adapter

- Dispatchers.Unconfined

  not change thread, execute immediately

there are some details we should concern

- Dispatchers.Default and Dispatchers.IO share the same thread pool

  but each thread has a flag, to indicate whether receive cpu-intensive-work or memory-intensive-work

- when Dispatchers.Unconfined is continuously used in child coroutines

  tasks will be post to event loop queue, maintained in current thread, to avoid StackOverflow error

##### Predefined CoroutineStart

`CoroutineStart` define the start strategy of coroutines

before introduce of those strategies, let's understand differences between `dispatch` and `execute` first

`execute` means `coroutine block` is executed

`dispatch` means `coroutine block` is post to thread or task queue, but not executed yet

`execute` always happens behind `dispatch`

- CoroutineStart.DEFAULT

  dispatched immediately, can be cancelled before dispatch

- CoroutineStart.ATOMIC

  dispatched immediately, but cannot be cancelled until block suspended

- CoroutineStart.LAZY

  not dispatched, until start api is actively called, such as `start` `join` `await`

- CoroutineStart.UNDISPATCHED

  executed immediately in current calling-stack, not dispatched, until block's first suspend

`CoroutineStart.DEFAULT` and `CoroutineStart.LAZY` are the most common ways

while `CoroutineStart.ATOMIC` and `CoroutineStart.UNDISPATCHED` are design for special scenarios

##### Job

represent the task coroutine handles, returned when coroutine is created

can be used to start or cancel a coroutine, and query coroutine state

- start : start coroutine
- cancel : cancel coroutine
- join : join another coroutine and wait completed
- parent : get parent job
- isCancelled : query whether coroutine is cancelled
- isCompleted : query whether coroutine is completed
- invokeOnCompletion : set cancel or complete callback

##### Create a Coroutine

`kotlinx.coroutines` framework offers several ways to create coroutines

- CoroutineScope.launch

  create a coroutine, without blocking current calling stack

- CoroutineScope.async

  create a coroutine, with a `FutureResult` called `Deferred` returned, not blocking current calling stack

  value in Deferred can be got by `await`, blocking current calling stack until value returned

- withContext(CoroutineContext, CoroutineScope.() -> R)

  create a coroutine, with specified context, blocking current calling stack until block finished and value returned

- CoroutineDispatcher.invoke(CoroutineScope.() -> R)

  create a coroutine, with specified dispatcher, blocking current calling stack until block finished and value returned

  in fact, this is a inline function, actually calls `withContext(dispatcher, block)`

- runBlocking(CoroutineContext, CoroutineScope.() -> T)

  create a coroutine, with specified context, blocking until block finished and value returned

  this function will not only block calling stack, but also block current thread

  it is designed to use suspend-style apis in non-suspend functions

  only suggest using it in main function or unit test functions
  
- coroutineScope(CoroutineScope.() -> R)

  create a coroutine based on current coroutine context, equivalent to `withContext(coroutineContext, block)`

- supervisorScope(CoroutineScope.() -> R)

  like coroutineScope, but child coroutine's exception won't be delivered to parent coroutine

all coroutines will start automatically by default, unless you specify other start strategy

##### ContinuationInterceptor

interceptor can intercept current coroutine, and extend extra operations based on origin coroutine

taking Dispatcher as an example, it handles the same work as the origin coroutine, with a thread switch operation extended

##### CoroutineExceptionHandler

there are two chances can trigger exception

- error occurs, and exception is thrown
- error saved in result, and get result is called

there are several handlers can handle exception

- exception handler from parent coroutine
- exception handler from current coroutine
- exception handler from thread

how exception deliver between coroutine depends on many factors

it is a complex subject, we will talk about in next blog
