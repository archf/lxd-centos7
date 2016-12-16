## Building LXD from source

### Using the provided Makefile

Configure a bridge called `br0`.

It must be active as a `dnsmasq` instance will launch and bind to it.

Then:

```bash
make all
```

### Manual steps

Those are kept here for information

Install build depencies

```bash
sudo yum install golang.x86_64 criu.x86_64 squashfs-tools.x86_64
```

```bash
mkdir -p ~/go
export GOPATH=~/go

go get github.com/lxc/lxd
cd $GOPATH/src/github.com/lxc/lxd
git checkout lxd-2.5
make
```

### Install

```bash
sudo cp $GOPATH/bin/lx* /usr/local/bin/
sudo groupadd --system lxd
sudo usermod -a -G lxd archf
sudo chown root:lxd /usr/local/bin/lx*
```

```bash
sudo cp $GOPATH/bin/lxc /usr/local/bin/lxc
sudo cp $GOPATH/bin/lxd /usr/local/bin/lxd
```

NOTE: Now you can run the daemon (the --group sudo bit allows everyone in the
    sudo group to talk to LXD; you can create your own group if you want):

### Start the daemon

```bash
sudo -E $GOPATH/bin/lxd --group lxd
```

## LXD minimal configuration

Everything is in a SQLITE db. There are no files.

### Remote hypervisor

Use the small configuration wizard.

``bash
sudo -E lxd init
```

Or manually:

This will tell LXD to listen to port 8443 on all addresses.

```bash
lxc config set core.https_address :8443
lxc config set core.trust_password <password>
```

### Remote LVM storage backend preparation

Create a lvm thinpool

```bash
lvcreate -l 80%Free --type thin-pool centos/LXDPool
lvcreate --size 800Gb --type thin-pool centos/LXDPool
```

### Client side

Add the remote that you allowed listening over tcp. You will have to
accept//validate the certificate fingerprint.

```bash
lxc remote add <remotename> <http_address>
```

You can now interact remotely.

```bash
lxc config show <remotename>:
```

## LXD advanced configuration

### core, storage and images

```bash
lxc config set storage.lvm_vg_name "centos"
lxc config set storage.lvm_volume_size 72GB
# lxc config set storage.lvm_thin_pool_name LXDPool
lxc config set images.remote_cache_expiry 30
lxc config set images.auto_update_interval 30
```

### Limit container ressources

```bash
lxc config set <container> limits.memory.enforce soft
lxc config set <container> limits.memory 9%
```

But you can add this to the default profile

```bash
lxc profile set default limits.memory.enforce soft
lxc profile set default limits.memory 9%
```

## Network management

Setting you can set using `lxc network set`:

```
bridge.driver
ipv4.address
ipv4.nat
ipv4.dhcp
ipv4.dhcp_ranges
ipv4.routing
dns.domain
dns.mode
```

## Image management

### find images

List images by filter:

```bash
lxc list images: centos/6/
```

Pull images:

```bash
# lxc image copy [remote:]<image> <remote>: [--alias=ALIAS].. [--copy-aliases]
# [--public] [--auto-update]
#     Copy an image from one LXD daemon to another over the network.
lxc image copy images:centos/6 local:
lxc image copy images:centos/7 local:
```

list by alias:

```bash
lxc image alias list images: centos/6/
```

You could also define your own aliases

Create an alias from an existing image:

```bash
[13:36:56|root@hp-380-01:/]# lxc image alias create centos6 ddc9c426da28
[13:36:56|root@hp-380-01:/]# lxc image alias create centos7
```

## Profile

```bash
lxc profile list

lxc profile set default security.privileged true
lxc profile set default limits.memory.enforce soft
lxc profile set default limits.memory 9%

set default nictype bridge
set default

lxc network set br0 bridge.driver native
lxc network set br0 dns.domain lan
lxc network set br0 ipv4.address 10.72.0.2/16

lxc network attach-profile br0 default

# this defaults to all addresses
lxc network set br0 ipv4.dhcp.ranges 10.72.0.2/1
```

## Launch container

sudo lxc launch centos7  c1
## debug

lxc info --show-log local:c1

container conf

less /var/log/lxd/c1/lxc.conf

lsblk ; umount /var/lib/lxd/containers/c1 ; lxc delete c1 ; lxc list

yum install dbus-devel.x86_64 automake libtool.x86_64 expat-devel.x86_64 expat
dbus-libs.x86_64 dbus-glib-devel.x86_64

# Python2-lxc

sudo yum install python-pip

disable repost w/o centos 6.7

sudo yum-config-manager --save --setopt=base.skip_if_unavailable=true

sudo yum install -y python-devel
sudo pip install --upgrade pip
sudo pip install git+https://github.com/lxc/python2-lxc.git

sudo yum install -y libvirt libvirt-login-shell.x86_64 libvirt-daemon-lxc.x86_64
libvirt-daemon-driver-lxc.x86_64 virt-manager.noarch xorg-x11-server-Xorg.x86_64

man lxc.container.conf
