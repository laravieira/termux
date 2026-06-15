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
scp -P 8022 alloy [phone-ip]:~/projects/alloy/alloy
```
You can than run alloy on your phone
```shell
cd ~/projects/alloy
chmod +x alloy
./alloy --version
```
###### If your android kernel is too old, you may require sudo to run alloy at all

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
exec ./alloy run config.alloy 2>&1
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

### Disk usage without root
```
pkg install cronie -y
rm $PREFIX/var/service/crond/down
sv enable crond
sv up crond
mkdir ~/projects/alloy/metrics
touch ~/projects/alloy/disk.sh
chmod +x ~/projects/alloy/disk.sh
```
-´´´p0
`~/projects/alloy/disk.sh`
```shell
#!/data/data/com.termux/files/usr/bin/bash

OUTFILE="$HOME/projects/alloy/metrics/disk_metrics.prom"
mkdir -p "$(dirname "$OUTFILE")"
TMP="$OUTFILE.tmp"

echo "# HELP node_filesystem_size_bytes Filesystem size in bytes" > "$TMP"
echo "# TYPE node_filesystem_size_bytes gauge" >> "$TMP"
echo "# HELP node_filesystem_avail_bytes Filesystem space available in bytes" >> "$TMP"
echo "# TYPE node_filesystem_avail_bytes gauge" >> "$TMP"
echo "# HELP node_filesystem_used_bytes Filesystem space used in bytes" >> "$TMP"
echo "# TYPE node_filesystem_used_bytes gauge" >> "$TMP"

# df -P gives POSIX portable output: Filesystem, 512B-blocks, Used, Available, Use%, Mounted on
df -P | tail -n +2 | while read -r fs blocks used avail pct mount; do
    [ "$blocks" -eq 0 ] && continue
    case "$mount" in
        /apex/*|/proc|/sys|/dev|/acct|/mnt/vendor/*|/storage/emulated) continue ;;
    esac
    case "$fs" in
        tmpfs|none|overlay|rootfs) continue ;;
    esac

    size_bytes=$(( blocks * 512 ))
    used_bytes=$(( used * 512 ))
    avail_bytes=$(( avail * 512 ))
    labels="device=\"$fs\",mountpoint=\"$mount\",fstype=\"unknown\""
    echo "node_filesystem_size_bytes{$labels} $size_bytes" >> "$TMP"
    echo "node_filesystem_avail_bytes{$labels} $avail_bytes" >> "$TMP"
    echo "node_filesystem_used_bytes{$labels} $used_bytes" >> "$TMP"
done

mv "$TMP" "$OUTFILE"
```

Add to crontab -e
```shell
* * * * * $HOME/projects/alloy/disk.sh
```

Add to `config.alloy`
```json
prometheus.exporter.unix "disk" {
  set_collectors = ["textfile"]
  textfile {
    directory = "/data/data/com.termux/files/home/projects/alloy/metrics"
  }
}

prometheus.scrape "disk_scrape" {
  targets         = prometheus.exporter.unix.disk.targets
  forward_to      = [prometheus.relabel.system.receiver]
  job_name        = "manager"

  scrape_interval = "30s"
}
```