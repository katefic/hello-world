## 1.云计算类型

云计算是一种按量付费模式，基础是虚拟化

##### 1.1 IaaS 基础设施即服务 ecs

##### 1.2 PaaS 平台即服务

##### 1.3 SaaS 软件即服务 cdn、rds

## 2.安装KVM

##### 2.1 安装

```shell
#查看CPU是否支持虚拟化技术
cat /proc/cpuinfo | egrep 'vmx|svm'
yum install libvirt virt-install qemu-kvm
#libvirtd配置文件
/etc/libvirt/libvirtd.conf

#开启libvirtd服务，可以看到有virbr0网卡
systemctl enable --now libvirtd
ip l
#virbr0的配置文件
less /etc/libvirt/qemu/networks/default.xml

#使用virt-install 安装一台kvm虚拟机
virt-install --virt-type kvm \
			 --os-type linux \
			 --os-variant rhel7 \
			 --name kvmh1 \
			 --memory 1024 \
			 --vcpus 1 \
			 --disk /opt/kvmh1.raw,format=raw,size=10 \
			 --network bridge=virbr0 \
			 --cdrom CentOS-7-x86_64-Minimal-2009.iso \
			 --graphics vnc,listen=0.0.0.0 \
			 --noautoconsole
			 --extra_args "ks=http://192.168.122.1/ks_kvm.cfg"
#然后在windows系统上通过vncviewer连接地址2.2.11.78:5900连接，打开安装界面
#在宿主机上可以通过ss命令查看qemu-kvm进程打开了5900端口，再启下一个虚拟机就是5901
```

##### 2.2 管理命令

```shell
virsh list [--all]
virsh start kvmh1
	  shutdown
	  destory
	  reboot
	  suspend
	  resume
	  vncdisplay kvmh1	//查询vnc端口号
	  domrename kvmh1 web-blog	//在关机状态下重命名虚拟机
	  
#导出配置文件	  
virsh dumpxml kvmh1 > filename.xml
#利用磁盘文件和配置文件恢复虚拟机
1.先将磁盘文件放在原来的目录下，如/opt
2.virsh define filename.xml

#删除虚拟机
1.先destory
2.再undefine kvmh1 --remove-all-storage

#编辑配置文件
virsh edit kvmh1

#批量备份配置文件，配置文件默认保存在/etc/libvirt/qemu目录下
for i in $(virsh list --all | awk -F'[ ]+' 'NR > 2 && NR < $NR {print $3}') ; do 
	virsh dumpxml $i > /tmp/$i.xml
done

#设置开机启动
virsh autostart kvmh1
#取消开机自启
virsh autostart --disable kvmh1
#查看已设置为开机自启的虚拟机
[root@rhel78 ~]# ll /etc/libvirt/qemu/autostart/
total 0
lrwxrwxrwx 1 root root 30 2021-03-06 10:08 web-blog.xml -> /etc/libvirt/qemu/web-blog.xml

#为kvm虚拟机修改console相关的启动参数，重启生效
grubby --update-kernel=ALL --args="console=ttyS0,115200n8"
reboot
#通过console连接kvm虚拟机
virsh console kvmh1

#可以通过查看virbr0通过dnsmasq分配出去的IP地址
cat /var/lib/libvirt/dnsmasq/virbr0.status
```

## 3.虚拟机磁盘文件格式、快照

##### 3.1 2种磁盘文件格式

​		raw	占用空间大，不支持快照，不方便传输，性能好

​		qcow2 (cow,copy on write)，支持快照，占用空间小，性能略差

##### 3.2 ==qemu-img命令==

```shell
#查看磁盘信息
qemu-img info /data/kvmh1.raw

#创建指定格式的磁盘文件，默认格式为raw
qemu-img create -f qcow2 kvmh2.qcow2 1G

#扩展磁盘容量，不要缩容shrinking
qemu-img resize kvmh2.qcow2 1G
```

```shell
#先将kvm虚拟机关机
virsh shutdown kvmh2
#转换磁盘文件格式
qemu-img convert -f raw -O qcow2 kvmh2.raw kvmh2.qcow2
```

修改对应的配置文件

```shell
virsh edit web-blog
```

```xml
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2'/>
  <source file='/data/kvmh1.qcow2'/>
  <target dev='vda' bus='virtio'/>
  <address type='pci' domain='0x0000' bus='0x00' slot='0x06' function='0x0'/>
</disk>
```

再用virsh start测试启动新磁盘文件

##### 3.3 快照

==raw格式不支持快照，qcow2格式支持，并且快照文件就保存在.qcow2磁盘文件中==

```shell
#创建命名快照
virsh snapshot-create-as web-blog --name env_ok

#查看快照
virsh snapshot-list web-blog

#删除快照，可以用virsh snapshot-delete --help查看帮助
virsh snapshot-delete web-blog --snapshotname env_ok

#还原快照
virsh snapshot-revert web-blog --snapshotname env_ok
```

##### 3.4 clone

```shell
#自动克隆，完整克隆
virt-clone --auto-clone -o kvmh1 -n web02
#链接克隆，手动创建磁盘文件
qemu-img create -f qcow2 -b orikvmname newkvmname
然后修改归档出来的配置文件，再define,start

#取ORIDISKFILE
virsh dumpxml kvmh2 | grep -oP "(?<=source file=').*(raw|qcow2)"
```

## 4. 虚拟机的网络

##### 4.1 虚拟机默认使用的网卡类型为NAT，像vmware一样；



