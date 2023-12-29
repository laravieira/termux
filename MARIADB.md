## Termux MariaDB (MySQL Server)
### Install
```shell
pkg install mariadb -y
```
After install, make the server secure by setting the root password and disabled everything else:
```shell
mariadb-secure-installation
```

### Start/stop the server
```shell
mariadbd
pkill mariadbd
```
Using termux-services:
```shell
sv-enable mysqld # enable as a service
sv up mysqld # start the service
sv down mysqld # stop the service
```

### Coonect to mysql as root
```shell
mysql -u root -p
```

### Create database/user and grant user access to database
```sql
CREATE DATABASE [db-name];
CREATE USER '[user-name]'@'%' IDENTIFYED BY '[user-password]';
GRANT ALL PRIVILEGES ON [db-name].* TO '[user-name]'@'%';
FLUSH PRIVILEGES;
```
###### Change `%` to the desired host if u want the user to only be able to access from that host enviroment.

### Drop user and its permissions
```sql
REVOKE ALL PRIVILEGES ON *.* FROM '[user-name]'@'%';
FLUSH PRIVILEGES;
DROP USER '[user-name]'@'%';
```
###### Change `%` to the right host if u changed it on creation.

### Drop database
```sql
DROP DATABASE [db-name];
```

### Create a table for database
```sql
CREATE TABLE [db-name].[table-name] ([colunms]);
```
