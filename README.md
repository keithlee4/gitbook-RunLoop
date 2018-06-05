---
description: 關於 iOS RunLoop 的整理、理解與實踐
---

# 理解 RunLoop

內容主要由這篇 [深入理解 RunLoop](https://blog.ibireme.com/2015/05/18/runloop/) 的文章整理而得。

包含個人的理解以及筆記，所以目前的架構也照著這篇文章走，部分會額外加上一些自己的反饋。



另外也參考了以下的 Source:

* [Apple Documentation](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)
* [KKBox 基本教材](https://zonble.gitbooks.io/kkbox-ios-dev/responder/run_loop.html)
* [iOS RunLoop 进阶](https://www.jianshu.com/p/2c067bdc7e47)
* [iOS中多线程原理与runloop介绍](http://mobile.51cto.com/iphone-386596.htm)
* [iOS刨根问底-深入理解RunLoop](https://www.cnblogs.com/kenshincui/p/6823841.html)
* [老司机出品——源码解析之RunLoop详解](https://www.jianshu.com/p/ec629063390f)

