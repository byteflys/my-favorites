try two solutions below

##### Set Gradle Properties File

```bash
org.gradle.jvmargs=-Xmx16g -Dfile.encoding=UTF-8
org.gradle.parallel=false
```

##### Set Run Configurations

Gradle - Tasks - Build - Assemble - Right Click - Modify Run Configurations

Modify Options - Add VM Options - Add Args Below

```bash
-Xmx16g
```

right click your gradle task, and continue to run again