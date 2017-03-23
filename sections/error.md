# 錯誤處理/偵錯/優化

* `[Doc]` Errors (異常)
* `[Doc]` Domain (域)
* `[Doc]` Debugger (偵錯程式)
* `[Doc]` C/C++ 外掛
* `[Doc]` V8
* `[Point]` 記憶體快照
* `[Point]` CPU剖析


## Errors

在 Node.js 中的錯誤主要有一下四種類型：

|錯誤|名稱|觸發|
|---|---|---|
|Standard JavaScript errors|標準 JavaScript 錯誤|由錯誤程式碼觸發|
|System errors|系統錯誤|由作業系統觸發|
|User-specified errors|使用者自定義錯誤|通過 throw 拋出|
|Assertion errors|斷言錯誤|由 `assert` 模組觸發|

其中標準的 JavaScript 錯誤常見有：

* [EvalError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/EvalError): 呼叫 eval() 出現錯誤時拋出該錯誤
* [SyntaxError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SyntaxError): 程式碼不符合 JavaScript 語法規範時拋出該錯誤
* [RangeError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RangeError): 陣列越界時拋出該錯誤
* [ReferenceError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/ReferenceError): 引用未定義的變數時拋出該錯誤
* [TypeError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/TypeError): 參數類型錯誤時拋出該錯誤
* [URIError](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/URIError): 誤用全局的 URI 處理函數時拋出該錯誤

而常見的系統錯誤列表可以通過 Node.js 的 os 物件常看列表：

```javascript
const os = require('os');

console.log(os.constants.errno);
```

目前搜尋 Node.js 面試題, 發現很多題目已經跟不上 Node.js 的發展了.比較老的 [NodeJS 錯誤處理最佳實踐](https://cnodejs.org/topic/55714dfac4e7fbea6e9a2e5d), 譯自 Joyent 的官方部落格, 其中有這樣的描述:

> 實際上, `try/catch` 唯一常用的是在 `JSON.parse` 和類似驗證使用者輸入的地方

然而實際上現在在 Node.js 中你已經可以輕鬆的使用 try/catch 去捕獲非同步的異常了. 並且在 Node.js v7.6 之後使用了升級引擎的新版 v8, 舊版中 try/catch 程式碼不能優化的問題也解決了. 所以我們現在再來看

> <a name="q-handle-error"></a> 怎麼處理未預料的出錯? 用 try/catch , domains 還是其它什麼?

在 Node.js 中錯誤處理主要有一下幾種方法:

* callback(err, data) 回撥約定
* throw / try / catch
* EventEmitter 的 error 事件

callback(err, data) 這種形式的錯誤處理起來繁瑣, 並不具備強制性, 目前已經處於僅需要了解, 不推薦使用的情況. 而 domain 模組則是半隻腳踏進棺材了.

1) 感謝 [co](https://github.com/visionmedia/co) 的先河, 現在的你已經簡單的使用 try/catch 保護關鍵的位置, 以 koa 為例, 可以通過中介軟體的形式來進行錯誤處理, 詳見 [Koa error handling](https://github.com/koajs/koa/wiki/Error-Handling). 之後的 async/await 均屬於這種模式.

2) 通過 EventEmitter 的錯誤監聽形式為各大關鍵的物件加上錯誤監聽的回撥. 例如監聽 http server, tcp server 等物件的 `error` 事件以及 process 物件提供的 `uncaughtException` 和 `unhandledRejection` 等等.

3) 使用 Promise 來封裝非同步, 並通過 Promise 的錯誤處理來 handle 錯誤.

