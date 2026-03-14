# termux
Termux tutorials for settings up a server on my phone

### Install
Download F-Droid store app and install it.
First install Termux from F-Droid and open it.

### Things to do on your Termux
```shell
pkg upgrade -y
pkg install git nodejs yarn php composer openjdk-25 maven wget -y
```

### Enable storage access
On your phone, type:
```shell
termux-setup-storage
```

### Disable All Performance Controls
On Galaxy phones you can:
- [x] Enable Wakelock on the Termux notification
- [x] Disable Adaptive battery
- [x] Disable Put unused apps to sleep
- [x] Set power mode to high performance
- [x] Disable adaptive power saving
- [x] Exclude Termux from memory cleaning
- [x] With developer debbug, set `adb shell settings put global settings_enable_monitor_phantom_procs false`
- [x] In developer options, disable "Logger buffer"
- [x] Install Termux:Float and keep the floating window always open on screen
- [x] Increase virtual memory