# Proxmox installation

- Once installed
- navigate to https://IP:8006

## Post installation

### Remove subscription popup
``` 
# SSH to your server
$ nano /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js
```
- Search for
Ext.Msg.show({
  title: gettext('No valid subscription'),
- Replace 'Ext.Msg.show' with 'void'

```
$ systemctl restart pveproxy.service
```

### Setup community repo

```
$ nano /etc/apt/sources.list
```
Add the following
```
deb http://ftp.debian.org/debian bullseye main contrib
deb http://ftp.debian.org/debian bullseye-updates main contrib

# PVE pve-no-subscription repository provided by proxmox.com,
# NOT recommended for production use
deb http://download.proxmox.com/debian/pve bullseye pve-no-subscription

# security updates
deb http://security.debian.org/debian-security bullseye-security main contrib
```

### Setup dark theme for proxmox
Veilbyte Discord Dark theme (https://github.com/Weilbyte/PVEDiscordDark)

Oneliner install
``` 
$ bash <(curl -s https://raw.githubusercontent.com/Weilbyte/PVEDiscordDark/master/PVEDiscordDark.sh ) install
```

### Add desired ZFS storages etc from the admin panel or using zpool

Show disks
```
$ lsblk
```

Create mirrored zpool from desired disks
```
zpool create POOLNAME mirror /dev/sda /dev/sdb
```

# Proxmox and OpnSense

## Network interfaces

### Physical
- enp3s0 is the INBOUND WAN interface from local modem and explicit for OpnSense
- enp2s0 is the OUTBOUND LAN interface connecting to local router and explicit for OpnSense
- wlp4s0 is the OUTBOUND LAN interface connecting to the local router, assigned to other VM:s inside Proxmox hypervisor

### Setup inside Proxmox Network settings
Datacenter->prox->System->Network
- vmbr0
  - Linux Bridge
  - Ports/Slaves: enp3s0
  - CIDR: 192.168.0.247/24 - **Proxmox host IP**
  - Gateway: 192.168.0.254 - **IP of local modem**
- vmbr1
  - Linux Bridge
  - Ports/Slaves: enp2s0
- vmbr2
  - Linux Bridge
  - Ports/Slaves: wlp4s0 
