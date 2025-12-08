1. Install Docker
``` bash
 sudo dnf -y install dnf-plugins-core
 sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
 sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```
2. Install keepalived
``` bash
sudo dnf install keepalived -y
sudo systemctl enable keepalived
```
3. /opt/lb/nginx.conf vm1
``` nano
events {}

http {
    upstream backend {
        server 10.0.0.10:8080;
        server 10.0.0.11:8080;
    }

    server {
        listen 80;
        location / {
            proxy_pass http://backend;
        }
    }
}
```
nginx-compose
``` vim
services:
  nginx:
    image: nginx:latest
    container_name: nginx_lb
    ports:
      - "80:80"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
#    network_mode: "host"
    networks:
      - nginx_net
    restart: always
networks:
  nginx_net:
    driver: bridge
```
``` bash
 docker compose -f nginx-compose.yaml up -d
```

4. Config /etc/keepalived/keepalived.conf vm1
``` nano
vrrp_instance VI_1 {
    state MASTER
    interface ens33  # یا نام اینترفیس شبکه شما
    virtual_router_id 51
    priority 101
    advert_int 1
    virtual_ipaddress {
        172.29.98.130
    }
}

```
5. Config /etc/keepalived/keepalived.conf vm2
``` nano
vrrp_instance VI_1 {
    state BACKUP
    interface ens33  # یا نام اینترفیس شبکه شما
    virtual_router_id 51
    priority 100
    advert_int 1
    virtual_ipaddress {
        172.29.98.130
    }
}

```
6. restart
``` bash
sudo systemctl restart keepalived
sudo systemctl status keepalived
```
