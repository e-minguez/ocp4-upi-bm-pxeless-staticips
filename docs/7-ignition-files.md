# Ignition files
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
  cp -u ${NGINX_DIRECTORY}/${TYPE}.ign ${NGINX_DIRECTORY}/${HOST}.ign.orig
  jq '.storage.files += [inputs]' ${NGINX_DIRECTORY}/${HOST}.ign.orig ${NGINX_DIRECTORY}/${HOST}-hostname.json ${HOST}-ifcfg-eno1.json ${HOST}-ifcfg-eno2.json ${NGINX_DIRECTORY}/hostname-mode.json > ${NGINX_DIRECTORY}/${HOST}.ign
  rm -f ${NGINX_DIRECTORY}/${HOST}-hostname.json ${HOST}-ifcfg-eno1.json ${HOST}-ifcfg-eno2.json
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

[<< Previous: Cluster files](6-cluster-files.md) | [README](../README.md) | [Next: RHCOS files >>](8-rhcos-files.md)
