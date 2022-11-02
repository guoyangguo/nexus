# Nexus3搭建Maven私服

## 安装Nexus

### 下载Nexus

  访问[官网地址](https://www.sonatype.com/download-oss-sonatype) https://www.sonatype.com/download-oss-sonatype，需要填写一些个人信息，本文在Centos下进行安装，所以下载UNIX版本。

![image-20221101224241084](/Users/guoyang/work/nexus/img/image-20221101224241084.png)

### 安装

  将下载的好的安装包复制到`/usr/local/nexus`下

```shell
mkdir -p /usr/local/nexus && mv nexus-3.42.0-01-unix.tar.gz /usr/local/nexus/
```

解压进行安装,	nexus依赖于JDK8 ，请确定安装好JDK8

```shell
tar vxf nexus-3.42.0-01-unix.tar.gz
```

启动 ./bin/nexus start

```shell
root@master nexus-3.42.0-01]# ./bin/nexus start
WARNING: ************************************************************
WARNING: Detected execution as "root" user.  This is NOT recommended!
WARNING: ************************************************************
Usage: ./bin/nexus {start|stop|run|run-redirect|status|restart|force-reload}
```

因为nexus并不推荐使用root使用进行启动，可以更改nexus将 **RUN_AS_USER**改为**RUN_AS_USER=root**,在这里还是使用官方推荐的，创建nexus用户用来启动nexus

创建nexus用户

```shell
useradd nexus && chown -R nexus:nexus /usr/local/nexus/ 
切换nexus用户启动
su nexus
./bin/nexus run

# 当看到如下log说明nexus已经启动成功
2022-11-01 23:26:25,140+0800 INFO  [jetty-main-1] *SYSTEM org.eclipse.jetty.server.AbstractConnector - Started ServerConnector@4f379b39{HTTP/1.1, (http/1.1)}{0.0.0.0:8081}
2022-11-01 23:26:25,141+0800 INFO  [jetty-main-1] *SYSTEM org.eclipse.jetty.server.Server - Started @44087ms
2022-11-01 23:26:25,141+0800 INFO  [jetty-main-1] *SYSTEM org.sonatype.nexus.bootstrap.jetty.JettyServer - 
-------------------------------------------------

Started Sonatype Nexus OSS 3.42.0-01

-------------------------------------------------
```

访问nexus: http://localhost:8081/

![image-20221101232845128](/Users/guoyang/work/nexus/img/image-20221101232845128.png)



 点击**[Sign in]**进行登录，默认用户名 **admin** 默认密码在**/usr/local/nexus/sonatype-work/nexus3/admin.password**中，第一次登录需要修改密码。

## 配置maven私服

### 查看仓库信息

进入Browse页面查看当的仓库信息

![image-20221101234713805](/Users/guoyang/work/nexus/img/image-20221101234713805.png)

默认仓库说明

- maven-central maven中央仓库，默认从https://repo1.maven.org/maven2/拉取依赖
- maven-releases 私库发行版
- maven-snapshots 私库快照版
- maven-public 仓库分组，将上面三种仓库组合在一起对外提供服务

仓库类型

- proxy，代理类型，从远程中央仓库中寻找数据的仓库（可以点击对应的仓库的Configuration页签下Remote Storage属性的值即被代理的远程仓库的路径），如可配置阿里云maven仓库
- group，仓库组，用户仓库地址选择Group的地址，即可访问Group中配置的，用于方便开发人员自己设定的仓库。maven-public就是一个Group类型的仓库，内部设置了多个仓库，访问顺序取决于配置顺序，3.x默认Releases，Snapshots
- hosted，私有仓库，内部项目的发布仓库，专门用来存储我们自己生成的jar文件
-  3rd party：未发布到公网的第三方jar (3.x去除了)
-  Snapshots：本地项目的快照仓库
-  Releases： 本地项目发布的正式版本
- Central：中央仓库
- Apache Snapshots：Apache专用快照仓库(3.x去除了)

### 配置阿里云私服代理

1. create repostory

![image-20221101235657183](/Users/guoyang/work/nexus/img/image-20221101235657183.png)

2. 选择maven2(proxy)

![image-20221101235803096](/Users/guoyang/work/nexus/img/image-20221101235803096.png)

3. Name 填写 aliyun, URL输入：http://maven.aliyun.com/nexus/content/groups/public/

![image-20221102000129836](/Users/guoyang/work/nexus/img/image-20221102000129836.png)

4. 选择Configuration > Repository, 双击 maven-public, 在Group区域将aliyun移到右侧Members, 上移到maven-central的上面, 点击 Save。

   ![image-20221102000346240](/Users/guoyang/work/nexus/img/image-20221102000346240.png)