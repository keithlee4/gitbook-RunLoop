# RunLoop 與 Public API

在 CoreFoundation 裡關於 RunLoop 有五種類別:

1. CFRunLoopRef
2. CFRunLoopModeRef \(沒有對外接口，只從 CFRunLoopRef 接口進行封裝\)
3. CFRunLoopSourceRef
4. CFRunLoopTimerRef
5. CFRunLoopObserverRef

他們的關係如下:

![](.gitbook/assets/runloop_0.png)

一個 RunLoop 裡包含多個 Mode，每個 Mode 包含多個 Source/Timer/Observer。 每次調用 RunLoop 的主函數時，**只能指定一個 Mode，該 Mode 被稱作 CurrentMode。如果需要切換 Mode，只能退出 Loop， 再重新指定一個 Mode 進入。這樣做主要是為了分隔不同組別的 Source/Timer/Observer，避免互相干擾。**

#### **CFRunLoopSourceRef** 

產生事件的地方。Source有兩種，Source 0 與 Source 1:

1. Source 0: 包含一個回調 \(函數指針\)，不能主動觸發事件。使用時需要先調用 CFRunLoopSourceSignal\(source\)，將這個 Source 標記為待處理，然後手動調用 CFRunLoopWakeUp\(runloop\) 喚醒 RunLoop 來處理該事件。
2. Source 1: 包含一個回調 \(函數指針\)，**以及一個 mach\_port，用於通過內核和其他 Thread 相互發送消息。 能主動喚醒 RunLoop。原理之後會提到。**

#### CFRunLoopTimerRef

一個時間觸發器，和 NSTimer 為 toll-free bridged \(表示可以混用\)。 包含一個時間長度和一個回調 \(函數指針\)。加入到 RunLoop 時會註冊對應的時間點，時間到 RunLoop 會被喚醒執行回調。

#### CFRunLoopObserverRef

觀察者。每個 Observer 包含一個回調 \(函數指針\)，當 RunLoop 的狀態發生變化時，觀察者能通過回調接受到這個變化。可以觀測的時間點包含

```c
typedef CF_OPTIONS(CFOptionFlags, CFRunLoopActivity) {
    kCFRunLoopEntry         = (1UL << 0), // 即將進入Loop
    kCFRunLoopBeforeTimers  = (1UL << 1), // 即將處理 Timer
    kCFRunLoopBeforeSources = (1UL << 2), // 即將處理 Source
    kCFRunLoopBeforeWaiting = (1UL << 5), // 即將進入休眠
    kCFRunLoopAfterWaiting  = (1UL << 6), // 剛從休眠中喚醒
    kCFRunLoopExit          = (1UL << 7), // 即將退出Loop
}
```

上述的 Source/Timer/Observer 統稱為 **mode item。**

#### Mode Item

一個 Item 可以被同時加入多個 mode，但同個 item 重複加入同個 mode 是不會有效果的。 如果 RunLoop 當前的 mode 中 一個 item 都沒有，RunLoop 會直接退出，不再進入循環。

