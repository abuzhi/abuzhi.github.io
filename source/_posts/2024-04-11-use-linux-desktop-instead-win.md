---
title: 用Linux桌面版替带windows办公
author: abuzhi
date: 2024-04-11 18:32:00
categories: [Linux]
tags: [Linux,Mint,Ubuntu]
---

用Linux桌面版替带windows办公

[TOC]

# 0. Linux桌面版替换Windows办公的可行性

鉴于日常工作中的一些特殊应用场景，需要在linux中进行部分开发工作，每次win和linux切换开发有点不方便，不如直接全转为linux内进行开发。
办公windows转为linux系统的一些前提条件：

* 1. 对office工具日常需求不高，只需要能打开excel查看，简单函数即可，word，ppt能打开编辑，pdf可打开。
* 2. java、python开发环境及各种开发工具，都在linux中有完好支持
* 3. 公司内部通讯软件有linux版本
* 4. 日常软件比如笔记、云盘、vpn、输入法、浏览器等都有合适的linux版本

如果日常中对以下要求比较高的话，最好就不要转linux了：

* 1. 对微软 office等办公套件重度使用
* 2. steam，epic等打游戏重度要求
* 3. 懒，懒得动，懒得折腾，想省心的。（可以直接mac，最好的linux桌面版）

**重要重要重要：先确定个人需求及目标，能够满足并且接受的话，就可以进行下面的操作了**

我个人选的是Linux Mint。用Ubuntu也可以。

按下面顺序开始

# 1. 整理软件需求

## 1.1 日常软件

平时用到的非开发类软件，日常交流和基础办公使用

### 通讯

- 微信，后续可能会退出linux版本，当前只能用wine版，但是很难用，勉强接受。
- 公司通讯软件有专用linux版本
- qq，https://im.qq.com/linuxqq/index.shtml 有linux版本

### 笔记

* 有道：https://note.youdao.com/note-download/  有linux版本
* wiz笔记：https://www.wiz.cn/zh-cn/download.html   linux版本

### 工具

* 压缩解压：7z，https://www.7-zip.org/download.html ，linux版本，默认zip工具也可以
* 电子书阅读器：https://calibre-ebook.com/download  ，linux版本
* 天翼云盘：https://cloud.189.cn/web/static/download-client/index.html  ，linux
* VMware：https://www.vmware.com/products/workstation-pro/workstation-pro-evaluation.html   ，有对应linux
* 搜狗输入法：https://shurufa.sogou.com/linux
* 截图工具：https://zh.snipaste.com/

### 办公

office套件：
- wps替换：https://linux.wps.cn/，有linux，除了基础组件，还有pdf等功能，缺点是有广告
- 开源的LibreOffice：https://www.libreoffice.org/download/download-libreoffice/
- pdf阅读器：Foxit PDF Reader 福昕

### 浏览器

chrome，firefox在各软件包管理内有，或者直接官网下载最新的，都有linux

### 安全

个人用的卡巴斯基，有对应linux

但是公司内部的需要安装几个自研的软件，目前看，没有linux版本，eset好像也没有，所以这块替代不了。

### 音乐视频

- 视频vlc：https://www.videolan.org/vlc/#download   ，linux
- 音乐foobar2000的linux版本 ：https://deadbeef.sourceforge.io/download.html
- qq音乐：https://y.qq.com/download/download.html   ，居然有linux

### 小工具软件

* 桌面日历：https://www.rainlendar.net/
* 农历：ubuntu lunar date https://github.com/yetist/lunar-date

## 1.2 开发软件

工作中用到的开发相关软件和工具

### 基础环境

* jdk，maven，python，git等开发工具
* git：https://git-scm.com/download/linux，
* jdk：https://www.oracle.com/hk/java/technologies/javase/jdk11-archive-downloads.html
* maven：https://maven.apache.org/download.cgi
* python：https://www.python.org/downloads/
* nodejs：https://nodejs.org/en/download

### IDE

* jetbrains全家桶，都有linux版本，包含：IDEA，Webstorm，Pycharm，Datagrid四个基本用
* Visual Studio Code：https://code.visualstudio.com/download

### 工具

* drawio：https://github.com/jgraph/drawio-desktop/releases/tag/v24.1.0
* docker：https://docs.docker.com/desktop/install/linux-install/
* redis客户端：https://redis.com/redis-enterprise/redis-insight/
* ssh端：直接用自带的terminal
* mysql：官网有
* db工具：https://dbeaver.io/download/  ，支持mysql

