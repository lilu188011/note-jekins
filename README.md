

<font color=red size=5>文章链接</font>

~~~bash
https://gitee.com/fakerlove/jenkins
~~~



#  Jenkins 教程

# 1. 简介

## 1.1 介绍

> 　Jenkins是一个开源软件项目，是基于Java开发的一种持续集成工具，用于监控持续重复的工作，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。

我说下我以前开发的痛点，在一些中小型企业，每次开发一个项目完成后，需要打包部署，可能没有专门的运维人员，只能开发人员去把项目打成一个war包，可能这个项目已经上线了，需要把服务关，在部署到服务器上，将项目启动起来，这个时候可能某个用户正在操作某些功能上的东西，如果你隔三差五的部署一下，这样的话对用户的体验也不好，自己也是烦的很，总是打包拖到服务器上。希望小型企业工作人员学习一下，配置可能复杂，但是你配置好了之后，你只需要把代码提交到Git或者Svn上，自动构建部署，非常方便。有任何地方不懂的翻到最下方随时咨询我，想帮助更多的初学者共同一起努力成长！

![](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200505205327591-604014428-1622620648156.png)

## 1.2 环境准备

![image-20210510095331698](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510095331698.png)

### 1.2.1 安装jenkins

jenkins 镜像网站

~~~bash
https://mirrors.cloud.tencent.com/jenkins/
https://mirrors.huaweicloud.com/jenkins/
https://mirrors.tuna.tsinghua.edu.cn/jenkins/
http://mirrors.ustc.edu.cn/jenkins/
~~~

#### 1) 离线安装

~~~bash
yum install -y java-1.8.0-openjdk* -y
wget https://mirrors.cloud.tencent.com/jenkins/redhat-stable/jenkins-2.303.1-1.1.noarch.rpm
rpm -ivh jenkins-2.303.1-1.1.noarch.rpm
yum install jenkins
systemctl start jenkins
~~~

访问网站，即可出现网址

~~~bash
http://localhost:8080
~~~

查看应用状态

~~~bash
systemctl status jenkins
~~~

<font color=red size=4>可能会出现问题</font>

daemonize is needed by jenkins-2.303.1-1.1.noarch

~~~bash
yum  -y install epel-release
yum -y install daemonize
~~~



#### 2) docker 安装

安装docker

1.启动docker，下载Jenkins镜像文件

```
docker pull jenkins/jenkins
```



![img](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200506003956565-2129663266.png)

 2.创建Jenkins挂载目录并授权权限（我们在服务器上先创建一个jenkins工作目录 /var/jenkins_mount，赋予相应权限，稍后我们将jenkins容器目录挂载到这个目录上，这样我们就可以很方便地对容器内的配置文件进行修改。 如果我们不这样做，那么如果需要修改容器配置文件，将会有点麻烦，因为虽然我们可以使用docker exec -it --user root 容器id /bin/bash 命令进入容器目录，但是连简单的 vi命令都不能使用）

```
mkdir -p /var/jenkins_mount
chmod 777 /var/jenkins_mount
```

3.创建并启动Jenkins容器

> 　   -d 后台运行镜像
>
> 　　-p 10240:8080 将镜像的8080端口映射到服务器的10240端口。
>
> 　　-p 10241:50000 将镜像的50000端口映射到服务器的10241端口
>
> 　　-v /var/jenkins_mount:/var/jenkins_mount /var/jenkins_home目录为容器jenkins工作目录，我们将硬盘上的一个目录挂载到这个位置，方便后续更新镜像后继续使用原来的工作目录。这里我们设置的就是上面我们创建的 /var/jenkins_mount目录
>
> 　　-v /etc/localtime:/etc/localtime让容器使用和服务器同样的时间设置。
>
> 　　--name myjenkins 给容器起一个别名



```bash
docker run -d -p 10240:8080 -p 10241:50000 -v /var/jenkins_mount:/var/jenkins_home -v /etc/localtime:/etc/localtime --name myjenkins jenkins/jenkins
```

 

![img](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200506010532861-1239060303.png)

 自己的jenkins

~~~bash
mkdir ~/jenkins
mkdir  ~/jenkins/jenkins_mount
mkdir  ~/jenkins/etc
mkdir  ~/jenkins/maven
docker run -it -d -p 10240:8080 -p 10241:5000 -u root -v ~/jenkins/jenkins_mount:/var/jenkins_home -v /etc/localtime:/etc/localtime -v ~/jenkins/maven:/var/maven -v ~/jenkins/etc/:/etc/  --name myjenkins jenkins/jenkins:lts
~~~

 4.查看jenkins是否启动成功，如下图出现端口号，就为启动成功了

```
docker ps -l
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200506011320515-2141868163.png)

 5.查看docker容器日志。

```
docker logs myjenkins
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200506011426781-1187586218.png)

 6.配置镜像加速，进入 cd /var/jenkins_mount/ 目录。

```
cd /var/jenkins_mount/
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200506011630329-741219630.png)

**修改 vi hudson.model.UpdateCenter.xml里的内容**

**修改前**

![img](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200506012036877-994910766.png)

将 url 修改为腾讯云官方镜像：

~~~
https://mirrors.cloud.tencent.com/jenkins/updates/update-center.json
~~~

**修改后**

![img](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200506012119311-1420562634.png)



配置国内镜像源

~~~bash
mv /etc/apt/sources.list /etc/apt/sources.list.bak
echo "deb http://mirrors.163.com/debian/ jessie main non-free contrib" >/etc/apt/sources.list
echo "deb http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
echo "deb-src http://mirrors.163.com/debian/ jessie main non-free contrib" >>/etc/apt/sources.list
echo "deb-src http://mirrors.163.com/debian/ jessie-proposed-updates main non-free contrib" >>/etc/apt/sources.list
apt-get update
~~~





#### 3) 访问jenkins

1. 访问Jenkins页面，输入你的ip加上10240

![img](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200506012226430-1099181802.png)



2. 管理员密码获取方法，编辑initialAdminPassword文件查看，把密码输入登录中的密码即可，开始使用。

```
vi /var/jenkins_mount/secrets/initialAdminPassword
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200506013101851-1902660911.png)

 

