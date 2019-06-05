# Intro

The objective of this document is to provide instructions (automated ish) to install OCP4 on baremetal:
* without PXE (pretty common scenario in big companies)
* avoid installing stuff and use containers instead (instead yum/dnf install httpd, haproxy,... use containers)
* use rootless containers if possible
* use Fedora29/RHEL8 stuff (nmcli, firewalld, etc.)

# Current status

OCP4.1 GA installed

# Environment

| Usage     | Hostname                 | IP             | NOTES                                         |
|-----------|--------------------------|----------------|-----------------------------------------------|
| Helper    | ocp4-helper.minwi.lan    | 192.168.32.2   | DNS, httpd, etc.                              |
| Bootstrap | ocp4-bootstrap.minwi.lan | 192.168.32.99  | To be removed from the cluster once installed |
| Master-0  | ocp4-master-0.minwi.lan  | 192.168.32.100 |                                               |
| Master-1  | ocp4-master-1.minwi.lan  | 192.168.32.101 |                                               |
| Master-2  | ocp4-master-2.minwi.lan  | 192.168.32.102 |                                               |
| Worker-0  | ocp4-worker-0.minwi.lan  | 192.168.32.200 |                                               |

NOTE: Those baremetal servers are Dell based, so `racadm` will be used in order to manage the iDRAC to map a virtual cd, power off/on, etc.

# Prerequisites

The helper node used in this document is a Fedora 29 x86_64 VM and it will use a regular user instead root user (with some exceptions)

If a regular user is not created:

```
useradd -m ocp
echo ocp:password | chpasswd
echo "ocp ALL=(root) NOPASSWD:ALL" | tee -a /etc/sudoers.d/ocp_nopassword
visudo -cf /etc/sudoers.d/ocp_nopassword
```

Allow rootless containers for the 'ocp' user:

```
sudo usermod --add-subuids 10000-75535 ocp
sudo usermod --add-subgids 10000-75535 ocp
```

Install podman and other required utils

```
dnf clean all
dnf install -y podman jq libguestfs-tools-c
dnf update -y
```

Disable all not needed interfaces in the host to avoid messing networking stuff with containers as well as IPv6 if not needed:

```
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

Switch to the 'ocp' user created before:

```
su - ocp
```

Create an ssh key to be injected in the OCP hosts:

```
ssh-keygen -t rsa -N '' -f ~/.ssh/id_rsa
```

# Variables

Those variables will allow other environments/versions and modifications to where the files are hosted:

```
cat > ~/vars << EOF
# RHCOS and OCP4 versions
# From https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.1/
export RHCOSVERSION="4.1.0"
# From https://mirror.openshift.com/pub/openshift-v4/clients/ocp/latest/
export OCPVERSION="4.1.0"

# Where to store the required files
export NGINX_DIRECTORY="/home/ocp/containers/nginx"
export HAPROXY_DIRECTORY="/home/ocp/containers/haproxy"
export COREDNS_DIRECTORY="/home/ocp/containers/coredns"

# Network details
export DOMAIN_NAME="minwi.lan"
export CLUSTER_NAME="ocp4"
export GATEWAY="192.168.32.1"
export NETMASK="255.255.255.0"
export DNSFORWARDER="8.8.8.8"

# Hosts
export BOOTSTRAP_IP="192.168.32.99"
export MASTER0_IP="192.168.32.100"
export MASTER1_IP="192.168.32.101"
export MASTER2_IP="192.168.32.102"
export WORKER0_IP="192.168.32.200"

# We will use a single interface for the OCP4 cluster network traffic (same one in all hosts)
export NET_INTERFACE="eno2"

# iDRAC details
# Select bios/uefi depending on the hardware
# https://downloads.dell.com/solutions/servers-solution-resources/BootModeWhitepaper.pdf
# 'racadm get bios.BiosBootSettings.BootMode'
export BIOSMODE="bios"
export BOOTSTRAP_IDRAC_IP="192.168.31.99"
export MASTER0_IDRAC_IP="192.168.31.100"
export MASTER1_IDRAC_IP="192.168.31.101"
export MASTER2_IDRAC_IP="192.168.31.102"
export WORKER0_IDRAC_IP="192.168.31.200"
export IDRACUSER="root"
export IDRACPASS="calvin"

# We will use this host as DNS, static assets server and haproxy
export MY_IP="192.168.32.2"
export DNS="\${MY_IP}"
export URL="http://\${MY_IP}:8001"
export LB_IP="\${MY_IP}"

# Required to extract the ISO content with guestfish without any virtualization stuff installed
export LIBGUESTFS_BACKEND=direct

