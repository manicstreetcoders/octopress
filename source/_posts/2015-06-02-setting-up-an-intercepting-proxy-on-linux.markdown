---
layout: post
title: "Setting Up an Intercepting Proxy on Linux"
date: 2015-06-02 00:25:15 +0900
comments: true
categories: 
---

iPhone VPN 설정을 해서, 앱 트래픽을 VPN 서버로 돌리고, VPN 서버에서 트래픽을 까본다음에, 원래 목적지로 릴레이해주는 테크닉.

### Box

우선 DigitalOcean 에 Ubuntu box 를 하나 만든다.

### PPTP

이제 Ubuntu 박스에 VPN 서버를 설치해야 한다.

[How To Setup Your Own VPN With PPTP](https://www.digitalocean.com/community/tutorials/how-to-setup-your-own-vpn-with-pptp)

```
sudo apt-get install pptpd

/etc/pptpd.conf
localip 10.0.0.1
remoteip 10.0.0.100-200

/etc/ppp/chap-secrets
box1 pptpd kkkkkkkk

/etc/ppp/pptpd-options
ms-dns 8.8.8.8
ms-dns 8.8.4.4

service pptpd restart

netstat -alpn | grep :1723

/etc/sysctl.conf
net.ipv4.ip_forward = 1

sysctl -p
```

### iptables

NAT 룰 셋업을 해주고, 특정 포트로 향하는 트래픽만 리다이렉트한다.

```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE && iptables-save

iptables -t nat -A PREROUTING -i ppp+ -p tcp -d 1.2.3.4 --dport 25201 -j REDIRECT --to-port 9999
```

### socat

인터셉트된 트래픽을 덤프하고, 원래 호스트로 릴레이해준다.

```
socat -v TCP-LISTEN:9999,fork TCP:1.2.3.4:25201

tshark -i ppp0
```
