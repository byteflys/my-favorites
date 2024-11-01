##### Build Drag Data

build a ClipData and specify an Intent, to deliver data that you want post on drop

```kotlin
val intent = Intent()
val data = ClipData.newIntent("CustomDragType", intent)
```

##### Start Drag

```kotlin
val shadow = DragShadowBuilder(view)
view.startDragAndDrop(data, shadow, null, 0)
```

##### Response DragAndDrop Event

there are two ways to drop an element

- drop onto a view
- drop into blank spaces between serveral views

in case 1, you can use the target view responsing drop event

in case 2, you can use a common parent view responsing drop event

then decide which child to handle event, by comparing event's coordinate to view's coordinate

```kotlin
dropView.setOnDragListener { v, event ->
    if (event.action == DragEvent.ACTION_DRAG_ENDED) {
        val point = Point(event.x.toInt(), event.y.toInt())
				val data = event.clipData.getItemAt(0).intent
        handleDropEvent(v, point, data)
        dragView.cancelDragAndDrop()
    }
    return@setOnDragListener true
}
```

##### Utils

```kotlin
fun DragEvent.name() = when (action) {
    ACTION_DRAG_STARTED -> "START"
    ACTION_DRAG_ENTERED -> "ENTER"
    ACTION_DRAG_LOCATION -> "LOCATION"
    ACTION_DRAG_EXITED -> "EXIT"
    ACTION_DROP -> "DROP"
    ACTION_DRAG_ENDED -> "END"
    else -> "NONE"
}

fun DragEvent.info() = "${name()} $x $y"
```