export SSH_KEY=\$(cat ~/.ssh/id_rsa.pub)
# This may not work until the pull_secret is created
export PULL_SECRET=\$(cat ~/ocp-clusters/pull_secret.json)
EOF
```

Then, use the vars file

```
source ~/vars
```

# Haproxy
To quickly spin up an HAProxy server, we will use the HAProxy official container image:

```
mkdir -p ${HAPROXY_DIRECTORY}

cat > ${HAPROXY_DIRECTORY}/haproxy.cfg << EOF
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          300s
    timeout server          300s
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 20000

# Useful for debugging, dangerous for production
listen stats
    bind :9000
    mode http
    stats enable
    stats uri /

frontend openshift-api-server
    bind *:6443
    default_backend openshift-api-server
    mode tcp
    option tcplog

backend openshift-api-server
    balance source
    mode tcp
    server bootstrap ${BOOTSTRAP_IP}:6443 check
    server master-0 ${MASTER0_IP}:6443 check
    server master-1 ${MASTER1_IP}:6443 check
    server master-2 ${MASTER2_IP}:6443 check

frontend machine-config-server
    bind *:22623
    default_backend machine-config-server
    mode tcp
    option tcplog

backend machine-config-server
    balance source
    mode tcp
    server bootstrap ${BOOTSTRAP_IP}:22623 check
    server master-0 ${MASTER0_IP}:22623 check
    server master-1 ${MASTER1_IP}:22623 check
    server master-2 ${MASTER2_IP}:22623 check

# As we are using rootless containers, we will bind to the 8080/tcp port in the host
# so we can map 1:1 (and then firewalld will perform the redirection)
frontend ingress-http
    bind *:8080
    default_backend ingress-http
    mode tcp
    option tcplog

backend ingress-http
    balance source
    mode tcp
    server worker-0 ${WORKER0_IP}:80 check

# As we are using rootless containers, we will bind to the 8443/tcp port in the host
# so we can map 1:1 (and then firewalld will perform the redirection)   
frontend ingress-https
    bind *:8443
    default_backend ingress-https
    mode tcp
    option tcplog

backend ingress-https
    balance source
    mode tcp
    server worker-0 ${WORKER0_IP}:443 check
EOF
```

Then, run the container:

```
podman run -d \
  --expose=9000 --expose=22623 --expose=6443 --expose=8080 --expose=8443 \
  -p 9000:9000 -p 22623:22623 -p 6443:6443 -p 8080:8080 -p 8443:8443 \
  -v ${HAPROXY_DIRECTORY}/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:Z \
  --name haproxy \
  haproxy:alpine
```

Configure firewall to forward 8443 to 443 and 8080 to 80

```
sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" \
  --add-forward-port=port=443:proto=tcp:toport=8443 --permanent
sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" \
  --add-forward-port=port=80:proto=tcp:toport=8080 --permanent
```

Open those ports to the outside world

```
for service in http https
do
  sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" \
    --add-service=${service} --permanent
done

for port in 9000 22623 6443
do
  sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" \
  --add-port=${port}/tcp --permanent
done
```

Reload the firewall

```
sudo firewall-cmd --reload
```

## Systemd user unit
A systemd user unit can be created to automatically start/stop the podman container as a user:

```
# Enable start user unit without being logged first
sudo loginctl enable-linger ocp

# Create the folder and unit file
mkdir -p ~/.config/systemd/user/
cat > ~/.config/systemd/user/haproxy.service << EOF
[Unit]
Description=HAProxy

[Service]
Restart=always
ExecStart=/usr/bin/podman start -a haproxy
ExecStop=/usr/bin/podman stop -t 10 haproxy
KillMode=process

[Install]
WantedBy=multi-user.target
EOF

# Reload the service and enable/start the service
systemctl --user daemon-reload
systemctl --user enable haproxy.service --now
```

# Web server

To quickly spin up a web server, we will use the official NGINX container image:

```
mkdir -p ${NGINX_DIRECTORY}

podman run -d \
  --expose=8001 \
  -p 8001:80 \
  -v ${NGINX_DIRECTORY}:/usr/share/nginx/html:z \
  --name nginx \
  nginx:latest
```

Open the 8001 port to the outside world

```
sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" \
  --add-port=8001/tcp --permanent
```

Reload the firewall:

```
sudo firewall-cmd --reload
```

## Systemd user unit
A systemd user unit can be created to automatically start/stop the podman container as a user:

```
# This shouldn't be needed if has been previously done for the HAProxy pod
# Enable start user unit without being logged first
# sudo loginctl enable-linger ocp

