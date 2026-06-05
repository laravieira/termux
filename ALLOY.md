## Termux Alloy
### Build from source
To build Alloy for Termux you will need a powerful linux system, your phone will not handle the build. I choose Debian on WSL, since my machine has 32G of RAM.

Make sure you have on the linux machine the same Go version as your Termux, in my case is 1.26.3:
```shell
sudo apt remove golang golang-go -y

wget https://go.dev/dl/go1.26.3.linux-amd64.tar.gz
sudo tar -C /usr/local -xzf go1.26.3.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
source ~/.bashrc
go version
```

Clone the Alloy repository and go to the `collector` folder
```shell
sudo apt install golang
git clone https://github.com/grafana/alloy
cd alloy/collector
```
The alloy can be built in any version from 1.22, but the files riquire the version 1.26.4,
 so you have to replace the required version to be the one you have
```shell
find .. -name "go.mod" -exec sed -i 's/^go 1\.26\.4/go 1.26.3/' {} +
```
Now you can build Alloy, this will take a fill minutes
```shell
GOOS=android GOARCH=arm64 go build -o alloy .
```
After build, you can transfer the file to your phone
```shell
scp -P 8022 alloy [phone-user]@[phone-ip]:~/projects/alloy/alloy
```
You can than run alloy on your phone
```shell
cd ~/projects/alloy
chmod +x alloy
./alloy --version
```

### Enable as service
```shell
mkdir -p $PREFIX/var/service/alloy/log
ln -sf $PREFIX/share/termux-services/svlogger $PREFIX/var/service/alloy/log/run
touch $PREFIX/var/service/alloy/run
chmod +x $PREFIX/var/service/alloy/run
```
`$PREFIX/var/service/alloy/run`
```shell
#!/data/data/com.termux/files/usr/bin/sh
cd ~/projects/alloy
exec 2>&1
./alloy run config.alloy
```
Enable the service and start it
```shell
sv enable alloy && sv up alloy
```

### Config Sample
```json
// ============================================================
// REMOTES
// ============================================================

loki.write "lostworld" {
  endpoint {
    url = "http://192.168.2.101:3100/loki/api/v1/push"
    basic_auth {
      username = "username"
      password = "password"
    }
  }
  external_labels = {
    environment = "lostworld",
    job         = "manager",
    source      = "system",
  }
}

prometheus.remote_write "lostworld" {
  endpoint {
    url = "http://192.168.2.101:9090/api/v1/write"
    basic_auth {
      username = "username"
      password = "password"
    }
  }
  external_labels = {
    environment = "lostworld",
    job         = "manager",
    source      = "system",
  }
}

// ============================================================
// METRICS — disk usage + memory/swap
// ============================================================

prometheus.exporter.unix "system" {
  disable_collectors = [
    "timex",
    "bonding",
    "selinux",
    "btrfs",
    "edac",
    "fibrechannel",
    "infiniband",
    "ipvs",
    "mdadm",
    "nfs",
    "nfsd",
    "nvme",
    "powersupplyclass",
    "rapl",
    "tapestats",
    "zfs",
    "cpu",
    "schedstat",
    "diskstats",
    "udp_queues",
    "conntrack",
    "stat",
    "arp",
    "loadavg",
    "filefd",
    "sockstat",
    "softnet",
    "netstat",
    "hwmon",
    "vmstat",
    "pressure",
    "netdev",
    "processes",
  ]
}

prometheus.relabel "system" {
  forward_to = [prometheus.remote_write.lostworld.receiver]
  rule {
    action       = "replace"
    target_label = "environment"
    replacement  = "lostworld"
  }
  rule {
    action       = "replace"
    target_label = "job"
    replacement  = "manager"
  }
  rule {
    action       = "replace"
    target_label = "source"
    replacement  = "system"
  }
}

prometheus.scrape "system" {
  targets    = prometheus.exporter.unix.system.targets
  forward_to = [prometheus.relabel.system.receiver]
  job_name = "manager"

  scrape_interval = "15s"
}

// ============================================================
// LOGS — termux-services (all services)
// ============================================================

local.file_match "termux_services" {
  path_targets = [{
    __path__ = "/data/data/com.termux/files/usr/var/log/sv/**/current",
  }]
  sync_period = "5s"
}

loki.source.file "termux_services" {
  targets    = local.file_match.termux_services.targets
  forward_to = [loki.process.sys.receiver]
  tail_from_end = true
}

loki.process "sys" {
  stage.regex {
    expression = "/sv/(?P<service_name>[^/]+)/"
    source     = "filename"
  }

  stage.labels {
    values = {
      service_name = "service_name",
    }
  }

  stage.static_labels {
    values = {
      environment = "lostworld",
      job         = "manager",
      source      = "system",
    }
  }

  forward_to = [loki.write.lostworld.receiver]
}
```