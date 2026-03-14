### Termux GOWA Deploy
go-whatsapp-web-multidevice, a multi-user WhatsApp Web that expose a REST API.

### Requires
```bash
pkg install golang ffmpeg libwebp -y
```

### Build
```bash
cd ~/projects
git clone https://github.com/aldinokemal/go-whatsapp-web-multidevice gowa
cd  gowa/src
go build -o ../gowa
cd ..
```

### Enable as service
```shell
mkdir -p $PREFIX/var/service/gowa/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/gowa/log/run
touch $PREFIX/var/service/gowa/run
chmod +x $PREFIX/var/service/gowa/run
```
`$PREFIX/var/service/gowa/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/projects/gowa
exec 2>&1
APP_PORT=3000 \
APP_OS=GOWA \
APP_BASIC_AUTH=user:pass \
DB_URI=postgres://gowa:password@localhost:5432/gowa \
WHATSAPP_AUTO_DOWNLOAD_MEDIA=false \
WHATSAPP_WEBHOOK= \
WHATSAPP_WEBHOOK_EVENTS=message,message.reaction,message.revoked,message.edited,message.ack,message.deleted,group.joined,newsletter.joined \
WHATSAPP_PRESENCE_ON_CONNECT=available \
CHATWOOT_ENABLED=false \
cd $HOME/projects/gowa \
./gowa rest
```
Enable the service and start it
```shell
sv enable gowa && sv up gowa
```