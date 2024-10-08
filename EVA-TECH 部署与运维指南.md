# EVA-TECH 部署与运维指南

本指南主要介绍了EVA-TECH目前已有的服务、生产环境与相关部署步骤。

## 硬件设备

EVA-TECH目前共有三台服务器

- 云服务器(101.35.229.174):部署于腾讯云，公网ipv4，有效期一年，每年到期换届更换
- 本地生产服务器(10.79.6.84):部署于204,无公网ipv4,有公网ipv6，双核E5，性能较强
- 本地测试服务器(10.79.6.84):部署于204,仅限校内v4访问

其中，本地生产服务器端口号为20089，本地测试服务器端口号为20088

2022年本地服务器进行了升级，新购入了一台双核E5 32G内存的生产服务器，原生产服务器转为测试服务器。测试服务器的bios电池损坏，意外断电后玄学重置bios设置，需要外接亮机卡启动进入bios修改启动模式为uefi。测试服务器的后背板USB存在玄学失灵等问题。

## 服务

### 概览

EVA-TECH目前提供的服务包括:

Oreo(工单系统前台)、XMS(工单系统后台)、EVA-Auth、EVA-File、EVA-gitea、EVA-drone、EVA-Wiki、纳新系统、EVA-mail、EVA-MC、EVA-MC-Skin、EVA-Matlab、EVA-ball(在线小游戏)

托管的服务包括:

ical、大英默写器、思政刷题器

不可用的服务包括:

EVA活动系统

弃用的服务包括

EVA-LDAP EVA-new-wiki

### 静态页面部分

绝大多数静态页面只需要将相应的静态文件置于相应位置即可

#### 协会主页

部署位置: 云服务器 /app/static/main/

项目地址: https://git.zjueva.net/WangJialiang/landing-2022

项目网址: https://a.zjueva.net

项目简介:E志者协会主页

部署须知: 项目仓库是2020年时的协会主页+纳新页 2021年对协会主页文案进行了部分修改，并把纳新页单独分割出了一个项目。

#### 纳新系统报名表

部署位置: 云服务器 /app/static/joinus-2022/

项目地址: https://git.zjueva.net/WangJialiang/joinus-2022

项目网址: https://joinus.zjueva.net

项目简介: 纳新系统的报名表页

部署须知: 关于图片上传等问题请看本文后端部分纳新系统的记录

#### Oreo

部署位置: 云服务器 /app/static/oreo/

项目地址: https://git.zjueva.net/zjueva/Oreo

项目网址: https://oreo.zjueva.net

项目简介:E志者协会工单管理前台 基于antd pro

现存问题:认证token有效期过短、token过期前端页面不进行错误处理

#### EVA-Ball

部署位置: 云服务器 /app/static/eva-ball/

项目网址: https://ball.zjueva.net

项目简介: 亦可顶球小游戏

#### 大英默写器

部署位置: 云服务器 /app/static/Eng/

项目地址: https://github.com/ADSR1042/SQTP2022

项目网址: https://eng.zjueva.net

项目简介: 托管项目

#### 思政刷题器

部署位置:本地生产服务器 /srv/study/

项目地址: https://github.com/ADSR1042/ZJURefresher

项目网址: https://study.zjueva.net

项目简介: 托管项目

### 后端部分

协会的大部分后端服务均以docker的形式提供，这极大节省了运维的工作量。一般而言，一些公有的docker镜像可以直接从docker hub自动拉取，类似XMS等自己构建的镜像需要拉去源代码仓库，并使用源代码仓库中的docker file自行进行构建。

如果对镜像进行了某些修改，可以用docker commit 等方式更新镜像，请务必留下更新说明。

需要注意的是，协会的服务器均未配置网络代理，由于目前dockerhub国内访问受限，部分镜像拉取时需要通过镜像站。


user.zjueva.net
#### XMS

部署位置: 云服务器 /app/dockerapps/backend-mysql/

部署类型: docker

项目地址: https://git.zjueva.net/zjueva/xms

