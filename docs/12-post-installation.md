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

[<< Previous: Installation](11-installation.md) | [Next: Upgrade >>](13-upgrade.md)
