1. Install Docker
``` bash
sudo dnf -y install dnf-plugins-core
sudo dnf config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
```
3. Tree
``` tree
.
└── vault-docker-compose
    ├── tls
    │   ├── vault-cert.pem
    │   └── vault-key.pem
    ├── vault-compose.yaml
    └── vault-config.hcl
```
3. Config vault-config.hcl
``` yaml
# پیکربندی Vault برای استفاده از Consul به عنوان backend
storage "consul" {
  address = "http://consul:8500"  # آدرس Consul
  path    = "vault/"
  scheme  = "http"
}

# Listener تنظیم شده برای استفاده از TLS
listener "tcp" {
  address = "0.0.0.0:8200"
  tls_cert_file = "/vault/tls/vault-cert.pem"  # مسیر گواهینامه سرور
  tls_key_file  = "/vault/tls/vault-key.pem"   # مسیر کلید خصوصی
}

# listener دوم برای cluster communication
listener "tcp" {
  address     = "0.0.0.0:8201"
  tls_cert_file = "/vault/tls/vault-cert.pem"
  tls_key_file  = "/vault/tls/vault-key.pem"
}

# غیر فعال کردن MLock (برای جلوگیری از مشکلات در Docker)
disable_mlock = true

# پیکربندی API و Cluster
api_addr = "https://172.29.98.156:8200"  # آدرس API Vault
cluster_addr = "https://172.29.98.156:8201"  # آدرس Cluster Vault

# فعال کردن UI
ui = true
```
4. Config vault-compose.yaml
``` yaml
services:
  vault:
    image: hashicorp/vault:latest
    container_name: vault
    ports:
      - "8200:8200"
      - "8201:8201"
    #environment:
    #  - VAULT_ADDR=https://172.29.98.156:8200
    #  - VAULT_DEV_ROOT_TOKEN_ID=root
    volumes:
      - ./vault-config.hcl:/vault/config/vault-config.hcl
      - ./tls:/vault/tls
    networks:
      - vault-network
    depends_on:
      - consul
    restart: always
    command: server
    cap_add:
      - IPC_LOCK  # این خط را اضافه کنید

  consul:
    image: consul:1.15.4
    container_name: consul
    ports:
      - "8500:8500"
    #environment:
    #  - CONSUL_CLIENT_INTERFACE=0.0.0.0  # آدرس IP Consul را برای اتصال تنظیم می‌کنیم
    #  - CONSUL_BIND_INTERFACE=eth0  # گوش دادن به همه interfaces
    networks:
      - vault-network
    restart: always
    command: "consul agent -server -bootstrap-expect=1 -ui -client=0.0.0.0 -data-dir=/consul/data"
    volumes:
      - ./consul-data:/consul/data  # اتصال دایرکتوری داده‌ها

networks:
  vault-network:
    driver: bridge
```
5.Create TLS
```bash
openssl req -newkey rsa:4096 -nodes -keyout vault-key.pem -x509 -out vault-cert.pem
```
6.
``` bash
docker compose -f vault-compose.yaml up -d
```
7. use https://ip:8200/ui for 
