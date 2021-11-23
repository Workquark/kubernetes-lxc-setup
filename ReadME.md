# Running LXC container and k3s on LXC alpine container - 

### Commands on HOST (alpine)- 

<p>
These commands would prepare the host and create the container to be launched on LXC.Mountpoints for the lxc containers are very important for k3s to understand here in the config.
</p>


### Refer to this for installing apine on LXD 

```
https://wiki.alpinelinux.org/wiki/LXD
```

### Add the kernel tools -
```
apk add conntrack-tools 
```

### Prepare the storage for LXD storage - 

```
 sudo mkdir -p /data/lxd (make storage data folder)
 sudo chgrp lxd /data/lxd ( assign the folder to lxd group)
 sudo chmod g+rwx /data/lxd ( give the group permission to /data/lxd)
```

### Create the storage underneath for lxc using /data/lxd 
```
lxc storage create lxc-data dir source=/data/lxd
```

### Create and edit the profile 

```
 lxc profile copy default k8s
 lxc profile edit k8s ( add the config below mentioned in section)
```

### Add this config using vi tool to edited LXC profile ( k8s in this case)
```
config:
  raw.lxc: |-
    lxc.init.cmd = /sbin/init systemd.unified_cgroup_hierarchy=1
    lxc.mount.auto = cgroup:rw:force

    lxc.cgroup2.devices.allow=a
    linux.kernel_modules=ip_tables,ip6_tables,netlink_diag,nf_nat,overlay,xt_conntrack
    lxc.autodev=1
    lxc.mount.entry = /dev/kmsg /dev/kmsg none defaults,bind,create=file
    lxc.apparmor.profile=unconfined
    security.nesting=true
description: Default LXD profile
devices:
  eth0:
    name: eth0
    network: lxdbr0
    type: nic
  root:
    path: /
    pool: lxc-data
    type: disk
name: k8s

```

### See the profile 
```
 lxc config show k3s-master --expanded ( optional to see the lxc configuration for k3s-master)
```

### Launch the container using the profile 
```
 lxc launch images:debian/10/amd64 k3s-master  --profile k8s ( launch the k3s-master with the `k8s` LXC config)
```

### Exec into the container bash - 

```
 lxc exec k3s-master bash ( exec into the k3s-master lxc container )
```

### Notes: LXC Host should be configured

```
On Alpine, here's a list of package add-ons applied to the host. This may be more than what's strictly necessary.

sudo apk add acl alpine-base alpine-baselayout alpine-conf alpine-keys apk-tools argon2-libs attr blkid brotli-libs busybox busybox-initscripts busybox-suid ca-certificates ca-certificates-bundle cfdisk cgmanager cgmanager-openrc chrony chrony-openrc conntrack-tools conntrack-tools-openrc cryptsetup-libs curl dbus dbus-libs dbus-openrc device-mapper-libs dnsmasq dqlite e2fsprogs e2fsprogs-extra e2fsprogs-libs eudev-libs expat findmnt findutils flock fts fuse fuse-common fuse-openrc gmp gnutls hexdump ifupdown-ng ifupdown-ng-iproute2 ip6tables ip6tables-openrc iproute2 iproute2-minimal iproute2-ss iproute2-tc iptables iptables-openrc json-c kmod kmod-libs kmod-openrc lddtree libacl libattr libblkid libbsd libbz2 libc-utils libcap libcap-ng libcom_err libcrypto1.1 libcurl libeconf libedit libelf libevent libfdisk libffi libgcc libintl libmd libmnl libmount libnetfilter_conntrack libnetfilter_cthelper libnetfilter_cttimeout libnetfilter_queue libnfnetlink libnftnl libnih libproc libretls libseccomp libsmartcols libssl1.1 libstdc++ libtasn1 libunistring libuuid libuv linux-firmware linux-lts linux-pam logger lsblk lua-lunix lua-optarg lua5.1 lua5.1-libs lua5.1-lunix lua5.1-optarg lxc lxc-libs lxc-openrc lxcfs lxcfs-openrc lxd lxd-openrc lz4-libs lzo mcookie mkinitfs mtools musl musl-utils nano ncurses-libs ncurses-terminfo-base netcat-openbsd nettle nghttp2-libs nvme-cli openrc openssh openssh-client-common openssh-client-default openssh-keygen openssh-server openssh-server-common openssh-sftp-server p11-kit partx popt procps psmisc raft rsync rsync-openrc runuser scanelf setpriv sfdisk shadow shadow-uidmap sqlite-libs squashfs-tools ssl_client sudo syslinux tar tiny-ec2-bootstrap tmux tzdata uidmapshift util-linux util-linux-misc util-linux-openrc uuidgen wipefs xz xz-libs zlib zstd zstd-libs

The kernel used is linux-lts

The changes to /boot/extlinux.conf look like:

APPEND root=LABEL=/ modules=sd-mod,usb-storage,ext4,nvme,ena console=ttyS0,115200n8 nvme_core.io_timeout=4294967295 cgroup_enable=cpuset,memory cgroup_memory=1 swapaccount=1 systemd.unified_cgroup_hierarchy=1 elevator=noop cgroup_cpuset=1

(The part after the io_timeout is the part that was added).

There's a chance that using some later packages helped (when I did a big upgrade, this was with the edge/testing repository enabled. Will need to see how well it works with stable).

/etc/modules contains: af_packet ipv6 ip_tables ip6_tables netlink_diag nf_nat overlay xt_conntrack ip_vs ip_vs_rr ip_vs_wrr ip_vs_sh nf_conntrack br_netfilter tun
```

