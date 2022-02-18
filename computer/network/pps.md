# How many Packets per Second per port are needed to achieve Wire-Speed?

from: <https://kb.juniper.net/InfoCenter/index?page=content&id=KB14737>

## SUMMARY

When evaluating or measuring an Ethernet device's (switches, routers, firewalls) performance capabilities, the main indicator that most will consider is the raw bandwidth that the device backplane can provide.

However it is also important to make sure that the device has the capacity or the ability to switch/route as many packets as required to achieve wire rate performance. This metric is called the ‘Packets per Second’ or PPS for short.

This article details how to calculate how many packets per second processing capabilities is required from a port to achieve wire-rate performance.

## SOLUTION

*Note: This article focuses on Ethernet; other mediums such as ATM will have other considerations for calculating PPS.*

To calculate the amount of packets per second a port must be able to handle to achieve wire-rate performance we need to take into consideration the fact that the IP protocol allows for variable payload sizes which in turn plays a part in our PPS calculation.

The smaller the packet passing on the wire the more packets that need to be switched to achieve wire-rate performance; while on the other hand, larger packets will require less PPS throughput to achieve wire-rate. As such to calculate how many PPS needed to achieve wire-speed we need only be concerned with small packet sizes since they will be the most taxing for the switch and will yield the larger PPS number. Naturally we will assume that no collisions occur on the medium.

We need to see how much ‘space’ each packet will occupy so we will look at the frame size in which the smallest packet will be encapsulated, as well as the inter-frame gap, and the preamble since they occupy ‘space’ in between frames.

![Figure1](resource/figure1.jfif "Figure 1: ‘Space’ Occupied by the smallest packet")

As seen in figure 1, a packet will occupy at least 84 bytes on the wire. So taking the example of a 1G port we first have to convert the speed into bytes:

**1Gbps = 1,000,000,000 bits/s = (1,000,000,000 bits/s) / (8 bits/byte)= 125,000,000 bytes/s**

Then we can calculate how many packets per second need to be processed if the port is to transmit at wire speed:

**PPS = (125,000,000 bytes/s) / (84 bytes/packet) = 1,488,095 pps.**

Similarly this same calculation can be extended to different port speeds:

|Speed|bits/second|bytes/sencond|maximum PPS|
|-|-|-|-|
|10Mbps|10,000,000|1,250,000|14,881|
|100Mbps|100,000,000|12,500,000|148,810|
|1Gbps|1,000,000,000|125,000,000|1,488,095|
|10Gbps|10,000,000,000|1,250,000,000|14,880,952|
|100Gbps|100,000,000,000|12,500,000,000|148,809,524|

From the above table we see how many packets per second an Ethernet device must be able to handle per port in order to achieve wire speed. The above calculation however does not take into consideration additional tags that might be available in an Ethernet frame such as VLAN tags or MPLS labels. These will be covered in the following sections.

## VLAN Tags

VLAN Tagging or Trunking is used to carry multiple VLANs over one physical wire. To do so we need a distinguisher for the different VLANs traversing the link. There are multiple methods to perform the VLAN identification but for the scope of this article we will only look at the standards based method which is the IEEE 802.1q.

The IEEE 802.1q standard states that Ethernet ports in trunk mode will insert a 32bit field between the Source MAC address and the Type field. Consequently our ‘minimum’ frame will have the following appearance on the wire:

![Figure2](resource/figure2.jfif "Figure 2: 802.1Q tagged minimum frame")

As we can see the minimum frame has increased in size by 4 bytes. Let’s examine how this will affect the calculations we have performed above since the minimum ‘space’ that a frame will occupy has increased from 84 bytes to 88 bytes while taking the wire speed of 1Gbps as an example:

**1Gbps = 1,000,000,000 bits/s = (1,000,000,000 bits/s) / (8 bits/byte)= 125,000,000 bytes/s**

**PPS = (125,000,000 bytes/s) / (88 bytes/packet) = 1,420,454 pps.**

From the above calculation, it is apparent that the amount of packets that the wire can pass per second has decreased slightly. This is to be expected since the wire now has to carry more information for every packet processed.

The above table is shown below with the values recalculated for 802.1Q tagged packets.

|Speed|bits/second|bytes/sencond|maximum PPS|
|-|-|-|-|
|10Mbps|10,000,000|1,250,000|14,205|
|100Mbps|100,000,000|12,500,000|142,045|
|1Gbps|1,000,000,000|125,000,000|1,420,455|
|10Gbps|10,000,000,000|1,250,000,000|14,204,545|
|100Gbps|100,000,000,000|12,500,000,000|142,045,455|

It is worth noting that within the 802.1Q standard, there is a concept of a ‘Native VLAN’. This means that dot1Q will not tag the frame egressing the trunk port for one select VLAN. For these particular frames, the first calculation will apply since they do not contain the dot1Q tag.

## Q-in-Q

An amendment to the 802.1Q standard is the 802.1ad or otherwise known as Q-in-Q. The purpose of this amendment was to create a method for users to run their own VLANs inside the VLANs offered by a Metro Ethernet Service Provider. To achieve this goal a second tag is inserted in the Ethernet frame to distinguish the customer VLANs as shown below:

![Figure3](resource/figure3.jfif "Figure 3: 802.1ad tagged frame")

For the purposes of our calculation this second tag will increase the ‘space’ a single frame will occupy by another 4 bytes. As such the smallest frame size will increase again from 88 bytes to 92 bytes total. Therefore our speeds/PPS table will look as follows:

|Speed|bits/second|bytes/sencond|maximum PPS|
|-|-|-|-|
|10Mbps|10,000,000|1,250,000|13,587|
|100Mbps|100,000,000|12,500,000|135,870|
|1Gbps|1,000,000,000|125,000,000|1,358,696|
|10Gbps|10,000,000,000|1,250,000,000|13,586,957|
|100Gbps|100,000,000,000|12,500,000,000|135,869,565|

As it is becoming readily noticeable, the more information we encode in the packet the more the maximum number of packets per second limit drops for each wire speed.

## 参考

<https://zh.wikipedia.org/wiki/%E4%BB%A5%E5%A4%AA%E7%BD%91%E5%B8%A7%E6%A0%BC%E5%BC%8F>

<http://en.wikipedia.org/wiki/Wire_speed>

<http://en.wikipedia.org/wiki/Interframe_gap>

<http://www.erg.abdn.ac.uk/users/gorry/course/lan-pages/mac.html>

<http://en.wikipedia.org/wiki/IEEE_802.1Q>

<http://en.wikipedia.org/wiki/IEEE_802.1ad-2005>

<http://www.ietf.org/rfc/rfc3032.txt>
