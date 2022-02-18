# AAC音频编码

## 简介

高级音频编码（英语：Advanced Audio Coding，AAC），出现于1997年，为一种基于MPEG-2的有损数字音频压缩的专利音频编码标准，最早定义在MPEG-2标准（ISO/IEC 13818-7）中，由Fraunhofer IIS、杜比实验室、AT&T、Sony、Nokia等公司共同开发。2000年，MPEG-4标准在原本的基础上加上了PNS（Perceptual Noise Substitution）等技术，并提供了多种扩展工具。为了区别于传统的MPEG-2 AAC又称为MPEG-4 AAC。AC标准是作为MP3的继承者而设计出来的，相同的比特率之下，AAC比MP3有更好的音质。

为了适应不同的应用场景，AAC定义9种Profile：

1. MPEG-2 AAC LC 低复杂度规格（Low Complexity）--比较简单，没有增益控制，但提高了编码效率，在中等码率的编码效率以及音质方面，都能找到平衡点
2. MPEG-2 AAC Main 主规格
3. MPEG-2 AAC SSR 可变采样率规格（Scaleable Sample Rate）
4. MPEG-4 AAC LC低复杂度规格（Low Complexity）------现在的手机比较常见的MP4文件中的音频部份就包括了该规格音频文件
5. MPEG-4 AAC Main 主规格 ------包含了除增益控制之外的全部功能，其音质最好
6. MPEG-4 AAC SSR 可变采样率规格（Scaleable Sample Rate）
7. MPEG-4 AAC LTP 长时期预测规格（Long Term Predicition）
8. MPEG-4 AAC LD 低延迟规格（Low Delay）
9. MPEG-4 AAC HE 高效率规格（High Efficiency）-----这种规格适合用于低码率编码，有Nero ACC 编码器支持

目前使用最多的是LC和HE。其中LC-AAC用于中高码率(>=80Kbps)，HE-AAC(LC + SBR技术)主要用于中低码(<=80Kbps)，新推出的HE-AACv2(LC+SBR+PS)主要用于低码率(<=48Kbps），事实上大部分编码器设成<=48Kbps自动启用PS技术，而>48Kbps就不加PS。流行的Nero AAC编码程序只支持LC，HE，HEv2这三种规格，编码后的AAC音频，规格显示都是LC。

## AAC

## AAC流

AAC码流打包格式：

|用途|格式|初始定义文档|最新定义文档|描述|
|-|-|-|-|-|
|Multiplex|FlexMux|ISO/IEC 14496-1:2001 (MPEG4 Systems)|-|Flexible multiplex scheme|
|Multiplex|LATM|ISO/IEC 14496-3:2001(MPEG-4 Audio)|-|Low Overhead Audio Transport Multiplex|
|Storage|ADIF|ISO/IEC 13818-7:1997(MPEG-2 Audio)|ISO/IEC 14496-3:2001(MPEG-4 Audio)|(MPEG-2 AAC) AudioData InterchangeFormat, AAC only|
|Storage|MP4FF|ISO/IEC 14496-1:2001(MPEG-4 Systems)|-|MPEG-4 File format|
|Transmission|ADTS|ISO/IEC 13818-7:1997(MPEG-2 Audio)|ISO/IEC 14496-3:2001(MPEG-4 Audio)|Audio Data Transport Stream, AAC only|
|Transmission|LOAS|ISO/IEC 14496-3:2001(MPEG-4 Audio)|-|Low Overhead Audio Stream, based on LATM, three versions are available:AudioSyncStream() EPAudioSyncStream() AudioPointerStream()|

Multiplex

这里我们只进一步分析多媒体流传输应用较多的：[ADTS](aac_adts.md)和[LOAS](aac_loas.md)。

### 备注

#### flv/rtmp中AAC的负载方式

#### 有两种方法可以将 AAC 放入MPEG-TS流中

1. 使用 ADTS 语法（MPEG2 样式）。
在这种情况下，PMT 的音频stream_type=0x0F（ISO/IEC 13818-7 音频（具有 ADTS 传输语法）。这样只能使用MPEG2-AAC版本，无法支持SBR和PS。
2. 使用 LATM+LOAS/AudioSyncStream 语法（MPEG4 样式）。
在这种情况下，PMT 的音频stream_type=0x11（ISO/IEC 14496-3 音频，使用 LATM 传输语法）。使用MPEG4-AAC 功能的所有特性，包括 SBR 和 PS。

> [DVB (Digital Video Broadcasting)](https://dvb.org/)标准ETSI TS 101 154 要求：HE/HEv2 AAC 应使用[LATM+LOAS](aac_loas.md)格式传输。
> 🎈<https://stackoverflow.com/questions/49006675/incorporate-hev2-aac-into-an-mpeg-ts-for-hls-content>

## 参考资料

- <https://www.iana.org/assignments/media-types/audio/aac> IANA注册格式：AAC
- <https://juejin.cn/post/6844904083900350478> PCM转AAC
