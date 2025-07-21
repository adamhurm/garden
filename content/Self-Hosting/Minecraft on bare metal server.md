---
created: 2024-08-30T16:37
updated: 2025-07-21T18:38
tags:
  - ubuntu
  - game_hosting
  - shareable
---

## Bare Metal Server
I got a 50% off deal for LiquidWeb, so I figured I would try out one of their unmanaged bare metal servers with the following specs:
> **Processor:** Intel Xeon E-2356G 6 Cores \
> **RAM:** 32GB DDR4 SDRAM \
> **HDD:** 2 x 960 GB SSD in Hardware Raid-1 \
> **OS:** Ubuntu 22.04 LTS 64-bit  

One of the projects I had in mind for this server was hosting a Minecraft game server that could support really heavy mods. I documented this pretty barebones minecraft hosting setup, which you could also reproduce on any other bare metal server.

## Initial system setup
After my server was provisioned, I got access to my starting credentials via [LiquidWeb management portal](https://manage.LiquidWeb.com/manage/account/#one_time_secret), then performed the following setup tasks:

### Set up user and basic security
```shell
# Log in to LiquidWeb server and create user account.
ssh root@lw.my.domain
passwd
adduser myuser
usermod -aG sudo myuser

# On personal machine, create ssh key with 1Password CLI and copy public key.
op item create --ssh-generate-key=ed25519 --title='lwsite myuser' --category 'SSH Key' | sed -n -e 's/^\ *public key:\ *//p'

# Back on LiquidWeb server, add the public key, configure ssh, and install security utilities.
ssh myuser@lw.my.domain
mkdir -p .ssh/
vim .ssh/authorized_keys
sudo -e /etc/ssh/sshd_config
> PermitRootLogin no
> PubkeyAuthentication yes
sudo apt update
sudo apt install fail2ban ufw -y
```
### Optional: Enable X11 Forwarding
Here's an example of what you might do to enable X11 forwarding on your LW host and connect via Windows machine:
```shell
# LW host
sudo -e /etc/ssh/sshd_config
> X11Forwarding yes
> X11DisplayOffset 10
> X11UseLocalhost no

# Local windows machine
winget install XcXsrv
```
### LW-only: Fix lwauth bug
Ran into a bug with LiquidWeb's package `lwauth` failing to reinstall/uninstall, and here's how I addressed that:
```shell
sudo dpkg --list | grep LiquidWeb
iHR lwauth 0.4.4 amd64          LiquidWeb management utility to aid logins.
ii  lwauth-configs 0.1-2 amd64  LiquidWeb management utility to aid logins.
ii  sonarperl 5.34.0~jammy amd64  LiquidWeb sonarperl

mkdir Downloads && cd Downloads
wget 'http://launchpadlibrarian.net/580608781/dpkg-repack_1.50_all.deb' # https://launchpad.net/ubuntu/jammy/+package/dpkg-repack
sudo dpkg -i dpkg-repack_1.50_all.deb

myuser@host:~/Downloads$ dpkg-repack lwauth
dpkg-repack: error: package lwauth is not fully installed: install reinstreq half-installed
dpkg-repack: warning: problems found processing lwauth, the package may be broken

myuser@host:~/Downloads$ dpkg-repack lwauth-configs
cp: cannot open '/usr/local/lp/etc/lwauth/lwauth-key-pub.pem' for reading: Permission denied
dpkg-repack: error: cp -pd /usr/local/lp/etc/lwauth/lwauth-key-pub.pem dpkg-repack.lwauth-configs.LNEaH7//usr/local/lp/etc/lwauth/lwauth-key-pub.pem subprocess returned exit status 1
dpkg-repack: warning: problems found processing lwauth-configs, the package may be broken

myuser@host:~$ dpkg-repack sonarperl
dpkg-deb: building package 'sonarperl' in './sonarperl_5.34.0~jammy_amd64.deb'.

myuser@host:~$ sudo rm -f /var/lib/dpkg/info/lwauth.postinst /var/lib/dpkg/info/lwauth.postrm
myuser@host:~$ sudo dpkg --purge --force-all lwauth
dpkg: warning: overriding problem because --force enabled:
dpkg: warning: package is in a very bad inconsistent state; you should
 reinstall it before attempting a removal
(Reading database ... 86117 files and directories currently installed.)
Removing lwauth (0.4.4) ...
```

---
## Hosting minecraft
The following approach can be repeated to create `game2`, `game3`, and so on. So you can maintain multiple servers in their own directories under `/opt/minecraft` without interference. Also, I used playit.gg to tunnel traffic, but there are tons of other tunneling solutions available.
```shell
# Set up openjdk
sudo add-apt-repository ppa:openjdk-r/ppa
sudo apt install openjdk-21-jre-headless

# Set up minecraft user
sudo adduser --system --shell /bin/bash --home /opt/minecraft --group minecraft
sudo mkdir /opt/minecraft/game1
wget https://piston-data.mojang.com/v1/objects/59353fb40c36d304f2035d51e7d6e6baa98dc05c/server.jar -O /opt/minecraft/game1/server.jar
sudo chown -R minecraft.minecraft /opt/minecraft

# Initialize minecraft, modify config
sudo su minecraft
cd /opt/minecraft/game1/
echo "eula=true" >> eula.txt
java -Xmx16G -Xms16G -jar server.jar nogui
vim server.properties

# Set up mcrcon
cd /opt/minecraft
git clone https://github.com/Tiiffi/mcrcon
cd mcrcon
make
sudo make install

# Download config, modify config
wget https://raw.githubusercontent.com/brianwarner/minecraft-server-hosting/main/minecraft%40.service -O /etc/systemd/system/minecraft@.service
sudo -e /etc/systemd/system/minecraft@.service
wget https://raw.githubusercontent.com/brianwarner/minecraft-server-hosting/main/mcrcon.conf -O /etc/mcrcon.conf
sudo -e /etc/mcrcon.conf

# Set up playit.gg
curl -SsL https://playit-cloud.github.io/ppa/key.gpg | gpg --dearmor | sudo tee /etc/apt/trusted.gpg.d/playit.gpg >/dev/null
echo "deb [signed-by=/etc/apt/trusted.gpg.d/playit.gpg] https://playit-cloud.github.io/ppa/data ./" | sudo tee /etc/apt/sources.list.d/playit-cloud.list
sudo apt update
sudo apt install playit

# Enable and configure playit
sudo systemctl start playit
sudo systemctl enable playit
playit setup
> https://playit.gg/account/tunnels/{tunnel_id}
> {custom_domain_name} | {ip_address}:{port}

# Check playit logs if needed
sudo tail -f /var/log/playit/playit.log

# Run minecraft as a service
sudo systemctl start minecraft@game1
sudo systemctl enable minecraft@game1


# OPTIONAL ---------------------------

# Firewall
sudo ufw allow 25565

# Port forwarding
echo 1 | sudo tee /proc/sys/net/ipv4/ip_forward
sudo nano /etc/sysctl.conf
> net.ipv4.ip_forward = 1
sudo sysctl -p
sudo sysctl --system
```


