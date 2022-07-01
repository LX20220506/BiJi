# Tomcat的使用

## 什么是Tomcat

Tomcat用来装载javaweb程序，可以称它为web容器，jsp/servlet程序需要运行在Web容器上，Web容器有很多种，JBoss、WebLogic等等，Tomcat是其中一种。和Asp.Net中的IIS服务器一样

## 安装

去下载和JDK版本匹配的Tomcat，放入文件夹后，直接解压就可以使用

## 目录介绍

- bin：专门用来存放Tomcat服务器的可执行文件
- conf：专门用来存放Tomcat服务器的配置文件
- lib：专门用来存放Tomcat服务器的jar包
- logs：专门用来存放Tomcat服务器运行输出的日志信息
- temp：专门用来存放Tomcat运行时产生的临时数据
- webapps：专门存用来放部署的web工程。一个文件夹对应一个工程
- work：是tomcat工作时的目录，用来存放Tomcat运行时Jsp翻译为Servlet的源码，和Session钝化的目录。（钝化就是指序列化）

## 如何启动Tomcat服务器

1. 在安装Tomcat文件夹下的bin文件夹下，点击startup.bat，执行
2. 使用命令行，cd 到bin文件夹下，执行**catalina  run**

### 如何测试启动成功

启动后，在浏览器页面输入地址（8080端口为Tomcat默认端口）：

1. http://localhost:8080/
2. http://127.0.0.1:8080/
3. http://真实ip:8080/

## Tomcat的停止

1. 在bin文件夹下找到shutdown.bat，双击打开，就会关闭startup.bat窗口（推荐）
2. 在startup.bat窗口页面执行 Ctrl + C
3. 直接关闭startup.bat窗口

## 如何修改Tomcat的启动端口

1. 找到conf文件夹
2. 找到server.xml文件
3. 找到Connector标签
4. 修改Connector标签的port属性
   1. port默认为8080
   2. 可以选择1~65535；注意：1~1000一般由系统使用，推荐使用8000之后的端口

## Tomcat部署Web工程

> 输入的http://127.0.0.1:8080/就是直接将读取文件路径指向了webapps文件夹

一共有两种部署方式：

1. 直接复制

   1. 在webapps文件夹中新建文件夹Test
   2. 将项目文件拷贝到Test文件夹中
   3. 启动Tomcat后，在浏览器中输入http://localhost/8100/Test/Index.html；可以直接访问Test文件夹下的Index.html文件

2. xml配置访问

   1. 在webapps文件夹中新建文件夹Test

   2. 在Test文件夹下创建xml文件

   3. 配置xml文件 

      ~~~xml
      <!-- 
       Context表示创建一个工程上下文
       path表示工程访问的路径：/index
       docBase表示你的工程目录在哪里
      -->
      <Context path="/index" docBase="E:\book" />
      ~~~

      











