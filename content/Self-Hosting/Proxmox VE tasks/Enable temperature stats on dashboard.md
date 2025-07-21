---
created: 2025-07-17T13:23
updated: 2025-07-21T18:33
tags:
  - shareable
  - proxmox
---
The whole process is described by creator Meliox in [PVE Mods: node-sensor-readings-view](https://github.com/Meliox/PVE-mods?tab=readme-ov-file#node-sensor-readings-view). Here are the steps to perform in a root shell on your proxmox server:

```shell
# install lm-sensors and enable appropriate module(s)
apt install lm-sensors -y
sensors-detect                   # follow output guidance
echo "coretemp" >> /etc/modules  # example for intel CPUs

# run install script
mkdir /opt/pve-mod-gui-sensors
cd /opt/pve-mod-gui-sensors
wget https://raw.githubusercontent.com/Meliox/PVE-mods/main/pve-mod-gui-sensors.sh
bash pve-mod-gui-sensors.sh install
```