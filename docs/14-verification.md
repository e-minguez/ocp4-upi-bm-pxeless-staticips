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
