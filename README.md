![ElemeFE-background](assets/ElemeFE-background.png)

# 如何通過餓了麼 Node.js 面試

Hi, 歡迎來到 ElemeFE, 如標題所示本教程的目的是教你如何通過餓了麼大前端的面試, 職位是 2~3 年經驗的 Node.js 服務端程式設計師 (並不是全棧), 如果你對這個職位感興趣或者學習 Node.js 一些進階的內容, 那麼歡迎圍觀.

需要注意的是, 本文針對的並不是零基礎的同學, 你需要有一定的 JavaScript/Node.js 基礎, 並且有一定的工作經驗. 另外本教程的重點更準確的說是服務端基礎中 Node.js 程式設計師需要了解的部分.

如果你覺得大多不瞭解, 就不用投簡歷了 <del>(這樣兩邊都節約了時間)</del>, 如果你覺得大都有了解或者**光看大綱都都覺得很簡單那麼歡迎投遞簡歷至 ElemeFe (fe.job@ele.me)**.

## 導讀

雖然說目的是要通過面試, 但是本教程並不是簡單的把所有面試題列出來, 而**主要是將面試中需要確認你是否懂的點列舉出來**, 並進行一定程度的討論.

本文將一些常見的問題劃分歸類, 每類標明涵蓋的一些`覆蓋點`, 並且列舉幾個`常見問題`, 通常這些問題都是 2~3 年工作經驗需要了解或者面對的. 如果你對某類問題感興趣, 或者想知道其中列舉問題的答案, 可以通過該類下方的 `閱讀更多` 檢視更多的內容.

整體上大綱列舉的並不是很全面, 細節上覆蓋率不高, 很多討論只是點到即止, 希望大家帶著問題去思考.

## [Js 基礎問題](sections/js-basic.md)

> 與前端 Js 不同, 後端是直面伺服器的, 更加偏向記憶體方面.

