**学习目标**

1. 知道jenkins应用场景
2. 能够安装部署jenkins服务器
3. 能够实现git+github+jenkins手动构建
4. 能够实现git+gitlab+jenkins自动发布系统



# 认识jenkins

​       Jenkins是一个可扩展的持续集成引擎，是一个开源软件项目，旨在提供一个开放易用的软件平台，使软件的持续集成变成可能。Jenkins非常易于安装和配置，简单易用。

官网：https://jenkins.io/



# jenkins应用场景 

场景1: 

![1553085139748](图片/场景1.png)

* 研发人员上传开发好的代码到github代码仓库
* 需要将代码下载到nginx服务器部署
  * 运维人员手动下载再部署
  * 运维人员使用脚本下载再部署

场景2:

![1553085753649](图片/场景2.png)

![1553085793824](图片/场景2-2.png)



# jenkins下载

![1552997312959](图片/jenkins下载1.png)



![1552997516046](图片/jenkins下载2.png)



![1552997762840](图片/jenkins下载3.png)



# jenkins安装

准备一台服务器安装jenkins（我这里IP为10.1.1.13)

* 静态IP（要求能上外网)
* 主机名
* 关闭防火墙,selinux
* 时间同步
* 确认openjdk1.8版本已经安装

~~~powershell
[root@jenkins_server ~]# java -version
openjdk version "1.8.0_161"
OpenJDK Runtime Environment (build 1.8.0_161-b14)
OpenJDK 64-Bit Server VM (build 25.161-b14, mixed mode)
~~~



第1步: 将下载好的软件包拷贝到jenkins服务器上直接rpm命令安装

~~~powershell
[root@jenkins_server ~]# rpm -ivh jenkins-2.150.3-1.1.noarch.rpm
~~~

第2步: 启动服务并验证端口

~~~powershell
[root@jenkins_server ~]# systemctl start jenkins
[root@jenkins_server ~]# chkconfig jenkins on

[root@jenkins_server ~]# lsof -i:8080
COMMAND  PID    USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
java    4334 jenkins  163u  IPv6  45095      0t0  TCP *:webcache (LISTEN)
~~~

第3步: 查看密码文件里的密码(此为初始管理员用户admin的密码)。通过浏览器访问填上密码(地址为服务器ip的8080端口)

~~~powershell
[root@jenkins_server ~]# cat /var/lib/jenkins/secrets/initialAdminPassword
d750b101634b453b87deccfd06365fc9
~~~



![1552918042530](图片/jenkins界面1.png)

第4步: 选择安装推荐的插件(如果是offline状态,可以先跳过后续再安装)

![1552918596047](图片/jenkins界面2.png)



![1552922174952](图片/jenkins界面3.png)

第5步: 创建新管理员用户(创建了新的管理员用户后，原来的admin用户就不能用了)，也可直接使用初始管理员admin登录

![1552922392772](图片/jenkins界面4.png)

第6步:确认访问地址

![1552922907022](图片/jenkins界面5.png)



![1552922960030](图片/jenkins界面6.png)

第7步: 进入jenkins主页面

![1552923661333](图片/jenkins界面7.png)



## 插件安装(拓展) 

1. 如果因为网速等原因实在是无法下载插件,可以将别人下载好的插件打包给你，解压到/var/lib/jenkins/plugins/目录



2. 可以在下面地址下载插件(插件为.hpi结尾的文件), 然后上传到jenkins.(这种方法适合单个插件安装)

插件地址：http://updates.jenkins-ci.org/download/plugins

![1553504433537](图片/jenkins上传插件.png)





# git+github+jenkins

## 架构图

![1552924791657](图片/jenkins物理架构.png)



**实验架构图**

![1553086621836](图片/git+github+jenkins实验架构规划图.png)





## github上新建项目仓库

![1552927158944](图片/github创建仓库.png)



![1552927865360](图片/github创建仓库2.png)



![1552927958538](图片/github创建仓库3.png)



## 开发者电脑准备

第1步: 在开发者电脑上安装git工具

