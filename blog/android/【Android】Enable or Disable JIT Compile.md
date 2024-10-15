##### Enable/Disable JIT

```properties
adb root
adb shell stop
adb shell setprop dalvik.vm.usejit false
adb shell start
```

##### Force Compile

```properties
adb shell cmd package compile -m speed -f com.onyx.galaxy.note
```

##### Clean Compiled Data

```properties
adb shell pm compile --reset -f com.onyx.galaxy.note
```

##### Optional Modes

```properties
assume-verified
extract
verify
quicken
space-profile
space
speed-profile
speed
everything
```



