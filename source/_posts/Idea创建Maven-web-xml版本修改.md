---
title: Idea创建Maven web.xml版本修改
date: 2021-07-13 22:29:52
tags: springmvc
---

一、问题描述
1.在使用Maven创建web工程的时候发现默认web.xml版本居然是2.4的，这个版本连EL表达式都用不了，所以很是糟心

2.所以为了解决Idea创建Maven Web工程的web.xml版本问题，给大家提供了两种解决办法

二、问题分析
1.首次创建Maven工程，会联网下载web有关的jar包，其中最重要的一个就是我们创建工程的时候选择的maven-archetype-webapp-1.3.jar这个jar包

2.在IdeanMaven web项目中生成的web.xml文件就是从该jar包中拷贝出来的，所以我们要做的就是改动web.xml和此jar包

三、问题解决
3.1 暂时解决
1.暂时解决方法只能解决当前项目，新建一个项目还会出现这个问题

2.要做的就是将项目的web.xml头换成需要的版本，比如我换成4.0版本

将需要的版本头替换原来的2.4版本头

需要4.0头的可以直接在这里复制
`<?xml version="1.0" encoding="UTF-8"?>`
一、问题描述
1.在使用Maven创建web工程的时候发现默认web.xml版本居然是2.4的，这个版本连EL表达式都用不了，所以很是糟心



2.所以为了解决Idea创建Maven Web工程的web.xml版本问题，给大家提供了两种解决办法

二、问题分析
1.首次创建Maven工程，会联网下载web有关的jar包，其中最重要的一个就是我们创建工程的时候选择的maven-archetype-webapp-1.3.jar这个jar包

2.在IdeanMaven web项目中生成的web.xml文件就是从该jar包中拷贝出来的，所以我们要做的就是改动web.xml和此jar包

三、问题解决
3.1 暂时解决
1.暂时解决方法只能解决当前项目，新建一个项目还会出现这个问题

2.要做的就是将项目的web.xml头换成需要的版本，比如我换成4.0版本

将需要的版本头替换原来的2.4版本头

需要4.0头的可以直接在这里复制
`<?xm`l version="1.0" encoding="UTF-8"?>`
`<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">`
    
</web-app>`

3.不建议随便粘贴一个头替换原来的web.xml头，最好是根据自己的服务器如Tomcat的版本来替换，推荐从Tomcat服务器中的web.xml中把头部分粘贴过来进行替换或者直接将web.xml文件拷贝过来替换为原来的web.xml，图示

![img](https://magickio.oss-cn-shenzhen.aliyuncs.com/img/20210714233506.png)

在Tomcat安装目录下的webapp/ROOT/WEB-INF中有我们需要的web.xml

把不需要的部分删除，就可以得到我们需要的部分，也可以不删除，没什么影响
`<?xm`l version="1.0" encoding="UTF-8"?>`
`<web-app xmlns="http://xmlns.jcp.org/xml/ns/javaee"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/javaee http://xmlns.jcp.org/xml/ns/javaee/web-app_4_0.xsd"
         version="4.0">`
    
</web-app>`
4.重启Idea 的服务器，如Tomcat，问题解决

3.2 永久解决
1.上述方法只能解决一个项目问题，但是我们并不想每次创建web项目都要像上面一样，很麻烦，所以我们这里永久性解决

2.我们创建web项目的时候发现使用:分隔了一个路径和jar包名称，前者其实就是Maven仓库坐标，后者就是web项目核心jar包

![img](https://magickio.oss-cn-shenzhen.aliyuncs.com/img/20210714233512.png)

3.根据提供的坐标（路径）找到maven-archetype-webapp这个jar包

我的路径：d:\maven\MavenRepository\.m2\repository\org\apache\maven\archetypes\maven-archetype-webapp\1.3\

![img](https://magickio.oss-cn-shenzhen.aliyuncs.com/img/20210714233514.png)


4.我们使用压缩软件打开这个jar包，注意是打开而不是解压，如使用2345好压打开，依次进入以下路径到WEB-INF目录中就可以看到有一个web.xml文件

选择打开

依次进入maven-archetype-webapp-1.3.jar\archetype-resources\src\main\webapp\WEB-INF目录中，找到web.xml

![img](https://magickio.oss-cn-shenzhen.aliyuncs.com/img/20210714233517.png)

![img](https://magickio.oss-cn-shenzhen.aliyuncs.com/img/20210714233519.png)


5.双击打开，注意不是解压！将此web.xml的头内容替换为我们需要的头信息（也可以直接删除这个web.xml，然后直接从Tomcat安装目录下的webapp/ROOT/WEB-INF中将web.xml给复制过来替换原来的web.xml），两种方式都行

![img](https://magickio.oss-cn-shenzhen.aliyuncs.com/img/20210714233522.png)

6.修改完成，保存，然后关闭打开的文件，这个时候压缩软件会提示信息已经改变，是否重新压缩，选择是，修改完成

7.重新创建web工程，出现的web.xml就是我们刚刚修改的web.xml
