## Termux Nginx
### Install
```shell
pkg install nginx -y
```

### Using termux-services:
```shell
rm $PREFIX/var/service/nginx/down
sv enable nginx # enable as a service
sv up nginx # start the service
sv down hnginxd # stop the service
```

### Edit configuration file
```shell
nano $PREFIX/etc/nginx/nginx.conf
```

### Test configuration files
```shell
nginx -t
```

### Always reload Nginx
Remember to always reload Nginx after changing its config and after changing php.ini as well.
Using termux-services:
```shell
sv reload nginx && sv up nginx
```

