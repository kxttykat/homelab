# proxmox post-install script
```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pve-install.sh)"
```
### proxmox backup server script
```
bash -c "$(curl -fsSL https://raw.githubusercontent.com/community-scripts/ProxmoxVE/main/tools/pve/post-pbs-install.sh)"
```

# cloud images
```
wget https://cdimage.debian.org/images/cloud/trixie/latest/debian-13-generic-amd64.qcow2

qm set 1000 --serial0 socketr --vga serial0

qemu-img resize debian-13-generic-amd64.qcow2 32G

virt-customize -a debian-13-generic-amd64.qcow2 --install qemu-guest-agent,docker.io,docker-compose,git,wget,curl,rsync,htop,nano,vim,tmux,fastfetch,fail2ban,ufw,zip,unzip,tar

virt-customize -a debian-13-generic-amd64.qcow2 --run-command "sudo usermod -aG docker $USER"

virt-customize -a debian-13-generic-amd64.qcow2 --run-command "truncate -s 0 /etc/machine-id"

qm importdisk 1000 debian-13-generic-amd64.qcow2 ocean
```
# fan speed
#### because I run a supermicro server sometimes the fan speed gets really out of hand and so I have to manually adjust it. luckily I've automated this.
be sure you're in the /root directory of your proxmox
`mkdir scripts`
`cd scripts`
`touch fan-speed.sh;nano fan-speed.sh`
```bash
#!/usr/bin/bash

ipmitool -H 192.168.1.3 -U ADMIN -P ADMIN raw 0x30 0x70 0x66 0x01 0x00 0x16
ipmitool -H 192.168.1.3 -U ADMIN -P ADMIN raw 0x30 0x70 0x66 0x01 0x01 0x16
```
`chmod +x fan-speed.sh`

`cd /etc/systemd/system`
`touch fan-speed.service;nano fan-speed.service`
```systemd
[Service]
Type=oneshot
RemainAfterExit=yes
ExecStartPre=/bin/sleep 5
ExecStart=/root/scripts/fan_speed.sh

[Install]
WantedBy=multi-user.target
```
`systemctl enable --now fan-speed.service`
`systemctl status fan-speed.service`
