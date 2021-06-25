## 1. add-forward-port

```
#防火墙可以实现伪装IP的功能，下面的端口转发就会用到这个功能。
firewall-cmd --query-masquerade # 检查是否允许伪装IP
firewall-cmd --add-masquerade # 允许防火墙伪装IP
firewall-cmd --remove-masquerade# 禁止防火墙伪装IP

#启用之后，添加转发规则，转发到本机，toaddr参数省略了
firewall-cmd --add-forward-port=port=445:proto=tcp:toport=21
#然后通过445端口就可以访问ftp了，445端口原来的功能不能用了，但通过21端口也能访问ftp
#再将ftp服务移除
firewall-cmd --remove-service=ftp
#把samba服务也移除
firewall-cmd --remove-service=samba

nmap -n -Pn 2.2.11.81 -p1-65535

Starting Nmap 6.40 ( http://nmap.org ) at 2021-06-22 16:08 CST
Nmap scan report for 2.2.11.81
Host is up (0.00011s latency).
Not shown: 65533 filtered ports
PORT    STATE SERVICE
22/tcp  open  ssh
445/tcp open  microsoft-ds
MAC Address: 00:0C:29:4C:3E:5A (VMware)

Nmap done: 1 IP address (1 host up) scanned in 141.81 seconds

+++++++++++++++++++++++++++++++++++++++++
#添加对外部主机的转发
firewall-cmd --add-forward-port=port=80:proto=tcp:toaddr=2.2.11.208:toport=8080
#将抓包保存为文件
tcpdump -qenn port 80 or 8080 -w tomcat.cap
#然后用wireshark打开
```



## 2. rich rules

```
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="2.2.11.81/24" service name="ssh" accept"

firewall-cmd --permanent --add-rich-rule='rule family=ipv4 source address="27.115.42.170/32" port port=45685 protocol=tcp accept'
```



## 3. iptables

### 3.1 

iptables与firewalld服务不能同时启动，启动一个会自动关闭另外一个，而且两者保存的规则也不共享

```
# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[  OK  ]
```

```
# firewall-cmd --runtime-to-permanent
success

#保存在/etc/firewalld/zones/public.xml
```



**Consequently, `firewalld` can change the settings during runtime without existing connections being lost.**



### 3.2 iptables放通ftp

```
iptables -I INPUT -p tcp -m tcp --dport 21 -m conntrack --ctstate NEW,UNTRACKED -j ACCEPT --modprobe=nf_conntrack_ftp

++++++++++++++++++++++++++++++

#加载模块后管用了
modprobe nf_conntrack_ftp
#或者
modprobe nf_nat_ftp

++++++++++++++++++++++++++++++++++++++++++++
被动模式tls时，在服务器防火墙上放开50000-55000端口
```



++++++++++++++++++++++++++++++

### 3.3 centos7下端口转发

**转发到外部**

```
echo 1 > /proc/sys/net/ipv4/ip_forward

iptables -t nat -I PREROUTING -p tcp --dport 1521 -j DNAT --to-destination 2.2.11.208:8080
iptables -t nat -I POSTROUTING -p tcp --dport 8080 -j MASQUERADE

++++++++++++++++++++++++++++++++++++++++
#若还不通，查看FORWARD链中的设置
iptables -I FORWARD -p tcp --sport 8080 -j ACCEPT
iptables -I FORWARD -p tcp --dport 8080 -j ACCEPT
```

**转发到本机**

```
iptables -t mangle -A PREROUTING -p tcp -m tcp --dport 445 -j MARK --set-xmark 0x64
iptables -t nat -I PREROUTING -p tcp --dport 445 -j DNAT --to-destination 21
iptables -I INPUT -m conntrack --ctstate NEW,UNTRACKED -m mark --mark 0x64 -j ACCEPT

```

**本机访问**

```
iptables -t nat -A OUTPUT -d localhost -p tcp --dport 445 -j REDIRECT --to-ports 21
```