~~~powershell
[root@vm1 ~]# yum install git -y
~~~

第2步: 在开发者电脑上创建空密码密钥对

![1553006798473](图片/jenkins密钥工作图.png)

~~~powershell
[root@vm1 ~]# ssh-keygen -t rsa -f /root/.ssh/id_rsa -C "dev1@itcast.cn" -N ""
~~~

第3步: 在开发者电脑上查看并复制公钥

~~~powershell
[root@vm1 ~]# cat /root/.ssh/id_rsa.pub 
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC+8f9vVJ8kskBojsxqaA95DQzLemCuA3o9nWE1sCjHHy/xxT4Ev57WbVCCPnCy3/pR49o5i7RE+5/dA4Ct+QXpj02pE2mPiehMIGFmjolhYFqIq7lnTSQ+zVtetIxnn2zmOx0qz+Zdr/wSCh/Czl7+Y2RClSq2sgD80/eF/uBpdlku2ejXAnIKFn3NekbqM4gYao/XTDLMW7D7pyQ0CFaI0xwEdXroy7ozAyFo76kvxs4IztAslcUeEj/CGha3WsLATRTeDNK5YGlI8jcw0WEcZocEhbS2RhkikQjACGgrae3WpJY/szH9BQeH8rIF2vR5s0DlPy9PJtBAxuUe8hJ/ dev1@itcast.cn

~~~

第4步: 将开发者公钥添加到github

![1553007869603](图片/github添加开发者密钥.png)



![1553007983358](图片/github添加开发者密钥2.png)



![1553008141619](图片/github添加开发者密钥3.png)



![1553008207784](图片/github添加开发者密钥4.png)



## 开发者提交文件测试

第1步: 在github上获取ssh免密地址

![1553009294980](图片/获取github免密地址.png)

第2步: 开发者电脑上设置开发者身份

~~~powershell
[root@vm1 ~]# git config --global user.name "dev"
[root@vm1 ~]# git config --global user.email "dev@itcast.cn"
[root@vm1 ~]# git config --global color.ui true
~~~

第3步: clone项目到开发者本地电脑

~~~powershell
[root@vm1~]# git clone git@github.com:linux-daniel/jenkins.git
Cloning into 'jenkins'...
The authenticity of host 'github.com (52.74.223.119)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
RSA key fingerprint is MD5:16:27:ac:a5:76:28:2d:36:63:1b:56:4d:eb:df:a6:48.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,52.74.223.119' (RSA) to the list of known hosts.
warning: You appear to have cloned an empty repository.

~~~

第4步: 提交测试代码文件

~~~powershell
[root@vm1 ~]# cd jenkins/
[root@vm1 jenkins]# echo "test" >> README.md

[root@vm1 jenkins]# git add README.md
[root@vm1 jenkins]# git commit -m "add README.md"

[root@vm1 jenkins]# git push -u origin master
~~~

第5步: github上验证

![1553010362245](图片/github上验证push是否成功.png)



## nginx服务器准备

在nginx服务器上安装nginx,并启动服务

~~~powershell
[root@nginx ~]# yum install epel-release
[root@nginx ~]# yum install nginx -y
[root@nginx ~]# systemctl start nginx
[root@nginx ~]# systemctl enable nginx
~~~





## jenkins私钥配置

第1步: jenkins图形安装push over ssh插件

![1553010890677](图片/jenkins私钥配置1.png)



![1553010947301](图片/jenkins私钥配置2.png)



![1553011281250](图片/jenkins私钥配置3.png)



![1553011546506](图片/jenkins私钥配置4.png)



![1553011660527](图片/jenkins私钥配置5.png)



第2步: 在jenkins服务器上生成空密码密钥对

~~~powershell
[root@jenkins_server ~]# ssh-keygen -t rsa -f /root/.ssh/id_rsa -C "jenkins-server@itcast.cn" -N ""
~~~

第3步: 查看并复制私钥

