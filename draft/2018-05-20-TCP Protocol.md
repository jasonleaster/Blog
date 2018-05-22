---
title: '那个精心设计的传输控制协议 -- TCP '
date: 2018-05-20 23:41:55
tags: Network
---

声明: 以下内容有较强的个人总结观点，请批判归纳采用。

### 背景

![images](/images/img_for_2018_05/communication.jpg)

TCP(Transmission Control Protocol) 传输控制协议，是一种面向连接的、可靠的、基于字节流的传输控制层的通信协议。而每次说到TCP又不得不提另外一个传输协议，UDP(User Datagram Protocol)用户数据包协议。两者共筑互联网的重要基石。计算机历史上，一定是为了解决某个问题，再去探寻解决方案，再规范化成协议，这两个传输协议亦是如此，两者表现不同，各有优劣，各自有各自的使用场景。

TCP的出现的目的其实上面已经讲了，重点就是要建立**可靠**的网络通信机制。而下面的网络层协议IP协议是不会保证两端通信数据的一致性的，丢了也不管。要在复杂不稳定的网络环境下创建一套机制去保证通信的可靠性，一致性/幂等性 是不容易的。

首先个人观点，实现一致性是比较困难的，我目前知道的就两种方法:

1. 客户端、服务端在通信时，给数据包带上顺序号和标签，用来校验识别、检验"实际情况和真实情况是否一致"。 (场景: 基于连接的网络通信)

2. 采用消息记录的策略，给每个消息一个sid，对应的要有个消息表确认字段。如果该同一消息id如果成功处理过了，或者正在处理则忽略该消息(保证幂等性)。消息处理成功，服务端负责发送消息给客户端通知处理成功。(场景: 银行转账，消息事务)

前者实际上就是TCP采用的策略，而后者则是网络应用层常见的消息幂等性的策略。

回到主题，分三个部分，连接创建、数据传送和连接终止，具体的讲讲TCP协议。

![images](/images/img_for_2018_05/TCP_lifecycle.jpg)

### 连接创建

TCP连接的创建需要三次"握手"，

    * 客户端通过向服务器端发送一个SYN来创建一个主动打开，作为三路握手的一部分。客户端把这段连接的序号设定为随机数A。
    * 服务器端应当为一个合法的SYN回送一个SYN/ACK。ACK的确认码应为A+1，SYN/ACK包本身又有一个随机产生的序号B。
    * 最后，客户端再发送一个ACK。当服务端收到这个ACK的时候，就完成了三路握手，并进入了连接创建状态。此时包的序号被设定为收到的确认号A+1，而响应号则为B+1。

FAQ:

Q: 为什么握手的过程要三次，不是两次，一次，或者四次？
A: 通过序列号即响应+1校验的策略，因为第一次和第二次能够让客户端知道，服务端有能力响应自己的请求。第二次和第三次是为了让服务端知道，客户端有能力响应自己的请求。双方彼此确认。如果SYN+ACK分开，握手也可以分成4次握手，但是服务端接收到第一次的SYN之后，发送ACK和SYN的过程合并在一起了，于是就把4次握手的需求降到了三次完成了，一个TCP包头同时拥有这两个字段。(后面会解释为什么挥手的过程需要四次)

Q: 如果第二次握手的数据包在到达之前，客户端挂掉了呢？

任何一个数据包发送出去之后，只要没有收到ACK响应，或者ACK不等于SEQ+1，那么数据发送方就会尝试重传。这是TCP协议最基本的原则。如果第一次握手数据包就到不到服务端，或者服务端的数据客户端接受不到，那么客户端就会重发，重发的次数等于tcp_syn_retrie（默认6次）s。如果是第三次握手服务端收不到数据包，那么服务端会认为客户端“听不到自己”，会重发第二次握手的信息给客户端，重发的次数等于tcp_synack_retries（默认5次）。上述提到的两个参数可能随着内核版本的不同而不同，也可以人为手动配置，通过 /etc/sysctl.conf 系统配置文件进行配置。至此问题的答案就很明显了。

更多详细参数的介绍可以参考下面的文章或者内核说明文档。

http://colobu.com/2014/09/18/linux-tcpip-tuning/

https://stackoverflow.com/questions/16259774/what-if-a-tcp-handshake-segment-is-lost

Q: TCP最大连接数，客户端和服务端是一样的吗？如果不是，服务端的理论最大连接数是多少，实际最大连接数可能达到多少，说明考虑的因素？

http://www.cnblogs.com/fjping0606/p/4729389.html

### 数据传送

Q: 如果在三次握手成功之后，数据开始传输的过程中丢包了，TCP会如何处理?

A: 此种情况下，在指定的时间内，发送者没有收到ack响应之后就会触发重传，如果重传的次数超过系统设置参数，则放弃重传，视为网络故障不可用。

tcp_retries1 (integer; default: 3; since Linux 2.2)

The number of times TCP will attempt to retransmit a packet on an established connection normally, without the extra effort of getting the network layers involved. Once we exceed this number of retransmits, we first have the network layer update the route if possible before each new retransmit. The default is the RFC specified minimum of 3.

tcp_retries2 (integer; default: 15; since Linux 2.2)

The maximum number of times a TCP packet is retransmitted in established state before giving up. The default value is 15, which corresponds to a duration of approxi‐mately between 13 to 30 minutes, depending on the retransmission timeout. The RFC 1122 specified minimum limit of 100 seconds is typically deemed too short.

Q: 如果TCP在没有数据需要继续传输的情况下，会作何处理？
A: 为了维持连接状态，在空闲了(tcp_keepalive_time)秒之后，一方会每个一段时间(tcp_keepalive_intvl)会发送一个特殊的数据包(Probe Segment)强制对方回应，如果在指定的时间内没有得到回应，则重新发送Probe数据包，重传次数超过tcp_keepalive_probes时，就认为对方已经无法正常通行，连接挂掉。(补充：此时发起方的连接是否会被清理掉还需要继续研究)

### 连接终止

![images](/images/img_for_2018_05/TCP_CLOSE.svg.png)

Q: 为什么TCP的连接终止需要四次？
和之前链接建立的过程的原理类似，但是客户端和服务端由于关闭链接的时候需要释放链接持有的计算资源，两者耗时不一样，故需要独立的关闭。被动关闭的一方会先相应ACK，待被动关闭者资源正确关闭之后(也可能会是还有数据需要继续发送)，才会发送FIN，去通知主动发起方，自己也可以关闭了。之前合并的FIN+ACK过程被分开，故变成四次挥手。

Q: 如果主动断开连接方发送FIN之后，并且受到ACK，此时不能发送任何数据了，只等待被动关闭方发送FIN。但是如果对方一直不发送FIN呢？
A: 这样主动断开方就“被晾着了”，一直很尴尬的处于FIN_WAIT_2状态，如果真的一直收不到被动方发来的FIN，且超过tcp_fin_timeout设定的超时秒数，此时主动方的套接字会被强制关闭。
