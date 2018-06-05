# RunLoop 與 Thread

對於 Thread， iOS 在開發時可以使用兩種物件:

1. pthread\_t
2. NSThread

曾經 Apple 有文件說明 NSThread 為 pthread\_t 的封裝，但該文件已失效。現在可能都是直接從最底層的 mach thread 封裝。目前並沒有這兩種物件轉換的接口，但肯定的是，**pthread\_t 和 NSThread 是一一對應的。**

比如: 

1. **獲取 main thread:**  可以通過 pthread\_main\_thread\_np\(\) 或 \[NSThread mainThread\] / Thread.main 來取得。
2. **獲取 current thread:** 可以通過 pthread\_self\(\) 或 \[NSThread currentThread\] / Thread.current 來取得

而 CFRunLoop 是基於 pthread 來管理的。



Apple 是不允許直接創建 RunLoop 的，只提供兩個自動獲取的函數:

1. CFRunLoopGetMain\(\)
2. CFRunLoopGetCurrent\(\)

內部邏輯大概是這樣 \(我就直接從 [Source](https://blog.ibireme.com/2015/05/18/runloop/) 拿過來了 ：P \) :

```c
/// 全局的Dictionary，key 是 pthread_t， value 是 CFRunLoopRef
static CFMutableDictionaryRef loopsDic;
/// 访问 loopsDic 时的锁
static CFSpinLock_t loopsLock; 
/// 获取一个 pthread 对应的 RunLoop。
CFRunLoopRef _CFRunLoopGet(pthread_t thread) {
    OSSpinLockLock(&loopsLock);
    if (!loopsDic) {
        // 第一次进入时，初始化全局Dic，并先为主线程创建一个 RunLoop。
        loopsDic = CFDictionaryCreateMutable();
        CFRunLoopRef mainLoop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, pthread_main_thread_np(), mainLoop);
    }
    
    /// 直接从 Dictionary 里获取。
    CFRunLoopRef loop = CFDictionaryGetValue(loopsDic, thread))
    if (!loop) {
        /// 取不到时，创建一个
        loop = _CFRunLoopCreate();
        CFDictionarySetValue(loopsDic, thread, loop);
        /// 注册一个回调，当线程销毁时，顺便也销毁其对应的 RunLoop。
        _CFSetTSD(..., thread, loop, __CFFinalizeRunLoop);
    }       
    
    OSSpinLockUnLock(&loopsLock);
    return loop;
} 

CFRunLoopRef CFRunLoopGetMain() {
    return _CFRunLoopGet(pthread_main_thread_np());
} 

CFRunLoopRef CFRunLoopGetCurrent() {
    return _CFRunLoopGet(pthread_self());
  }

```

代碼可以看出幾點:

1. Thread 和 RunLoop 是一一對應的，關係保存在一個 global Dictionary 裡。 
2. Thread 在創建時並沒有 RunLoop，要直到第一次主動獲取時，才會創建。
3. RunLoop 的銷毀發生在 Thread 結束時。
4. 只能在 Thread 的內部獲取其 RunLoop（Main Thread 除外）。



