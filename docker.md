## 1. docker cli连接远程主机

server端修改systemd文件，添加        

```shell
	ExecStart=/usr/bin/dockerd -H fd:// -H tcp://127.0.0.1:2375        
```

或者修改/etc/docker/daemon.json to connect to the UNIX socket and an IP address, as follows:        

```shell
   {  "hosts": ["unix:///var/run/docker.sock", "tcp://127.0.0.1:2375"] }     
```

client端通过-H参数指定服务器端，来连接

```shell
docker -H ...   ps
				exec -it container_name sh
```

也可以配置环境变量

```shell
export DOCKER_HOST=tcp://<your host's ip>:2375
docker ps 
```



#### 1.2 使用socat查看docker cli与daemon的交互

```shell
socat -v UNIX-LISTEN:/tmp/dockerapi.sock,fork \
UNIX-CONNECT:/var/run/docker.sock &

docker -H unix:///tmp/dockerapi.sock ps -a
============================================================

socat -v TCP-LISTEN:2375,fork \
UNIX-CONNECT:/var/run/docker.sock &

docker -H tcp://2.2.11.79:2375 ps
```



### 2. dockerfile

> **entrypoint.sh脚本中最后一行用**
>
> **exec "$@"**
>
> **同时用CMD[ ]来设置默认参数**
>
> You can only override the entrypoint if you explicitly pass in an
> --entrypoint flag to the docker run command
>
> 





#### 2.1 Only the instructions `RUN`, `COPY`, `ADD` create layers.



#### 2.2 The `CMD` instruction has three forms:

- `CMD ["executable","param1","param2"]` (*exec* form, this is the **preferred** form)
- `CMD ["param1","param2"]` (as *default parameters to ENTRYPOINT*)
- `CMD command param1 param2` (*shell* form)

#### 2.3 `CMD` and `ENTRYPOINT`

**Both `CMD` and `ENTRYPOINT` instructions define what command gets executed when running a container. There are few rules that describe their co-operation.**

1. Dockerfile should specify at least one of `CMD` or `ENTRYPOINT` commands.
2. `ENTRYPOINT` should be defined when using the container as an executable.
3. `CMD` should be used as a way of defining default arguments for an `ENTRYPOINT` command or for executing an ad-hoc command in a container.
4. `CMD` will be overridden when running the container with alternative arguments.

The table below shows what command is executed for different `ENTRYPOINT` / `CMD` combinations:

|                                | No ENTRYPOINT              | ENTRYPOINT exec_entry p1_entry | ENTRYPOINT [“exec_entry”, “p1_entry”]          |
| :----------------------------- | :------------------------- | :----------------------------- | :--------------------------------------------- |
| **No CMD**                     | *error, not allowed*       | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry                            |
| **CMD [“exec_cmd”, “p1_cmd”]** | exec_cmd p1_cmd            | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry exec_cmd p1_cmd            |
| **CMD [“p1_cmd”, “p2_cmd”]**   | p1_cmd p2_cmd              | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry p1_cmd p2_cmd              |
| **CMD exec_cmd p1_cmd**        | /bin/sh -c exec_cmd p1_cmd | /bin/sh -c exec_entry p1_entry | exec_entry p1_entry /bin/sh -c exec_cmd p1_cmd |



> centos的镜像，exec和shell格式的entrypoint，都会将进程ID设置为1，但是docker stop却还是要用10s



#### 2.4 copy

