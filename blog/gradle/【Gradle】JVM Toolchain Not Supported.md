add this script to `settings.gradle`

```kotlin
plugins {
    id("org.gradle.toolchains.foojay-resolver-convention") version "0.8.0" apply true
}
```

this is a plugin for gradle script to find matched toolchains
