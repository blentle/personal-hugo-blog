---
title: "Linux Oom Killer机制" # Title of the blog post.
date: 2019-11-04T22:10:14+08:00 # Date of post creation.
description: "Article description." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
featureImage: "/images/linux-oom-killer-00.jpg" # Sets featured image on blog post.
thumbnail: "/images/linux-oom-killer-001.jpeg" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/dollar.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - Linux
tags:
  - oom
  - kernel
  - Linx内核
# comment: false # Disable comment if false.
---

## linux oom killer机制

### 简介

The OOM Killer 是内核中的一个进程，当系统出现严重内存不足时，它就会启用自己的算法去选择某一个进程并杀掉. 之所以会发生这种情况，是因为Linux内核在给某个进程分配内存时，会比进程申请的内存多分配一些. 这是为了保证进程在真正使用的时候有足够的内存，因为进程在申请内存后并不一定立即使用，当真正使用的时候，可能部分内存已经被回收了. 比如 当一个进程申请2G内存时，内核可能会分配2.5G的内存给它.通常这不会导致什么问题.然而一旦系统内大量的进程在使用内存时，就会出现内存供不应求.很快就会导致内存耗尽. 这时就会触发这个oom killer,它会选择性的杀掉某个进程以保证系统能够正常运行.



### The OOM Killer选择哪个进程杀掉?

The OOM Killer通过检查所有正在运行的进程，然后根据自己的算法给每个进程一个badness分数.拥有最高 badness分数的进程将会在内存不足时被杀掉.它打分的算法如下:

+ 某一个进程和它所有的子进程都占用了很多内存的将会打一个高分;
+ 优先选择进程号最小的那个进程
+ 内核进程和其他较重要的进程会被打成相对较低的分.

The OOM Killer给每一个进程打的分数都放在 /proc/{pid}/oom_score文件中，其实这里有三个文件，依次是

oom_score、oom_adj、oom_score_adj. 这三个文件按Linux官方文档来说就是：

oom_score是存储最终的分数，也就是badneess分数，最高的会被kill掉, man 一下 proc，找到:

```bash
    /proc/[pid]/oom_score (since Linux 2.6.11)
    This  file displays the current score that the kernel gives to this process for the
    purpose of selecting a process for the OOM-killer.  A higher score means  that  the
    process  is more likely to be selected by the OOM-killer.  The basis for this score
    is the amount of memory used by the process, with increases (+)  or  decreases  (-)
    for factors including:

    * whether the process creates a lot of children using fork(2) (+);

    * whether  the  process has been running a long time, or has used a lot of CPU time
    (-);

    * whether the process has a low nice value (i.e., > 0) (+);

    * whether the process is privileged (-); and

    * whether the process is making direct hardware access (-).

    The oom_score also reflects  the  adjustment  specified  by  the  oom_score_adj  or
    oom_adj setting for the process.
```



oom_adj这个文件已经过时了，当前存在 是为了兼容旧版本的内核,， 同样man一下 proc 找到:

```bash
    /proc/[pid]/oom_adj (since Linux 2.6.11)
    This file can be used to adjust the score used to select which  process  should  be
    killed  in an out-of-memory (OOM) situation.  The kernel uses this value for a bit-
    shift operation of the process's oom_score value: valid values are in the range -16
    to  +15, plus the special value -17, which disables OOM-killing altogether for this
    process.  A positive score increases the likelihood of this process being killed by
    the OOM-killer; a negative score decreases the likelihood.

    The  default  value for this file is 0; a new process inherits its parent's oom_adj
    setting.  A process must be privileged (CAP_SYS_RESOURCE) to update this file.

    Since  Linux   2.6.36,   use   of   this   file   is   deprecated   in   favor   of
    /proc/[pid]/oom_score_adj.
```

oom_score_adj 是新版本内核官方建议使用的,看一下使用说明:

```bash
    /proc/[pid]/oom_score_adj (since Linux 2.6.36)
    This  file can be used to adjust the badness heuristic used to select which process
    gets killed in out-of-memory conditions.

    The badness heuristic assigns a value to each candidate task ranging from 0  (never
    kill)  to 1000 (always kill) to determine which process is targeted.  The units are
    roughly a proportion along that range of allowed memory the  process  may  allocate
    from, based on an estimation of its current memory and swap use.  For example, if a
    task is using all allowed memory, its badness score will be 1000.  If it  is  using
    half of its allowed memory, its score will be 500.

    There  is  an  additional  factor included in the badness score: root processes are
    given 3% extra memory over other tasks.

    The amount of "allowed" memory depends on the context in which the  OOM-killer  was
    called.   If it is due to the memory assigned to the allocating task's cpuset being
    exhausted, the allowed memory represents the set of mems assigned  to  that  cpuset
    (see  cpuset(7)).   If  it  is  due  to  a mempolicy's node(s) being exhausted, the
    allowed memory represents the set of mempolicy nodes.  If it is  due  to  a  memory
    limit  (or  swap limit) being reached, the allowed memory is that configured limit.
    Finally, if it is due to the entire system being out of memory, the allowed  memory
    represents all allocatable resources.

    The  value  of  oom_score_adj  is  added  to the badness score before it is used to
    determine   which   task   to   kill.    Acceptable   values   range   from   -1000
    (OOM_SCORE_ADJ_MIN)  to  +1000 (OOM_SCORE_ADJ_MAX).  This allows user space to con‐
    trol the preference for OOM-killing, ranging from always preferring a certain  task
    or  completely disabling it from OOM-killing.  The lowest possible value, -1000, is
    equivalent to disabling OOM-killing entirely for that task, since  it  will  always
    report a badness score of 0.

    Consequently,  it  is  very simple for user space to define the amount of memory to
    consider for each task.  Setting a oom_score_adj value of  +500,  for  example,  is
    roughly  equivalent  to  allowing  the  remainder of tasks sharing the same system,
    cpuset, mempolicy, or memory controller resources to use at least 50% more  memory.
    A  value of -500, on the other hand, would be roughly equivalent to discounting 50%
    of the task's allowed memory from being considered as scoring against the task.

    For backward compatibility with previous kernels, /proc/[pid]/oom_adj can still  be
    used to tune the badness score.  Its value is scaled linearly with oom_score_adj.

    Writing  to  /proc/[pid]/oom_score_adj or /proc/[pid]/oom_adj will change the other
    with its scaled value.
```

