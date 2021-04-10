---
title: "Linux内存管理-虚拟内存篇" # Title of the blog post.
date: 2018-03-10T14:09:33+08:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
featureImage: "/images/building.jpg" # Sets featured image on blog post.
thumbnail: "/images/dollar.jpg" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/dollar.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Linux
tags:
  - kernel
  - Linx内核
# comment: false # Disable comment if false.
---

## 介绍

* 虚拟内存是现代所有的操作系统支持的一个核心功能。虽然内存的容量增长快速,但
是软件的大小增长更快。这一发展的最终结果就是需要运行的程序会有可能大到内存
无法容纳，而且必然需要系统能够支持多个程序的同时运行，即使内存可以满足其中
单一程序的需要，总体来看可能仍然会超出内存的大小。早期出现了覆盖技术，思想
是把程序分割成许多片段,每个片段就是一个覆盖序列.在程序开始执行时,将覆盖管理
模块载入内存,该覆盖管理模块立即装入并运行序列0.执行完,序列0就通知覆盖管理模
块载入序列1,如果有多余的空间就占用序列0的上方位置,没有则占用序列0.覆盖块存
放在磁盘中,在需要时，由操作系统换入换出.这样看起来好像没有太大的问题,因为是
由操作系统来进行换入换出.但是应用程序需要被分割成多个片段.把一个大的程序分
割成小的、模块化的片段是非常耗时和枯燥的,并且容易出错.而且这个事要交给程序
员去做,那估计要崩溃了.没过多久聪明的人类就找到了解决版本,提出了虚拟内存的
概念.

* 虚拟内存的基本思想是:每个程序都拥有自己的地址空间,这个空间被分成多个块,
每个块叫做一个页或一个页面.每一页有连续的地址范围.这些页被映射到物理内存,
但并不是所有的页必须在内存中才能运行程序.当程序引用到一部分在物理内存中
的地址空间时,由硬件立刻执行必要的映射.当程序引用到一部分不在物理内存中的
地址空间时,再由操作系统负责将缺失的部分装入物理内存并重新执行失败的命令,
这个过程叫缺页中断处理处理. 虚拟内存很适合在多道程序设计系统中使用,许多
程序的片段同时保存在内存中. 当一个程序等待它的一部分读入内存时,可以把CPU
交给另一个程序使用. 这里解释一下什么是多道程序设计系统,所谓的多道程序设
计系统指的是允许多个程序同时进入一个计算机系统的主存储器(即内存)并启动进
行计算的方法.也就是说,计算机内存中可以同时存放多个(两个或以上相互独立的)
正在运行的程序,它们都处于开始和结束之间.从宏观上看是并行的,多道程序都处
于运行中,并且都没有运行结束;从微观上看是串行的,各道程序轮流使用CPU,交替
执行.引入多道程序设计技术的根本目的是为了提高CPU的利用率,充分发挥计算机
系统部件的并行性,现代计算机系统都采用了多道程序设计技术.

* 程序在访问一个内存地址指向的内存时,CPU不是直接把这个地址送到内存总线
上,而是被送到一个叫内存管理单元的硬件上(业界也叫MMU,是Memory Management
Unit的简称),然后由这个硬件把这个内存地址映射到实际的物理内存地址上.程序
操作的这个地址称为虚拟内存地址.MMU作为CPU芯片的一部分,其实是单独的一个
芯片.MMU把虚拟内存地址映射成物理内存地址再送到总线的过程如下图:
![mmu-convert01-1.jpg](https://upload-images.jianshu.io/upload_images/12069275-e116d1f9289cede2.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



## 分页技术

* 上面提到了虚拟内存地址,实际上程序所访问的这些内存地址,构成了一个虚拟
地址空间,正如上面提到的,虚拟地址空间按照固定的大小被划分成若干单元,每
个单元叫做一个页或者一个页面.同时实际的物理内存也被划分成若干单元,但是
物理内存对应的这个单元被业界称作Page Frame(页帧或页框).虚拟内存的页面
和物理内存的页框通常是一样大的,比如都是4KB或者2MB,实际的页面大小可能从
512 byte到1GB.如下图,按每页4KB划分一个32KB的物理内存空间和一个64KB的虚
拟地址空间,可得到8个页框和16个虚拟页面.内存和磁盘之间的交换总是以整个
页面为单元进行的.
![mmu-convert02.jpg](https://upload-images.jianshu.io/upload_images/12069275-b9b89b1c60b2803a.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
结合这两个图,下面简单的解释一下:
当程序试图访问地址0时,如执行下面的指令:
```assembly
MOV REG, 0
```
> 这句伪代码的意思是把地址0的数据送入寄存器中,REG是register的简称,
表示寄存器.但并不是真实的寄存器的名字,这里是伪代码.真实的寄存器如
AX,BX,CX,SS,SP等.

将虚拟地址0送到MMU,MMU看到虚拟地址0落在(0～4095)这个页面上,而这个页面
被映射到2(8K～12K)这个页框上,所以MMU把地址变成8192,并把地址8192送到总
线上.内存对MMU一无所知.它只看到一个对地址8192的读或写请求并执行它.
同理:
```assembly
MOV REG, 8192
```
被转换成
```assembly
MOV REG, 24576
```
页面8192被映射到第6个页框上即 1024 x 24 = 24576
上面的图中因为只有8个物理页框的内存,所以只有8个页面被映射到了物理内存
,正如图上有页框号的页面,其他的页面上都是叉号,表示没有被映射到物理内存
.在实际的硬件中,是用一个标志位("在/不在")来记录页面是否被映射到物理内
存.如0表示当前页没有被映射,1表示被映射.
再如,当程序访问了一个没有被映射的页面:
```assembly
MOV REG, 24576
```
MMU注意到该页面没有被映射,于是CPU通知操作系统,出现了上面提到的缺页错
误或缺页异常.操作系统找到一个很少使用的页框,且把它的内容写入磁盘.然
后把需要访问的这个页面的内容读到刚才的页框里,修改一下映射关系,然后重
新执行刚才的异常指令.这个过程就叫缺页中断处理.