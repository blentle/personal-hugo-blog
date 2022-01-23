---
title: "浅谈机械硬盘(hdd)和固态硬盘(ssd)内部结构及工作过程" # Title of the blog post.
date: 2018-04-17T09:57:51+08:00 # Date of post creation.
description: "机械硬盘与固态硬盘内部结构与工作过程." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: true # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
featureImage: "/images/disk-icon-01.jfif" # Sets featured image on blog post.
thumbnail: "/images/disk-icon-03.gif" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/disk-icon-02.jpg" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - 操作系统
tags:
  - 操作系统
  - IO
  - 磁盘
# comment: false # Disable comment if false.
---

## 机械硬盘(HDD)
### 构造
机械硬盘一般是由多个盘片构成,每个盘片有两面,表面附着磁性记录材料, 两面都可以存储数据.盘片被固定在中央可以旋转的主轴上,它使得盘片以通常5400 ~ 15000转
每分钟(不同类型的硬盘转速不同)的固定旋转速率旋转,如图1.盘面是由一组一组同心圆的磁道构成,每个磁道被划分成多个扇区(sector),每个扇区存储的就是实际的数据且固定512个字节.扇区之间是有间隙的,
间隙里不存储数据,但通常存储格式化的标识位,如图2.主轴的马达会在磁盘驱动器工作时, 恒速带动盘片旋转.读或写数据时,磁头必须定位到磁道上相应扇区的开始位置.磁盘臂摆动使得磁头定位到某一个磁道
上是一个机械动作, 相当耗时, 这个过程需要的时间称为寻道时间.磁道定位后, 磁盘控制器就一直等待,直到数据访问的扇区旋转到磁头处.扇区的开始处旋转到磁头处所需要的时间称为旋转延时.显然这也是一个
机械动作,自然也是也相当耗时. 一旦磁头定位成功,读写操作就随着扇区在磁头下的移动开始了.这个过程就是数据的传输过程,过程耗时也被称为传输时间.
![图 1](/images/disk/disk-001.jpg)
一个盘面一个磁头,磁头从上到下,编号从0开始,依次递增. 磁道没有编号,通常使用柱面来替代磁道. 所谓柱面(cylinders)就是多个盘面一组同心圆垂直方向组成的圆柱的外表面, 这样一看是不是就是磁道?
柱面从外到里也是编号从0开始, 依次递增. 同样扇区也是有编号的, 同一个磁道, 扇区编号也是从0开始绕圆环递增,如图7

![图 7](/images/disk/disk-007.jpg)
上面说了扇区之间存在间隙.每个扇区的格式, 如图8
![图 8](/images/disk/disk-008.png)


### 容量和耗时
#### 容量
早期的磁盘每个磁道的扇区数都是相同的, 所以整个磁盘容量会这么计算:
```shell script
    总容量(字节数) = 盘面数(磁头数) x 柱面数(磁道数) x 扇区数 x 每个扇区存储的字节数(512)
```
但每个磁道扇区数如果都相同,从上面的图7中就可以看出,外面的磁道(半径大的)如果与里面的磁道拥有相同的扇区,会造成资源的极大浪费.所以现代磁盘会被划分成多个环带,
外层的环带比内层的环带拥有更多的扇区.如图9的磁盘具有两个环带,外层环带每个磁道有32个扇区,内层环带每个磁道16个扇区. 一个现代化具体的磁盘通常有16个环带,从内层到
外层,每个环带扇区数增加大约4%. 为了让操作系统更好的调用硬件接口,不需要复杂的计算.现代磁盘都会隐藏每个磁道有多少个扇区的细节,转而提供给操作系统一个虚拟的几何规格.
让软件在调用的时候,仿佛还是存在着固定的柱面,固定的扇区数.就如同图7所示
![图 9](/images/disk/disk-009.png)
打开一台服务器执行如下Linux命令校验一下(可能需要sudo)
```shell script
    fdisk -l
```
输出结果如下:
```shell script
    Disk /dev/sda: 1999.8 GB, 1999844147200 bytes
    255 heads, 63 sectors/track, 243133 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x0005f4cf
    
       Device Boot      Start         End      Blocks   Id  System
    /dev/sda1   *           1          26      204800   83  Linux
    Partition 1 does not end on cylinder boundary.
    /dev/sda2              26      243134  1952766976   8e  Linux LVM
    
    Disk /dev/mapper/vg00-swap: 16.8 GB, 16777216000 bytes
    255 heads, 63 sectors/track, 2039 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
    
    Disk /dev/mapper/vg00-root: 193.3 GB, 193273528320 bytes
    255 heads, 63 sectors/track, 23497 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
    
    
    Disk /dev/mapper/vg00-data: 1759.2 GB, 1759221121024 bytes
    255 heads, 63 sectors/track, 213879 cylinders
    Units = cylinders of 16065 * 512 = 8225280 bytes
    Sector size (logical/physical): 512 bytes / 512 bytes
    I/O size (minimum/optimal): 512 bytes / 512 bytes
    Disk identifier: 0x00000000
```

