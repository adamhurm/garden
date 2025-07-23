---
created: 2025-06-17T14:25
updated: 2025-07-21T18:01
tags:
  - pangolin
  - authentik
  - homeassistant
---

I’m trying out Pangolin and I’ll be using RackNerd for hosting (recommended by and partnered with Pangolin dev team). They charge \$10.96 for a year of 1 vCPU/1GB RAM VPS, vastly undercutting the cheapest $6/mo VPS from DigitalOcean.

I am using Cloudflare for`my.domain`, so I will visit the Cloudflare dashboard and configure the DNS records to point to the RackNerd VPS:
```
A      @            x.x.x.x
CNAME  *.my.domain  my.domain
```

SSH into the RackNerd VPS to get things set up:
```shell
ssh root@my.domain -o PubkeyAuthentication=false

root@racknerd-9703dbc:~# apt update && apt upgrade -y
root@racknerd-9703dbc:~# passwd
root@racknerd-9703dbc:~# adduser adam
root@racknerd-9703dbc:~# usermod -aG sudo adam
root@racknerd-9703dbc:~# su - adam
adam@racknerd-9703dbc:~$ sudo apt install fail2ban -y
adam@racknerd-9703dbc:~$ wget -O installer "https://github.com/fosrl/pangolin/releases/download/1.5.1/installer_linux_$(uname -m | sed 's/x86_64/amd64/;s/aarch64/arm64/')" && chmod +x ./installer
adam@racknerd-9703dbc:~$ sudo ./installer
```
Follow the installer prompts and then visit https://pangolin.my.domain in a browser.


## Using pangolin for ...
### Authentik

Pangolin now offers [Identity Provider integration](https://docs.fossorial.io/Pangolin/Identity%20Providers/configuring-identity-providers).

In order to access my homelab IdP, I set up a new unprotected newt tunnel so that `authentik.my.domain` redirects to `authentik.my.internal`

This was done by adding a sidecar container:
```yaml
# k8s/authentik/values.yaml
  extraContainers:
    - name: newt
      image: ghcr.io/pangolin-community/newt:latest
      env:
        - name: PANGOLIN_ENDPOINT
          value: "https://pangolin.my.domain"
      envFrom:
        - secretRef:
            name: newt-creds
```
and using 1Password operator to fetch `NEWT_ID` and `NEWT_SECRET` :
```yaml
# k8s/1password-connect/OnePasswordItems/authentik.yaml
apiVersion: onepassword.com/v1
kind: OnePasswordItem
metadata:
  name: newt-creds
  namespace: authentik
spec:
  itemPath: "vaults/VAULT_ID/items/ITEM_ID"
```

Then in Pangolin Admin Settings, create Identity Provider with the recommended settings from [Authentik's Pangolin integration docs](https://docs.goauthentik.io/integrations/services/pangolin/), but disable auto-provisioning, add `groups` to **scope**, and under Token Configuration change **Identifier Path** from `sub` to `email`.

Manually provision user access to my.domain Organization by adding the appropriate username and email.

### HomeAssistant
https://ha.my.domain

Create HomeAssistant **Site** in Pangolin, and configured HomeAssistant **Resource** to proxy requests for `ha.my.domain` to `https://127.0.0.1:8123`


Install **newt** HomeAssistant add-on, providing the `PANGOLIN_ENDPOINT`, `NEWT_ID`, and `NEWT_SECRET`:
* https://docs.fossorial.io/Community%20Guides/homeassistant
* https://github.com/Ferdinand99/home-assistant-newt-addon
* https://github.com/orgs/fosrl/discussions/242

Using the [[HomeAssistant - VSCode add-on]], make the following changes to `configuration.yaml`:
```yaml
homeassistant:
  allowlist_external_urls:
    - "https://ha.my.domain"

http:
  use_x_forwarded_for: true
  trusted_proxies:
    - 127.0.0.1
    - 172.30.33.0/24 # internal homeassistant IP range for add-ons
    - x.x.x.x        # IP address of my.domain
```
