---
date: 2023/05/23
---

<img src="https://data.skywangdev.com/blog/S-8.jpeg" width="800" />


**一、需求描述**

服务器前两个硬盘是480G的SSD固态硬盘，后面4块硬盘是4T的机械硬盘，前两块SSD做RAID1安装系统，后四块机械硬盘做RIAD5放数据。

**二、创建RAID1和创建RAID5**

2.1、服务器一开机后，看到如下界面后，按F2进入System Setup。

[![](https://yan-jian.com/file/20231208/1.png)](https://yan-jian.com/file/20231208/1.png)

2.2、进入System Setup后，找到Device Settings。

[![](https://yan-jian.com/file/20231208/2.png)](https://yan-jian.com/file/20231208/2.png)

2.3、找到Integrated RAID Controller1\\:Dell\\Configuration Utility。选中后，点击进入。

[![](https://yan-jian.com/file/20231208/3.png)](https://yan-jian.com/file/20231208/3.png)

**注：上图就是服务器的RAID阵列卡。**

H740P mini阵列卡到底是什么样子的呢？如下图所示：

[![](https://yan-jian.com/file/20231208/4.png)](https://yan-jian.com/file/20231208/4.png)

H740P Mini阵列卡主要有一个8G的快速容量。这个是RAID卡可以作为服务器硬盘之间读写数据的缓存。

[![](https://yan-jian.com/file/20231208/5.png)](https://yan-jian.com/file/20231208/5.png)

2.4、然后选择Main Menu，主菜单。

[![](https://yan-jian.com/file/20231208/6.png)](https://yan-jian.com/file/20231208/6.png)

2.5、首先查看一下，硬盘是否都正常在线，我们选择Physical Disk Management。

[![](https://yan-jian.com/file/20231208/7.png)](https://yan-jian.com/file/20231208/7.png)

2.6、可以看到2块480G的固态盘已经Ready状态，准备就绪。4块4TB机械硬盘也已经Ready状态，准备就绪，返回上一级。

[![](https://yan-jian.com/file/20231208/8.png)](https://yan-jian.com/file/20231208/8.png)

2.7、选择Configuration Management。

[![](https://yan-jian.com/file/20231208/9.png)](https://yan-jian.com/file/20231208/9.png)

2.8、选择Create Virtual Disk。

[![](https://yan-jian.com/file/20231208/10.png)](https://yan-jian.com/file/20231208/10.png)

2.9、Select RAID Level选择RAID1。

[![](https://yan-jian.com/file/20231208/11.png)](https://yan-jian.com/file/20231208/11.png)

2.10、RAID等级已经选择RAID1。

[![](https://yan-jian.com/file/20231208/12.png)](https://yan-jian.com/file/20231208/12.png)

2.11、下面开始选择前两块SSD固态盘，点击Select Physical Disks。

[![](https://yan-jian.com/file/20231208/13.png)](https://yan-jian.com/file/20231208/13.png)

2.12、选择前两块SSD的480G的固态盘。使用键盘的上下键，选中一块硬盘后按空格键，就可以将硬盘前面的勾选中。两块硬盘选择完成后，点击Apply Changes，应用。

[![](https://yan-jian.com/file/20231208/14.png)](https://yan-jian.com/file/20231208/14.png)

2.13、点击OK。

[![](https://yan-jian.com/file/20231208/15.png)](https://yan-jian.com/file/20231208/15.png)

2.14、配置RAID1参数。

[![](https://yan-jian.com/file/20231208/16.png)](https://yan-jian.com/file/20231208/16.png)

\*\*Virtual Disk Name：\*\*为RAID1的虚拟磁盘命名，按自己需求命名即可。

\*\*Virtual Disk Size：\*\*空间大小，选择默认全部空间。

\*\*Virtual Disk Size Unit：\*\*选择磁盘大小的单位。

**Strip Element Size：**

条带大小，选择默认的256KB。条带的大写如何设置呢？基本上如果是数据库服务器应用选大小选择4-16KB，对于大文件，CAD，渲染大图文件建议设置128KB以上，WEB服务器文件打印服务器建议设置16-64KB即可。

**Read Policy：**

选择Read Ahead模式：预读模式，选择预读的模式优点是，当用户读数据的时候，硬盘将数据调用到RAID卡8G缓存中，进行用户数据交付，不用是用户直接读取硬盘中的数据。

如果选择No Read Ahead模式：那用户读数据，直接读硬盘中数据，相对于Read Ahead会慢一些。

**Wirte Policy:**

选择 Write Back模式： 回写模式，如果用户往硬盘里写数据，不是直接写到硬盘里面的，而是先将数据写到RAID卡的8G缓存中，然后缓存再将用户数据写到硬盘中，这样速度会更快。

Wirte Through模式：用户直接将数据写到硬盘当中，不写到RAID卡的缓存当中，这个模式会比Wirte Back慢一些。

Force Wirte Back模式：强制回写是什么意思呢？这里要说到RAID卡了，RAID卡是有电池来供电的，一般的这个电池可以用到3年左右，无论这个RAID卡有没有电，数据都是先写到RAID卡缓存后转到硬盘中，但如果三年后，正好RAID电池没电了，正好在往服务器写数据，数据正在写到缓存中，这时突然断电了，正在写向服务器数据会丢失。不建议选择Force Wirte Back。

如果选择Wiret Back模式，RAID卡电池没电了，这时会策略会自动更改为Wirte Through模式。

Disk Cahce：选择Default。

Default Initilization：初始化，选择Fast，快速初始化硬盘，不需要等RAID1创建完成后，就可以在RAID1中直接安装系统。如果选择了Full，那就必须等RAID1创建完成后，才可以在RAID1中安装系统，如何查看RAID1的初始化进度，在文章尾部查看。

2.15、选中confirm，然后点击Yes。

[![](https://yan-jian.com/file/20231208/17.png)](https://yan-jian.com/file/20231208/17.png)

2.16、点击OK。

[![](https://yan-jian.com/file/20231208/18.png)](https://yan-jian.com/file/20231208/18.png)

2.17、点击OK后，会返回到上级界面 ，继续创建RAID5。

Select RAID Level，选择RAID5。

[![](https://yan-jian.com/file/20231208/19.png)](https://yan-jian.com/file/20231208/19.png)

2.18、已经选中RAID5后，选择Select Physcal Disks。

[![](https://yan-jian.com/file/20231208/20.png)](https://yan-jian.com/file/20231208/20.png)

2.19、选择剩下的4块4TB机械硬盘。然后选择Apply Changes。

[![](https://yan-jian.com/file/20231208/21.png)](https://yan-jian.com/file/20231208/21.png)

2.20、点击OK。

[![](https://yan-jian.com/file/20231208/22.png)](https://yan-jian.com/file/20231208/22.png)

2.21、下面的RAID5的参数和RAID1同样。然后选择Create Virutal Disk。

[![](https://yan-jian.com/file/20231208/23.png)](https://yan-jian.com/file/20231208/23.png)

2.22、选中Confirm后，点击Yes。

[![](https://yan-jian.com/file/20231208/24.png)](https://yan-jian.com/file/20231208/24.png)

2.23、点击Ok。

[![](https://yan-jian.com/file/20231208/25.png)](https://yan-jian.com/file/20231208/25.png)

2.24、硬盘已经被创建完了，点击back。到这一步，RAID1和RAID5已经创建完成了。

[![](https://yan-jian.com/file/20231208/26.png)](https://yan-jian.com/file/20231208/26.png)

**三、校验查看**

3.1、查看RAID创建

返回到Main Menu主界面，找到Configuration Management。

[![](https://yan-jian.com/file/20231208/27.png)](https://yan-jian.com/file/20231208/27.png)

选择View Disk Group Properties。

[![](https://yan-jian.com/file/20231208/28.png)](https://yan-jian.com/file/20231208/28.png)

这时，可以看到两个Disk Group。一个是RAID1另外一个是RAID5。两个RAID已经创建完成。

[![](https://yan-jian.com/file/20231208/29.png)](https://yan-jian.com/file/20231208/29.png)

3.2、查看RAID初始化进度

返回到Main Menu中，找到Virtual Disk Management。

[![](https://yan-jian.com/file/20231208/30.png)](https://yan-jian.com/file/20231208/30.png)

这时可以看到RAID1正在初始化，初化到了12%

[![](https://yan-jian.com/file/20231208/31.png)](https://yan-jian.com/file/20231208/31.png)

初始化到16%，在创建RAID1选择参数时，如果选择RAID硬盘初始化选择Full的情况下，就必须要等RAID1，初始化到100%后，才可以进行操作系统的安装。

[![](https://yan-jian.com/file/20231208/32.png)](https://yan-jian.com/file/20231208/32.png)