```shell script
    (255个磁头 x 243133个柱面 x 63个扇区(假象) x 512个字节) / (1024 x 1024 x 1024) = 1862.49G 
```
> 与上面的 1999.8 GB 不符, 那是因为在数据通信的世界里 1Gb/s 意味着 1 000 000 000位/秒, 厂商是按照这个 1000来计算而不是计算机世界里的 1024.

#### 耗时
1. 寻道时间
磁盘臂移动到指定磁道所需要的时间,文章开头已经提到过.实际上这个时间是很难降低的. 寻道时间包括2部分:初始启动时间和 当磁盘臂提速完成后,越过其必须跨越的磁道所需的时间.
非常遗憾的是,跨越磁道的时间并不是关于磁道数量的线性函数(后面在调度策略里说),其中还包含一个稳定时间(从磁头定位到目标磁道之间的时间).所以磁盘的改进就是部件本身变得更小更轻.过去磁盘的直径
可以有36cm,而现在常见的都不到5cm,甚至更小.盘片直径的减少直接导致磁盘臂需要移动的距离减少.当下硬盘的普遍平均寻道时间都在 10ms以内了.

2. 旋转延时
上面也说了, 磁盘的寻址区域旋转到读或写磁头可以访问的位置所需要的时间,最坏的情况就是数据区刚在磁头定位到对应的磁道上时旋转过去,就不得不花费旋转一整圈的时间(这个是有优化的,会在后面的格式化里说).
盘片的旋转速度上面也提到,按最快 15000转/分钟的来换算一下,旋转一周大约是4ms.

3. 传输时间
传输时间取决于盘片的转速,若 T 表示传输时间, totalBytes表示要传输的字节数(读或者写), N 表示一个磁道的字节数, r表示转速,则
```shell script
    T = totalBytes/(r x N)
```

4. 所以对磁盘数据的读写(无论顺序读写,还是随机读写)所花费的总时间只与上述三个因素有关吗？不不不, 除了这些因素外, 还有一些排队延时.啥意思, 当进程发起一个I/O请求时,它必须先在队列中等待磁盘设备变成可用,然后该磁盘设备
被分配给该进程.如果该磁盘设备与其他磁盘驱动器共用一个或一组I/O通道,则还需要等待该通道变为可用.此后才开始执行寻道之后的步骤. 甚至更有一些高端的服务器中还使用了一种叫做RPS(Rotational Positional Sensing)技术,中文
翻译成旋转定位感知技术.工作流程大致如下: 
a. 寻道命令发出后, 通道被释放以处理其他I/O操作;
b. 寻道完成后,磁盘设备确定数据旋转到磁头下面的时间;
c. 当该扇区接近磁头时,磁盘设备再试图重新建立到主机的通信路径, 如果此时控制单元或通道正忙着处理另一个I/O, 则重新连接的尝试失败,磁盘设备必须再旋转一整圈,然后再次尝试重新连接.
![图 10](/images/disk/disk-010.png)