最后一句也就是说为了兼容旧版本的内核，oom_score_adj和oom_adj任何一个变动，另一个也会自动跟着改动.

这三个文件先了解到这.后面还会用到.



### 如何找到一个进程是被The OOM Killer杀掉的?

最简单的方法就是用dmesg看系统日志. 对于redhat系的:

```bash
    dmesg | egrep -i “killed process”
```

比如系统可能输出(这是我本地测试的):

```bash
    host kernel: Out of Memory: Killed process 13482 (mysql).
```

或者直接查看日志

```bash
    egrep -i 'killed process' /var/log/messages*
```



### 如何阻止一些重要的进程不被The OOM Killer杀掉

The OOM killer 通常是检查 oom_score_obj(上面提到的)值，并经过计算得出最终的oom_score来决定杀死哪个进程的. 所以我们查一下内核里面定义的这个值的取值范围再去修改其值 .这里我看的是4.13.16这个版本.

源代码是 oom_kill.c <https://elixir.bootlin.com/linux/v4.13.16/source/mm/oom_kill.c>，里面引用了头文件

```c
    #include <linux/oom.h>
```

而这个oom.h又引用了

```c
    uapi/linux/oom.h
```

这个头文件，查看这个文件

内核定义的值的范围: https://elixir.bootlin.com/linux/v4.13.16/source/include/uapi/linux/oom.h

```c
    #ifndef _UAPI__INCLUDE_LINUX_OOM_H
    #define _UAPI__INCLUDE_LINUX_OOM_H

    /*
    * /proc/<pid>/oom_score_adj set to OOM_SCORE_ADJ_MIN disables oom killing for
    * pid.
    */
    #define OOM_SCORE_ADJ_MIN	(-1000)
    #define OOM_SCORE_ADJ_MAX	1000

    /*
    * /proc/<pid>/oom_adj set to -17 protects from the oom killer for legacy
    * purposes.
    */
    #define OOM_DISABLE (-17)
    /* inclusive */
    #define OOM_ADJUST_MIN (-16)
    #define OOM_ADJUST_MAX 15

    #endif /* _UAPI__INCLUDE_LINUX_OOM_H */
```

这意味着我们可以把要保护的进程的oom_score_obj的值调整成一个较小的负值, 或者把oom_adj调成 -17,这两个文件已经在上面说过了.

```bash
    sudo echo -200 > /proc/{pid}/oom_score_adj  （如果-200是所有进程中最大的，当系统内存不足时，还是会被oom-killer杀掉）
    或
    sudo echo -17 > /proc/{pid}/oom_adj  （不会被oom-killer杀掉）
```

### 如何查看所有正在Running的进程的badnees score

这里我借用一下[Raunak Ramakrishnan ](https://dev.to/rrampage)[ ](http://twitter.com/OrdinalSpace)[ ](http://github.com/rrampage)大神写的一个脚本

```bash
    #!/bin/bash
    # Displays running processes in descending order of OOM score
    printf 'PID\tOOM Score\tOOM Adj\tCommand\n'
    while read -r pid comm; do [ -f /proc/$pid/oom_score ] && [ $(cat /proc/$pid/oom_score) != 0 ] && printf '%d\t%d\t\t%d\t%s\n' "$pid" "$(cat /proc/$pid/oom_score)" "$(cat /proc/$pid/oom_score_adj)" "$comm"; done < <(ps -e -o pid= -o comm=) | sort -k 2nr
```

### 如何强制触发The OOM Killer

在内核官方文档上有一篇文章:

<https://www.kernel.org/doc/html/v4.11/admin-guide/sysrq.html>

详细说明了 /proc/sysrq-trigger的各种操作和作用



### 参考文献
- Linux内核官方文档：[@Linux官方内核文档](<https://www.kernel.org/doc/html/v4.11/admin-guide/sysrq.html>)(<https://www.kernel.org/doc/html/v4.11/admin-guide/sysrq.html>)
- 博文：<<https://github.com/lorenzo-stoakes/linux-vm-notes/blob/master/sections/oom.md>>
- Oracle官方文档：<<https://www.oracle.com/technical-resources/articles/it-infrastructure/dev-oom-killer.html>>
- 人工触发The OOM Killer：<<https://www.lynxbee.com/how-to-invoke-oom-killer-manually-for-understanding-which-process-gets-killed-first/>>
- Raunak Ramakrishnan大神的博客: <<https://dev.to/rrampage/surviving-the-linux-oom-killer-2ki9>>