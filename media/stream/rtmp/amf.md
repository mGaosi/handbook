# Action Message Format

## 简介

AMF是一种二进制格式，用于序列化对象。 该格式通常与Adobe的RTMP结合使用，以建立连接和控制命令以交付流媒体。

AMF目前有两种版本，AMF0和AMF3，他们在数据类型的定义上有细微不同。AMF的官方文档参见[AMF0](./resource/amf0.pdf)和[AMF3](./resource/afm3.pdf)。

## 对象结构

__AMF数据的二进制结构：__

```txt
AMF0:

 0               
 0 1 2 3 4 5 6 7 ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| TYPE          |   VALUE   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+

AFM3

 0               1
 0 1 2 3 4 5 6 7 0 1 2 3 4 5 6 7 ...
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
| 0x11          |  TYPE         |   VALUE   |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

第一个字节固定为AMF对象类型，接着是对象值。如果TYPE==0x11，表示第二个字节也是是AMF对象类型，然后再是对象值。

__AMF0数据类型定义:__

|Type|Byte code|Value|
|-|-|-|
|Number|0×00|double类型，8字节长度|
|Boolean|0×01|bool类型，1个字节，0→false，1→true|
|String|0×02|前两个字节表示字符串长度，接着是字符串数据|
|Object|0×03|键值对，键使用String表示，值使用AMF对象，以 0x00 0x00 0x09结尾|
|MovieClip|0×04||
|Null|0×05|空|
|Undefined|0×06|空|
|Reference|0×07||
|MixedArray|0×08||
|EndOfObject|0×09|通常都是跟在Object数据最后|
|Array|0x0a||
|Date|0x0b||
|LongString|0x0c||
|Unsupported|0x0d||
|Recordset|0x0e||
|XML|0x0f||
|TypedObject (Class instance)|0×10||
|AMF3 data|0×11|Only Flash player 9+|

```c
enum AMF0Type
{
    Number           = 0x00,
    Boolean          = 0x01,
    String           = 0x02,
    Object           = 0x03,
    MovieClip        = 0x04,
    Null             = 0x05,
    Undefined        = 0x06,
    ReferencedObject = 0x07,
    MixedArray       = 0x08,
    EndOfObject      = 0x09,
    Array            = 0x0a,
    Date             = 0x0b,
    LongString       = 0x0c,
    TypeAsObject     = 0x0d,
    Recordset        = 0x0e,
    Xml              = 0x0f,
    TypedObject      = 0x10,
    AMF3data         = 0x11,
}
```

__AMF3数据类型定义:__

|Type|Byte code|Value|
|-|-|-|
|Undefined|0×00|空|
|Null|0×01|空|
|False|0×02||
|True|0×03||
|Integer|0×04||
|Number|0×05||
|String|0×06||
|Date|0×08||
|Array|0×09||
|Object|0×0a||
|ByteArray|0x0c||
|Dictionary|0×11||

```c
enum AMF3Type
{
    Undefined        = 0x00,
    Null             = 0x01,
    False            = 0x02,
    True             = 0x03,
    Integer          = 0x04,
    Number           = 0x05,
    String           = 0x06,
    Date             = 0x08,
    Array            = 0x09,
    Object           = 0x0a,
    ByteArray        = 0x0c,
    Dictionary       = 0x11,
}
```

## 参考

1. <https://developer.aliyun.com/article/344904> AMF目前有两种版本，AMF0和AMF3
2. <https://en.wikipedia.org/wiki/Action_Message_Format> Action Message Format
