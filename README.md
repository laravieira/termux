# termux
Termux tutorials for settings up a server on my phone

### Install
Download F-Droid store app and install it.
First install Termux from F-Droid and open it.

### Things to do on your Termux
```shell
pkg upgrade -y
pkg install git nodejs yarn php composer openjdk-17 maven yarn wget -y
```

### Disable All Performance Controls
On Galaxy phones you can:
- [x] Enable Wakelock on the Termux notification
- [x] Disable Adaptive battery
- [x] Disable Put unused apps to sleep
- [x] Set power mode to high performance
- [x] Disable adaptive poser saving
- [x] Exclude Termux from memory cleaning
- [x] Lock Termux app
- [x] With developer debbug, set `adb shell settings put global settings_enable_monitor_phantom_procs false`
- [x] Increase virtual memory