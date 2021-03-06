---
title: "编译原理学习笔记-术语介绍(开篇)" # Title of the blog post.
date: 2017-08-06T20:57:08+08:00 # Date of post creation.
description: "Principle of Compiler." # Description used for search engine.
featured: true # Sets if post is a featured post, making appear on the home page side bar.
draft: false # Sets whether to render this page. Draft of true will not be rendered.
toc: false # Controls if a table of contents should be generated for first-level links automatically.
# menu: main
featureImage: "/images/building.png" # Sets featured image on blog post.
thumbnail: "/images/dollar.png" # Sets thumbnail image appearing inside card on homepage.
shareImage: "/images/dollar.png" # Designate a separate image for social media sharing.
codeMaxLines: 10 # Override global value for how many lines within a code block before auto-collapsing.
codeLineNumbers: false # Override global value for showing of line numbers within code block.
figurePositionShow: true # Override global value for showing the figure label.
categories:
  - 编译原理
tags:
  - 编译器
  - 编译原理
# comment: false # Disable comment if false.
---
一直对lucene源码感兴趣，但是每次看到分词的地方，上面一大堆注释说是用xx工具生成的，就不了了之，也没能跳过这个地儿，作为一个打破砂锅问到底的人强迫症太厉害了，决定先攻编译原理，整明白词法分析，语法分析再来攻lucene，整好最近看redis 和mysql源码解析命令和sql语句，先来上个脑图:

![avatar](/images/compile-internal-0.md-1.png)

学习前，先来了解两个概念：

- **编译器** ：计算机上运行的所有软件都是用某种程序设计语言编写的，但是一个程序在运行之前需要被翻译成能够被计算机所识别的形式，也就是及机器语言，完成这项翻译任务的软件就是编译器，也就是说编译器本身也是一个软件,如下图:

