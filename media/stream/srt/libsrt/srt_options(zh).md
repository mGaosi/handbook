# SRT Socket Options：LibSRT支持的选项

在libsrt接口中有一种在套接字上设置选项的通用方法，跟系统“setsockopt/getsockopt”函数声明和使用方法类似。

>**NOTE**: 文档原文在这里✈<https://github.com/Haivision/srt/blob/master/docs/APISocketOptions.md>

1. [Types used in socket options](#types-used-in-socket-options)
   - [List of options](#list-of-options)
   - [Option Descriptions](#option-descriptions)
2. [Enumeration types used in options](#enumeration-types-used-in-options)
   - [`SRT_TRANSTYPE`](#1-srt_transtype)
   - [`SRT_KM_STATE`](#2-srt_km_state)
3. [Getting and setting options](#getting-and-setting-options)

## Types used in socket options：选项中使用的数据类型

当前使用的选项值类型如下：

- `int32_t` - 此类型通常可以被视为`int`等效类型，因为它在64位系统上不会更改大小。为清楚起见，选项使用此固定32-bit的整数。在某些情况下，枚举类型也转换到该类型（见下文）。
- `int64_t` - 部分选项使用64-bit的整数。
- `bool` - 当设置选项时，可以用`int`替代。但是获取时应当使用`bool`。
- `string` - 字符串，设置一个选项时，传递字符数组指针作为值，字符串长度作为数据长度。当获取选项的值时，传递一个足够大的数组（在size变量中指定）。
- `linger` - Linger structure. Used exclusively with `SRTO_LINGER`.

## Enumeration types used in options：选项中使用的枚举类型

### 1.SRT_TRANSTYPE

支持设置的值:

- `SRTT_LIVE`: 直播模式，尽可能可靠的数据交付。
- `SRTT_FILE`: 文件模式，可靠的数据交付。

### 2.SRT_KM_STATE

The defined encryption state as performed by the Key Material Exchange, used
by `SRTO_RCVKMSTATE`, `SRTO_SNDKMSTATE` and `SRTO_KMSTATE` options:

- `SRT_KM_S_UNSECURED`: no encryption/decryption. If this state is only on
the receiver, received encrypted packets will be dropped.

- `SRT_KM_S_SECURING`: pending security (HSv4 only). This is a temporary state
used only if the connection uses HSv4 and the Key Material Exchange is
not finished yet. On HSv5 this is not possible because the Key Material
Exchange for the initial key is done in the handshake.

- `SRT_KM_S_SECURED`: KM exchange was successful and the data will be sent
encrypted and will be decrypted by the receiver. This state is only possible on
both sides in both directions simultaneously.

- `SRT_KM_S_NOSECRET`: If this state is in the sending direction  (`SRTO_SNDKMSTATE`),then it means that the sending party has set a passphrase, but the peer did not.In this case the sending party can receive unencrypted packets from the peer, but packets it sends to the peer will be encrypted and the peer will not be able to decrypt them. This state is only possible in HSv5.

- `SRT_KM_S_BADSECRET`: The password is wrong (set differently on each party);
encrypted payloads won't be decrypted in either direction.

Note that with the default value of `SRTO_ENFORCEDENCRYPTION` option (true),
the state is equal on both sides in both directions, and it can be only
`SRT_KM_S_UNSECURED` or `SRT_KM_S_SECURED` (in other cases the connection
is rejected). Otherwise it may happen that either both sides have different
passwords and the state is `SRT_KM_S_BADSECRET` in both directions, or only
one party has set a password, in which case the KM state is as follows:

|                          | `SRTO_RCVKMSTATE`    | `SRTO_SNDKMSTATE`    |
|--------------------------|----------------------|----------------------|
| Party with no password:  | `SRT_KM_S_NOSECRET`  | `SRT_KM_S_UNSECURED` |
| Party with password:     | `SRT_KM_S_UNSECURED` | `SRT_KM_S_NOSECRET`  |

## Getting and setting options：获取和设置选项

旧版接口:

```cpp
int srt_getsockopt(SRTSOCKET socket,
                   int level,
                   SRT_SOCKOPT optName,
                   void* optval,
                   int& optlen);
int srt_setsockopt(SRTSOCKET socket,
                   int level,
                   SRT_SOCKOPT optName,
                   const void* optval,
                   int optlen);
```

新的接口:

```cpp
int srt_getsockflag(SRTSOCKET socket,
                    SRT_SOCKOPT optName,
                    void* optval,
                    int& optlen);
int srt_setsockflag(SRTSOCKET socket,
                    SRT_SOCKOPT optName,
                    const void* optval,
                    int optlen);
```

在旧版接口中，还有一个未使用的`level`参数。它只是在最初的UDT API中为了模拟系统的`setsockopt`函数，实际上内部实现忽略了这个参数。

某些选项需要类型为`bool`的值，而其他选项需要类型为`integer`的值，它们的字节大小可能不同，如果弄错了，最终可能会出现未知的程序行为。为了方便，设置选项函数可以同时接受`int32_t`和`bool`类型，但在获取选项值的情况下则不是这样。

旧版接口中几乎所有的选项都是派生UDT库中的（有一些已删除，包括一些已在UDT中弃用的）。后期增加了许多新的SRT选项。现在这些选项名称前缀都调整为`SRTO_`。旧名称在`udt.h`中通过别名提供：

- `UDT_`前缀开头的选项对应UDT选项，现在它们的前缀改为`SRTO_`。
- `UDP_`前缀开头的选项对应UDP选项，现在它们的前缀改为`SRTO_UDP_`。
- `SRT_`前缀开头的SRT选项，现在它们的前缀改为`SRTO_`。

在下面的选项表中列举了SRT的选项，表中列出的各个特征如下：

**Since:** 表示首次引入此选项时的SRT版本。如果这个字段为空，则它是从UDT派生的选项。“版本0.0.0”是最旧的版本。

**Binding:** 设置选项使用的限制，如果该选项是只读的那么这个字段为空，正确设置的时机：

- `pre:`调用`srt_connect()`或`srt_bind()`之前设置完毕，并且此后不能修改。侦听套接字应设置`listening`，并且由`srt_accept()`返回的每个套接字会派生设置的选项。
- `post:`可以随时设置更改此选项值，包括在套接字已连接（以及在可接受的套接字）。在监听时设置此标志套接字仅对该套接字本身有效。请注意，在连接之前设置了一些后置绑定选项，这些选项具有重要意义。

**Type:** 值的数据类型（参见上文）。

**Units:** 如果值表示长度或时间，这里只是给出数据单位。它还表示当类型为integer时其表示的真实类型：

- `enum:`可能的值在枚举类型中定义。
- `flags:`位标志的集合。

**Default:** 该选项的默认值。更复杂的默认值将在下文中选项说明章节的描述。对于不能设置的选项，此字段为空。

**Range:** 如果选项值是整数类型，并且有一个有效的范围，范围可以指定为：

- `X-...`仅指定最小值。
- `X-Y,Z`允许值介于X和Y之间，另外还有Z也是有效值。
- 如果值是`string`类型，则此字段将在方括号内说明其最大长度。
- 如果该范围中还附加了一个星号，则意味着对该值应用更详细的限制还需参考下文中选项说明章节的描述。

**Dir:** W 表示只能写入，R 表示只能读取，RW 表示可以读写。

**Entity:** 这表示是否可以在套接字或组上设置该选项。G和S选项可能同时出现，在这种情况下，两种可能性都适用。D和I选项互斥，但总是与G一起出现。+标记只能与GS共存。可能的规格有：

- S：此选项可在单个套接字上设置（如果不是GS，则只能在单个套接字上设置）。
- G：此选项可以在组上设置（如果不是GS，则只能在组中设置）。
- D：选项将由成员套接字派生。
- I: 选项将由该组独占地获取和管理。
- +：也允许通过`SRT_SOCKGROUPCONFIG`中由`SRT_create_config`准备的配置对象在组成员套接字上单独设置此选项。请注意，此设置可能会被组设置覆盖。

## List of options：选项表

下表按选项名字母顺序列出了SRT套接字选项。选项详情如下:

| Option Name                                            | Since | Binding | Type      | Units   | Default    | Range    | Dir | Entity |
| ------------------------------------------------------ | ----- | ------- | --------- | ------- | ---------- | -------- | --- | ------ |
| [`SRTO_BINDTODEVICE`](#SRTO_BINDTODEVICE)              | 1.4.2 | pre     | `string`  |         |            |          | RW  | GSD+   |
| [`SRTO_CONGESTION`](#SRTO_CONGESTION)                  | 1.3.0 | pre     | `string`  |         | "live"     | *        | W   | S      |
| [`SRTO_CONNTIMEO`](#SRTO_CONNTIMEO)                    | 1.1.2 | pre     | `int32_t` | msec    | 3000       | 0..      | W   | GSD+   |
| [`SRTO_DRIFTTRACER`](#SRTO_DRIFTTRACER)                | 1.4.2 | post    | `bool`    |         | true       |          | RW  | GSD    |
| [`SRTO_ENFORCEDENCRYPTION`](#SRTO_ENFORCEDENCRYPTION)  | 1.3.2 | pre     | `bool`    |         | true       |          | W   | GSD    |
| [`SRTO_EVENT`](#SRTO_EVENT)                            |       |         | `int32_t` | flags   |            |          | R   | S      |
| [`SRTO_FC`](#SRTO_FC)                                  |       | pre     | `int32_t` | pkts    | 25600      | 32..     | RW  | GSD    |
| [`SRTO_GROUPCONNECT`](#SRTO_GROUPCONNECT)              | 1.5.0 | pre     | `int32_t` |         | 0          | 0...1    | W   | S      |
| [`SRTO_GROUPSTABTIMEO`](#SRTO_GROUPSTABTIMEO)          | 1.5.0 | pre     | `int32_t` | ms      | 80         | 10-...   | W   | GSD+   |
| [`SRTO_GROUPTYPE`](#SRTO_GROUPTYPE)                    | 1.5.0 | pre     | `int32_t` | enum    |            |          | R   | S      |
| [`SRTO_INPUTBW`](#SRTO_INPUTBW)                        | 1.0.5 | post    | `int64_t` | B/s     | 0          | 0..      | RW  | GSD    |
| [`SRTO_IPTOS`](#SRTO_IPTOS)                            | 1.0.5 | pre     | `int32_t` |         | (system)   | 0..255   | RW  | GSD    |
| [`SRTO_IPTTL`](#SRTO_IPTTL)                            | 1.0.5 | pre     | `int32_t` | hops    | (system)   | 1..255   | RW  | GSD    |
| [`SRTO_IPV6ONLY`](#SRTO_IPV6ONLY)                      | 1.4.0 | pre     | `int32_t` |         | (system)   | -1..1    | RW  | GSD    |
| [`SRTO_ISN`](#SRTO_ISN)                                | 1.3.0 |         | `int32_t` |         |            |          | R   | S      |
| [`SRTO_KMPREANNOUNCE`](#SRTO_KMPREANNOUNCE)            | 1.3.2 | pre     | `int32_t` | pkts    | 0x1000     | 0.. *    | RW  | GSD    |
| [`SRTO_KMREFRESHRATE`](#SRTO_KMREFRESHRATE)            | 1.3.2 | pre     | `int32_t` | pkts    | 0x1000000  | 0..      | RW  | GSD    |
| [`SRTO_KMSTATE`](#SRTO_KMSTATE)                        | 1.0.2 |         | `int32_t` | enum    |            |          | R   | S      |
| [`SRTO_LATENCY`](#SRTO_LATENCY)                        | 1.0.2 | pre     | `int32_t` | ms      | 120 *      | 0..      | RW  | GSD    |
| [`SRTO_LINGER`](#SRTO_LINGER)                          |       | pre     | `linger`  | s       | on, 180    | 0..      | RW  | GSD    |
| [`SRTO_LOSSMAXTTL`](#SRTO_LOSSMAXTTL)                  | 1.2.0 | pre     | `int32_t` | packets | 0          | 0..      | RW  | GSD+   |
| [`SRTO_MAXBW`](#SRTO_MAXBW)                            | 1.0.5 | pre     | `int64_t` | B/s     | -1         | -1..     | RW  | GSD    |
| [`SRTO_MESSAGEAPI`](#SRTO_MESSAGEAPI)                  | 1.3.0 | pre     | `bool`    |         | true       |          | W   | GSD    |
| [`SRTO_MINVERSION`](#SRTO_MINVERSION)                  | 1.3.0 | pre     | `int32_t` | version | 0          | *        | W   | GSD    |
| [`SRTO_MSS`](#SRTO_MSS)                                |       | pre     | `int32_t` | bytes   | 1500       | 76..     | RW  | GSD    |
| [`SRTO_NAKREPORT`](#SRTO_NAKREPORT)                    | 1.1.0 | pre     | `bool`    |         |  *         |          | RW  | GSD+   |
| [`SRTO_OHEADBW`](#SRTO_OHEADBW)                        | 1.0.5 | post    | `int32_t` | %       | 25         | 5..100   | RW  | GSD    |
| [`SRTO_PACKETFILTER`](#SRTO_PACKETFILTER)              | 1.4.0 | pre     | `string`  |         | ""         | [512]    | W   | GSD    |
| [`SRTO_PASSPHRASE`](#SRTO_PASSPHRASE)                  | 0.0.0 | pre     | `string`  |         | ""         | [10..79] | W   | GSD    |
| [`SRTO_PAYLOADSIZE`](#SRTO_PAYLOADSIZE)                | 1.3.0 | pre     | `int32_t` | bytes   | \*         | *        | W   | GSD    |
| [`SRTO_PBKEYLEN`](#SRTO_PBKEYLEN)                      | 0.0.0 | pre     | `int32_t` | bytes   | 0          | *        | RW  | GSD    |
| [`SRTO_PEERIDLETIMEO`](#SRTO_PEERIDLETIMEO)            | 1.3.3 | pre     | `int32_t` | ms      | 5000       | 0..      | RW  | GSD+   |
| [`SRTO_PEERLATENCY`](#SRTO_PEERLATENCY)                | 1.3.0 | pre     | `int32_t` | ms      | 0          | 0..      | RW  | GSD    |
| [`SRTO_PEERVERSION`](#SRTO_PEERVERSION)                | 1.1.0 |         | `int32_t` | *       |            |          | R   | GS     |
| [`SRTO_RCVBUF`](#SRTO_RCVBUF)                          |       | pre     | `int32_t` | bytes   | 8192 bufs  | *        | RW  | GSD+   |
| [`SRTO_RCVDATA`](#SRTO_RCVDATA)                        |       |         | `int32_t` | pkts    |            |          | R   | S      |
| [`SRTO_RCVKMSTATE`](#SRTO_RCVKMSTATE)                  | 1.2.0 |         | `int32_t` | enum    |            |          | R   | S      |
| [`SRTO_RCVLATENCY`](#SRTO_RCVLATENCY)                  | 1.3.0 | pre     | `int32_t` | msec    | *          | 0..      | RW  | GSD    |
| [`SRTO_RCVSYN`](#SRTO_RCVSYN)                          |       | post    | `bool`    |         | true       |          | RW  | GSI    |
| [`SRTO_RCVTIMEO`](#SRTO_RCVTIMEO)                      |       | post    | `int32_t` | ms      | -1         | -1, 0..  | RW  | GSI    |
| [`SRTO_RENDEZVOUS`](#SRTO_RENDEZVOUS)                  |       | pre     | `bool`    |         | false      |          | RW  | S      |
| [`SRTO_RETRANSMITALGO`](#SRTO_RETRANSMITALGO)          | 1.4.2 | pre     | `int32_t` |         | 0          | [0, 1]   | W   | GSD    |
| [`SRTO_REUSEADDR`](#SRTO_REUSEADDR)                    |       | pre     | `bool`    |         | true       |          | RW  | GSD    |
| [`SRTO_SENDER`](#SRTO_SENDER)                          | 1.0.4 | pre     | `bool`    |         | false      |          | W   | S      |
| [`SRTO_SNDBUF`](#SRTO_SNDBUF)                          |       | pre     | `int32_t` | bytes   | 8192 bufs  | *        | RW  | GSD+   |
| [`SRTO_SNDDATA`](#SRTO_SNDDATA)                        |       |         | `int32_t` | pkts    |            |          | R   | S      |
| [`SRTO_SNDDROPDELAY`](#SRTO_SNDDROPDELAY)              | 1.3.2 | pre     | `int32_t` | ms      | *          | -1..     | W   | GSD+   |
| [`SRTO_SNDKMSTATE`](#SRTO_SNDKMSTATE)                  | 1.2.0 | post    | `int32_t` | enum    |            |          | R   | S      |
| [`SRTO_SNDSYN`](#SRTO_SNDSYN)                          |       | post    | `bool`    |         | true       |          | RW  | GSI    |
| [`SRTO_SNDTIMEO`](#SRTO_SNDTIMEO)                      |       | post    | `int32_t` | ms      | -1         | -1..     | RW  | GSI    |
| [`SRTO_STATE`](#SRTO_STATE)                            |       |         | `int32_t` | enum    |            |          | R   | S      |
| [`SRTO_STREAMID`](#SRTO_STREAMID)                      | 1.3.0 | pre     | `string`  |         | ""         | [512]    | RW  | GSD    |
| [`SRTO_TLPKTDROP`](#SRTO_TLPKTDROP)                    | 1.0.6 | pre     | `bool`    |         | *          |          | RW  | GSD    |
| [`SRTO_TRANSTYPE`](#SRTO_TRANSTYPE)                    | 1.3.0 | pre     | `int32_t` | enum    |`SRTT_LIVE` | *        | W   | S      |
| [`SRTO_TSBPDMODE`](#SRTO_TSBPDMODE)                    | 0.0.0 | pre     | `bool`    |         | *          |          | W   | S      |
| [`SRTO_UDP_RCVBUF`](#SRTO_UDP_RCVBUF)                  |       | pre     | `int32_t` | bytes   | 8192 bufs  | *        | RW  | GSD+   |
| [`SRTO_UDP_SNDBUF`](#SRTO_UDP_SNDBUF)                  |       | pre     | `int32_t` | bytes   | 65536      | *        | RW  | GSD+   |
| [`SRTO_VERSION`](#SRTO_VERSION)                        | 1.1.0 |         | `int32_t` |         |            |          | R   | S      |

### Option Descriptions：选项详细描述

#### SRTO_BINDTODEVICE

|OptName|Since|Binding|Type|Units|Default|Range|Dir|Entity|
|-|-|-|-|-|-|-|-|-|
|`SRTO_BINDTODEVICE`|1.4.2|pre|`string`||||RW|GSD+|

- Refers to the `SO_BINDTODEVICE` system socket option for `SOL_SOCKET` level.
This effectively limits the packets received by this socket to only those
that are targeted to that device. The device is specified by name passed as
string. The setting becomes effective after binding the socket (including
default-binding when connecting).

- NOTE: This option is only available on Linux and available there by default.
On all other platforms setting this option will always fail.

- NOTE: With the default system configuration, this option is only available
for a process that runs as root. Otherwise the function that applies the setting
(`srt_bind`, `srt_connect` etc.) will fail.

#### SRTO_CONGESTION

| OptName           | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_CONGESTION` | 1.3.0 | pre     | `string`   |         | "live"    | *      | W   | S      |

- The type of congestion controller used for the transmission for that socket.

- Its type must be exactly the same on both connecting parties, otherwise the connection is rejected - **however** you may also change the value of this option for the accepted socket in the listener callback (see `srt_listen_callback`) if an appropriate instruction was given in the Stream ID.

- Currently supported congestion controllers are designated as "live" and "file"

- Note that it is not recommended to change this option manually, but you shouldrather change the whole set of options using the [`SRTO_TRANSTYPE`](#SRTO_TRANSTYPE) option.

- 用于该套接字的传输的拥塞控制器的类型。
- 它的类型在两个连接方上必须完全相同，否则连接将被拒绝。但是对应侦听套接字，可以在回调中为接受的套接字更改此选项的值（请参见`srt_listen_callback`）。
- 当前支持的拥塞控制器:`"live"` `"file"`。
- 注意，不建议手动更改此选项，但您应该使用[`SRTO_TRANSTYPE`](#SRTO_TRANSTYPE)选项更改整个选项集。

#### SRTO_CONNTIMEO

| OptName            | Since | Binding |   Type    | Units  | Default  | Range  | Dir | Entity |
| ------------------ | ----- | ------- | --------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_CONNTIMEO`   | 1.1.2 | pre     | `int32_t` | msec   | 3000     | 0..    | W   | GSD+   |

- 连接超时时长，单位毫秒。此选项适用于连接发起者和集合连接模式。对于对等连接模式（请参见[SRTO_RENDEZVOUS](#SRTO_RENDEZVOUS)），有效连接超时是使用`SRTO_CONNTIMEO`值的10倍。

#### SRTO_DRIFTTRACER

| OptName           | Since | Binding | Type      | Units  | Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | --------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_DRIFTTRACER`| 1.4.2 | post    | `bool`    |        | true     |        | RW  | GSD    |

- 打开或者关闭时间漂移跟踪器。

#### SRTO_ENFORCEDENCRYPTION

| OptName                    | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| -------------------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_ENFORCEDENCRYPTION`  | 1.3.2 | pre     | `bool`     |         | true      |        | W   | GSD    |

- This option enforces that both connection parties have the same passphrase set, or both do not set the passphrase, otherwise the connection is rejected.

- When this option is set to FALSE **on both connection parties**, the connection is allowed even if the passphrase differs on both parties, or it was set only on one party. Note that the party that has set a passphrase is still allowed to send data over the network. However, the receiver will not be able to decrypt that data and will not deliver it to the application. The party that has set no passphrase can send (unencrypted) data that will be successfully received by its peer.

- This option can be used in some specific situations when the user knows both parties of the connection, so there's no possible situation of a rogue sender and can be useful in situations where it is important to know whether a connection is possible. The inability to decrypt an incoming transmission can be then reported as a different kind of problem.

>**IMPORTANT**: There is unusual and unobvious behavior when this flag is TRUE on the caller and FALSE on the listener, and the passphrase was mismatched. On the listener side the connection will be established and broken right after, resulting in a short-lived "spurious" connection report on the listener socket. This way, a socket will be available for retrieval from an `srt_accept` call for a very short time, after which it will be removed from the listener backlog just as if no connection attempt was made at all. If the application is fast enough to react on an incoming connection, it will retrieve it, only to learn that it is already broken. This also makes possible a scenario where `SRT_EPOLL_IN` is reported on a listener socket, but then an `srt_accept` call reports an `SRT_EASYNCRCV` error. How fast the connection gets broken depends on the network parameters -- in particular, whether the `UMSG_SHUTDOWN` message sent by the caller is delivered (which takes one RTT in this case) or missed during the interval from its creation up to the connection timeout (default = 5seconds). It is therefore strongly recommended that you only set this flag to FALSE on the listener when you are able to ensure that it is also set to FALSE on the caller side.

#### SRTO_EVENT

| OptName           | Since | Binding | Type      | Units  | Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | --------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_EVENT`      |       |         | `int32_t` | flags  |          |        | R   | S      |

- 只用于获取套接字上当前活动事件设置的位标志。
- 可能的值是在`SRT_EPOLL_OPT`枚举类型中定义的值（由`SRT_EPOLL_in`、`SRT_EPOLL_OUT`和`SRT_EPOLL_ERR`的组合）。

#### SRTO_FC

| OptName           | Since | Binding | Type      | Units  | Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | --------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_FC`         |       | pre     | `int32_t` | pkts   | 25600    | 32..   | RW  | GSD    |

- 飞行数据报数量，即未经确认的可发送的最大字节数。

>飞行数据指的是不用确认就可以发送的数据，也可以理解为在发送链路中的数据。

#### SRTO_GROUPCONNECT

| OptName              | Since | Binding | Type      | Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | --------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_GROUPCONNECT`  | 1.5.0 | pre     | `int32_t` |        | 0        | 0...1  | W   | S      |

- 当侦听器套接字上的此标志设置为1时，它允许此套接字接受组连接。当设置为默认值0时，组连接将被拒绝。请记住，如果`SRTO_GROUPCONNECT`标志设置为1（即允许组连接），`srt_accept`可能返回一个套接字或一个组ID。在允许组连接的侦听器套接字上调用`srt_accept`时，必须考虑到这一点。由这个函数的调用者来进行区分，并根据返回的实体类型采取适当的操作。

- 如果在传递给侦听器回调处理的套接字上将此标志设置为1，则表示此套接字是为组连接创建的，它将成为组的成员。注意，在这种情况下，只有组中的第一个连接需要`srt_accept`（后续的连接在内部处理），此函数将返回组，而不是此套接字ID。

#### SRTO_GROUPSTABTIMEO

| OptName               | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| --------------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_GROUPSTABTIMEO` | 1.5.0 | pre     | `int32_t`  | ms     | 80       | 10-... | W   | GSD+   |

- This setting is used for groups of type `SRT_GTYPE_BACKUP`. It defines the stability timeout, which is the maximum interval between two consecutive packets retrieved from the peer on the currently active link. These two packets can be of any type,but this setting usually refers to control packets while the agent is a sender.Idle links exchange only keepalive messages once per second, so they do not count. Note that this option is meaningless on sockets that are not members of the Backup-type group.

- This value should be set with a thoroughly selected balance and correspond to the maximum stretched response time between two consecutive ACK messages. By default ACK messages are sent every 10ms (so this interval is not dependent on the network latency), and so should be the interval between two consecutive received ACK messages. Note, however, that the network jitter on the public internet causes these intervals to be stretched, even to multiples of that interval. Both large and small values of this option have consequences:

- Large values of this option prevent overreaction on highly stretched response times, but introduce a latency penalty - the latency must be greater than this value (otherwise switching to another link won't preserve smooth signal sending). Large values will also contribute to higher packet bursts sent at the moment when an idle link is activated.

- Smaller values of this option respect low latency requirements very well, but may cause overreaction on even slightly stretched response times. This is unwanted,as a link switch should ideally happen only when the currently active link is really broken, as every link switch costs extra overhead (it counts for 100% for a time of one ACK interval).

> Note that the value of this option is not allowed to exceed the value of `SRTO_PEERIDLETIMEO`. Usually it is only meaningful if you change the latter option, as the default value of it is way above any sensible value of `SRTO_GROUPSTABTIMEO`.

#### SRTO_GROUPTYPE

| OptName              | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_GROUPTYPE`     | 1.5.0 | pre     | `int32_t`  | enum   |          |        | R   | S      |

- 此选项是只读的，一般是在Listenser回调处理程序中获取该选项值（请参见`srt_listen_callback`）。可能的值参考`SRT_GROUP_TYPE`枚举类型的定义。
- 在接入的新连接可以获取到该选项有效值。如果接入的新连接不会建立组连接，则返回的值为`SRT_GTYPE_UNDEFINED`。如果在监听器回调处理程序内部以外的任何上下文中读取此选项，则不能确保返回数据是有效的。

#### SRTO_INPUTBW

| OptName          | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| ---------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_INPUTBW`   | 1.0.5 | post    | `int64_t`  | B/s    | 0        | 0..    | RW  | GSD    |

- 当`SRTO_MAXBW`设置为0时，此选项才有效。它根据公式`MAXBW=INPUTBW*(100+OHEADBW)/100`来控制最大带宽。当此选项设置为0（自动）时，实际INPUTBW值将根据传输期间的输入速率（应用程序调用`srt_send*`函数）进行估计。

> 建议：在直播流的模式下可以将此选项设置为流的预期比特率，并将`SRTO_OHEADBW`的默认值保留为(`SRTO_INPUTBW` * 25%)。

#### SRTO_IPTOS

| OptName          | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| ---------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_IPTOS`     | 1.0.5 | pre     | `int32_t`  |        | (system) | 0..255 | RW  | GSD    |

- 设置网络包优先级。如果是IPv4网络，则设置TOS（请参见`IP_TOS`选项）。如果是IPv6网络，则设置Traffic Class（请参阅`IPv6_TCLASS`）。仅适用于发送方。
- 当读取该选项值时，返回的是未连接套接字的用户预置值或已连接套接字的实际值。
- 用户可配置，默认值：0xB8。

#### SRTO_IPTTL

| OptName          | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| ---------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_IPTTL`     | 1.0.5 | pre     | `int32_t`  | hops   | (system) | 1..255 | RW  | GSD    |

- IPv4生存时间（请参见IP的`IP_TTL`选项）或IPv6单播跃点（请参阅IPv6的`IPV6_UNICAST_HOPS`单播跃点），具体取决于套接字地址系列。仅适用于发送方。
- 当读取该选项值时，返回的是未连接套接字的用户预置值或已连接套接字的实际值。
- 发送方：用户可配置，默认值：64

#### SRTO_IPV6ONLY

| OptName          | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| ---------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_IPV6ONLY`  | 1.4.0 | pre     | `int32_t`  |        | (system) | -1..1  | RW  | GSD    |

- 设置系统套接字标志`IPV6_V6ONLY`。当设置为0时，侦听套接字绑定IPv6地址也接受IPv4连接（它们的地址是IPv4映射的IPv6地址）。默认情况下（-1）不设置此选项，使用平台默认值。

#### SRTO_ISN

| OptName          | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| ---------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_ISN`       | 1.3.0 |         | `int32_t`  |        |          |        | R   | S      |

- ISN（Initial Sequence Number:初始序列号）的值，它是发送的第一个UDP数据包上的序列号，这些UDP数据包携带SRT数据负载。
- 这个值对于一些更复杂的流控制方法的开发人员很有用，可能一次有多个SRT套接字。一般用不到该选项。

#### SRTO_KMPREANNOUNCE

| OptName               | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| --------------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_KMPREANNOUNCE`  | 1.3.2 | pre     | `int32_t`  | pkts   | 0x1000   | 0.. *  | RW  | GSD    |

- The interval (defined in packets) between when a new Stream Encrypting Key
(SEK) is sent and when switchover occurs. This value also applies to the
subsequent interval between when switchover occurs and when the old SEK is
decommissioned.

At `SRTO_KMPREANNOUNCE` packets before switchover the new key is sent
(repeatedly, if necessary, until it is confirmed by the receiver).

At the switchover point (see `SRTO_KMREFRESHRATE`), the sender starts
encrypting and sending packets using the new key. The old key persists in case
it is needed to decrypt packets that were in the flight window, or
retransmitted packets.

The old key is decommissioned at `SRTO_KMPREANNOUNCE` packets after switchover.

The allowed range for this value is between 1 and half of the current value of
`SRTO_KMREFRESHRATE`. The minimum value should never be less than the flight
window (i.e. the number of packets that have already left the sender but have
not yet arrived at the receiver).

#### SRTO_KMREFRESHRATE

| OptName               | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| --------------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_KMREFRESHRATE`  | 1.3.2 | pre     | `int32_t`  | pkts   | 0x1000000| 0..    | RW  | GSD    |

- The number of packets to be transmitted after which the Stream Encryption Key
(SEK), used to encrypt packets, will be switched to the new one. Note that
the old and new keys live in parallel for a certain period of time (see
`SRTO_KMPREANNOUNCE`) before and after the switchover.

- Having a preannounce period before switchover ensures the new SEK is installed
at the receiver before the first packet encrypted with the new SEK is received.
The old key remains active after switchover in order to decrypt packets that
might still be in flight, or packets that have to be retransmitted.


[Return to list](#list-of-options)



#### SRTO_KMSTATE

| OptName               | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| --------------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_KMSTATE`        | 1.0.2 |         | `int32_t`  | enum   |          |        | R   | S      |

- Keying Material state. This is a legacy option that is equivalent to
`SRTO_SNDKMSTATE`, if the socket has set `SRTO_SENDER` to true, and 
`SRTO_RCVKMSTATE` otherwise. This option is then equal to `SRTO_RCVKMSTATE`
always if your application disregards possible cooperation with a peer older
than 1.3.0, but then with the default value of `SRTO_ENFORCEDENCRYPTION` the
value returned by both options is always the same. See [`SRT_KM_STATE`](#2-srt_km_state)
for more details.

#### SRTO_LATENCY

| OptName               | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| --------------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_LATENCY`        | 1.0.2 | pre     | `int32_t`  | ms     | 120 *    | 0..    | RW  | GSD    |

- This flag sets both `SRTO_RCVLATENCY` and `SRTO_PEERLATENCY` to the same value. Note that prior to version 1.3.0 this is the only flag to set the latency. However this is effectively equivalent to setting `SRTO_PEERLATENCY`, when the side is sender (see `SRTO_SENDER`), and `SRTO_RCVLATENCY` when the side is receiver. Bidirectional stream sending in version 1.2.0 was not supported.
- 此标志将`SRTO_RCVLATENCY`和`SRTO_PEERLATENCY`设置为相同的值。在版本1.3.0之前，这是设置延迟的唯一标志。然而，这实际上相当于在一方为发送方时设置`SRTO_PEERLATENCY`（请参见`SRTO_SENDER`和`SRTO_RCVLATENCY`）。版本1.2.0中不支持双向流发送。

#### SRTO_LINGER

| OptName              | Since | Binding | Type       | Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------ | -------- | ------ | --- | ------ |
| `SRTO_LINGER`        |       | pre     | `linger`   | s      | on, 180  | 0..    | RW  | GSD    |

- 关闭时的socket保留时间，可以参考[SO_LINGER](http://man7.org/linux/man-pages/man7/socket.7.html)。
- 建议值: off (0)。

#### SRTO_LOSSMAXTTL

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_LOSSMAXTTL`    | 1.2.0 | pre     | `int32_t`  | packets | 0        | 0..    | RW  | GSD+   |

- The value up to which the *Reorder Tolerance* may grow. The *Reorder Tolerance* is the number of packets that must follow the experienced "gap" in sequence numbers of incoming packets so that the loss report is sent (in the hope that the gap is due to packet reordering rather than because of loss). The value of *Reorder Tolerance* starts from 0 and is set to a greater value when packet reordering is detected This happens when a "belated" packet, with sequence number older than the latest received, has been received, but without retransmission flag. When this is detected the *Reorder Tolerance* is set to the value of the interval between latest sequence and this packet's sequence, but not more than the value set by `SRTO_LOSSMAXTTL`.By default this value is set to 0, which means that this mechanism is off.

#### SRTO_MAXBW

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_MAXBW`         | 1.0.5 | pre     | `int64_t`  | B/s     | -1       | -1..   | RW  | GSD    |

- 最大发送带宽：
  - `-1`：无限制（实时模式下的限制为1 Gbps）。
  - `0`：相对于输入速率（请参见[`SRTO_INPUTBW`](#SRTO_INPUTBW)）。
  - `>0`:限制为指定的每秒单位字节数。

>注意，任何模式下这个选项的默认值是-1。对于实时流，通常建议在此处设置值0，并依赖`SRTO_INPUTBW`和`SRTO_OHEADBW`选项。但是，如果您想这样做，您应该确保流具有相当恒定的比特率，或者更改不是突然的，因为高比特率的更改可能会对测量。SRT无法确保实时流始终如此，因此即使在实时模式下，默认值-1也会保持不变。

#### SRTO_MESSAGEAPI

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_MESSAGEAPI`    | 1.3.0 | pre     | `bool`     |         | true     |        | W   | GSD    |

- When set, this socket uses the Message API[\*], otherwise it uses the Stream API. Note that in live mode (see [`SRTO_TRANSTYPE`](#SRTO_TRANSTYPE) option) only the Message API is available. In File mode you can chose to use one of two modes(note that the default for this option is changed with `SRTO_TRANSTYPE`option):

  - **Stream API** (default for file mode): In this mode you may send as many data as you wish with one sending instruction, or even use dedicated functions that operate directly on a file. The internal facility will take care of any speed and congestion control. When receiving, you can also receive as many data as desired. The data not extracted will be waiting for the next call. There is no boundary between data portions in Stream mode.
  
  - **Message API**: In this mode your single sending instruction passes exactly one piece of data that has boundaries (a message). Contrary to Live mode, this message may span multiple UDP packets, and the only size limitation is that it shall fit as a whole in the sending buffer. The receiver shall use as large a buffer as necessary to receive the message, otherwise reassembling and delivering the message might not be possible. When the message is not complete (not all packets received or there was a packet loss) it will not be copied to the application's buffer. Messages that are sent later, but were earlier reassembled by the receiver, will be delivered once ready, if the `inorder` flag was set to false.See [`srt_sendmsg`](https://github.com/Haivision/srt/blob/master/docs/API.md#sending-and-receiving)).
  
- As a comparison to the standard system protocols, the Stream API does transmission similar to TCP, whereas the Message API functions like the SCTP protocol.

#### SRTO_MINVERSION

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_MINVERSION`    | 1.3.0 | pre     | `int32_t`  | version | 0        | *      | W   | GSD    |

- 支持建立连接的最低SRT版本。不满足最低版本要求的握手将被拒绝。

>版本号数据格式参考☞[SRT Handshake](./handshake.md)。

#### SRTO_MSS

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_MSS`           |       | pre     | `int32_t`  | bytes   | 1500     | 76..   | RW  | GSD    |

- 提交给IP层最大分段大小。用于缓冲区分配和速率计算，计算时假定每个数据包都是满的。各方可以独立设置自己的MSS值。在握手过程中，双方交换MSS值，并使用最低值。

> 通常默认的MSS是1500。这是UDP数据包的最大大小，只能减小，除非在一些特殊专用网络中。此大小表示IP数据包的大小，包括了UDP和SRT包头。

#### SRTO_NAKREPORT

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_NAKREPORT`     | 1.1.0 | pre     | `bool`     |         |  *       |        | RW  | GSD+   |

- When set to true, every report for a detected loss will be repeated when the timeout for the expected retransmission of this loss has expired and the missing packet still wasn't recovered, or wasn't conditionally dropped (see[`SRTO_TLPKTDROP`](#SRTO_TLPKTDROP)).

- The default is true for Live mode, and false for File mode (see [`SRTO_TRANSTYPE`](#SRTO_TRANSTYPE)).

#### SRTO_OHEADBW

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_OHEADBW`       | 1.0.5 | post    | `int32_t`  | %       | 25       | 5..100 | RW  | GSD    |

- Recovery bandwidth overhead above input rate (see [`SRTO_INPUTBW`](#SRTO_INPUTBW)), in percentage of the input rate. It is effective only if `SRTO_MAXBW` is set to 0.

- Sender: user configurable, default: 25%.

- Recommendations:
  - *Overhead is intended to give you extra bandwidth for the case when a packet has taken part of the bandwidth, but then was lost and has to be retransmitted. Therefore the effective maximum bandwidth should be appropriately higher than your stream's bitrate so that there's some room for retransmission, but still limited so that the retransmitted packets don't cause the bandwidth usage to skyrocket when larger groups of packets are lost*
  - *Don't configure it too low and avoid 0 in the case when you have the `SRTO_INPUTBW` option set to 0 (automatic). Otherwise your stream will choke and break quickly at any rise in packet loss.*

> ***To do: set-only; get should be supported.***

#### SRTO_PACKETFILTER

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_PACKETFILTER`  | 1.4.0 | pre     | `string`   |         |  ""      | [512]  | W   | GSD    |

- Set up the packet filter. The string must match appropriate syntax for packet
filter setup.

As there can only be one configuration for both parties, it is recommended that 
one party defines the full configuration while the other only defines the matching 
packet filter type (for example, one sets `fec,cols:10,rows:-5,layout:staircase` 
and the other just `fec`). Both parties can also set this option to the same value. 
The packet filter function will attempt to merge configuration definitions, but if 
the options specified are in conflict, the connection will be rejected.

For details, see [Packet Filtering & FEC](packet-filtering-and-fec.md).


[Return to list](#list-of-options)



#### SRTO_PASSPHRASE

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_PASSPHRASE`    | 0.0.0 | pre     | `string`   |         | ""       |[10..79]| W   | GSD    |

- Sets the passphrase for encryption. This enables encryption on this party (or
disables it, if an empty passphrase is passed).

- The passphrase is the shared secret between the sender and the receiver. It is 
used to generate the Key Encrypting Key using [PBKDF2](http://en.wikipedia.org/wiki/PBKDF2) 
(Password-Based Key Derivation Function 2). It is used on the receiver only if 
the received data is encrypted.

- Note that since the introduction of bidirectional support, there's only one 
initial SEK to encrypt the stream (new keys after refreshing will be updated 
independently), and there's no distinction between "service party that defines 
the password" and "client party that is required to set matching password" - both 
parties are equivalent, and in order to have a working encrypted connection, they 
have to simply set the same passphrase. Otherwise the connection is rejected by 
default (see also [`SRTO_ENFORCEDENCRYPTION`](#SRTO_ENFORCEDENCRYPTION)).

#### SRTO_PAYLOADSIZE

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_PAYLOADSIZE`   | 1.3.0 | pre     | `int32_t`  | bytes   | \*        | \*      | W   | GSD    |

- Sets the maximum declared size of a single call to sending function in Live
mode. When set to 0, there's no limit for a single sending call.

- For Live mode: Default value is 1316, but can be increased up to 1456. Note that
with the `SRTO_PACKETFILTER` option additional header space is usually required,
which decreases the maximum possible value for `SRTO_PAYLOADSIZE`.

- For File mode: Default value is 0 and it's recommended not to be changed.

#### SRTO_PBKEYLEN

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_PBKEYLEN`      | 0.0.0 | pre     | `int32_t`  | bytes   | 0        | *      | RW  | GSD    |

- Sender encryption key length.

- Possible values:
  - 0  = `PBKEYLEN` (default value)
  - 16 = AES-128 (effective value)
  - 24 = AES-192
  - 32 = AES-256

- The use is slightly different in 1.2.0 (HSv4), and since 1.3.0 (HSv5):

  - **HSv4**: This is set on the sender and enables encryption, if not 0. The receiver shall not set it and will agree on the length as defined by the sender.
  
  - **HSv5**: The "default value" for `PBKEYLEN` is 0, which means that the `PBKEYLEN` won't be advertised. The "effective value" for `PBKEYLEN` is 16, but this applies only when neither party has set the value explicitly (i.e. when both are initially at the default value of 0). If any party *has* set an explicit value (16, 24, 32) it will be advertised in the handshake. If the other party remains at the default 0, it will accept the peer's value. The situation where both parties set a value should be treated carefully. Actually there are three intended methods of defining it, and all other uses are considered undefined behavior:
  
    - **Unidirectional**: the sender shall set `PBKEYLEN` and the receiver shall not alter the default value 0. The effective `PBKEYLEN` will be the one set on the sender. The receiver need not know the sender's `PBKEYLEN`, just the passphrase, `PBKEYLEN` will be correctly passed.

    - **Bidirectional in Caller-Listener arrangement**: it is recommended to use a rule whereby you will be setting the `PBKEYLEN` exclusively either on the Listener or on the Caller. The value set on the Listener will win, if set on both parties.

    - **Bidirectional in Rendezvous arrangement**: you have to know the passphrases for both parties, as well as `PBKEYLEN`. Set `PBKEYLEN` to the same value on both parties (or leave the default value on both parties, which will result in 16)

    - **Unwanted behavior cases**: if both parties set `PBKEYLEN` and the value on both sides is different, the effective `PBKEYLEN` will be the one that is set on the Responder party, which may also override the `PBKEYLEN` 32 set by the sender to value 16 if such value was used by the receiver. The Responder party is the Listener in a Caller-Listener arrangement. In Rendezvous it's a matter of luck which party becomes the Responder.

#### SRTO_PEERIDLETIMEO

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_PEERIDLETIMEO` | 1.3.3 | pre     | `int32_t`  | ms      | 5000     | 0..    | RW  | GSD+   |

- The maximum time in `[ms]` to wait until another packet is received from a peer 
since the last such packet reception. If this time is passed, the connection is 
considered broken on timeout.


[Return to list](#list-of-options)



#### SRTO_PEERLATENCY

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_PEERLATENCY`   | 1.3.0 | pre     | `int32_t`  | ms      | 0        | 0..    | RW  | GSD    |

- 数据发送方为接收方设置的最小接收延迟时间(`SRTO_RCVLATENCY`)。

>请注意，读取时返回未连接套接字上的预设值，或是已连接套接字上的有效值。

#### SRTO_PEERVERSION

| OptName              | Since | Binding | Type       |  Units  | Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | -------- | ------ | --- | ------ |
| `SRTO_PEERVERSION`   | 1.1.0 |         | `int32_t`  | *       |          |        | R   | GS     |

- SRT version used by the peer. The value 0 is returned if not connected, SRT handshake not yet performed (HSv4 only), or if peer is not SRT. See [`SRTO_VERSION`](#SRTO_VERSION) for the version format.

#### SRTO_RCVBUF

| OptName              | Since | Binding | Type       |  Units  |   Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | ---------- | ------ | --- | ------ |
| `SRTO_RCVBUF`        |       | pre     | `int32_t`  | bytes   | 8192 bufs  | *      | RW  | GSD+   |

- 接收缓冲区大小（字节）。但是这个值的内部设置是以缓冲区的数量为单位的（其实就是包数量），每个缓冲区的大小等于SRT有效负载大小，即`SRTO_MSS`的值减去UDP和SRT头大小（28和16）。此处设置的值将有效地与有效负载大小的倍数对齐。
- 最小值：32个缓冲区（大概是46592(32x(1500-28-16))，默认值为`SRTO_MSS`）。
- 最大值：`SRTO_FC`（接收器缓冲区不得大于飞行数据的大小）。

> 这个参数控制每个SRT会话的接收缓冲区大小，也会在握手包中的`Flow Window`字段携带。

#### SRTO_RCVDATA

| OptName           | Since | Binding | Type       |  Units  |   Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | ---------- | ------ | --- | ------ |
| `SRTO_RCVDATA`    |       |         | `int32_t`  | pkts    |            |        | R   | S      |

- Size of the available data in the receive buffer.

#### SRTO_RCVKMSTATE

| OptName           | Since | Binding | Type       |  Units  |   Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | ---------- | ------ | --- | ------ |
| `SRTO_RCVKMSTATE` | 1.2.0 |         | `int32_t`  | enum    |            |        | R   | S      |
 
- KM state on the agent side when it's a receiver.

- Values defined in enum [`SRT_KM_STATE`](#2-srt_km_state).


[Return to list](#list-of-options)



#### SRTO_RCVLATENCY

| OptName           | Since | Binding | Type       |  Units  |   Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | ---------- | ------ | --- | ------ |
| `SRTO_RCVLATENCY` | 1.3.0 | pre     | `int32_t`  | msec    | *          | 0..    | RW  | GSD    |

- Latency value in the receiving direction. This value is only significant when`SRTO_TSBPDMODE` is set to true.

- Latency refers to the time that elapses from the moment a packet is sent to the moment when it's delivered to a receiver application. The SRT latency setting should be a buffer large enough to cover the time spent for sending, unexpectedly extended RTT time, and the time needed to retransmit any lost UDP packet. The effective latency value will be the maximum between the `SRTO_RCVLATENCY` value and the value of `SRTO_PEERLATENCY` set by the peer side. **This option in pre-1.3.0 version is available only as** `SRTO_LATENCY`. Note that the real latency value may be slightly different than this setting due to the impossibility of perfectly measuring exactly the same point in time at both parties simultaneously. What is important with latency is that its actual value, once set with the connection, is kept constant throughout the duration of a connection.

- Default value: 120 in Live mode, 0 in File mode (see [`SRTO_TRANSTYPE`](#SRTO_TRANSTYPE)).

- Note that when reading, the value will report the preset value on a non-connected socket, and the effective value on a connected socket.

#### SRTO_RCVSYN

| OptName           | Since | Binding | Type       |  Units  |   Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | ---------- | ------ | --- | ------ |
| `SRTO_RCVSYN`     |       | post    | `bool`     |         | true       |        | RW  | GSI    |

- When true, sets blocking mode on reading function when it's not ready to
perform the operation. When false ("non-blocking mode"), the reading function
will in this case report error `SRT_EASYNCRCV` and return immediately. Details
depend on the tested entity:

- On a connected socket or group this applies to a receiving function
(`srt_recv` and others) and a situation when there are no data available for
reading. The readiness state for this operation can be tested by checking the
`SRT_EPOLL_IN` flag on the aforementioned socket or group.

- On a freshly created socket or group that is about to be connected to a peer
listener this applies to any `srt_connect` call (and derived), which in
"non-blocking mode" always returns immediately. The connected state for that
socket or group can be tested by checking the `SRT_EPOLL_OUT` flag. Note
that a socket that failed to connect doesn't change the `SRTS_CONNECTING`
state and can be found out only by testing the `SRT_EPOLL_ERR` flag.

- On a listener socket this applies to `srt_accept` call. The readiness state
for this operation can be tested by checking the `SRT_EPOLL_IN` flag on
this listener socket. This flag is also derived from the listener socket
by the accepted socket or group, although the meaning of this flag is
effectively different.

- Note that when this flag is set only on a group, it applies to a
specific receiving operation being done on that group (i.e. it is not
derived from the socket of which the group is a member).



[Return to list](#list-of-options)



#### SRTO_RCVTIMEO

| OptName           | Since | Binding | Type       |  Units  |   Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | ---------- | ------ | --- | ------ |
| `SRTO_RCVTIMEO`   |       | post    | `int32_t`  | ms      | -1         | -1, 0..| RW  | GSI    |

- Limits the time up to which the receiving operation will block (see
[`SRTO_RCVSYN`](#SRTO_RCVSYN) for details), such that when this time is exceeded, 
it will behave as if in "non-blocking mode". The -1 value means no time limit.


[Return to list](#list-of-options)



#### SRTO_RENDEZVOUS

| OptName           | Since | Binding | Type       |  Units  |   Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | ---------- | ------ | --- | ------ |
| `SRTO_RENDEZVOUS` |       | pre     | `bool`     |         | false      |        | RW  | S      |

- Use Rendezvous connection mode (both sides must set this and both must use the
procedure of `srt_bind` and then `srt_connect` (or `srt_rendezvous`) to one another.


[Return to list](#list-of-options)



#### SRTO_RETRANSMITALGO

| OptName               | Since | Binding | Type      | Units  | Default | Range  | Dir | Entity |
| --------------------- | ----- | ------- | --------- | ------ | ------- | ------ | --- | ------ |
| `SRTO_RETRANSMITALGO` | 1.4.2 | pre     | `int32_t` |        | 0       | [0, 1] | W   | GSD    |

- Retransmission algorithm to use (SENDER option):
   - 0 - Default (retransmit on every loss report).
   - 1 - Reduced retransmissions (not more often than once per RTT); reduced 
     bandwidth consumption.

- This option is effective only on the sending side. It influences the decision 
as to whether particular reported lost packets should be retransmitted at a 
certain time or not.


[Return to list](#list-of-options)



#### SRTO_REUSEADDR

| OptName           | Since | Binding | Type       |  Units  |   Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | ---------- | ------ | --- | ------ |
| `SRTO_REUSEADDR`  |       | pre     | `bool`     |         | true       |        | RW  | GSD    |

- When true, allows the SRT socket to use the binding address used already by 
another SRT socket in the same application. Note that SRT socket uses an 
intermediate object called Multiplexer to access the underlying UDP sockets, 
so multiple SRT sockets may share one UDP socket, and the packets received by this 
UDP socket will be correctly dispatched to the SRT socket to which they are 
currently destined. This has some similarities to the `SO_REUSEADDR` system socket 
option, although it's only used inside SRT. 

- *TODO: This option weirdly only allows the socket used in **bind()** to use the 
local address that another socket is already using, but not to disallow another 
socket in the same application to use the binding address that the current 
socket is already using. What it actually changes is that when given an address in 
**bind()** is already used by another socket, this option will make the binding 
fail instead of adding the socket to the shared group of that socket that 
already has bound this address - but it will not disallow another socket to reuse 
its address.* 


[Return to list](#list-of-options)



#### SRTO_SENDER

| OptName           | Since | Binding | Type       |  Units  |   Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | ---------- | ------ | --- | ------ |
| `SRTO_SENDER`     | 1.0.4 | pre     | `bool`     |         | false      |        | W   | S      |

- Set sender side. The side that sets this flag is expected to be a sender. This 
flag is only required when communicating with a receiver that uses SRT version 
less than 1.3.0 (and hence *HSv4* handshake), in which case if not set properly, 
the TSBPD mode (see [`SRTO_TSBPDMODE`](#SRTO_TSBPDMODE)) or encryption will not 
work. Setting `SRTO_MINVERSION` to 1.3.0 is therefore recommended.


[Return to list](#list-of-options)



#### SRTO_SNDBUF

| OptName           | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_SNDBUF`     |       | pre     | `int32_t`  | bytes   |8192 bufs  | *      | RW  | GSD+   |

- Sender Buffer Size. See [`SRTO_RCVBUF`](#SRTO_RCVBUF) for more information.

#### SRTO_SNDDATA

| OptName           | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_SNDDATA`    |       |         | `int32_t`  | pkts    |           |        | R   | S      |

- Size of the unacknowledged data in send buffer.

#### SRTO_SNDDROPDELAY

| OptName              | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_SNDDROPDELAY`  | 1.3.2 | pre     | `int32_t`  | ms      | *         | -1..   | W   | GSD+   |

- Sets an extra delay before `TLPKTDROP` is triggered on the data sender.
This delay is added to the default drop delay time interval value. Keep in mind that the longer the delay, the more probable it becomes that packets would be retransmitted uselessly because they will be dropped by the receiver anyway.

- `TLPKTDROP` discards packets reported as lost if it is already too late to send them (the receiver would discard them even if received). The delay before the `TLPKTDROP` mechanism is triggered consists of the SRT latency (`SRTO_PEERLATENCY`), plus `SRTO_SNDDROPDELAY`, plus `2 * interval between sending ACKs` (where the default `interval between sending ACKs` is 10 milliseconds).
The minimum delay is `1000 + 2 * interval between sending ACKs` milliseconds.

- **Special value -1**: Do not drop packets on the sender at all (retransmit them always when requested).

- Default: 0 in Live mode, -1 in File mode.

#### SRTO_SNDKMSTATE

| OptName              | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_SNDKMSTATE`    | 1.2.0 | post    | `int32_t`  |  enum   |           |        | R   | S      |

- Peer KM state on receiver side for `SRTO_KMSTATE`

- Values defined in enum [`SRT_KM_STATE`](#2-srt_km_state).


[Return to list](#list-of-options)



#### SRTO_SNDSYN

| OptName              | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_SNDSYN`        |       | post    | `bool`     |         | true      |        | RW  | GSI    |

- When true, sets blocking mode on writing function when it's not ready to perform the operation. When false ("non-blocking mode"), the writing function will in this case report error `SRT_EASYNCSND` and return immediately.

- On a connected socket or group this applies to a sending function(`srt_send` and others) and a situation when there's no free space in the sender buffer, caused by inability to send all the scheduled data over the network. Readiness for this operation can be tested by checking the `SRT_EPOLL_OUT` flag.

- On a freshly created socket or group it will have no effect until the socket enters a connected state.

- 对于Listener模式套接字，它继承自 it will be derived by the accepted socket or group,but will have no effect on the listener socket itself.

#### SRTO_SNDTIMEO

| OptName              | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_SNDTIMEO`      |       | post    | `int32_t`  | ms      | -1        | -1..   | RW  | GSI    |

- 限制发送操作阻塞的时间（有关详细信息，请参见`SRTO_SNDSYN`），当超过此时间时，它将表现为“非阻塞模式”。-1值表示没有时间限制。

#### SRTO_STATE

| OptName              | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_STATE`         |       |         | `int32_t`  |  enum   |           |        | R   | S      |

- 获取当前套接字的状态，建议直接使用`srt_getsockstate`。

#### SRTO_STREAMID

| OptName              | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| -------------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_STREAMID`      | 1.3.0 | pre     | `string`   |         | ""        | [512]  | RW  | GSD    |

- 连接之前在套接字上设置的流ID，字符串类型。侦听器端将能够从`srt_accept`返回的套接字中检索此流ID（对于具有该流ID的已连接套接字）。通常在`srt_connect`的套接字上设置，然后使用从`srt_accept`检索的套接字。此字符串的值定义可以是任意值，但是，强烈建议遵循[SRT访问控制指南](access_control.md)。
- 因为这在内部使用“STD::String”类型，所以在Brime/C++ API（UDT .h）中有附加的函数：“SRT:：StStrasID”和“SRT::GETStrayID”。
- 此选项对于会合连接不起作用，因为一方会覆盖另一方的值，从而导致任意赢家。同样，在这种情况下，两个对等点彼此都是已知的，并且在连接中都具有相同的角色。

#### SRTO_TLPKTDROP

| OptName           | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_TLPKTDROP`  | 1.0.6 | pre     | `bool`     |         | *         |        | RW  | GSD    |

- 丢弃太晚达到的包。当在接收器上启用时，它将丢弃收到的超时的数据包，并在其播放时间到来时将后续数据包传递给应用程序。它会向发送方发送一个ACK，以避免发送方的重传。当在发送方和接收方启用时，发送方将丢弃没有机会及时传递的旧数据包。如果接收方支持它，它将在发送方自动启用。
- 默认值: 在LIVE模式下为true，在文件模式下为false（请参见[`SRTO_TRANSTYPE`](#SRTO_TRANSTYPE)）。

#### SRTO_TRANSTYPE

| OptName           | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_TRANSTYPE`  | 1.3.0 | pre     | `int32_t`  |  enum   |`SRTT_LIVE`| *      | W   | S      |

- 设置套接字的传输类型，默认时LIVE模式，设置此选项可根据特定传输类型为其它选择设定默认值。
- 由枚举`SRT_TRANSTYPE`定义的值：`SRTT_LIVE` `SRTT_FILE`。

#### SRTO_TSBPDMODE

| OptName           | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_TSBPDMODE`  | 0.0.0 | pre     | `bool`     |         | *         |        | W   | S      |

- 如果为true，则使用基于时间戳的数据包传递模式。在这种模式下，包的时间戳在发送时设置（或也可以被预定义），在SRT报文头中携带传输，然后在接收端恢复，以便在接收端应用程序区分报文分组之间的时间间隔。
- 默认值：在LIVE模式下为true，在文件模式下为false（请参见[`SRTO_TRANSTYPE`](#SRTO_TRANSTYPE)）。

#### SRTO_UDP_RCVBUF

| OptName           | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_UDP_RCVBUF` |       | pre     | `int32_t`  | bytes   | 8192 bufs | *      | RW  | GSD+   |

- UDP套接字的接收缓冲区字节大小。

#### SRTO_UDP_SNDBUF

| OptName           | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_UDP_SNDBUF` |       | pre     | `int32_t`  | bytes   | 65536     | *      | RW  | GSD+   |

- UDP套接字的发送缓冲区字节大小。

#### SRTO_VERSION

| OptName           | Since | Binding | Type       |  Units  |  Default  | Range  | Dir | Entity |
| ----------------- | ----- | ------- | ---------- | ------- | --------- | ------ | --- | ------ |
| `SRTO_VERSION`    | 1.1.0 |         | `int32_t`  |         |           |        | R   | S      |

- 如果未连接，获取的值是本地SRT版本。如果已连接，则为双方支持的最高版本。
- 对于版本号x.y.z，十六进制版本格式为`0x00XXYYZZ`。例如，版本1.4.2编码为`0x00010402`。