~~~powershell
[root@jenkins_server ~]# cat /root/.ssh/id_rsa
-----BEGIN RSA PRIVATE KEY-----
MIIEpQIBAAKCAQEAvWekHSkS23a/8kN6SkDTjFdcdw5zuupaVY9KJd2Ejnfg7/ZU
6fqOO1b8bAzYMkf8aKTcStKEwBWV6TYA/ljGG/6oAz1UGwBke/Sw6wASp7wN4MUQ
HQDeHKv+odPHuSloK47e3LrjzCssXYPrvNmzsLKXIzbmiGJNAZYznYMBtnDNxpiy
1L+4zQeSTOfT0MCbF/KTgbyffRCNHC/m8Ow89NF1XW37he3XV8wimM6NrGtndw1N
YVZMyx85xEsK9m2Wxee+P7KgIdL5nmbGJOBbTUi4dlt9j4T+mOxr4AF9oaZKS0o2
oJlJyyl9TKtHQM6d2Qw8BGXOGMDVSB8WT1Q5UwIDAQABAoIBAHsCS4iQu2mDBwhN
IKgG0B2OQ0QjQ7A6Ma7tn6dV5ZgtbQ4Lenx3OFZ7mPaHpQWK0PgZUeTaMlMZ8cGD
TEPj3c4ipnVsKCpdJ+WFNj15T6RWMuEuutdLT/VpEreA9m5f4QKhCEZsrjNUOr0F
R13gOZ5hblz1c+VRilekeCMtCTi1jKkDdVNFRcdQe8m8kttFVC2hSPB7tJxEoxmp
kAPkPBwU0E/6pA591JYCUk7lNQ9eBDgoBbb9cglEa4tn3Hh7AoyXcJTvYoC1vH1M
zN/1MLHyME76a/intrQM9frYwgM1gUdZU5i5kt4/SFQfd/qM8Axvy7s0qcFf5jxo
Ey4Ob4kCgYEA9HkCeBMexLZnVbZrDNiaapANdPafY9PILduZ9nVQmWh4rRcu47r0
D1V6yDtBS8+p/sPJu/4KYEn/8yDWHjxd3O194XaAKl9xgqUPP23txohp99pHy2mL
21eqthC0QKk2bH23jAQxjd0MAz5mK4uO1r/BFhSnJpK4i5/jVIHqm3cCgYEAxlXq
lqSruCYzwzMX/7Gv1lqexmUGmXiqQ4LWtOlQIaL+BoPOajp2mJjJMsIF1qkQwZnp
L+WdP02j7esBD1hI9G+lISCyqjTG+OCdNFiQ3SFJDZoLrcBN1uByjgANPleMiP9y
zq+xa+zBQ2YEtAEA8gp37QzfA2P6zihCNQqmUAUCgYEAgcrFHs635SQaFI12pClT
QgQcwN42nR9RBdezFAAQvIGUoADQ6iLVdFajiy66ae9kh1eXAPHMvHZNJt1mEENo
aeTEkjEBtn1ZnEzZnYlVVbQS3n3K5BmzIM6YWXTg3ft4Y30TN4j6biDPQeGdCL1d
JnJDpt9sJrR6udY3MSSQU90CgYEAnqnkvRaG+Q42opWhQUAYdtaP5g6ztNq++rsU
oC11mTMXHIcc/gY/EdxIOH7WxN8DNJ232kVKAnZOCerSMkBiPImEBHhv9ZG7CyZF
HLctTHlwQ51Ucm9A1gFAIzEPZywKlR4l7grHWJtSEGTwpj+XTgnp3o1JayD0Zy/1
pxEZ8zECgYEAnS/PHs+164GYPkWlYxGw04UE4SlQGa0QqT1WfctFPctU5PVqpwHi
imEFS+V89p1N+bQpgEI+WqlRGgODcjIE0ho8DqhsKaqu0AeVPN91Dgi20giJE8xo
TnYioS3qpxXtPiwrVR8PUZ/WZ/YtM0jwvYXowsSHeqVfCUBNGHqvwEA=
-----END RSA PRIVATE KEY-----
~~~



第4步: 在jenkins中添加ssh私钥

![1553012342059](图片/jenkins私钥配置6.png)



