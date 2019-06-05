# Cluster files
openshift-install requires some files (pull secret and install-config) and creates some other assets (manifests, ignition files, logs, etc.). In order to have a proper directory structure, they will be stored in ~/ocp-clusters/<cluster-name>:

```
mkdir -p ~/ocp-clusters/${CLUSTER_NAME}/
```

## Pull secret
Visit cloud.openshift.com, download your pull secret and copy it into `~/ocp-clusters/pull_secret.json` (to be used by all clusters) as:

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

[<< Previous: OpenShift files](5-openshift-files.md) | [Next: Ignition files >>](7-ignition-files.md)
