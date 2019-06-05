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

[<< Previous: Prerequisites](0-prerequisites.md) | [Next: Load balancer >>](2-load-balancer.md)
