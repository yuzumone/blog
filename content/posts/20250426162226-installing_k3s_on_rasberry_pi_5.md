+++
title = "Installing k3s on rasberry pi 5"
author = ["yuzumone"]
date = 2025-03-05
slug = "installing-k3s-on-rasberry-pi-5"
tags = ["Kubernetes"]
draft = false
toc = true
image = "https://images.unsplash.com/photo-1631553127988-36343ac5bb0c?w=800&auto=format&fit=crop&q=60&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxzZWFyY2h8N3x8cmFzcGJlcnJ5JTIwcGl8ZW58MHx8MHx8fDA%3D"
+++

{{< figure title="Photo by Jainath Ponnala on Unsplash" src="https://images.unsplash.com/photo-1631553127988-36343ac5bb0c?q=80&w=2670&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D" >}}


## Raspberry Pi Imager {#raspberry-pi-imager}

ssh とか昔は mkdir とかやった記憶なんですが，今は Imager の設定で ssh や鍵認証まで設定できるようになっていました． <br/>


## Static IP {#static-ip}

現在は NetworkManager で設定するのがナウいようです． <br/>
初期は設定ファイルがないので名前を設定することでまずはファイルを生成します． <br/>

```bash
yuzumone@raspberrypi:~ $ nmcli connection show
NAME                UUID                                  TYPE      DEVICE
Wired Connection 1  9f707e45-298e-39b3-80ef-5bbd96c03f46  ethernet  eth0

yuzumone@raspberrypi:~ $ nmcli connection modify 9f707e45-298e-39b3-80ef-5bbd96c03f46 connection.id wired_eth0
```

/etc/NetworkManager/system-connections 配下に 名前.nmconnection でファイルが生成されます． <br/>
あとは `[ipv4]` のところに設定します． <br/>

```bash
yuzumone@raspberrypi:~ $ sudo cat /etc/NetworkManager/system-connections/wired_eth0.nmconnection
[connection]
id=wired_eth0
uuid=9f707e45-298e-39b3-80ef-5bbd96c03f46
type=ethernet
autoconnect-priority=-999
interface-name=eth0
timestamp=1741091271

[ethernet]

[ipv4]
method=manual
addresses=192.168.0.100/24
gateway=192.168.0.1
dns=8.8.8.8;8.8.4.4;

[ipv6]
addr-gen-mode=default
method=auto

[proxy]
```

設定したあとは `sudo systemctl restart NetworkManager` をして適応します． <br/>


## Storage {#storage}

Longhorn を利用するので /var/lib/longhorn に mount することにします． <br/>

```bash
yuzumone@raspberrypi:~ $ sudo mkfs -t ext4 /dev/sda
yuzumone@raspberrypi:~ $ sudo mkdir /var/lib/longhorn
yuzumone@raspberrypi:~ $ sudo chmod 777 /var/lib/longhorn
yuzumone@raspberrypi:~ $ sudo mount -t ext4 /dev/sda /var/lib/longhorn
```

blkid で UUID を確認して /etc/fstab に追記しておきます． <br/>

```bash
yuzumone@raspberrypi:~ $ blkid
/dev/sda: UUID="919fb770-af9c-487f-a758-6190b524497c" BLOCK_SIZE="4096" TYPE="ext4"
yuzumone@raspberrypi:~ $ cat /etc/fstab
proc            /proc           proc    defaults          0       0
PARTUUID=b5d3bfc3-01  /boot/firmware  vfat    defaults          0       2
PARTUUID=b5d3bfc3-02  /               ext4    defaults,noatime  0       1
# a swapfile is not a swap partition, no line here
#   use  dphys-swapfile swap[on|off]  for that

UUID="919fb770-af9c-487f-a758-6190b524497c" /var/lib/longhorn ext4 defaults 0 0
```


## swap off {#swap-off}

swap の利用は非推奨なので off します． <br/>
`cat /proc/swaps` で確認ができます． <br/>

```bash
yuzumone@raspberrypi:~ $ sudo swapoff --all
yuzumone@raspberrypi:~ $ sudo systemctl stop dphys-swapfile
yuzumone@raspberrypi:~ $ sudo systemctl disable dphys-swapfile
yuzumone@raspberrypi:~ $ sudo systemctl status dphys-swapfile
yuzumone@raspberrypi:~ $ cat /proc/swaps
Filename                                Type            Size            Used            Priority
```


## Enable cgroups {#enable-cgroups}

cgroup_memory=1 cgroup_enable=memory を /boot/firmware/cmdline.txt に追記． <br/>

```bash
yuzumone@raspberrypi:~ $ cat /boot/firmware/cmdline.txt
console=serial0,115200 console=tty1 root=PARTUUID=b5d3bfc3-02 rootfstype=ext4 fsck.repair=yes rootwait cgroup_memory=1 cgroup_enable=memory
```


## Install k3s {#install-k3s}

クラスタは組まないのでワンライナーのやつで適当にインストールします． <br/>
クラスタを組む場合は ansible role を利用するのが良さそうです． <br/>

```bash
yuzumone@raspberrypi:~ $ curl -sfL https://get.k3s.io | sh -s - --write-kubeconfig-mode 644
yuzumone@raspberrypi:~ $ kubectl get nodes
NAME          STATUS   ROLES                  AGE   VERSION
raspberrypi   Ready    control-plane,master   17s   v1.31.6+k3s1
```


## Install Longhorn {#install-longhorn}

実際に立てるサービスは cloudflare-tunnel-ingress-controller を利用しようと思うので，内々で見れれば良いものは NodePort で適当にアクセスすることにします． <br/>

```bash
sudo apt -y install open-iscsi
```

```bash
helm repo add longhorn https://charts.longhorn.io
helm repo update
helm install longhorn longhorn/longhorn --namespace longhorn-system --create-namespace -f values.yaml
```

```bash
$ kubectl -n longhorn-system get po1
NAME                                        READY   STATUS    RESTARTS   AGE
engine-image-ei-c2d50bcc-mnnll              1/1     Running   0          10s
longhorn-driver-deployer-7fc7ffcb7f-mqkdg   1/1     Running   0          20s
longhorn-manager-l5ggl                      2/2     Running   0          20s
longhorn-ui-5fcc4bcfc7-dpgcw                1/1     Running   0          20s
longhorn-ui-5fcc4bcfc7-ll6wm                1/1     Running   0          20s
```

