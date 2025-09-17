---
created: 2025-01-07T17:54
updated: 2025-09-17T14:47
tags:
  - proxmox
  - unifi
---
Once I had the need for Unifi Protect, I ended up replacing this setup with a Dream Machine Pro. [Ubiquiti has also published UnifiOS](https://blog.ui.com/article/introducing-unifi-os-server) that can be self-hosted if you want to use your own hardware.

Resources used:
- [debian-vm script from ProxmoxVE community repo](https://community-scripts.github.io/ProxmoxVE/scripts?id=debian-vm)
	- [useful debian-vm config guide from MickLesk](https://github.com/community-scripts/ProxmoxVE/discussions/836)
- [Unifi Installation Script by Glenn R.](https://community.ui.com/questions/UniFi-Installation-Scripts-or-UniFi-Easy-Update-Script-or-UniFi-Lets-Encrypt-or-UniFi-Easy-Encrypt-/ccbc7530-dd61-40a7-82ec-22b17f027776)

```bash
# Run debian install script in the proxmox node shell:
bash -c "$(wget -qLO - https://github.com/community-scripts/ProxmoxVE/raw/main/vm/debian-vm.sh)"

# Shut down the VM and resize the drive from 4GB -> 32GB in Proxmox UI:
# Unifi VM -> Hardware -> Hard Disk -> Disk Action -> Resize -> 
#   Size Increment: `28`

# Adjust partition with `parted` in the unifi VM shell:
apt update; apt install parted -y
parted
	(parted) resizepart
	Fix/Ignore? Fix
	Partition number? 1
	Yes/No? Yes
	End? [2146MB]? -0
	(parted) quit
reboot

# unifi install script needs the `ps` utility (not in debian image)
apt install procps -y

# create digitalocean credentials file to use for dns challenge
KEY_FILE=/root/digitalocean-api-key.ini
touch $KEY_FILE; chmod 700 $KEY_FILE

# `DO_AUTH_TOKEN` is for the unifi script
# `dns_digitalocean_token` value needs to be replaced with actual API token
 printf "DO_AUTH_TOKEN = 1\ndns_digitalocean_token = dop_v1_+++++" >> $KEY_FILE

# download unifi installation script from Glenn R.
cd /root
curl -sO https://get.glennr.nl/unifi/install/unifi-9.0.114.sh
curl -sO https://get.glennr.nl/unifi/extra/unifi-easy-encrypt.sh

# run unifi installation
bash unifi-9.0.114.sh

# run unifi LetsEncrypt setup, pointing to local DNS and local IP
bash unifi-easy-encrypt.sh --skip --fqdn unifi.hurm.io --server-ip 10.0.0.x --external-dns 10.0.0.x --dns-challenge --dns-provider digitalocean --dns-provider-credentials $KEY_FILE
```
