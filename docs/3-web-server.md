# Web server

To quickly spin up a web server, we will use the official NGINX container image:

```
mkdir -p ${NGINX_DIRECTORY}

podman run -d \
  --expose=8001 \
  -p 8001:80 \
  -v ${NGINX_DIRECTORY}:/usr/share/nginx/html:z \
  --name nginx \
  nginx:latest
```

Open the 8001 port to the outside world

```
sudo firewall-cmd --zone="$(firewall-cmd --get-default-zone)" \
  --add-port=8001/tcp --permanent
```

Reload the firewall:

```
sudo firewall-cmd --reload
```

## Systemd user unit
A systemd user unit can be created to automatically start/stop the podman container as a user:

```
# This shouldn't be needed if has been previously done for the HAProxy pod
# Enable start user unit without being logged first
# sudo loginctl enable-linger ocp

# Create the folder and unit file
mkdir -p ~/.config/systemd/user/
cat > ~/.config/systemd/user/nginx.service << EOF
[Unit]
Description=nginx

[Service]
Restart=always
ExecStart=/usr/bin/podman start -a nginx
ExecStop=/usr/bin/podman stop -t 10 nginx
KillMode=process

[Install]
WantedBy=multi-user.target
EOF

# Reload the service and enable/start the service
systemctl --user daemon-reload
systemctl --user enable nginx.service --now
```

[<< Previous: Load Balancer](2-load-balancer.md) | [README](../README.md) | [Next: DNS >>](4-dns.md)
