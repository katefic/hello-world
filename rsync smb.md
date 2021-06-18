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



useradd hehe

pdbedit -a -u hehe

输入密码





先systemctl start supervisord

再把*.ini放到/etc/supervisord.d下

再supervisorctl update