copy指令，只能将当前目录中的内容(包含子目录）复制到容器中，不能复制目录本身，

Optionally `COPY` accepts a flag `--from=<name>` that can be used to set the source location to a previous build stage (created with `FROM .. AS <name>`) that will be used instead of a build context sent by the user

```shell
#要注意context_dir
docker build -t tag_name -f path/to/dockerfile context_dir
```

为镜像配置时区

```dockerfile
COPY /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```



#### 2.5 ENV

为镜像配置编码locale



#### 2.6 VOLUME

在dockerfile中指定多个volume

```dockerfile
VOLUME ["/data1", "/data2"]
```

docker run时，不指定-v选项，则自动创建dockerfile中VOLUME指令定义的匿名卷；

若手动用-v选项，则除dockerfile中创建的以外，还会再创建-v选项指定的卷

不管是docker run，还是docker volume create创建的命名卷，位置都是由docker daemon来管理；要想手动指定位置，用bind mount

--volume-from 除了自己带的卷外，再链接from过来的卷；

先stop，再rm容器，则匿名卷不会被删除；docker run --rm则容器停止后匿名卷也会被删除



#### 2.7 EXPOSE

EXPOSE指令，仅是为了在docker run 用-P选项时，将指令中的端口映射到主机；若不用该指令，但在docke run 中指定-p80:80，则直接将80端口映射出去

```
docker inspect -f '{{json .Mounts}}' container_id | jq
```



#### 2.6 其他

- HEALTHCHECK

  HEALTHCHECK --period --retry 等等参数 CMD 检测命令

- USER

- ARG

  dockerfile中使用ARG指令，然后在docker build --build-arg ARG_NAME=VALUE



### 3. ssl registry

> 使用http的registry只能通过localhost访问

#### 3.1 生成自签证书

```shell
cd /etc/pki/tls/
#编辑openssl.cnf,在[v3_ca]下面添加：subjectAltName = IP:域名|IP地址
[ v3_ca ]
subjectAltName = IP:2.2.11.79
```

> 否则将会报错：
>
> ```
> x509: cannot validate certificate for <ipaddress> because it doesn't contain any IP SANs
> ```

```shell
mkdir certs
openssl req -subj '/C=CN/ST=BeiJing/L=HaiDian/CN=2.2.11.79 ' -newkey rsa:2048 -new -nodes -x509 -days 3650 -keyout certs/domain.key -out certs/domain.crt
```



#### 3.2 追加证书

```shell
#将生成的私有证书追加到系统的证书管理文件中，否则后面push和login和pull时会报如下错误：
###########################################报错
The push refers to repository [192.168.0.123/rabbitmq]
Get https://<IpAddress>/v2/: x509: certificate signed by unknown authority

++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
#客户端只需要执行这两步
cat certs/domain.crt >> /etc/pki/tls/certs/ca-bundle.crt
#重启docker, 该步骤一定不要省略，否则有可能加载私钥失败
systemctl restart docker
++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++++

```



#### 3.3 启动registry

```shell
#启动私有仓库镜像 registry,注意：在启动时，有参数需要配置
docker run -d   --restart=always   --name registry   -v "$(pwd)"/certs:/certs   -e REGISTRY_HTTP_ADDR=0.0.0.0:443   -e REGISTRY_HTTP_TLS_CERTIFICATE=/certs/domain.crt   -e REGISTRY_HTTP_TLS_KEY=/certs/domain.key   -p 443:443   registry

#验证
docker tag ...
docker push/pull ...
```



### 4. 网络

#### 4.1 创建网络

```shell
$ docker network create my_network
docker network create --gateway 1.1.1.1 --subnet 1.1.1.0/24 n9e-network
#将已运行的容器，添加一个到指定网络的连接，原有连接不变
$ docker network connect my_network container_name
```



#### 4.2 --link，

**这个是旧的命令，新版中不用link，连接到同一网段的容器自动能根据主机名获取到IP地址解析**

```shell
$ docker run --name wp-mysql \
-e MYSQL_ROOT_PASSWORD=123456 -d mysql
$ docker run --name wordpress \
--link wp-mysql:mysql -p 10003:80 -d wordpress
```



#### 4.3 修改docker0地址

/etc/docker/daemon.json添加

```shell
{
"bip" : "1.1.1.1/24"
}
```





### 5. docker-compose

docker compose 向entrypoint添加参数，目前用的是添加command行，启动后再修改docker-compose.yml文件，注释掉command行，**这样不行**





### 6. 官方

- This command changes the restart policy for an already running container named `redis`.

```
$ docker update --restart unless-stopped redis
```



- Use the following JSON to enable `live-restore`.

  ```
  {
    "live-restore": true
  }
  ```



#### 6.1 Runtime options with Memory, CPUs, and GPUs

```
#查看cgroup的挂载点
grep cgroup /proc/mounts

docker stats [container_id]

docker run --memory [--cpu*...]
```



#### 6.2 logging

```
#配置默认log-driver
{
  "log-driver": "local"
}

#查看docker daemon的默认log-driver
docker info --format '{{.LoggingDriver}}'

#查看运行中容器的log-driver
docker inspect -f '{{.HostConfig.LogConfig.Type}}' <CONTAINER>

#指定容器运行时的log-driver
docker run -it --log-driver none alpine ash

#安装新的log-driver plugin
docker plugin install <org/image>

#查看安装的plugin
docker plugin ls
```



### 7.安全

```
docker run -ti --cap-drop=单个|all
```

![image-20210518182128564](D:\typora\images\image-20210518182128564.png)

![image-20210518182206279](D:\typora\images\image-20210518182206279.png)

![image-20210518182241606](D:\typora\images\image-20210518182241606.png)



#查看镜像压缩后大小

docker save ubuntu:14.04.2 | gzip | wc -c





docker build --no-cache 







docker inspect --format='{{.NetworkSettings.IPAddress}}'



docker-machine创建的是一台安装docker环境的虚拟机，然后通过设置环境变量，将docker指令直接作用于新虚拟机上的docker daemon



docker ps -a -q --filter=status=exited

安装时要求container-selinux，需要添加repo

baseurl=http://mirror.centos.org/centos/7/extras/x86_64/