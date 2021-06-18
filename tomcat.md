## 1. 安装

windows版下载.zip包，解压；需要安装java，设置JAVA_HOME环境变量



> 安装成服务，可以到bin文件夹下执行

```
service.bat install service_name
```

   或者直接在bin目录下用start.bat启动



> 控制台日志乱码，修改conf\logging.properties

```
java.util.logging.ConsoleHandler.encoding = GBK
```



> 静态html页面乱码

1.    查看一下网页文件中的meta属性中是否设置了编辑，用UTF-8
2. 用notepad将网页文件打开，转换成ANSI编码



## 2. Tomcat进入manager管理页面

### 2.1 conf/tomcat-users.xml添加如下内容，不是在行尾添加

```xml
<role rolename="admin-gui"/>

<role rolename="manager-gui"/>

<role rolename="manager-jmx"/>

<role rolename="manager-script"/>

<role rolename="manager-status"/>

<user username="admin" password="admin" roles="admin-gui,manager-gui,manager-jmx, manager-script,manager-status"/>

```



### 2.2 修改webapps/manager/META-INF/context.xml替换如下内容

```
 <Valve className="org.apache.catalina.valves.RemoteAddrValve"
 	allow="^.*$" />

```

