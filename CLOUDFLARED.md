## Termux SSH Server
### Install
```shell
pkg install cloudflared -y
```

### Create a Cloudflare Tunnel
- Goto your [Zero Trust Painel](https://one.dash.cloudflare.com/);
- Open the "Access" menu and goto "Tunnels";
- Create a Tunnel;
- Get the tunnel token from the snippets on the page;

### Connect to your tunnel
- On your tunnel overview page, select "Docker";
- Copy the token on the docker command;
- Execute the connector:
```shell
cloudflared tunnel --no-autoupdate run --token [copied-token]
```
- See if the tunnel status change to `HEALTHY`;

### Configure as a service
```shell
mkdir -p $PREFIX/var/service/cloudflare/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/cloudflare/log/run
touch $PREFIX/var/service/cloudflare/run
chmod +x $PREFIX/var/service/cloudflare/run
```
`$PREFIX/var/service/cloudflare/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
exec cloudflared tunnel --no-autoupdate run --token [copied-token] 2>&1
```

### Enable the service and start it
```shell
sv enable cloudflare
sv up cloudflare
```

### Expose a server
- On your tunnel overview page, click on the "Public Hostname" tab;
- Click on "Add a public hostname";
- Fill the subdomain and domain;
- Select the service connection type, probably "HTTP";
- Type the internal host to the server you want to expose; 
- Save it and wait a fill seconds for the dns to propagate;