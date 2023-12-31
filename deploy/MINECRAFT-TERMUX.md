# Termux Minecraft Deploy
How to deploy a Minecraft Server on the environment set by following this repository guides.

### Install Java
###### Termux has a limited version of Java, for Minecraft u will see ethernet interfce errors, since Termux without root doesn't have access to it.
```shell
pkg install openjdk-17 -y
```
You can use a complete version of Java by installing Ubuntu inside Termux.
- [Hosting Minecraft Server on Termux with Ubuntu](https://www.reddit.com/r/Android/comments/czau7s/hosting_a_minecraft_server_on_android/)

### Useful plugins for Paper/Spigot/Bukkit
- [DriverBackupV2](https://dev.bukkit.org/projects/drivebackupv2) Periodically backup your server to the cloud.
- [LuckPerms](https://luckperms.net/) Manage permissions and chat style.
- [DiscordSRV](https://www.spigotmc.org/resources/discordsrv.18494/) Minecraft status/console on Dsicord.
- [LPC Chat Formatter](https://www.spigotmc.org/resources/lpc-chat-formatter-1-7-10-1-20.68965/) Change chat style, requires LuckPerms.
- [SkinRestorer](https://www.spigotmc.org/resources/skinsrestorer.2124/) Only if `online-mode=false`, restorer skins on pirate servers.

### Configure your server
- Configure and test your server your machine ([Paper](https://papermc.io/downloads/paper));
- Upload your server files to `~/minecraft`, easiest way is using SFTP ([FileZilla](https://filezilla-project.org/), [Cyberduck](https://cyberduck.io/));

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

### Start your server
You have to add the `-DPaper.IgnoreJavaVersion=true` system property, since Termux has a limited Java version and paper don't support it.
```shell
java -DPaper.IgnoreJavaVersion=true -Xmx2G -Xms512M -jar paper.jar --nogui
```

### Use your server as service
- If you use minecraft as a service, make sure u have a way to nicely stop your server, like permission to `/stop` inside the game, as a service u don't have access to the console input.
- Use of `sv down minecraft` or `pkill java` can (and probably will) corrupt your world.
```shell
mkdir -p $PREFIX/var/service/minecraft/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/minecraft/log/run
touch $PREFIX/var/service/minecraft/run
chmod +x $PREFIX/var/service/minecraft/run
```
`$PREFIX/var/service/minecraft/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/minecraft
exec 2>&1
java -DPaper.IgnoreJavaVersion=true -Xmx2G -Xms512M -jar paper.jar --nogui
```
Enable the service and start it
```shell
sv enable minecraft && sv up minecraft
```
