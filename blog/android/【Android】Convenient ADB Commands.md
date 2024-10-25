##### Install

```bash
adb install -r <path>
```

##### Uninstall

```bash
adb uninstall <pkg>
```

##### Start

```bash
adb shell am start -n <pkg>/.SplashActivity
```

##### Stop

```bash
adb shell am force-stop <pkg>
```

#### Reset

```bash
adb shell pm clear <pkg>
```

##### Reboot

```bash
adb reboot
```