项目网址: https://xms.zjueva.net 

项目简介: E志者协会工单管理系统后台

部署须知: XMS服务一共由两个镜像组成，app服务使用xms_mysql，db服务使用mysql镜像。目前mysql镜像未指定版本，默认使用lastest标签，需要锁定版本。**在进行迁移部署前请务必先关闭服务，否则迁移后mysql数据无法读取!**

#### EVA-Auth

部署位置:云服务器 /app/dockerapps/EVA-Auth/

部署类型:docker

项目地址: https://git.zjueva.net/WangJialiang/EVA-Auth

项目网址: https://auth.zjueva.net

项目简介: E志者协会统一身份认证

部署须知: EVA-Auth采用了内置的sqlite数据库，因此只需要一个eva-auth/eva-auth:test镜像。比较特别的是，由于认证服务的特殊性，从docker到客户端的全链路必须强制使用https。在其他常规情况下，我们只需要将服务端口通过nginx转发，并在nginx中添加证书相关信息启用https。但是在EVA-Auth部署过程中，我们必须在docker中启用https(nginx转发也还需要)。事实上，EVA-Auth后端在不提供证书的情况下根本不会启用https通讯。目前的做法是生成一个自签证书，并将证书传入容器中(详细请看docker compose file)。

需要注意的是，自签证书**有效期为一年**，本次证书有效期截止2024年8月1日。跨年迁移时请生成新的自签证书。

#### EVA-File

部署位置: 本地生产服务器 /srv/file/

部署类型: docker

项目地址: 基于NextCloud

项目网址: https://file.zjueva.net

项目简介: E志者私有云盘服务

部署须知: 

- 自适用无感ipv6跳转:目前EVA File支持校内ipv4 ipv6 校外ipv6访问。具体跳转原理如下

  1. 将file.zjueva.net域名解析到云服务器 云服务器重定向至https://file.autochthon.zjueva.net:1984 更改端口号
  2. file.autochton.zjueva.net cname指向autochton.zjueva.net autochon.zjueva.net拥有双解析

  以上跳转主要实现了无感端口号更改(校内ipv6封锁80与443)，校内校外dns正常解析。需要注意的是,除file主域名外，NextCloud还有配套的协作套件，域名跳转规则如同上述主域名。若Nextcloud的office文件预览失败，大概率是套件工作异常or证书过期

- EVA-Auth接入

  EVA-File原先采用ladp认证的方式，将账户密码通过ldap外接，NextCloud本身并不存储账户的密码。NextCloud的ladp接入套件工作极其独立，用户组只读无法写入。为进行迁移，我们将LDAP的用户数据进行导出，转化为NextCloud普通用户，密码缺省。通过EVA-Auth登录，由学号作为主键，用户可以登录原先账户。登录原先账户后可覆盖设置密码，但不建议。

  每次通过EVA-Auth登录EVA file都会同步用户个人信息，包括用户姓名、部门、邮箱等，个人头像做了特殊处理不同步。部门设定关系到EVA-File的用户组，当用户组发生变化时EVA-File会授予/剥夺用户组。

- 硬盘挂载

  EVA生产服务器上目前仍有一块1T硬盘处于闲置状态，当EVA-File文件容量快满时可以挂载到另外一块盘上。

  **请至少每月关注硬盘的SMART信息 本地生产服务器没有硬盘阵列 硬盘损坏意味着数据死亡**

#### EVA-gitea

部署位置: 云服务器 /app/dockerapps/gitea/

部署类型: docker

项目地址: 基于gitea

项目网址: https://git.zjueva.net

项目简介: EVA 私有git服务

部署须知 : 见EVA - drone 

#### EVA-drone

部署位置: 云服务器 /app/dockerapps/drone/

部署类型: docker

项目地址: 基于drone

项目网址: https://ci.zjueva.net

项目简介: gitea配套持续部署工具，可在项目仓库代码发生推送时自动部署项目

