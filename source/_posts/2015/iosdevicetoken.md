layout: post
title: "iOS的DeviceToken随生产环境和开发环境变化"
date: "2015-10-16 17:44"
categories: 技术
---
当我用XCode直接运行到手机上的时候，Device token是以3开头的，而当我打包上传到fir.im，再下载安装的时候，Device token就变成以5开头了

其实这是生产环境和开发环境的问题，在这两个环境下Device token是不同的

如果你手机中的App是通过XCode直接安装的话，那么你的App就属于开发环境，想要推送成功就需要创建开发环境的证书；
如果你的App是打包成ipa文件安装的（<font color="red">不管是正式上线还是自己测试</font>），那么就是生产环境，需要创建生产环境证书，这两个环境的Device token是不同的
