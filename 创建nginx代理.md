``` bash
mkdir -p /etc/nginx

cat << EOF >> /etc/nginx/nginx.conf
error_log stderr notice;

worker_processes auto;
events {
  multi_accept on;
  use epoll;
  worker_connections 1024;
}

stream {
    upstream kube_apiserver {
        least_conn;
        server 10.100.158.144:6666;
        server 10.100.158.145:6666;
        server 10.100.158.146:6666;
    }

    server {
        listen        0.0.0.0:6443;
        proxy_pass    kube_apiserver;
        proxy_timeout 10m;
        proxy_connect_timeout 1s;
    }
}
EOF
```

chmod +r /etc/nginx/nginx.conf

``` bash
cat << EOF >> /etc/systemd/system/nginx-proxy.service
[Unit]
Description=kubernetes apiserver docker wrapper
Wants=docker.socket
After=docker.service

[Service]
User=root
PermissionsStartOnly=true
ExecStart=/root/local/bin/docker run -p 127.0.0.1:6443:6443 \\
                              -v /etc/nginx:/etc/nginx \\
                              --name nginx-proxy \\
                              --net=host \\
                              --restart=on-failure:5 \\
                              --memory=512M \\
                              nginx:1.13.7
ExecStartPre=-/root/local/bin/docker rm -f nginx-proxy
ExecStop=/root/local/bin/docker stop nginx-proxy
Restart=always
RestartSec=15s
TimeoutStartSec=30s

[Install]
WantedBy=multi-user.target
EOF
```

``` bash
systemctl daemon-reload
systemctl start nginx-proxy
systemctl enable nginx-proxy
```