sudo k3s server &
# Kubeconfig is written to /etc/rancher/k3s/k3s.yaml
sudo k3s kubectl get node


Token - K10f8474fb1a0f04751444f2ef8f8f570ce1fb2ed85ec86070d55d1c0c67cec9a20::server:bd20737640ee5c19aaca77ef4703a6df

# On a different node run the below. NODE_TOKEN comes from /var/lib/rancher/k3s/server/node-token
# on your server
sudo k3s agent --server https://myserver:6443 --token ${NODE_TOKEN}

curl ... | K3S_TOKEN=xxx K3S_URL=https://server-url:6443 sh -


curl -sfL https://get.k3s.io | K3S_TOKEN=K10f8474fb1a0f04751444f2ef8f8f570ce1fb2ed85ec86070d55d1c0c67cec9a20::server:bd20737640ee5c19aaca77ef4703a6df K3S_URL=https://10.71.54.47:6443 --snapshotter=fuse-overlayfs sh -




curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent" sh -




curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL=latest  sh -s -     agent     --node-external-ip 10.71.54.226 --token K10f8474fb1a0f04751444f2ef8f8f570ce1fb2ed85ec86070d55d1c0c67cec9a20::server:bd20737640ee5c19aaca77ef4703a6df --disable traefik --disable servicelb --server https://10.71.54.47:6443



https://github.com/k3s-io/k3s/issues/3700



https://github.com/corneliusweig/kubernetes-lxd/blob/master/README-k3s.md


# k3s commands
- curl -sfL https://get.k3s.io | sh -

# commands - 

- export K3S_NODE_NAME=k3s-worker
- export K3S_TOKEN=K10f8474fb1a0f04751444f2ef8f8f570ce1fb2ed85ec86070d55d1c0c67cec9a20::server:bd20737640ee5c19aaca77ef4703a6df
- export K3S_URL=https://10.71.54.47:6443

- lxc launch images:debian/11/amd64 k3s-master  --profile k8s
- lxc exec k3s-master bash
- apt install curl
- curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--cluster-init --flannel-backend=vxlan --snapshotter=zfs --container-runtime-endpoint unix:///run/containerd/containerd.sock" sh -
- lxc stop k3s-master
- lxc delete k3s-master
- lxc profile edit k8s
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
    lxc.autodev=1
    lxc.mount.entry = /dev/kmsg dev/kmsg none defaults,bind,create=file
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



level=info msg="To join node to cluster: k3s agent -s https://10.71.54.81:6443 -t ${NODE_TOKEN}"


curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="server --snapshotter=zfs --container-runtime-endpoint unix:///run/containerd/containerd.sock" sh -
