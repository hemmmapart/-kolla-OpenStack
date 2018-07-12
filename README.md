## kolla 相关笔记
--------
前半部分为kolla ansible部署  后面为kolla定制化部署  
等着边安装实验室30台机子边总结  未完待续
## 快速使用安装 kolla 安装 OpenStack

主机最低配置：

* 2个网络接口 

* 8 GB 主存 

* 40GB 磁盘空间

root 权限必须

  ### 安装前准备事项

pip 升到最新版本，用的 CentOS，指令为：

```bash 
yum install epel-releaseyum install python-pippip install -U pip 
//ubuntu 除使用指令不同外，还有调用的库的名称也不同，具体查阅英文文档
```

**下载必备的软件和库文件**    //For CentOS
```bash
yum install python-devel libffi-devel gcc openssl-devel libselinux-python
```

安装  Ansible 时需要注意，一些 Ansible 版本过老，不能使用发行版本，Ansible2.2适用于本文的CentOS系统。写法如下：
```bash
yum install ansible
```

往 /etc/ansible/ansible.cfg 里加配置文件
```ini
[defaults]
host_key_checking=False
pipelining=True
forks=100
```
### 开始安装
**安装部署、评估组件**
使用 pip 安装 kolla-ansible 及其组件
```bash
pip install kolla-ansible
```
复制 globals.yml 和 passwords.yml 到 / etc / kolla 目录。
```bash
cp -r /usr/share/kolla-ansible/etc_examples/kolla /etc/
```
整合文件到当前目录下
```bash
cp /usr/share/kolla-ansible/ansible/inventory/* .
```
**安装开发相关东西**
克隆 kolla 和 kolla-来自 git 的可扩展存储库。
```bash
git clone https://github.com/openstack/kolla
git clone https://github.com/openstack/kolla-ansible
```
安装 kolla 和 kolla-ansible所需的 txt 文件
```bash
pip install -r kolla/requirements.txt
pip install -r kolla-ansible/requirements.txt
```

将配置文件复制到 / etc / kolla 目录（globals.yml 和 passwords.yml）。 

```bash
mkdir -p /etc/kolla
cp -r kolla-ansible/etc/kolla/* /etc/kolla
```
将库存文件复制到工作目录。
```
cp kolla-ansible/ansible/inventory/* .
```
文件配置部分








----------



#### 建立容器镜像 



文档推荐在安装 kolla 时使用 kolla 的自带文件指令 **tools/build.py ** ，而不是直接使用 **kolla-build** 安装(本文默认使用 **tool/build.py** 方式安装)

#### 生成文件配置

......

#### 建立kolla镜像

使用

> python tools/build.py

语句安装

default 的话是默认安装 CentOS 镜像的，加` -b` 换，如

> python  tools/build.py -b ubuntu

可以换 `centos`  `debian `  ` oraclelinu`  `xrhel ` `ubuntu` ，但是 **fodora 被禁用了**

也可以选择不安装全部，只安装里面的一部分，使用形如如下的语句

> python tools/build.py keystone

这样写的话，在安装时，会独立安装所有含有 **keystone** 的镜像服务。可一次安装多个，语句如下:

> python tools/build.py keystone nova

建立镜像这个过程本身也可以用 **profile** 方式写好直接执行，kolla 本身给了以下几个预定义的 profile:

- **`infra`** 基础设置相关的镜像
- **`main`** OpenStack 核心镜像
- **`aux`** 辅助性的镜像
- **`default`** 最小的可运行镜像

例如，假如 profiles 是这样写的：

> **[profiles]**
> 
> magnum = magnum,heat  //因为 magnum 的使用需要 heat

就可以这样写来实现镜像布置

> kolla-build --profile magnum
>
> //我觉得 python tools/build.py  --profile magnum 也行

另一种写法就是在` kolla-build.conf` 文件的 `DEFAULT `节里面加上如下的语句：

> [DEFAULT]
> 
> profile = magnum

镜像建立的默认命名为 **kolla**，可使用 `-n` 命令来进行更改，加 `--push` 指令可将其上传到 Dockerhub，如下命名为 mykolla:

> kolla-build -n mykolla --push

若要把镜像传到本地注册表，则需添加`--registry`指令：

> kolla-build --registry 172.22.2.81:5000 --push



#### 拉取 openstack 并建立

OpenStack 有两种安装方式，二进制和源码，默认为使用二进制串安装，可加 `-t` 指令更换：

> python tools/build.py -t source

源码在 **etc/kolla/kolla-build.conf** 里，支持`url  ` `git` `local`三种安装方式。

若要建立 redhat 旗下系统的镜像，需记录账号信息，使用模板的头重写文件，写法如下：

> RUN subscription-manager register --user=<user-name> \
> --password=<password> && subscription-manager attach --pool <pool-id>



#### Dockerfile 定制化更改

kolla-build tool 支持 `Jinja2`，所以可对 `Dockerfile` 进行定制化更改，由此可进行安装额外的工具包、调整设置、安装插件等工作。

##### 一般的定制化

一切语句形如 **{% block ... %} ** 的都可以进行更改，以下是一个对Horizen Dockerfile 进行更改的实例。

此处定制化Horizon组件，首先建立一个文件来包含定制化内容，形如对 template-overrides.j2 ，文件写法形如：

> ```
> {% extends parent_template %}
> 
> # Horizon
> {% block horizon_redhat_binary_setup %}
> RUN useradd --user-group myuser
> {% endblock %}
> ```

然后使用以下的指令重建 Horizon 组件

> python tools/build.py --template-override template-overrides.j2 horizon

##### 安装包定制







