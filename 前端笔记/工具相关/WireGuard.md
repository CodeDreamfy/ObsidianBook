
参考，cloudflare token生成
https://zhuanlan.zhihu.com/p/581967733

```yml
version: '3.2'
services:
  ddns-go:
    image: jeessy/ddns-go
    ports:
      - 9876:9876        # 默认的webui控制端口
    restart: always
    volumes:
      - ./ddns-go:/root
  wg-easy:
    image: weejewel/wg-easy
    container_name: wg-easy
    environment:
      - WG_HOST=wg.todojs.xyz
      - PASSWORD=Zhang1918
      - WG_DEFAULT_ADDRESS=10.0.8.1
      - WG_DEFAULT_DNS=114.114.114.114,223.6.6.6
      - WG_ALLOWED_IPS=10.0.8.0/24,192.168.31.0/24
      - WG_PERSISTENT_KEEPALIVE=25
    volumes:
      - ./wireguard:/etc/wireguard
    ports:
      - "51820:51820/udp"
      - "51821:51821/tcp"
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
      - net.ipv4.ip_forward=1
    restart: unless-stopped
```

### 启动
```bash
docker compose up -d
docker compose logs -l
```

### 访问DDNS-go
```bash
http://192.168.31.54:9876/
```
#### 访问WireGuard
```bash
http://192.168.31.54:51821/
```

路由器要设置NAT转发，暴露两个端口转发到对应的机器上

1. UDP端口51820：用于WireGuard VPN服务。
2. TCP端口51821：用于Web UI界面。


使用interfaces方式：/etc/network/interfaces
```conf
iface ens18 inet static
    address 192.168.31.54
    netmask 255.255.255.0
    gateway 192.168.31.1
    dns-nameservers 223.6.6.6 114.114.114.114

```
重启
```bash
systemctl restart networking
```

使用dhcp方式： /etc/dhcpcd.conf
```conf
interface ens18
static ip_address=192.168.31.54/24
static routers=192.168.31.1
static domain_name_servers=223.6.6.6 114.114.114.114

```
重启
```bash
sudo systemctl restart dhcpcd
```