---
title: Windows下 Git，TortoiseGit及Git在IDE工具中的使用
author: abuzhi
date: 2017-05-18 18:32:00
categories: [Java, 开发工具]
tags: [版本控制, Git ,TortoiseGit , IDEA]
---

Git，TortoiseGit及Git在IDE工具中的使用

--------------------

## 一. Git基础介绍

* 要认识Git是做什么的，为什么要使用这个工具，就需要先阅读一下Git相关的入门知识，及与其他版本控制工具的异同，优劣。

* **注意：本文重点偏重于Git,TortoiseGit以及Git在IDEA等开发工具日常安装与使用上。其中对TortoiseGit和IDEA的使用会着重讲，但是对Git命令行操作，不做讲解，如想了解git bash相关命令，可点击下面两篇资料进行学习。**

* 对于Git的认识及入门知识，笔者先贴两篇资料，可以先补充一下这方面知识，或者做全面学习资料亦可：

> 	Git起步：<http://blog.jobbole.com/25775/> , <https://git-scm.com/book/zh/v2>

> 	Git教程：<https://www.liaoxuefeng.com/wiki/0013739516305929606dd18361248578c67b8067c8c017b000>




## 二. Git 与 TortoiseGit 安装

> Git 下载地址：<https://git-scm.com/downloads>

> TortoiseGit 下载地址：<https://tortoisegit.org/download> 

按自己的操作系统选择下载相应版本。

#### 1. 安装Git

先安装Git.xxx.exe，安装路径自己设置或者默认，其他选项都按默认来就行。其中需要说明中间几个选项的意思：

![Git安装过程-图1](/images/2017-09-12/git-1.jpg) 图-2.1


* 选项1 ：此选项不会先环境变量中加入git命令目录，只能使用Git Bash命令行操作。不建议新手安装此项。

* 选项2 ：默认选项，只会加入最小量的信息到环境变量中，可以使用Git Bash 和windows 命令行进行操作。建议安装此项。

* 选项3 ：此项会把unix tools 相关命令和Git Bash命令都加入到windows环境变量中，会污染系统，比如会覆盖掉windows自带的find等命令，不建议安装此项。

---

![Git安装过程-图2](/images/2017-09-12/git-2.jpg) 图-2.2

> CRLF是Carriage-Return Line-Feed的缩写，意思是回车换行，就是回车(CR, ASCII 13, \r) 换行(LF, ASCII 10, \n)
windows系统默认换行是两个字符\r\n，即CRLF；；unix和linux系统的换行符只有一个字符\n，即LF。
此安装选项就是设置windows和unix类系统下，换行符的转换。


* 选项1 ：git check out代码下来到windows系统时，转换LF为CRLF，即转为windows换行符。当commit 操作时，转换CRLF为LF，即转为unix换行符。此选项比较推荐用于跨平台的代码情况下，在windows机器上安装git时，比如git 远程仓库的所在系统为linux系统，本地开发环境为windows时。用此项。

* 选项2 ：check out时，不进行转换，只有commit时，才转换CRLF 为LF，此项用于跨平台时，linux和unix环境机器上安装git时，选用些项。

* 选项3 ：check out 和  commit时都不进行转换，这种只能用于相同平台下的开发。不建议用在跨平台的代码系统中。

------

![Git安装过程-图3](/images/2017-09-12/git-3.jpg) 图-2.3 

![Git安装过程-图3](/images/2017-09-12/git-4.jpg) 图-2.4

* 选项1 ：Git Bash命令在MinTTY中进行操作，也可以在windows命令行操作。（图-2.4即为安装好后，MinTTY，分为shell和gui两种界面。如果安装了TortoiseGit，这个基本就不需要用了，但是还是建议安装这个选项）

* 选项2 ：只使用windows命令行操作。

------

其他安装都用默认即可。

#### 2. 安装TortoiseGit

安装TortoiseGit比较简单，只需要按默认安装即可。安装路径可自己修改。


------


## 三. TortoiseGit 使用

着重讲述TortoiseGit 界面操作的使用。

### 1.创建仓库

创建仓库有两种方式。一种是在本地创建，然后push到远程地址。另一种是在远程创建好仓库后，clone仓库到本地。

##### 远程创建，pull到本地

* **在远程创建仓库后或者直接取已经创建好的远程仓库链接，git    clone到本地文件夹。**

以GitHub 为例：登录github帐号后，new repository,新建一个仓库。然后复制连接（比如：https://github.com/abuzhi/rdb-to-hbase.git），
在windows本地机上打开一个文件夹，空白处右击鼠标，选择git clone：（图-2.5）

<figure class="half">
    <img src="/images/2017-09-12/git-5.jpg">
    <img src="/images/2017-09-12/git-6.jpg">
</figure>

图-2.6默认点ok即可下载远程仓库代码到本地文件夹。下载完成后，本地文件夹如图：（图-2.7）

![Git安装过程-图7](/images/2017-09-12/git-7.jpg) 图-2.7

其中.git 文件夹默认是隐藏文件，需要开启windows显示隐藏文件功能才能查看。（笔者这里是开了的）

其他打了对勾的文件就是受git版本控制中的文件。

##### 本地创建，push到远程

* **在本地创建文件夹和仓库，git  push到远程库。**

有一种情形，比如我本地已经在开发某个项目了，开发了一部分后，我想把这个放入git，让其他人也下载下来开发。这时，需要首先在远程仓库先创建一个空的repostory。

依然以github为例：


在github登录自己帐号，创建一个空的仓库（图-2.8）

![Git安装过程-图8](/images/2017-09-12/git-8.jpg)


把本地开发目录加入git


比如我在本地的开发目录是这样的（图-2-9），我需要把这个test-git123目录push到远程上面建立的仓库中。

![Git安装过程-图9](/images/2017-09-12/git-9.jpg)

操作如下：先在文件夹空白处右击，选create repostory here...，弹框点ok，在此处建立仓库。这里建立的是本地仓库，除了不能push代码到远程，其他相关的git操作和版本控制都可以正常进行。

![Git安装过程-图10](/images/2017-09-12/git-10.jpg)

创建成功后，就有了.git文件夹，还有其他文件会多了问号标志。这个是什么含义？下面会分节来介绍相关图标的含义。现在暂时不管，你只需要知道这些带问号图标的文件，是属于被git识别，但是还未加入版本控制内的文件。如图-2.11

![Git安装过程-图11](/images/2017-09-12/git-11.jpg)

创建本地库成功后，把本文件夹内除去.git文件夹之外的文件，都加入到git版本控制中。
如图-2.12，选择要加入版本控制的文件，右击，按图选择加入。弹框后，如图-2.13，select all或者按自己需要选择要加入的文件。

![Git安装过程-图11](/images/2017-09-12/git-12.jpg)

![Git安装过程-图11](/images/2017-09-12/git-13.jpg)

上面文件加入完成后，再空白处右击，commit -> master （图-2.15）,弹出提交框后，要写入本次提交修改的注释，这个最好还是写明每次提交都有那些改进，方便开发和版本回滚（图-2.16）。

![Git安装过程-图15](/images/2017-09-12/git-15.jpg)
![Git安装过程-图15](/images/2017-09-12/git-16.jpg)


### 2.版本控制流程

