---
author: Adam Hurm
date-created: 2023-05-10
tags:
  - guide
  - homelab
  - dns
  - synology
  - shareable
created: 2023-05-10T21:03
updated: 2025-07-21T18:41
---

## Building golang project for Synology NAS
*This guide is made for macOS. It's also pretty sloppy since I only needed to build one binary. Please update to a newer go version after building.*
```
MacBook:~$ brew install go@1.16    ## move ~/go to ~/go-backup if you have an existing installation
MacBook:~$ brew link go@1.16
MacBook:~$ brew install FiloSottile/musl-cross/musl-cross #[2]
MacBook:~$ go get github.com/mitchellh/go-homedir #[1]
MacBook:~$ mkdir -p ~/go/src/github/anaginsk #[1]
MacBook:~$ git clone https://github.com/anaganisk/digitalocean-dynamic-dns-ip.git ~/go/src/github/anaginsk #[1]
MacBook:~$ cd ~/go/src/github/anaginsk/digitalocean-dynamic-dns-ip/
MacBook:digitalocean-dynamic-dns-ip$ CC=x86_64-linux-musl-gcc \
  CXX=x86_64-linux-musl-g++ \
  GOARCH=amd64 \
  GOOS=linux \
  CGO_ENABLED=1 \
  go build -ldflags "-linkmode external -extldflags -static" #[2]
MacBook:digitalocean-dynamic-dns-ip$ cp digitalocean-dynamic-ip.sample.json .digitalocean-dynamic-ip.json
```
## Configuration file and DigitalOcean DNS
- [Create an AA entry](https://docs.digitalocean.com/products/networking/dns/how-to/manage-records/) in DigitalOcean. (and an optional AAAA entry if your droplet supports IPv6.)
- [Create an API Key](https://docs.digitalocean.com/reference/api/create-personal-access-token/) in DigitalOcean.
- Follow the [usage guide](https://github.com/anaganisk/digitalocean-dynamic-dns-ip#usage) to configure .digitalocean-dynamic-ip.json with your DigitalOcean API key and the domains you want to update.

## Installing on Synology NAS
- Log in to the Synology portal and add a Shared Folder to your NAS with restricted access control.
- Toggle Synology SSH (if you like to leave it disabled).
```
MacBook:digitalocean-dynamic-dns-ip$ scp digitalocean-dynamic-dns-ip .digitalocean-dynamic-ip.json user@SynNAS:/path/to/sharedfolder
MacBook:digitalocean-dynamic-dns-ip$ ssh user@SynNAS
SynNAS:/$ sudo -i
SynNAS:/# vim /etc/crontab
```

```
*/30  *  *   *   *   user  /path/to/digitalocean-dynamic-dns-ip /path/to/.digitalocean-dynamic-ip.json
```

```
SynNAS:/# systemctl restart crond 
SynNAS:/# systemctl restart synoscheduler
```

## Cleanup
```
MacBook:~$ rm digitalocean-dynamic-dns-ip/.digitalocean-dynamic-ip.json    ## no need to keep the API key on two machines
MacBook:~$ brew remove go@1.16    ## remove ~/go if you made a backup earlier
````
- Toggle Synology SSH (if you like to leave it disabled).

## Resources
- \[1\] [DigitalOcean Dynamic DNS script - credit: Anagani Sai Kiran](https://github.com/anaganisk/digitalocean-dynamic-dns-ip)
- \[2\] [Compiling golang project for Synology NAS - credit: Anthony Fox](https://anthonyfox.io/posts/compiling-go-for-synology-nas)