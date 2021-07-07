## 1.模块导入

一个模块被另一个程序第一次引入时，其主程序将运行。如果我们想在模块被引入时，模块中的某一程序块不执行，我们可以用__name__属性来使该程序块仅在该模块自身运行时执行。

```python
#!/usr/bin/python3
# -*- coding: utf-8 -*-
# Filename: using_name.py

if __name__ == '__main__':
   print('程序自身在运行')
else:
   print('我来自另一模块')
```

```python
import module_name	//之后通过module_name.unit_name来访问

from module_name import unit_name,...|*		//可以直接通过unit_name来访问
from module_name import unit_name as alias	//可以通过别名访问，但是模块本身没有被导入
```

**说明：** 每个模块都有一个__name__属性，当其值是'__main__'时，表明该模块自身在运行，否则是被引入。



## 2.函数

### 2.1 位置参数

传入时可以直接给值，按定义时的顺序传入

### 2.2 关键字参数

传入时指定名称，可以不按顺序

### 2.3 默认参数

调用函数时，如果没有传递参数，则会使用默认参数

**定义时，默认参数后面不能再跟非默认参数**

### 2.4 不定长参数

定义中参数前面有一个星号的，按元组传入

前面有两个星号的，按字典传入

### 2.5 lambda表达式

```python
#!/usr/bin/python3
 
# 可写函数说明
sum = lambda arg1, arg2: arg1 + arg2
 
# 调用sum函数
print ("相加后的值为 : ", sum( 10, 20 ))
print ("相加后的值为 : ", sum( 20, 20 ))
```

## 3.输入输出

### 3.1 str.format()

```python
#{:}，:左边指定输出列表的索引，右边指定填充、宽度和对齐
>>> print('{}网址： "{}!"'.format('菜鸟教程', 'www.runoob.com'))
菜鸟教程网址： "www.runoob.com!"

```

str() repr() str.rjust() str.center()

> 解决方法：
>
> 可以用中文空格填充，中文空格 chr(12288) 
>
>  print("{0[0]:<4}{0[1]:\u3000<20}{0[2]:<8}".format(line.split()))

### 3.2 input

```python
str = input("请输入：");
print ("你输入的内容是: ", str)

num = int(input('input a number: '))
```

### 3.3 读写文件

```python
open(filename, mode)
#向文件写入
with open(filename, mode) as f:
    f.write("")
    f.writeline(list)
    print("hello", file=f)
    
```

文件对象的方法：

​	read

​	readline

​	readlines

​	write

​	seek

## 4. 异常

![](D:\typora\images\try_except_else_finally.png)

## 5. 面向对象

在类的内部，使用 def关键字来定义一个方法，与一般函数定义不同，**类方法必须包含参数 self, 且为第一个参数**，self 代表的是类的实例。

### 类的私有属性

**__private_attrs**：两个下划线开头，声明该属性为私有，不能在类的外部被使用或直接访问

### 类的私有方法

**__private_method**：两个下划线开头，声明该方法为私有方法，只能在类的内部调用 ，不能在类的外部调用

### 类的专有方法：

- **__init__ :** 构造函数，在生成对象时调用

## 6. 内置函数

查看方法：

```python
dir(__builtins__)
help(open)

#查看模块内定义的函数
>>> import os
>>> help(os.open)

#查看类的说明
import shutil
help(shutil)
```



## 7. 常用模块

psutil

os

## 8. 其他

#### 8.1 字符串中带localhost会报错，换成127.0.0.1正常

## 9. pip

### 9.1 apk安装

```
cat /etc/os-release
cd /etc
sed -i 's/dl-cdn.alpinelinux.org/mirrors.ustc.edu.cn/g' /etc/apk/repositories
apk update
apk add py-pip

pip install --upgrade pip install -i https://pypi.tuna.tsinghua.edu.cn/simple docker-compose
```



### 9.2 yum安装

```
yum install python3-pip
#直接用下面的清华源更新
#python3 -m pip install --upgrade pip "pip < 21.0"
```



### 9.3 pypi 镜像使用帮助

pypi 镜像每 5 分钟同步一次。

### 临时使用

```
pip install -i https://pypi.tuna.tsinghua.edu.cn/simple some-package
```

注意，`simple` 不能少, 是 `https` 而不是 `http`

### 设为默认

升级 pip 到最新的版本 (>=10.0.0) 后进行配置：

```

[root@c81 ~]# python3 -m pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
Writing to /root/.config/pip/pip.conf


```

如果您到 pip 默认源的网络连接较差，临时使用本镜像站来升级 pip：

```
pip3 install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U
```