3. 到此以全部安装成功，尽情的使用吧！

![img](https://gitee.com/fakerlove/picture_1/raw/master/1578696-20200506013252174-1483206896.png)

 

### 1.2.2 安装gitlab

#### 一、安装及配置

##### 1.gitlab镜像拉取

```ruby
# gitlab-ce为稳定版本，后面不填写版本则默认pull最新latest版本
$ docker pull gitlab/gitlab-ce
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-c866f117b53fadf8.png)

拉取镜像

##### 2.运行gitlab镜像

```csharp
docker run -d  -p 443:443 -p 80:80 -p 222:22 --name gitlab --restart always -v /home/gitlab/config:/etc/gitlab -v /home/gitlab/logs:/var/log/gitlab -v /home/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
# -d：后台运行
# -p：将容器内部端口向外映射
# --name：命名容器名称
# -v：将容器内数据文件夹或者日志、配置等文件夹挂载到宿主机指定目录
```

运行成功后出现一串字符串



![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-5818ed22c0bc1ee7.png)

运行成功

~~~bash
docker run -d  -p 443:443 -p 8088:80 -p 222:22 --name gitlab --restart always -v ~/gitlab/config:/etc/gitlab -v ~/gitlab/logs:/var/log/gitlab -v ~/gitlab/data:/var/opt/gitlab gitlab/gitlab-ce
~~~



##### 3.配置

按上面的方式，gitlab容器运行没问题，但在gitlab上创建项目的时候，生成项目的URL访问地址是按容器的hostname来生成的，也就是容器的id。作为gitlab服务器，我们需要一个固定的URL访问地址，于是需要配置gitlab.rb（宿主机路径：/home/gitlab/config/gitlab.rb）。

```ruby
# gitlab.rb文件内容默认全是注释
vim /home/gitlab/config/gitlab.rb
```

配置文件

```ruby
# 配置http协议所使用的访问地址,不加端口号默认为80
external_url 'http://192.168.199.231'

# 配置ssh协议所使用的访问地址和端口
gitlab_rails['gitlab_ssh_host'] = '192.168.199.231'
gitlab_rails['gitlab_shell_ssh_port'] = 222 # 此端口是run时22端口映射的222端口
:wq #保存配置文件并退出
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-a480f0de6409620a.png)

修改gitlab.rb文件

```ruby
# 重启gitlab容器
docker restart gitlab
```

此时项目的仓库地址就变了。如果ssh端口地址不是默认的22，就会加上ssh:// 协议头
 打开浏览器输入ip地址(因为我的gitlab端口为80，所以浏览器url不用输入端口号，如果端口号不是80，则打开为：ip:端口号)

##### 4.创建一个项目

第一次进入要输入新的root用户密码，设置好之后确定就行

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-6b04cfddeccf17bf.png)

gitlab页面

下面我们就可以新建一个项目了，点击Create a project

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-2a40551dc13c2826.png)

Create a project

创建完成后：



![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-0dd085723ef61677.png)

创建完成！

#### 二、用户使用

##### 1.下载git.exe

双击git.exe安装git（一直点下一步，直到完成）
 点击电脑桌面空白地方右键看到如下两行即安装成功

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-7bbdee06b7ca7711.png)

image.png

##### 2.登录gitlab网页

> **url**：http://192.168.1.111
>  填写账号密码登录

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-249a984d541801a1.png)

登录页面

##### 3.设置ssh

1.打开本地git bash,使用如下命令生成ssh公钥和私钥对

```ruby
ssh-keygen -t rsa -C 'xxx@xxx.com'
```

然后一路回车(-C 参数是你的邮箱地址)

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-40be69316850b690.png)

生成密匙

2.然后输入命令：

```ruby
# ~表示用户目录，比如我的windows就是C:\Users\Administrator，并复制其中的内容
cat ~/.ssh/id_rsa.pub
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-b71993bc58477957.png)

公匙

3.打开gitlab,找到Profile Settings-->SSH Keys--->Add SSH Key,并把上一步中复制的内容粘贴到Key所对应的文本框

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-d14c5051911fe20c.png)

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-f7319dc3a9d83828.png)

添加公匙到gitlab

##### 4.从gitlab克隆代码

1.回到gitlab页面点击projects->your projects

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-0105a84b02d1ed9e.png)

2.选择一个需要克隆的项目，进入

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-7a7f3af7efcff81c.png)

我的项目页面

3.点击按钮复制地址

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-27e842b185a62e69.png)

复制ssh地址

4.新建一个文件夹，我在这里在我的电脑D盘下新建project文件夹

![img](https:////upload-images.jianshu.io/upload_images/15087669-6c9571d4d0a5d321.png?imageMogr2/auto-orient/strip|imageView2/2/w/171/format/webp)

5.进入projects文件夹右键选择->Git Bash Here

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-e59886b042c02edf.png)

点击Git Bash Here

6.设置用户名和邮箱



```csharp
git config --global user.name "你的名字"
git config --global user.email "你的邮箱"
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-9592daf5642e3c11.png)

设置名字和邮箱

7.克隆项目

```bash
git clone 项目地址
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-dc8bafb214fa578e.png)

克隆项目

8.查看projects文件夹，项目已经克隆下来了

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-94ec492febe71420.png)

项目目录

##### 5.提交代码到gitlab

1.基于以上步骤，在克隆的项目文件夹下新增一个测试文件

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-c230295d84a79064.png)

新增txt文件

2.查看同步状态
 在项目文件夹下右键点击->Git Bash Here

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-1fd767543b22d445.png)

输入

```ruby
git status
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-364b83d323ceb46b.png)

状态

可以看到红色部分有需要提交的文件
 3.提交代码
 输入

