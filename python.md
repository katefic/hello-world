## 1.模块导入

一个模块被另一个程序第一次引入时，其主程序将运行。如果我们想在模块被引入时，模块中的某一程序块不执行，我们可以用__name__属性来使该程序块仅在该模块自身运行时执行。

```python
#!/usr/bin/python3
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
>>> print('{}网址： "{}!"'.format('菜鸟教程', 'www.runoob.com'))
菜鸟教程网址： "www.runoob.com!"

```

str() repr() str.rjust() str.center()

### 3.2 input

```python
str = input("请输入：");
print ("你输入的内容是: ", str)

num = int(input('input a number: '))
```

### 3.3 读写文件

```python
open(filename, mode)
with open(filename, mode) as f:
    
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
