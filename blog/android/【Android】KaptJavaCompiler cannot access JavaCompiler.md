##### KaptJavaCompiler cannot access JavaCompiler

```bash
class org.jetbrains.kotlin.kapt3.base.javac.KaptJavaCompiler cannot access class com.sun.tools.javac.main.JavaCompiler
```

##### Reason

Kapt Plugin not compatible with JDK version

##### Solution

- force allow illegal access of gradle apis

- use JDK version less than 16

##### Allow Illegal Access

open global gradle properties file

```bash
open ~/.gradle/gradle.properties
```

add this property

```bash
org.gradle.jvmargs=--illegal-access=permit
```

rebuild project
