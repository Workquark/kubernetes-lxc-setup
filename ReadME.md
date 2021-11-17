## Build and install LXD from source - 

- Download LXD from `https://linuxcontainers.org/lxd/downloads/` 
- Example - `wget https://linuxcontainers.org/downloads/lxd/lxd-4.20.tar.gz` 
- Extract the tar - `tar zxvf lxd-4.20.tar.gz`
- Get into folder - `cd lxd-4.20`
- Add non-free debian-11 repos from here - `https://wiki.debian.org/SourcesList`
```
deb http://deb.debian.org/debian-security/ bullseye-security main contrib non-free
deb-src http://deb.debian.org/debian-security/ bullseye-security main contrib non-free
```
- Install components for build - 
```
sudo apt-get install autoconf libtool liblz4-dev sqlite3 libsqlite3-dev
```

## Add user to sudoers group (debian) -

```
#usermod -aG sudo $(whoami)
```

## Add debian sources list -
```
# add to /etc/apt/sources.list from -

- https://wiki.debian.org/SourcesList 

```

### - As example see section -
```
 

deb http://deb.debian.org/debian bullseye main contrib non-free
deb-src http://deb.debian.org/debian bullseye main contrib non-free

deb http://deb.debian.org/debian-security/ bullseye-security main contrib non-free
deb-src http://deb.debian.org/debian-security/ bullseye-security main contrib non-free

deb http://deb.debian.org/debian bullseye-updates main contrib non-free
deb-src http://deb.debian.org/debian bullseye-updates main contrib non-free

```

### Install snap on debian -

```
sudo snap install snapd

sudo snap install core

 - reboot ( important ) 
```


## Install LXC (ubuntu)- 

- sudo apt-get install lxc

## Install LXC on Debian - 

- sudo snap install lxc

## Enable lxd daemon - 

- sudo systemctl enable snap.lxd.daemon

### add user to group

- sudo adduser $(whoami) lxd
- REBOOT completely OR execute commmand - loginctl terminate-user $(whoami)
- The above command is to take effect for the change of assignment of the user to new group.

# k3s commands
- curl -sfL https://get.k3s.io | sh -



# Kubeconfig is written to /etc/rancher/k3s/k3s.yaml
sudo k3s kubectl get node


Token - K10e40e16d96834cc6be8344db89f55fb149b6ea7f6ceac76f6e2b4b851eef28f05::server:c7832d09227c98127d79cb887cc6a886

# On a different node run the below. NODE_TOKEN comes from /var/lib/rancher/k3s/server/node-token
# on your server
sudo k3s agent --server https://myserver:6443 --token ${NODE_TOKEN}

curl ... | K3S_TOKEN=xxx K3S_URL=https://server-url:6443 sh -


curl -sfL https://get.k3s.io | K3S_TOKEN=K10f8474fb1a0f04751444f2ef8f8f570ce1fb2ed85ec86070d55d1c0c67cec9a20::server:bd20737640ee5c19aaca77ef4703a6df K3S_URL=https://10.71.54.47:6443 --snapshotter=fuse-overlayfs sh -




curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent" sh -




curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL=latest  sh -s -     agent     --node-external-ip 10.71.54.226 --token K10f8474fb1a0f04751444f2ef8f8f570ce1fb2ed85ec86070d55d1c0c67cec9a20::server:bd20737640ee5c19aaca77ef4703a6df --disable traefik --disable servicelb --server https://10.71.54.47:6443



https://github.com/k3s-io/k3s/issues/3700



https://github.com/corneliusweig/kubernetes-lxd/blob/master/README-k3s.md




# commands agent - 

- export K3S_NODE_NAME=k3s-worker
- export K3S_TOKEN=K10e40e16d96834cc6be8344db89f55fb149b6ea7f6ceac76f6e2b4b851eef28f05::server:c7832d09227c98127d79cb887cc6a886
- export K3S_URL=https://10.71.54.47:6443

# k3s commands server - 
- lxc launch images:debian/10/amd64 k3s-master  --profile k8s
- lxc config show k3s-master
- lxc exec k3s-master bash
- apt install curl containerd kmod -y
- curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--cluster-init --flannel-backend=vxlan --snapshotter=zfs --container-runtime-endpoint unix:///run/containerd/containerd.sock" sh -
- lxc stop k3s-master
- lxc delete k3s-master
- lxc profile edit k8s
- lxc file pull k3s-master/etc/rancher/k3s/k3s.yaml ~/.kube/config
- sed -i 's:127.0.0.1:k3s-master:;s:default:k3s-lxc:g' ~/.kube/config
- Uninstall k3s - /usr/local/bin/k3s-uninstall.sh


- journalctl -xeu k3s -f 

# LXC Profile

```
config:
  linux.kernel_modules: ip_tables,ip6_tables,netlink_diag,nf_nat,overlay,xt_conntrack
  raw.lxc: lxc.mount.auto=proc:rw sys:rw
  security.nesting: "true"
  security.privileged: "true"
description: Default LXD profile
devices:
  eth0:
    name: eth0
    network: lxdbr0
    type: nic
  root:
    path: /
    pool: btrfs-storage
    type: disk
name: k8s
used_by: []

```



```
config:
  raw.lxc: |-
    linux.kernel_modules= ip_tables,ip6_tables,netlink_diag,nf_nat,overlay,xt_conntrack
    lxc.autodev=1
    lxc.mount.entry = /dev/kmsg dev/kmsg none defaults,bind,create=file
    lxc.apparmor.profile=unconfined
    security.nesting=true
description: Kubernetes LXD profile
devices:
  eth0:
    name: eth0
    network: lxdbr0
    type: nic
  root:
    path: /
    pool: default 
    type: disk
name: k8s

```





https://wiki.kobol.io/helios64/software/zfs/install-zfs/




curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable-network-policy --cluster-init  --container-runtime-endpoint unix:///run/containerd/containerd.sock" sh -



curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--cluster-init --snapshotter=zfs --container-runtime-endpoint unix:///run/containerd/containerd.sock --cluster-cidr 10.45.0.0/16" sh -



# Calico Setup - 

- SEE - https://coder.com/docs/coder/latest/setup/kubernetes/k3s

- curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--flannel-backend=none --cluster-cidr=10.45.0.0/16 --disable-network-policy --disable=traefik" sh -
