---
title: 计算机延迟（Latency time）
author: huimingz
date: 2021-02-11 20:17:05 +0800
category: [facility]
tags: [facility]
pin: true
---

2001 年，Peter Norvig（曾任 Google 搜索质量总监，经典教材《人工智能：一种现代方法》的作者之一）在 “ Teach Yourself Programming in Ten Years” 一文中，首次讨论了计算机领域中 memory，Cache，Disk，network 等相关延时的数据（By the way，“ Teach Yourself Programming in Ten Years”这篇文章本身也是神文，20 年过去了，依旧经典）。

![](https://mmbiz.qpic.cn/mmbiz_jpg/la8s6uvJibdR3AKJ73XnRn9lyCPwrmC0ibrUO2pRfus2ibFibUU5dic8e4ydkagbYzwUjmml1knpOYnONLFsXB7EEvA/640?wx_fmt=jpeg&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

而让这些数字再次受关注的，是 Google 的另一位大神。2010 年，Jeff  Dean 在斯坦福做了一次精彩的演讲：“Building Software Systems at Google and Lessons Learned”，其中又讨论了这些计算机领域的 latency 。毕竟，要做出优秀的架构，反而需要更加关注那些基础的领域知识。

![](/assets/img/posts/Xnip2021-02-11_20-36-36.png)

随后，一名 UC Berkeley 的学生将这些计算机系统中的 latency time 做了一个可交互的网页，得出这些指标的计算公式也写在网页的源码里（https://colin-scott.github.io/personal_website/research/interactive_latency.html）。

拖动标记年份的滚动条,  我们可以很直观地看到，不同年份下，CPU cache，main memory，network 这些技术发展对系统延迟时间的影响。

![](/assets/img/posts/Xnip2021-02-11_20-20-38.png)

下面是 2020 年的数据：

| L1 cache reference<br>读取 CPU 一级缓存 latency                                            	| 1 ns<br>1 纳秒 = 10 亿分之一秒            	|
| L2 cache reference<br>读取 CPU 二级缓存 latency                                            	| 4 ns                                      	|
| CPU Branch mispredict<br>CPU 分支错误预测耗时                                              	| 3 ns                                      	|
| Mutex Lock / Unlock<br><br>加互斥锁 / 解锁                                                 	| 17 ns                                     	|
| Main memory reference<br><br>读取主内存数据                                                	| 100 ns                                    	|
| Compress 1 KB with Zippy<br>1 k 字节压缩                                                   	| 2000 ns = 2 μs<br>1 微秒 = 100 万分之一秒 	|
| Read 1,000,000 bytes sequentially from main memory<br><br>从主内存中顺序读取 1M 字节数据   	| 3 μs                                      	|
| Solid State Drive (SSD) random read<br> 从 SSD 中读取数据                                  	| 16 μs                                     	|
| Read 1,000,000 bytes sequentially from SSD<br>从 SSD 中顺序读取 1M 字节数据                	| 49 μs                                     	|
| Read 1,000,000 bytes sequentially from disk<br>从机械硬盘中顺序读取 1M 字节数据            	| 825 μs                                    	|
| Disk (Hard / magnetic drive) seek<br><br>从机械硬盘中随机读取数据                          	| 2 ms<br>1 毫秒 = 1000 分之一秒            	|
| 网络 latency                                                                               	|                                           	|
| Send 2000 bytes over commodity <br>network<br>通过商用网络发送 2k 字节数据                 	| 44 ns                                     	|
| Round trip network request in same <br>data centre<br>在同一数据中心的一次网络请求往返耗时 	| 500 μs                                    	|
| Packet roundtrip CA to Netherlands 从加拿大到荷兰的一次网络请求往返耗时                    	| 150 ms                                    	|

通过分析这些变化，可以发现一些有意思的事情：

1.  从 2006 年开始，前两列操作的数值（L1 cache，L2 cache，解压缩）基本不再变化。从另一侧面反映了 CPU 芯片发展的时代烙印：从注重提升主频到往多核方向发展；

2.  机械硬盘的随机读写性能很差，比内存慢 1000 倍。而异地访问的网络请求时延，又比硬盘读慢 100 倍；SSD 的随机读取速度 20 年间基本没有提升，而顺序读取速度却从 50 ms 提升到 49 μs，提升了 1000 倍。在某些场景下，基于 SSD 的顺序读性能甚至好于内存的随机读；

3.  不管是在同一数据中心还是异地数据中心，RTT 基本不变，一致保持 500 μs 和 150 ms。假设信号在光纤中以近似光速传播，理论上 RTT 由物理规律决定。比如：杭州到北京距离大约为 1500 km，RTT 延迟至少在： ( 2 * 1500 km ) / (300,000 km/s) = 10 ms。

