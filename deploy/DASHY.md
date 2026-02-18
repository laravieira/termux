# Termux Dashy Deploy
How to deploy the Dashy dashboard on the environment set by following this repository guides.

### Download dashy
```shell
cd ~/projects
git clone https://github.com/Lissy93/dashy
```

### Update the config file
`~/projects/dahsy/user-data/conf.yml`
```yaml
pageInfo:
  title: Nexus Home Server
  description: Welcome to your new dashboard!
  navLinks:
    - title: Termux
      path: https://github.com/laravieira/termux
    - title: Documentation
      path: https://dashy.to/docs
    - title: Cloudflare
      path: https://dash.cloudflare.com
  footerText: Termux Server with 💜 by Lara.
appConfig:
  language: en
  statusCheck: true
  statusCheckInterval: 0
  theme: adventure
  disableConfigurationForNonAdmin: true
  showSplashScreen: false
  layout: auto
  iconSize: medium
sections:
  - name: Servers
    icon: fas fa-rocket
    items:
      - title: STA
        description: Scandiweb Test Assignment
        icon: https://i.ibb.co/qWWpD0v/astro-dab-128.png
        url: https://sta.laravieira.me
        target: newtab
      - title: MySQL
        description: The Server MySQL
        url: tcp://127.0.0.1:3306
        icon: favicon
        statusCheckAllowInsecure: true
      - title: Athenas Front
        description: Athenas Front
        provider: Cloudflare
        icon: far fa-book
        url: https://dashy.to/docs
    displayData:
      sortBy: default
      rows: 1
      cols: 4
      collapsed: false
      hideForGuests: false
```
If your config file references local icons, paste the into:
`~/projects/dahsy/user-data/item-icons`

### Build the project
```shell
yarn install --ignore-engines
NODE_OPTIONS=--openssl-legacy-provider yarn --ignore-engines build
```

### Test it
```shell
PORT=4001 yarn --ignore-engines start
```

### Create a service for dashy
```shell
mkdir -p $PREFIX/var/service/dashy/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/dashy/log/run
touch $PREFIX/var/service/dashy/run
chmod +x $PREFIX/var/service/dashy/run
```
`$PREFIX/var/service/dashy/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/projects/dashy
exec 2>&1
PORT=4001 \
yarn --ignore-engines start
```

### Enable the service and start it
```shell
sv enable dashy
sv up dashy
```

### Configure a public hostname on the Cloudflare Tunnel
###### See [Cloudflare Tutorial](/CLOUDFLARED.md).
- On your tunnel overview page, click on the "Public Hostname" tab;
- Click on "Add a public hostname";
- Fill the subdomain and domain;
- Select the service connection type, probably "HTTP";
- Type the internal host to the server you want to expose; 
- Save it and wait a fill seconds for the dns to propagate;