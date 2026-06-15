## Termux Traefik
To run traefik on Termux you need to consider 3 points:
1. Traefik without root can't use ports 80 and 443
2. Go default DNS doesn't work on android, so you need to build traefik from source
3. Building Traefik requires Android NDK of your Android version

### Build from source
To build Traefik for Termux I recomend an external system. I choose Debian on WSL.

##### Download and install Android compilers
Get the commandlinetools zip on [this page](https://developer.android.com/studio?utm_source=chatgpt.com#command-tools)
```shell
sudo apt install openjdk-17-jdk unzip -y
mkdir -p $HOME/Android/Sdk
mkdir -p $HOME/Android/Sdk/cmdline-tools
cd /tmp

wget https://dl.google.com/android/repository/commandlinetools-linux-14742923_latest.zip
unzip commandlinetools-linux-*.zip
mv cmdline-tools $HOME/Android/Sdk/cmdline-tools/latest

export ANDROID_HOME=$HOME/Android/Sdk
export ANDROID_SDK_ROOT=$ANDROID_HOME
export PATH=$PATH:$ANDROID_HOME/cmdline-tools/latest/bin
export PATH=$PATH:$ANDROID_HOME/platform-tools

source ~/.bashrc
sdkmanager --version
yes | sdkmanager --licenses
sdkmanager "platforms;android-30" "build-tools;30.0.3" "platform-tools" "ndk;27.0.12077973"

export NDK=$HOME/Android/Sdk/ndk/27.0.12077973
export TOOLCHAIN=$NDK/toolchains/llvm/prebuilt/linux-x86_64/bin
export CC=$TOOLCHAIN/aarch64-linux-android30-clang
export CXX=$TOOLCHAIN/aarch64-linux-android30-clang++
```
###### The Android compiler version needs to match your Android device

Make sure you have on the linux machine the same Go version as your Termux, in my case is 1.26.3:
```shell
sudo apt remove golang golang-go -y

wget https://go.dev/dl/go1.26.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.26.3.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

Clone the Termux repository and build
```shell
git clone https://github.com/traefik/traefik.git
cd traefik
GOOS=android GOARCH=arm64 CGO_ENABLED=1 CC=$CC CXX=$CXX go build -o traefik ./cmd/traefik
```
After build, you can transfer the file to your phone
```shell
scp -P 8022 traefik [phone-ip]:~/projects/traefik/traefik
```
You can than run traefik on your phone
```shell
cd ~/projects/traefik
chmod +x traefik
mkdir dynamic
./traefik --configFile=traefik.yml
```

### Enable as service
```shell
mkdir -p $PREFIX/var/service/traefik/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/traefik/log/run
touch $PREFIX/var/service/traefik/run
chmod +x $PREFIX/var/service/traefik/run
```
`$PREFIX/var/service/traefik/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/projects/traefik
exec sudo env CLOUDFLARE_DNS_API_TOKEN=<cloudflare-token> ./traefik --configFile=traefik.yml 2>&1
```
Enable the service and start it
```shell
sv enable traefik && sv up traefik
```

### Config Sample
```yaml
entryPoints:
  http:
    address: ":80"
  https:
    address: ":443"
log:
  level: INFO
providers:
  file:
    watch: true
    directory: /data/data/com.termux/files/home/projects/traefik/dynamic/
http:
  middlewares:
    redirect-to-https:
      redirectScheme:
        scheme: https
        permanent: true
  routers:
    http-catchall:
      entryPoints:
        - web
      rule: "HostRegexp(`^.+$`)"
      middlewares:
        - redirect-to-https
      service: noop
  services:
    noop:
      loadBalancer:
        servers:
          - url: "http://127.0.0.1"
certificatesResolvers:
  letsencrypt:
    acme:
      email: <cloudflare-email>
      storage: /data/data/com.termux/files/home/projects/traefik/acme.json
      dnsChallenge:
        provider: cloudflare
        delayBeforeCheck: 0
```
