# RunLoop 與 內部邏輯

首先先來幾張各篇文章都會引用的圖

![From Apple Documentation](.gitbook/assets/ying-mu-kuai-zhao-20180604-xia-wu-2.59.34%20%281%29.png)

上圖是 Apple 官方的示意圖，可以看出 RunLoop 在 Thread 中的作用是從 input source / timer source 接受事件，然後在 Thread 中處理事件。這也驗證 RunLoop 會一對一對應 Thread 的觀念。

![&#x7C21;&#x6613;&#x7248;&#x7684;&#x6D41;&#x7A0B;&#x5716;](.gitbook/assets/runloop_1.png)

內部代碼[在這](https://opensource.apple.com/source/CF/CF-855.17/CFRunLoop.c)。搜尋CFRunLoopRun即可。

![&#x8A73;&#x7D30;&#x7248;&#x672C;](.gitbook/assets/162a5b8323373d82.png)

步驟從以上兩個圖就能知曉，但還有個要理解的地方：

#### Source0 與 Source1

這個概念先前有提到， Source1 比起  Source0 多一個 mach\_port，是基於 port 主動喚醒 RunLoop 的一種 source。在這特別備註兩種 Source 主要 handle 的事件。

Source0 主要處理：

* App 內部事件，如 UIEvent, CFSocket...

Source1 主要處理：

* RunLoop 和 內核管理，如 CFMachPort、CFMessagePort



