---
title: "WakeOnLan"
date: 2022-09-29T15:50:34+08:00
draft: False
---

# Wake On Lan

> 出于CNSS recruit一道四千分的题，华华翻出了尘封的WOL配置记忆......

## 前言

其实wol华华在很早之前就配置过了，实现校园网内可以随时连回寝室的台式机作为服务器和生产环境使用，不过在购置了新的笔记本之后就逐渐冷落台式机了，基本只用来挂下载器和存储定期的手机、笔记本备份。不过配置过一次了就上手非常快，这次还尝试了通过局域网内的客户端主动连接公网服务器，进行指令转发的方式实现了公网唤醒（但是我又连不上，有锤锤用）（但是有整整两千分欸）

## 前期配置

* 确认主板和网卡支持wol，经google确实可以
* 查找网卡ip及mac，通过 $ ipconfig /all 完成

## 杂七杂八的设置

### 驱动层

右键状态栏网络图标 >> Open Network & Internet settings >> Change adapter option >> 选网卡 >> Properties >> 选中Client for Microsoft Networks 然后点上方的 Configure >> Advanced 栏 >> 倒数第三个 Wake on Magic Packet 设置为 Enable


然后还要记得在Power Management里允许此设备唤醒计算机

### 硬件设定

重启，进入BIOS；

在APM里启用“由PCI-E”设备唤醒，与关闭“Erp支持”（实际上就是所谓的S4休眠，感知不强）

### 系统电源选项

关闭快速启动。

## 公网唤醒

由于华华在的学校有人手一个的公网ip，想试着能不能直接进行唤醒。可惜用socket扫过所有的端口后，一个都没开，只能另寻他路。

此处更改思路采用华的腾讯云作为中转站，外网设备可以连接腾讯云，而内网再开一台从机，连接到腾讯云接受启动信号，然后内网发送幻数据包唤醒台式机。

## 手搓唤醒程序

这里比较暴力，直接用udp对两个可能的wol端口（7和9）都发十次，然后总能叫醒。

对于内网连接公网的部分也挺暴力，从机上的脚本不断尝试connect服务器，但服务器没开listen的时候会报错，于是继续尝试，如果没报错就说明该唤醒台式机了。

服务器上就写一个脚本，要wol的时候就打开socket并listen某端口即可。

## 参考资料
[主板](https://rog.asus.com.cn/motherboards/rog-crosshair/rog-crosshair-viii-extreme-model/)

[零碎设置](https://blog.csdn.net/weixin_44607961/article/details/89136069)

[幻数据包](https://www.cnblogs.com/zhanggaoxing/p/9657545.html)

[python端口扫描](https://cloud.tencent.com/developer/article/1773261)
