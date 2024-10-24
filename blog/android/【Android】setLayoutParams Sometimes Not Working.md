setLayoutParams may fail when called during another layout period

or inner of a callback function in layout period

we can use `post` to setLayoutParams in next layout period

```kotlin
binding.image.post {
    val fixedHeight = (binding.image.measuredWidth / ratio).toInt()
    binding.image.setLayoutParams { height = fixedHeight }
}
```

