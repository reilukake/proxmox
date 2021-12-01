# Networking
## Network diagram
![image](https://user-images.githubusercontent.com/49016081/144271980-5b42dc06-103e-45eb-8c23-433c6f1ce8d9.png)

## Network interfaces

### Physical
- enp3s0 is the INBOUND WAN interface from local modem and explicit for OpnSense
- enp2s0 is the OUTBOUND LAN interface connecting to local router and explicit for OpnSense
- wlp4s0 is the OUTBOUND LAN interface connecting to the local router, assigned to other VM:s inside Proxmox hypervisor

![image](https://user-images.githubusercontent.com/49016081/144244869-32ebfc48-588e-4abd-aa48-26570e036471.png)


## Setup inside Proxmox Network settings
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


# Proxmox setup
## Proxmox post-installation
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

### Disable enterprise repositories

```
$ echo "# deb https://enterprise.proxmox.com/debian/pve bullseye pve-enterprise" > /etc/apt/sources.list.d/pve-enterprise.list
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

## OpnSense install inside Proxmox

### Resource allocation and devices
- RAM: 8912MB
- CPU: 4CPU
- HDD: 120Gb
- vmbr1 - **Inbound traffic from Modem**
- vmbr2 - **Outbound traffic to Router**

![image](https://user-images.githubusercontent.com/49016081/144275772-7ad58146-d13f-42df-8c66-31585bd1efae.png)

### Installation process

1. Press any key when it prompts to start the manual interface assignment.
2. You will see both of your interfaces as their MAC addresses. You may need to refer to the MAC addresses in Proxmox to double check you are selecting the right interface (you can see the MAC addresses on the “Hardware” page of the VM). Enter “N” for configuring VLANs (or press “Enter” since “N” is the default value).
3. For the WAN interface name, enter “vtnet0” or “vtnet1” depending on the interface you want for the WAN. Check for corresponding MAC Address.
4. Enter the LAN interface name. It will be the other interface you did not use for the LAN. You will see the note about enabling the firewall/NAT mode once you enter the LAN interface. That is ok.
5. For the optional interface, just press “Enter”. There are no additional interfaces to set up.
6. Continue until reboot
7. ![image](https://user-images.githubusercontent.com/49016081/144276748-c7becf56-4b6c-4dc6-a343-7ceae26d8618.png) The static IP address of the LAN will need to be changed later.
8. Log in with the username "_installer_" and password "_opnsense_" to start the installation process.
9. Continue with desired options. Select GPT/UEFI mode.
10. Enter a new password for the "_root_" user
11. Exit and reboot
12. Wait for reboot and login using your new password and root username



## OpnSense post-install
- Open your browser and navigate to your OpnSense's IP address
- Log in using the credentials you provided in the install process. root/password

1. After successfully installing OPNsense and accessing the web interface, there are a few settings to consider modifying. On the “System > Settings > General” page, you may set a “Hostname”, “Domain”, and “Time zone”.
2. From "Interfaces > WAN" disable "Block private networks" **if** the VM is not public-facing. Default setting for production is left ticked.