```csharp
git add  测试提交的文件.txt
```

(“git add“后加“.”则添加全部文件，也可以加"*.txt"表示添加全部需要提交的txt文件 )

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-2e3e95ecf96cc668.png)

add需要提交的文件

然后输入以下命令提交并添加提交信息

```ruby
git commit -m "message"
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-6d689c385f30fe33.png)

commit

最后输出以下命令提交到gitlab

```ruby
git push origin master
```

![img](https://gitee.com/fakerlove/picture_1/raw/master/15087669-889169ad267f2997.png)

### 1.2.3 gitlab占用内存太多问题

修改/etc/gitlab/gitlab.rb

```
unicorn['worker_processes'] = 2
postgresql['max_worker_processes'] = 4
nginx['worker_processes'] = 4
```

重新运行

```
gitlab-ctl reconfigure  #gitlab会读取配置文件参数并修改各个插件的配置文件去（我猜的）
gitlab-ctl restart
```



# 2. 入门

## 2.1 安装中文插件

![image-20210510095033219](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510095033219.png)



## 2.2 角色控制Role-Based Strategy



![image-20210510102844184](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510102844184.png)

点击"Manage Roles"

![image-20210510102901425](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510102901425.png)

进行角色的管理

![image-20210510102914324](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510102914324.png)



## 2.3 安装凭证





## 2.4 jenkins拉取项目

### 2.4.1 拉取gitlab 项目

#### 1. 设置全局凭据

![image-20210510103928691](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510103928691.png)

点击添加凭据

![image-20210510104120595](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510104120595.png)

输入Gitee 账号、密码 ID可以任意填写，描述框可以根据项目来撰写
![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172004897.png)

类型项一定要选择

![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172017359.png)

#### 2. 新建Item

Jenkins首页—Item—新建
也可以进入到已有的Item里，再新建一个Item

![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172039419.png)

输入项目名称，一般用项目名称来命名，然后选择流水线，确定

![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172051784.png)

项目创建完毕

![image-20210510104546158](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510104546158.png)

#### 3. 具体任务配置

Jenkins首页—相应Item—配置----流水线—址—生成流水线脚本

![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172112844.png)

\#去gitee克隆代码地址

![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172126126.png)

配置在Jenkins上

![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210210201846492.png)

![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172145431.png)

复制脚本

![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172156393.png)

\#这下面是配置构建脚本，注意Jenkins支持任何语言编写的项目的构建
![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172206281.png)

粘贴需要构建的代码链接和要执行的ssh命令，建议每次构建都加上ls -l 验证是否成功，应用 保存

![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172221619.png)

#### 4、构建验证

进入任务，点击立即构建，成功的话会显示绿色，可以在Build History验证是否成功

![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172234687.png)

点击任务，进入任务页面，选择控制台输出，可以看到构建的详细日志，如果构建失败，可以在日志中查看失败原因：
![在这里插入图片描述](https://gitee.com/fakerlove/picture_1/raw/master/20210125172250520.png)

### 2.4.2 拉取gitee 项目

#### 1. 安装gitee 插件

![image-20210510104930930](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510104930930.png)

#### 2. 添加Gitee链接配置

前往 Jenkins -> Manage Jenkins -> Configure System -> Gitee Configuration -> Gitee connections

![image-20210510105938146](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510105938146.png)

点击进行数据的配置

![image-20210510105954742](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510105954742.png)

在 `Connection name` 中输入 `Gitee` 或者你想要的名字

`Gitee host URL` 中输入Gitee完整 URL地址： `https://gitee.com` （Gitee私有化客户输入部署的域名）

`Credentials`中如还未配置Gitee APIV5 私人令牌，点击Add ->Jenkins

> 1. `Domain` 选择 `Global credentials`
> 2. `Kind` 选择 `Gitee API Token`
> 3. `Scope` 选择你需要的范围
> 4. `Gitee API Token` 输入你的Gitee私人令牌，获取地址：https://gitee.com/profile/personal_access_tokens
> 5. `ID`, `Descripiton` 中输入你想要的 ID 和描述即可。

![image-20210510110612637](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510110612637.png)

1. `Credentials` 选择配置好的 Gitee APIV5 Token

2. 点击 `Advanced` ，可配置是否忽略 SSL 错误（视您的Jenkins环境是否支持），并可设置链接测超时时间（视您的网络环境而定）

3. 点击 `Test Connection` 测试链接是否成功，如失败请检查以上 3，5，6 步骤。

![image-20210510110655803](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510110655803.png)

