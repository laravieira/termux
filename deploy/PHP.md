# Termux PHP Project Deploy
How to deploy a php project on the environment set by following this repository guides.

### Download your project on `~/` folder
```shell
cd ~
git clone [your-project-repo]
```

### Configure your project
- Install the dependencies (`composer install`);
- If needed, [create a database/user](#if-you-need-a-mysql-database);
- Add `.env` file and populate it;
- Do anything else u might need to do;
- Test it, if possible, like with `php -S localhost:400 index.php`;

### Configure a V-Host on Apache
Add the next available port on Apache:

`$PREFIX/etc/apache2/httpd.conf`
```properties
Listen 8082
```
Add the v-host config to your project:

`$PREFIX/etc/apache2/extra/httpd-vhosts.conf`
```properties
<VirtualHost *:8082>
    DocumentRoot "/data/data/com.termux/files/home/[your-project-dir]/public_html"
    ServerName [your-project-domain]
    SetEnv ENV production
</VirtualHost>
```
Check if u made a typo on Apache configs:
```shell
httpd -t
```
Reload Apache:
```shell
sv reload httpd && sv up httpd
```

### If you need a mysql database
###### See [MariaDB Tutorial](/MARIADB.md).
```shell
mysql -u root -p
```
```sql
CREATE DATABASE [db-name];
CREATE USER '[user-name]'@'%' IDENTIFIED BY '[user-password]';
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