> 每启动一个虚拟机，就会在virbr0上面接入一个vnet#的网卡，该网卡会通过virbr0的dhcp获取到一个192.168.122.x的地址，ssh登录到虚拟机，可以ping通2.2.11.78，说明地址是主机的，不是网卡的；而ping不通2.2.11.1，是因为/proc/sys/net/ipv4/ip_forward为0

默认的default(nat)和桥接到virbr0有什么区别



##### 4.2 创建桥接模式的网络

```shell
#创建桥接用的桥，删除用iface-unbridge
[root@rhel78 ~]# virsh iface-bridge ens32 kvmbr0
Created bridge kvmbr0 with attached device ens32
Bridge interface kvmbr0 started
#此时会将原ens32网卡上的地址移动到kvmbr0这个网桥上，具体可以到/etc/sysconfig/network-scripts目录中查看ifcfg-ens32和ifcfg-kvmbr0的配置
[root@rhel78 ~]# brctl show
bridge name	bridge id		STP enabled	interfaces
kvmbr0		8000.000c2915aaa9	yes		ens32
virbr0		8000.fe5400780621	yes		vnet1

```



## 5. 虚拟机热添加设备

##### 5.1 添加硬盘

```shell
#创建要添加的磁盘文件
qemu-img create -f qcow2 kvmh1-add.qcow2 10
#附加至虚拟机
virsh attach-disk kvmh1 /data/kvmh1-add.qcow2 vdb [--config | --persistent]

#若要对虚拟机已有磁盘进行扩容，则
先umount,
再detach，
再qemu-img resize，
再attach
再xfg_growfs /dev/vdb
```



##### 5.2 添加网卡

```shell
virsh attach-interface web03 bridge virbr0 --model virtio [--config | --persistent]
virsh detach-interface web03 bridge --mac 52:54:00:d3:d4:93
```

##### 5.3 添加cpu，内存

```shell
qemu-img create -f qcow2 -b kvmh1.qcow2 web03.qcow2
#创建一个可以热插拔cpu,内存的虚拟机
virt-install --virt-type kvm --os-type linux --os-variant rhel7 --vcpus 1,maxvcpus=10 --memory 512,maxmemory=2048 --disk /opt/web03.qcow2 --network bridge=kvmbr0 --graphics vnc,listen=0.0.0.0 --name web03 --boot hd --noautoconsole
#添加内存
virsh setmem web03 1024M [--config]
#添加cpu
virsh setvcpus web03 2 [--config]
#针对已存在的虚拟机，也可修改配置文件，重启

```

## 6 热迁移

> nfs客户端配置hosts
>
> nfs服务器端，cat /etc/exports
>
> **/data 2.2.11.0/24(rw,async,no_root_squash,no_anno_squash)**
>
> systemctl start rpcbind nfs

```shell
#在kvm01和kvm02上面
showmount -e 2.2.11.80
mount -t nfs 2.2.11.80:/data /opt -o fsc
#迁移，可以用长Ping测试
virsh migrate --live --verbose web03 qemu+ssh://2.2.11.79/system --unsafe
```

## 7 kvm和ESXi

##### 7.1 迁移KVM虚拟机到ESXi中

```shell
#压缩qcow2磁盘空间
qemu-img convert -c -O qcow2 kvmh2.qcow2 kvmh2_c.qcow2

#在KVM主机中转换qcow2格式到vmdk，转换完发现和没压缩之前大小一样
#为了防止镜像被拆分为2GB的小块，需要增加compat6的选项
qemu-img convert -f qcow2 -O vmdk kvmh2_c.qcow2 kvmh2_tmp.vmdk -o compat6

#然后将转换后的镜像复制一份到esxi主机中,通过SSH登入esxi主机并进入相关目录
#在esxi主机里，使用vmkfstools命令进行格式转换,转换成精简备置的磁盘：
vmkfstools -i kvmh2_tmp.vmdk -d thin kvmh2.vmdk
#然后添加一个新设备，选择“现有磁盘”，并选择上一步转换的磁盘，完成后启动虚拟机即可


#查看kvm虚拟机信息
virsh dominfo kvmh2
```

##### 7.2 迁移ESXi虚拟机到KVM中

> 1. **将ESXi虚拟机导出为ova文件**
> 2. **将ova文件上传到KVM宿主机**
> 3. **使用virt-v2v命令，将ova文件转换为xml和qcow2格式的磁盘文件**

## 8 openstack

##### 8.1 安装

```shell
#准备两块网卡，配置/etc/hosts解析到各主机（controller和node1），配置chronyd

#安装openstack包
yum install https://rdoproject.org/repos/rdo-release.rpm
yum install python-openstackclient

#安装数据库
yum install mariadb mariadb-server python2-PyMySQL
```

> Create and edit the `/etc/my.cnf.d/openstack.cnf`

```ini
[mysqld]
bind-address = 2.2.11.78

default-storage-engine = innodb
innodb_file_per_table = on
max_connections = 4096
collation-server = utf8_general_ci
character-set-server = utf8
```

```shell
mysql_secure_installation
```

> 安装message quene

```shell
yum install rabbitmq-server
systemctl start rabbitmq-server.service
rabbitmqctl add_user openstack 123456
rabbitmqctl set_permissions openstack ".*" ".*" ".*"
```

```sql
#创库授权
MariaDB [(none)]> CREATE DATABASE keystone;
MariaDB [(none)]> GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'%' \
IDENTIFIED BY '123456';
```

> **httpd通过mod_wsgi模块来连接python程序，就像nginx通过fastcgi来连接php一样**



