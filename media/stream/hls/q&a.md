# Q&A

## HLS协议是如何实现多码率和多音轨的？

> 原文链接：<https://bbs.huaweicloud.com/blogs/152607>

在一级索引文件中放置不同码率的m3u8索引文件位置，二级索引文件中再放置媒体分片。播放器下载一级索引文件后，可以根据网络状态和用户喜好选择一个码率进行下载和播放。一个简单的一级索引如下：

```m3u
#EXTM3U
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="aac",NAME="English", \
DEFAULT=YES,AUTOSELECT=YES,LANGUAGE="en", \
URI="main/english-audio.m3u8"
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="aac",NAME="Deutsch", \
DEFAULT=NO,AUTOSELECT=YES,LANGUAGE="de", \
URI="main/german-audio.m3u8"
#EXT-X-MEDIA:TYPE=AUDIO,GROUP-ID="aac",NAME="Commentary", \
DEFAULT=NO,AUTOSELECT=NO,LANGUAGE="en", \
URI="commentary/audio-only.m3u8"
#EXT-X-STREAM-INF:BANDWIDTH=1280000,CODECS="...",AUDIO="aac"
low/video-only.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=2560000,CODECS="...",AUDIO="aac"
mid/video-only.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=7680000,CODECS="...",AUDIO="aac"
hi/video-only.m3u8
#EXT-X-STREAM-INF:BANDWIDTH=65000,CODECS="mp4a.40.5",AUDIO="aac"
main/english-audio.m3u8
```

在这个索引文件中，除了有low/video-only.m3u8、mid/video-only.m3u8和hi/video-only.m3u8三路不同码率索引文件多信息，还有三路不同语种的音轨的索引文件的信息，根据这些信息，播放器就可以支持多码率和多音轨了。