### LBA
+ 事实上当操作系统在访问磁盘的时候不会一个字节一个字节的读或写, 也不会一个扇区一个扇区的读或写, 而是以块(block)的方式进行,因此块才是文件存储的最小单位.一个块通常是4k的大小,也就是8个连续扇区(一个扇区512个字节).这样减少了磁盘臂摆动的次数(也就是寻道次数).
+ 现代磁盘构造复杂,有多个盘面,盘面上磁道扇区的多少与环带有关. 为了对操作系统隐藏不必要的复杂计算, 磁盘都会封装一个小的固件,这个固件就是磁盘控制器,它维护着逻辑块号与物理磁盘扇区之间的映射关系.
+ 当操作系统想要执行一个I/O操作时,如读取一个扇区的数据到主内存,它会先发送一个命令给磁盘控制器,让其读到某个逻辑块号.控制器上的固件设备会执行一个快速表查询,将其逻辑块号翻译成一个(盘面, 磁道, 扇区)的三元组,这个三元组唯一地标识了对应的物理扇区.控制器上的硬件
会解释这个三元组,将磁头移动到适当的柱面(磁道), 等待对应的扇区移动到磁头下.将磁头感知到的位放到控制器上的一个小缓冲区中,然后将他们复制到主存中. 这就是逻辑块寻址(Logic Block Addressing), 不管磁盘的几何规格如何,就像上面说的,让操作系统认为每个磁道上(柱面)
的扇区都是一样的(从上面的命令fdisk执行结果也能看出).


### 格式化
+ 磁盘控制器必须对磁盘进行格式化,才能在磁盘上存储数据.格式化包括: 1. 用标识扇区的信息填写扇区之间的间隙, 标识出表面有故障的的柱面,且不再使用它; 2. 在某些区域中预留出一组柱面作为备用, 如果某一片区域部分在使用过程中坏掉了, 就可以使用这些备用的区域. 这也是造成格式化后的
容量比最大容量要小的原因.
+ 磁盘在使用之前, 每个盘片必须经受由软件完成的低级格式化.格式包含一系列同心圆磁道, 每个磁道包含若扇区, 扇区间存在小的间隙, 扇区如上面的图8所示, 前导码以一定的位模式开始, 位模式使得磁头等硬件得以识别一个扇区的开始.前导码还包含柱面(磁道)和扇区号.
数据部分的大小是由低级格式化程序决定的,通常磁盘使用512个字节的扇区.ECC区域包含一些冗余信息,可以用来恢复读错误.该区域的大小取决于厂商为了更高的可靠性愿意放弃多少磁盘空间以及磁盘控制器能够处理的ECC编码有多复杂. 
+ 在低级格式化时,每个磁道的第0号扇区的位置与前一个磁道存在偏移,就如同跑道上的起始位置一样,每一圈的起始位置并不是相同的,如图11. 这个偏移称为柱面斜进(cylinder skew).这样做是为了改进性能,目的是让磁盘在一次连续的操作中读取多个磁道不丢失数据.

![图 11 - 柱面斜进示意图](/images/disk/disk-011.jpg)

举个例子,假如一个读请求需要最里面的磁道上从第0号扇区开始的连续18个扇区,磁盘旋转一周可以读取前16个扇区. 假设这里最内圈磁道只有16个扇区(上面提到,现代磁盘为了节约资源, 磁道从外到内,扇区数量逐渐减少),但是为了得到第17个扇区的数据,还需要一次寻道操作以便
磁头向外移动一个磁道.到磁头移动到了那个磁道时,第0号扇区已经转过了磁头,就不得不再旋转一周才能等到它再次经过磁头.此时斜面柱进就解决了这个问题.

+ 在低级格式化完成后,要对磁盘进行分区. 逻辑上,每个分区就是一个独立的磁盘.分区对于多个操作系统共存是必须的. 在X86和多数其他计算机上,0号扇区包含主引导记录(Master Boot Record),也就是常说的MBR, 它包含某些引导代码及其处在扇区末尾的分区表.为了能够从硬盘上引导,在分区表中必须有
一个分区被标记为活动的.
+ 磁盘能够使用的最后一步是对每一个分区执行一次高级格式化.这一操作要设置一个引导块、空闲存储管理(空闲列表或位图)、根目录和一个空文件系统.这个操作中还要将一个代码设置在分区表中,以标明在分区中使用的是哪个文件系统, 许多操作系统支持多个兼容的文件系统. 这是系统就可以引导了.
+ 当电源打开时,BIOS最先运行, 它读入主引导记录并跳转到主引导记录.然后这一引导程序检查哪个分区是活动的.引导扇区包含一个小的程序,一般是会装入一个较大的引导程序装载器,该引导程序装载器将搜索文件系统找到操作系统内核, 然后把内核装入内存并执行,这样系统就启动了.


### 调度策略 

## 固态硬盘(SSD)