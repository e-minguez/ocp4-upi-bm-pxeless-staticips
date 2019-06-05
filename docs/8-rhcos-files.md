# RHCOS files
Download the RHCOS iso, BIOS and UEFI image files:

```
for asset in 'installer.iso' 'metal-bios.raw.gz' 'metal-uefi.raw.gz'; do
  curl -J -L https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.1/${OCPVERSION}/rhcos-${RHCOSVERSION}-x86_64-${asset} -o ${NGINX_DIRECTORY}/rhcos-${RHCOSVERSION}-x86_64-${asset}
done
```
