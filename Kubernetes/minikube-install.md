# 安装 Minikube

see [Install Minikube](https://kubernetes.io/docs/tasks/tools/install-minikube/)

整体大步骤是：

- 安装管理软件 // 安装虚拟机支持
- 安装 kubectl // [Install kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl/)
- Install Minikube // [Install Minikube](https://github.com/kubernetes/minikube/releases)

照着官方文档一步步来就好。

Minikube 使用说明见 [README](https://github.com/kubernetes/minikube/blob/v0.26.1/README.md)

以下是自己记录的一些快速安装的步骤/命令，每次都跟文档也挺累。

## 一、笔记：Centos 安装 Minikube

### Centos 安装 kubectl

```bash
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

# 检查安装结果
kubectl version
kubectl cluster-info
```

### Centos 安装 minikue

```bash
# 版本号要改成最新的
curl -Lo minikube https://storage.googleapis.com/minikube/releases/v0.26.1/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```

### 安装 [KVM2 driver](https://github.com/kubernetes/minikube/blob/master/docs/drivers.md#kvm2-driver)

```bash
# Install libvirt and qemu-kvm on your system, e.g.
sudo yum install libvirt-daemon-kvm qemu-kvm

# Add yourself to the libvirtd group (use libvirt group for rpm based distros) so you don't need to sudo
sudo usermod -a -G libvirt $(whoami)

# Update your current session for the group change to take effect
newgrp libvirt

# Then install the driver itself:
curl -LO https://storage.googleapis.com/minikube/releases/latest/docker-machine-driver-kvm2 && chmod +x docker-machine-driver-kvm2 && sudo mv docker-machine-driver-kvm2 /usr/local/bin/
```

启动时要指定 driver

```bash
minikube start --vm-driver kvm2
```

也可以不安装 KVM2，直接在宿主机跑Kubernetes，要指定驱动为 none：

```bash
minikube start --vm-driver none
```

## 二、笔记：Windows 安装 Minikube

### Win 安装 kubectl

```bash
# 查看最新版本
https://storage.googleapis.com/kubernetes-release/release/stable.txt

# 把 {last-version} 替换成最新版本下载
curl -LO https://storage.googleapis.com/kubernetes-release/release/{last-version}/bin/windows/amd64/kubectl.exe

# 为以上 kubectl.exe 设置环境变量 PATH

# 检查安装结果
kubectl version
kubectl cluster-info
```

### Win 安装 minikube

```bash
# 下载 minikube-windows-amd64.exe 文件，重名重成 minikube.exe
# 为以上 minikube.exe 设置环境变量 PATH

# 版本号要改成最新的
https://github.com/kubernetes/minikube/releases/download/v0.26.1/minikube-windows-amd64
```

### 安装 [VirtualBox](https://www.virtualbox.org/wiki/Downloads) or [Hyper-V](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install)

## 三、关键点/教训

因为虚拟机是另外一台主机了，宿主机上的 VPN 并不能被该虚拟机也具备翻墙的能力！（也许通过配置 VirtualBox 的虚拟机网络连接可以做到，没深入研究）

[Kubernetes牆內使用技巧](http://blog.samemoment.com/articles/kubernetes/)这篇文章启发了我！

其实官方文档 [Create a Minikube cluster](https://kubernetes.io/docs/tutorials/stateless-application/hello-minikube/#create-a-minikube-cluster)  和 [Using Minikube with an HTTP Proxy](https://kubernetes.io/docs/getting-started-guides/minikube/#using-minikube-with-an-http-proxy) 都提到了设置代理！

```bash
minikube start --docker-env HTTP_PROXY=http://192.168.1.118:1081 --docker-env HTTPS_PROXY=https://192.168.1.118:1081
```

以上命令中的 `IP` 和 `端口`，对应改成实际的宿主机上的 `IP` 和 `VPN 的代理端口` 即可！