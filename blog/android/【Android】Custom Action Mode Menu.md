##### Custom EditText

```kotlin
import android.content.Context
import android.util.AttributeSet
import android.view.ActionMode
import androidx.appcompat.R
import androidx.appcompat.widget.AppCompatEditText

class ActionModeEdit : AppCompatEditText {

    constructor(context: Context) : this(context, null)
    constructor(context: Context, attrs: AttributeSet?) : this(context, attrs, R.attr.editTextStyle)
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr) {
        initView()
    }

    private fun initView() {
        customSelectionActionModeCallback = ActionModeCallback()
        customInsertionActionModeCallback = ActionModeCallback()
    }

    override fun startActionMode(delegate: ActionMode.Callback, type: Int): ActionMode {
        val callback = ActionModeCallback2(delegate)
        return super.startActionMode(callback, ActionMode.TYPE_FLOATING)
    }
}
```

##### Define Menu

*res/menu/menu_edit.xml*

```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">
    <item
        android:id="@+id/select"
        android:orderInCategory="1"
        android:title="Select All"
        app:showAsAction="always" />
    <item
        android:id="@+id/cancel"
        android:orderInCategory="2"
        android:title="Cancel"
        app:showAsAction="always" />
</menu>
```

##### Custom Action Mode Menu

```kotlin
import android.view.ActionMode
import android.view.Menu
import android.view.MenuItem
import com.android.code.R

// control menu creation
class ActionModeCallback : ActionMode.Callback {

    override fun onCreateActionMode(mode: ActionMode, menu: Menu): Boolean {
        return true
    }

    override fun onPrepareActionMode(mode: ActionMode, menu: Menu): Boolean {
        menu.clear()
        mode.menuInflater.inflate(R.menu.menu_edit, menu)
        return true
    }

    override fun onActionItemClicked(mode: ActionMode, item: MenuItem): Boolean {
        val id = item.itemId
        when (id) {
            R.id.select -> println("select")
            R.id.cancel -> println("cancel")
        }
        mode.finish()
        return true
    }

    override fun onDestroyActionMode(mode: ActionMode) {

    }
}
```

##### Custom Action Mode Display Rect

```kotlin
import android.graphics.Rect
import android.view.ActionMode
import android.view.Menu
import android.view.MenuItem
import android.view.View

// control display rect of menu
class ActionModeCallback2(
    private val delegate: ActionMode.Callback
) : ActionMode.Callback2() {

    override fun onCreateActionMode(mode: ActionMode, menu: Menu): Boolean {
        return delegate.onCreateActionMode(mode, menu)
    }

    override fun onPrepareActionMode(mode: ActionMode, menu: Menu): Boolean {
        return delegate.onPrepareActionMode(mode, menu)
    }

    override fun onActionItemClicked(mode: ActionMode, item: MenuItem): Boolean {
        return delegate.onActionItemClicked(mode, item)
    }

    override fun onDestroyActionMode(mode: ActionMode) {
        delegate.onDestroyActionMode(mode)
    }

    override fun onGetContentRect(mode: ActionMode, view: View, outRect: Rect) {
        outRect.set(0, 0, view.width, view.height)
    }
}
```