部署须知: 为了启动drone实际上要开启两个镜像，一个是drone本身，另一个是drone-runner-docker。docker作为容器并不能直接向宿主机发送执行指令，这被认为是虚拟机逃逸，是非常危险的。我们需要drone-runner-docker以docker in docker的方式执行命令。除了下载启动镜像外，还需要额外安装drone-runner-exec服务，参考地址如下https://docs.drone.io/runner/exec/installation/linux/

如果不安装此执行服务，可能会出现drone deploy阶段无限转圈或提示deploy成功实际并未执行的情况。

#### EVA-Wiki

部署位置: 云服务器 /app/dockerapps/evawiki/

部署类型: docker

项目地址: 基于mediawiki

项目网址: https://wiki.zjueva.net

项目简介: E志者百科，记录维修经历与知识

部署须知: PHP项目,注意nginx的配置

#### 纳新系统

部署位置: 云服务器 /app/dockerapps/joinus-backend-2021/

部署类型:docker

项目地址: https://git.zjueva.net/zjueva/nx-backend-2021

项目网址: https://joinus-backend.zjueva.net

项目简介: E志者协会纳新系统

部署须知: 

纳新系统组成相对复杂。报名者的报名表是由静态页面 joinus.zjueva.net提交的，报名表并不上传图片至服务后端。报名表中的图片上传链接到腾讯云的云函数，首先由云函数进行图片异常检测(防止注入)，之后检测正常由云函数写入储存桶。储存桶被配置在了公开读私有写的状态，写入需要帐号的appid授权，这部分敏感权限必然是放在云函数报名者接触不到的地方。云函数上传图片至储存桶以用户的学号作为文件名，纳新系统后端再通过储存桶链接加学号的方式获取图片。**这样实际上存在学号爆破下载图片的风险** 建议修改为使用uuid。

纳新系统还需要使用短信。由于短信签名等问题，协会至今使用的还是20级组长的短信帐号。为了发送短信，需要注册短信签名，正文模板，注册完后腾讯云会给予响应的签名id与正文id。之后在后端中调用腾讯SDK，并加入参数发送请求即可。纳新系统目前的群发短信实际上是一条条发的，写的还是同步函数。

报名者确定时间的逻辑是。报名成功即刻发送报名成功短信，面试开始前几天开始群发面试场次选择短信，报名者收到面试场次选择短信后选择场次上传服务器，服务器适时进行排班，排班成功后发送最终面试场次短信。

需要注意的是，假设某场面试容量五人，当已经有四人安排在此场次时，选场次页面便不会显示此场次。面试场次安排并不是实时的，需要手动安排。**建议在报名高峰期多手动排班**，否则极易出现无法安排的情况。

#### EVA-mail

部署位置: 腾讯的某台服务器

部署类型: 企业微信配置好的

项目简介: EVA邮箱服务

部署须知: 邮箱域名cname到了腾讯的企业微信

#### EVA-MC

部署位置: 本地生产服务器 /srv/dockerapps/MCSManager/ 

部署类型: docker

项目简介: EVA MC游戏服

部署须知: 服务分为守护进程与web前端两部分。游戏存档挂载到了另一块SSD /opt/docker-mcsm/

管理地址: 10.79.6.84:8081 

#### EVA-MC-Skin

部署位置: 云服务器 /app/dockerapps/MCBlessingskin/

部署类型: docker

项目网址: https://skin.zjueva.net

项目简介: EVA-MC 配套皮肤站与外置登录系统

部署须知:  如果是从头构建镜像的话，需要进入容器执行一次install-bsskin脚本

EVA-Matlab

部署位置:本地生产服务器 /app/matlab/

部署类型:docker 

项目简介: EVA私有Matlab服务 提供与本地运行一样的使用体验 已预装深度学习扩展

部署须知: 目前此服务加上了nginx的帐号密码授权，后续可以改为nginx的oauth授权接入EVA-Auth。docker版本的Matlab出于未知原因无法安装扩展，目前运行的是自带深度学习套件的docker版本。