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

$ oc get pods --all-namespaces -o wide
NAMESPACE                                               NAME                                                              READY   STATUS      RESTARTS   AGE     IP             NODE                      NOMINATED NODE   READINESS GATES
openshift-apiserver-operator                            openshift-apiserver-operator-85cb746d55-kvjgn                     1/1     Running     1          21h     10.129.0.4     ocp4-master-2.minwi.lan   <none>           <none>
openshift-apiserver                                     apiserver-6ks2j                                                   1/1     Running     0          21h     10.131.0.27    ocp4-master-1.minwi.lan   <none>           <none>
openshift-apiserver                                     apiserver-hcrvg                                                   1/1     Running     0          21h     10.130.0.28    ocp4-master-0.minwi.lan   <none>           <none>
openshift-apiserver                                     apiserver-sqbk8                                                   1/1     Running     0          21h     10.129.0.45    ocp4-master-2.minwi.lan   <none>           <none>
openshift-authentication-operator                       authentication-operator-69d5d8bf84-lcbrp                          1/1     Running     0          21h     10.129.0.51    ocp4-master-2.minwi.lan   <none>           <none>
openshift-authentication                                oauth-openshift-7bb77f879-8sxvf                                   1/1     Running     0          21h     10.130.0.35    ocp4-master-0.minwi.lan   <none>           <none>
openshift-authentication                                oauth-openshift-7bb77f879-nslpl                                   1/1     Running     0          21h     10.129.0.53    ocp4-master-2.minwi.lan   <none>           <none>
openshift-cloud-credential-operator                     cloud-credential-operator-74b9b4bff6-b6cjj                        1/1     Running     0          21h     10.129.0.11    ocp4-master-2.minwi.lan   <none>           <none>
openshift-cluster-machine-approver                      machine-approver-7cd7f97455-8wk8p                                 1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-cluster-node-tuning-operator                  cluster-node-tuning-operator-8598b6c957-89pww                     1/1     Running     0          21h     10.129.0.32    ocp4-master-2.minwi.lan   <none>           <none>
openshift-cluster-node-tuning-operator                  tuned-2vcqf                                                       1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-cluster-node-tuning-operator                  tuned-mf8mf                                                       1/1     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-cluster-node-tuning-operator                  tuned-vsqvq                                                       1/1     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-cluster-node-tuning-operator                  tuned-xlwzr                                                       1/1     Running     0          21h     192.168.32.200    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-cluster-samples-operator                      cluster-samples-operator-6b48ccf677-zj79s                         1/1     Running     0          21h     10.131.0.21    ocp4-master-1.minwi.lan   <none>           <none>
openshift-cluster-storage-operator                      cluster-storage-operator-868dbc4698-qpjdg                         1/1     Running     0          21h     10.131.0.20    ocp4-master-1.minwi.lan   <none>           <none>
openshift-cluster-version                               cluster-version-operator-6f8fc78789-k4ln8                         1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-console-operator                              console-operator-f6d5f6d4f-2r66h                                  1/1     Running     0          21h     10.131.0.35    ocp4-master-1.minwi.lan   <none>           <none>
openshift-console                                       console-7996445b88-pwx8f                                          1/1     Running     0          21h     10.129.0.50    ocp4-master-2.minwi.lan   <none>           <none>
openshift-console                                       console-7996445b88-tks4n                                          1/1     Running     0          21h     10.130.0.34    ocp4-master-0.minwi.lan   <none>           <none>
openshift-console                                       downloads-65877c7d-65rfw                                          1/1     Running     0          21h     10.131.0.17    ocp4-master-1.minwi.lan   <none>           <none>
openshift-console                                       downloads-65877c7d-6vx2p                                          1/1     Running     0          21h     10.130.0.15    ocp4-master-0.minwi.lan   <none>           <none>
openshift-controller-manager-operator                   openshift-controller-manager-operator-7d7b899bdf-xlgcp            1/1     Running     1          21h     10.129.0.12    ocp4-master-2.minwi.lan   <none>           <none>
openshift-controller-manager                            controller-manager-bwzj8                                          1/1     Running     0          3h23m   10.130.0.41    ocp4-master-0.minwi.lan   <none>           <none>
openshift-controller-manager                            controller-manager-gnl6m                                          1/1     Running     0          3h22m   10.131.0.45    ocp4-master-1.minwi.lan   <none>           <none>
openshift-controller-manager                            controller-manager-scgqr                                          1/1     Running     0          3h24m   10.129.0.59    ocp4-master-2.minwi.lan   <none>           <none>
openshift-dns-operator                                  dns-operator-7f54c7fd95-pzr2g                                     1/1     Running     0          21h     10.131.0.12    ocp4-master-1.minwi.lan   <none>           <none>
openshift-dns                                           dns-default-f8wlt                                                 2/2     Running     0          21h     10.131.0.2     ocp4-master-1.minwi.lan   <none>           <none>
openshift-dns                                           dns-default-k9nhs                                                 2/2     Running     0          21h     10.128.0.2     ocp4-worker-0.minwi.lan   <none>           <none>
openshift-dns                                           dns-default-rglzx                                                 2/2     Running     0          21h     10.129.0.13    ocp4-master-2.minwi.lan   <none>           <none>
openshift-dns                                           dns-default-twbfk                                                 2/2     Running     0          21h     10.130.0.2     ocp4-master-0.minwi.lan   <none>           <none>
openshift-etcd                                          etcd-member-ocp4-master-0.minwi.lan                               2/2     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-etcd                                          etcd-member-ocp4-master-1.minwi.lan                               2/2     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-etcd                                          etcd-member-ocp4-master-2.minwi.lan                               2/2     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-image-registry                                cluster-image-registry-operator-5fc86678cf-sw567                  1/1     Running     0          21h     10.129.0.37    ocp4-master-2.minwi.lan   <none>           <none>
openshift-image-registry                                image-registry-684768fc9c-6ppcg                                   1/1     Running     0          21h     10.128.0.10    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-image-registry                                node-ca-7cr2d                                                     1/1     Running     0          21h     10.129.0.43    ocp4-master-2.minwi.lan   <none>           <none>
openshift-image-registry                                node-ca-bjb26                                                     1/1     Running     0          21h     10.131.0.24    ocp4-master-1.minwi.lan   <none>           <none>
openshift-image-registry                                node-ca-pfkxk                                                     1/1     Running     0          21h     10.130.0.22    ocp4-master-0.minwi.lan   <none>           <none>
openshift-image-registry                                node-ca-qdb92                                                     1/1     Running     0          21h     10.128.0.11    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-ingress-operator                              ingress-operator-7694cfbdb7-xh2wj                                 1/1     Running     0          21h     10.131.0.19    ocp4-master-1.minwi.lan   <none>           <none>
openshift-ingress                                       router-default-6c5cf4dccc-lnxtd                                   1/1     Running     0          21h     192.168.32.200    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-kube-apiserver-operator                       kube-apiserver-operator-7d8b4bd84-pw2xf                           1/1     Running     1          21h     10.129.0.6     ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-2-ocp4-master-0.minwi.lan                               0/1     Completed   0          21h     10.130.0.8     ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-2-ocp4-master-1.minwi.lan                               0/1     Completed   0          21h     10.131.0.5     ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-2-ocp4-master-2.minwi.lan                               0/1     Completed   0          21h     10.129.0.29    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-3-ocp4-master-0.minwi.lan                               0/1     Completed   0          21h     10.130.0.11    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-3-ocp4-master-1.minwi.lan                               0/1     Completed   0          21h     10.131.0.18    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-6-ocp4-master-0.minwi.lan                               0/1     Completed   0          21h     10.130.0.29    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-6-ocp4-master-1.minwi.lan                               0/1     Completed   0          21h     10.131.0.26    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-6-ocp4-master-2.minwi.lan                               0/1     Completed   0          21h     10.129.0.46    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-7-ocp4-master-0.minwi.lan                               0/1     Completed   0          3h35m   10.130.0.37    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-7-ocp4-master-1.minwi.lan                               0/1     Completed   0          3h39m   10.131.0.37    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-7-ocp4-master-2.minwi.lan                               0/1     Completed   0          3h37m   10.129.0.55    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-8-ocp4-master-0.minwi.lan                               0/1     Completed   0          3h21m   10.130.0.42    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-8-ocp4-master-1.minwi.lan                               0/1     Completed   0          3h25m   10.131.0.40    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                installer-8-ocp4-master-2.minwi.lan                               0/1     Completed   0          3h23m   10.129.0.61    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-apiserver                                kube-apiserver-ocp4-master-0.minwi.lan                            2/2     Running     0          3h21m   192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                kube-apiserver-ocp4-master-1.minwi.lan                            2/2     Running     0          3h25m   192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                kube-apiserver-ocp4-master-2.minwi.lan                            2/2     Running     0          3h23m   192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-2-ocp4-master-0.minwi.lan                         0/1     Completed   0          21h     10.130.0.12    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-2-ocp4-master-1.minwi.lan                         0/1     Completed   0          21h     10.131.0.13    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-2-ocp4-master-2.minwi.lan                         0/1     Completed   0          21h     10.129.0.36    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-3-ocp4-master-0.minwi.lan                         0/1     Completed   0          21h     10.130.0.17    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-3-ocp4-master-1.minwi.lan                         0/1     Completed   0          21h     10.131.0.25    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-6-ocp4-master-0.minwi.lan                         0/1     Completed   0          21h     10.130.0.32    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-6-ocp4-master-1.minwi.lan                         0/1     Completed   0          21h     10.131.0.30    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-6-ocp4-master-2.minwi.lan                         0/1     Completed   0          21h     10.129.0.48    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-7-ocp4-master-0.minwi.lan                         0/1     Completed   0          3h33m   10.130.0.38    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-7-ocp4-master-1.minwi.lan                         0/1     Completed   0          3h37m   10.131.0.38    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-7-ocp4-master-2.minwi.lan                         0/1     Completed   0          3h35m   10.129.0.56    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-8-ocp4-master-0.minwi.lan                         0/1     Completed   0          3h19m   10.130.0.43    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-8-ocp4-master-1.minwi.lan                         0/1     Completed   0          3h23m   10.131.0.43    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-apiserver                                revision-pruner-8-ocp4-master-2.minwi.lan                         0/1     Completed   0          3h21m   10.129.0.62    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-controller-manager-operator              kube-controller-manager-operator-7f585f879c-87ctc                 1/1     Running     1          21h     10.129.0.7     ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       installer-2-ocp4-master-2.minwi.lan                               0/1     Completed   0          21h     10.129.0.21    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       installer-3-ocp4-master-0.minwi.lan                               0/1     Completed   0          21h     10.130.0.7     ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       installer-3-ocp4-master-1.minwi.lan                               0/1     Completed   0          21h     10.131.0.14    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       installer-3-ocp4-master-2.minwi.lan                               0/1     Completed   0          21h     10.129.0.26    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       installer-4-ocp4-master-0.minwi.lan                               0/1     Completed   0          21h     10.130.0.24    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       installer-4-ocp4-master-1.minwi.lan                               0/1     Completed   0          21h     10.131.0.29    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       installer-4-ocp4-master-2.minwi.lan                               0/1     Completed   0          21h     10.129.0.39    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       installer-5-ocp4-master-0.minwi.lan                               0/1     Completed   0          3h24m   10.130.0.39    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       installer-5-ocp4-master-1.minwi.lan                               0/1     Completed   0          3h24m   10.131.0.41    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       installer-5-ocp4-master-2.minwi.lan                               0/1     Completed   0          3h25m   10.129.0.57    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       kube-controller-manager-ocp4-master-0.minwi.lan                   2/2     Running     0          3h24m   192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       kube-controller-manager-ocp4-master-1.minwi.lan                   2/2     Running     0          3h24m   192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       kube-controller-manager-ocp4-master-2.minwi.lan                   2/2     Running     0          3h25m   192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       revision-pruner-2-ocp4-master-2.minwi.lan                         0/1     Completed   0          21h     10.129.0.27    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       revision-pruner-3-ocp4-master-0.minwi.lan                         0/1     Completed   0          21h     10.130.0.13    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       revision-pruner-3-ocp4-master-1.minwi.lan                         0/1     Completed   0          21h     10.131.0.15    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       revision-pruner-3-ocp4-master-2.minwi.lan                         0/1     Completed   0          21h     10.129.0.34    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       revision-pruner-4-ocp4-master-0.minwi.lan                         0/1     Completed   0          21h     10.130.0.26    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       revision-pruner-4-ocp4-master-1.minwi.lan                         0/1     Completed   0          21h     10.131.0.32    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       revision-pruner-4-ocp4-master-2.minwi.lan                         0/1     Completed   0          21h     10.129.0.42    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       revision-pruner-5-ocp4-master-0.minwi.lan                         0/1     Completed   0          3h24m   10.130.0.40    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       revision-pruner-5-ocp4-master-1.minwi.lan                         0/1     Completed   0          3h23m   10.131.0.44    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-controller-manager                       revision-pruner-5-ocp4-master-2.minwi.lan                         0/1     Completed   0          3h24m   10.129.0.58    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-scheduler-operator                       openshift-kube-scheduler-operator-6dc6d5c469-48tzl                1/1     Running     1          21h     10.129.0.8     ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-scheduler                                installer-2-ocp4-master-2.minwi.lan                               0/1     Completed   0          21h     10.129.0.17    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-scheduler                                installer-3-ocp4-master-2.minwi.lan                               0/1     Completed   0          21h     10.129.0.22    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-scheduler                                installer-4-ocp4-master-0.minwi.lan                               0/1     Completed   0          21h     10.130.0.9     ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-scheduler                                installer-4-ocp4-master-1.minwi.lan                               0/1     Completed   0          21h     10.131.0.16    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-scheduler                                installer-4-ocp4-master-2.minwi.lan                               0/1     Completed   0          21h     10.129.0.25    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-scheduler                                installer-5-ocp4-master-0.minwi.lan                               0/1     Completed   0          21h     10.130.0.25    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-scheduler                                installer-5-ocp4-master-1.minwi.lan                               0/1     Completed   0          21h     10.131.0.31    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-scheduler                                installer-5-ocp4-master-2.minwi.lan                               0/1     Completed   0          21h     10.129.0.41    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-scheduler                                openshift-kube-scheduler-ocp4-master-0.minwi.lan                  1/1     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-scheduler                                openshift-kube-scheduler-ocp4-master-1.minwi.lan                  1/1     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-scheduler                                openshift-kube-scheduler-ocp4-master-2.minwi.lan                  1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-scheduler                                revision-pruner-2-ocp4-master-2.minwi.lan                         0/1     Completed   0          21h     10.129.0.20    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-scheduler                                revision-pruner-3-ocp4-master-2.minwi.lan                         0/1     Completed   0          21h     10.129.0.24    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-scheduler                                revision-pruner-4-ocp4-master-0.minwi.lan                         0/1     Completed   0          21h     10.130.0.14    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-scheduler                                revision-pruner-4-ocp4-master-1.minwi.lan                         0/1     Completed   0          21h     10.131.0.22    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-scheduler                                revision-pruner-4-ocp4-master-2.minwi.lan                         0/1     Completed   0          21h     10.129.0.35    ocp4-master-2.minwi.lan   <none>           <none>
openshift-kube-scheduler                                revision-pruner-5-ocp4-master-0.minwi.lan                         0/1     Completed   0          21h     10.130.0.27    ocp4-master-0.minwi.lan   <none>           <none>
openshift-kube-scheduler                                revision-pruner-5-ocp4-master-1.minwi.lan                         0/1     Completed   0          21h     10.131.0.33    ocp4-master-1.minwi.lan   <none>           <none>
openshift-kube-scheduler                                revision-pruner-5-ocp4-master-2.minwi.lan                         0/1     Completed   0          21h     10.129.0.44    ocp4-master-2.minwi.lan   <none>           <none>
openshift-machine-api                                   cluster-autoscaler-operator-65cbb65f5c-99g8k                      1/1     Running     0          21h     10.129.0.3     ocp4-master-2.minwi.lan   <none>           <none>
openshift-machine-api                                   machine-api-operator-54bf977cc8-679r9                             1/1     Running     0          21h     10.129.0.9     ocp4-master-2.minwi.lan   <none>           <none>
openshift-machine-config-operator                       etcd-quorum-guard-66b78568d6-8vqcw                                1/1     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-machine-config-operator                       etcd-quorum-guard-66b78568d6-jm5sv                                1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-machine-config-operator                       etcd-quorum-guard-66b78568d6-vgrtt                                1/1     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-machine-config-operator                       machine-config-controller-69876f9fc5-w2pms                        1/1     Running     0          21h     10.129.0.14    ocp4-master-2.minwi.lan   <none>           <none>
openshift-machine-config-operator                       machine-config-daemon-g85cl                                       1/1     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-machine-config-operator                       machine-config-daemon-m9pjv                                       1/1     Running     0          21h     192.168.32.200    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-machine-config-operator                       machine-config-daemon-pl2vv                                       1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-machine-config-operator                       machine-config-daemon-tk6rm                                       1/1     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-machine-config-operator                       machine-config-operator-84f487cc5d-z24lq                          1/1     Running     0          21h     10.129.0.5     ocp4-master-2.minwi.lan   <none>           <none>
openshift-machine-config-operator                       machine-config-server-97l6k                                       1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-machine-config-operator                       machine-config-server-9b4m6                                       1/1     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-machine-config-operator                       machine-config-server-jjg7q                                       1/1     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-marketplace                                   certified-operators-679bc8bccd-85kff                              1/1     Running     0          21h     10.128.0.3     ocp4-worker-0.minwi.lan   <none>           <none>
openshift-marketplace                                   community-operators-7969fd8f5f-978g2                              1/1     Running     0          16h     10.128.0.21    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-marketplace                                   marketplace-operator-fc68ffc58-cxzh8                              1/1     Running     0          21h     10.130.0.19    ocp4-master-0.minwi.lan   <none>           <none>
openshift-marketplace                                   redhat-operators-78b7b5b467-vk26p                                 1/1     Running     0          21h     10.128.0.5     ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    alertmanager-main-0                                               3/3     Running     0          21h     10.128.0.13    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    alertmanager-main-1                                               3/3     Running     0          21h     10.128.0.18    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    alertmanager-main-2                                               3/3     Running     0          21h     10.128.0.19    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    cluster-monitoring-operator-6b875c9f45-pzn4t                      1/1     Running     0          21h     10.130.0.18    ocp4-master-0.minwi.lan   <none>           <none>
openshift-monitoring                                    grafana-7cbddfd4f6-kg7dt                                          2/2     Running     0          21h     10.128.0.9     ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    kube-state-metrics-76dbd866ff-6g29v                               3/3     Running     0          21h     10.128.0.6     ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    node-exporter-bp2fx                                               2/2     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-monitoring                                    node-exporter-fcwtr                                               2/2     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-monitoring                                    node-exporter-hml9t                                               2/2     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-monitoring                                    node-exporter-tlc9c                                               2/2     Running     0          21h     192.168.32.200    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    prometheus-adapter-6669f77fcc-49xx8                               1/1     Running     0          3h23m   10.128.0.25    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    prometheus-adapter-6669f77fcc-5tl9t                               1/1     Running     0          3h23m   10.128.0.24    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    prometheus-k8s-0                                                  6/6     Running     1          21h     10.128.0.16    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    prometheus-k8s-1                                                  6/6     Running     1          21h     10.128.0.17    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    prometheus-operator-7bfd67bf6c-ws5cr                              1/1     Running     0          21h     10.128.0.12    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-monitoring                                    telemeter-client-d5d6757bb-9f7ps                                  3/3     Running     0          21h     10.128.0.8     ocp4-worker-0.minwi.lan   <none>           <none>
openshift-multus                                        multus-8mlgf                                                      1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-multus                                        multus-9r2c2                                                      1/1     Running     0          21h     192.168.32.200    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-multus                                        multus-h9rpm                                                      1/1     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-multus                                        multus-pk8bx                                                      1/1     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-network-operator                              network-operator-5f8b568759-b64gg                                 1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-operator-lifecycle-manager                    catalog-operator-7ff7b858bb-2kqfr                                 1/1     Running     0          21h     10.129.0.18    ocp4-master-2.minwi.lan   <none>           <none>
openshift-operator-lifecycle-manager                    olm-operator-6b6bc6fc8f-n55xv                                     1/1     Running     0          21h     10.129.0.16    ocp4-master-2.minwi.lan   <none>           <none>
openshift-operator-lifecycle-manager                    olm-operators-t7t2w                                               1/1     Running     0          21h     10.129.0.23    ocp4-master-2.minwi.lan   <none>           <none>
openshift-operator-lifecycle-manager                    packageserver-76b7856754-cz768                                    1/1     Running     0          3h24m   10.129.0.60    ocp4-master-2.minwi.lan   <none>           <none>
openshift-operator-lifecycle-manager                    packageserver-76b7856754-qm8jr                                    1/1     Running     0          3h23m   10.131.0.42    ocp4-master-1.minwi.lan   <none>           <none>
openshift-sdn                                           ovs-4cbk7                                                         1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-sdn                                           ovs-9vwmh                                                         1/1     Running     0          21h     192.168.32.200    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-sdn                                           ovs-d9fd4                                                         1/1     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-sdn                                           ovs-wgl97                                                         1/1     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-sdn                                           sdn-2j7fk                                                         1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-sdn                                           sdn-controller-g5lkj                                              1/1     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-sdn                                           sdn-controller-mmkqj                                              1/1     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-sdn                                           sdn-controller-xsnhm                                              1/1     Running     0          21h     192.168.32.102   ocp4-master-2.minwi.lan   <none>           <none>
openshift-sdn                                           sdn-dpw5w                                                         1/1     Running     0          21h     192.168.32.200    ocp4-worker-0.minwi.lan   <none>           <none>
openshift-sdn                                           sdn-rp777                                                         1/1     Running     0          21h     192.168.32.100   ocp4-master-0.minwi.lan   <none>           <none>
openshift-sdn                                           sdn-v6zwf                                                         1/1     Running     0          21h     192.168.32.101   ocp4-master-1.minwi.lan   <none>           <none>
openshift-service-ca-operator                           service-ca-operator-9654d9559-xwbf2                               1/1     Running     0          21h     10.129.0.10    ocp4-master-2.minwi.lan   <none>           <none>
openshift-service-ca                                    apiservice-cabundle-injector-5bf59477d7-ndr8q                     1/1     Running     0          21h     10.130.0.3     ocp4-master-0.minwi.lan   <none>           <none>
openshift-service-ca                                    configmap-cabundle-injector-6cfcd498d7-6788j                      1/1     Running     0          21h     10.129.0.15    ocp4-master-2.minwi.lan   <none>           <none>
openshift-service-ca                                    service-serving-cert-signer-7789d64745-xdmrt                      1/1     Running     0          21h     10.131.0.3     ocp4-master-1.minwi.lan   <none>           <none>
openshift-service-catalog-apiserver-operator            openshift-service-catalog-apiserver-operator-54dcb96555-ms9k4     1/1     Running     0          21h     10.131.0.10    ocp4-master-1.minwi.lan   <none>           <none>
openshift-service-catalog-controller-manager-operator   openshift-service-catalog-controller-manager-operator-f8cfjc2wj   1/1     Running     0          21h     10.131.0.9     ocp4-master-1.minwi.lan   <none>           <none>
```

[<< Previous: Upgrade](13-upgrade.md) | [Back to README](../README.md)