配置成功后如图所示：
![Gitee链接配置](https://gitee.com/fakerlove/picture_1/raw/master/185651_68707d16_58426.png)

#### 3. 新建构建任务

前往 Jenkins -> New Item , name 输入 'Gitee Test'，选择 `Freestyle project` 保存即可创建构建项目。

#### 4. 任务全局配置

任务全局配置中需要选择前一步中的Gitee链接。前往某个任务（如'Gitee Test'）的 Configure -> General，Gitee connection 中选择前面所配置的Gitee链接，如图：

![image-20210510111209353](https://gitee.com/fakerlove/picture_1/raw/master/image-20210510111209353.png)





## 2.5 安装maven





## 2.6 安装tomcat

安装jdk

~~~bash
yum install -y java-1.8.0-openjdk* -y
~~~

安装tomcat

~~~
mkdir tomcat
cd tomcat
wget https://mirrors.tuna.tsinghua.edu.cn/apache/tomcat/tomcat-10/v10.0.6/bin/apache-tomcat-10.0.6.tar.gz
tar -zxvf apache-tomcat-10.0.6.tar.gz
~~~

编辑用户信息

~~~bash
vim apache-tomcat-10.0.6//conf/tomcat-users.xml
~~~

设置用户信息

~~~bash
<tomcat-users>
   <role rolename="tomcat"/> 
   <role rolename="role1"/>
   <role rolename="manager-script"/> 
   <role rolename="manager-gui"/>
   <role rolename="manager-status"/>
   <role rolename="admin-gui"/>
   <role rolename="admin-script"/> 
   <user username="tomcat" password="tomcat" roles="manager-gui,manager- script,tomcat,admin-gui,admin-script"/> 
</tomcat-users>
~~~

开放外网访问权限

~~~bash
vim webapps/manager/META-INF/context.xml
~~~

注释信息

![image-20210518154402642](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518154402642.png)

~~~bash
<!-- 
<Valve className="org.apache.catalina.valves.RemoteAddrValve" allow="127\.\d+\.\d+\.\d+|::1|0:0:0:0:0:0:0:1" /> 
-->
~~~

修改端口号--看需求

~~~bash
vim conf/server.xml
~~~

修改完配置为

![image-20210518154843683](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518154843683.png)



开启tomcat

~~~bash
./startup.sh
~~~

![image-20210518155025222](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518155025222.png)



输入tomcat,密码tomcat

![image-20210518155050999](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518155050999.png)

进入管理界面

![image-20210518155250939](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518155250939.png)



# 3. jenkins 风格类型



## 3.1 自由风格类型

拉取代码->编译->打包->部署

### 3.1.1 拉取代码

![image-20210518160244635](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518160244635.png)

### 3.1.2 编译

构建->添加构建步骤->Executor Shell

![image-20210518160456620](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518160456620.png)

~~~bash
echo "开始编译和打包" 
mvn clean package 
echo "编译和打包结束"
~~~

![image-20210518160557558](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518160557558.png)



这个时候会报错，

![image-20210518161038144](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518161038144.png)

解决方法

~~~bash
echo $PATH
~~~

复制path 路径，点击全局配置，添加全局属性

![image-20210518161757355](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518161757355.png)

### 3.1.3 部署

把项目部署到远程的Tomcat里面

安装 Deploy to container插件

点击构建后操作

![image-20210518162545887](https://gitee.com/fakerlove/picture_1/raw/master/image-20210518162545887.png)

添加tomcat 的凭证

![image-20210519101442247](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519101442247.png)

构件后操作

![image-20210519101503486](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519101503486.png)

就可以打包成功

## 3.2 Maven 风格类型

### 3.2.1 安装Maven Integration插件

![image-20210519111628230](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519111628230.png)

### 3.2.2 创建Maven项目

![image-20210519111646577](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519111646577.png)



### 3.2.3 配置项目

![image-20210519111712992](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519111712992.png)



## 3.3 流水线风格类型

> Pipeline 脚本是由 **Groovy** 语言实现的，但是我们没必要单独去学习 Groovy
>
> Pipeline 支持两种语法：**Declarative**(声明式)和 **Scripted Pipeline**(脚本式)语法
>
> Pipeline 也有两种创建方法：可以直接在 Jenkins 的 Web UI 界面中输入脚本；也可以通过创建一
>
> 个 Jenkinsfifile 脚本文件放入项目源码库中（一般我们都推荐在 Jenkins 中直接从源代码控制(SCM)
>
> 中直接载入 Jenkinsfifile Pipeline 这种方法）。



### 3.3.1 安装pipline 

![image-20210519160939109](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519160939109.png)



### 3.3.2 创建项目

![image-20210519161009364](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519161009364.png)



### 3.3.3 拉取代码

![image-20210519161033183](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519161033183.png)

自定义流水线



![image-20210519161120562](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519161120562.png)

构建流水线语法

![image-20210519162445601](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519162445601.png)

拉取代码

![image-20210519162647350](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519162647350.png)



生成pipline 代码

![image-20210519162806786](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519162806786.png)

构件代码

~~~
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '1527f9a8-f6ed-41b3-93a7-c05a88f36de9', url: 'https://gitee.com/fakerlove/clock']]])
            }
        }
          stage('build project') {
            steps {
               sh 'mvn clean package'
            }
        }
          stage("sa") {
            steps {
                echo 'goujian构件wancheng'
            }
        }
    }
    
}

~~~

### 3.3.4 构建成功

![image-20210519161212583](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519161212583.png)

## 3.4 jenkins 常见的触发器

jenkins 常见触发器

![image-20210519170109920](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519170109920.png)

其中gitee为安装了gitee 插件后，出现的触发

默认有4种

* 触发远程构建
* 轮训SCM
* 其他工程构建后触发
* 定时构建

### 3.4.1 gitee 触发器

gitee 设置签发文章

~~~bash
https://gitee.com/help/articles/4193#article-header9
~~~

在jenkins 上设置触发器，点击生成秘钥

![image-20210519171306631](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519171306631.png)

gitee设置webHook

![image-20210519171342064](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519171342064.png)



修改代码，提交至gitee,jenkins立即进行了刷新测试

![image-20210519171638974](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519171638974.png)

### 3.4.2 触发远程构建

设置token信息

![image-20210519171843116](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519171843116.png)

输入网址即可进行触发

~~~bash
http://192.168.66.101:8888/job/web_demo_pipeline/build?token=6666
~~~

### 3.4.3 其他工程创建触发

创建pre_job流水线工程

![image-20210519173346089](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519173346089.png)

配置需要触发的工程

![image-20210519173408602](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519173408602.png)

### 3.4.4 定时构建

