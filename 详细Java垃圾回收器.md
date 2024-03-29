# 详解Java垃圾回收器

**吞吐量（吞吐量=运行用户代码的时间 / (运行用户代码的时间 + 垃圾收集时间))**

## 为什么需要使用垃圾收集器？

    JVM（Java虚拟机）需要垃圾收集器是为了自动管理和回收程序运行时产生的内存中的垃圾对象，以提高内存的利用率和程序的性能。

以下是几个原因：

1. **动态分配内存：Java程序在运行时使用动态分配的内存。**
   
   程序员无需显式地分配和释放内存，而是依赖于垃圾收集器来自动管理内存。这减轻了程序员的负担，并减少了内存泄漏和悬挂指针等常见的低级错误。

2. **自动回收垃圾对象：Java程序创建的对象可能会在程序的执行过程中变得不再可访问或不再需要。**
   
   如果没有垃圾收集器，这些废弃的对象将继续占用内存空间，导致内存泄漏和内存耗尽。垃圾收集器会自动识别和回收这些垃圾对象，释放内存供其他对象使用。

3. **解决内存碎片问题：在程序执行过程中，对象的创建和销毁是动态的，可能导致内存中出现不连续的空闲内存碎片。**
   
   这使得分配大对象或连续内存块变得困难。垃圾收集器通过执行内存整理操作，将存活对象移动到一起，从而减少了内存碎片，使得内存分配更加高效。

4. **提高性能：垃圾收集器可以根据应用程序的行为和工作负载动态地调整内存回收的策略。**
   
   通过优化垃圾收集的算法和策略，可以最大限度地减少垃圾收集的停顿时间，提高应用程序的响应性能和吞吐量。

## JVM各垃圾收集器特点

| 垃圾收集器             | 回收区域       | 回收算法  | 多线程支持 | 目标                 |
|:-----------------:|:---------- | ----- | ----- |:------------------ |
| Serial            | 年轻代        | 标记-复制 | 否     | 小型应用，停顿时间较长        |
| ParNew            | 年轻代        | 标记-复制 | 是     | 与CMS收集器搭配使用，减少停顿时间 |
| Parallel Scavenge | 年轻代        | 标记-复制 | 是     | 高吞吐量应用，减少停顿时间      |
| Serial Old        | 老年代        | 标记-整理 | 否     | Serial收集器无法满足的需求   |
| Parallel Old      | 老年代        | 标记-整理 | 是     | 高吞吐量应用的老年代收集       |
| CMS               | 老年代        | 标记-清除 | 是     | 对停顿时间敏感的应用         |
| G1                | 年轻代+老年代    | 分代收集  | 是     | 大堆、低延迟的应用          |
| ZGC               | Region内存布局 | 标记-整理 | 是     | 大堆、极低延迟的应用         |

1. ##### Serial垃圾收集器
   
   Serial垃圾收集器是Java虚拟机中的一种垃圾收集器，具有以下特点：
   
   1. **单线程**：Serial回收器是单线程收集器，即在垃圾收集时只是用一个线程。这意味着它只能利用单个CPU核心来执行垃圾收集操作。
   
   2. **标记-复制算法**：Serial收集器采用“标记-复制”算法来进行垃圾收集。首先，它会标记所有活动对象，然后将活动对象复制到一个新的区域，同时清除非活动对象。这样可以保证新生代的内存空间始终是连续的。
   
   3. **针对新生代**：Serial收集器主要用于收集新生代，即新生代中Eden区和Survivor区。它通过复制算法来实现高效的垃圾回收。
   
   4. **暂停应用程序**：在进行垃圾收集期间，Serial收集器会暂停所有的应用线程。这意味着在垃圾收集期间，应用程序的执行将会停顿。

2. ##### ParNew垃圾收集器
   
   ParNew（并行新生代）垃圾收集器，是一种多线程的垃圾回收器，主要用于新生代的垃圾回收。具有以下特点：
   
   1. **多线程收集器**：ParNew垃圾收集器使用多线程进行垃圾回收操作，利用多核CPU的优势，可以并行的进行垃圾收集。多线程的并行性可以加快垃圾回收的速度。
   
   2. **标记-复制算法**：ParNew垃圾收集器采用“标记-复制”算法。它将新生代分成一个或多个存活区和一个空闲区，在垃圾回收过程中，将存活的对象复制到空闲区，然后清理存活区中的垃圾对象。
   
   3. **可与CMS收集器配合使用**：ParNew垃圾收集器可以和CMS收集器搭配使用，在新生代进行并行垃圾回收，而CMS负责老年代的并发垃圾回收。
   
   需要注意的是，ParNew垃圾收集器主要关注吞吐量，而不是最低停顿时间。如果应用程序对停顿时间要求非常严格，可以考虑其他的垃圾回收器，如G1和ZGC。

3. ##### Parallel Scavenge收集器
   
   Parallel Scavenge（并行清扫）垃圾收集器，是一种多线程垃圾收集器，主要用于新生代的垃圾回收。
   
   1. **多线程收集器**：Parallel Scavenge垃圾收集器使用多线程进行垃圾收集，利用多核CPU的优势，可以并行的进行垃圾收集。多线程的并行性可以加快垃圾回收的速度。
   
   2. **高吞吐量**：Parallel Scavenge垃圾收集器主要关注应用程序的吞吐量，即在给定时间内完成尽可能的工作量。它通过
   
   3. **标记-复制算法**：

4. 

https://zhuanlan.zhihu.com/p/248709769?utm_id=0
