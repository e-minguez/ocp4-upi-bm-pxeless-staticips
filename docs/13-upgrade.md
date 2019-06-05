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

[<< Previous: Post Installation](12-post-installation.md) | [Next: Verification >>](14-verification.md)
