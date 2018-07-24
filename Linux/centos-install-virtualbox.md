# CentOS 安装 Vertualbox

我用的 `CentOS` 是阿里云上的 `CentOS7.4`。

在 `Docker` 的 [Get Started, Part 4: Swarms -> Create a cluster](https://docs.docker.com/get-started/part4/#create-a-cluster)中，需要安装 `Virtualbox`。

## 一|安装方式

[Download VirtualBox for Linux Hosts](https://www.virtualbox.org/wiki/Linux_Downloads) 说了各种发行版的安装方式。对于 `CentOS` 来说，最方便的就是 `yum` 安装啦。

官方文档 [RPM-based Linux distributions](https://www.virtualbox.org/wiki/Linux_Downloads#RPM-basedLinuxdistributions) 这里只是点了一下，也没讲过程，当然过程都是一样的，也写下来记录一下。

## 二|安装过程

### 1. 配置 `yum` 源

进入 /etc/yum.repos.d/ 目录：

```bash
cd /etc/yum.repos.d/
```

根据不同的 LINUX 发行版，下载不同的 `virtualbox.repo` 文件：

```bash
# Users of Oracle Linux / RHEL
wget https://download.virtualbox.org/virtualbox/rpm/el/virtualbox.repo

# Users of Fedora
wget https://download.virtualbox.org/virtualbox/rpm/fedora/virtualbox.repo
```

`CentOS` 是 `RHEL` 系的，执行第一个下载命令就可以了，下载过来的文件内容长这样：

```bash
cat /etc/yum.repos.d/virtualbox.repo

# 打印出：
[virtualbox]
name=Oracle Linux / RHEL / CentOS-$releasever / $basearch - VirtualBox
baseurl=http://download.virtualbox.org/virtualbox/rpm/el/$releasever/$basearch
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://www.virtualbox.org/download/oracle_vbox.asc
```

### 2. 更新源

```bash
yum upgrade
```

这个命令实际上会升级所有已经安装的包，甚至我的 `Docker` 版本都被更新了，感觉不太可控啊！

### 3. 安装

```bash
yum install VirtualBox-5.2
```

具体的版本，需要看 [Download VirtualBox for Linux Hosts](https://www.virtualbox.org/wiki/Linux_Downloads) 页面的说明。

到这里，可以认为已经成功地在 `CentOS` 上安装了 `VirtualBox` ！

## 三|后续问题

因为我是学习 `Docker` 的 [Get Started, Part 4: Swarms -> Create a cluster](https://docs.docker.com/get-started/part4/#create-a-cluster)，所以后面还遇到些使用上的问题。

照着教程：

```bash
docker-machine create --driver virtualbox myvm1
# 打印出：
Running pre-create checks...
Error with pre-create check: "We support Virtualbox starting with version 5. Your VirtualBox install is \"WARNING: The vboxdrv kernel module is not loaded. Either there is no module\\n         available for the current kernel (3.10.0-693.2.2.el7.x86_64) or it failed to\\n         load. Please recompile the kernel module and install it by\\n\\n           sudo /sbin/vboxconfig\\n\\n         You will not be able to start VMs until this problem is fixed.\\n5.2.14r123301\". Please upgrade at https://www.virtualbox.org"
```

同时尝试简单的命令：

```bash
virtualbox --version
# 打印出：
WARNING: The vboxdrv kernel module is not loaded. Either there is no module
         available for the current kernel (3.10.0-693.2.2.el7.x86_64) or it failed to
         load. Please recompile the kernel module and install it by

           sudo /sbin/vboxconfig

         You will not be able to start VMs until this problem is fixed.
Qt FATAL: QXcbConnection: Could not connect to display 
Aborted
```

按提示，执行 `/sbin/vboxconfig` 命令：

```bash
/sbin/vboxconfig
# 打印出：
vboxdrv.sh: Stopping VirtualBox services.
vboxdrv.sh: Starting VirtualBox services.
vboxdrv.sh: Building VirtualBox kernel modules.
This system is currently not set up to build kernel modules.
Please install the Linux kernel "header" files matching the current kernel
for adding new hardware support to the system.
The distribution packages containing the headers are probably:
    kernel-devel kernel-devel-3.10.0-693.2.2.el7.x86_64
This system is currently not set up to build kernel modules.
Please install the Linux kernel "header" files matching the current kernel
for adding new hardware support to the system.
The distribution packages containing the headers are probably:
    kernel-devel kernel-devel-3.10.0-693.2.2.el7.x86_64

There were problems setting up VirtualBox.  To re-start the set-up process, run
  /sbin/vboxconfig
as root.
```

按提示安装 `kernel-devel-3.10.0-693.2.2.el7.x86_64`：

```bash
yum install kernel-devel-3.10.0-693.2.2.el7.x86_64
# 打印出：
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * ius: mirrors.tuna.tsinghua.edu.cn
No package kernel-devel-3.10.0-693.2.2.el7.x86_64 available.
Error: Nothing to do
```

提示 `kernel-devel-3.10.0-693.2.2.el7.x86_64` 不存在！变通一下，发现 `yum list kernel-devel` 有更新的版本：

```bash
yum list kernel-devel
# 打印出：
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
 * ius: mirrors.tuna.tsinghua.edu.cn
Available Packages
kernel-devel.x86_64     3.10.0-862.6.3.el7      updates
```

安装 `kernel-devel`：

```bash
yum install kernel-devel

# 安装后重启
reboot

# 重启后继续前面的步骤
/sbin/vboxconfig

# 继续创建VMs
docker-machine create --driver virtualbox myvm1
```

然而又报错了！报错提示：

```bash
docker-machine create --driver virtualbox myvm1
# 打印出：
Running pre-create checks...
Error with pre-create check: "This computer doesn't have VT-X/AMD-v enabled. Enabling it in the BIOS is mandatory"
```

恩，提示没开启 `VT-X/AMD-v`，经查 [GIthub issue: This computer doesn't have VT-X/AMD-v enabled. Enabling it in the BIOS is mandatory](https://github.com/docker/machine/issues/427)，创建 VMs 时不检查虚拟化是否开启，命令改为如下：

```bash
docker-machine create --driver virtualbox --virtualbox-no-vtx-check myvm1
```

偶尔下载连接超时，重试几次就可以了。

然而，下载 `boot2docker.iso` 文件成功了，创建 `machine` 还是失败了！一直阻塞在获取 IP 的阶段，最终报错：

```bash
docker-machine create --driver virtualbox --virtualbox-no-vtx-check myvm1
# 打印：
Running pre-create checks...
(myvm1) Default Boot2Docker ISO is out-of-date, downloading the latest release...
(myvm1) Latest release for github.com/boot2docker/boot2docker is v18.06.0-ce
(myvm1) Downloading /root/.docker/machine/cache/boot2docker.iso from https://github.com/boot2docker/boot2docker/releases/download/v18.06.0-ce/boot2docker.iso...
(myvm1) 0%....10%....20%....30%....40%....50%....60%....70%....80%....90%....100%
Creating machine...
(myvm1) Copying /root/.docker/machine/cache/boot2docker.iso to /root/.docker/machine/machines/myvm1/boot2docker.iso...
(myvm1) Creating VirtualBox VM...
(myvm1) Creating SSH key...
(myvm1) Starting the VM...
(myvm1) Check network to re-create if needed...
(myvm1) Waiting for an IP...

Error creating machine: Error in driver during machine creation: Too many retries waiting for SSH to be available.  Last error: Maximum number of retries (60) exceeded
```

`docker-machine ls` 命令查看，发现是报错了：

```bash
docker-machine ls
# 打印：
NAME    ACTIVE   DRIVER       STATE     URL   SWARM   DOCKER    ERRORS
myvm1   *        virtualbox   Running                 Unknown   ssh command error:
command : ip addr show
err     : exit status 255
output  :
```

google N 遍也没有结果，可能是因为我的阿里云ECS不是独立的服务器，是共享型的ECS吧！（前面步骤没法开启 `VT-X/AMD-v`）

最后，还是乖乖地回到实体的宿主机的 `CentOS`上，开启 `VT-X/AMD-v`，然后终于跑通了整个教程！

## 四|参考

[CentOS 7 安装Oracle VirtualBox](https://blog.csdn.net/m0_37904728/article/details/78601868)

[This computer doesn't have VT-X/AMD-v enabled. Enabling it in the BIOS is mandatory](https://github.com/docker/machine/issues/4271)
