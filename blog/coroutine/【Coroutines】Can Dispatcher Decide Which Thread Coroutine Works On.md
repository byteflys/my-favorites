let's create a dispatcher through `XDispatchers.share()`

this api return a thread-pool that has 10 shared threads

##### Case 1

```kotlin
suspend fun main() {
    val dispatcher = XDispatchers.share()
    GlobalScope.launch(dispatcher) {
        for (i in 1..5) {
            delay(100)
            printWithThreadInfo("1")
        }
    }
    delay(10 * 1000L)
}
```

```bash
1 <tid=23 XDispatchers-share-3cd2b6>
1 <tid=23 XDispatchers-share-3cd2b6>
1 <tid=23 XDispatchers-share-3cd2b6>
1 <tid=23 XDispatchers-share-3cd2b6>
1 <tid=23 XDispatchers-share-3cd2b6>
```

##### Case 2

```kotlin
suspend fun main() {
    val dispatcher = XDispatchers.share()
    GlobalScope.launch(dispatcher) {
        for (i in 1..5) {
            delay(100)
            printWithThreadInfo("1")
        }
    }
    GlobalScope.launch(dispatcher) {
        for (i in 1..5) {
            delay(100)
            printWithThreadInfo("2")
        }
    }
    delay(10 * 1000L)
}
```

```bash
2 <tid=24 XDispatchers-share-67436d>
1 <tid=25 XDispatchers-share-b5cee3>
2 <tid=25 XDispatchers-share-b5cee3>
1 <tid=24 XDispatchers-share-67436d>
2 <tid=24 XDispatchers-share-67436d>
1 <tid=25 XDispatchers-share-b5cee3>
2 <tid=25 XDispatchers-share-b5cee3>
1 <tid=24 XDispatchers-share-67436d>
1 <tid=25 XDispatchers-share-b5cee3>
2 <tid=24 XDispatchers-share-67436d>
```

##### Conclusions

- dispatcher can only partially decide coroutine's working thread

- when thread-pool is single-instance, coroutine will work on single thread

- when thread-pool is only used by current coroutine, coroutine will work on single thread

  as thread will enter idle state, when coroutine suspend, and reused after next resume

- when thread-pool has multiple threads, and shared by other coroutines, result in unsure