![image-20210519173428203](https://gitee.com/fakerlove/picture_1/raw/master/image-20210519173428203.png)

> 每30分钟构建一次：H代表形参 H/30 * * * * 10:02 10:32
>
> 每2个小时构建一次: H H/2 * * *
>
> 每天的8点，12点，22点，一天构建3次： (多个时间点中间用逗号隔开) 0 8,12,22 * * *
>
> 每天中午12点定时构建一次 H 12 * * *
>
> 每天下午18点定时构建一次 H 18 * * *
>
> 在每个小时的前半个小时内的每10分钟 H(0-29)/10 * * * *
>
> 每两小时一次，每个工作日上午9点到下午5点(也许是上午10:38，下午12:38，下午2:38，下午
>
> 4:38) H H(9-16)/2 * * 1-5

### 3.4.5 参数化构建





## 3.5 邮件发送

### 3.5.1 安装插件

![image-20210520102548901](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520102548901.png)



### 3.5.2 开启邮箱支持

开启权限

![image-20210520103028041](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520103028041.png)

点击生成授权码

![image-20210520103307250](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520103307250.png)

发送配置短信，发送完成后，点击我已发送、获取密码

![image-20210520103323603](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520103323603.png)



### 3.5.3 配置

配置邮件管理员

![image-20210520105311778](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520105311778.png)

配置Extended

![image-20210520105355585](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520105355585.png)

配置 jenkins 默认邮箱

![image-20210520105543453](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520105543453.png)



点击测试

![image-20210520105600860](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520105600860.png)





在项目的根目录下创建email.html

~~~html
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>${ENV, var="JOB_NAME"}-第${BUILD_NUMBER}次构建日志</title>
</head>
<body leftmargin="8" marginwidth="0" topmargin="8" marginheight="4" offset="0">
<table width="95%" cellpadding="0" cellspacing="0"
       style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans- serif">
    <tr>
        <td>(本邮件是程序自动下发的，请勿回复！)</td>
    </tr>
    <tr>
        <td>
            <h2>
                <span style="color: #0000FF; ">构建结果 - ${BUILD_STATUS}</span>
            </h2>
        </td>
    </tr>
    <tr>
        <td><br/> <b><span style="color: #0B610B; ">构建信息</span></b>
            <hr size="2" width="100%" align="center"/>
        </td>
    </tr>
    <tr>
        <td>
            <ul>
                <li>项目名称&nbsp;：&nbsp;${PROJECT_NAME}</li>
                <li>构建编号&nbsp;：&nbsp;第${BUILD_NUMBER}次构建</li>
                <li>触发原因：&nbsp;${CAUSE}</li>
                <li>构建日志：&nbsp;<a href="${BUILD_URL}console">${BUILD_URL}console</a></li>
                <li>构建&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${BUILD_URL}">${BUILD_URL}</a></li>
                <li>工作目录&nbsp;：&nbsp;<a href="${PROJECT_URL}ws">${PROJECT_URL}ws</a></li>
                <li>项目&nbsp;&nbsp;Url&nbsp;：&nbsp;<a href="${PROJECT_URL}">${PROJECT_URL}</a></li>
            </ul>
        </td>
    </tr>
    <tr>
        <td><b><span style="color: #0B610B; ">Changes Since Last Successful Build:</span></b>
            <hr size="2" width="100%" align="center"/>
        </td>
    </tr>
    编写Jenkinsfile添加构建后发送邮件
    <tr>
        <td>
            <ul>
                <li>历史变更记录 : <a href="${PROJECT_URL}changes">${PROJECT_URL}changes</a></li>
            </ul>
            ${CHANGES_SINCE_LAST_SUCCESS,reverse=true, format="Changes for Build #%n:<br/>%c<br/>",showPaths=true,changesFormat="
            <pre>[%a]<br/>%m</pre>
            ",pathFormat="&nbsp;&nbsp;&nbsp;&nbsp;%p"}
        </td>
    </tr>
    <tr>
        <td><b>Failed Test Results</b>
            <hr size="2" width="100%" align="center"/>
        </td>
    </tr>
    <tr>
        <td>
            <pre style="font-size: 11pt; font-family: Tahoma, Arial, Helvetica, sans-serif">$FAILED_TESTS</pre>
            <br/>
        </td>
    </tr>
    <tr>
        <td>
            <b>
                <span style="color: #0B610B; ">构建日志 (最后 100行):</span>
            </b>
            <hr size="2" width="100%" align="center"/>
        </td>
    </tr>
    <tr>
        <td>
            <textarea cols="80" rows="30" readonly="readonly" style="font-family: Courier New">
                ${BUILD_LOG, maxLines=100}
            </textarea>
        </td>
    </tr>
</table>
</body>
</html>
~~~

![image-20210520110622468](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520110622468.png)



修改流水线的语法，在最后添加pipline 语法

~~~groovy
pipeline {
    agent any

    stages {
        stage('pull code') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], extensions: [], userRemoteConfigs: [[credentialsId: '1527f9a8-f6ed-41b3-93a7-c05a88f36de9', url: 'https://gitee.com/fakerlove/clock']]])
            }
        }
          stage('build project') {
            steps {
               sh 'mvn clean package'
            }
        }
          stage("sa") {
            steps {
                echo 'goujian构件wancheng'
            }
        }
    }
    post { 
        always {
            emailext(
                subject: '构建通知：${PROJECT_NAME} - Build # ${BUILD_NUMBER} - ${BUILD_STATUS}!', 
                body: '${FILE,path="email.html"}', 
                to: '203462009@qq.com' ) 
        }
    }
}
~~~



点击重新构建，构建完成后，会接收到邮件信息

![image-20210520112034362](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520112034362.png)



## 3.6 SonarQube 代码审查





# 4. Docker+SpringCloud 

## 4.1 环境准备

![image-20210520163648458](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520163648458.png)

* docker
* jenkins 
* maven
* harbor 私有的镜像仓库

### 4.1.1 docker的安装

#### centos 7

~~~bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
yum install -y yum-utils
yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
yum install docker-ce docker-ce-cli containerd.io -y
systemctl start docker
docker version
~~~

#### centos 8

~~~bash
yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
yum install -y yum-utils
yum makecache 
yum install -y wget
wget -O /etc/yum.repos.d/CenOS-Base.repo https://mirrors.aliyun.com/repo/Centos-8.repo
yum clean all
yum install -y https://download.docker.com/linux/centos/8/x86_64/stable/Packages/containerd.io-1.4.3-3.1.el8.x86_64.rpm
yum install docker-ce docker-ce-cli -y
systemctl start docker
docker version
~~~

