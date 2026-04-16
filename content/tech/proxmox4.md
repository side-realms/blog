---
title: "自宅サーバ4: ネットワーク設定"
date: 2023-10-09T17:57:25+09:00
draft: false
---

## 固定IP

- ```sudo vim /etc/systemd/network/20-wired.network```
  ```sh
  [Match]
  Name=enp3s0

  [Network]
  Address=192.168.0.169/24
  Gateway=192.168.0.1
  DNS=9.9.9.9
  DNS=1.1.1.1
  ```
- ```sudo ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf```
- ```sudo systemctl restart systemd-networkd```
- ```sudo systemctl restart systemd-resolved```

## nftable 設定

```sh
[pondering@Pondering ~]$ cat /etc/nftables.conf
#!/usr/bin/nft -f
# vim:set ts=2 sw=2 et:

# IPv4/IPv6 Simple & Safe firewall ruleset.
# More examples in /usr/share/nftables/ and /usr/share/doc/nftables/examples/.

flush ruleset

# destroy table inet filter

table inet filter {
  chain input {
    type filter hook input priority 0; policy drop;

    # ct state invalid drop comment "early drop of invalid connections"
    ct state established, related accept comment "allow tracked connections"
    iif lo accept comment "allow from loopback"

    meta l4proto { icmp, icmpv6 } accept comment "allow icmp"

    tcp dport ssh accept comment "allow sshd"
    tcp dport 5678 accept

    pkttype host limit rate 5/second counter reject with icmpx type admin-prohibited
    counter
  }
  chain forward {
    type filter hook forward priority 0; policy drop;
  }
  chain output {
    type filter hook output priority 0; policy accept;
  }
}
```

## fail2ban

```sh
[DEFAULT]
bantime = 24h
findtime = 10m
maxretry = 5
banaction = nftables-multiport

[sshd]
enabled = true
port = ssh
filter = sshd
logpath = /var/log/auth.log
backend = systemd
```

## サーバのステータス通知

```sh
#!/bin/bash
WEBHOOK_URL="xxx"

CPU=$(top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}')
MEM=$(free -m | awk 'NR==2{printf "%.1f%%  (%dMB / %dMB)", $3*100/$2, $3, $2}')
DISK=$(df -h / | awk 'NR==2{printf "%s  (%s / %s)", $5, $3, $2}')
TEMP=$(cat /sys/class/thermal/thermal_zone0/temp 2>/dev/null | awk '{printf "%.1f°C", $1/1000}')
UPTIME=$(uptime -p)
CONTAINERS=$(podman ps --format "{{.Names}}" 2>/dev/null | wc -l)

DISK_PCT=$(df / | awk 'NR==2{print $5}' | tr -d '%')
if [ "$DISK_PCT" -ge 80 ]; then
  COLOR=15158332
elif [ "$DISK_PCT" -ge 60 ]; then
  COLOR=16750848
else
  COLOR=3066993
fi

curl -s -H "Content-Type: application/json" -X POST "$WEBHOOK_URL" -d "{
  \"embeds\": [{
    \"title\": \"🍩 Pondering Status 🍩\",
    \"color\": $COLOR,
    \"fields\": [
      {\"name\": \"CPU\", \"value\": \"${CPU}%\", \"inline\": true},
      {\"name\": \"Memory\", \"value\": \"$MEM\", \"inline\": true},
      {\"name\": \"Disk\", \"value\": \"$DISK\", \"inline\": true},
      {\"name\": \"Temp\", \"value\": \"$TEMP\", \"inline\": true},
      {\"name\": \"Uptime\", \"value\": \"$UPTIME\", \"inline\": true},
      {\"name\": \"Containers\", \"value\": \"$CONTAINERS running\", \"inline\": true}
    ]
  }]
}"
```

- ```~/scripts/monitor.sh```
- ```sudo vim /etc/systemd/system/monitor.service```
```sh
[Unit]
Description=Server monitor Discord notification

[Service]
Type=oneshot
User=pondering
ExecStart=/home/pondering/scripts/monitor.sh
```
- ```sudo vim /etc/systemd/system/monitor.timer```
```sh
[Unit]
Description=Run server monitor every 6 hours

[Timer]
OnCalendar=*-*-* 6:00:00
Persistent=true

[Install]
WantedBy=timers.target
```
- ```sudo systemctl enable monitor.timer```
- ```sudo systemctl start monitor.timer```
