# Termux MCSManager Deploy
How to deploy the MCS Manager on the environment set by following this repository guides.

### Requirements
- NodeJS
- Java

### Install the MCSManager from git
[reference](https://github.com/MCSManager/MCSManager#linux)
```shell
mkdir ~/mcsmanager
cd ~/mcsmanager

wget https://github.com/MCSManager/MCSManager/releases/latest/download/mcsmanager_linux_release.tar.gz
tar -zxf mcsmanager_linux_release.tar.gz

./install-dependency.sh
```

### Test the service
The daemon instance:
```shell
./start-daemon.sh
```
The web painel instance:
```shell
./start-web.sh
```
When open the painel for the first time, you will create an admin account.

Access it using http://10.147.20.100:23333

### Setup as a service
You need to services to run the MCSManager, the Daemon and the Painel
```shell
mkdir -p $PREFIX/var/service/mcsmanager-daemon/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/mcsmanager-daemon/log/run
touch $PREFIX/var/service/mcsmanager-daemon/run
chmod +x $PREFIX/var/service/mcsmanager-daemon/run
```
`$PREFIX/var/service/mcsmanager-daemon/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/mcsmanager/daemon
exec 2>&1
node app.js
```
```shell
mkdir -p $PREFIX/var/service/mcsmanager-painel/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/mcsmanager-painel/log/run
touch $PREFIX/var/service/mcsmanager-painel/run
chmod +x $PREFIX/var/service/mcsmanager-painel/run
```
`$PREFIX/var/service/mcsmanager-painel/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/mcsmanager/web
exec 2>&1
node app.js
```

### Enable the services and start them
```shell
sv-enable mcsmanager-daemon
sv-enable mcsmanager-painel
sv up mcsmanager-daemon
sv up mcsmanager-painel
```

### Configure your server
- Configure and test your server your machine ([Paper](https://papermc.io/downloads/paper));
- Upload your server files to `~/[my-server]`, easiest way is using SFTP ([FileZilla](https://filezilla-project.org/), [Cyberduck](https://cyberduck.io/));

Files/folders u need to upload:
```yaml
#.console_history
#backups
banned-ips.json
banned-players.json
bukkit.yml
#cache
commands.yml
config
eula.txt
help.yml
#libraries
#logs
ops.json
paper.jar
permissions.yml
plugins
server-icon.png
server.properties
spigot.yml
usercache.json
#version_history.json
#versions
whitelist.json
world
world_nether
world_the_end
```

On the MCSManager:
1. Click on Applications
2. In the "Instance List", make sure the daemon you want to run the server is selected.
3. Click "+ New Instance"
4. Minecraft Server: Java Edition
5. Upload nothing os select existing file.
6. Give it a name
7. Type the start command: 
```shell
java "-DPaper.IgnoreJavaVersion=true" -Xmx2G -Xms512M -jar paper.jar -nogui
```
8. Type the server directory:
```shell
/data/data/com.termux/files/home/[my-server]
```
9. Create instance
10. Go to the console and start the server instance

You will see two errors while executing the server
- Failed retrieving info for group processor
- Failed retrieving info for group memory
You can just ignore them, they are because you don't have root access to fetch CPU and Memory info.