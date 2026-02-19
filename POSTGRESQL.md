## Termux Postgresql
### Install
```shell
pkg install postgresql -y
```

### Create a data folder and initialize it
```shell
mkdir -p $PREFIX/var/lib/postgresql
initdb $PREFIX/var/lib/postgresql
```

### Start/stop the server
```shell
postgres -D $PREFIX/var/lib/postgresql
```

### Configure as a service
```shell
mkdir -p $PREFIX/var/service/postgresql/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/postgresql/log/run
touch $PREFIX/var/service/postgresql/run
chmod +x $PREFIX/var/service/postgresql/run
```
`$PREFIX/var/service/postgresql/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
exec postgres -D $PREFIX/var/lib/postgresql
```

### Enable the service and start it
```shell
sv enable postgresql
sv up postgresql
```

### Create database/user and grant user access to database
```shell
createdb [db-name]
psql [db-name]
```
Type into pg console:
```sql
CREATE USER [user-name] WITH PASSWORD '[user-password]';
ALTER DATABASE [db-name] OWNER TO [user-name];
```
