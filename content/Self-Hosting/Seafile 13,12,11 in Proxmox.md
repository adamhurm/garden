---
created: 2025-07-04T14:34
updated: 2025-07-21T18:39
tags:
  - homelab
  - seafile
---


Context: These notes are from when I was attempting to use Seafile for external file sharing. I'm not currently using it because I found that support for reverse proxies was broken in recent releases, or at the very least I couldn't resolve issues with URL re-writes.

In this attempt, I created a Debian 12 VM in Proxmox and installed docker. I started with Seafile 13 (with a few version number tweaks because it’s in testing state), then tried Seafile 12, and finally had to revert to Seafile 11 to fix the mixed content issues.  See [[#Active Troubleshooting]] section for more details.

# Setup debian VM

Create new VM in Proxmox UI and use a Debian 12 ISO. Go through installation wizard to create a a non-root user account.

## Add some additional security

```bash
> ssh root@debianVM

# Add your user account to the `sudo` group.
usermod -aG sudo adam


> ssh adam@debianVM

# Install ufw for firewall, fail2ban for intrusion prevention
sudo apt install ufw fail2ban
sudo ufw enable
sudo systemctl start ufw
sudo systemctl enable ufw

# Disable root login
sudo passwd -l root
sudo sed -i '/^root:/s/:[^:]*$/:\/sbin\/nologin/' /etc/passwd
```

## Use larger storage device for `seafile-data`
I want to have the largest possible storage device for `seafile-data`, so I added a 4TB HDD via Proxmox UI. Then created a link to `/opt/seafile-data` where the docker container expects to read it.
```bash
# Find the appropriate drive. In this example, we'll use sdb.
sudo fdisk -l

# Create partition table, filesystem, and mount points
sudo mkfs.ext4 /dev/sdb1
sudo mount /dev/sdb1 /mnt/big-storage
sudo mkdir -p /mnt/big-storage

# Get the partition UUID and set up auto-mount in /etc/fstab
sudo blkid /dev/sdb1
echo "UUID=xxxxxx  /mnt/big-storage  ext4  defaults  0  2" | sudo tee -a /etc/fstab

# Create symbolic link to seafile-data
sudo ln -s /mnt/big-storage/seafile-data /opt/seafile-data
```

## Install docker via apt
```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/debian/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
# Add the repository to Apt sources:
echo   "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/debian \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" |   sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

# Seafile 13 / 12

## 13.0 docker deployment following [manual](https://manual.seafile.com/13.0/setup/setup_ce_by_docker/)
```bash
mkdir -p /opt/seafile/13
cd /opt/seafile/13
wget -O .env https://manual.seafile.com/13.0/repo/docker/ce/env
wget https://manual.seafile.com/13.0/repo/docker/ce/seafile-server.yml
wget https://manual.seafile.com/13.0/repo/docker/seadoc.yml
wget https://manual.seafile.com/13.0/repo/docker/caddy.yml
```
## 12.0 docker deployment following [manual](https://manual.seafile.com/12.0/setup/setup_ce_by_docker/)
```bash
mkdir -p /opt/seafile/12
cd /opt/seafile/12
wget -O .env https://manual.seafile.com/12.0/repo/docker/ce/env
wget https://manual.seafile.com/12.0/repo/docker/seadoc.yml
wget https://manual.seafile.com/12.0/repo/docker/ce/seafile-server.yml
wget https://manual.seafile.com/12.0/repo/docker/caddy.yml
```
https://manual.seafile.com/12.0/setup/caddy/

Add your required setup configuration in `.env` and use `docker compose up -d`

## Active Troubleshooting
I can't use any file upload or seadoc functionalities in the Web UI. My browser console will be full of warnings about mixed content because the application serves errors about URLs like `http://sf.my.domain/seafhttp` . The rewrite rules seem to have changed at some point, still determining how to address this without forking the seafile application itself.

[This update](https://manual.seafile.com/13.0/changelog/server-changelog/#1204-beta-2024-11-21) removed `SERVICE_URL` + `FILE_SERVER_ROOT`  and  [`FORCE_HTTPS_IN_CONF`](https://github.com/haiwen/seafile-docker/blob/8dfce0892665c301a419d0e5b6f8b96a4669a07e/scripts/scripts_11.0/bootstrap.py#L115-L120) was [removed](https://github.com/haiwen/seafile-docker/blob/8dfce0892665c301a419d0e5b6f8b96a4669a07e/scripts/scripts_12.0/bootstrap.py#L115C1-L120C17) from `get_proto()`.  Seafile 11 is where it last worked, so a dirty fix is to revert back to Seafile 11 for now.

### Custom image
- [seafile-docker on GitHub](https://github.com/haiwen/seafile-docker)
- [seafile-server on GitHub](https://github.com/haiwen/seafile-server)
- [seafile-mc on DockerHub](https://hub.docker.com/r/seafileltd/seafile-mc)

patch [11.0 `get_proto()`](https://github.com/haiwen/seafile-docker/blob/master/scripts/scripts_11.0/bootstrap.py#L115-L120) into seafile 13.0:
```bash
# Download seafile-docker repo
git clone https://github.com/haiwen/seafile-docker
cd seafile-docker

# Prep env for build
## patch L115-120 with scripts/scripts_11.0/bootstrap.py#L115-L120
vim scripts/scripts_13.0/bootstrap.py

# Build seafile package 13.0.6 via docker
## patch L37 with `libhiredis-dev \`
vim build/seafile_12.0/seafile-build.sh
cd build/seafile_12.0/
docker run --rm -it --volume=/$(pwd):/seafile ubuntu:24.04 /seafile/seafile-build.sh 13.0.6

# Build seafile container 13.0.6
## patch L2 with `SEAFILE_VERSION=13.0.6`
## patch L48 with `COPY scripts/scripts_13.0 /scripts`
## patch L75 with `COPY build/seafile_12.0/seafile-server-${SEAFILE_VERSION}`
cd ../../
cp image/seafile_13.0/Dockerfile .
vim Dockerfile

# Buld image and push to dockerhub
docker login
docker build -t adamhurm/seafile:13.0.6 .
```

Initially the build script left the `build/seafile_12.0/src/seafile-server` directory empty, and by manually populating that directory the `setup-seafile-mysql.sh` error below will be resolved. The fix is to replace `build/seafile_12.0/seafile-server-13.0.6` with the appropriate git repo and then hide/remove `.git` .
```bash
## ERROR: seafile docker container can't find `setup-seafile-mysql.sh`
[2025-07-05 21:45:19] Now running setup-seafile-mysql.py in auto mode.
/bin/sh: 1: /opt/seafile/seafile-server-13.0.6/setup-seafile-mysql.sh: not found
Traceback (most recent call last):
   File "/scripts/start.py", line 94, in <module>
     main()
   File "/scripts/start.py", line 59, in main
     init_seafile_server()
   File "/scripts/bootstrap.py", line 156, in init_seafile_server
     call('{} auto -n seafile'.format(setup_script), env=env)
   File "/scripts/utils.py", line 71, in call
     return subprocess.check_call(*a, **kw)
            ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
   File "/usr/lib/python3.12/subprocess.py", line 413, in check_call
     raise CalledProcessError(retcode, cmd)
 subprocess.CalledProcessError: Command '/opt/seafile/seafile-server-13.0.6/setup-seafile-mysql.sh auto -n seafile' returned non-zero exit status 127.

---

## ERROR: go build fails for seafile-server because `.git` directory is present
error obtaining VCS status: exit status 128
        Use -buildvcs=false to disable VCS stamping.
[ERROR] error when running command:
        cd fileserver && go build && cd -
```

Even after all the tweaks above, I am still getting mixed content warnings and attempts to reach out to `http://sf.my.domain/seafhttp/*` so I will have to change more than just `get_proto()`.

Right now this is still tbd, and I'm not sure if I'll come back to it.

# Seafile 11
## 11.0 docker deployment following [manual](https://manual.seafile.com/11.0/docker/deploy_seafile_with_docker/)
```bash
mkdir -p /opt/seafile/11
cd /opt/seafile/11
wget -O "docker-compose.yml" "https://manual.seafile.com/11.0/docker/docker-compose/ce/11.0/docker-compose.yml"
```

Add all the environmental variables to `docker-compose.yml` manually because `.env` isn't supported yet.

## Force HTTPS URLs and let Pangolin handle the certs
#### Fix mixed content warnings
```yaml
# /opt/seafile/docker-compose.yml
  seafile:
    environment:
      - FORCE_HTTPS_IN_CONF=true
```
### Fix [CSRF error](https://forum.seafile.com/t/forbidden-403-csrf-verification-failed-on-docker-11-0-1/18711/10)
```python
# /opt/seafile-data/seafile/conf/seahub_settings.py
SERVICE_URL = "https://sf.my.domain"              # edit SERVICE_URL
CSRF_TRUSTED_ORIGINS = ['https://sf.my.domain']   # add CSRF_TRUSTED_ORIGINS
```



# Setting up pangolin redirect
### Add a newt container to the docker network
```yaml
# /opt/seafile/docker-deployment.yml (v11)
# /opt/seafile/caddy.yaml (v12,13)
  newt:
    image: fosrl/newt
    container_name: newt
    restart: unless-stopped
    environment:
      - DOCKER_SOCKET=/var/run/docker.sock
      - PANGOLIN_ENDPOINT=https://pangolin.my.domain
      - NEWT_ID=xxxxxxxxxxxxxxx
      - NEWT_SECRET=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
    networks:
      - seafile-net
```
### Set redirect rule in pangolin web UI
#### using docker container name (recommended, dynamic)
In pangolin **Resources**, under **Proxy** tab:
	**Method:** `http`
	**IP / Hostname:** `seafile`
	**Port:** `80`
#### using docker container name (recommended, easy)
If you used the newt container addon above and enabled Docker Socket discovery in your pangolin **Site**, you can go to pangolin **Resources** under **Proxy** tab:

	**Method:** `http`

	**IP / Hostname:** Click 'View Docker Containers' and select your caddy container

#### docker inspect (manual example)
Get IP of caddy container and add to Pangolin rule: \
`docker inspect -f '{{range.NetworkSettings.Networks}}{{.IPAddress}}{{end}}' seafile-caddy`

In pangolin **Resources** under **Proxy** tab: \
	**Method:** `http` \
	**IP / Hostname:** 172.18.0.x \
	**Port:** `80` \