![编译器.png](https://upload-images.jianshu.io/upload_images/12069275-67cb3161c1996048.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)



- **解释器**：直接使用用户提供的输入的源程序进行计算执行，同时把结果输出给用户，如下图:

![解释器.png](https://upload-images.jianshu.io/upload_images/12069275-8b8f32307fac5718.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


> 显然：由编译器产生的及其语言目标程序要比解释器要快的多，因为解释器顺序读取用户的输入或者叫客户端字符流，再同时进行转换成及机器语言交给CPU计算执行，最后把结果返回给客户端.这其中还有程序优化的原因，稍后说明.



因为我个人是java出身，所以这里简单聊一下Java编译器的编译过程.

一个java源代码首先经过javac 被编译成子解码的中间表示形式，这里子解码并不是最终的机器语言，为什么呢,因为java 需要解决跨平台的问题，编译后的目标程序自然不能是机器码，因为每个平台和硬件上的结构都是不一样的.导致底层执行的差异.完成这个适配的软件就是java虚拟机,所以java虚拟机不是跨平台的.这样一看站在java虚拟机的角度上看，虚拟机就是一个解释器，解释的是javac编译成的字节码. 是不是突然明白了为什么java的速度不如C/C++了? 哈哈, C/C++ 经过gcc 编译器最终生成的是机器码，或者非常接近机器码(链接以后)，这里给出unix上源程序从编译到最终能运行的示例图, 应该会好理解一点:

![执行过程](http://upload-images.jianshu.io/upload_images/12069275-41e27658b0d3d174.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

所以可以确认java是同时拥有编译和解释两种执行器.   javac 命令的动作是编译,  对生成的.class文件执行java命令的动作是解释执行.

对上面的图解释以下：源程序经过 编译器 会产生一个汇编程序源码作为输出, 然后这个汇编程序源码经过汇编器进行处理, 并生成可重定位的机器码,最后可重定位的机器码与其他可重定位的文件或者库进行链接, 如系统打印函数. 最终生成真正可执行文件. 最后我们在执行./xx 的时候由加载器把所有可执行的目标文件放到内存中执行.

说的通俗一点，其实可以把编译器看成一个黑盒,它能把源程序映射成在语义上等价的目标程序.其内部可以总结成按如下步骤顺序执行的:

![编译器的各个步骤.png](https://upload-images.jianshu.io/upload_images/12069275-4323122583acada4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


下面具体来聊这几个步骤

- 词法分析

这是编译器的第一个步骤, 上游接着程序员编写的源代码程序, 下游是语法分析的入口. 所有词法分析器读入源程序的字符流, 然后将其拆分成有意义的词素的序列, 将其信息存放在一个被称作是符号表的数据结构中 对于每个词素, 词法分析器产生一个词法单元token传递给语法分析器, 这个词法单元表示成:

```xml
    <token-name,attribute-value/>
```

这个token-name 是一个由语法分析步骤使用的抽象符号,  attribute-value指向符号表中关于这个词法单元的条目. 这些条目的信息会被语义分析和代码生成步骤使用.

如下源程序：

```java
  position = initial + rate * 60
```

这条语句可以映射成如下词法单元. 并且这些词法单元将被传递给语法分析阶段.

- position 是一个词素，被映射成词法单元 <id , 1> , id表示标识符的抽象符号,  1 指向符号表position对应的条目, 可以理解成数组下标. 一个标识符对应的符号表条目存放该标识符有关的信息，比如它的名字和类型.
- = 是赋值符号, 也是一个词素, 被映射成词法单元 < = >. 因为这个词法单元不需要属性值, 所以省略了第二个分量.
- initial 同样是一个词素, 被映射成词法单元 <id , 2> , 2 指向initial对应的符号表条目
- +也是一个词素, 被映射成词法单元< + > .
- rate 被映射成词法单元 <id , 3>, 3 指向rate对应的符号条目.
- *是一个词素, 被映射成词法单元< * >.
- 60是一个词素,被映射成词法单元 < 60 >
- 分割词素的空格被词法分析器忽略掉

经过这么多步骤后，上面的父之语句可以表示成如下的词法单元序列:

```java
   <id , 1> < = > <id , 2> + < + > <id, 3> < * > < 60 >
```

这个表示中, 词法单元名 = 、 + 和 * 分别表示赋值、加法运算符、乘法运算符的抽象符号.

- 语法分析

这个阶段语法分析器使用由词法分析器生成的各个词法单元的第一个分量来创建树形中间表示. 这个表示方法也叫语法树, 书中的每个节点表示一个运算，而子节点表示运算的分量，如图:

![语法分析器.png](https://upload-images.jianshu.io/upload_images/12069275-825cc0306cab02c2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 语义分析

语义分析器(semantic analyzer) 使用语法树和符号表中的信息来检查源程序是否和语言定义的语义一致, 同时它也收集类型信息，并把这些信息存放在语法树或符号表中,供随后的中间代码生成 使用.

语义分析有个重要的步骤是类型检查. 编译器检查每个运算符是否具有匹配的运算分量. 如Java和C中要求数组的下标必须是整数.  若果用一个浮点数作为数组的下标. 编译器必须报出错误, 并给出相应的提示. 有些编译器允许自动类型转换, 如Java和C. 当一个运算符应用于一个浮点数和一个整数时, 编译器自动会将该整数转换成一个浮点数. 如下图显示了语义分析器的工作,展示了一个自动类型转换.

![语义分析器-new.png](https://upload-images.jianshu.io/upload_images/12069275-0c609b7c42275524.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


与上面的语法分析器不同的是，这里多了一个关于运算符inttofloat的额外节点. inttofloat明确的指出了把它的整数类型参数转换成一个浮点数, 后面的文章再次介绍.

- 中间代码生成

把源代码翻译成目标代码的过程中, 编译器可能构造成一个或者多个中间表示. 这些中间表示可以有多种形式.语法树也是其中的一种, 他们通常在语法分析和语义分析中使用.所以中间表示应该由两个重要的性质:

- 易于生成
- 能够被轻松的翻译成目标机器上的语言

通常情况下这种中间表示由一组指令组成, 每个指令有三个运算分量. 前面的程序可以生成的中间代码如下:

```java
   t1 = inttofloat(60)
   t2 = id3 + t1
   t3 = id2 + t2
   id1 = t3
```

每一个指令的右边最多只能由一个运算符, 并且这些指令确定了运算完成的顺序. 正如上图, 乘法应该在加法之前完成. 而且编译器生成了很多临时的名字存放每个指令计算的结果. 变成图就如:

![中间代码生成器-new2.png](https://upload-images.jianshu.io/upload_images/12069275-0507d0759f47b14c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


- 代码优化

机器无关的代码优化步骤试图改进中间代码, 以便生成更好的目标代码. 更好在这里的意思是 更快 执行时间更短或者能耗更低的目标代码.

代码优化器对上面图中生成的中间代码做优化:

1. 把60从整数类型转化成浮点数类型60.0, 消除相应的inttofloat运算

2. 上面的指令中t3仅仅被用了一次, 值是把它的值传递给id1 .所以上面的程序最终被优化器执行生成如下简洁的代码:

```java
   t1 = id3 * 60.0
   id1 = id2 + t1
```

用图表示即为:

![代码优化 (1).png](https://upload-images.jianshu.io/upload_images/12069275-10a81b1e2d3e9c7c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


不同的编译器生成所作的代码优化工作量相差很大. 优化工作做的最多的编译器, 会在优化阶段花费相当多的时间. 简单的优化方法可以极大的提高目标程序的运行效率而不会降低编译的速度.

- 代码生成

代码生成器的以源程序的中间表示形成作为输入, 把它映射成新的目标语言. 如果目标语言是机器代码,必须为程序使用的每一个变量选择寄存器或内存位置.然后中间指令被翻译成能够完成相同任务的机器指令序列. 代码生成的一个至关重要的方面是合理的分配寄存器以存放变量的值. 如上面程序可以被翻译成如下机器代码:

```java
   LDF  R2,  id3
   MULF R2,  R2,  #60.0
   LDF  R1,  id2
   ADDF R1,  R1,  R2
   STF  id1, R1
```

稍微对这个机器代码解释以下,详细的内容,后面的文章会由依依揭开(参考汇编语言)：

1. 第一行 把地址id3的内容加载到寄存器R2中;

2. 第二行 将寄存器R2中的值取出与60.0相乘, 然后再把计算的结果重新加载到寄存器R2中. 其中 # 表示60应该作为一个立即数处理(后面的文章解释,暂时现有个印象);

3. 第三行  把地址id2的内容加载到寄存器R1中;

4. 第四行  取出寄存器R2的值, 取出寄存器R1,把两者向加后的结果重新加到寄存器R1中;

5. 最后一行 将寄存器R1中的值存放在id1的地址中(这里应该是经过地址总线).

![代码生成器.png](https://upload-images.jianshu.io/upload_images/12069275-54eacc4224ac126e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


了解了代码优化器后, 现在想一想为什么java中的反射构造对象、动态代理加载自定义自己码 没有new 构造对象快了. 对的, 因为编译器无法优化代码.

- 符号表管理

编译器还有一个重要的功能就是记录源程序中使用的变量的名字,并收集每个名字的各种属性相关的信息.这些属性可以提供一个名字的存储分配、类型、作用域.对于过程名字(函数或方法),这些信息还包含: 参数数量和类型、每个参数的传递方法(值传递还是地址传递)以及返回类型.

符号表在马上接下来的文章介绍.

- 多个步骤组合成趟

在一个特定的实现中, 前面的多个步骤可以被组合成一趟. 每趟读入一个输入文件并产生一个输出文件. 比如前面提到的词法分析、语法分析、语义分析、中间代码生成可以诶组合在一起成为一趟. 代码优化作为一个可选的趟.

- 编译器构造工具

成功的工具都能隐藏生成算法的细节,并且他们生成的组件易于和编译器的其他部分整合. 常用的编译器构造工具包括:

1. 语法分析器生成器
2. 词法分析器生成器
3. 语法制导的翻译引擎
4. 代码生成器的生成器
5. 数据流分析引擎
6. 编译器构造工具集

这些内容稍后介绍.这里总结一下:

> 本篇对开篇的脑图中提供的术语做了简单的一一解释, 现在大概能明白整个编译器每个结构的职责.以图文的方式加以说明帮助理解. 接下来的篇幅中会深入每个环节进行讨论.

由于个人水平限制,如有错误,欢迎各位留言指正.



## 参考文献

1. 编译原理 龙书 第二版
2. 深入理解计算机系统 第三版



















