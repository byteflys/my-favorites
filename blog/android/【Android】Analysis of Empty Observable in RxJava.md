##### About Empty Observable

In rxjava, there is a function called `Observable.empty()` 

This function return an object that is instance of `ObservableEmpty`

This class will emit a complete event instead of next event, then rx chain will ended

##### Empty Observable with Map Operator

Now let's test what will happen, when `ObservableEmpty` is not top level observables

I means it was mapped from other observables by operator such as `flatMap`

##### Trial Code

```kotlin
Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9)
    .flatMap {
        if (it != 5) {
            return@flatMap Observable.just(it.toString())
        }
        return@flatMap Observable.empty<String>()
    }
    .doOnNext { println(it) }
    .subscribe()
```

We get this result

```shell
1
2
3
4
6
7
8
9
```

When it's a mediator observable, it won't interrupt all top observables

It just break the sub event stream that related  to itself

##### Empty Observable with ConcatArray Operator

This time, we make it the top ones, but not the single one

We combine it with other normal observables by `concatArray` , as top level observable in total

```kotlin
Observable.concatArray(
    Observable.just(1),
    Observable.just(2),
    Observable.just(3),
    Observable.empty(),
    Observable.just(4),
    Observable.just(5)
).flatMap {
    if (it != 5) {
        return@flatMap Observable.just(it.toString())
    }
    return@flatMap Observable.empty<String>()
}
    .doOnNext { println(it) }
    .subscribe()
```

This is the output

```shell
1
2
3
4
```

We can see that, `ObservableEmpty` interrupt other observables in the same level

Also, it interrupt the whole rx chain, directlly jump to `onComplete`

##### Summary

- `ObservableEmpty` will interrupt datas or observables mapped from it

- `ObservableEmpty` will interrupt observables at same level, but after it in order

##### Bless

Lesson is Over, Have A Rest, and Enjoy Your Life .

Good Work, Good Study, Good Progress, and Good Mood !