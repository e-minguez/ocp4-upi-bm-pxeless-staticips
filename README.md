# Intro

The objective of this document is to provide instructions (automated ish) to install OCP4 on baremetal:
* without PXE (pretty common scenario in big companies)
* avoid installing stuff and use containers instead (instead yum/dnf install httpd, haproxy,... use containers)
* use rootless containers if possible
* use Fedora29/RHEL8 stuff (nmcli, firewalld, etc.)

# Current status

OCP4.1 GA installed

# Environment

| Usage     | Hostname                 | IP             | NOTES                                         |
|-----------|--------------------------|----------------|-----------------------------------------------|
| Helper    | ocp4-helper.minwi.lan    | 192.168.32.2   | DNS, httpd, etc.                              |
| Bootstrap | ocp4-bootstrap.minwi.lan | 192.168.32.99  | To be removed from the cluster once installed |
| Master-0  | ocp4-master-0.minwi.lan  | 192.168.32.100 |                                               |
| Master-1  | ocp4-master-1.minwi.lan  | 192.168.32.101 |                                               |
| Master-2  | ocp4-master-2.minwi.lan  | 192.168.32.102 |                                               |
| Worker-0  | ocp4-worker-0.minwi.lan  | 192.168.32.200 |                                               |

NOTE: Those baremetal servers are Dell based, so `racadm` will be used in order to manage the iDRAC to map a virtual cd, power off/on, etc.

# References

* https://github.com/christianh814/openshift-toolbox/tree/master/ocp4_upi_beta
* https://docs.openshift.com/container-platform/4.1/installing/installing_bare_metal/installing-bare-metal.html

# Steps

* [0 - Prerequisites](docs/0-prerequisites.md)
* [1 - Variables](docs/1-variables.md)
* [2 - Load balancer](docs/2-load-balancer.md)
* [3 - Web server](docs/3-web-server.md)
* [4 - DNS](docs/4-dns.md)
* [5 - OpenShift files](docs/5-openshift-files.md)
* [6 - Cluster files](docs/6-cluster-files.md)
* [7 - Ignition files](docs/7-ignition-files.md)
* [8 - RHCOS files](docs/8-rhcos-files.md)
* [9 - Modify ISOs](docs/9-modify-isos.md)
* [10 - Configure `oc` command](docs/10-configure-oc-command.md)
* [11 - Installation](docs/11-installation.md)
* [12 - Post installation](docs/12-post-installation.md)
* [13 - Upgrade](docs/13-upgrade.md)
* [14 - Verification](docs/14-verification.md)
* [99 - Tips and tricks](docs/99-tips-and-tricks.md)
