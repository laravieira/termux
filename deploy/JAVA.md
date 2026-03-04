# Termux Java Deploy
How to deploy a Java Project on the environment set by following this repository guides.

### Install Java
###### Termux has a limited version of Java.
```shell
pkg install openjdk-25 -y
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

### Have other java version to use without nedding to install
Java 21
```
mkdir ~/projects/openjdk21
cd ~/projects/openjdk21
pkg install wget zip unzip -y
wget https://github.com/zryyoung/openjdk-Termux/releases/download/openjdk-21.0.1/openjdk-21.0.1-aarch64.zip
unzip openjdk-21.0.1-aarch64.zip
mv openjdk-21.0.1/* .
rm -r openjdk-21.0.1-aarch64.zip openjdk-21.0.1
cd ~
~/projects/openjdk21/bin/java --version
```
Java 17
```
mkdir ~/projects/openjdk17
cd ~/projects/openjdk17
pkg install wget xz-utils -y
wget https://github.com/zryyoung/openjdk-Termux/releases/download/openjdk-17/openjdk-17-aarch64.tar.xz
tar -xf openjdk-17-aarch64.tar.xz
rm openjdk-17-aarch64.tar.xz
cd ~
~/projects/openjdk17/bin/java --version
```
Java 11
```
mkdir ~/projects/openjdk11
cd ~/projects/openjdk11
pkg install wget zip unzip -y
wget https://github.com/zryyoung/openjdk-Termux/releases/download/openjdk-11.0.12/openjdk-11.0.12-aarch64.zip
unzip openjdk-11.0.12-aarch64.zip
rm openjdk-11.0.12-aarch64.zip
cd ~
~/projects/openjdk11/bin/java --version
```