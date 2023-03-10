---
title:  专业术语
showOnHome: true
...


## 数据分析
类似Google Analytics这类数据分析，常用于广告投放 (Google Analytics )等分析。 参考 https://prestigemarketing.ca/uv-pi-pv-cpm-cpc-ppc-cpa-ron-roc-ros-sov-ctr-rtb-online-advertising-acronyms/

* Impression  - 印象次数  代表这个“广告”进入过你的视线；这个在搜索引擎场景也存在，被你搜索出来了，就代表给了你Impression，无论你是否点击进入
* CLICK - 广告活动被点击的次数。 相对Impression来讲， Impression只代表出现在了你的实视线内。 Ads Campaign or ad group clicks over time. (Total number of times users have clicked on an ad to reach the property)
* UV 	在特定时间段内访问网站页面的不同个人的数量，这个计算忽略了多次访问。
* PV  网页浏览次数；Page View， 这个针对Web网站来讲的，指定时间网页被浏览了多少次
* CPM - Cost Per Thousands (Impressions)： 网站上 1,000 次广告展示的成本。
* CPC - Cost Per Click ：每个点击的成本；即：每次点击需支付的价格。
* CPD - Cost Per Day:  每天的成本
* CTR - Click Through Rate： 个人理解就是“印象转换率”，简单来就讲就是多少比率的Impression 转换成CLICK了， 这个其实可以用来统计广告够不够吸引人
* CSR - Corporate Social Responsibility （这个不太确定）
* CPS - Cost-Per-Sale：=  Total cost / Total sales number


## 性能相关
PV=PV（Page View）访问量, 即页面浏览量或点击量

UV（Unique Visitor）独立访客，统计1天内访问某站点的用户数(以cookie为依据);访问网站的一台电脑客户端为一个访客

IP（Internet Protocol）独立IP数，是指1天内多少个独立的IP浏览了页面，即统计不同的IP浏览用户数量。

TPS=transactions per second

QPS=queries per second 

RPS=requests per second

RPS=并发数/平均响应时间
## 服务指标
可用性、准确性、系统容量和延迟

可扩展性 一致性 持久性

## CAP 定理
- Consistency  一致性
- Availability  可用性
- Partition-tolerance  分区容错性

三个只能二选一

## 数据库相关
#### OLTP（on-line transaction processing）
联机事务处理；OLTP是传统的关系型数据库的主要应用，主要是基本的、日常的事务处理，例如银行交易。OLAP是数据仓库系统的主要应用，支持复杂的分析操作，侧重决策支持，并且提供直观易懂的查询结果。 

#### OLAP（On-Line Analytical Processing）
联机分析处理系统，有的时候也叫DSS决策支持系统，就是我们说的数据仓库。在这样的系统中，语句的执行量不是考核标准，因为一条语句的执行时间可能会非常长，读取的数据也非常多。所以，在这样的系统中，考核的标准往往是磁盘子系统的吞吐量（带宽），如能达到多少MB/s的流量。
    磁盘子系统的吞吐量则往往取决于磁盘的个数，这个时候，Cache基本是没有效果的，数据库的读写类型基本上是db file scattered read与direct path read/write。应尽量采用个数比较多的磁盘以及比较大的带宽，如4Gb的光纤接口。
在OLAP系统中，常使用分区技术、并行技术。

## 安全相关
#### 防火墙 （Firewall）

别名防护墙，于1993发明并引入国际互联网。

他是一项信息安全的防护系统，依照特定的规则，允许或是限制传输的数据通过。在网络中，所谓的防火墙是指一种将内网和外网分开的方法，他实际上是一种隔离技术

防火墙对流经它的网络通信进行扫描， 这样就能够过滤掉一些攻击，以免其在目标计算机上执行。

通常的防火墙主要工作第二到第四层，重心是在网络层，用于过滤IP和协议类型。

#### WAF (Web Application Firewall) Web应用防火墙

Web应用防火墙是通过执行一系列针对HTTP/HTTPS的安全策略还专门为Web应用提供保护的一款产品。

与传统防火钱不同，WAF工作在应用层。

#### IDS (Intrusion Detection System) 入侵检测系统

依照一定的安全策略，通过软件、硬件、对网络、系统的运行状况进行监视尽可能的发现各种攻击企图、攻击行为或者攻击结果，以保证网络系统资源的机密性、完整性和可用性。

入侵检测检测可分为实时入侵检测和事后入侵检测。

#### IPS (Intrusion Prevention System) 入侵预防系统

IPS是对防病毒软件和防火墙的补充，入侵预防系统是一部能够监视网络或网络设备的网络资料传输行为的计算机网络安全设备，能够即时的中断、调整或隔离一些不正常或是具有伤害性的网络资料传输行为。

## 网络相关

### NAT 

维基百科：
Network address translation (NAT) is a method of remapping an IP address space into another by modifying network address information in the IP header of packets while they are in transit across a traffic routing device.

百度百科：
NAT（Network Address Translation，网络地址转换）是1994年提出的。当在专用网内部的一些主机本来已经分配到了本地IP地址（即仅在本专用网内使用的专用地址），但现在又想和因特网上的主机通信（并不需要加密）时，可使用NAT方法。

SMTP (Simple Mail Transfer Protocol) 是一种提供可靠且有效的电子邮件传输的协议。SMTP是建立在FTP文件传输服务上的一种邮件服务，主要用于系统之间的邮件信息传递，并提供有关来信的通知。
## 编程相关
命令式编程（Imperative）：详细的命令机器怎么（How）去处理一件事情以达到你想要的结果（What）；
声明式编程（ Declarative）：只告诉你想要的结果（What），机器自己摸索过程（How）

## 其他

TTL -  Time To Live  存活时间

