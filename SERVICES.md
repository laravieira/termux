## Termux Services
### Install
```shell
pkg install termux-services -y
```
After installing, restart the termux completly, for the daemon to start.

### Base Commands
```shell
sv-enable [service] # enable as a service
sv-reload [service] # reload configs
sv-disable [service] # disable as a service
sv up [service] # start the service
sv down [service] # stop the service
tail -f $PREFIX/var/log/sv/[service]/current # open logs
```

### Create a Service
Create the service folder and its log folder:
```shell
mkdir -p $PREFIX/var/service/[srevice]/log
```
Create a link of termux-services logger to your service log folder:
```shell
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/[service]/log/run
```
Create the run file for your service:
```shell
touch $PREFIX/var/service/[service]/run
```
Give execution permission to the run file:
```shell
chmod +x $PREFIX/var/service/[service]/run
```
At this point your service is created, but not enabled.


### Populating the run file
`$PREFIX/var/service/[service]/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
exec [command] 2>&1
```

Custom folder:
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd [folder]
exec 2>&1
[command]
```

Set environment variables:
```shell
#!/data/data/com.termux/files/usr/bin/sh
exec 2>&1
ENV=VALUE \
ENV2=VALUE2 \
[command]
```
###### The file has to start with the commented bash line

### Termux:Boot
To start the sevices at device boot, change the boot file to:

`~/.termux/boot/[start_file]`
```shell
#!/data/data/com.termux/files/usr/bin/sh
termux-wake-lock
. $PREFIX/etc/profile
```