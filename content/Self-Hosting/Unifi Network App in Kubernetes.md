---
created: 2024-12-04T18:06
updated: 2025-07-21T18:29
tags:
  - unifi
  - kubernetes
  - shareable
---

I found that setting up Unifi Network App in Kubernetes was needlessly complicated and required too many cluster networking changes, so it was abandoned in favor of [[Unifi Network App in Proxmox VM]] (which was later replaced by a Dream Machine Pro once I needed a Unifi Protect controller).

# unifi-network-application
- https://github.com/bpetrikovics/unifi-network-application/tree/main
- https://github.com/adamhurm/unifi-network-application/tree/tls-and-secrets
- https://github.com/helm/chart-releaser
- https://github.com/helm/chart-releaser-action

## database creation
I ended up needing to use the explicit user+pass in configmap, the environmental variable wasn't working for mongo database creation:
```
db.getSiblingDB("{{ .Values.mongodb.dbname }}").createUser({
  user: "{{ .Values.mongodb.username }}",
  pwd: "{{ .Values.mongodb.password}}",
  roles: [
   {role: "dbOwner", db: "{{ .Values.mongodb.dbname}}"},
   {role: "dbOwner", db: "{{ .Values.mongodb.dbname_stat}}"},
   {role: "readWrite", db: "{{ .Values.mongodb.dbname }}"}
]});
db.getSiblingDB("{{ .Values.mongodb.dbname_stat}}").createUser({
  user: "{{ .Values.mongodb.username }}",
  pwd: "{{ .Values.mongodb.password}}",
  roles: [
	{role: "dbOwner", db: "{{ .Values.mongodb.dbname_stat}}"},
	{role: "readWrite", db: "{{ .Values.mongodb.dbname_stat}}"}
]});
```

## traefik ingress
https://www.adamhancock.co.uk/blog/deploying-unifi-on-kubernetes#traefik
I had to allow insecure backend certificates by editing the traefik deployment:
```
- --serverstransport.insecureskipverify=true
```

## device adoption issues
https://www.reddit.com/r/UNIFI/comments/jhx013/unifi_controller_selfhosted_public_fqdn_remote/

unifi-network-application-app logs:
```
:0 TCP candidates not supported yet
:0 Permanent error code on allocate request: 420 - . This was after receiving a valid nonce
:0 TURN instance failed: TURN id: 3; fd: 220 0.0.0.0:37316 -> 141.101.90.1:3478 (all_interfaces) DTLS id:
:0 Unable to read data from SCTP socket. Permanent error: (104) Connection reset by peer
:0 SCTP ingest failed
:0 SCTP ingest failed
:0 Unable to do SSL I/O
:0 webRtcId 1 terminated with code: (-2147090409) WebRTC connection interrupted from far side
```
- https://forums.unraid.net/topic/147455-support-unifi-controller-unifi-unraid-reborn/page/13/
- https://community.ui.com/questions/Importing-a-Site-get-error-There-was-an-error-adding-the-site-Algoma-Township/0f5b1a74-af58-443b-94e2-dd08eb727e1b
- https://medium.com/@reefland/migrating-unifi-network-controller-from-docker-to-kubernetes-5aac8ed8da76

