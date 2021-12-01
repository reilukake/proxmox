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

![image](https://user-images.githubusercontent.com/49016081/144244869-32ebfc48-588e-4abd-aa48-26570e036471.png)


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

## Modem and router settings

### Modem settings
![image](https://user-images.githubusercontent.com/49016081/144244766-25b94597-28a9-4372-8612-f3f4b64775d9.png)
- Enable ethernet WAN
- Bridged
- Transparent Bridging

### Router settings
![image](https://user-images.githubusercontent.com/49016081/144245320-7ce780d9-0679-4aff-81db-eb6d620e630d.png)
- Internet connection type: Static IP
- IP Address: 192.168.2.1 - **Router's IP Address**
- Subnet mask: 255.255.255.0
- Default Gateway: 192.168.2.2 - **Proxmox OpnSense IP Address**
- Primary DNS: 8.8.8.8
