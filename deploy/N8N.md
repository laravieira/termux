# Termux N8N Deploy
How to deploy N8N on the environment set by following this repository guides.

### Install required packages
```shell
pkg update && pkg upgrade -y
pkg install nodejs-lts -y
```

### Download N8N globally
```shell
npm install n8n -g
```

### Create postgres database and user
```shell
createdb n8n
psql n8n
```
Type into pg console:
```sql
CREATE USER n8n WITH PASSWORD '[user-password]';
ALTER DATABASE n8n OWNER TO n8n;
```

### Then you can run n8n
```shell
export DB_TYPE=postgresdb
export DB_POSTGRESDB_USER=n8n
export DB_POSTGRESDB_PASSWORD=[user-password]
export N8N_SECURE_COOKIE=false
n8n start
```

### Configure as a service
```shell
mkdir -p $PREFIX/var/service/n8n/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/n8n/log/run
touch $PREFIX/var/service/n8n/run
chmod +x $PREFIX/var/service/n8n/run
```
`$PREFIX/var/service/n8n/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
exec 2>&1
DB_TYPE=postgresdb \
DB_POSTGRESDB_HOST=localhost \
DB_POSTGRESDB_PORT=5432 \
DB_POSTGRESDB_DATABASE=n8n \
DB_POSTGRESDB_USER=n8n \
DB_POSTGRESDB_PASSWORD=[user-password] \
N8N_DEFAULT_BINARY_DATA_MODE=filesystem \
N8N_SECURE_COOKIE=true \
N8N_RUNNERS_ENABLED=false \
NODE_OPTIONS=--max_old_space_size=3048 \
N8N_PAYLOAD_SIZE_MAX=8192 \
N8N_FORMDATA_FILE_SIZE_MAX=8192 \
NODES_EXCLUDE=[] \
WEBHOOK_URL=https://[host] \
N8N_EDITOR_BASE_URL=https://[host] \
N8N_HOST=https://[host] \
N8N_ENFORCE_SETTINGS_FILE_PERMISSIONS=true \
GENERIC_TIMEZONE=America/Sao_Paulo \
TZ=America/Sao_Paulo \
n8n start
```

### Enable the service and start it
```shell
sv enable n8n
sv up n8n
```

### Configure a public hostname on the Cloudflare Tunnel
###### See [Cloudflare Tutorial](/CLOUDFLARED.md).
- On your tunnel overview page, click on the "Public Hostname" tab;
- Click on "Add a public hostname";
- Fill the subdomain and domain;
- Select the service connection type, probably "HTTP";
- Type the internal host to the server you want to expose; 
- Save it and wait a fill seconds for the dns to propagate;