# Debian Server

## System

* OS: Debian Server
* Hostname: `debian-server`
* No desktop environment
* SSH server installed
* Filesystem: ext4

## LVM

Volume group: `debian-vg`

| LV   | Size | Mount   |
| ---- | ---: | ------- |
| root | 6 GB | `/`     |
| home | 4 GB | `/home` |
| var  | 8 GB | `/var`  |
| swap | 2 GB | swap    |

Free space was left in the VG for future expansion.

Useful commands:

```bash
sudo pvs
sudo vgs
sudo lvs
df -hT
/sbin/swapon --show
```

### Live expansion

Example:

```bash
sudo lvextend -L +2G /dev/debian-vg/root
sudo resize2fs /dev/debian-vg/root
df -hT /
```

The filesystem can be expanded while the server is running.

## Network

Configuration file:

```text
/etc/network/interfaces
```

```text
allow-hotplug enp0s3

iface enp0s3 inet static
    address 10.0.2.15
    netmask 255.255.255.0
    gateway 10.0.2.2
    dns-nameservers 10.0.2.3 8.8.8.8
```

Test:

```bash
ping -c 3 deb.debian.org
```

## Custom Service

Script:

```text
scripts/myservice.sh
```

```bash
#!/bin/bash

while true
do
    echo "myservice is running at $(date)"
    sleep 10
done
```

Systemd unit:

```text
systemd/myservice.service
```

```ini
[Unit]
Description=My Custom Service
After=network.target

[Service]
Type=simple
ExecStart=/usr/local/bin/myservice.sh
Restart=always
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Enable and start:

```bash
sudo systemctl daemon-reload
sudo systemctl enable myservice.service
sudo systemctl start myservice.service
```

Check:

```bash
systemctl status myservice.service
```

The service must show `enabled` and `active (running)`.

### Restart test

Find the PID:

```bash
systemctl show -p MainPID myservice.service
```

Kill it:

```bash
sudo kill <PID>
```

Check the status again. The PID should change and the service should remain `active (running)` because of `Restart=always`.

### Journal

```bash
sudo journalctl -u myservice.service
```

The script output should appear in the journal.

## Boot Analysis

```bash
systemd-analyze
systemd-analyze blame
sudo journalctl -b
```

Record the relevant boot-time observations here.

## Repository

```text
linux/
├── README.md
├── scripts/
│   └── myservice.sh
└── systemd/
    └── myservice.service
```