# Create the folder and unit file
mkdir -p ~/.config/systemd/user/
cat > ~/.config/systemd/user/nginx.service << EOF
[Unit]
Description=nginx

[Service]
Restart=always
ExecStart=/usr/bin/podman start -a nginx
ExecStop=/usr/bin/podman stop -t 10 nginx
KillMode=process

[Install]
WantedBy=multi-user.target
EOF

# Reload the service and enable/start the service
systemctl --user daemon-reload
systemctl --user enable nginx.service --now
```

# DNS
To quickly spin up a DNS server, we will use the official CoreDNS container image:

```
mkdir -p ${COREDNS_DIRECTORY}

# CoreDNS configuration file
cat > ${COREDNS_DIRECTORY}/Corefile << EOF
.:53 {
    log
    errors
    forward . ${DNSFORWARDER}
}

${DOMAIN_NAME}:53 {
    log
    errors
    file /etc/coredns/db.${DOMAIN_NAME}
}
EOF

```
Then, the proper zone file, including the SRV records, CNAMES, etc. To avoid escaping dollar symbols, etc. we create the template file first then replace the variables using sed:

```
cat > ${COREDNS_DIRECTORY}/db.${DOMAIN_NAME} << 'EOF'
$ORIGIN DOMAIN_NAME.
$TTL 10800      ; 3 hours
@       3600 IN SOA sns.dns.icann.org. noc.dns.icann.org. (
                                2019010101 ; serial
                                7200       ; refresh (2 hours)
                                3600       ; retry (1 hour)
                                1209600    ; expire (2 weeks)
                                3600       ; minimum (1 hour)
                                )

_etcd-server-ssl._tcp.CLUSTER_NAME.DOMAIN_NAME. 8640 IN    SRV 0 10 2380 etcd-0.CLUSTER_NAME.DOMAIN_NAME.
_etcd-server-ssl._tcp.CLUSTER_NAME.DOMAIN_NAME. 8640 IN    SRV 0 10 2380 etcd-1.CLUSTER_NAME.DOMAIN_NAME.
_etcd-server-ssl._tcp.CLUSTER_NAME.DOMAIN_NAME. 8640 IN    SRV 0 10 2380 etcd-2.CLUSTER_NAME.DOMAIN_NAME.

api.CLUSTER_NAME.DOMAIN_NAME.                        A                LB_IP
api-int.CLUSTER_NAME.DOMAIN_NAME.                    A                LB_IP
CLUSTER_NAME-master-0.DOMAIN_NAME.                   A                MASTER0_IP
CLUSTER_NAME-master-1.DOMAIN_NAME.                   A                MASTER1_IP
CLUSTER_NAME-master-2.DOMAIN_NAME.                   A                MASTER2_IP
CLUSTER_NAME-worker-0.DOMAIN_NAME.                   A                WORKER0_IP
CLUSTER_NAME-bootstrap.DOMAIN_NAME.                  A                BOOTSTRAP_IP
etcd-0.CLUSTER_NAME.DOMAIN_NAME.                     IN  CNAME CLUSTER_NAME-master-0.DOMAIN_NAME.
etcd-1.CLUSTER_NAME.DOMAIN_NAME.                     IN  CNAME CLUSTER_NAME-master-1.DOMAIN_NAME.
etcd-2.CLUSTER_NAME.DOMAIN_NAME.                     IN  CNAME CLUSTER_NAME-master-2.DOMAIN_NAME.

$ORIGIN apps.CLUSTER_NAME.DOMAIN_NAME.
*                                                    A                LB_IP
EOF

sed -i -e "s/MASTER0_IP/${MASTER0_IP}/g" \
       -e "s/MASTER1_IP/${MASTER1_IP}/g" \
       -e "s/MASTER2_IP/${MASTER2_IP}/g" \
       -e "s/WORKER0_IP/${WORKER0_IP}/g" \
       -e "s/CLUSTER_NAME/${CLUSTER_NAME}/g" \
       -e "s/DOMAIN_NAME/${DOMAIN_NAME}/g" \
       -e "s/BOOTSTRAP_IP/${BOOTSTRAP_IP}/g" \
       -e "s/LB_IP/${LB_IP}/g" \
       ${COREDNS_DIRECTORY}/db.${DOMAIN_NAME}
```

I've not been able to query from localhost or $(hostname -I) from the host running the coredns container when running rootless.
`/etc/resolv.conf` doesn't allow specific ports and redirecting localhost/ip is really messy, so, this container runs with sudo (and binds to :53):

```
sudo podman run -d \
  --expose=53 --expose=53/udp \
  -p ${DNS}:53:53 -p ${DNS}:53:53/udp \
  -v ${COREDNS_DIRECTORY}:/etc/coredns:z \
  --name coredns \
  coredns/coredns:latest -conf /etc/coredns/Corefile

sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" --add-service=dns --permanent
sudo firewall-cmd --reload
```

## Systemd unit
A systemd unit can be created to automatically start/stop the podman container. In this case, as we require root, we will create a regular systemd unit:

```
sudo bash -c 'cat > /etc/systemd/system/coredns.service << EOF
[Unit]
Description=CoreDNS

[Service]
Restart=always
ExecStart=/usr/bin/podman start -a coredns
ExecStop=/usr/bin/podman stop -t 10 coredns
KillMode=process

[Install]
WantedBy=multi-user.target
EOF'

# Reload the service and enable the service
systemctl daemon-reload
systemctl enable coredns.service
```

To see if it works:

```
$ sudo podman ps
CONTAINER ID  IMAGE                             COMMAND               CREATED        STATUS            PORTS                                           NAMES
abf77a3da374  docker.io/coredns/coredns:latest  /coredns -conf /e...  9 minutes ago  Up 2 seconds ago  10.19.138.7:53->53/tcp, 10.19.138.7:53->53/udp  coredns

$ sudo systemctl stop coredns.service
$ sudo podman ps
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
```

Then, modify `/etc/resolv.conf` to use the coredns container:

```
sudo nmcli con mod 'System eno1' ipv4.ignore-auto-dns yes
sudo nmcli con mod 'System eno1' ipv4.dns "${DNS}"
sudo systemctl restart NetworkManager
```

# OpenShift binaries
Download and extract the openshift-install and oc/kubectl binaries into the helper node

```
curl -sL https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCPVERSION}/openshift-client-linux-${OCPVERSION}.tar.gz | sudo tar -C /usr/local/bin -xzf - oc kubectl

# Use https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux/oc.tar.gz for latest oc
# it doesn't include kubectl tho

curl -sL https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCPVERSION}/openshift-install-linux-${OCPVERSION}.tar.gz | sudo tar -C /usr/local/bin -xzf - openshift-install

sudo chmod 755 /usr/local/bin/{oc,kubectl,openshift-install}
```

# Cluster files
openshift-install requires some files (pull secret and install-config) and creates some other assets (manifests, ignition files, logs, etc.). In order to have a proper directory structure, they will be stored in ~/ocp-clusters/<cluster-name>:

```
mkdir -p ~/ocp-clusters/${CLUSTER_NAME}/
```

## Pull secret
Visit cloud.openshift.com, download your pull secret and copy it into ~/ocp-clusters/pull_secret.json (to be used by all clusters) as:

```
cat ~/ocp-clusters/pull_secret.json
# pull_secret content...
export PULL_SECRET=$(cat ~/ocp-clusters/pull_secret.json)
```

## install-config.yaml
Instead creating the file directly (it will be removed by openshift-install), it is created with the cluster prefix, then copied to the proper location:

```
cat > ~/ocp-clusters/${CLUSTER_NAME}-install-config.yaml << EOF
apiVersion: v1
baseDomain: ${DOMAIN_NAME}
compute:
- name: worker
  replicas: 0
controlPlane:
  name: master
  replicas: 3
metadata:
  name: ${CLUSTER_NAME}
networking:
  clusterNetworks:
  - cidr: 10.128.0.0/14
    hostPrefix: 23
  networkType: OpenShiftSDN
  serviceNetwork:
  - 172.30.0.0/16
platform:
  none: {}
pullSecret: |
  ${PULL_SECRET}
sshKey: |
  ${SSH_KEY}
EOF

cp ~/ocp-clusters/${CLUSTER_NAME}-install-config.yaml \
   ~/ocp-clusters/${CLUSTER_NAME}/install-config.yaml
```

# Ignition configs
Create the ignition files once the install-config.yaml file has been created:

```
openshift-install create ignition-configs --dir=$(readlink -f ~/ocp-clusters/${CLUSTER_NAME})

