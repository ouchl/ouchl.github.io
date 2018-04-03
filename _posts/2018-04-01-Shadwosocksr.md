---
layout: post
title: Shadwosocksr ssr在Ubuntu下实现透明代理
---

## 相关软件包

sudo apt install dnsmasq ipset shadowsocks-libev

## ss-redir 配置

```shell
# 创建配置文件
sudo vim /etc/shadowsocks/ss-redir.json
# systemd 守护进程
sudo vim /etc/systemd/system/ss-redir.service
[Unit]
Description=Shadowsocks-Libev Client Service Redir Mode
After=network.target

[Service]
Type=simple
User=nobody
CapabilityBoundingSet=CAP_NET_ADMIN CAP_NET_BIND_SERVICE
ExecStart=/usr/bin/ss-redir -c /etc/shadowsocks/ss-redir.json -u

[Install]
WantedBy=multi-user.target
# 保持自启动
sudo systemctl start ss-redir.service
sudo systemctl enable ss-redir.service
```

## iptables + ipset 实现 chnroute 分流

```shell
sudo su
wget -O- 'http://ftp.apnic.net/apnic/stats/apnic/delegated-apnic-latest' | awk -F\| '/CN\|ipv4/ { printf("%s/%d\n", $4, 32-log($5)/log(2)) }' > /etc/chnroute.txt
exit
```

```shell
# 停用默认防火墙自启动服务
sudo systemctl disable iptables.service
# 新建启动脚本
sudo vim /etc/iptables/ss-up.sh
#!/bin/bash

ipset -N chnroute hash:net maxelem 65536

for ip in $(cat '/etc/chnroute.txt'); do
  ipset add chnroute $ip
done

iptables -t nat -N SHADOWSOCKS

# 直连服务器 IP
iptables -t nat -A SHADOWSOCKS -d [server_ip]/24 -j RETURN

# 允许连接保留地址
iptables -t nat -A SHADOWSOCKS -d 0.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 10.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 127.0.0.0/8 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 169.254.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 172.16.0.0/12 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 192.168.0.0/16 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 224.0.0.0/4 -j RETURN
iptables -t nat -A SHADOWSOCKS -d 240.0.0.0/4 -j RETURN

# 直连中国 IP
iptables -t nat -A SHADOWSOCKS -p tcp -m set --match-set chnroute dst -j RETURN
iptables -t nat -A SHADOWSOCKS -p icmp -m set --match-set chnroute dst -j RETURN

# 重定向到 ss-redir 端口
iptables -t nat -A SHADOWSOCKS -p tcp -j REDIRECT --to-port 10800
iptables -t nat -A SHADOWSOCKS -p udp -j REDIRECT --to-port 10800
iptables -t nat -A OUTPUT -p tcp -j SHADOWSOCKS
```

```shell
#测试执行情况
sudo chmod +x /etc/iptables/ss-up.sh
sudo sh /etc/iptables/ss-up.sh
# 创建停止脚本
sudo vim /etc/iptables/ss-down.sh
sudo chmod +x /etc/iptables/ss-down.sh
iptables -t nat -D OUTPUT -p icmp -j SHADOWSOCKS
iptables -t nat -D OUTPUT -p tcp -j SHADOWSOCKS
iptables -t nat -F SHADOWSOCKS
iptables -t nat -X SHADOWSOCKS
ipset destroy chnroute
# 复制修改原 iptables 的 systemd 配置
sudo cp /usr/lib/systemd/system/iptables.service /etc/systemd/system/iptables-proxy.service
sudo vim /etc/systemd/system/iptables-proxy.service

[Unit]
Description=Packet Filtering Framework and Shadowsocks-chnroute
Before=network-pre.target
Wants=network-pre.target

[Service]
Type=oneshot
ExecStart=/etc/iptables/ss-up.sh
ExecStop=/etc/iptables/ss-down.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
# 检查并添加自启动
sudo systemctl daemon-reload
sudo systemctl start iptables-proxy.service
sudo systemctl status iptables-proxy.service
sudo systemctl enable iptables-proxy.service
```

## dnsmasq 的配置
Dnsmasq 提供 DNS 缓存和 DHCP 服务功能，通过缓存 DNS 请求来提高对访问过的网址的连接速度。

```shell
sudo vim /etc/dnsmasq.conf
# 去掉这几行的注释
resolv-file=/etc/resolv.dnsmasq.conf
listen-address=127.0.0.1
# 修改 Dnsmasq DNS
sudo vim /etc/resolv.dnsmasq.conf 
nameserver 8.8.8.8
nameserver 8.8.4.4
# 修改系统 DNS
sudo vim /etc/resolv.conf
nameserver 127.0.0.1
# 启动 Dnsmasq 
sudo systemctl start dnsmasq.service
sudo systemctl status dnsmasq.service
sudo systemctl enable dnsmasq.service
# 重启系统看看各项功能是否正常
sudo reboot
```

