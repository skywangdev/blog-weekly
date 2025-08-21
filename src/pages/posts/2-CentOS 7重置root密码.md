---
date: 2020/05/12
---

<img src="https://gw.alipayobjects.com/zos/k/p4/234.jpg" width="800" />

<small>封面图拍摄于周末拿着新到的富士 XE5 出门拍照的场景，哈哈，主要是给潮流周刊拍封面图，质感非常喜欢，照片直出拍出来也很讨喜，总之非常喜欢这个小玩具。</small>

> 在centos服务器中，普通用户拥有的权限是被root（超级管理员）限定的。有时在下载centos中下载安装软件时很不方便，所以需要进入root（超级管理员)的用户界面，但是有时会忘记root用户的密码，那么就需要进行重置root用密码。centos的版本很多，但是重置root用户的密码的方法都是相似的，都是进入单用户模式修改root密码。

## 重置root密码具体步骤

1. 重启服务器，进入系统加载页面后按键盘 e键，进入编辑模式
<img src="https://data.skywangdev.com/centos7-root-reset-1.png" width="800" />

2. 移动光标，找到 linux16开头 的那行，从末尾开始删除到 ro 停止

<img src="https://data.skywangdev.com/centos7-root-reset-2.png" width="800" />

3. 并将把 ro 改为 rw，空一格后，追加 rd.break，然后按 ctrl+x 进行引导启动

``` 

修改前：linux16 /vmlinuz-3.10.0-1160.e17.x86_64 root=/dev/mapper/centos-root ro crashkernel=auto rd.lvm.lv=centos/root rd.lvm.lv=centos/swap rhgb quiet LANG=en_US.UTF-8
修改后：linux16 /vmlinuz-3.10.0-1160.e17.x86_64 root=/dev/mapper/centos-root rw rd.break

```

结果如下：

<img src="https://data.skywangdev.com/centos7-root-reset-3.png" width="800" />

4. 进入单用户模式

<img src="https://data.skywangdev.com/centos7-root-reset-4.png" width="800" />

5. 输入如下命令，重新挂载根目录并重置root密码（比如改为 Aa@123456），最后重启系统生效

```
mount -o remount,rw /sysroot
chroot sysroot
echo 'Aa@123456' | passwd root --stdin
touch /.autorelabel     
exit
reboot

```

过程如下：

<img src="https://data.skywangdev.com/centos7-root-reset-5.png" width="800" />

6. 等待一段时间后，重新进入登陆界面时就可以使用刚才设置的密码以 root 登陆了。

