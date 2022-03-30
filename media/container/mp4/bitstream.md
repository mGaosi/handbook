# MP4数据结构

## 概要

## Box

Box这个概念起源于QuickTime中的atom，MP4文件就是由一个个Box组成的，可以将其理解为一个数据块，它由Header+Data组成，Data 可以存储媒体元数据和实际的音视频码流数据。Box里面可以直接存储数据块也可以嵌套包含其它类型的Box，我们把这种Box又称为Container Box。

Box的数据排列结构：

```txt
               +—————+—————+—...—+—————+
               | Box | Box | ... | Box |
               +—————+—————+—...—+—————+
                \                    /
                 \                  /
                  \                /
                   \              /
                    \            /
        +———————————+———————————+
        | Header    | Body      |
        +———————————+———————————+
       /             \
      /               \
     /                 +-----------------------------------+
    /                                                       \
   /                                                         \
  |  4 Bytes     4 Bytes  |   8 Bytes        |  1Byte   3Bytes| 
  +———————————+———————————+------------------+--------+-------+
  |  box size |  box type | large size       | version| flags | 
  +———————————+———————————+------------------+--------+-------+
   \_______ header ______/ \_ box_size = 0 _/ \__ full_box __/ 
```

### BoxHeader

字段定义如下：

- box type：box类型，包括 “预定义类型”、“自定义扩展类型”，占4个字节；
  预定义类型：比如ftyp、moov、mdat等预定义好的类型；
  自定义扩展类型：如果type==uuid，则表示是自定义扩展类型。type（或largesize）随后的16字节，为自定义类型的值（extended_type）。
- box size：包含box header在内的整个box的大小，单位是字节。当size为0或1时，需要特殊处理：
  size等于0：当前box为文件的最后一个box，通常包含在mdat box中；
  size等于1：box的大小由后续的largesize确定（一般只有装载媒体数据的mdat box会用到largesize）。
- largesize：box的大小，占8个字节。
- extended_type：自定义扩展类型，占16个字节。

下面介绍几个常用重要的Box。

### ISO Base Box

```txt
|
```

## 资料

1. <https://blog.csdn.net/teamlet/article/details/9103609> Mov文件格式分析。
2. <http://guoh.org/lifelog/2013/08/format-of-container-file-and-iso-iec-14496-12/> 媒体文件格式以及ISO/IEC 14496-12规范
   >简单介绍ISOBMFF文件结构。
