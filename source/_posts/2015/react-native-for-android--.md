layout: post
title: "React Native for Android 环境搭建-坑"
date: "2015-10-21 17:31"
categories: 技术
---
# 准备工作
- 操作系统：OSX

# 基本环境
## Homebrew
### 新安装
```
ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
```
### 已安装-<font color="red">务必更新</font>
```
cd /usr/loacl
git fetch origin
git reset --hard origin/master
brew update
```
## Nodejs
### 安装 nvm
```
brew install curl  # 确保安装了 curl
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.29.0/install.sh | bash
```
### 安装 Nodejs
此处需要安装4.0以上版本
```
nvm install node && nvm alias default node
```
### NPM 国内源
```
npm config set registry https://registry.npm.taobao.org
npm config set disturl https://npm.taobao.org/dist
```
## watchman
[watchman](https://facebook.github.io/watchman/docs/install.html) 是 Facebook 的一个用于监控文件变更并触发指定操作的工具：
```
brew install watchman
```
## flow
[Flow](http://flowtype.org/) 是一个 JavaScript 的静态类型检查器，建议安装它，以方便找出代码中可能存在的类型错误：

```
brew install flow
```
## Android SDK
### SDK:
- Android SDK Tools
- Android SDK Platform-tools
- Android SDK Build-tools
- Android SDK Platform
- Android Support Repository
### 模拟器
- API 19以上
### 环境变量
- ANDROID_SDK
- ANDROID_NDK

详细介绍可以参考 [这篇文章](http://wiki.jikexueyuan.com/project/react-native/DevelopmentSetupAndroid.html) 。
## React Native
```
npm install -g react-native-cli
```
# 使用步骤
## 新建工程
```
react-native init demoProject
```
## 运行工程
```
react-native run-android
```
## 常见问题
### 启动速度慢
Android App使用 gradle 打包，在第一次启动时，会下载 gradle 和相应的依赖包，所以会造成启动速度缓慢。可以自行修改 app 中 /android/gradle.properties的配置，使用本地已经配置好的环境优化启动速度。
### 端口占用
如果 Running Packager 提示 “Packager can’t listen on port 8081” ，说明 8081 端口被占用，可以检查是什么程序占用了这个端口并杀掉它：
```
$ sudo lsof -n -i4TCP:8081 | grep LISTEN
$ kill -9 <进程id>
```
### 加载失败
![加载失败](http://7xnhcl.com1.z0.glb.clouddn.com/post-react-android-1.png)
你还需要进行如下设置：

- 更新 brew 和 watchman ：brew update && brew upgrade watchman；参考 brew 安装中的说明
- 按下菜单按钮呼出菜单
![菜单](http://7xnhcl.com1.z0.glb.clouddn.com/post-react-android-2.png)
然后点击【Dev Settings】菜单项进入应用的选项界面，再点击【Debug server host for device】选项，填入你的Mac主机的 ip，然后重启 APP ；
![Debug server host for device](http://7xnhcl.com1.z0.glb.clouddn.com/post-react-android-3.png)
其他选项，按需勾选。
Auto reload on JS Change - 自动刷新界面

# Example App
## 下载
```
git clone https://github.com/facebook/react-native.git
git checkout 0.13-stable
```
<font color="red">注意选择稳定分支，不要使用 master 分支</font>
## 运行
Exmaple App 中，Movies 和 UIExplorer 两个是 IOS 和 Android 都可以的，剩下俩个都是只有 IOS 版本的。这两个 APP 和上面说的通过 `react-native init`方式创建的不同，需要使用不同的方式来启动。
[详情](https://github.com/facebook/react-native/tree/master/ReactAndroid#prerequisites)
### 安装依赖
- react-native依赖
```
cd react-native
npm install
```
- Android
需要 SDK、Build-tools、NDK、Emulator

### 编译基础
```
cd react-native
./gradlew :ReactAndroid:assembleDebug
```
### 编译运行
编译过程中需要下载依赖包，所以要耐心等待。
- UIExplorer
```
cd react-native
./gradlew :Examples:UIExplorer:android:app:installDebug
./package./packager/packager.sh
```

- Movies
```
cd react-native
./gradlew :Examples:Movies:android:app:installDebug
./package./packager/packager.sh
```
