# kolla--
### kolla文档阅读笔记

----------



#### 建立容器镜像 

文档推荐在安装kolla时使用kolla的自带文件指令**tools/build.py ** ，而不是直接使用**kolla-build**安装(本文默认使用**tool/build.py**方式安装)

#### 生成文件配置

......

#### 建立kolla镜像

使用

> python tools/build.py

语句安装

default的话是默认安装CentOS镜像的，加 -b 换，如

> python  tools/build.py -b ubuntu

可以换centos、debian、oraclelinu、xrhel、ubuntu，但是**fodora被禁用了**

也可以选择不安装全部，只安装里面的一部分，使用形如如下的语句

> python tools/build.py keystone

这样写的话，在安装时，会独立安装所有含有**keystone**的镜像服务。可一次安装多个，语句如下:

> python tools/build.py keystone nova

建立镜像这个过程本身也可以用**profile**方式写好直接执行，kolla本身给了以下几个预定义的profile:

- **`infra`** 基础设置相关的镜像
- **`main`** OpenStack 核心镜像
- **`aux` ** 辅助性的镜像
- **`default`** 最小的可运行镜像

例如，假如profiles是这样写的：

> **[profiles]**
> magnum = magnum,heat  //因为magnum的使用需要heat

就可以这样写来实现镜像布置

> kolla-build --profile magnum
>
> //不确定python tools/build.py  --profile magnum 行不行

另一种写法就是在kolla-build.conf 文件的**DEFAULT**节里面加上如下的语句：

> [DEFAULT]
> profile = magnum

镜像建立的默认命名为**kolla**，可使用-n 命令来进行更改，加push指令可将其上传到Dockerhub，如下命名为mykolla:

> kolla-build -n mykolla --push

若要把镜像传到本地注册表，则需添加--registry指令：

> kolla-build --registry 172.22.2.81:5000 --push



#### 拉取openstack并建立