# They are going to be modified and be served by the NGINX container
cp ~/ocp-clusters/${CLUSTER_NAME}/*.ign ${NGINX_DIRECTORY}
```

NOTE: The ignition files include certificates that are only valid for 24h

## Ignition configs modifications
In order to be able to set static IP addresses for the hosts, it is required to inject the proper configuration (`/etc/sysconfig/network-scripts/ifcfg-<interface>` & `/etc/hostname`) via ignition:

```
create_ifcfg(){
  cat > ${NGINX_DIRECTORY}/${HOST}-eno2 << EOF
DEVICE=eno2
BOOTPROTO=none
ONBOOT=yes
NETMASK=${NETMASK}
IPADDR=${IP}
GATEWAY=${GATEWAY}
PEERDNS=no
DNS1=${DNS}
IPV6INIT=no
EOF

  ENO2=$(cat ${NGINX_DIRECTORY}/${HOST}-eno2 | base64 -w0)
  rm ${NGINX_DIRECTORY}/${HOST}-eno2

  cat > ${NGINX_DIRECTORY}/${HOST}-ifcfg-eno2.json << EOF
{
  "append" : false,
  "mode" : 420,
  "filesystem" : "root",
  "path" : "/etc/sysconfig/network-scripts/ifcfg-eno2",
  "contents" : {
    "source" : "data:text/plain;charset=utf-8;base64,${ENO2}",
    "verification" : {}
  },
  "user" : {
    "name" : "root"
  },
  "group": {
    "name": "root"
  }
}
EOF

  cat > ${NGINX_DIRECTORY}/${HOST}-eno1 << EOF
DEVICE=eno1
BOOTPROTO=none
ONBOOT=no
EOF
  ENO1=$(cat ${NGINX_DIRECTORY}/${HOST}-eno1 | base64 -w0)
  rm ${NGINX_DIRECTORY}/${HOST}-eno1
  cat > ${NGINX_DIRECTORY}/${HOST}-ifcfg-eno1.json << EOF
{
  "append" : false,
  "mode" : 420,
  "filesystem" : "root",
  "path" : "/etc/sysconfig/network-scripts/ifcfg-eno1",
  "contents" : {
    "source" : "data:text/plain;charset=utf-8;base64,${ENO1}",
    "verification" : {}
  },
  "user" : {
    "name" : "root"
  },
  "group": {
    "name": "root"
  }
}
EOF

cat > ${NGINX_DIRECTORY}/${HOST}-hostname << EOF
${CLUSTER_NAME}-${HOST}.${DOMAIN_NAME}
EOF
  HN=$(cat ${NGINX_DIRECTORY}/${HOST}-hostname | base64 -w0)
  rm ${NGINX_DIRECTORY}/${HOST}-hostname
  cat > ${NGINX_DIRECTORY}/${HOST}-hostname.json << EOF
{
  "append" : false,
  "mode" : 420,
  "filesystem" : "root",
  "path" : "/etc/hostname",
  "contents" : {
    "source" : "data:text/plain;charset=utf-8;base64,${HN}",
    "verification" : {}
  },
  "user" : {
    "name" : "root"
  },
  "group": {
    "name": "root"
  }
}
EOF
}

# Disable set hostname via reverse lookup
# Common to all hosts
cat > ${NGINX_DIRECTORY}/hostname-mode << EOF
[main]
hostname-mode=none
EOF
  HM=$(cat ${NGINX_DIRECTORY}/hostname-mode | base64 -w0)
  rm ${NGINX_DIRECTORY}/hostname-mode
  cat > ${NGINX_DIRECTORY}/hostname-mode.json << EOF
{
  "append" : false,
  "mode" : 420,
  "filesystem" : "root",
  "path" : "/etc/NetworkManager/conf.d/hostname-mode.conf",
  "contents" : {
    "source" : "data:text/plain;charset=utf-8;base64,${HM}",
    "verification" : {}
  },
  "user" : {
    "name" : "root"
  },
  "group": {
    "name": "root"
  }
}
EOF

modify_ignition(){
  cp ${NGINX_DIRECTORY}/${TYPE}.ign ${NGINX_DIRECTORY}/${HOST}.ign.orig
  jq '.storage.files += [input]' ${NGINX_DIRECTORY}/${HOST}.ign.orig ${NGINX_DIRECTORY}/${HOST}-hostname.json > ${NGINX_DIRECTORY}/${HOST}.ign.tmp
  jq '.storage.files += [input]' ${NGINX_DIRECTORY}/${HOST}.ign.tmp ${NGINX_DIRECTORY}/${HOST}-ifcfg-eno1.json > ${NGINX_DIRECTORY}/${HOST}.ign.new
  jq '.storage.files += [input]' ${NGINX_DIRECTORY}/${HOST}.ign.new ${NGINX_DIRECTORY}/${HOST}-ifcfg-eno2.json > ${NGINX_DIRECTORY}/${HOST}.ign.tmp
  jq '.storage.files += [input]' ${NGINX_DIRECTORY}/${HOST}.ign.tmp ${NGINX_DIRECTORY}/hostname-mode.json > ${NGINX_DIRECTORY}/${HOST}.ign
  rm -f ${NGINX_DIRECTORY}/${HOST}.ign.new ${NGINX_DIRECTORY}/${HOST}.ign.tmp ${NGINX_DIRECTORY}/${HOST}-hostname.json ${NGINX_DIRECTORY}/${HOST}-ifcfg-eno1.json ${NGINX_DIRECTORY}/${HOST}-ifcfg-eno2.json
}

HOST="bootstrap"
TYPE="bootstrap"
IP=${BOOTSTRAP_IP}
create_ifcfg
modify_ignition

TYPE="master"
HOST=master-0
IP=${MASTER0_IP}
create_ifcfg
modify_ignition

HOST=master-1
IP=${MASTER1_IP}
create_ifcfg
modify_ignition

HOST=master-2
IP=${MASTER2_IP}
create_ifcfg
modify_ignition

TYPE="worker"
HOST=worker-0
IP=${WORKER0_IP}
create_ifcfg
modify_ignition
```

NOTE: I'm 100% sure this whole code block can be improved… any suggestions appreciated :)

# RHCOS assets
Download the RHCOS iso, BIOS and UEFI image files:

```
for asset in 'installer.iso' 'metal-bios.raw.gz' 'metal-uefi.raw.gz'; do
  curl -J -L https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.1/${OCPVERSION}/rhcos-${RHCOSVERSION}-x86_64-${asset} -o ${NGINX_DIRECTORY}/rhcos-${RHCOSVERSION}-x86_64-${asset}
done
```

# Modify iso files
It is required to add some parameters to the kernel line for the RHCOS installation:
* `coreos.inst=yes`
* `coreos.inst.install_dev=sda`
To set the image and ignition file location:
* `coreos.inst.image_url=<bare_metal_image_URL>`
* `coreos.inst.ignition_url=http://example.com/config.ign`
To provide the hosts fixed IPs it is required to specify the IP address as:
* `ip=<ip>::<gateway>:<netmask>:<hostname>:<interface>:none`
To specify the DNS nameserver IP:
* `nameserver=<nameserver_ip>`

Instead of doing it manually, different isos will be created (for bootstrap, masters and nodes) as:

```
export VOLID=$(isoinfo -d -i ${NGINX_DIRECTORY}/rhcos-${RHCOSVERSION}-x86_64-installer.iso | awk '/Volume id/ { print $3 }')
TEMPDIR=$(mktemp -d)

cd ${TEMPDIR}
# Extract the ISO content using guestfish (to avoid sudo mount)
guestfish -a ${NGINX_DIRECTORY}/rhcos-${RHCOSVERSION}-x86_64-installer.iso \
  -m /dev/sda tar-out / - | tar xvf -

# Helper function to modify the config files
modify_cfg(){
  for file in "EFI/fedora/grub.cfg" "isolinux/isolinux.cfg"; do
    # Append the proper image and ignition urls
    sed -e '/coreos.inst=yes/s|$| coreos.inst.install_dev=sda coreos.inst.image_url='"${URL}"'\/rhcos-'"${RHCOSVERSION}"'-x86_64-metal-'"${BIOSMODE}"'.raw.gz coreos.inst.ignition_url='"${URL}"'\/'"${NODE}"'.ign ip='"${IP}"'::'"${GATEWAY}"':'"${NETMASK}"':'"${FQDN}"':'"${NET_INTERFACE}"':none nameserver='"${DNS}"'|' ${file} > $(pwd)/${NODE}_${file##*/}
    # Boot directly in the installation
    sed -i -e 's/default vesamenu.c32/default linux/g' -e 's/timeout 600/timeout 10/g' $(pwd)/${NODE}_${file##*/}
  done
}

