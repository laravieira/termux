### Termux crontab
```shell
pkg install cronie -y
```
You can run it with
```shell
crond
```

### Enable as service
Enable the service and start it
```shell
rm $PREFIX/var/service/crond/down
sv enable crond
sv up crond
```
