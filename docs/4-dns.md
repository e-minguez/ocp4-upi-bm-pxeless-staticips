# DNS
To quickly spin up a DNS server, we will use the official CoreDNS container image:

```
mkdir -p ${COREDNS_DIRECTORY}

# CoreDNS configuration file
cat > ${COREDNS_DIRECTORY}/Corefile << EOF
.:53 {
    log
    errors
    forward . ${DNSFORWARDER}
}

${DOMAIN_NAME}:53 {
    log
    errors
    file /etc/coredns/db.${DOMAIN_NAME}
}
EOF

```
Then, the proper zone file, including the SRV records, CNAMES, etc. To avoid escaping dollar symbols, etc. we create the template file first then replace the variables using sed:

```
cat > ${COREDNS_DIRECTORY}/db.${DOMAIN_NAME} << 'EOF'
$ORIGIN DOMAIN_NAME.
$TTL 10800      ; 3 hours
@       3600 IN SOA sns.dns.icann.org. noc.dns.icann.org. (
                                2019010101 ; serial
                                7200       ; refresh (2 hours)
                                3600       ; retry (1 hour)
                                1209600    ; expire (2 weeks)
                                3600       ; minimum (1 hour)
                                )

_etcd-server-ssl._tcp.CLUSTER_NAME.DOMAIN_NAME. 8640 IN    SRV 0 10 2380 etcd-0.CLUSTER_NAME.DOMAIN_NAME.
_etcd-server-ssl._tcp.CLUSTER_NAME.DOMAIN_NAME. 8640 IN    SRV 0 10 2380 etcd-1.CLUSTER_NAME.DOMAIN_NAME.
_etcd-server-ssl._tcp.CLUSTER_NAME.DOMAIN_NAME. 8640 IN    SRV 0 10 2380 etcd-2.CLUSTER_NAME.DOMAIN_NAME.

api.CLUSTER_NAME.DOMAIN_NAME.                        A                LB_IP
api-int.CLUSTER_NAME.DOMAIN_NAME.                    A                LB_IP
CLUSTER_NAME-master-0.DOMAIN_NAME.                   A                MASTER0_IP
CLUSTER_NAME-master-1.DOMAIN_NAME.                   A                MASTER1_IP
CLUSTER_NAME-master-2.DOMAIN_NAME.                   A                MASTER2_IP
CLUSTER_NAME-worker-0.DOMAIN_NAME.                   A                WORKER0_IP
CLUSTER_NAME-bootstrap.DOMAIN_NAME.                  A                BOOTSTRAP_IP
etcd-0.CLUSTER_NAME.DOMAIN_NAME.                     IN  CNAME CLUSTER_NAME-master-0.DOMAIN_NAME.
etcd-1.CLUSTER_NAME.DOMAIN_NAME.                     IN  CNAME CLUSTER_NAME-master-1.DOMAIN_NAME.
etcd-2.CLUSTER_NAME.DOMAIN_NAME.                     IN  CNAME CLUSTER_NAME-master-2.DOMAIN_NAME.

$ORIGIN apps.CLUSTER_NAME.DOMAIN_NAME.
*                                                    A                LB_IP
EOF

sed -i -e "s/MASTER0_IP/${MASTER0_IP}/g" \
       -e "s/MASTER1_IP/${MASTER1_IP}/g" \
       -e "s/MASTER2_IP/${MASTER2_IP}/g" \
       -e "s/WORKER0_IP/${WORKER0_IP}/g" \
       -e "s/CLUSTER_NAME/${CLUSTER_NAME}/g" \
       -e "s/DOMAIN_NAME/${DOMAIN_NAME}/g" \
       -e "s/BOOTSTRAP_IP/${BOOTSTRAP_IP}/g" \
       -e "s/LB_IP/${LB_IP}/g" \
       ${COREDNS_DIRECTORY}/db.${DOMAIN_NAME}
```

I've not been able to query from localhost or $(hostname -I) from the host running the coredns container when running rootless.
`/etc/resolv.conf` doesn't allow specific ports and redirecting localhost/ip is really messy, so, this container runs with sudo (and binds to :53):

```
sudo podman run -d \
  --expose=53 --expose=53/udp \
  -p ${DNS}:53:53 -p ${DNS}:53:53/udp \
  -v ${COREDNS_DIRECTORY}:/etc/coredns:z \
  --name coredns \
  coredns/coredns:latest -conf /etc/coredns/Corefile

sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" --add-service=dns --permanent
sudo firewall-cmd --reload
```

## Systemd unit
A systemd unit can be created to automatically start/stop the podman container. In this case, as we require root, we will create a regular systemd unit:

```
sudo bash -c 'cat > /etc/systemd/system/coredns.service << EOF
[Unit]
Description=CoreDNS

[Service]
Restart=always
ExecStart=/usr/bin/podman start -a coredns
ExecStop=/usr/bin/podman stop -t 10 coredns
KillMode=process

[Install]
WantedBy=multi-user.target
EOF'

# Reload the service and enable the service
systemctl daemon-reload
systemctl enable coredns.service
```

To see if it works:

```
$ sudo podman ps
CONTAINER ID  IMAGE                             COMMAND               CREATED        STATUS            PORTS                                           NAMES
abf77a3da374  docker.io/coredns/coredns:latest  /coredns -conf /e...  9 minutes ago  Up 2 seconds ago  10.19.138.7:53->53/tcp, 10.19.138.7:53->53/udp  coredns

$ sudo systemctl stop coredns.service
$ sudo podman ps
CONTAINER ID  IMAGE  COMMAND  CREATED  STATUS  PORTS  NAMES
```

Then, modify `/etc/resolv.conf` to use the coredns container:

```
sudo nmcli con mod 'System eno1' ipv4.ignore-auto-dns yes
sudo nmcli con mod 'System eno1' ipv4.dns "${DNS}"
sudo systemctl restart NetworkManager
```

[<< Previous: Web server](3-web-server.md) | [Next: OpenShift files >>](5-openshift-files.md)