* [`[Basic]` 類型判斷](sections/js-basic.md#類型判斷)
* [`[Basic]` 作用域](sections/js-basic.md#作用域)
* [`[Basic]` 引用傳遞](sections/js-basic.md#引用傳遞)
* [`[Basic]` 記憶體釋放](sections/js-basic.md#記憶體釋放)
* [`[Basic]` ES6 新特性](sections/js-basic.md#es6-新特性)

### 常見問題

* js 中什麼類型是引用傳遞, 什麼類型是值傳遞? 如何將值類型的變數以引用的方式傳遞? [[more]](sections/js-basic.md#q-value)
* js 中， 0.1 + 0.2 === 0.3 是否為 true ? 在不知道浮點數位數時應該怎樣判斷兩個浮點數之和與第三數是否相等？
* const 定義的 Array 中間元素能否被修改? 如果可以, 那 const 修飾物件的意義是? [[more]](sections/js-basic.md#q-const)
* JavaScript 中不同類型以及不同環境下變數的記憶體都是何時釋放? [[more]](sections/js-basic.md#q-mem)

[閱讀更多](sections/js-basic.md)

## [模組](sections/module.md)

* [`[Basic]` 模組機制](sections/module.md#模組機制)
* [`[Basic]` 熱更新](sections/module.md#熱更新)
* [`[Basic]` 上下文](sections/module.md#上下文)

### 常見問題

* a.js 和 b.js 兩個檔案互相 require 是否會死迴圈? 雙方是否能匯出變數? 如何從設計上避免這種問題? [[more]](sections/module.md#q-loop)
* 如果 a.js require 了 b.js, 那麼在 b 中定義全局變數 `t = 111` 能否在 a 中直接列印出來? [[more]](sections/module.md#q-global)
* 如何在不重啟 node 程序的情況下熱更新一個 js/json 檔案? 這個問題本身是否有問題? [[more]](sections/module.md#q-hot)

[閱讀更多](sections/module.md)

## [事件/非同步](sections/event-async.md)

* [`[Basic]` Promise](sections/event-async.md#promise)
* [`[Doc]` Events (事件)](sections/event-async.md#events)
* [`[Doc]` Timers (定時器)](sections/event-async.md#timers)
* [`[Point]` 阻塞/非同步](sections/event-async.md#阻塞非同步)
* [`[Point]` 並行/併發](sections/event-async.md#並行併發)

### 常見問題

* Promise 中 .then 的第二參數與 .catch 有什麼區別? [[more]](sections/event-async.md#q-1)
* Eventemitter 的 emit 是同步還是非同步? [[more]](sections/event-async.md#q-2)
* 如何判斷介面是否非同步? 是否只要有回撥函數就是非同步? [[more]](sections/event-async.md#q-3)
* nextTick, setTimeout 以及 setImmediate 三者有什麼區別? [[more]](sections/event-async.md#q-4)
* 如何實現一個 sleep 函數? [[more]](sections/event-async.md#q-5)
* 如何實現一個非同步的 reduce? (注:不是非同步完了之後同步 reduce) [[more]](sections/event-async.md#q-6)

[閱讀更多](sections/event-async.md)

## [程序](sections/process.md)

* [`[Doc]` Process (程序)](sections/process.md#process)
* [`[Doc]` Child Processes (子程序)](sections/process.md#child-process)
* [`[Doc]` Cluster (叢集)](sections/process.md#cluster)
* [`[Basic]` 程序間通訊](sections/process.md#程序間通訊)
* [`[Basic]` 守護程序](sections/process.md#守護程序)

### 常見問題

* 程序的當前工作目錄是什麼? 有什麼作用? [[more]](sections/process.md#q-cwd)
* child_process.fork 與 POSIX 的 fork 有什麼區別? [[more]](sections/process.md#q-fork)
* 父程序或子程序的死亡是否會影響對方? 什麼是孤兒程序? [[more]](sections/process.md#q-child)
* cluster 是如何保證負載均衡的? [[more]](sections/process.md#how-it-works)
* 什麼是守護程序? 如何實現守護程序? [[more]](sections/process.md#守護程序)

[閱讀更多](sections/process.md)


## [IO](sections/io.md)

* [`[Doc]` Buffer](sections/io.md#buffer)
* [`[Doc]` String Decoder (字元串解碼)](sections/io.md#string-decoder)
* [`[Doc]` Stream (流)](sections/io.md#stream)
* [`[Doc]` Console (控制檯)](sections/io.md#console)
* [`[Doc]` File System (檔案系統)](sections/io.md#file)
* [`[Doc]` Readline](sections/io.md#readline)
* [`[Doc]` REPL](sections/io.md#repl)

### 常見問題

* Buffer 一般用於處理什麼資料? 其長度能否動態變化? [[more]](sections/io.md#buffer)
* Stream 的 highWaterMark 與 drain 事件是什麼? 二者之間的關係是? [[more]](sections/io.md#緩衝區)
* Stream 的 pipe 的作用是? 在 pipe 的過程中資料是引用傳遞還是拷貝傳遞? [[more]](sections/io.md#pipe)
* 什麼是檔案描述符? 輸入流/輸出流/錯誤流是什麼? [[more]](sections/io.md#file)
* console.log 是同步還是非同步? 如何實現一個 console.log? [[more]](sections/io.md#console)
* 如何同步的獲取使用者的輸入?  [[more]](sections/io.md#如何同步的獲取使用者的輸入)
* Readline 是如何實現的? (有思路即可) [[more]](sections/io.md#readline)

[閱讀更多](sections/io.md)

## [Network](sections/network.md)

* [`[Doc]` Net (網路)](sections/network.md#net)
* [`[Doc]` UDP/Datagram](sections/network.md#udp)
* [`[Doc]` HTTP](sections/network.md#http)
* [`[Doc]` DNS (域名伺服器)](sections/network.md#dns)
* [`[Doc]` ZLIB (壓縮)](sections/network.md#zlib)
* [`[Point]` RPC](sections/network.md#rpc)

### 常見問題

* cookie 與 session 的區別? 服務端如何清除 cookie? [[more]](sections/network.md#q-cookie-session)
* HTTP 協議中的 POST 和 PUT 有什麼區別? [[more]](sections/network.md#q-post-put)
* 什麼是跨域請求? 如何允許跨域? [[more]](sections/network.md#q-cors)
* TCP/UDP 的區別? TCP 粘包是怎麼回事，如何處理? UDP 有粘包嗎? [[more]](sections/network.md#q-tcp-udp)
* `TIME_WAIT` 是什麼情況? 出現過多的 `TIME_WAIT` 可能是什麼原因? [[more]](sections/network.md#q-time-wait)
* ECONNRESET 是什麼錯誤? 如何復現這個錯誤?
* socket hang up 是什麼意思? 可能在什麼情況下出現? [[more]](sections/network.md#socket-hang-up)
* hosts 檔案是什麼? 什麼叫 DNS 本地解析?
* 列舉幾個提高網路傳輸速度的辦法?

[閱讀更多](sections/network.md)

## [OS](sections/os.md)

* [`[Doc]` TTY](sections/os.md#tty)
* [`[Doc]` OS (作業系統)](sections/os.md#os-1)
* [`[Doc]` 命令列參數](sections/os.md#命令列參數)
* [`[Basic]` 負載](sections/os.md#負載)
* [`[Point]` CheckList](sections/os.md#checklist)

### 常見問題

* 什麼是 TTY? 如何判斷是否處於 TTY 環境? [[more]](sections/os.md#tty)
* 不同作業系統的換行符 (EOL) 有什麼區別? [[more]](sections/os.md#os)
* 伺服器負載是什麼概念? 如何檢視負載? [[more]](sections/os.md#負載)
* ulimit 是用來幹什麼的? [[more]](sections/os.md#ulimit)

[閱讀更多](sections/os.md)

## [錯誤處理/偵錯/優化](sections/error.md)

* [`[Doc]` Errors (異常)](sections/error.md#errors)
* [`[Doc]` Domain (域)](sections/error.md#domain)
* [`[Doc]` Debugger (偵錯程式)](sections/error.md#debugger)
* [`[Doc]` C/C++ 外掛](sections/error.md#c-c++-addon)
* [`[Doc]` V8](sections/error.md#v8)
* [`[Point]` 記憶體快照](sections/error.md#記憶體快照)
* [`[Point]` CPU profiling](sections/error.md#cpu-profiling)

### 常見問題

* 怎麼處理未預料的出錯? 用 try/catch ，domains 還是其它什麼? [[more]](sections/error.md#q-handle-error)
* 什麼是 `uncaughtException` 事件? 一般在什麼情況下使用該事件? [[more]](sections/error.md#uncaughtexception)
* domain 的原理是? 為什麼要棄用 domain? [[more]](sections/error.md#domain)
* 什麼是防禦性程式設計? 與其相對的 let it crash 又是什麼?
* 為什麼要在 cb 的第一參數傳 error? 為什麼有的 cb 第一個參數不是 error, 例如 http.createServer?
* 為什麼有些異常沒法根據報錯資訊定位到程式碼呼叫? 如何準確的定位一個異常? [[more]](sections/error.md#錯誤棧丟失)
* 記憶體洩漏通常由哪些原因導致? 如何分析以及定位記憶體洩漏? [[more]](sections/error.md#記憶體快照)

[閱讀更多](sections/error.md)

## [測試](sections/test.md)

* [`[Basic]` 測試方法](sections/test.md#測試方法)
* [`[Basic]` 單元測試](sections/test.md#單元測試)
* [`[Basic]` 整合測試](sections/test.md#整合測試)
* [`[Basic]` 基準測試](sections/test.md#基準測試)
* [`[Basic]` 壓力測試](sections/test.md#壓力測試)
* [`[Doc]` Assert (斷言)](sections/test.md#assert)

### 常見問題

* [為什麼要寫測試? 寫測試是否會拖累開發進度?](sections/test.md#q-why-write-test)
* [單元測試的單元是指什麼? 什麼是覆蓋率?](sections/test.md#單元測試)
* [測試是如何保證業務邏輯中不會出現死迴圈的?](sections/test.md#q-death-loop)
* [mock 是什麼? 一般在什麼情況下 mock?](sections/test.md#mock)

[閱讀更多](sections/test.md)

## util

* `[Doc]` URL
* `[Doc]` Path (路徑)
* `[Doc]` Utilities (實用函數)
* `[Doc]` Query Strings (查詢字元串)
* `[Basic]` 正規表示式

* 如何獲取某個資料夾下所有的檔名?

`更多整理中`

## 儲存

* `[Point]` Sql
* `[Point]` NoSql
* `[Point]` 快取
* `[Point]` 資料一致性

### 常見問題

* 索引有什麼用，大致原理是什麼?設計索引有什麼注意點?
* Session/Cookie 有什麼區別?
* 連線超時有可能是什麼問題導致的?
* 什麼情況下資料會出現髒讀? 如何避免?

`更多整理中`

## 安全

* `[Doc]` HTTPS
* `[Doc]` TLS/SSL
* `[Point]` XSS
* `[Point]` CSRF
* `[Point]` 中間人攻擊
* `[Point]` Sql/Nosql 注入攻擊
* `[Doc]` Crypto (加密)

### 常見問題

* CSRF 的攻擊和防範方法?
* 加密如何保證使用者密碼的安全性?
* 如何避免中間人攻擊?

`更多整理中`

## 最後

目前 repo 處於施工現場的情況，如果發現問題歡迎在 [issues](https://github.com/ElemeFE/node-interview/issues) 中指出。如果有比較好的問題/知識點/指正，也歡迎提 PR。

另外關於 Js 基礎 是個比較大的話題，在本教程不會很細緻深入的討論，更多的是列出一些重要或者跟服務端更相關的地方，所以如果你拿著《JavaScript 權威指南》給教程提 PR 可能不會採納。本教程的重點更準確的說是服務端基礎中 Node.js 程式設計師需要了解的部分。
