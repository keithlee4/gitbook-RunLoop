# RunLoop 與 概念

先談到 [Event Loops](https://en.wikipedia.org/wiki/Event_loop) 這個 Pattern. 通常一個 thread 只執行一個 task, 執行完成後就會退出。 但既然它要是個 Loop, 我們就需要一個能讓 thread **「隨時處理事件」**的機制。而這機制通常會藉由類似這樣的代碼實現:

{% code-tabs %}
{% code-tabs-item title="Event Loop" %}
```cpp
function loop() {
    initialized();
    do {
        var msg = get_next_msg();
        process_msg(msg);
    }while (msg != quit);
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}

其實就是個簡單的 do-while，在接受到事件完結的 msg 之前，while 都不會返回。實現這種模型的關鍵點在於 **事件的管理**，也就是如何讓 thread 在沒有消息時休眠避免佔資源，以及在有消息時立刻被喚醒。**這種等待屬於「閒等待」，和一般的 while loop 不同，不會使 CPU 進入「忙等待」狀態。**

在 iOS/OSX 裡的 RunLoop 其實就是一個物件，它管理其需要處理的事件，並且提供入口函數來執行上述的 Event Loop 的邏輯。 Thread執行該函數後，就會一直處於該函數內部的 「接受消息 -&gt; 等待 -&gt; 處理」的循環裡。 直到循環結束 （像是傳入 quit \)，函數返回。

在 iOS/OSX 裡提供了兩個 RunLoop 相關的物件: 

1. **CFRunLoopRef:** **屬於 CoreFoundation 框架，提供的是 C 的 API，所有 API 都是 thread-safe。** 順道一提是[開源](https://opensource.apple.com/source/CF/CF-855.17/CFRunLoop.c)的。Swift 開源後 Apple 釋出了一個跨平台的 CoreFoundation 原始碼，你可以[在此](https://github.com/apple/swift-corelibs-foundation/blob/master/CoreFoundation/RunLoop.subproj/CFMachPort.c)找到。
2. **NSRunLoop: 是 CFRunLoopRef 的封裝，提供了 OO 的 API，但不是 thread-safe 的。**



