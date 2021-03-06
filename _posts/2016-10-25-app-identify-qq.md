---
layout: post
title: 手机QQ号识别
excerpt: "通过抓包分析手机QQ号的提取方法"
categories: [app-identify]
comments: true
---

## 报文首部特征

* 报文类型：TCP报文

* 报文IP： 无固定目的IP

* 报文端口：目前抓包观察服务器端口号为80，443，8080，14000，16666等

## 报文数据特征

### 特征1

* TCP数据区开始，偏移4个字节，取第5个字节开始的3字节长度必须为**0x00,0x00,0x00**，认为可能是手机QQ协议报文；

* 再偏移5个字节，取第13个字节开始的1个字节，并转换成十进制数字，记为偏移量X；

* 从TCP数据区起始位置开始，偏移12+X字节，取偏移后开始的2个字节，转换为十进制数字，并减4，该数字为QQ号长度记为Y；

* 接着取长度字段后续的Y个字节，为QQ号；

以下图为例：  
第13字节为0x44，也就是十进制68，加上12为80；  
图中从TCP数据区开始后的80字节（0x50）的位置为0x0036 + 0x0050 = 0x0086位置；  
读取2个字节为0x00 0x0d，转换成十进制为13，减去4为9，表示接下来的QQ号长度为9；  
读取接下来的9个数字就为QQ号  

![qq1](/img/qq.png) 

### 特征2

刚好巧合第13个字节是QQ长度字段的0x00，因此刚好从TCP数据区的第13个字节开始取QQ号长度，能取到正确的结果。

好像是巧合？

![qq2](/img/qq2.png) 

### 特征3

遇到如下报文特征时，如果按照特征1或者特征2的采集方式，可能就导致采到的QQ号为0，那真实的QQ号其实在该报文下半段

* 取TCP数据区的前4个字节，转换为十进制数字，记为长度或偏移量X；

* 从TCP数据区开始偏移X个字节进行查找，并按照特征1的提取方法对QQ号进行提取；

* QQ号可能需要在报文中多次循环偏移长度查找才能找到，并且一个报文中可能携带多个QQ号；

以下图为例：  
前4字节为0x00 0x00 0x01 0x2b，TCP首部位置0x0036 + 0x012b = 0x0161，就是下一个匹配区域的起始位置

![qq3](/img/qq3.png) 

Author: chenxiawei@gmail.com  
