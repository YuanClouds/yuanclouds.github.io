---
title: 线程优化系列】16ms？60Fps？Android中的多线程
date: 2018-09-12 19:51:00
tags:
---

﻿# 【线程优化系列】16ms？60Fps？Android中的多线程
> 相信很多童鞋都知道优化响应的终极目标就是60FPS，笔者在做优化的时候发现在比较重的页面的时候55FPS都是一个难啊。在做优化的时候，基础的工作也是并不可少的，首先为什么是60FPS、16ms，为什么多线程优化可以优化UI响应？所以我们抱着疑问的心态来阅读下这一偏文章吧~
## 为什么是16ms?
当我们开发的页面，过度发生卡顿的时候。不用怀疑，很大的原因是有一些逻辑操作没有在16ms中完成导致了应用页面卡顿。如果因为太卡顿导致用户吐槽后卸载，流失率简直会扎心啊
### 16ms的由来
首先大多数Android系统的显示屏幕都是60hz的，则意味着屏幕以每秒60帧的速率来刷新页面。即每帧花的时间是 1000/60 = 16ms。一帧可以理解成在显示器显示的一张照片，一秒则显示刷新了60张（人类肉眼并不能识别~）。在Android系统中，系统会每隔16ms就会发出一次VSYNC信号触发对UI进行渲染，加入这16ms内我们的逻辑没有完成视图UI的绘画工作，这时候就会出现卡顿，所谓的掉帧。
+ VSYNC
VSYNC 表示垂直同步（Vertical Synchronization），可以理解成“帧同步”。Android系统中会在每16ms发送VSYNC信号触发更新帧达到UI渲染更新，并且要求CPU和GPU每秒要有处理60帧的能力，一帧花费的时间在16ms内，附SYNC工作图如下：

![](https://upload-images.jianshu.io/upload_images/661427-ed0d5759d8d7e75b.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/668/format/webp)

![]https://upload-images.jianshu.io/upload_images/661427-6c2aee41397b62ad.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/668/format/webp)

这个图怎么理解呢？
首先Display表示每一帧操作流程，一般这个过程像上面说的是16ms。在第0帧的时候，cpu和gpu已经在准备第1帧的数据，所以在图上第0帧的时候可以看到GPU和CPU准备1的数据，这就是Android的更新缓冲机制。“理论上如果在下一次VSYNC信号来临之前，GPU和CPU及时完成渲染和交换数据，是可以完全可以体验到60FPS的感觉”
图2描述的是Android的双缓冲机制，和上面的图1表示功能大概一致。在Android系统中，帧的数据是会保存在两个缓冲区中，分别是A、B。如果当前帧显示的是缓冲A的数据，则缓冲B则准备下一帧的数据。等到下一次VSYNC信号来的时候，当前帧显示缓冲B的数据，缓冲A准备下一帧的数据，以此类推。
+ 卡顿导致丢帧
丢帧，不明觉厉则是当VSYNC信号来临的时候，该准备的数据没准备好，只好舍弃这一帧等待下一次VSYNC信号来临的时候使用新的一帧。图文说明：

![](https://upload-images.jianshu.io/upload_images/661427-79517e484fee2f7e.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/668/format/webp)
在正常流程的是，当显示缓冲A区数据的帧，在下一次VSYNC信号来的时候，这时候GPU突然的工作做太多，来不及在VSYNC信号来临的时候提供对应的帧数据。这时候由于又不能直接清空缓冲A区的数据，所以这时候只能暂时性丢弃这一帧，在等下一帧的时候再显示缓冲B区数据。这个过程就是所谓的丢帧！Jank!

当然，Android为了弥补这个缺陷，引入了一个“三倍缓冲机制”，如图：

![](https://upload-images.jianshu.io/upload_images/661427-46cdc316ed47970f.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/668/format/webp)
C缓冲区用于在丢帧的时候用来及时缓冲下一帧的数据，在下一帧的时候直接会和原来缓冲区的数据一起给下一帧。例如图C+A。

## Android 有哪些多线程？
从图中了解到发生丢帧的原因有两个：GPU一次工作超过16ms、或CPU一次工作超过16ms
GPU方面尽量减少层级、过渡绘制原因，这个系列就不多说GPU的操作哈
CPU方面尽量减少在UI线程做太多的耗时工作，所以多线程优化是这次系列想描述的方面
+ IntentService
适合由UI触发的后台Service任务，可以通过一定的机制反馈给UI
+ AsyncTask
提供主线程可以快速切换到异步线程的异步机制，适合短暂切换异步执行的场景
+ HandlerThread
提供了线程任务的调度机制（looper+handler），用于专属的线程执行某些特定的任务
+ ThreadPool
线程池，适合并发处理任务的机制


## 参考文献
+ [关于Android中的16ms的问题](https://www.jianshu.com/p/02800806356c)
+ [脑洞大开：为啥帧率达到 60 fps 就流畅？](https://www.jianshu.com/p/71cba1711de0)
