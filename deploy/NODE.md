# Termux Node.js Project Deploy
How to deploy a Node.js project on the environment set by following this repository guides.

### Download your project on `~/` folder
```shell
cd ~
git clone [your-project-repo]
```

### Configure your project
- Install the dependencies (`npm install`);
- If needed, [create a database/user](#if-you-need-a-mysql-database);
- Add `.env` file and populate it;
- Do anything else u might need to do;
- Test it, if possible, like with `node build/index.js`;

### Create a service for your project
```shell
mkdir -p $PREFIX/var/service/[project]/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/[project]/log/run
touch $PREFIX/var/service/[project]/run
chmod +x $PREFIX/var/service/[project]/run
```
`$PREFIX/var/service/[project]/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/[your-project-dir]
exec 2>&1
PORT=4000 \
node ./build/index.js
```

### Enable the service and start it
```shell
sv-enable [project]
sv up [project]
```


### If you need a mysql database
###### See [MariaDB Tutorial](/MARIADB.md).
```shell
mysql -u root -p
```
```sql
CREATE DATABASE [db-name];
CREATE USER '[user-name]'@'%' IDENTIFYED BY '[user-password]';
GRANT ALL PRIVILEGES ON [db-name].* TO '[user-name]'@'%';
FLUSH PRIVILEGES;
```

### Configure a public hostname on the Cloudflare Tunnel
###### See [Cloudflare Tutorial](/CLOUDFLARED.md).
- On your tunnel overview page, click on the "Public Hostname" tab;
- Click on "Add a public hostname";
- Fill the subdomain and domain;
- Select the service connection type, probably "HTTP";
- Type the internal host to the server you want to expose; 
- Save it and wait a fill seconds for the dns to propagate;