<!-- - apt install curl kmod containerd -y -->

<!-- - export INSTALL_K3S_EXEC="--write-kubeconfig ~/.kube/config --write-kubeconfig-mode 666 --snapshotter=native --node-external-ip 10.204.153.82 --kubelet-arg=cgroup-driver=systemd" -->

<!-- - curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--cluster-init --flannel-backend=vxlan --snapshotter=zfs --container-runtime-endpoint unix:///run/containerd/containerd.sock" sh - -->


# Master Instance - 
<p>These commands are to be run inside the bash for the master LXC instance . So first you need to exec into bash for the LXC master instance ( **k3s-master** in this case )</p>

### Get inside the container - 

```
lxc exec k3s-master bash
```

### Run inside LXC container bash 

```
- apt install curl kmod -y
- sudo hostnamectl set-hostname k3s-master
- export K3S_NODE_NAME=k3s-master
- export INSTALL_K3S_EXEC="--write-kubeconfig ~/.kube/config --write-kubeconfig-mode 666 --snapshotter=native --node-external-ip 10.162.97.207"

```

### Install K3S on your linux container bash - 
```
curl -sfL https://get.k3s.io | sh -
```

### Check logs for k3s systemctl service -

``` 
journalctl -xeu k3s -f 
```

### Notes link /dev/kmsg for container - 

```
ln -s /dev/console /dev/kmsg
```

# Worker Instance -

### Launch the worker instance - 

```
lxc launch images:debian/10/amd64 k3s-worker  --profile k8s

```

### Check the master LXC IP from host - 

```
lxc list

output - 

+------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
|    NAME    |  STATE  |         IPV4          |                     IPV6                      |   TYPE    | SNAPSHOTS |
+------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| k3s-master | RUNNING | 10.42.0.1 (cni0)      | fd42:8e24:3335:1237:216:3eff:fe78:b1cf (eth0) | CONTAINER | 0         |
|            |         | 10.42.0.0 (flannel.1) |                                               |           |           |
|            |         | 10.204.153.164 (eth0) |                                               |           |           |
+------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+
| k3s-worker | RUNNING | 10.204.153.198 (eth0) | fd42:8e24:3335:1237:216:3eff:fee8:e9f8 (eth0) | CONTAINER | 0         |
+------------+---------+-----------------------+-----------------------------------------------+-----------+-----------+

```

### Also get the token - 

```
:~$ lxc exec k3s-master cat /var/lib/rancher/k3s/server/node-token
K1021fd4add35fb06418a1e2e91badd1abf4f95a1feaba8cccd02170abdc9dc0014::server:59f3a09d5f7f12c275a95b81a6a5b777
```

### Export variables in Worker LXC container - 

```
export K3S_NODE_NAME=k3s-worker
export K3S_KUBECONFIG_MODE="644"

export K3S_URL="https://10.204.153.164:6443" ( Give the master IP here)
export K3S_TOKEN=K1021fd4add35fb06418a1e2e91badd1abf4f95a1feaba8cccd02170abdc9dc0014::server:59f3a09d5f7f12c275a95b81a6a5b777
export INSTALL_K3S_EXEC="--snapshotter=native"
```

### Install the k3s agent - 
```
curl -sfL https://get.k3s.io | sh -
```




#  Work in progress section 

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

### Stop LXC container - 
- lxc stop k3s-master
- lxc delete k3s-master --force
- lxc profile edit k8s
- lxc file pull k3s-master/etc/rancher/k3s/k3s.yaml ~/.kube/config
- sed -i 's:127.0.0.1:k3s-master:;s:default:k3s-lxc:g' ~/.kube/config
- Uninstall k3s - /usr/local/bin/k3s-uninstall.sh


### Add user to sudoers group (debian) -

```
#usermod -aG sudo $(whoami)
```

### Add debian sources list -
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



---

# Kubeconfig is written to /etc/rancher/k3s/k3s.yaml
sudo k3s kubectl get node


