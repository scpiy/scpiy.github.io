---
layout:     post
title:      Ubuntu下使用IDEA部署tomcat容器内的Java web Application
subtitle:   
date:       2020-09-24
author:     scpi
header-img: img/post-2020-09-24.jpg
catalog:	true
tags:
    - java
    - jsp
---



### Ubuntu下使用IDEA部署tomcat容器内的Java web Application

---

> 操作系统为Ubuntu18.04,IDEA版本为2020.2,docker版本为19.03.13

#### 安装并运行docker

参考[Install Docker Engine on Ubuntu](https://docs.docker.com/engine/install/ubuntu/)，首先删除系统中已有的docker,使用

```shell
$ sudo apt-get remove docker docker-engine docker.io containerd runc
```

一般来讲建议的安装方法是添加Docker的仓库并安装，有其他需求的可以参考上面的链接。

更新并允许apt命令使用基于HTTPS协议的仓库：

```shell
$ sudo apt-get update

$ sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
```

添加Docker的GPG密匙

```shell
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

使用下面的命令并检查输出

```shell
$ sudo apt-key fingerprint 0EBFCD88
```

如果没有问题，你应该可以看到类似的输出：

```shell
pub   rsa4096 2017-02-22 [SCEA]
      9DC8 5822 9FC7 DD38 854A  E2D8 8D81 803C 0EBF CD88
uid           [ unknown] Docker Release (CE deb) <docker@docker.com>
sub   rsa4096 2017-02-22 [S]
```

着重检查这些字符序列是否正确`9DC8 5822 9FC7 DD38 854A E2D8 8D81 803C 0EBF CD88`

一般PC的cpu都是x86_64/amd64架构，使用下面的命令添加仓库：

```shell	
$ sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

> 这个仓库是稳定版的仓库，如果有其他需求依旧是参考上面给出的链接，基于其他架构的cpu的添加仓库的命令也可以在上面的链接里找到。

最后就是安装Docker了，使用下面的命令安装最新版本：

```shell
$ sudo apt-get update
 $ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

> 如果需要特定的版本，请参考上面的链接

最后之后的最后是一个小小的测试，测试是否安装成功，运行下面的命令并等待一段时间：

```shell
$ sudo docker run hello-world
```

> 这个命令会下载一个测试用image并在一个容器里运行他，如果安装成功，他会在下载完成后输出一条信息并退出

输出的信息看起来像是这个样子：

```shell
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```

如果你看到了上面的输出，那么恭喜你已经安装好了Docker,这时Docker deamon应该已经在后台运行了，但你也可以通过`sudo docker version`这条命令来检查是否docker deamon已经在运行，如果这条命令同时打印了Client端信息与Server端信息，则说明docker deamon已经确实运行在后台了。

但很遗憾关于docker的工作还没有完成，不过剩下的工作已经很简单了，尽管之前的工作也很简单。

你可能需要将你目前的用户加入docker用户组，这样你在使用docker时就不必加`sudo`前缀，这并不仅仅是为了方便，如果你不这么做，在之后IDEA中连接Docker时可能会出现无法连接的错误，原因大概是因为权限不足，感兴趣的可以自己试试。

为了避免这个问题，所需要做的事情仅仅是将你目前的用户加入docker用户组，使用下面的命令即可：

```shell
$ sudo groupadd docker
```

> 上面的命令可能出现"用户组已存在"的错误，这是因为docker在安装完成后会自动创建docker用户组，这个错误可以忽视不管，继续下一步

```shell
$ sudo usermod -aG docker $USER
```

之后退出登录再重新登录即可。我是重启了一下电脑。

检测是否成功，可以执行

```shell
$ docker run hello-world
```

如果输出正确的信息，则没有问题。

#### 在IDEA中配置docker

参考自[Docker](https://www.jetbrains.com/help/idea/docker.html)。

> 这个页面是IDEA的帮助文档一类的东西，没必要全看完，只看自己需要的部分就好，对我来说就是"Docker"这一点东西。

在setting中找到`Build, Execution, Deployment | Docker`

[![wvjJTP.md.png](https://s1.ax1x.com/2020/09/23/wvjJTP.md.png)](https://imgchr.com/i/wvjJTP)

这是我的界面，你在未添加docker之前是没有这个`Docker`的，点击那个Build下面的加号，选择`Unix socket`如果没有问题，那么几秒过后就会有`connection successful`的提示，就是我的界面里那个。

这样就添加完了。真快。

#### 部署Java Web Application

参考自[Deploy a Java web application inside a Tomcat server container](https://www.jetbrains.com/help/idea/deploying-a-web-app-into-an-app-server-container.html).

新建一个项目，在项目创建页面选择**Java Enterprise**，全部设置保持默认直接下一步。

[![wvxBaq.md.png](https://s1.ax1x.com/2020/09/23/wvxBaq.md.png)](https://imgchr.com/i/wvxBaq)

在**Libraries and Frameworks**列表里勾选上**Web Profile**，此时你可能发现一些其他东西被自动勾选了，不用管直接下一步。

输出你的项目的名称`DockerJavaWebApp`，点击Finish，项目创建完成。

在你的项目资源管理器里，就是显示你项目里文件的那个窗口里，找到`src/main/java`目录，右键新建`Servelt`文件，输入名字`MyServlet`，完成。

打开你新建的`MyServlet`文件，将默认存在的`doGet()`函数改为下面的内容，复制粘贴就行。

```java
protected void doGet(HttpServletRequest request, HttpServletResponse response) throws ServletException, IOException {
    response.setContentType("text/html");
    response.setCharacterEncoding("UTF-8");

    try (PrintWriter writer = response.getWriter()) {
        writer.println("<!DOCTYPE html><html>");
        writer.println("<head>");
        writer.println("<meta charset=\"UTF-8\" />");
        writer.println("<title>MyServlet.java:doGet(): Servlet code!</title>");
        writer.println("</head>");
        writer.println("<body>");

        writer.println("<h1>This is a simple java servlet.</h1>");

        writer.println("</body>");
        writer.println("</html>");
    }
}
```

import依赖包，因为我开了自动导入，不知道需要导入什么包，总之就是哪里报错就import对应的包。

再去项目资源管理器里找到`src/main/webapp`目录，新建`JSP/JSPX`文件，输入名字`index`，点击OK。

打开`index.jsp`文件并将其中的内容更换为下面的内容，依旧是复制粘贴：

```java
<%@ page contentType="text/html;charset=UTF-8" language="java" %>
<html>
    <head>
        <title>Hello, I am a Java web app!</title>
    </head>
    <body>
        <h1>Simple Java Web App Demo</h1>
        <p>To invoke the java servlet click <a href="MyServlet">here</a></p>
    </body>
</html>
```

之后打开`src/main/webapp/WEB-INF/web.xml`文件并将其中的内容更换为下面的内容，复制粘贴。**注意你的servlet文件的名字要与此文件中的名字对应**。

```java
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_3_1.xsd"
         version="3.1">

    <servlet>
        <servlet-name>MyServlet</servlet-name>
        <servlet-class>MyServlet</servlet-class>
    </servlet>

    <servlet-mapping>
        <servlet-name>MyServlet</servlet-name>
        <url-pattern>/MyServlet</url-pattern>
    </servlet-mapping>

</web-app>
```

打开**Build**菜单并选择**Rebuild Project**以确保你的servlet被正确的编译。如果一切正常，**Event Log** 中应该输出`Build completed successfully`。

如果你找不到Build在哪，不妨和我一样连按两次`shift`，在出现的导航栏中输入`Rebuild　Project`，选中该项后回车。

在**Build**菜单中选中**Build Artifacts**，再在出现的界面中选择 **DockerJavaWebApp:war**。

好了，接下来就是最后步骤了，终于要结束了。区区一个hello world。

打开`Services`窗口，其位置在**View | Tool Windows | Services** ，或者你可以试试快捷键`Alt+8`。

如果你前面的步骤没有问题，这个时候你应该可以看到一个`Docker`在`Services`窗口里。

你应该可以看到一个`images`目录在他下面，如果没有的话，你可能需要双击一下`Services`中的`Docker`。

右键`images`，选择**Pull image**，在输入框中输入`tomcat`，并按下`Ctrl+Enter`。

接下来又是一段下载时间，不过很快。

下载完成后，在`images`中选中`tomcat:latest`并右键他，选择**Create container**。

在出现的配置页中，需要你填写的内容如下:

* configuration的名字：`TomcatConfig`
* container的名字：`TomcatContainer`
* bind ports那一栏这么填：

[![wxPlZQ.md.png](https://s1.ax1x.com/2020/09/23/wxPlZQ.md.png)](https://imgchr.com/i/wxPlZQ)

> 需要点击后面那个文件夹图标进行配置，不能直接填写，你照我的数据填就可以。

* bind mounts那一栏同理，只不过两个值分别是`[PROJECT_PATH]/target`和`/usr/local/tomcat/webapps`，第一个值是`你的项目目录/target`，例如我的这个值是`/home/scpi/DockerJavaWebApp/target`，你的这个值和我的是不一样的。

完成之后点击`Run`即可，如果一切正常你应该可以看到`Build log`的正常输出。

此时访问http://127.0.0.1:8080/DockerJavaWebApp-1.0-SNAPSHOT/这个地址就可以看到你的程序了。

终于结束了，区区一个Hello world。