# BOOTSTRAP
TYPE="bootstrap"
NODE="bootstrap"
IP=${BOOTSTRAP_IP}
FQDN="${CLUSTER_NAME}-bootstrap.${DOMAIN_NAME}"
modify_cfg

# MASTERS
TYPE="master"
# MASTER-0
NODE="master-0"
IP=${MASTER0_IP}
FQDN="${CLUSTER_NAME}-${NODE}.${DOMAIN_NAME}"
modify_cfg

# MASTER-1
NODE="master-1"
IP=${MASTER0_IP}
FQDN="${CLUSTER_NAME}-${NODE}.${DOMAIN_NAME}"
modify_cfg

# MASTER-2
NODE="master-2"
IP=${MASTER0_IP}
FQDN="${CLUSTER_NAME}-${NODE}.${DOMAIN_NAME}"
modify_cfg

# WORKERS
TYPE="worker"
# WORKER-0
NODE="worker-0"
IP=${WORKER0_IP}
FQDN="${CLUSTER_NAME}-${NODE}.${DOMAIN_NAME}"
modify_cfg

# Generate the images, one per node as the IP configuration is different...
# https://github.com/coreos/coreos-assembler/blob/master/src/cmd-buildextend-installer#L97-L103
for node in master-0 master-1 master-2 worker-0 bootstrap; do
  # Overwrite the grub.cfg and isolinux.cfg files for each node type
  for file in "EFI/fedora/grub.cfg" "isolinux/isolinux.cfg"; do
    cp $(pwd)/${node}_${file##*/} ${file}
  done
  # As regular user!
  genisoimage -verbose -rock -J -joliet-long -volset ${VOLID} \
    -eltorito-boot isolinux/isolinux.bin -eltorito-catalog isolinux/boot.cat \
    -no-emul-boot -boot-load-size 4 -boot-info-table \
    -eltorito-alt-boot -efi-boot images/efiboot.img -no-emul-boot \
    -o ${NGINX_DIRECTORY}/${node}.iso .
done

# Optionally, clean up
# cd
# rm -Rf ${TEMPDIR}
```

# oc command
Configure 'oc' to be system:admin as:

```
mkdir -p ~/.kube/
cp ~/ocp-clusters/${CLUSTER_NAME}/auth/kubeconfig ~/.kube/config
```

# Installation
With everything prepared, it is time to install the baremetal servers. If the previous steps have been doing properly, the ISOs are modified to avoid requiring manual input and the ignition files shall configure the networking to use the DNS we have set.
NOTE: In my environment, it is required to have an iDRAC version >= 2.60.60.60 to be able to map virtual media iso from http.

We will be using a containerized 'racadm' command to avoid installing it locally. The source of this container image can be seen in https://github.com/e-minguez/racadm-container

## Install bootstrap

```
# Note: racadm commands will complain about certificates
podman pull quay.io/eminguez/container-racadm

# Bootstrap
export IDRACIP=${BOOTSTRAP_IDRAC_IP}
alias racadm='podman run --rm eminguez/container-racadm -r ${IDRACIP} -u ${IDRACUSER} -p ${IDRACPASS}'
racadm remoteimage -d
racadm remoteimage -c -u "foo" -p "bar" -l ${URL}/bootstrap.iso
racadm jobqueue delete -i JID_CLEARALL
racadm set BIOS.OneTimeBoot.OneTimeBootMode OneTimeBootSeq
racadm set BIOS.OneTimeBoot.OneTimeBootSeqDev Optical.iDRACVirtual.1-1
racadm jobqueue create BIOS.Setup.1-1 -r pwrcycle
```

## Install hosts

```
install_host(){
  alias racadm='podman run --rm eminguez/container-racadm -r ${IDRACIP} -u ${IDRACUSER} -p ${IDRACPASS}'
  racadm remoteimage -d
  racadm remoteimage -c -u "foo" -p "bar" -l ${URL}/${NODE}.iso
  racadm jobqueue delete -i JID_CLEARALL
  racadm set BIOS.OneTimeBoot.OneTimeBootMode OneTimeBootSeq
  racadm set BIOS.OneTimeBoot.OneTimeBootSeqDev Optical.iDRACVirtual.1-1
  racadm jobqueue create BIOS.Setup.1-1 -r pwrcycle
}

# Masters
export IDRACIP=${MASTER0_IDRAC_IP}
export NODE='master-0'
install_host

export IDRACIP=${MASTER1_IDRAC_IP}
export NODE='master-1'
install_host

export IDRACIP=${MASTER2_IDRAC_IP}
export NODE='master-2'
install_host

# Workers
export IDRACIP=${WORKER0_IDRAC_IP}
export NODE='worker-0'
install_host
```

After a while, the hosts will be running RHCOS, then wait until the bootstrap process ends:

```
openshift-install --dir=$(readlink -f ~/ocp-clusters/${CLUSTER_NAME}) --log-level debug \
  wait-for bootstrap-complete
```

Then, wait until the install is complete:

```
openshift-install --dir=$(readlink -f ~/ocp-clusters/${CLUSTER_NAME}) --log-level debug \
  wait-for install-complete
```

NOTE: The installation won't finish until the image registry has been deployed. As there are no storage available, we will use emptydir (not recommended for production!!!):

```
oc patch configs.imageregistry.operator.openshift.io cluster --type merge --patch '{"spec":{"storage":{"emptyDir":{}}}}'
```

# Post-installation
* Remove bootstrap from the load balancer:

```
cp ${HAPROXY_DIRECTORY}/haproxy.cfg{,.orig}
sed -i -e '/server bootstrap/d' ${HAPROXY_DIRECTORY}/haproxy.cfg
systemctl --user restart haproxy
```

* As the environment has a single worker, scale down the router replica to 1 pod:

```
oc patch \
   --namespace=openshift-ingress-operator \
   --patch='{"spec": {"replicas": 1}}' \
   --type=merge \
   ingresscontroller/default
```

* Configure authentication backend (htpasswd)
The following script will create an "admin" user with password "admin" with cluster-admin role:

```
user=admin
password=admin
htpasswd=$(printf "$user:$(openssl passwd -apr1 $password)\n")
htpasswd=$(echo $htpasswd | base64)

oc apply -f - <<EOF
apiVersion: v1
kind: Secret
metadata:
  name: htpass-secret
  namespace: openshift-config
data:
  htpasswd: $htpasswd
EOF

# configure HTPasswd IDP
oc apply -f - <<EOF
apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: htpassidp
    challenge: true
    login: true
    mappingMethod: claim
    type: HTPasswd
    htpasswd:
      fileData:
        name: htpass-secret
EOF

oc adm policy add-cluster-role-to-user cluster-admin admin
```

# Upgrade

Upgrade to the latest bits:

```
oc adm upgrade --to-latest
```

If wanted to switch to any other channel (such as prerelease-4.1):

```

oc patch \
   --patch='{"spec": {"channel": "prerelease-4.1"}}' \
   --type=merge \
   clusterversion/version
```

In order to force the update to a specific version/hash, first, get the hash of the image version of the release to upgrade to:

```
curl -sH 'Accept: application/json' 'https://api.openshift.com/api/upgrades_info/v1/graph?channel=prerelease-4.1' | jq .
```

Then force apply the update:

```
# RC.9 sha256 = 49c4b6bf70061e522e3525aed534d087c9abfba7c39cbcbdd1bd770ab096bf9e
oc adm upgrade --force=true \
--to-image=quay.io/openshift-release-dev/ocp-release@sha256:49c4b6bf70061e522e3525aed534d087c9abfba7c39cbcbdd1bd770ab096bf9e
```

# Verification

```
$ oc get nodes
NAME                      STATUS   ROLES    AGE   VERSION
ocp4-master-0.minwi.lan   Ready    master   22m   v1.13.4+cb455d664
ocp4-master-1.minwi.lan   Ready    master   22m   v1.13.4+cb455d664
ocp4-master-2.minwi.lan   Ready    master   22m   v1.13.4+cb455d664
ocp4-worker-0.minwi.lan   Ready    worker   23m   v1.13.4+cb455d664

$ oc get clusterversion
NAME      VERSION   AVAILABLE   PROGRESSING   SINCE   STATUS
version   4.1.0     True        False         11s     Cluster version is 4.1.0

$ oc get clusteroperators
NAME                                 VERSION   AVAILABLE   PROGRESSING   DEGRADED   SINCE
authentication                       4.1.0     True        False         False      49s
cloud-credential                     4.1.0     True        False         False      21m
cluster-autoscaler                   4.1.0     True        False         False      21m
console                              4.1.0     True        False         False      5m16s
dns                                  4.1.0     True        False         False      21m
image-registry                       4.1.0     True        False         False      15m
ingress                              4.1.0     True        False         False      16m
kube-apiserver                       4.1.0     True        False         False      20m
kube-controller-manager              4.1.0     True        False         False      19m
kube-scheduler                       4.1.0     True        False         False      19m
machine-api                          4.1.0     True        False         False      21m
machine-config                       4.1.0     True        False         False      20m
marketplace                          4.1.0     True        False         False      15m
monitoring                           4.1.0     True        False         False      14m
network                              4.1.0     True        False         False      21m
node-tuning                          4.1.0     True        False         False      19m
openshift-apiserver                  4.1.0     True        False         False      18m
openshift-controller-manager         4.1.0     True        False         False      21m
openshift-samples                    4.1.0     True        False         False      10m
operator-lifecycle-manager           4.1.0     True        False         False      21m
operator-lifecycle-manager-catalog   4.1.0     True        False         False      21m
service-ca                           4.1.0     True        False         False      21m
service-catalog-apiserver            4.1.0     True        False         False      19m
service-catalog-controller-manager   4.1.0     True        False         False      19m
storage                              4.1.0     True        False         False      16m

$ oc get pods --all-namespaces | grep -v -E 'Running|Completed'
NAMESPACE                                               NAME                                                              READY   STATUS      RESTARTS   AGE
$ oc get pods --all-namespaces -o name | wc -l
162
```
