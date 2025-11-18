---
created: 2025-09-29T12:45
updated: 2025-09-29T15:07
---
I am setting up a dev kubernetes cluster so that I can have a testing ground before deploying things to my prod cluster. That being said, that probably won't stop me from pushing breaking changes to prod to keep things interesting.

## Cluster hardware

**"Prod"**
- TP-Link 4-port Switch (TL-SG105)
- 3x Dell OptiPlex Mini 7060
**"Dev"**
- TP-Link 4-port PoE+ Switch (TL-SG1005P)
- 4x Raspberry Pi 4 (4GB RAM) w/ PoE+ HATs

## Getting Started

### Talos Linux image
I will be using [Talos Linux ](https://www.talos.dev), "The Kubernetes Operating System". I used the Talos [Image Factory](https://factory.talos.dev) service to create an image that I will flash to all my nodes. The setup wizard does a great job describing what information is needed and provides easily-searchable list of the system extensions.

I used balenaEtcher to flash the image to a microSD card for each Pi.

### Setup with talosctl
On my Mac machine, I installed talosctl and started the setup process:
```shell
# Install talosctl utility
brew install siderolabs/tap/talosctl

# Populate environment with cluster and node details
export CONTROL_PLANE_IP=10.0.0.20
export WORKER_IP=("10.0.0.21" "10.0.0.22" "10.0.0.23")
export CLUSTER_NAME=dev

# Confirm the target disk for installation
talosctl get disks --insecure --nodes $CONTROL_PLANE_IP
NODE   NAMESPACE   TYPE   ID        VERSION   SIZE     READ ONLY   TRANSPORT
       runtime     Disk   loop0     2         4.1 kB   true
       runtime     Disk   loop1     2         66 MB    true                   
       runtime     Disk   mmcblk0   2         31 GB    false       mmc       
export DISK_NAME=mmcblk0

# Generate talosconfig
talosctl gen config $CLUSTER_NAME https://$CONTROL_PLANE_IP:6443 --install-disk /dev/$DISK_NAME

# Apply talosconfig to control plane node
talosctl apply-config --insecure --nodes $CONTROL_PLANE_IP --file controlplane.yaml

# Apply talosconfig to worker nodes
for ip in "${WORKER_IP[@]}"; do
    echo "Applying config to worker node: $ip"
    talosctl apply-config --insecure --nodes "$ip" --file worker.yaml
done

# Configure endpoints
talosctl --talosconfig=./talosconfig config endpoints $CONTROL_PLANE_IP

# Watch health until cluster is in state 'waiting for etcd to be healthy'
talosctl health --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig

# Bootstrap etcd
talosctl bootstrap --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig

# Add talosconfig to kubeconfig
talosctl kubeconfig --nodes $CONTROL_PLANE_IP --talosconfig=./talosconfig
```


### Add additional cluster to ArgoCD
```shell
# Install ArgoCD CLI
adam@node0:~/$ curl -sSL -o argocd-linux-amd64 https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64
sudo install -m 555 argocd-linux-amd64 /usr/local/bin/argocd
rm argocd-linux-amd64

# Get ArgoCD Server IP
adam@node0:~/$ kubectl get svc -n argocd argocd-server
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
argocd-server   ClusterIP   10.49.213.39   <none>        80/TCP,443/TCP   385d

# Log in to ArgoCD Server via ArgoCD CLI
adam@node0:~$ argocd login 10.49.213.39:443
WARNING: server is not configured with TLS. Proceed (y/n)? y
Username: myuser
Password: 
'myuser:login' logged in successfully
Context '10.49.213.39:443' updated

# Add cluster to ArgoCD using kubectl context
adam@node0:~$ argocd cluster add admin@dev
WARNING: This will create a service account `argocd-manager` on the cluster referenced by context `admin@dev` with full cluster level privileges. Do you want to continue [y/N]? y
{"level":"info","msg":"ServiceAccount \"argocd-manager\" already exists in namespace \"kube-system\"","time":"2025-09-29T19:03:31Z"}
{"level":"info","msg":"ClusterRole \"argocd-manager-role\" updated","time":"2025-09-29T19:03:31Z"}
{"level":"info","msg":"ClusterRoleBinding \"argocd-manager-role-binding\" updated","time":"2025-09-29T19:03:31Z"}
{"level":"info","msg":"Using existing bearer token secret \"argocd-manager-long-lived-token\" for ServiceAccount \"argocd-manager\"","time":"2025-09-29T19:03:31Z"}
Cluster 'https://10.0.0.20:6443' added
```
## Additional configuration
- Talos provides [support documentation for iSCSI Storage with Synology CSI](https://www.talos.dev/v1.11/kubernetes-guides/configuration/synology-csi/)
- Longhorn provides [support documentation for Talos Linux](https://longhorn.io/docs/1.9.0/advanced-resources/os-distro-specific/talos-linux-support/)

