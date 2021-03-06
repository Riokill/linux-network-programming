.. VRRP虚IP漂移
    FileName:   vrrp-vip-floating.rst
    Author:     Fasion Chan
    Created:    2018-10-10 14:16:09
    @contact:   fasionchan@gmail.com
    @version:   $Id$

    Description:

    Changelog:

.. meta::
    :keywords: VRRP, VIP, virtual IP, 虚IP, 虚IP漂移, 虚IP高可用

============
VRRP虚IP漂移
============

简介
====

`VRRP`_ 是 `Virtual Router Redundancy Protocol` 的简称，即 **虚拟路由冗余协议** 。

`VRRP`_ 最早被设计来解决网关的高可用问题：

我们知道，计算机进行网络通讯时，需要网关来传输网络报文。
每台机器只能配置一个网关地址，这时网关的可靠性就非常重要了。
如果网关不幸故障了，那么使用该网关的所有机器都将受影响——断网了！

解决网关单点问题的思路非常直观——部署一个备用网关，在主网关故障时切换过去。

然而，由于机器只能配置一个网关地址，因此每次切换网关都需要修改该配置。
这个解决方案没能做到自动化，并不优雅。

这时， `VRRP`_ 应运而生！接下来，以一个简单的例子介绍 `VRRP`_ 是如何工作的：

.. figure:: /_images/distributed/vrrp-vip-floating/a35831e7a7910e0b945c47187a99caf6.jpg

    *VRRP最早用于网关高可用*

事情是这样的。

这个网络部署了两台 **路由** 进行互备，本网络内其他机器以这两台路由为网关进行网络通讯。
两台路由的 `IP` 地址分别是： `192.168.1.1` 以及 `192.168.1.2` 。
但路由并不直接通过这些地址提供转发服务，而是使用一个 **虚拟地址** `192.168.1.253` 。
其他计算机，如 `192.168.1.3` 将网关地址配置为 `192.168.1.253` 。

通过 `VRRP`_ ，两台路由互相进行 **健康检查** 。
当两台路由都是健康的情况下，只有主路由对外提供虚拟地址的 `ARP` 响应。
这时，发往虚拟地址 `192.168.1.253` 的流量都由主路由处理。

当主路由故障时，备用路由将检测到。
这时，备用路由开始通过 `ARP` 协议对外通告：虚拟地址 `192.168.1.253` 对应的 `MAC` 地址是我， 被我接管了！

接下来，发往虚拟地址 `192.168.1.253` 的流量就开始由备用路由处理了。
这时，虚拟地址 `192.168.1.253` 看上去就像是 **漂移** 到备用路由上一样。
换句话讲，网关成功进行切换，而且无需修改其他机器的网关配置！

主路由恢复后，将通过类似的手段，重新拿回流量的处理权。
这部分将不再赘述。

完整流程如下：

#. 两台路由互相进行健康检查；
#. 主路由对外响应虚拟地址的 `ARP` 请求，通告其 `MAC` 地址；
#. 虚拟地址网络流量被主路由处理；
#. 备用路由发现主路由故障，开始响应虚拟地址的 `ARP` 请求，通告其 `MAC` 地址；
#. 虚拟地址网络流量被备用路由处理；
#. 主路由恢复，重新响应 `ARP` 请求，夺回流量；
#. 备用路由发现主路由恢复，停止响应 `ARP` 请求，释放流量处理权；

总结起来， `VRRP`_ 主要做两件事情：

#. 通过 `ARP` 响应 `MAC` 地址实现虚 `IP` 漂移；
#. 通过健康检查决定什么时候进行虚 `IP` 漂移；

应用场景
========

本质上， `VRRP`_ 是用来实现高可用的，与网关无关。

我们可以将其应用于一些网络服务的高可用，如 `Web` 服务：

.. figure:: /_images/distributed/vrrp-vip-floating/53524cf41abce963473ce0e7fc6ba399.jpg
    :width: 360px

    *Web服务高可用*

服务高可用方案有很多， `VRRP`_ 特别适用于以下场景：

#. 服务对外只能呈现为单个 `IP` ；
#. 同一时刻只允许一个实例对外服务；

此外， `VRRP`_ 也可用于实现负载均衡设施的高可用。
应用的高可用通过负载均衡设施解决，那么负载均衡设施如何实现高可用呢？
答案是—— `VRRP`_ ！

下面是一个非常典型的例子：

.. figure:: /_images/distributed/vrrp-vip-floating/3661c2082103036ecb23a3f29be740be.gif

    *负载均衡设施高可用*

局限性
======

由于 `VRRP`_ 依赖 `ARP` 实现 `IP` 漂移，因此相关机器必须在同个网络内， **不能跨网段** 。

下一步
======

.. include:: /_fragments/next-step-to-wechat-mp.rst

.. include:: /_fragments/wechat-reward.rst

.. include:: /_fragments/disqus.rst

.. comments
    comment something out below

.. _VRRP: https://en.wikipedia.org/wiki/Virtual_Router_Redundancy_Protocol
