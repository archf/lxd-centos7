## Build and configure LXC and LXD on centOS 7

You need to build LXC first and then and LXD. See the `README` files in respective
folders for details.

There are `Makefiles` you can use to grab dependancies, configure and compile
with the right flags that worked for me.

## Network setup

#### Disable firewalld

```
sudo systemctl stop firewalld.service
```

Alternatively you could allow traffic to the 8443 port.

### Permanently disable checksum offloading on your bridge device

```bash
cat << 'EOF' | sudo tee /sbin/ifup-local

if [ '${DEVICE}' = 'br0' ]
then
  /sbin/ethtool -K ${DEVICE} tx off
fi

EOF
sudo chmod +x /sbin/ifup-local
```

### With NAT

```bash
sudo systemctl start iptables
sudo systemctl enable iptables
sudo systemctl start ebtables
sudo systemctl enable ebtables
```

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward
LXC_BRIDGE=br0
LXC_NETWORK=10.72.0.0/16
use_iptables_lock="-w"
iptables $use_iptables_lock -I INPUT -i ${LXC_BRIDGE} -p udp --dport 67 -j ACCEPT
iptables $use_iptables_lock -I INPUT -i ${LXC_BRIDGE} -p tcp --dport 67 -j ACCEPT
iptables $use_iptables_lock -I INPUT -i ${LXC_BRIDGE} -p udp --dport 53 -j ACCEPT
iptables $use_iptables_lock -I INPUT -i ${LXC_BRIDGE} -p tcp --dport 53 -j ACCEPT
iptables $use_iptables_lock -I FORWARD -i ${LXC_BRIDGE} -j ACCEPT
iptables $use_iptables_lock -I FORWARD -o ${LXC_BRIDGE} -j ACCEPT
iptables $use_iptables_lock -t nat -A POSTROUTING -s ${LXC_NETWORK} ! -d ${LXC_NETWORK} -j MASQUERADE
```

OFACE=ens1f3
iptables $use_iptables_lock -I INPUT -i ${OFACE} -p udp --dport 53 -j ACCEPT
iptables $use_iptables_lock -I INPUT -i ${OFACE} -p tcp --dport 53 -j ACCEPT

Add nating:

```bash
iptables $use_iptables_lock -t mangle -A POSTROUTING -o ${LXC_BRIDGE} -p udp -m udp --dport 68 -j CHECKSUM --checksum-fill
```

### Without NAT

This is not recommended if there is another DHCP on your network. I did not
find a way to block outside DHCP traffic.

*Tentative rules*

you might need this if you have another dhcp server on the network and no
nating.

Block outside DHCP requests

```bash
iptables $use_iptables_lock -I INPUT -p udp --dport 67 -m physdev --physdev-out ens1f3 -j DROP
iptables $use_iptables_lock -I INPUT -p tcp --dport 67 -m physdev --physdev-out ens1f3 -j DROP

iptables $use_iptables_lock -I OUTPUT -p udp --dport 67 -m physdev --physdev-in ens1f3 -j DROP
iptables $use_iptables_lock -I OUTPUT -p tcp --dport 67 -m physdev --physdev-in ens1f3 -j DROP

iptables $use_iptables_lock -I FORWARD -p udp --dport 67 -m physdev --physdev-in ens1f3 -j DROP
iptables $use_iptables_lock -I FORWARD -p tcp --dport 67 -m physdev --physdev-in ens1f3 -j DROP

iptables $use_iptables_lock -I FORWARD -p tcp --dport 67 -o ens1f3 -j DROP
iptables $use_iptables_lock -I FORWARD -p udp --dport 67 -o ens1f3 -j DROP

iptables $use_iptables_lock -I FORWARD -p tcp --dport 67 -i ens1f3 -j DROP
iptables $use_iptables_lock -I FORWARD -p udp --dport 67 -i ens1f3 -j DROP

iptables $use_iptables_lock -I OUTPUT -p tcp --dport 67 -o ens1f3 -j DROP
iptables $use_iptables_lock -I OUTPUT -p udp --dport 67 -o ens1f3 -j DROP

iptables $use_iptables_lock -I OUTPUT -p tcp --dport 67 -j DROP
iptables $use_iptables_lock -I OUTPUT -p udp --dport 67 -j DROP

iptables $use_iptables_lock -I FORWARD -p tcp --dport 67 -j DROP
iptables $use_iptables_lock -I FORWARD -p udp --dport 67 -j DROP

iptables $use_iptables_lock -I FORWARD -p udp --dport 67 -m physdev --physdev-in br0 -j DROP
iptables $use_iptables_lock -I FORWARD -p tcp --dport 67 -m physdev --physdev-in br0 -j DROP

iptables $use_iptables_lock -A FORWARD -p tcp --dport 67 -j LOG

```

Block outgoing response:

```bash
iptables $use_iptables_lock -I OUTPUT -p udp --dport 68 -m physdev --physdev-in ens1f3 -j DROP
iptables $use_iptables_lock -I OUTPUT -p tcp --dport 68 -m physdev --physdev-in ens1f3 -j DROP

iptables $use_iptables_lock -I FORWARD -p udp --dport 68 -m physdev --physdev-in ens1f3 -j DROP
iptables $use_iptables_lock -I FORWARD -p tcp --dport 68 -m physdev --physdev-in ens1f3 -j DROP

iptables $use_iptables_lock -I FORWARD -p udp --dport 68 -m physdev --physdev-in eno2 -j DROP
iptables $use_iptables_lock -I FORWARD -p tcp --dport 68 -m physdev --physdev-in eno2 -j DROP

```bash
iptables $use_iptables_lock -I INPUT -p udp --dport 68 -m physdev --physdev-in eno2 -j DROP
iptables $use_iptables_lock -I INPUT -p tcp --dport 68 -m physdev --physdev-in eno2 -j DROP
```

## debugging

```bash
sudo netstat -l -n -4 -p
```bash

## Centos 7.x quirks

### Sudoers

* Add `/usr/local/bin` to securepath
* Remove `always_set_home` directive

## Libvirt quirks

Avoid conflict with existing dnsmasq instance from libvirt.

To disable `virbr0` or create `lxcbr0` bridge with `libvirt`.

```bash
sudo virsh net-stop virbr0
virsh net-autostart --disable default

sudo virsh net-define --file  ~/lxcbr0.xml
sudo virsh net-autostart lxcbr0
sudo virsh net-start lxcbr0
```