Token - K10e40e16d96834cc6be8344db89f55fb149b6ea7f6ceac76f6e2b4b851eef28f05::server:c7832d09227c98127d79cb887cc6a886

# On a different node run the below. NODE_TOKEN comes from /var/lib/rancher/k3s/server/node-token
# on your server
sudo k3s agent --server https://myserver:6443 --token ${NODE_TOKEN}

curl ... | K3S_TOKEN=xxx K3S_URL=https://server-url:6443 sh -


curl -sfL https://get.k3s.io | K3S_TOKEN=K1021fd4add35fb06418a1e2e91badd1abf4f95a1feaba8cccd02170abdc9dc0014::server:59f3a09d5f7f12c275a95b81a6a5b777 K3S_URL=https://10.204.153.164:6443 --snapshotter=fuse-overlayfs sh -




curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="agent" sh -




curl -sfL https://get.k3s.io | INSTALL_K3S_CHANNEL=latest  sh -s -     agent     --node-external-ip 10.71.54.226 --token K10f8474fb1a0f04751444f2ef8f8f570ce1fb2ed85ec86070d55d1c0c67cec9a20::server:bd20737640ee5c19aaca77ef4703a6df --disable traefik --disable servicelb --server https://10.71.54.47:6443



https://github.com/k3s-io/k3s/issues/3700



https://github.com/corneliusweig/kubernetes-lxd/blob/master/README-k3s.md






## Agent Configuration


# Configure Worker Agent - 

### Get the Token from the Server - 
```
lxc exec k3s-master cat /var/lib/rancher/k3s/server/node-token

```
### Export the variables required for agent - 
```
- export K3S_NODE_NAME=k3s-worker
- export K3S_TOKEN=K1021fd4add35fb06418a1e2e91badd1abf4f95a1feaba8cccd02170abdc9dc0014::server:59f3a09d5f7f12c275a95b81a6a5b777
- export K3S_URL=https://10.204.153.164:6443
```


# LXC Profile



**Notes**: 
```


They were cgroup related. I added: cgroup_enable=cpuset,memory cgroup_memory=1 swapaccount=1 systemd.unified_cgroup_hierarchy=1 elevator=noop cgroup_cpuset=1

Note that this assumes that cgroup2 is preferred/needed. The systemd.unified_cgroup_hierarchy=1 may be spurious (it seems to indicate to use v2) but at least it wouldn't hurt. Maybe some non-systemd tools parse that and decide upon cgroup use.
```

**Notes**: 

```
Added to the Alpine instance. Also, the Ubuntu 18 LTS is ubuntu@10.144.177.86

Note that the /boot/extlinux.conf additions themselves are an experiment. They may need some tweaking if this doesn't work, or there may be other issues.

A counterexample comment (may or may not be spot on): Make sure you have removed cgroup_enable from your cmdline as this will fail to mount cgroups and fail LXC service / ref https://wiki.alpinelinux.org/wiki/LXC

Note that if the guest container needs a certain configuration (and apparently, if the guest container also uses systemd), it seems there's an option to pass in a parameter, like: lxc.init.cmd = /sbin/init systemd.unified_cgroup_hierarchy=1
There is another means to do the above I believe: lxc.mount.auto = cgroup:rw:force
These have interesting reading: https://wiki.debian.org/LXC#Unprivileged_container https://wiki.debian.org/LXC/CGroupV2 https://linuxcontainers.org/lxc/manpages/man5/lxc.container.conf.5.html

There are potential options like:

lxc.cgroup.cpuset.cpus = 0,1
lxc.cgroup.cpu.shares = 1234
lxc.cgroup.devices.deny = a
lxc.cgroup.devices.allow = c 1:3 rw
lxc.cgroup.devices.allow = b 8:0 rw


This https://github.com/cnrancher/autok3s may embed common principles related to k3s instantiation.

This https://github.com/tianon/cgroupfs-mount may involve a way to manually mount cgroupfs points.


```




https://wiki.kobol.io/helios64/software/zfs/install-zfs/




<!-- curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--disable-network-policy --cluster-init  --container-runtime-endpoint unix:///run/containerd/containerd.sock" sh - -->



<!-- curl -sfL https://get.k3s.io | INSTALL_K3S_EXEC="--cluster-init --snapshotter=zfs --container-runtime-endpoint unix:///run/containerd/containerd.sock --cluster-cidr 10.45.0.0/16" sh - -->



<!-- # Calico Setup -  -->

<!-- - SEE - https://coder.com/docs/coder/latest/setup/kubernetes/k3s -->

<!-- - curl -sfL https://get.k3s.io | K3S_KUBECONFIG_MODE="644" INSTALL_K3S_EXEC="--flannel-backend=none --cluster-cidr=10.45.0.0/16 --disable-network-policy --disable=traefik" sh - -->
