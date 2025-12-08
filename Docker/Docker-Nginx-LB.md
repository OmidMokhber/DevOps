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
docker run -d \
  --name nginx-lb \
  -p 80:80 \
  -v /opt/lb/nginx.conf:/etc/nginx/nginx.conf \
  nginx:latest

4. Config /etc/keepalived/keepalived.conf vm1
``` nano
vrrp_script chk_nginx {
    script "/usr/bin/docker ps | grep nginx-lb"
    interval 2
}

vrrp_instance VI_1 {
    interface eth0
    state MASTER
    virtual_router_id 52
    priority 150
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        172.29.98.130
    }

    track_script {
        chk_nginx
    }
}
```
5. Config /etc/keepalived/keepalived.conf vm2
``` nano
vrrp_script chk_nginx {
    script "/usr/bin/docker ps | grep nginx-lb"
    interval 2
}

vrrp_instance VI_1 {
    interface eth0
    state BACKUP
    virtual_router_id 52
    priority 100
    advert_int 1

    authentication {
        auth_type PASS
        auth_pass 1234
    }

    virtual_ipaddress {
        172.29.98.130
    }

    track_script {
        chk_nginx
    }
}
```
6. restart
``` bash
sudo systemctl restart keepalived
sudo systemctl status keepalived
```
