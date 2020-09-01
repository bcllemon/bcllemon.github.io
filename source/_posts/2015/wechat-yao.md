layout: post
title: "微信公众号摇一摇开发"
date: "2015-12-03 16:37"
categories: 技术
---
# 前言
看了[那么多“扫一扫”，不妨“摇一摇”——微信“摇一摇周边”功能开发实录](http://yalishizhude.github.io/2015/11/17/shake-nearby/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)的文章，玩了一次开发。结果发现有些微信已经提供 web 方式操作，还有些逻辑说的不甚明了，所以记录下过程。
# 先决条件
订阅号或者服务号完成认证
# 开通摇一摇周边功能。
开通方式：添加功能插件->摇一摇周边->使用公众号登录->申请开通
填写相关信息及证明文件后，等待审核。
![](http://7xnhcl.com1.z0.glb.clouddn.com/yao-1.png)
![](http://7xnhcl.com1.z0.glb.clouddn.com/yao-2.png)
![](http://7xnhcl.com1.z0.glb.clouddn.com/yao-3.png)
![](http://7xnhcl.com1.z0.glb.clouddn.com/yao-4.png)

# iBeacon设备
## 购买设备iBeacon设备。
这个可以从淘宝上购买，也可以通过认证后，从[微信官方](https://zb.weixin.qq.com/nearby/html/guide/buyDeviceGuide.html)渠道进行购买。

## 新增设备
从周边后台添加新增设备
![](http://7xnhcl.com1.z0.glb.clouddn.com/yao-5.png)
由于我是自行购买设备，所以选择了方式2。之后填写需要的数量和理由后，完成申请。
![](http://7xnhcl.com1.z0.glb.clouddn.com/yao-6.png)

申请之后的设备在未激活设备处可以看到。
![](http://7xnhcl.com1.z0.glb.clouddn.com/yao-7.png)
查看设备信息，可以得到对应的UUID、Major、Minor信息
![](http://7xnhcl.com1.z0.glb.clouddn.com/yao-8.png)

## 配置设备
使用厂家提供的工具将上面获得的UUID、Major、Minor写入设备

## 激活设备
保证设备电源处于开启状态，打开手机蓝牙，打开微信摇一摇功能，选择周边，然后摇，能够发现设备，点击后，可完成激活。完成后，可在[周边后台](https://zb.weixin.qq.com/nearby/html/device/manage.html)看到设备状态。具体可以参考[设备配置指引](https://zb.weixin.qq.com/nearby/instruction.xhtml?section=4) [如何激活设备？](https://zb.weixin.qq.com/nearby/instruction.xhtml?section=5)
# 页面配置
点击页面管理、新建页面、选择配置设备
