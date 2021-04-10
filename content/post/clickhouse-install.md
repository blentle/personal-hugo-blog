---
title: "ClickHouse源码安装踩坑笔记" # Title of the blog post.
date: 2019-05-30T11:111:24+08:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
featureImage: "/images/clickhouse-install-002.jpeg" # Sets featured image on blog post.
thumbnail: "/images/clickhouse-install-001.gif" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/clickhouse-install-003.jpeg" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - clickhouse
tags:
  - clickhouse
# comment: false # Disable comment if false.
---

###  前情概要

由于工作需要用到clickhouse, 这里暂不介绍概念，应用场景，谷歌，百度一大把. 将安装过程踩下的坑记录下来备用

### ClickHouse源码 git clone安装（直接下载源码包安装失败）

> 源码安装一定要用git克隆下来，不要下载github上已经打好的包，因为有些依赖的子模块时时刻刻在更新.要不然编译各种报错.   一定要克隆，一定要克隆，一定要克隆

1. github上找到一个最新的稳定版本，一般以 -stable结尾的

2. 创建一个clickhouse目录如 mkdir /data/clickhouse-19.7.3.9-stable,这里我带上版本号，方便以后好升级

3. clone

- cd /data/clickhouse-19.7.3.9-stable

- git clone -b 'v19.7.3.9-stable' --recursive https://github.com/yandex/ClickHouse.git

这里递归下载依赖可能需要一点时间， 200kb/s的速度，我这里用了将近半个小时

4. 升级子模块

cd /data/clickhouse-19.7.3.9-stable/ClickHouse

git submodule update --init --recursive

5. 创建编译目录

cd /data/clickhouse-19.7.3.9-stable/ClickHouse

mkdir build

cd build

cmake3 ..     （这一步生成可执行编译的makefile文件,如果系统没有cmake3 执行: sudo yum -y install cmake3 ）

这一步会报缺少很多依赖错误，报什么依赖错误，安装什么依赖即可,如果嫌麻烦，可以一次性安装所有依赖,按官方文档来：<https://clickhouse.yandex/docs/en/development/build/>，注意把debian系列的 apt-get命令换成 yum

```bash
##安裝gcc,g++ 7以上的版本,clickhouse 用了很多 C++ 11的新特性
sudo yum -y install devtoolset-7-gcc
sudo yum -y install devtoolset-7-gcc-c++
source /opt/rh/devtoolset-7/enable
#安装完确认一下版本号
gcc --version
```

![avatar](/images/clickhouse/clickhouse-dep-01.png)

```bash
#安装jemalloc （clickhouse 没有用glibc的内存分配器）
sudo yum -y install jemalloc-devel
#安装openssl
sudo yum -y install openssl-devel
#安装epel
sudo yum -y install epel-release
```

我这里可能系统自带了很多已经装好的软件，如果在cmake3后还是报很多依赖缺失，按报错一次安装依赖即可.

装完依赖后要再次回去清掉build里产生的缓存，可以直接删除 ,删除完重新执行cmake3指向clickhouse根目录.

```bash
cd build
rm -rf *
cmake3 ..
```

这步最后会生成日志:

```bash
-- /data/clickhouse-19.7.3.9-stable/ClickHouse/utils: Have 63450 megabytes of memory. Limiting concurrent linkers jobs to 18 and compiler jobs to OFF
-- /data/clickhouse-19.7.3.9-stable/ClickHouse/dbms: Have 63450 megabytes of memory. Limiting concurrent linkers jobs to 18 and compiler jobs to 25
-- Will build ClickHouse 19.7.3.1 revision 54419
-- Using internal=OFF compiler=0: headers=/usr/share/clickhouse/headers/19.7.3.1 :  /usr/local/bin/clickhouse-clang   -pipe -msse4.1 -msse4.2 -mpopcnt  -fno-omit-frame-pointer  -Wall  -Wnon-virtual-dtor  -Wextra -Werror -O2 -g -DNDEBUG -O3  -std=c++1z -x c++ -march=native -fPIC -fvisibility=hidden -fno-implement-inlines -nostdinc -nostdinc++ -Wno-unused-command-line-argument -Bprefix=/usr/share/clickhouse -isysroot=/usr/share/clickhouse/headers/19.7.3.1; clickhouse-lld
-- Target check already exists
-- Configuring done
-- Generating done
-- Build files have been written to: /data/clickhouse-19.7.3.9-stable/ClickHouse/build
```

说明可执行编译的文件已经生成到build目录,在接下来的那一步去执行编译即可

6. 编译

cd /data/clickhouse-19.7.3.9-stable/ClickHouse/build

make

大约在7%的时候会报错:

![avatar](/images/clickhouse/clickhouse-install-error-01.png)

查了一下官网的issue,有同样的小伙儿碰到这个问题:

<https://github.com/yandex/ClickHouse/issues/5043>

按推荐的方式试了一下:

![avatar](/images/clickhouse/clickhouse-install-error-solve01.png)

这里我暂时不需要指定编译后的文件的目录，就没有指定编译目录，直接让其生成在build目录

cd /data/clickhouse-19.7.3.9-stable/ClickHouse/build

rm -rf *         (这里清理已经生成的文件)

\## 再次执行cmake3带上参数如下, 大约10分钟左右生成编译文件成功

cmake3 ..  -DGLIBC_COMPATIBILITY=OFF -DCMAKE_BUILD_TYPE:STRING=RelWithDebInfo

成功后，再次编译

make

这个过程比较漫长 ，个人从 16:45开始编译，直到 20:35才编译完,成功编译的结果:

![avatar](/images/clickhouse/clickhouse-install-success-end.png)

最终编译成功的可执行的程序在

build/dbms/programs下,如  clickhouse-server


7. 启动

直接 进入build/dbms/programs执行:

```ba
./clickhouse-server
```

报错

![avatar](/images/clickhouse/clickhouse-start-error.png)

显然找不到配置文件,我们可以按惯例查看一下help，还好它做了help

```bash
./clickhouse-server --help
```

![avatar](/images/clickhouse/clickhouse-start-01.png)

嗯，good .这个help做的还不错，很好懂.
这个配置文件上哪儿去找呢.看官方文档: <https://clickhouse.yandex/docs/en/getting_started/> 官方提供中文版文档，但是还没翻译完,只有一半是中文的

![1559185229473](/images/clickhouse/clickhouse-doc.png)

OK ，去我们拉下来的源代码目录 src/dbms/programs/server/config.xml 拷一份放到一个目录，改一下配置，启动,具体的配置慢慢参考官网文档研究..