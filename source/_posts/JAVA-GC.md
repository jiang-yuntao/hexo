---
title: JAVA  GC
date: 2019-10-23 10:58:09
tags: GARBAGE COLLECTION
categories: 

---
# GARBAGE COLLECTIONS:
## *1.Serial:*
>Client模式默认新生代收集器 

>单线程收集器,“ Stop The World ”--垃圾收集时暂停所有的工作线程

>新生代复制算法,老年代标记-整理算法

## *2.ParNew:*
>Serial多线程版本,两收集器代码共用了很多

>许多虚拟机Sever模式下首选新生代收集器

>新生代复制算法,老年代标记-整理算法

## *3.Parallel Scavenge:*
> 新生代收集器,复制算法收集器

> 并行多线程收集器

> 关注于可控制吞吐量

> 吞吐量=运行用户代码时间/(运行用户代码时间+垃圾收集时间)

>也称为“吞吐量优先”收集器

>通过-XX:+UseAdaptiveSizePolicy参数值,可以让虚拟机根据当前系统运行情况收集性能监控信息,动态调整新生代的大小、Eden与Survivor比例、晋升老年代对象大小等参数,来提供最合适的停顿时间或最大吞吐量,这种调节方式称为GC自适应的调节策略,这也是```Parallel Scavenge```与```ParNew```重要区别

## *4.Serial Old:*
>单线程

>标记整理算法

>```Serial```老年代版本

## *5.Parallel Old:*
>多线程

>标记整理算法

>```Parallel Scavenge```老年代版本

>Parallel Scavenge 的老年代收集器只能选择Serial Old *(JDK1.6以前)*,因为Serial Old是单线程的,导致无法充分发挥服务器多CPU处理能力,因此这种组合吞吐量可能不一定高于ParNew 和 CMS 组合 *(虽然Parallel注重吞吐量)*

>Parallel Old 改善了这种情况,注重吞吐量以及CPU资源场合,优先考虑Parallel 和Parallel Old搭配的组合

## *6.CMS(Concurrent Mark Sweep):*
>标记-清除算法

>注重最短回收时间

>运行过程比前面的收集器更加复杂

>互联网站或B/S系统服务端上的Java应用(占Java应用比很高)非常重视服务的响应速度

>并发收集、低停顿

>对CPU资源敏感,并发阶段容易导致应用程序变慢

>无法处理浮动垃圾(FLoating Garbage,标记过程之后由于用户程序还在运行,继续产生的垃圾--当前GC处理不了了,留待下次GC清理),可能出现“Concurrent Mode Failure”(内存无法满足程序需求)导致另一次Full GC的产生,需要预留一部分空间提供并发收集时的程序运作使用

>标记-清除算法导致大量空间碎片产生,大对象分配麻烦,容易出发Full GC

>通过-XX:+UseCMSCompactAtFullCollection(开关参数)来应对内存碎片导致Full GC时进行内存碎片合并整理过程,但是内存整理过程无法并发,因此停顿时间增加

>通过—XX:CMSFullGCsBeforeCompaction设置执行多少次不压缩Full GC之后来一次带压缩的,默认值为0

###### 运行过程:
1.初始标记(CMS initial mark)
>需要“Stop the World”

>标记的是GC Roots直接关联的对象,速度快

2.并发标记(CMS concurrent mark)
>进行GC Roots Tracing

3.重新标记(CMS remark)

>远短于并发标记时间

>比初始标记阶段稍长

>修正并发标记期间因用户程序继续运行导致标记产生变动的那一部分对象的标记记录

4.并发清除(CMS concurrent sweep)

---
>*整个过程耗时最长的并发标记和并发清除过程可以与用户线程一起工作,所以整体上来说,CMS内存回收过程是与用户线程一起并发执行的*


## *7.G1(Garbage-First):*
>并发与并行:充分利用了多CPU、多核环境的硬件优势,多个CPU缩短Stop-The—World停顿时间,部分其他收集器原来需要停顿Java线程执行的GC动作,G1可以通过并发让Java程序继续执行

>分代收集:不需要其他收集器配合,独立管理整个GC堆;可以采用不同方式处理新创建的和已经存活一段时间、熬过多次GC的旧对象以获取更好的收集效果

>空间整合:整体上是“标记-整理”算法,局部是“复制”算法实现,不会产生空间碎片

>可预测的停顿:注重停顿时间的同时,可以建立可预测的停顿时间模型,能让使用者在一个长度为M毫秒的时间片段内,消耗在垃圾收集上的时间不超过N毫秒(这几乎是实时Java(*RTSJ*)的垃圾收集器的特征)

>内存布局和其他相差很大,Java堆划分为多个大小相等的独立区域(Region),停顿时间模型的建立依靠有计划的避免在整个Java堆中进行全区域的垃圾收集

>G1跟踪各个Region里面的垃圾堆积的价值大小(回收所获得的空间大小以及回收所需要的时间的经验值),在后台维护一个优先列表,每次根据允许的收集时间,优先回收价值最大的Region(这也就是
Grabage-First名称的由来)

>每个Region都有一个与之对应的Remembered Set,通过在GC根节点的枚举范围中加入Remembered Set即可保证不对全堆扫描也不会有遗漏
######  运作过程大致如下(不计算维护Remembered 与CMS的过程很相似)
1.初始标记(Initial Marking)
2.并发标记(Concurrent Marking)
3.最终标记(Final Marking)
4.筛选回收(Live Data Counting and Evacuation)
