# Termux Java Deploy
How to deploy a Java Project on the environment set by following this repository guides.

### Install Java
###### Termux has a limited version of Java.
```shell
pkg install openjdk-17 -y
```
You can use a complete version of Java by installing Ubuntu inside Termux.
- [Hosting Minecraft Server on Termux with Ubuntu](https://www.reddit.com/r/Android/comments/czau7s/hosting_a_minecraft_server_on_android/)

### Start your server
```shell
java -Xmx1G -Xms512M -jar [project].jar --nogui
```

### Enable as service
```shell
mkdir -p $PREFIX/var/service/[project]/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/[project]/log/run
touch $PREFIX/var/service/[project]/run
chmod +x $PREFIX/var/service/[project]/run
```
`$PREFIX/var/service/[project]/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/[project]
exec 2>&1
java -Xmx1G -Xms512M -jar target/[project].jar --nogui
```
Enable the service and start it
```shell
sv enable [project] && sv up [project]
```
