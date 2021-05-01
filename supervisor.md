## 1.supervisor 安装

```shell
方式一：
		pip install supervisor
		
方式二：
		wget https://files.pythonhosted.org/packages/d3/7f/c780b7471ba0ff4548967a9f7a8b0bfce222c3a496c3dfad0164172222b0/supervisor-4.2.2.tar.gz
		tar xf supervisor-4.2.2.tar.gz
		cd supervisor
		python setup.py install
		
		#在rhel6上面以该方式安装，先升级python到3，然后升级glibc（--prefix=/usr），又升级libffi，最后安装成功；还需要手动创建配置文件和.d/.ini文件，再以supervisord -c /path/to/supervisord.conf方式启动
		
方式三：
		yum install supervisor
```
## -2.配置

生成示例配置文件，该文件主要配置supervisord服务本身，具体program的配置文件通过include section包含

yum**安装自带配置文件**

```ini
[include]
files = /relative/path/*.conf
```

nginx.conf配置

```ini
[program:Nginx-srv]
command=nginx -g 'daemon off;'
;command=/usr/sbin/nginx
autostart=true
autorestart=true
startsecs=1
startretries=3
stopsignal=QUIT
stderr_logfile=/var/log/supervisor_nginx.err.log
stdout_logfile=/var/log/supervisor_nginx.out.log
;stopsignal=KILL

```



## 3.启动

```shell
supervisord -c /path/to/supervisord.conf

systemctl start supervisord
```

## 4.管理

4.1

```shell
#指定所用的配置文件，该文件中可以配置为连接远程的supervisord服务

#4.2.2版本中，将进程sock的chown=zs后，可以通过普通用户zs来直接使用supervisorctl，不用用sudo了
[supervisorctl]
serverurl=unix:///var/run/supervisor/supervisor.sock ; use a unix:// URL  for a unix socket
serverurl=http://2.2.11.81:9001 ; use an http:// url to specify an inet socket


supervisorctl -c /path/to/supervisord.conf

#不修改配置文件，直接指定连接的服务器
supervisorctl -s http://2.2.11.81:9001


命令式：
supervisorctl status program
					  start
					  stop
					  reload
					  update
					  pid
					  tail -f
					  
交互式：
supervisorctl
> status
  start name
  ...
```

4.2 web页面

直接通过浏览器访问

http://2.2.11.79:9001