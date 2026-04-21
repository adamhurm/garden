---
author: Adam Hurm
created: 2021-02-20T00:00
tags:
  - gaming
  - ubuntu
  - game_hosting
---

This is a brief guide on how to:
- Set up a Valheim Dedicated Server on Ubuntu
- Migrate existing Valheim worlds to your new server
- Troubleshoot server connection issues

*This guide was created while using Ubuntu 20.04 (LTS). Your experience may vary depending on Linux distro.*


# Setting up Valheim Dedicated Server

Instead of going down the route of manually setting up SteamCMD, LinuxGSM can help us tremendously. [Follow the instructions for installing *vhserver* here](https://linuxgsm.com/lgsm/vhserver/).

### Installation steps

1. Install the vhserver dependencies. \
	   ```bash sudo dpkg --add-architecture i386; sudo apt update; sudo apt install curl wget file tar bzip2 gzip unzip bsdmainutils python util-linux ca-certificates binutils bc jq tmux netcat lib32gcc1 lib32stdc++6 steamcmd```

2. Create a new user to run the server on. \
   `adduser vhserver`

3. (Optional) Remove the password for vhserver user. Then we can use `su` to access the account without worrying about password strength. \
   `passwd -d vhserver`

4. Start a new session as vhserver. \
   `su - vhserver`

5. Download linuxgsm.sh. \
   `wget -O linuxgsm.sh https://linuxgsm.sh && chmod +x linuxgsm.sh && bash linuxgsm.sh vhserver`

6. Run the installer and follow the onscreen instructions. \
   `./vhserver install`

7. If you saw any missing dependencies, the installer will provide a command to fetch them. Run it as root/sudoer.

Once you've completed the setup, test your server (as vhserver user): \
`./vhserver start`

### Network configuration

Configure your server firewall:
```bash
ufw allow 2456:2568/tcp
ufw allow 2456:2568/udp
ufw enable
```

To connect to your server, use the correct port for your connection method:
- **Port 2456** — in-game server browser ("Join IP" option)
- **Port 2457** — Steam Server Browser (Steam → View → Servers)


# Migrating existing Valheim worlds to your new server

First, identify where your world save is currently located. If you have Valheim installed on Windows, your save files live here: 
`C:\Users\%USERNAME%\AppData\LocalLow\IronGate\Valheim\worlds`

The Linux world save location is different **and** it will not be created until vhserver is run for the first time. The save files live here: 
`/home/vhserver/.config/unity3d/IronGate/Valheim/worlds`

Zip up the files and move them between locations. You're looking for: \
`%worldname%.db  %worldname%.db.old  %worldname%.fwl  %worldname%.fwl.old`

To make sure vhserver uses your migrated world, append the following to the common.cfg file: \
`vim /home/vhserver/lgsm/config-lgsm/vhserver/common.cfg`

```bash
servername="%exampleServer%"
serverpassword="%examplePassword%"
gameworld="%worldname%"
```

Next time you run `./vhserver start`, you should be good to go!


# Not enough RAM?

If RAM is becoming a bottleneck on your VM, add swap space as a buffer.

First check if you already have swap enabled:
`swapon --show` (prints swap list if enabled; empty output means none)

If you don't have existing swap space, create a swap file (adjust size to fit your needs):
```bash
fallocate -l 2G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
```

(Optional) Make this persist across reboots by adding to `/etc/fstab`:
```
/swapfile swap swap defaults 0 0
```


# DigitalOcean fix

Having trouble connecting a Valheim client to the server? Checked that you can access your server remotely and firewall settings aren't getting in the way?

If so, you probably need to edit your droplet's Netplan configuration.

1. Open the Netplan config file as root:
   `vim /etc/netplan/50-cloud-init.yaml`

2. Remove the private IP address under eth0 (the `10.x.x.x` entry).

3. Apply the config:
   `netplan apply`

4. Reboot your server.

credit: [Somuchtogrok on Reddit](https://old.reddit.com/r/valheim/comments/ll927v/valheim_dedicated_liknux_server_disconnect_when/gnwm89s/)