![1553012931484](图片/jenkins私钥配置7.png)



第5步: 在jenkins服务器上配置对nginx服务器的免密登录

~~~powershell
[root@jenkins_server ~]# ssh-copy-id -i 10.1.1.14
10.1.1.14为nginx服务器的IP
~~~



第6步: 然后填写连接nginx信息，测试连接成功后保存

![1553013194254](图片/jenkins私钥配置8.png)



## 添加Jenkins服务器公钥到github

添加过程见上面笔记(这里省略，直接看下面结果)

![1553013462165](图片/添加jenkins服务器公钥到github.png)



## 为jenkins服务器添加凭据

第1步: 在jenkins界面添加凭据

![1553013641895](图片/jenkins添加凭据.png)





![1553013717332](图片/jenkins添加凭据2.png)



![1553013776330](图片/jenkins添加凭据3.png)





![1553013819399](图片/jenkins添加凭据4.png)

第2步: 添加凭据信息

![1553014084926](图片/jenkins添加凭据5.png)



![1553014143212](图片/jenkins添加凭据6.png)









## jenkins任务创建

第1步: 创建新任务

![1553078055293](图片/jenkins创建任务1.png)

第2步: 自定义任务名称与风格

![1553078335434](图片/jenkins创建任务2.png)

第3步: 自定义任务描述

![1553078510045](图片/jenkins创建任务3.png)

第4步: 定义源码管理

![1553079064976](图片/jenkins创建任务4.png)



第5步:定义构建方法

![1553079481477](图片/jenkins创建任务5.png)

第6步: 定义构建的源码,目标主机和目标目录

![1553079716269](图片/jenkins创建任务6.png)



第7步: 设置完毕，保存，并验证

![1553079846398](图片/jenkins创建任务7.png)

![1553080073223](图片/jenkins创建任务8.png)



## 手动构建

第1步: 立即构建

![1553082456795](图片/jenkins构建1.png)

第2步: 在workspace工作区间查看

![1553082562299](图片/jenkins构建2.png)

![1553082689184](图片/jenkins构建3.png)

第3步: 查看控制台输出信息

![1553083607694](图片/jenkins构建4.png)



第4步: nginx服务器上验证文件是否被传到nginx家目录

~~~powershell
[root@nginx ~]# ls /usr/share/nginx/html/
404.html  index.html      poweredby.png
50x.html  nginx-logo.png  README.md
可以看到README.md被传过来了
~~~

**练习:** 在开发者电脑上再次上传文件,并构建测试



# 自动发布系统



![1553098138330](图片/自动发布系统实验环境图.png)

## Gitlab上创建自动构建仓库

第1步: gitlab上创建新仓库

![1553174861062](图片/gitlab创建自动构建web1.png)



第2步: 自定义项目名称等

![1553174585594](图片/gitlab创建自动构建web2.png)

第3步: 确认创建成功

![1553175117175](图片/gitlab创建自动构建web3.png)



## jenkins安装对应插件

安装gitlab与gitlab hook插件

![1553176922376](图片/jenkins安装gitlab与gitlab_hook插件.png)



![1553185157195](图片/jenkins安装gitlab与gitlab_hook插件2.png)

## 添加Jenkins服务器公钥到gitlab

第1步: 添加Jenkins服务器公钥到gitlab

![1553185839595](图片/添加jenkins公钥到gitlab.png)

第2步: 复制gitlab上自动发布项目地址

![1553183183209](图片/获取自动发布项目地址.png)

第3步: 在jenkins服务器上克隆仓库，确认连接OK

~~~powershell
[root@jenkins-server ~]# git clone git@10.1.1.12:root/auto_build_web.git
Cloning into 'auto_build_web'...
warning: You appear to have cloned an empty repository.
~~~

## jenkins创建自动构建任务



![1553184493070](图片/jenkins创建自动构建任务.png)



![1553184602674](图片/jenkins创建自动构建任务2.png)



![1553186294154](图片/jenkins创建自动构建任务3.png)



![1553186616179](图片/jenkins创建自动构建任务4.png)



