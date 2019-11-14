# Prerequisites

The helper node used in this document is a Fedora 29 x86_64 VM and it will use
a regular user instead root user (with some exceptions)

If a regular user is not created:

```bash
useradd -m ocp
echo ocp:password | chpasswd
echo "ocp ALL=(root) NOPASSWD:ALL" | tee -a /etc/sudoers.d/ocp_nopassword
visudo -cf /etc/sudoers.d/ocp_nopassword
```

Allow rootless containers for the `ocp` user:

```bash
sudo usermod --add-subuids 10000-75535 ocp
sudo usermod --add-subgids 10000-75535 ocp
```

Install podman and other required utils

```bash
dnf clean all
dnf install -y podman jq libguestfs-tools-c
dnf update -y
```

Disable all not needed interfaces in the host to avoid messing networking stuff
with containers as well as IPv6 if not needed:

```bash
# As root
for interface in 'eno2' 'eth2' 'eth3'; do
  cat > /etc/sysconfig/network-scripts/ifcfg-${interface} << EOF
DEVICE=${interface}
BOOTPROTO=none
ONBOOT=no
NETWORKING_IPV6=no
IPV6_AUTOCONF=no
EOF
done

# Disable IPV6
cat > /etc/sysctl.d/ipv6.conf << EOF
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
EOF

sed -i \
  -e 's/IPV6_AUTOCONF.*/IPV6_AUTOCONF=no/g' \
  -e 's/IPV6INIT.*/IPV6INIT=no/g' \
  /etc/sysconfig/network-scripts/*

sysctl -p /etc/sysctl.d/ipv6.conf
# Disable virbr0
rm -f /etc/libvirt/qemu/networks/autostart/default.xml
systemctl stop libvirtd

nmcli connection reload
systemctl restart NetworkManager
# Or even better
# reboot
```

Switch to the `ocp` user created before:

```bash
su - ocp
```

Create an ssh key to be injected in the OCP hosts:

```bash
ssh-keygen -t rsa -N '' -f ~/.ssh/id_rsa
```

[<< Back to README](../README.md) | [Next: Variables >>](1-variables.md)
