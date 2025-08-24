---
date: 2021/04/13
---

<img src="https://data.skywangdev.com/blog/S-3.jpeg" width="800" />

# 一、前期准备

## 1.1、查看GPU信息

```shell
lspci | grep -i nvidia
```



## 1.2、配置内核

```shell
dnf install -y gcc dkms kernel-devel-$(uname -r) kernel-headers-$(uname -r)
```

- 安装的版本要和当前内核版本一致



## 1.3、禁用nouveau

```shell
# 查看nouveau
lsmod | grep nouveau

# 禁用nouveau
cat >  /etc/modprobe.d/blacklist.conf << EOF
blacklist nouveau
options nouveau modeset=0
EOF
```



## 1.4、更新initramfs

```shell
# AlmaLinux/RockyLinux
mv /boot/initramfs-$(uname -r).img /boot/initramfs-$(uname -r).img.bak
dracut /boot/initramfs-$(uname -r).img $(uname -r)

# Ubuntu
sudo update-initramfs -u
```

- 此步骤完成后，重启操作系统再进行下一步。



# 二、安装驱动

## 2.1、下载驱动

从[NVIDIA 驱动程序下载](https://www.nvidia.cn/Download/index.aspx?lang=cn)下载对应显卡的驱动程序，建议使用.run可执行文件。如需安装CUDA工具包（CUDA工具包内置了驱动），可以跳过此步骤安装。



## 2.2、安装驱动

```shell
bash NVIDIA-Linux-x86_64-470.256.02.run
或
bash NVIDIA-Linux-x86_64-470.256.02.run --kernel-source-path=/usr/src/kernels/$(uname -r) -k $(uname -r)
```



## 2.3、验证安装

```shell
nvidia-smi
```

返回GPU相关信息，即表示安装成功。



# 三、安装CUDA工具包

## 3.1、下载cuda安装包

访问[CUDA](https://developer.nvidia.com/cuda-toolkit-archive)选择与GPU匹配的操作系统和版本。因CUDA工具包包含驱动程序，可以跳过第二步，直接执行CUDA工具包安装。[CDDA工具包对应的驱动版本](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/index.html)。



## 3.2、安装cuda

```shell
bash cuda_11.4.0_470.256.02_linux.run
```

- 如果已安装驱动，请务必取消驱动安装选项，否则安装可能失败



## 3.3、验证安装

```shell
/usr/local/cuda/bin/nvcc -V
```

返回cuda版本信息，即表示安装成功。



# 四、安装nvidia-fabricmanager

## 4.1、添加软件源

```shell
# AlmaLinux/RockyLinux
# 根据自己系统版本添加对应版本源
dnf config-manager --add-repo http://developer.download.nvidia.com/compute/cuda/repos/rhel8/x86_64/cuda-rhel8.repo

# Ubuntu
# 根据自己系统版本添加对应版本源
wget https://developer.download.nvidia.cn/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin

mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600

wget https://developer.download.nvidia.cn/compute/cuda/repos/ubuntu2004/x86_64/7fa2af80.pub

apt-key add 7fa2af80.pub

rm 7fa2af80.pub

echo "deb http://developer.download.nvidia.cn/compute/cuda/repos/ubuntu2004/x86_64 /" | tee /etc/apt/sources.list.d/cuda.list
```



## 4.2、安装nvidia-fabric-manager

```shell
# AlmaLinux/RockyLinux
dnf module enable -y nvidia-driver:470
dnf install -y nvidia-fabric-manager:470.256.02 nvidia-fabric-manager-devel-0:470.256.02

# Ubuntu
apt-get update
apt-get -y install nvidia-fabricmanager-470=470.256.02-1
```



## 4.3、启动服务

```shell
systemctl start nvidia-fabricmanager
systemctl status nvidia-fabricmanager
systemctl enable nvidia-fabricmanager
```



## 4.4、验证

```shell
nvidia-smi topo -m
```

返回结果中有`NV*` 字样表示 GPU 之间有 NVLink 连接。如果所有预期的 GPU 之间都有 NVLink 连接，并且没有错误信息，那么 NVLink 应该是运行正常的。



