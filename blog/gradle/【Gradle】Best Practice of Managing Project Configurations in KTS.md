##### Configure System Properties

system properties can be defined in these ways

- system global `gradle.properties` file
- project `gradle.properties` file
- command-line parameters
- set by `setProperty(name, object)` (only for existed properties)

properties defined in `gradle.properties` file like this

```properties
android.useAndroidX=true
kotlin.code.style=official
```

properties can delivered from command-line like this

```shell
./gradlew assemble  -Pandroid.useAndroidX=true  -Pkotlin.code.style=official
```

##### Read System Properties

properties saved in global scope of gradle script

you can read it anywhere in script like this

```kotlin
property("org.gradle.jvmargs")
```

##### Create And Read from Config File

kts script can load a sub script from another file, but only `.gradle` file is supported

***config.gradle***

```groovy
def config = new Properties()
rootProject.extensions["config"] = config

config["versionCode"] = 100
config["versionName"] = "2.0.100"
```

***build.gradle.kts***

```kotlin
import java.util.Properties

plugins {
    id("org.jetbrains.kotlin.android")
    id("com.android.application")
}

apply {
    from("config.gradle")
}

inline fun <reified T> config(name: String): T {
    val properties = rootProject.extensions["config"] as Properties
    return properties[name] as T
}

android {

    defaultConfig {
        versionCode = config("versionCode")
        versionName = config("versionName")
    }
}
```

##### Modify Android Manifest Variables

you can dynamically modify value of an android manifest field like this

manifest fields can be read by android code later

***AndroidManifest.xml***

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest>
  <application>
    <meta-data
      android:name="market"
      android:value="${market}" />
  </application>
</manifest>
```

***build.gradle.kts***

```kotlin
android {
    buildTypes {
        create("Base") {
            isDefault = true
            isDebuggable = true
            isMinifyEnabled = false
            signingConfig = signingConfigs.getByName("common")
            manifestPlaceholders["market"] = "Huawei"
        }
    }
}
```

***HomeActivity.kt***

```kotlin
fun Context.getMetaData(key: String): String {
    val info = packageManager.getApplicationInfo(packageName, PackageManager.GET_META_DATA)
    return info.metaData.getString(key).orEmpty()
}
```

##### Inject Variables Into Android Project

`Android-Gradle-Plugin` generate a class named `BuildConfig` to save build configurations

contents of `BuildConfig` class can be defined in gradle script like this

***build.gradle.kts***

```kotlin
android{
  buildFeatures {
    buildConfig = true
  }
  buildTypes {
    create("Base") {
      isDefault = true
      buildConfigField("Boolean", "debuggable", "true")
    }
  }
}
```

***HomeActivity.kt***

```kotlin
println(BuildConfig.debuggable)
```

injected variables only take effect after a project rebuild

if you cannot access it, please run `Build - Rebuild Project` first