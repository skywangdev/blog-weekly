---
date: 2024/09/17
---

<img src="https://data.skywangdev.com/blog/S-9.jpeg" width="800" />


##### [](#合并一个文件夹内的所有视频 "合并一个文件夹内的所有视频")合并一个文件夹内的所有视频

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br></pre></td><td class="code"><pre><span class="line">find *.mp4 | sed 's:\ :\\\ :g'| sed 's/^/file /' &gt; fl.txt</span><br><span class="line">ffmpeg -f concat -i fl.txt -c copy output.mp4</span><br><span class="line">// 忽略错误信息</span><br><span class="line">ffmpeg -safe 0 -f concat -i fl.txt -c copy output.mp4</span><br><span class="line">rm fl.txt</span><br></pre></td></tr></tbody></table>

[参考资源](https://stackoverflow.com/questions/28922352/how-can-i-merge-all-the-videos-in-a-folder-to-make-a-single-video-file-using-ffm)

##### [](#视频压缩 "视频压缩")视频压缩

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br><span class="line">4</span><br><span class="line">5</span><br><span class="line">6</span><br></pre></td><td class="code"><pre><span class="line">// 视频使用h.264编码，声音使用aac编码</span><br><span class="line">ffmpeg -i input.mp4 -vcodec h264 -acodec aac output.mp4</span><br><span class="line">// 视频使用h.265编码，压缩到更小文档</span><br><span class="line">ffmpeg -i input.mp4 -vcodec libx265 -crf 28 output.mp4</span><br><span class="line">// 视频使用h.264编码，保留更好的质量</span><br><span class="line">ffmpeg -i input.mp4 -vcodec libx264 -crf 20 output.mp4</span><br></pre></td></tr></tbody></table>

crf 越小，视频质量越高；crf 越大，视频文件越小

编码参数也可以简写，从`-vcodec`和`-acodec`改为`-c:v`和`-c:a`：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br><span class="line">3</span><br></pre></td><td class="code"><pre><span class="line">ffmpeg -i input.mp4 -c:v libx264 -crf 23 output.mp4</span><br><span class="line">ffmpeg -i input.mp4 -c:v libx265 -crf 28 output.mp4</span><br><span class="line">ffmpeg -i input.mp4 -c:v libvpx-vp9 -crf 31 -b:v 0 output.mkv</span><br></pre></td></tr></tbody></table>

[参考资源](https://slhck.info/video/2017/02/24/crf-guide.html)

其中`AVC/H264`和`HEVC/H265`都是软件编码，速度很慢。可以选择英伟达的硬件编码：hevc_nvenc 与 h264_nvenc，它们使用硬件加速，速度很快。

[参考资源](https://www.bilibili.com/opus/376578377423593855)

使用英伟达显卡进行编码：

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">ffmpeg -i video.mp4 -c:v hevc_nvenc -crf 28 output.mp4</span><br></pre></td></tr></tbody></table>

将视频从 H.264 转码到 H.265，花了 55 分钟，视频体积从 3.8GB 减小到 430MB，效果立竿见影。转码命令：`ffmpeg -i 1.mp4 -c:v libx265 -vtag hvc1 -c:a copy 1_hevc.mp4`

在 win10 可以用 scoop 安装 ffmpeg，更新 Windows 上面通过 scoop 安装的所有程序  
`scoop list | foreach { scoop update $_.Name }`。

将视频以同样的编码，按照指定时间进行裁剪

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">ffmpeg -ss 00:05 -to 08:53.500 -i ./input.mp4 -c copy video.mp4</span><br></pre></td></tr></tbody></table>

利用 ffmpeg 快速剪辑视频

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">ffmpeg -ss 07:18 -to 13:45 -i ./aaa.mkv -c copy bbb.mkv</span><br></pre></td></tr></tbody></table>

- \-ss 表示开始时间
- \-to 表示结束时间
- \-i 是输入文档
- \-c 表示使用被剪辑视频一样的编码
- bbb 是输出文档的名称

合并视频和声音，视频使用原始编码，声音改为 aac 编码

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">ffmpeg -i 1.mp4 -i 1.opus -c:v copy -c:a aac output.mp4</span><br></pre></td></tr></tbody></table>

将 PNG 格式图片转为 JPG 格式图片

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">ffmpeg -i image.png -preset ultrafast image.jpg</span><br></pre></td></tr></tbody></table>

修改图片的尺寸

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br><span class="line">2</span><br></pre></td><td class="code"><pre><span class="line">ffmpeg -i image.jpeg -vf scale=413:626 2寸.jpeg</span><br><span class="line">ffmpeg -i image.jpeg -vf scale=390:567 1寸.jpeg</span><br></pre></td></tr></tbody></table>

将一个音频重复 10 次

<table><tbody><tr><td class="gutter"><pre><span class="line">1</span><br></pre></td><td class="code"><pre><span class="line">ffmpeg -stream_loop 10 -i input.m4a -c copy output.m4a</span><br></pre></td></tr></tbody></table>
