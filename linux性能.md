## 1.dstat

​	

```
--top-cpu-adv

--top-bio-adv

--tcp		//查看tcp状态

-cC 0	//显示0号CPU

-dD vda	//显示vda磁盘

-t	//显示时间
#显示网卡
-nf --net-packets
#显示内存
-m
#相当于vmstat
-v
```

​	

sysstat包安装后包含这些命令

sar

​	mpstat

​	pidstat

​	iostat

​	vmstat

## 2. mpstat

mpstat是MultiprocessorStatistics的缩写，是实时监控工具，报告与cpu的一些统计信息这些信息都存在/proc/stat文件中，在多CPU系统里，其不但能查看所有的CPU的平均状况的信息，而且能够有查看特定的cpu信息，mpstat最大的特点是:可以查看多核心的cpu中每个计算核心的统计数据；而且类似工具vmstat只能查看系统的整体cpu情况

​	

```
mpstat -P ALL 2 5	//显示所有CPU信息
mpstat -I			//显示中断信息
```



## 3. pidstat

​	

```
-C pattern interval count	//以正则过滤进程名，默认输出的是进程的CPU使用信息，可以再结合mpstat -P ALL

-d //显示进程的io信息

#显示指定进程对cpu的占用		
-p pid
```

​	

## 4. iostat

iostat命令用于输出CPU和磁盘I/O相关的统计信息

​	

```
-t	显示时间戳

-m	以mb单位显示

-x	extend format

-N	显示LVM

-p	若未划分lvm，则指定具体设备，如/dev/sda
```

主要查看

​	CPU信息中的

​		%iowait

​	Device信息中的

​		await	//平均IO请求等待时间，包括排队和处理的时间，单位毫秒

​		avgqu-sz //分配给设备的平均请求队列长度

​		%util	//IO请求被分配给设备后用掉的时间百分比，越高表示越接近饱和

## 5. sar

sar命令和dstat命令选项类似，都是可以进行多指标查询

​	-d	//查设备

​	-n Dev	//查网卡

-n NFS	//查看NFS client

​	-r		//查内存

​	-h		//看帮助

/usr/lib64/sa/sa1,sa2,sadc

/var/log/sa/sadd

nfsiostat 2 -d /nfs_tt		//这个命令只能显示op/s，其他值都是0

watch nfsstat

## 6. nethogs

```
#查找最占用网卡的进程
nethogs -i eth1
```

