---
date: 2021/12/19
---

<img src="https://data.skywangdev.com/blog/S-5.jpeg" width="800" />


```shell
#host备份机器操作:
wsl --shutdown
#windowsterminall
 C:\Users\戴尔\Downloads\LxRunOffline.exe e -n Ubuntu-20.04 -f D:\wsl_Ubuntu.tar.gz
 #类似但不等同于wsl --export <WSL名称> <压缩包路径>.tar。LxRunOf­fline 备份完会生成一个.xml后缀的同名配置文件，比如WSL.tar.gz.xml。
#target机器操作
lxrunoffline i -n <WSL名称> -d <安装路径> -f <压缩包路径>.tar.gz
#类似但不等同于wsl --import <WSL名称> <安装路径> <压缩包路径>.tar。LxRunOf­fline 会读取备份时生成的配置文件并写入配置，前提是同目录且同名。否则你需要加入-c参数指定配置文件。
```

# 将备份的wsl文件安装到新的机器

```
D:\tools\LxRunOffline-v3.5.0-33-gbdc6d7d-msvc\LxRunOffline.exe i -n Ubuntu-20.04 -d D:\wsl -f D:\tools\wsl_Ubuntu_patch_new.tar.gz

# 进入linux ubuntu2004
id -u kingwq
# 一般uid为1000
lxrunoffline su -n <WSL名称> -v 1000(UID)
```

# 一些由于特殊字符引起的文件不能删除问题

```bat
DEL /F /A /Q \\?\%1
RD /S /Q \\?\%1
# 新建一个fiel_del.bat脚本，将文件移动到这个脚本就可以删除
```

# 关于LxRunOffLine我提出的问题并解决的