![1553224131699](图片/jenkins创建自动构建任务5.png)



![1553186863931](图片/jenkins创建自动构建任务6.png)

![1553187361214](图片/jenkins创建自动构建任务7.png)



~~~powershell
#!/bin/bash

#源目录为jenkins存放任务文件的目录 
SOURCE_DIR=/var/lib/jenkins/workspace/$JOB_NAME/
#目标目录为nginx服务器的家目录
DEST_DIR=/usr/share/nginx/html
#使用rsync同步源到nginx服务器家目录(需要免密登录)，IP为nginx服务器IP
/usr/bin/rsync -av --delete $SOURCE_DIR root@10.1.1.14:$DEST_DIR
~~~



## 配置jenkins服务器上的jenkins用户



~~~powershell
[root@jenkins-server ~]# grep jenkins /etc/passwd
jenkins:x:988:982:Jenkins Automation Server:/var/lib/jenkins:/bin/false

[root@jenkins-server ~]# usermod -s /bin/bash jenkins

[root@jenkins-server ~]# grep jenkins /etc/passwd
jenkins:x:988:982:Jenkins Automation Server:/var/lib/jenkins:/bin/bash
~~~

~~~powershell
[root@jenkins-server ~]# su - jenkins
-bash-4.2$ ssh-keygen -t rsa -C "jenkins user" -N ""

-bash-4.2$ ssh-copy-id -i root@10.1.1.14
~~~



## jenkins全局安全配置



![1553228664875](图片/jenkins全局安全配置.png)



![1553228811977](图片/jenkins全局安全配置2.png)

## 配置gitlab允许本地网络使用webhook

gitlab默认在本地网络不能使用webhook，所以需要我们配置允许



![1553229742769](图片/配置gitlab允许本地使用webhook1.png)



![1553229844953](图片/配置gitlab允许本地使用webhook2.png)



![1553229928009](图片/配置gitlab允许本地使用webhook3.png)



![1553229997098](图片/配置gitlab允许本地使用webhook4.png)



## 为gitlab自动构建项目添加webhook

![1553230226025](图片/为gitlab自动构建项目添加webhook支持1.png)



![1553230580824](图片/为gitlab自动构建项目添加webhook支持2.png)



![1553230666947](图片/为gitlab自动构建项目添加webhook支持3.png)



![1553230778636](图片/为gitlab自动构建项目添加webhook支持4.png)



## 代码自动发布测试

开发者电脑上使用git提交测试文件

~~~powershell
[root@vm1 ~]# cd auto_build_web/
[root@vm1 auto_build_web]# echo "auto_build_web" > index.html
[root@vm1 auto_build_web]# git add index.html
[root@vm1 auto_build_web]# git commit -m "add index.html"
~~~

在nginx服务器上验证

~~~powershell
[root@nginx ~]# cat /usr/share/nginx/html/index.html 
auto_build_web
~~~



# pycharm与自动发布系统结合(拓展)

开发者开发代码一般会使用IDE集成开发工具（比如pycharm这种),那么使用pycharm开发的代码能否直接利用自动发布系统发布到业务服务器上呢？ 答案是肯定的。



这次使用windows模拟开发者电脑

![1553236082137](图片/windows上使用git.png)

![1553240986320](图片/windows上产生空密码密钥对.png)



![1553241082141](图片/gitlab上添加windows开发者公钥.png)



![1553241253905](图片/windows开发者克隆自动构建项目目录.png)



![1553241573165](图片/pycharm与自动构建项目关连.png)



![1553242156637](图片/pycharm与自动构建项目关连2.png)



![1553242357090](图片/pycharm与自动构建项目关连3.png)



![1553243341039](图片/pycharm与自动构建项目关连4.png)

![1553242657428](图片/pycharm与自动构建项目关连5.png)



![1553243402693](图片/pycharm与自动构建项目关连6.png)



nginx服务器上测试验证

~~~powershell
[root@nginx ~]# cat /usr/share/nginx/html/index.html 
auto_build_web
pycharm测试
~~~

