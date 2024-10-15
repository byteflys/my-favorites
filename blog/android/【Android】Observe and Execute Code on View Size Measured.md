##### Issue

When an android view was just created, its size is zero

If we want to do some work depend on its size at this time, it does not work

So we need a callback to get notify from view, when it is measured

#### Solution

In earlier android versions, we can achieve this by using :

`addOnLayoutChangeListener` `isLaidOut` `isLayoutRequested`

- addOnLayoutChangeListener can observe layout changes
- isLaidOut means view have size that greater than 0
- isLayoutRequested means layout will change later, it's not the final value we want

In latest kotlin android extension library, we have a fast way to do these steps

there is a extend function of View called `doOnLayout` , just write code like this

```kotlin
view.doOnLayout {
  println(it.measuredWidth)
  println(it.measuredHeight)
}
```

##### Sumary

It's a simple extend function, but greatly changed android ui programming

You need not to write some code like `view.post()` or `view.postDelayed()` any more
