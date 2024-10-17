##### Create Receiver

```kotlin
class SystemLanguageReceiver : BroadcastReceiver() {

    override fun onReceive(context: Context, intent: Intent) {
        // do clean or switch work
    }
}
```

##### Register Receiver

```kotlin
private fun registerSystemLanguageReceiver() {
    val receiver = SystemLanguageReceiver()
    val filter = IntentFilter(Intent.ACTION_LOCALE_CHANGED)
    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU) {
        registerReceiver(receiver, filter, Context.RECEIVER_EXPORTED)
    } else {
        registerReceiver(receiver, filter)
    }
}
```

