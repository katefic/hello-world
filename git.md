##### 1.安装

```shell
#卸载掉系统自带的旧版本，通过源码编译安装，也可以IUS库中下载新的rpm包
yum install curl-devel expat-devel perl-CPAN  openssl-devel zlib-devel gcc
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
git push --set-upstream nnd master
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

##### 7.删除untrack files

```
#显示要删除的文件和目录
git clean -nxfd
#执行删除
git clean -xfd
```

## 8. git 撤销，放弃本地修改

### 8.1 未使用 git add 缓存代码时

可以使用 

```
git checkout -- filename 
```

(比如： git checkout -- readme.md ，**不要忘记中间的 “--”** ，不写就成了检出分支了！！)。

放弃所有的文件修改可以使用

```
 git checkout . 
```

此命令用来放弃掉所有还没有加入到缓存区（就是 git add 命令）的修改：内容修改与整个文件删除。但是此命令不会删除掉刚新建的文件。因为刚新建的文件还没已有加入到 git 的管理系统中。所以对于git是未知的。自己手动删除就好了。



### 8.2 已经使用了 git add 缓存了代码

可以使用 

```
git reset HEAD filepathname
```

 （比如： git reset HEAD readme.md）来放弃指定文件的缓存，放弃所以的缓存可以使用

```
 git reset HEAD .
```

此命令用来清除 git 对于文件修改的缓存。相当于撤销 git add 命令所在的工作。在使用本命令后，本地的修改并不会消失，而是回到了如（一）所示的状态。继续用（一）中的操作，就可以放弃本地的修改。



### 8.3 已经用 git commit 提交了代码

可以使用

```
 git reset --hard HEAD^ 
```

来回退到上一次commit的状态。此命令可以用来回退到任意版本：

```
git reset --hard commitid
```

git log 可以查看请交历史记录