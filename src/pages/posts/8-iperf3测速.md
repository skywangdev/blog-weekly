---
date: 2025/03/12
---

<img src="https://data.skywangdev.com/blog/S-10.jpeg" width="800" />


使用 iperf3 来测试不同主机之间的连接速度。

1.  安装 iperf3

```
# CentOS
yum -y install iperf3
dnf install iperf3
# Ubuntu
apt-get -y install iperf3
# Alpine
apk add iperf3
# Arch Linux
pacman -S iperf3
# Windows 版 iperf3 下载地址
# https://files.budman.pw/
# https://github.com/ar51an/iperf3-win-builds
```

2.  iperf3 测速用法

```
iperf3 -s -p 5201
```

其中

+   `-s` 参数表示服务器端
+   `-p` 指定使用端口（默认端口 5201。别忘了防火墙放行端口）
+   `-D` 以守护进程后台运行，追加 -D 参数

然后在本机发起测速。

```
iperf3 -c 1.1.1.31 -p 5201 -t 30 -P 5 -R
```

其中

+   `-c` 参数表示客户端并指定测速服务器地址
+   `-p` 指定服务器端口
+   `-t` 指定测试时长（单位秒）
+   `-P` 指定并发连接数（越高越能测试到速度极限）
+   `-R` 为反向测试，表示下载测速（不加参数则测试上传速度）
+   `-u` 要测试 UDP 连接，追加 -u 参数
+   `-i` 指定测试间隔，单位为秒

\[SUM\] 行就是测试数据（以 receiver 为准），带宽测速平均每秒 74.9 Mbits。

```
root@cloudcone-42:~# iperf3 -c 1.1.1.31 -p 5201 -i 1 -t 10 -P 5 -R
Connecting to host 1.1.1.31, port 5201
Reverse mode, remote host 1.1.1.31 is sending
[  5] local 64.69.40.42 port 55344 connected to 1.1.1.31 port 5201    # 启动了 5 了线程，开头方括号内数字为线程编号，主要为了在接下来的数据中可以分辨出各个线程。
[  7] local 64.69.40.42 port 55356 connected to 1.1.1.31 port 5201    # 五个线程都成功连接上了服务器，端口号默认5001
[  9] local 64.69.40.42 port 55366 connected to 1.1.1.31 port 5201
[ 11] local 64.69.40.42 port 55370 connected to 1.1.1.31 port 5201
[ 13] local 64.69.40.42 port 55378 connected to 1.1.1.31 port 5201
[ ID] Interval           Transfer     Bitrate
[  5]   0.00-1.00   sec  15.8 MBytes   133 Mbits/sec   # 第一列，interval，表示时间区间    
[  7]   0.00-1.00   sec  15.2 MBytes   128 Mbits/sec   # 第二列，transfer，表示传输的数据量             
[  9]   0.00-1.00   sec  25.8 MBytes   216 Mbits/sec   # 第三列，bandwidth，表示传输的带宽
[ 11]   0.00-1.00   sec  22.7 MBytes   190 Mbits/sec   
[ 13]   0.00-1.00   sec  23.3 MBytes   196 Mbits/sec   # 由于是多个线程同时运行，所以每个线程的带宽都不是真正的带宽
[SUM]   0.00-1.00   sec   103 MBytes   863 Mbits/sec   # 真正的带宽是以SUM标识的行，我们的客户端命令是1s总结一次带宽报告，所以我们把相同时间区间的线程的运行带宽相加会发现大约等于SUM的实际带宽
- - - - - - - - - - - - - - - - - - - - - - - - -
[  5]  29.00-30.00  sec  12.0 MBytes   101 Mbits/sec                  
[  7]  29.00-30.00  sec  17.6 MBytes   147 Mbits/sec                  
[  9]  29.00-30.00  sec  26.9 MBytes   226 Mbits/sec                  
[ 11]  29.00-30.00  sec  25.7 MBytes   215 Mbits/sec                  
[ 13]  29.00-30.00  sec  24.7 MBytes   207 Mbits/sec                  
[SUM]  29.00-30.00  sec   107 MBytes   896 Mbits/sec 
...                 
- - - - - - - - - - - - - - - - - - - - - - - - -
[ ID] Interval           Transfer     Bitrate         Retr
[  5]   0.00-10.00  sec   191 MBytes   160 Mbits/sec  968             sender
[  5]   0.00-10.00  sec   189 MBytes   158 Mbits/sec                  receiver
[  7]   0.00-10.00  sec   196 MBytes   165 Mbits/sec  873             sender
[  7]   0.00-10.00  sec   193 MBytes   162 Mbits/sec                  receiver
[  9]   0.00-10.00  sec   290 MBytes   244 Mbits/sec  1284             sender
[  9]   0.00-10.00  sec   287 MBytes   241 Mbits/sec                  receiver
[ 11]   0.00-10.00  sec   161 MBytes   135 Mbits/sec  744             sender
[ 11]   0.00-10.00  sec   160 MBytes   134 Mbits/sec                  receiver
[ 13]   0.00-10.00  sec   156 MBytes   131 Mbits/sec  724             sender
[ 13]   0.00-10.00  sec   155 MBytes   130 Mbits/sec                  receiver
[SUM]   0.00-10.00  sec   995 MBytes   835 Mbits/sec  4593             sender
[SUM]   0.00-10.00  sec   984 MBytes   826 Mbits/sec                  receiver
iperf Done.
```

测试结果解释：

并发流：

测试显示了 5 个并发流（流 ID 为 5, 7, 9, 11, 13），每个流的传输情况如下：

```
[ 5]
传输量：18.7 MB（约 157 Mbps）
发送速率：160 Mbps
接收速率：158 Mbps
重传：968 次
...
[ 13]
传输量：17.0 MB（约 142 Mbps）
发送速率：131 Mbps
接收速率：130 Mbps
重传：724 次
```

总和统计：

```
发送总量: 995 MB（约 835 Mbps）
接收总量: 984 MB（约 826 Mbps）
重传总数: 4593 次
```

分析：

吞吐量：通过 5 个并发流的测试，接收端总共接收了约 984 MB 的数据，发送端总共发送了约 995 MB 的数据。整体的吞吐量大致在 826 - 835 Mbps 之间，表明网络连接性能相对稳定。

重传：重传次数（4593 次）表明网络在传输过程中发生了一定的丢包。虽然丢包不算极其严重，但重传的存在说明网络并不是完全无误的。

不同流的速度差异：流 9 的传输量明显高于其他流（252 Mbps），而流 11 和流 13 的速率较低（139 Mbps 和 142 Mbps）。这可能表明网络在不同流之间的负载均衡并不完全均匀，或者某些流受到了网络瓶颈或其他因素的限制。

总结：

测试结果显示，网络连接在 10 秒内的平均速率大约为 835 Mbps（发送）和 826 Mbps（接收），速度还算较为理想。

虽然有一定的重传，但没有明显的丢包现象，且流的传输速率差异表明可能有不均匀的网络负载或其他因素影响了流的表现。