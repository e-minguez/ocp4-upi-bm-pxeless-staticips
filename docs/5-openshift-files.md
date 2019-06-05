# OpenShift files
Download and extract the `openshift-install` `and oc/kubectl` binaries into the helper node

```
curl -sL https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCPVERSION}/openshift-client-linux-${OCPVERSION}.tar.gz | sudo tar -C /usr/local/bin -xzf - oc kubectl

# Use https://mirror.openshift.com/pub/openshift-v4/clients/oc/latest/linux/oc.tar.gz for latest oc
# it doesn't include kubectl tho

curl -sL https://mirror.openshift.com/pub/openshift-v4/clients/ocp/${OCPVERSION}/openshift-install-linux-${OCPVERSION}.tar.gz | sudo tar -C /usr/local/bin -xzf - openshift-install

sudo chmod 755 /usr/local/bin/{oc,kubectl,openshift-install}
```
