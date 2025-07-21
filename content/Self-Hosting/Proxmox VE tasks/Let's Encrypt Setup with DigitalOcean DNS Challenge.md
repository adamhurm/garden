---
created: 2025-01-08T16:38
updated: 2025-07-21T17:35
tags:
  - proxmox
  - digitalocean
  - letsencrypt
  - shareable
---
Proxmox documentation: https://pve.proxmox.com/wiki/Certificate_Management

## Configure ACME

Datacenter -> ACME -> 
	Accounts -> Click **Add**
		Account Name: `default`
		E-Mail: `me@my.domain`
		ACME Directory: `Let's Encrypt V2`
		Accept TOS: ✅
	Challenge Plugins -> Click **Add**
		Plugin ID: `proxmox-my-domain`
		DNS API: `DigitalOcean DNS`
		DO_API_KEY: `dop_v1_+++++`

## Create and submit certificate request

proxmox (node) -> Certificates -> ACME -> 
	Click **Add**
		Challenge Type: `DNS`
		Plugin: `proxmox-my-domain`
		Domain: `proxmox.my.domain`
	 Click **Order Certificates Now**
