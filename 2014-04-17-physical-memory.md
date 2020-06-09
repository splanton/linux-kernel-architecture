---
layout:    post
title:     物理内存布局
category:  内存寻址
description: 物理内存布局...
tags: 内存 物理内存 物理内存布局
---
在系统初始化阶段，内核必须建立一个物理地址映射来指定哪些物理地址范围对内核可用，哪些对内核不可用，因为会有一些物理内存被用作特殊的目的，例如BIOS的一些数据会包含在前1M的物理内存中。

内核将下列页框记为保留：

1. 在不可用的物理地址范围内的页框。
2. 含有内核代码和已初始化的数据结构的页框。

保留页框中的页框绝对不能被动态分配或交换到磁盘上。

一般来说，Linux内核安装在RAM中的从物理地址0x00100000开始的地方，就是从第二个MB开始，所需页框总数依赖于内核的配置方案，景点的配置所得到的内核可以被安装在小于3MB的RAM中。

之所以内核没有被安装在物理内存最开始的地方，是因为PC体系结构有几个特殊的地方必须考虑到，例如，页框0由BIOS使用，存放加电自检（*Power-On Self-Test，POST*）期间检查到的系统硬件配置。因此，很多笔记本电脑的BIOS在系统初始化后还将数据写回该页框。

物理地址从0x000a0000到0x000fffff的范围通常刘改BIOS例程，并且映射ISA图形卡上的内部内存。这个区域是所有IBM兼容PC上从640KB到1MB之间著名的洞[^1]。第一个MB内的其他页框可能由特定计算机模型保留。

[^1]: 物理地址存在但被保留，对应的页框不能由操作系统使用。

在启动过程的早期阶段，内核询问BIOS并了解物理内存的大小，通常，内核页调用BIOS过程建立一组物理地址范围和其对应的内存类型。随后，内核执行*machine_specific_memory_setup()*函数，该函数建立物理地址映射，如果这张表是可获取的，那是内核在BIOS列表的基础上构建的，否则，内核保守的设置这张表为默认值[^2]。

[^2]: 从0x9f（LOWMEMSIZE）到0x100（HIGH_MEMORY）号的所有页框都标记为保留。

内核可能不会见到BIOS报告的所有物理内存，如果没有使用PAE机制，则最高寻址4GB大小的RAM。*setup_memory()*函数在*machine_specific_memory_setup()*函数之后被调用，它分析物理内存区域表示并初始化一些变量来描述内核物理内存布局。

![system](images/page_frame.png)
物理内存的布局

为了避免把内核装入一组不连续的页框里，Linux更愿意跳过RAM的第一个MB，明确地说，Linux用PC体系结构未保留的页框来动态存放所分配的页。

符号\_text对应于物理地址0x00100000，来表示内核代码第一个字节的地址。内核代码的结束位置由另外一个类似的符号\_etext。内核数据分为两组，初始化过的数据的和没有初始化的数据。初始化过的数据在\_etext后开始，在\_edata处结束。紧接着是未初始化的数据并以\_end结束。这些符号并没有在Linux源代码中定义，它们是内核编译中产生的[^3]。

[^3]: 在System.map文件中找到这些符号的线性地址，System.map是编译内核以后所创建的。