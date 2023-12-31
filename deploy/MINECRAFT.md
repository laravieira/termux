# Termux Minecraft Deploy
How to deploy a Minecraft Server on the environment set by following this repository guides.

### Install Java
Termux has a limited version of Java, for Minecraft u will see ethernet interfce errors, since Termux without root doesn't have access to it.

So this tutorial will configure minecraft server inside Ubuntu on Ternux, where we can get a full Java version.

###### Install Ubuntu using [Ubuntu 20 CLI Only install](https://github.com/tuanpham-dev/termux-ubuntu).
```shell
pkg install wget curl proot tar -y
wget https://raw.githubusercontent.com/tuanpham-dev/termux-ubuntu/master/ubuntu.sh
chmod +x ubuntu.sh
bash ubuntu.sh nde
```
Inside Ubuntu install Java 17
```shell
sudo apt install openjdk-17-jdk -y
```

### Useful plugins for Paper/Spigot/Bukkit
- [DriverBackupV2](https://dev.bukkit.org/projects/drivebackupv2) Periodically backup your server to the cloud.
- [LuckPerms](https://luckperms.net/) Manage permissions and chat style.
- [DiscordSRV](https://www.spigotmc.org/resources/discordsrv.18494/) Minecraft status/console on Dsicord.
- [LPC Chat Formatter](https://www.spigotmc.org/resources/lpc-chat-formatter-1-7-10-1-20.68965/) Change chat style, requires LuckPerms.
- [SkinRestorer](https://www.spigotmc.org/resources/skinsrestorer.2124/) Only if `online-mode=false`, restorer skins on pirate servers.

### Configure your server
- Configure and test your server your machine ([Paper](https://papermc.io/downloads/paper));
- Upload your server files to `~/ubuntu20-fs/home/[user]/minecraft`, easiest way is using SFTP ([FileZilla](https://filezilla-project.org/), [Cyberduck](https://cyberduck.io/));

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
To get inside Ubuntu use:
```shell
~/start-ubuntu20.sh
```
Inside Ubuntu, start your server with:
```shell
java -Xmx2G -Xms512M -jar paper.jar --nogui
```
### Ubuntu Details
- When u close the console windows or use `exit` the Ubuntu instance will be hard killed, incluing your server, this can (probably will) corrupt you world files.
