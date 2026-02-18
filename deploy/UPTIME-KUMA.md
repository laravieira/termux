# Termux Uptime Kuma Deploy
How to deploy Uptime Kuma on the environment set by following this repository guides.

### Install required packages and configure gyp
```shell
pkg update && pkg upgrade -y
pkg install binutils make clang python3 git nodejs-lts -y
pip install setuptools
npm install node-gyp -g
mkdir ~/.gyp && echo "{'variables':{'android_ndk_path':''}}" > ~/.gyp/include.gypi
```

### Download Uptime Kuma on `~/projects` folder
```shell
cd ~/projects
git clone https://github.com/louislam/uptime-kuma.git
cd uptime-kuma
```

### Install packages
```shell
npm install --ignore-scripts --omit=optional
npm run download-dist
```

### Build the sqlite3 package
```shell
sed -i 's/static_cast<napi_typedarray_type>(-1)/static_cast<napi_typedarray_type>(0)/g' node_modules/@louislam/sqlite3/node_modules/node-addon-api/napi.h
npm rebuild @louislam/sqlite3
```

### Change playwright platform checks
###### playwright is not available in Termux
```shell
sed -i "s/process.platform === 'linux'/process.platform === 'android'/" node_modules/playwright-core/lib/server/registry/index.js
```

### Then you can run Uptime Kuma
```shell
node server/server.js
```

### Configure as a service
```shell
mkdir -p $PREFIX/var/service/uptime-kuma/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/uptime-kuma/log/run
touch $PREFIX/var/service/uptime-kuma/run
chmod +x $PREFIX/var/service/uptime-kuma/run
```
`$PREFIX/var/service/uptime-kuma/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/projects/uptime-kuma
exec 2>&1
node server/server.js
```

### Enable the service and start it
```shell
sv enable uptime-kuma
sv up uptime-kuma
```

### Configure a public hostname on the Cloudflare Tunnel
###### See [Cloudflare Tutorial](/CLOUDFLARED.md).
- On your tunnel overview page, click on the "Public Hostname" tab;
- Click on "Add a public hostname";
- Fill the subdomain and domain;
- Select the service connection type, probably "HTTP";
- Type the internal host to the server you want to expose; 
- Save it and wait a fill seconds for the dns to propagate;