##### Intent

build an activity looks like a dialog, float over main activity or desktop

##### Create Transparent Theme

```xml
<?xml version="1.0" encoding="utf-8"?>
<resources>

    <style name="Theme.TransparentActivity" parent="Theme.Material3.Light.NoActionBar">
        <item name="android:windowBackground">@color/transparent</item>
        <item name="android:windowNoTitle">true</item>
        <item name="android:windowIsTranslucent">true</item>
        <item name="android:windowAnimationStyle">@android:style/Animation.Translucent</item>
    </style>
</resources>
```

##### Apply Transparent Theme to Activity

```xml
<activity
    android:name=".viewlocator.DialogActivity"
    android:theme="@style/Theme.TransparentActivity" />
```

##### Make Activity Non-Fullscreen

```kotlin
class DialogActivity : AppCompatActivity() {

    private lateinit var binding: ActivityHomeBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityHomeBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
  
    override fun onWindowFocusChanged(hasFocus: Boolean) {
        super.onWindowFocusChanged(hasFocus)
        if (hasFocus) {
            val lp = window.attributes
            lp.width = 500
            lp.height = 500
            lp.gravity = Gravity.CENTER
            window.attributes = lp
            window.setBackgroundDrawable(ColorDrawable(Color.TRANSPARENT))
        }
    }
}
```

