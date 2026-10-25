---
title: SQM智能队列管理使用及针对PCDN/BT/PT调优的方案
id: 019d3647-aa41-771c-9428-dff721882038
date: 2026-03-29 05:09:35
auther: squaly
featured_image: Pasted%20image%2020261024074116.png
featured_image_preview: Pasted%20image%2020261024074116.png
summary: 配置教程： 基于你的上下行带宽*85%-95%设置一个值 使用layer_cake.qos，才能为PCDN/BT/PT的流量分类 注意黄色框的内容是要填写的： 入口（下行）填 nat dual-dsthost ingress 出口（上行）填 nat dual-srchost diffserv4 这里
permalink: /archives/uTNnmrUh
categories:
  - network
tags:
  - openwrt
---

配置教程：

基于你的上下行带宽*85%-95%设置一个值

![](Pasted%20image%2020261024074203.png)

使用layer_cake.qos，才能为PCDN/BT/PT的流量分类

![](Pasted%20image%2020261024074219.png)

注意黄色框的内容是要填写的：

入口（下行）填
nat dual-dsthost ingress
出口（上行）填
nat dual-srchost diffserv4

这里的nat dual-xxx意思是在NAT的环境执行下每IP公平策略，即所有IP均分带宽

下行的ingress忘了啥意思了，diffserv4是流量分类的等级

![](Pasted%20image%2020261024074237.png)

如果你是普通家庭宽带、光猫桥接、OpenWrt 拨号或者 DHCP 上网，最省事的默认值就是：

* 链路层适应：Ethernet with overhead
* 单个数据包开销：44
* MPU：64 或 84

44 基本属于“保守但稳”的值，宁可略大也不要太小，不然 SQM 会低估真实带宽占用，反而容易出现 bufferbloat。

大概可以这样理解：

* 普通 DHCP 光纤：22 左右
* PPPoE：30 左右
* PPPoE + VLAN：34 左右
* 不确定自己线路细节：直接填 44 最稳

家里大概率是 PPPoE 或带 VLAN 的光纤宽带，所以直接填 44 就行，不太需要再细扣。

![](Pasted%20image%2020261024074253.png)

到这里，SQM的设置就结束了，但是光这样还不行，因为大家都不怎么care DSCP标识，PCDN/BT/PT的流量还是会跑到高优先级队列。

所以我们还要在iptables里对这些流量打个DSCP标识，以便QOS可以识别它们。因为我家的PT都是单独的机器，所以这里只做单独机器演示，如果不是，按照这个思路想办法：

这句的意思是，对某个mac地址出来的所有流量，都打上CS1的DSCP标签，意为低优先级数据，在SQM的diffserv4分类下，对应的是Bulk，拥塞情况下最高的速度只有3648Kbit

注意，通过iptables对流量打DSCP标志需要关闭快速转发引擎/fastpath之类的东西，有硬件转发的话更不用说了，否则数据包都没到你这就转走了

```
iptables -t mangle -A PREROUTING -m mac --mac-source xx:xx:xx:xx:xx:xx -j DSCP --set-dscp-class CS1 -m comment --comment "dcsp-wxedge"
```
