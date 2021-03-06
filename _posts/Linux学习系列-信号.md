---
date: 2015-01-11 10:47
status: public
author: Haijin Ma
title: 【原创】Linux学习系列-信号
categories: [Linux]
tags: [Linux,信号]
---

#信号的定义和分类
信号是软件中断，提供了典型的异步机制。每个信号有一个编号，信号分为两类：非实时信号和实时信号。0-31编号属于非实时信号；31-63编号属于实时信号。为什么会分为这两类信号呢？这个主要是因为历史原因，首先实现的是非实时信号，非实时信号也成为不可靠信号，是因为其实现机制导致这类信号可能会丢失；而实时信号，由于存在排队机制，所以不会丢失。关于这点会在信号的处理过程图示中详细阐述。
#信号的来源
信号事件的发生有两个来源：硬件来源(比如我们按下了键盘或者其它硬件故障)；软件来源，最常用发送信号的系统函数是kill, raise, alarm和setitimer以及sigqueue函数，软件来源还包括一些非法运算等操作。
#信号的处理时机
每个信号都可以被关联1个信号处理函数，如果没有关联，其处理动作是系统默认的，大部分都是动作都是忽略，具体每个信号的默认处理动作可以谷歌查询。在目标进程执行过程中，会检测是否有信号等待处理（每次从系统空间返回到用户空间时都做这样的检查），如果有，则调用信号处理函数。
#信号的状态
实际执行信号的处理动作称为信号递达，信号从产生到递达之间的状态成为未决。进程既可以忽略信号，也可以阻塞信号。阻塞的意思是信号不会被递达，而忽略是信号递达之后的一个可选动作。信号在内核中的表示包括两个标记位：阻塞标记位和未决标记位以及信号处理函数的指针。
#信号的处理过程
1. 如果存在未决信号等待处理且该信号没有被进程阻塞，则在运行相应的信号处理函数前，进程会把信号在未决信号链中占有的结构卸掉。是否将信号从进程未决信号集中删除对于实时与非实时信号是不同的。
2. 对于非实时信号来说，由于在未决信号信息链中最多只占用一个sigqueue结构，因此该结构被释放后，应该把信号在进程未决信号集中删除（信号注销完毕）。
3. 对于实时信号来说，可能在未决信号信息链中占用多个sigqueue结构，因此应该针对占用sigqueue结构的数目区别对待：如果只占用一个sigqueue结构（进程只收到该信号一次），则应该把信号在进程的未决信号集中删除（信号注销完毕）。否则，不应该在进程的未决信号集中删除该信号（信号注销完毕）。

下图就是我理解的信号处理过程：

![](http://7xj5jf.com1.z0.glb.clouddn.com/信号处理机制-1.png)






