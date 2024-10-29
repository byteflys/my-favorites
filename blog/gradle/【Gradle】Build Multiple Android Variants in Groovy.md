##### Gradle Script

***app/build.gradle***

```groovy
flavorDimensions("market", "version")

productFlavors {
    create("HUAWEI") {
        dimension = "market"
        manifestPlaceholders["market"] = name
    }
    create("HONOR") {
        dimension = "market"
        manifestPlaceholders["market"] = name
    }
    create("1_0_0") {
        dimension = "version"
        versionCode = 6000
        versionName = "1.0.0"
        manifestPlaceholders["version"] = versionName
    }
    create("2_0_0") {
        dimension = "version"
        versionCode = 7000
        versionName = "2.0.0"
        manifestPlaceholders["version"] = versionName
    }
}

buildTypes {
    debug {
        manifestPlaceholders = []
    }
}

variantFilter { variant ->
    if (variant.buildType.name.endsWith('release')) {
        variant.setIgnore(true)
    }
}

applicationVariants.all { variant ->
    def market = variant.productFlavors.find { it.dimension == "market" }.name
    def version = variant.productFlavors.find { it.dimension == "version" }.name.replaceAll("_", ".")
    variant.outputs.all { output ->
        output.outputFileName = "${market}-${version}.apk"
    }
}
```

##### Build Multiple Apks

`Build - Generate Signed APK`
