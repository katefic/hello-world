##### 1.安装

```shell
#卸载掉系统自带的旧版本，通过源码编译安装，也可以IUS库中下载新的rpm包
yum install curl-devel expat-devel perl-CPAN  openssl-devel zlib-devel
make prefix=/usr/local/git all
make prefix=/usr/local/git install
添加man路径后，在man目录下查看一下提供的是哪些命令的手册
```

##### 2.安装完先生成配置文件

.git/config -> ~/.gitconfig (--global)-> /etc/gitconfig(--system)

```shell
[root@rhel78 ~]# git config --global user.name benben
[root@rhel78 ~]# git config --global user.email katefic@qq.com
[root@rhel78 ~]# git config --global -l
user.name=benben
user.email=katefic@qq.com
```

编辑 git 配置文件:

```
$ git config -e    # 针对当前仓库 
```



##### 3. 3种初始化本地仓库的方式

> 1.手动创建，自定义仓库名称

```shell
[root@rhel78 ~]# cd git_root
[root@rhel78 ~/git_root]# 
[root@rhel78 ~/git_root]# git init g1
Initialized empty Git repository in /root/git_root/g1/.git/
```

> 2.在已有目录中初始化

```shell
[root@rhel78 ~]# cd path/to/code 
[root@rhel78 ~/path/to/code]# git init
Initialized empty Git repository in /root/path/to/code/.git/
```

> 3.克隆远程

```
git clone git@github.com:fsliurujie/test.git         --SSH协议
git clone git://github.com/fsliurujie/test.git          --GIT协议
git clone https://github.com/fsliurujie/test.git      --HTTPS协议
```

##### 4.远程仓库

```shell
#这样是将远程的一个仓库添加到本地来访问，以后git push -u nnd master就是这样；还不能直接操作gitee来远程创建一个仓库
git remote add nnd git@gitee.com:tsg2/scripts
git push nnd master

#指定默认分支，以后可以直接git pull/push
git branch --set-upstream-to=nnd/master
```

git pull=git fetch + git merge

```shell
#从远程库获取
git fetch origin master
#切换到远程库
git checkout origin/master
#切回本地库
git checkout master
#合并
git merge origin/master
```







##### 5.分支

```shell
git branch 分支名->新建分支 或者

git checkout -b 分支名 ->直接创建并切换到新分支

#切换分支后，当前工作目录的内容会随着变化

#将指定分支合并到当前分支
git merge 分支名
```



##### 6.日志

```shell
git log --oneline
git reflog
git reset --hard index
```

##### 7.