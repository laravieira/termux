## Termux SSH Server
### Install
```shell
pkg install openssh -y
```

### Authorize SSH key
Add the public key (a single line per key) at `~/.ssh/authorized_keys`.

### Configure SSHD
`$PREFIX/etc/ssh/sshd_config`
```properties
PrintMotd no
PasswordAuthentication no
Subsystem sftp /data/data/com.termux/files/usr/libexec/sftp-server
# Add max tries if u have many ssh keys to try
MaxAuthTries 16
# Connect only through 10.147.20.100/24 network
ListenAddress 10.147.20.100
# Port 22 is not available on Android
Port 8022
```

### Access SSH
```shell
ssh -p 8022 10.147.20.100
```

### Start/Stop the server
```shell
sshd # to start
pkill sshd # to stop
```

### Termux Services
```shell
sv-enable sshd # enable as a service
sv-reload sshd # reload configs
sv-disable sshd # disable as a service
sv up sshd # start the service
sv down sshd # stop the service
tail -f $PREFIX/var/log/sv/sshd/current # open logs
```
