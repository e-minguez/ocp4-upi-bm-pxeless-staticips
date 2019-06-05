# Haproxy
To quickly spin up an HAProxy server, we will use the HAProxy official container image:

```
mkdir -p ${HAPROXY_DIRECTORY}

cat > ${HAPROXY_DIRECTORY}/haproxy.cfg << EOF
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          300s
    timeout server          300s
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 20000

# Useful for debugging, dangerous for production
listen stats
    bind :9000
    mode http
    stats enable
    stats uri /

frontend openshift-api-server
    bind *:6443
    default_backend openshift-api-server
    mode tcp
    option tcplog

backend openshift-api-server
    balance source
    mode tcp
    server bootstrap ${BOOTSTRAP_IP}:6443 check
    server master-0 ${MASTER0_IP}:6443 check
    server master-1 ${MASTER1_IP}:6443 check
    server master-2 ${MASTER2_IP}:6443 check

frontend machine-config-server
    bind *:22623
    default_backend machine-config-server
    mode tcp
    option tcplog

backend machine-config-server
    balance source
    mode tcp
    server bootstrap ${BOOTSTRAP_IP}:22623 check
    server master-0 ${MASTER0_IP}:22623 check
    server master-1 ${MASTER1_IP}:22623 check
    server master-2 ${MASTER2_IP}:22623 check

# As we are using rootless containers, we will bind to the 8080/tcp port in the host
# so we can map 1:1 (and then firewalld will perform the redirection)
frontend ingress-http
    bind *:8080
    default_backend ingress-http
    mode tcp
    option tcplog

backend ingress-http
    balance source
    mode tcp
    server worker-0 ${WORKER0_IP}:80 check

# As we are using rootless containers, we will bind to the 8443/tcp port in the host
# so we can map 1:1 (and then firewalld will perform the redirection)   
frontend ingress-https
    bind *:8443
    default_backend ingress-https
    mode tcp
    option tcplog

backend ingress-https
    balance source
    mode tcp
    server worker-0 ${WORKER0_IP}:443 check
EOF
```

Then, run the container:

```
podman run -d \
  --expose=9000 --expose=22623 --expose=6443 --expose=8080 --expose=8443 \
  -p 9000:9000 -p 22623:22623 -p 6443:6443 -p 8080:8080 -p 8443:8443 \
  -v ${HAPROXY_DIRECTORY}/haproxy.cfg:/usr/local/etc/haproxy/haproxy.cfg:Z \
  --name haproxy \
  haproxy:alpine
```

Configure firewall to forward 8443 to 443 and 8080 to 80

```
sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" \
  --add-forward-port=port=443:proto=tcp:toport=8443 --permanent
sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" \
  --add-forward-port=port=80:proto=tcp:toport=8080 --permanent
```

Open those ports to the outside world

```
for service in http https
do
  sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" \
    --add-service=${service} --permanent
done

for port in 9000 22623 6443
do
  sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" \
  --add-port=${port}/tcp --permanent
done
```

Reload the firewall

```
sudo firewall-cmd --reload
```

## Systemd user unit
A systemd user unit can be created to automatically start/stop the podman container as a user:

```
# Enable start user unit without being logged first
sudo loginctl enable-linger ocp

# Create the folder and unit file
mkdir -p ~/.config/systemd/user/
cat > ~/.config/systemd/user/haproxy.service << EOF
[Unit]
Description=HAProxy

[Service]
Restart=always
ExecStart=/usr/bin/podman start -a haproxy
ExecStop=/usr/bin/podman stop -t 10 haproxy
KillMode=process

[Install]
WantedBy=multi-user.target
EOF

# Reload the service and enable/start the service
systemctl --user daemon-reload
systemctl --user enable haproxy.service --now
```

[<< Previous: Variables](1-variables.md) | [Next: Web server >>](3-web-server.md)