- [采用LxRunOffline工具导出wsl2后](https://github.com/DDoSolitary/LxRunOffline/issues/187)
- 迁移到其他盘后，需要修改.wslconfig文件

```
#I find the cause ; have to modify C:\Users\戴尔.wslconfig file;
[wsl2]
#kernel=C:\\temp\\myCustomKernel
memory=4GB # Limits VM memory in WSL 2 to 4 GB
processors=2 # Makes the WSL 2 VM use two virtual processors
```

# 注意下载LxRunOffline的路径，下载最新的，要不wsl转移貌似会出问题

- [下载](https://ddosolitary-builds.sourceforge.io/LxRunOffline/)

# LxRunOffline 基础命令

```
# 查看所有wsl
LxRunOffline l
# 查看安装的路径
LxRunOffline di -n Ubuntu
lxrunoffline ur -n <wsl名称>
```

# wsl出现wsl --update后显示未注册，问题解决

- 首先在设置---应用----(搜索linux)会找到最新更新的子系统，卸载掉就可以了

# 将wsl2的某个镜像设为wsl1

```
wsl -l -v
wsl --set-version  Ubuntu-18.04 1
```

# wsl2最新版本支持静态ip

- 参考:https://zhuanlan.zhihu.com/p/593263088
- 更新wsl:

```
wsl --update --pre-release
```

- 在用户目录:`%USERPROFILE%`下配置`.wslconfig`:

```
[experimental]
networkingMode=mirrored
dnsTunneling=true
firewall=true
autoProxy=true

# 备注:果你遇到 docker 无法从 Windows 访问的问题，这个是 iptables 的问题，在 /etc/docker/daemon.json 里添加一句 "iptables": false 就好了
```

# 删除wsl

```
wsl --unregister Ubuntu
```

# 导出wsl

```
wsl --export Ubuntu-22.04 C:\wsl_ws\ubuntu2204-tools.tar
```

# 导入wsl

```
wsl --import Ubuntu22.04 D:\wsl_ws\ubuntuww C:\wsl_ws\ubuntu2204-tools.tar
```

# wsl 导入后配置用户

```
su -kingwq#可以切入
cat /etc/passwd 可以看到kingwq的uuid为1000
# 默认进入wsl的用户会是root，修改DefaultUid; 修改注册表:
计算机\HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Lxss\{...}
DefaultUid值设置为子系统用户对应的gid值;十进制为1000
```

# wsl问题

```
# 安装nodejs问题
https://learn.microsoft.com/en-us/windows/dev-environment/javascript/nodejs-on-wsl
```

# wsl的配置wslconfig

```
https://learn.microsoft.com/zh-cn/windows/wsl/wsl-config#configure-settings-with-wslconfig-and-wslconf
wsl1或者wsl2采用/etc/wsl.conf
```

# wsl配置显示的语言

```
sudo dpkg-reconfigure locales
sudo apt-get install fonts-arphic-bsmi00lp fonts-arphic-gbsn00lp fonts-arphic-gkai00mp
sudo vim /etc/default/locale
# LANG=zh_CN.UTF-8 可以显示终端输出的中文
sudo apt install zh-autoconvert
sudo apt install zhcon
sudo apt install language-pack-zh-hans
locale -a || locale
```

# wsl 终端下 vim block <ctrl-v> 不生效

- 需要设置一下windows terminal的ctrl-v使用为粘贴快捷键删除。

# wsl重启

```
taskkill /f /im wslservice.exe
```

# wsl报错时相关问题解决

## 错误1，找不到wsl文件路径

```
windows terminal中配置的启动有问题，需要更改启动配置
默认是:
C:\Windows\system32\wsl.exe -d Ubuntu-22.04
实际上需要找到wsl.exe; 更改为wsl.exe的实际位置，可以解决报错
```

# wsl1不能运行m32程序

```
sudo apt-get install gcc-multilib g++-multilib
sudo apt-get install lib32stdc++6 lib32z1
gcc test.c -m32 -o test
# 程序运行错误，主要原因是wsl不支持32位运行
# wsl2上运行没有问题
```

# wsl设置从目录中进入的路径

```
需要将启动目录中的:~ 修改为: .
或者目录输入: wt /d .
```

# wsl安装离线安装包

```
mkdir ~/samba-offline && cd ~/samba-offline
sudo apt download -d libsmbclient
cd /var/cache/apt/archives/ #这里会有一些安装包 libtalloc2 libtevent0 libwbclient0
# 创建目录并下载包及依赖
sudo apt download samba-libs
sudo apt download samba-libs libtalloc2 libtevent0 libwbclient0
# 查看依赖
apt-cache depends samba-libs
# 输出:
samba-libs
  依赖: libacl1
  依赖: libavahi-client3
  依赖: libavahi-common3
  依赖: libbsd0
  依赖: libc6
  依赖: libcap2
  依赖: libcups2
  依赖: libgnutls30
  依赖: libjansson4
  依赖: libldap-2.5-0
  依赖: libldb2
  依赖: libpam0g
  依赖: libpopt0
  依赖: libpython3.10
  依赖: libtalloc2
  依赖: libtdb1
  依赖: libtevent0
  依赖: libtirpc3
  依赖: libwbclient0
  依赖: python3-ldb
  依赖: python3-talloc
  依赖: zlib1g
  破坏: <libndr-standard0>
  破坏: <libsamba-credentials0>
  破坏: <libsamba-hostconfig0>
  破坏: <libsamba-util0>
  破坏: sssd-ad-common
  替换: <libndr-standard0>
  替换: <libsamba-credentials0>
  替换: <libsamba-hostconfig0>
  替换: <libsamba-util0>
  替换: samba
#命令
  dpkg -s samba-libs | grep Depends
Depends: libacl1 (>= 2.2.23), libavahi-client3 (>= 0.6.16), libavahi-common3 (>= 0.6.16), libbsd0 (>= 0.5.0), libc6 (>= 2.34), libcap2 (>= 1:2.10), libcups2 (>= 1.7.0), libgnutls30 (>= 3.7.3), libjansson4 (>= 2.1), libldap-2.5-0 (>= 2.5.4), libldb2 (>= 2:2.4.4-0ubuntu0.22.04.2), libpam0g (>= 0.99.7.1), libpopt0 (>= 1.14), libpython3.10 (>= 3.10.0), libtalloc2 (>= 2.3.3~), libtdb1 (>= 1.4.4~), libtevent0 (>= 0.11.0~), libtirpc3 (>= 1.0.2), libwbclient0 (= 2:4.15.13+dfsg-0ubuntu1.6), python3-ldb (>= 2:2.1.0), python3-talloc (>= 2.3.1~), zlib1g (>= 1:1.1.4)
# 思路把这些包放在packages.txt.下载
xargs -a packages.txt sudo apt-get download
```

# arm-linux-gdb依赖

```
sudo echo "deb http://security.ubuntu.com/ubuntu focal-security main universe" > /etc/apt/sources.list.d/ubuntu-focal-sources.list
sudo apt install libncurses5
```