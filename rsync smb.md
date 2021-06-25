## 1.rsync配置

/etc/rsyncd.conf

```shell
uid = root
gid = root
# use chroot = yes
# max connections = 4
# pid file = /var/run/rsyncd.pid
# exclude = lost+found/
 transfer logging = yes
 log file = /var/log/rsyncd.conf
 reverse lookup = 0
# timeout = 900
# ignore nonreadable = yes
# dont compress= *.gz *.tgz *.zip *.z *.Z *.rpm *.deb *.bz2

[hehe]
	path = /vdb
	read only = no
	auth users = hehe
	secrets file = /etc/rsync.pas

```

supervisord.d/rsync.ini配置

```shell
[program:rsync]
command= /usr/bin/rsync --daemon --no-detach
environment=TZ='CST-8'
autostart=true
#user=baoleiji
startsecs=1
startretries=3
stopsignal=KILL
```



### 1.1

rsync使用-e选项通过ssh传输时，要求c/s两端都要安装rsync软件；不用-e时也要两端都装

```
rsync -avz -e "sshpass -pgfqh2011 ssh" run.log 2.2.11.81:/tmp
```

### 1.2 include

```shell
rsync -rvz -e "sshpass -pgfqh2011 ssh" --include=M0[12]0*/***  --exclude=* pp/ 2.2.11.81:$(pwd)
```



##  2.smb.conf

```shell
[global]
	workgroup = SAMBA
	security = user

	passdb backend = tdbsam

	#map to guest = bad user
[s1]
	comment = Home Directories
	#valid users = %S, %D%w%S
	browseable = yes
	read only = No
	path = /opt/data
	#以//分隔的不使见文件
	veto files = /lost+found/.../
```

supervisord.d/smb.ini配置

```shell
[program:smb]
command= /usr/sbin/smbd --foreground --no-process-group
autostart=true
#user=baoleiji
startsecs=1
startretries=3
stopsignal=KILL
```

先systemctl start supervisord

再把*.ini放到/etc/supervisord.d下

再supervisorctl update



### 3. 使用

#### 3.1 服务器端

useradd hehe

pdbedit -a -u hehe

输入密码	#该密码与操作系统密码无关

忘记密码，用pdbedit -r -u zs先删除用户，再重新添加



#### 3.2 客户端

挂载命令：

```
mount -t cifs //2.2.11.81/s1 /media/smb/ -o username=zs
```

报错：

1. No route to host

   查看防火墙

2. mount: cannot mount //2.2.11.81/s1 read-only

   yum install cifs-utils

3. mount error(13): Permission denied

   确认密码是否正确



### 4. 防火墙

linux上启动的smb服务，只监听tcp的139和445端口，防火墙只需要放通445端口就可以让客户端来访问



### 5. vsftpd

#### 5.1 db_load

```
db_load -T -t hash -f /etc/vsftpd/virtual.txt /etc/vsftpd/vsftpd_login.db
```



#### 5.2 iptables

```
iptables -I INPUT 5 -p tcp -m tcp --dport 21 -m conntrack --ctstate NEW,UNTRACKED -j ACCEPT
#加载模块后管用了
modprobe nf_conntrack_ftp
#或者
modprobe nf_nat_ftp
++++++++++++++++++++++++++++++
#或者直接
firewall-cmd --add-service=ftp
```

