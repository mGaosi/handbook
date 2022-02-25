# MP4数据结构

## 概要

## Box

Box这个概念起源于QuickTime中的atom，MP4文件就是由一个个Box组成的，可以将其理解为一个数据块，它由Header+Data组成，Data 可以存储媒体元数据和实际的音视频码流数据。Box里面可以直接存储数据块也可以嵌套包含其它类型的Box，我们把这种Box又称为Container Box。

Box的数据排列结构：

```txt
     4 Bytes     4 Bytes
  +-----------+-----------+------------>
  |  box size |  box type |  box data..  
  +-----------+-----------+------------>
   \_______ header ______/
```

下面介绍几个常用重要的Box。

### ISO Base Box

```txt
|
```

## 资料

1. <https://blog.csdn.net/teamlet/article/details/9103609> Mov文件格式分析。
