# RHCOS files

Download the RHCOS ISO, BIOS and UEFI image files:

```bash
for asset in 'installer.iso' 'metal-bios.raw.gz' 'metal-uefi.raw.gz'; do
  curl -J -L https://mirror.openshift.com/pub/openshift-v4/dependencies/rhcos/4.1/${OCPVERSION}/rhcos-${RHCOSVERSION}-x86_64-${asset} \
  -o ${NGINX_DIRECTORY}/rhcos-${RHCOSVERSION}-x86_64-${asset}
done
```

[<< Previous: Ignition files](7-ignition-files.md) | [README](../README.md) | [Next: Modify ISOs >>](9-modify-isos.md)