#### 容器镜像加速

~~~bash
sudo mkdir -p /etc/docker
sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://ip92h4jn.mirror.aliyuncs.com"]
}
EOF
sudo systemctl daemon-reload
sudo systemctl restart docker
~~~

### 4.1.2 docker-compose 安装

compose 网址

~~~bash
https://github.com/docker/compose/releases
~~~

安装docker-compose

~~~bash
sudo curl -L "https://gitee.com/fakerlove/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
docker-compose --version
~~~

centos7 可以选择如下安装

~~~bash
yum -y install python-pip
pip install --upgrade pip
pip install docker-compose
docker-compose -version
~~~

### 4.1.3 harbor 安装

在github 上找到适合自己的版本

~~~bash
https://github.com/goharbor/harbor/tags
~~~

最低的需求配置

![image-20210520151346044](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520151346044.png)

上传下载的软件，并解压

~~~bash
 tar -zxvf harbor-offline-installer-v2.2.2.tgz -C /usr/local/
~~~

复制yml

~~~bash
cd /usr/local/harbor/
cp harbor.yml.tmpl harbor.yml
vim harbor.yml
~~~

修改一下三个部分信息

![image-20210521212145966](https://gitee.com/fakerlove/picture_1/raw/master/image-20210521212145966.png)



启动脚本

~~~bash
./install.sh
~~~

![image-20210520161403576](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520161403576.png)

等启动成功,输入网址

~~~bash
http://121.37.175.163/
~~~

![image-20210520161615617](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520161615617.png)



如果默认80 网址，有服务，即可修改端口信息

![image-20210520161626732](https://gitee.com/fakerlove/picture_1/raw/master/image-20210520161626732.png)





### 4.1.4 Harbor 使用

创建harbor项目

![image-20210521094535809](https://gitee.com/fakerlove/picture_1/raw/master/image-20210521094535809.png)

创建镜像，给镜像把tag

121.37.175.163 为Harbor的网址，jenkins为Harbor的用户信息，myjenkins 为镜像名字

<font color=red>注意点</font>

80端口的话，打tag 时不能添加80，直接就是ip 地址即可，其中clock 为项目名。kk 为在仓库中镜像的名字

~~~bash
docker tag jenkins/jenkins:latest 121.37.175.163/clock/kk:1.0
~~~

![image-20210521213410151](https://gitee.com/fakerlove/picture_1/raw/master/image-20210521213410151.png)

添加受信任名单

![image-20210521095547573](https://gitee.com/fakerlove/picture_1/raw/master/image-20210521095547573.png)

~~~bash
vim /etc/docker/daemon.json
~~~

添加一下内容

~~~bash
{
  "registry-mirrors": ["https://ip92h4jn.mirror.aliyuncs.com"],
  "insecure-registries": ["121.37.175.163:80"]
}
~~~

使用Harbor 账户登录docker

~~~bash
systemctl daemon-reload
systemctl restart docker
docker login 121.37.175.163 -u admin -p 123456
~~~

上传镜像

~~~bash
docker push  121.37.175.163/clock/kk:1.0
~~~

![image-20210522115421027](https://gitee.com/fakerlove/picture_1/raw/master/image-20210522115421027.png)

如果报错

~~~bash
Get http://harbor.phc-dow.com/v2/: Get http://harbor.phc-dow.com:180/service/token?account=admin&client_id=docker&offline_token=true&service=harbor-registry: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers) (Client.Timeout exceeded while awaiting headers)
~~~

解决方案

1. 检查是否添加信任名单，
2. 检查是否填写外网网址
3. 检查用户名和密码
4. 检查端口号，是否填写正确
5. 检查防火墙和安全组是否开放

最后试试以下内容

~~~bash
vim /etc/sysctl.conf
net.ipv4.ip_forward = 1 #最后一行写入
sysctl -p
~~~



## 4.2 使用DockerFile插件生成镜像

### 4.2.1添加打包插件

~~~bash
  <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>1.4.13</version>
                <configuration>
                    <repository>${project.artifactId}</repository>
                    <buildArgs>
                        <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                    </buildArgs>
                </configuration>
            </plugin>
~~~

### 4.2.2 编写Dockerfile 文件

~~~bash
FROM openjdk:8-jdk-alpine
EXPOSE 8081
ARG JAR_FILE
ADD ${JAR_FILE} /app.jar
ENTRYPOINT ["java", "-jar","/app.jar"]
~~~



![image-20210522170028697](https://gitee.com/fakerlove/picture_1/raw/master/image-20210522170028697.png)

修改jenkins 信息

### 4.2.3 打包镜像，并上传服务器

可以使用阿里云的镜像容器服务，或者自己的harbor私人镜像仓库

使用流水线语法，进行上传服务

![image-20210523095155330](https://gitee.com/fakerlove/picture_1/raw/master/image-20210523095155330.png)

~~~groovy
pipeline {
    agent any
    environment {
        tag = 'latest'
        //阿里云的镜像存储空间
        harbor_url = 'registry.cn-shanghai.aliyuncs.com'
        //镜像存储的命名空间
        harbor_project_name = 'jokerak' //Harbor的凭证
        harbor_auth = 'ef499f29-f138-44dd-975e-ff1ca1d8c933'
        // 项目名
        project_name = 'clock'

        imageName = "${project_name}:${tag}"
    }

    stages {
        stage('拉取代码') {
            steps {
                checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/master']],
                            extensions: [],
                            userRemoteConfigs:
                            [[credentialsId:
                            '1527f9a8-f6ed-41b3-93a7-c05a88f36de9',
                            url: 'https://gitee.com/fakerlove/clock']
                            ]
                       ])
            }
        }
        stage('打包编译项目，创建镜像，上传镜像') {
            steps {
                // 打包编译项目
                sh 'mvn clean package dockerfile:build'

                // 给镜像打tag
                sh "docker tag ${imageName} ${harbor_url}/${harbor_project_name}/${imageName}"

                withCredentials([usernamePassword(credentialsId: '5b07edf3-bfb8-49e7-8b27-a6d600ef99d8', passwordVariable: 'password', usernameVariable: 'username')]) {
                    // 登录镜像
                    sh "docker login -u ${username} -p ${password} ${harbor_url}"
                    //上传镜像
                    sh "docker push ${harbor_url}/${harbor_project_name}/${imageName}"
                }
            }
        }
    }
    post {
        always {
            // 发送邮件
            emailext(
                subject: '构建通知：${PROJECT_NAME} - Build # ${BUILD_NUMBER} - ${BUILD_STATUS}!',
                body: '${FILE,path="email.html"}',
                to: '203462009@qq.com' )
        }
    }
}
~~~

### 4.2.4 拉取镜像并部署

#### 1) 安装插件

安装以下插件，可以实现远程发送Shell命令

![image-20210523095613420](https://gitee.com/fakerlove/picture_1/raw/master/image-20210523095613420.png)



首先在jenkins 所在服务器下生成ssh ,<font color=red>2.277 版本一下不支持一下生成公钥。不知道现在2.932 还支不支持</font>

~~~bash
ssh-keygen -o
~~~

可以试一下

~~~bash
ssh-keygen -m PEM -t rsa -b 4096
~~~

说明：

```
-m 参数指定密钥的格式，PEM是rsa之前使用的旧格式
 -b 指定密钥长度。对于RSA密钥，最小要求768位，默认是2048位。
```

拷贝公钥到服务器中

~~~bash
ssh-copy-id 121.37.175.163
~~~

![image-20210523100212577](https://gitee.com/fakerlove/picture_1/raw/master/image-20210523100212577.png)



#### 2) 添加远程服务器

系统配置->添加远程服务器

修改公钥权限,复制公钥值jenkins目录下，以防没有权限访问

~~~bash
mkdir /var/lib/jenkins/.ssh
cp /root/.ssh/* /var/lib/jenkins/.ssh/
~~~

![image-20210523101501916](https://gitee.com/fakerlove/picture_1/raw/master/image-20210523101501916.png)

点击测试

![image-20210523161243920](https://gitee.com/fakerlove/picture_1/raw/master/image-20210523161243920.png)

#### 3) 上传代码

使用流水线语法生成器

![](https://gitee.com/fakerlove/picture_1/raw/master/image-20210523165536347.png)



创建流水线语法

~~~bash
  sshPublisher(
                    publishers:
                [sshPublisherDesc(configName: '华为云服务器',
                transfers: [
                    sshTransfer(
                        cleanRemote: false,
                        excludes: '',
                        execCommand: "/root/deploy.sh $harbor_url $harbor_project_name $project_name $tag $port",
                        execTimeout: 120000,
                        flatten: false,
                        makeEmptyDirs: false,
                        noDefaultExcludes: false,
                        patternSeparator: '[, ]+',
                        remoteDirectory: '',
                        remoteDirectorySDF: false,
                        removePrefix: '',
                        sourceFiles: '')],
                        usePromotionTimestamp: false,
                        useWorkspaceInPromotion: false,
                        verbose: false)])
~~~

在部署服务的服务器上创建deploy.sh

~~~bash
#! /bin/sh
harbor_url=$1 
harbor_project_name=$2 
project_name=$3 
tag=$4 
port=$5 
imageName=$harbor_url/$harbor_project_name/$project_name:$tag 
echo "$imageName" #查询容器是否存在，存在则删除 
containerId=`docker ps -a | grep -w ${project_name}:${tag} | awk '{print $1}'` 
if [ "$containerId" != "" ] ; then 
#停掉容器 
  docker stop $containerId 
#删除容器 
  docker rm $containerId 
  echo "成功删除容器"
fi
#查询镜像是否存在，存在则删除 
imageId=`docker images | grep -w $project_name | awk '{print $3}'`
if [ "$imageId" != "" ] ; then 
#删除镜像 
 docker rmi -f $imageId
 echo "成功删除镜像" 
fi
# 登录Harbor私服 
docker login -u itcast -p Itcast123 $harbor_url 
# 下载镜像 
docker pull $imageName 
# 启动容器 
docker run -di -p $port:$port -v /etc/localtime:/etc/localtime $imageName
echo "容器启动成功"
~~~

### <font color=red>4.2.5 出现的错误如下</font>

<font color=red>找不到打包插件</font>

更换maven 镜像

~~~bash
	<mirror>
			<id>
				alimaven
			</id>
			
			<mirrorOf>
				*
			</mirrorOf>
			
			<name>
				aliyun maven
			</name>
			
			<url>
				https://maven.aliyun.com/repository/public
			</url>
		</mirror>
~~~

<font color=red>无法创建maven包</font>

修改maven 仓库权限

~~~bash
chmod 777 /var/lib/res
~~~

<font color=red>Jenkins: unix://localhost:80: Permission denied</font>

~~~bash
chmod 777 /var/run/docker.sock
~~~

<font color=red>还有常见的错误</font>

~~~bash
https://blog.csdn.net/weixin_43824748/article/details/109044848
~~~

修改配置文件

~~~bash
vim /etc/sysconfig/jenkinsjenkins改成root
~~~

jenkins改成root

~~~bash
JENKINS_USER="root"
~~~



修改Jenkins相关文件夹用户权限

~~~bash
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
~~~





### 4.2.6 流水线的总代码

~~~groovy
pipeline {
    agent any
    environment {
        tag = 'latest'
        //阿里云的镜像存储空间
        harbor_url = 'registry.cn-shanghai.aliyuncs.com'
        //镜像存储的命名空间
        harbor_project_name = 'jokerak' //Harbor的凭证
        harbor_auth = 'ef499f29-f138-44dd-975e-ff1ca1d8c933'
        // 项目名
        project_name = 'clock'
        port = '8081'
        imageName = "${project_name}:${tag}"
    }

    stages {
        stage('拉取代码') {
            steps {
                checkout([
                            $class: 'GitSCM',
                            branches: [[name: '*/master']],
                            extensions: [],
                            userRemoteConfigs:
                            [[credentialsId:
                            '1527f9a8-f6ed-41b3-93a7-c05a88f36de9',
                            url: 'https://gitee.com/fakerlove/clock']
                            ]
                       ])
            }
        }
        stage('打包编译项目，创建镜像，上传镜像') {
            steps {
                // 打包编译项目
                sh 'mvn clean package dockerfile:build'

                // 给镜像打tag
                sh "docker tag ${imageName} ${harbor_url}/${harbor_project_name}/${imageName}"

                withCredentials([usernamePassword(credentialsId: '5b07edf3-bfb8-49e7-8b27-a6d600ef99d8', passwordVariable: 'password', usernameVariable: 'username')]) {
                    // 登录镜像
                    sh "docker login -u ${username} -p ${password} ${harbor_url}"
                    //上传镜像
                    sh "docker push ${harbor_url}/${harbor_project_name}/${imageName}"
                }
            }
        }
        stage('拉取镜像，部署服务') {
            steps {
                sshPublisher(
                    publishers:  [
                     sshPublisherDesc(
                     configName: 'cloud',
                     transfers: [
                      sshTransfer(
                        cleanRemote: false,
                        excludes: '',
                        execCommand: "/opt/jenkins_shell/deploy.sh $harbor_url $harbor_project_name $project_name $tag $port",
                        execTimeout: 120000,
                        flatten: false,
                        makeEmptyDirs: false,
                        noDefaultExcludes: false,
                        patternSeparator: '[, ]+',
                        remoteDirectory: '',
                        remoteDirectorySDF: false,
                        removePrefix: '',
                        sourceFiles: '')],
                        usePromotionTimestamp: false,
                        useWorkspaceInPromotion: false,
                        verbose: false)])
            }
        }
    }
    post {
        always {
            // 发送邮件
            emailext(
                subject: '构建通知：${PROJECT_NAME} - Build # ${BUILD_NUMBER} - ${BUILD_STATUS}!',
                body: '${FILE,path="email.html"}',
                to: '203462009@qq.com' )
        }
    }
}
~~~



## 4.3 部署静态网站

### 4.3.1 安装ngnix

~~~bash
yum install epel-release
yum -y install nginx
~~~

修改配置文件,修改端口信息

~~~bash
vi /etc/nginx/nginx.conf
~~~

![image-20210524103241829](https://gitee.com/fakerlove/picture_1/raw/master/image-20210524103241829.png)



要关闭selinux

~~~bash
vi /etc/selinux/config
~~~

永久关闭 SELINUX=disabled

![image-20210524103436326](https://gitee.com/fakerlove/picture_1/raw/master/image-20210524103436326.png)



启动ngnix

~~~bash
systemctl start nginx
~~~

![image-20210524103555607](https://gitee.com/fakerlove/picture_1/raw/master/image-20210524103555607.png)



### 4.3.2 安装nodejs 插件

搜索插件NodeJS

![image-20210524104356108](https://gitee.com/fakerlove/picture_1/raw/master/image-20210524104356108.png)

### 4.3.3 安装nodejs

尽量使用自己的npm,因为需要配置镜像，自己的环境搭建起来

~~~bash
wget https://npm.taobao.org/mirrors/node/v14.16.1/node-v14.16.1-linux-x64.tar.xz
xz -d node-v14.16.1-linux-x64.tar.xz
tar -xvf node-v14.16.1-linux-x64.tar
mkdir -p  /var/lib/node 
rm -rf /var/lib/node/*
cp ~/node-v14.16.1-linux-x64/* /var/lib/node -r
chmod 777 -R /var/lib/node
ln -s /var/lib/node/bin/node /usr/bin/node
ln -s /var/lib/node/bin/npm /usr/bin/npm
npm config set registry https://registry.npm.taobao.org
vim /etc/profile
export PATH=/var/lib/node/bin:$PATH
source /etc/profile
~~~

![image-20210524145722467](https://gitee.com/fakerlove/picture_1/raw/master/image-20210524145722467.png)

### 4.3.4 创建项目

一定要修改连接的根目录，不然会出现问题

![image-20210524153948557](https://gitee.com/fakerlove/picture_1/raw/master/image-20210524153948557.png)

创建一个Vue的项目。vue-cli3 脚手架创建项目。然后创建Jenkinsfile 文件

在流水线语法生成器，生成语法

![image-20210524145646139](https://gitee.com/fakerlove/picture_1/raw/master/image-20210524145646139.png)

### 4.3.5 Jenkinsfile 代码

~~~bash

~~~



## 4.4 SpringCloud 打包服务

注意点

* common 工程中，不能够使用springboot 的打包插件，必须移除common 父工程的打包插件

使用,-f 指定特定工程进行打包

~~~bash
mvn -f common clean install
~~~

![image-20210524190538534](https://gitee.com/fakerlove/picture_1/raw/master/image-20210524190538534.png)





### 4.4.1 修改配置

~~~bash
# 集群版
spring:
  application:
    name: EUREKA-HA
---
server:
  port: 10086
spring: # 指定profile=eureka-server1
  profiles: eureka-server1
eureka:
  instance: # 指定当profile=eureka-server1时，主机名是eureka-server1
    hostname: 192.168.66.103
  client:
    service-url: # 将自己注册到eureka-server1、eureka-server2这个Eureka上面去
      defaultZone: http://192.168.66.103:10086/eureka/,http://192.168.66.104:10086/eureka/
      
---

server:
 port: 10086
spring:
 profiles: eureka-server2
eureka:
 instance:
  hostname: 192.168.66.104
 client:
  service-url:
   defaultZone: http://192.168.66.103:10086/eureka/,http://192.168.66.104:10086/eureka/
~~~



### 4.4.2 安装插件

安装Extended Choice Parameter 