4) 如果上述辦法不能起到良好的作用, 那麼你需要學習如何優雅的 [Let It Crash](http://wiki.c2.com/?LetItCrash)

> 為什麼要在 cb 的第一參數傳 error? 為什麼有的 cb 第一個參數不是 error, 例如 http.createServer?

TODO


### 錯誤棧丟失

```javascript
function test() {
  throw new Error('test error');
}

function main() {
  test();
}

main();
```

可以收穫報錯:

```javascript
/data/node-interview/error.js:2
  throw new Error('test error');
  ^

Error: test error
    at test (/data/node-interview/error.js:2:9)
    at main (/data/node-interview/error.js:6:3)
    at Object.<anonymous> (/data/node-interview/error.js:9:1)
    at Module._compile (module.js:570:32)
    at Object.Module._extensions..js (module.js:579:10)
    at Module.load (module.js:487:32)
    at tryModuleLoad (module.js:446:12)
    at Function.Module._load (module.js:438:3)
    at Module.runMain (module.js:604:10)
    at run (bootstrap_node.js:394:7)
```

可以發現報錯的行數, test 函數, main 函數的呼叫關係都在 stack 中清晰的體現.

當你使用 setImmediate 等定時器來設定非同步的時候:

```javascript
function test() {
  throw new Error('test error');
}

function main() {
  setImmediate(() => test());
}

main();

```

我們發現

```javascript
/data/node-interview/error.js:2
  throw new Error('test error');
  ^

Error: test error
    at test (/data/node-interview/error.js:2:9)
    at Immediate.setImmediate (/data/node-interview/error.js:6:22)
    at runCallback (timers.js:637:20)
    at tryOnImmediate (timers.js:610:5)
    at processImmediate [as _immediateCallback] (timers.js:582:5)
```

錯誤棧中僅輸出到 test 函數內呼叫的地方位置, 再往上 main 的呼叫資訊就丟失了. 也就是說如果你的函數呼叫深度比較深的情況下, 你使用非同步呼叫某個函數出錯了的情況下追溯這個非同步的呼叫是一個很困難的事情, 因為其之上的棧都已經丟失了. 如果你用過 [async](https://github.com/caolan/async) 之類的模組, 你還可能發現, 報錯的 stack 會非常的長而且曲折, 光看 stack 很難去定位問題.

這項目不大/作者清楚的情況下不是問題, 但是當項目大起來, 開發人員多起來之後, 這樣追溯錯誤會變得異常痛苦. 關於這個問題, 在上文中提到 [錯誤處理的最佳實踐](https://cnodejs.org/topic/55714dfac4e7fbea6e9a2e5d) 中, 關於 `編寫新函數的具體建議` 那一帶的內容有描述到. 通過使用 [verror](https://www.npmjs.com/package/verror) 這樣的方式, 讓 Error 一層層封裝, 並在每一層將錯誤的資訊一層層的包上, 最後拿到的 Error 直接可以從 message 中獲取用於定位問題的關鍵資訊.

以昨天的資料為準（2017-3-13）各位只要對比一下看看 npm 上上個月 [verror](https://www.npmjs.com/package/verror) 的下載量 `1100w` 比 [express](https://www.npmjs.com/package/express) 的 `1070w` 還高. 應該就能感受到這種寫法有多流行了.

### 防禦性程式設計

錯誤並不可怕, 可怕的是你不去準備應對錯誤————[防禦性程式設計的介紹和技巧](http://blog.jobbole.com/101651/)

### let it crash

[Let It Crash](http://wiki.c2.com/?LetItCrash)

### uncaughtException

當異常沒有被捕獲一路冒泡到 Event Loop 時就會觸發該事件 process 物件上的 `uncaughtException` 事件. 預設情況下, Node.js 對於此類異常會直接將其堆棧跟蹤資訊輸出給 `stderr` 並結束程序, 而為 `uncaughtException` 事件新增監聽可以覆蓋該預設行為, 不會直接結束程序.

```javascript
process.on('uncaughtException', (err) => {
  console.log(`Caught exception: ${err}`);
});

setTimeout(() => {
  console.log('This will still run.');
}, 500);

// Intentionally cause an exception, but don't catch it.
nonexistentFunc();
console.log('This will not run.');
```

#### 合理使用 uncaughtException

`uncaughtException` 的初衷是可以讓你拿到錯誤之後可以做一些回收處理之後再 process.exit. 官方的同志們還曾經討論過要移除該事件 (詳見 [issues](https://github.com/nodejs/node-v0.x-archive/issues/2582))

所以你需要明白 `uncaughtException` 其實已經是非常規手段了, 應儘量避免使用它來處理錯誤. 因為通過該事件捕獲到錯誤後, 並不代表 `你可以愉快的繼續執行 (On Error Resume Next)`. 程式內部存在未處理的異常, 這意味著應用程式處於一種未知的狀態. 如果不能適當的恢復其狀態, 那麼很有可能會觸發不可預見的問題. (使用 domain 會很誇張的加劇這個現象, 併產生新人不能理解的各類幽靈問題)

如果在 `.on` 指定的監聽回撥中報錯不會被捕獲, Node.js 的程序會直接終端並返回一個非零的退出碼, 最後輸出相應的堆棧資訊. 否則, 會出現無限遞迴. 除此之外, 記憶體崩潰/底層報錯等情況也不會被捕獲, **目前猜測**是 v8/C++ 那邊撂擔子不幹了, Node.js 完全插不上話導致的 (TODO 整理到這裡才想起來這個念頭尚未驗證, 如果有空的朋友幫忙驗證下).

所以官方建議的使用 `uncaughtException` 的正確姿勢是在結束程序前使用同步的方式清理已使用的資源 (檔案描述符、控制代碼等) 然後 process.exit.

在 uncaughtException 事件之後執行普通的恢復操作並不安全. 官方建議是另外在專門準備一個 monitor 程序來做健康檢查並通過 monitor 來管理恢復情況, 並在必要的時候重啟 (所以官方是含蓄的提醒各位用 pm2 之類的工具).


### unhandledRejection

當 Promise 被 reject 且沒有繫結監聽處理時, 就會觸發該事件. 該事件對排查和追蹤沒有處理 reject 行為的 Promise 很有用.

該事件的回撥函數接收以下參數：

* `reason` [`<Error>`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Error) | `<any>` 該 Promise 被 reject 的物件 (通常為 Error 物件)
* `p` 被 reject 的 Promise 本身

例如

```javascript
process.on('unhandledRejection', (reason, p) => {
  console.log('Unhandled Rejection at: Promise', p, 'reason:', reason);
  // application specific logging, throwing an error, or other logic here
});

somePromise.then((res) => {
  return reportToUser(JSON.pasre(res)); // note the typo (`pasre`)
}); // no `.catch` or `.then`
```

以下程式碼也會觸發 `unhandledRejection` 事件：

```javascript
function SomeResource() {
  // Initially set the loaded status to a rejected promise
  this.loaded = Promise.reject(new Error('Resource not yet loaded!'));
}

var resource = new SomeResource();
// no .catch or .then on resource.loaded for at least a turn
```

> In this example case, it is possible to track the rejection as a developer error as would typically be the case for other 'unhandledRejection' events. To address such failures, a non-operational `.catch(() => { })` handler may be attached to resource.loaded, which would prevent the 'unhandledRejection' event from being emitted. Alternatively, the 'rejectionHandled' event may be used.


## Domain

Node.js 早期, try/catch 無法捕獲非同步的錯誤, 而錯誤優先的 callback 僅僅是一種約定並沒有強制性並且寫起來十分繁瑣. 所以為了能夠很好的捕獲異常, Node.js 從 v0.8 開始引入 domain 這個模組.

domain 本身是一個 EventEmitter 物件, 其中文意思是 "域" 的意思, 捕獲非同步異常的基本思路是建立一個域, cb 函數會在定義時會繼承上一層的域, 報錯通過當前域的 `.emit('error', err)` 方法觸發錯誤事件將錯誤傳遞上去, 從而使得非同步錯誤可以被強制捕獲. (更多內容詳見 [Node.js 非同步異常的處理與domain模組解析](https://cnodejs.org/topic/516b64596d38277306407936))

但是 domain 的引入也帶來了更多新的問題. 比如依賴的模組無法繼承你定義的 domain, 導致你寫的 domain 無法 cover 依賴模組報錯. 而且, 很多人 (特別是新人) 由於不瞭解 Node.js 的記憶體/非同步流程等問題, 在使用 domain 處理報錯的時候, 沒有做到完善的處理並盲目的讓程式碼繼續走下去, 這很可能導致**項目完全無法維護** (可能出現的問題真是不勝列舉, 各種夢魘...)

該模組目前的情況: [deprecate domains](https://github.com/nodejs/node/issues/66)


## Debugger

![node-js-survey-debug](/assets/node-js-survey-debug.png)

類似 gdb 的命令列下 debug 工具 (上圖中的 build-in debugger), 同時也支援遠端 debug (類似 [node-inspector](https://github.com/node-inspector/node-inspector), 目前處於試驗狀態). 當然, 目前有不少同學覺得 [vscode](https://code.visualstudio.com/) 對 debug 工具整合的比較好.

關於這個 build-in debugger 使用推薦看[官方文件](https://nodejs.org/dist/latest-v6.x/docs/api/debugger.html). 如果要深入一點, 你可能對本文感興趣: [動態修改 NodeJS 程式中的變數值](http://code.oneapm.com/nodejs/2015/06/27/intereference/)


## C/C++ Addon

在 Node.js 中開發 addon 最痛苦的地方莫過於升級 V8 導致的 C/C++ 程式碼不能相容的問題, 這個問題在很早就出現了. 為了解決這個問題前人開了一個叫 [nan](https://github.com/nodejs/nan) 的項目.

要學習 addon 開發, 除了[官方文件](https://nodejs.org/docs/latest/api/addons.html)也推薦閱讀這個: https://github.com/nodejs/node-addon-examples


## V8

這裡並不是介紹 V8, 而是介紹 Node.js 中的 V8 這個模組. 該模組用於開放 Node.js 內建的 V8 引擎的事件和介面. 這些介面由 V8 底層決定, 所以無法保證絕對的穩定性.

|介面|描述|
|---|---|
|v8.getHeapStatistics()|獲取 heap 資訊|
|v8.getHeapSpaceStatistics()|獲取 heap space 資訊|
|v8.setFlagsFromString(string)|動態設定 V8 options|

### v8.setFlagsFromString(string)

該方法用於新增額外的 V8 命令列標誌. 該方法需謹慎使用, 在 VM 啟動後修改配置可能會發生不可預測的行為、崩潰和資料丟失; 或者什麼反應都沒有.

通過 `node --v8-options` 命令可以查詢當前 Node.js 環境中有哪些可用的 V8 options. 此外, 還可以參考非官方維護的一個 [V8 options 列表](https://github.com/thlorenz/v8-flags/blob/master/flags-0.11.md).

用法:

```javascript
// Print GC events to stdout for one minute.
const v8 = require('v8');
v8.setFlagsFromString('--trace_gc');
setTimeout(function() { v8.setFlagsFromString('--notrace_gc'); }, 60e3);
```

## 記憶體快照

記憶體快照常用與解決記憶體洩漏的問題. 快照工具推薦使用 [heapdump](https://github.com/bnoordhuis/node-heapdump) 用來儲存記憶體快照, 使用 [devtool](https://github.com/Jam3/devtool) 來檢視記憶體快照. 使用 heapdump 儲存記憶體快照時, 只會有 Node.js 環境中的物件, 不會受到干擾（如果使用 [node-inspector](https://github.com/node-inspector/node-inspector) 的話, 快照中會有前端的變數干擾）.

使用以及記憶體洩漏的常見原因詳見: [如何分析 Node.js 中的記憶體洩漏](https://zhuanlan.zhihu.com/p/25736931?group_id=825001468703674368).

## CPU profiling

CPU profiling (剖析) 常用於效能優化. 有許多用於做 profiling 的第三方工具, 但是大部分情況下, 使用 Node.js 內建的是最簡單的. 其內建呼叫的就是 [V8 本身的 profiler](https://github.com/v8/v8/wiki/Using%20V8%E2%80%99s%20internal%20profiler), 它可以在程式執行過程中中是對 stack 間隔性的抽樣分析.

使用 `--prof` 開啟內建的 profilling

```shell
node --prof app.js
```

程式執行之後會生成一個 `isolate-0xnnnnnnnnnnnn-v8.log` 在當前執行目錄.

你可以使用 `--prof-process` 來生成報告檢視

```
node --prof-process isolate-0xnnnnnnnnnnnn-v8.log
```

報告形如:

```
Statistical profiling result from isolate-0x103001200-v8.log, (12042 ticks, 2634 unaccounted, 0 excluded).

 [Shared libraries]:
   ticks  total  nonlib   name
     35    0.3%          /usr/lib/system/libsystem_platform.dylib
     27    0.2%          /usr/lib/system/libsystem_pthread.dylib
      7    0.1%          /usr/lib/system/libsystem_c.dylib
      3    0.0%          /usr/lib/system/libsystem_kernel.dylib
      1    0.0%          /usr/lib/system/libsystem_malloc.dylib

 [JavaScript]:
   ticks  total  nonlib   name
    208    1.7%    1.7%  Stub: LoadICStub
    187    1.6%    1.6%  KeyedLoadIC: A keyed load IC from the snapshot
    104    0.9%    0.9%  Stub: VectorStoreICStub
     69    0.6%    0.6%  LazyCompile: *emit events.js:136:44
     68    0.6%    0.6%  Builtin: CallFunction_ReceiverIsNotNullOrUndefined
     65    0.5%    0.5%  KeyedStoreIC: A keyed store IC from the snapshot {2}
     47    0.4%    0.4%  Builtin: CallFunction_ReceiverIsAny
     43    0.4%    0.4%  LazyCompile: *storeHeader _http_outgoing.js:312:21
     34    0.3%    0.3%  LazyCompile: *removeListener events.js:315:28
     33    0.3%    0.3%  Stub: RegExpExecStub
     33    0.3%    0.3%  LazyCompile: *_addListener events.js:210:22
     32    0.3%    0.3%  Stub: CEntryStub
     32    0.3%    0.3%  Builtin: ArgumentsAdaptorTrampoline
     31    0.3%    0.3%  Stub: FastNewClosureStub
     30    0.2%    0.3%  Stub: InstanceOfStub
     ...

 [C++]:
   ticks  total  nonlib   name
    460    3.8%    3.8%  _mach_port_extract_member
    329    2.7%    2.7%  _openat$NOCANCEL
    199    1.7%    1.7%  ___bsdthread_register
    136    1.1%    1.1%  ___mkdir_extended
    116    1.0%    1.0%  node::HandleWrap::Close(v8::FunctionCallbackInfo<v8::Value> const&)
    112    0.9%    0.9%  void v8::internal::BodyDescriptorBase::IterateBodyImpl<v8::internal::StaticScavengeVisitor>(v8::internal::Heap*, v8::internal::HeapObject*, int, int)
    106    0.9%    0.9%  _http_parser_execute
    103    0.9%    0.9%  _szone_malloc_should_clear
     99    0.8%    0.8%  int v8::internal::BinarySearch<(v8::internal::SearchMode)1, v8::internal::DescriptorArray>(v8::internal::DescriptorArray*, v8::internal::Name*, int, int*)
     89    0.7%    0.7%  node::TCPWrap::Connect(v8::FunctionCallbackInfo<v8::Value> const&)
     86    0.7%    0.7%  v8::internal::LookupIterator::State v8::internal::LookupIterator::LookupInRegularHolder<false>(v8::internal::Map*, v8::internal::JSReceiver*)
     ...

 [Bottom up (heavy) profile]:
  Note: percentage shows a share of a particular caller in the total
  amount of its parent calls.
  Callers occupying less than 2.0% are not shown.

   ticks parent  name
   2634   21.9%  UNKNOWN
    764   29.0%    LazyCompile: *connect net.js:815:17
    764  100.0%      LazyCompile: ~<anonymous> net.js:966:30
    764  100.0%        LazyCompile: *_tickCallback internal/process/next_tick.js:87:25
    193    7.3%    LazyCompile: *createWriteReq net.js:732:24
    101   52.3%      LazyCompile: *Socket._writeGeneric net.js:660:42
     99   98.0%        LazyCompile: ~<anonymous> net.js:667:34
     99  100.0%          LazyCompile: ~g events.js:287:13
     99  100.0%            LazyCompile: *emit events.js:136:44
     92   47.7%      LazyCompile: ~Socket._writeGeneric net.js:660:42
     91   98.9%        LazyCompile: ~<anonymous> net.js:667:34
     91  100.0%          LazyCompile: ~g events.js:287:13
     91  100.0%            LazyCompile: *emit events.js:136:44
	 ...
```

|欄位|描述|
|---|---|
|ticks|時間片|
|total|當前操作執行的時間佔總時間的比率|
|nonlib|當前非 System library 執行時間比率|

整理中
