# Installation
With everything prepared, it is time to install the baremetal servers. If the previous steps have been doing properly, the ISOs are modified to avoid requiring manual input and the ignition files shall configure the networking to use the DNS we have set.
NOTE: In my environment, it is required to have an iDRAC version >= 2.60.60.60 to be able to map virtual media iso from http.

We will be using a containerized `racadm` command to avoid installing it locally. The source of this container image can be seen in https://github.com/e-minguez/racadm-container

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

[<< Previous: Configure `oc` command](10-configure-oc-command.md) | [Next: Post installation >>](12-post-installation.md)
