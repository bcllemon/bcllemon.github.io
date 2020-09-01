layout: post
title: "hexo+github page搭建博客"
date: "2015-12-04 23:07"
categories: 技术
---
# hexo安装
## 安装 nvm
``curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.29.0/install.sh | bash``
## 安装 Node
```
nvm install node && nvm alias default node
nvm use --delete-prefix v4.2.1 --silent
```
## 切换taobao 源
```
npm config set registry https://registry.npm.taobao.org
npm config set disturl https://npm.taobao.org/dist
```
## 安装 hexo
```
npm install -g hexo
```
# hexo 使用
## 初始化
```
mkdir hexo
cd hexo
hexo init
npm install
```
## 生成文件
```
hexo generate
```
或
```
hexo g
```
## 清理文件
```
hexo clean
```
## 本地预览
```
hexo server
```
或
```
hexo s
```
