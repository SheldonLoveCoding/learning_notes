# 小林codeing的图解网络阅读笔记

## Questions

### 1. 路由器在发送数据的时候，知道下一步的IP地址之后，如何确定下一步的IP地址对应的MAC地址呢？

首先，在整个数据传输的过程中，网络层确定的IP报文的头部中的 源IP 和 目标IP 是不会发生变化的。在客户端的链路层部分，确定的MAC地址是根据 ==目标IP== 经过 ==子网掩码== &计算之后得到的一个IP_1，找到路由表中的destination列为IP_1的所在**行a**，那么下一步的 IP 就是**行a的Gateway列**的IP_2，然后就查询ARP的缓存表看看是否有IP_2对应的MAC地址，如果没有就通过ARP协议广播找到IP_2的MAC地址，放到MAC的报文头部里，但是并**不修改客户端一开始在网络层确定的IP报文头部中的目标IP**。

如果目标主机不是本地局域网，填入的MAC地址是**路由器**。路由器在发送数据的时候也会利用ARP协议去得到下一步的IP地址对应的MAC地址。（==ARP协议可以在路由器发送数据的时候使用吗？应该也是利用广播吗？==）

 `ARP` 协议在Wiki上的例子：

> Two computers in an office (*Computer 1* and *Computer 2*) are connected to each other in a [local area network](https://en.wikipedia.org/wiki/Local_area_network) by [Ethernet](https://en.wikipedia.org/wiki/Ethernet) cables and [network switches](https://en.wikipedia.org/wiki/Network_switch), with no intervening [gateways](https://en.wikipedia.org/wiki/Gateway_(telecommunications)) or [routers](https://en.wikipedia.org/wiki/Router_(computing)). *Computer 1* has a packet to send to *Computer 2*. Through [DNS](https://en.wikipedia.org/wiki/DNS), it determines that *Computer 2* has the IP address *192.168.0.55*.
>
> To send the message, it also requires *Computer 2*'s [MAC address](https://en.wikipedia.org/wiki/MAC_address). First, *Computer 1* uses a cached ARP table to look up *192.168.0.55* for any existing records of *Computer 2'*s MAC address (*00:EB:24:B2:05:AC*). If the MAC address is found, it sends an Ethernet [frame](https://en.wikipedia.org/wiki/Frame_(networking)) containing the IP packet onto the link with the destination address *00:EB:24:B2:05:AC*. If the cache did not produce a result for *192.168.0.55*, *Computer 1* has to send a broadcast ARP request message (destination *FF:FF:FF:FF:FF:FF* MAC address), which is accepted by all computers on the local network, requesting an answer for *192.168.0.55*.
>
> *Computer 2* responds with an ARP response message containing its MAC and IP addresses. As part of fielding the request, *Computer 2* may insert an entry for *Computer 1* into its ARP table for future use.
>
> *Computer 1* receives and caches the response information in its ARP table and can now send the packet.