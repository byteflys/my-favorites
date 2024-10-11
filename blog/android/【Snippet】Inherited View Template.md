default style must be correctly set, otherwise may cause unknwon problems

```kotlin
import android.content.Context
import android.util.AttributeSet
import androidx.appcompat.widget.AppCompatEditText

class ActionModeEdit : AppCompatEditText {

    constructor(context: Context) : this(context, null)
    constructor(context: Context, attrs: AttributeSet?) : this(context, attrs, R.attr.editTextStyle)
    constructor(context: Context, attrs: AttributeSet?, defStyleAttr: Int) : super(context, attrs, defStyleAttr) {
        parseAttribute(attrs)
        initView()
    }

    private fun parseAttribute(attributeSet: AttributeSet?) {
        val typedArray = context.obtainStyledAttributes(attrs, R.styleable.ActionModeEdit)
        if (typedArray.hasValue(R.styleable.ActionModeEdit_padding)) {
            padding = typedArray.getDimension(R.styleable.ActionModeEdit_padding, 0f)
        }
        typedArray.recycle()
    }

    private fun initView() {

    }
}
```

