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
![image](https://user-images.githubusercontent.com/49016081/144438380-a2abf0ac-a5d7-4ed2-9203-947399298b93.png)

7.  The static IP address of the LAN will need to be changed later.
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


# OpnSense preferences

## IDS/IPS Setup
1. Navigate to "Services > Intrusion Detection > Administration"
2. Tick "Enabled"
3. Tick "IPS mode" if Intrusion Prevention is desired
4. Tick "Promiscuous mode" if using VLANs and LAN monitoring is needed. This is important in order to capture data on the physical network interface.
5. Pattern matcher "Hyperscan" for better network performance. Select "**Aho-Corasick**" if the system fails to start up. "Hyperscan" is limited to certain NIC:s like Intel.
6. Click "Apply"
![image](https://user-images.githubusercontent.com/49016081/144437476-b69cb924-38e8-4ee0-afc9-a92a21f163f5.png)

### IF the selected IDS interface is WAN
- Click on advanced settings
- Add WAN ip to home networks
![image](https://user-images.githubusercontent.com/49016081/144437984-906e6273-08af-4767-b2b0-daa23ed37d01.png)

### Download rulesets
Next we need to download and enable rulesets and policies. The IDS is basically useless before them.
7. Click on the "Download" tab
8. Choose and enable your desired rulesets. Remember that more rulesets require more processing power and RAM.
9. Click on "Download & Update Rules"
![image](https://user-images.githubusercontent.com/49016081/144437536-a1c7c9af-21f5-4a16-a027-e69d1668392d.png)

### Creating a policy
At least one policy must be created for intrusion detection to work properly.
1. Navigate to "Services > Intrusion Detection > Policy"
2. Click on the "+" icon on the far right corner to set up a policy
![image](https://user-images.githubusercontent.com/49016081/144437565-aee98c1d-cd02-4581-ac7e-79d2c1fa8841.png)
4. Enable required rulesets you wish to enable for the current policy. Simple solution is to enable all.
![image](https://user-images.githubusercontent.com/49016081/144437585-7e80ac6c-b91c-4546-b16f-b5847978f37d.png)
5. Define desired actions for this policy. Select "alert" for only alert and not block. Select "Alert, Drop" if you wish to alert and block.
6. Click "Save" when finished

You will now see your created policy in the "Policies" page. You can manual adjustments to specific rules from the "Rule adjustments" tab. (**OpnSense recommends keeping manual adjustments to a minimum**)
![image](https://user-images.githubusercontent.com/49016081/144437616-452c9ff0-1783-4514-870b-67f06cb7e71b.png)

### Scheduling ruleset updates
Now we have enabled the IPS service, downloaded rulesets and enables policies. Now we need to schedule updates to keep the rulesets updated.

1. Navigate to"Services > Intrusion Detection > Administration" and open the "Schedule" tab
2. Enter the desired frequency for ruleset updates.
3. Tick "Enabled" and insert "2" Hours. In this example the rulesets will update every night at 02:00.
![image](https://user-images.githubusercontent.com/49016081/144437672-e4bd97fb-22e7-4aaa-aa5f-2a9168893e2e.png)

**Intrusion detection/prevention configuration is now complete**

### Viewing logs

After some time of running intrusion detection, you may navigate to "Services > Intrusion Detection > Administration" and check the “Alerts” tab to see the activity that is occurring on your network.



# ToDo
- ACL setup
- Monit settings
- Firewall Rules
