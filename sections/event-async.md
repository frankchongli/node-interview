# 事件/非同步

* [`[Basic]` Promise](https://github.com/ElemeFE/node-interview/blob/master/sections/event-async.md#promise)
* [`[Doc]` Events (事件)](https://github.com/ElemeFE/node-interview/blob/master/sections/event-async.md#events)
* [`[Doc]` Timers (定時器)](https://github.com/ElemeFE/node-interview/blob/master/sections/event-async.md#timers)
* [`[Point]` 阻塞/非同步](https://github.com/ElemeFE/node-interview/blob/master/sections/event-async.md#阻塞非同步)
* [`[Point]` 並行/併發](https://github.com/ElemeFE/node-interview/blob/master/sections/event-async.md#並行併發)

## 簡述

非同步還是不非同步? 這是一個問題.

## Promise

![callback-hell](/assets/callback-hell.jpg)

相信很多同學在面試的時候都碰到過這樣一個問題, `如何處理 Callback Hell`. 在早些年的時候, 大家會看到有很多的解決方案例如 [Q](https://www.npmjs.com/package/q), [async](https://www.npmjs.com/package/async), [EventProxy](https://www.npmjs.com/package/eventproxy) 等等. 最後從流行程度來看 `Promise` 當之無愧的獨領風騷, 並且是在 ES6 的 Javascript 標準上贏得了支援.

關於它的基礎知識/概念推薦看阮一峰的 [Promise 物件](http://javascript.ruanyifeng.com/advanced/promise.html#toc9) 這裡就不多不贅述.

> <a name="q-1"></a> Promise 中 .then 的第二參數與 .catch 有什麼區別?

參見 [We have a problem with promises](https://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)

另外關於同步與非同步, 有個問題希望大家看一下, 這是很簡單的 Promise 的使用例子:

```javascript
let doSth = new Promise((resolve, reject) => {
  console.log('hello');
  resolve();
});

doSth.then(() => {
  console.log('over');
});
```

毫無疑問的可以得到一下輸出結果:

```
hello
over
```

但是首先的問題是, 該 Promise 封裝的程式碼肯定是同步的, 那麼這個 then 的執行是非同步的嗎?

其次的問題是, 如下程式碼, `setTimeout` 到 10s 之後再 `.then` 呼叫, 那麼 `hello` 是會在 10s 之後在列印嗎, 還是一開始就列印?

```javascript
let doSth = new Promise((resolve, reject) => {
  console.log('hello');
  resolve();
});

setTimeout(() => {
  doSth.then(() => {
    console.log('over');
  })
}, 10000);
```

以及理解如下程式碼的執行順序 ([出處](https://zhuanlan.zhihu.com/p/25407758)):

```javascript
setTimeout(function() {
  console.log(1)
}, 0);
new Promise(function executor(resolve) {
  console.log(2);
  for( var i=0 ; i<10000 ; i++ ) {
    i == 9999 && resolve();
  }
  console.log(3);
}).then(function() {
  console.log(4);
});
console.log(5);
```

如果你不瞭解這些問題, 可以自己在本地嘗試研究一下列印的結果. 這裡希望你掌握的是 Promise 的狀態轉換, 包括非同步與 Promise 的關係, 以及 Promise 如何幫助你處理非同步, 如果你研究過 Promise 的實現那就更好了.

## Events

`Events` 是 Node.js 中一個非常重要的 core 模組, 在 node 中有許多重要的 core API 都是依賴其建立的. 比如 `Stream` 是基於 `Events` 實現的, 而 `fs`, `net`, `http` 等模組都依賴 `Stream`, 所以 `Events` 模組的重要性可見一斑.

通過繼承 EventEmitter 來使得一個類具有 node 提供的基本的 event 方法, 這樣的物件可以稱作 emitter, 而觸發(emit)事件的 cb 則稱作 listener. 與前端 DOM 樹上的事件並不相同,  emitter 的觸發不存在冒泡, 逐層捕獲等事件行為, 也沒有處理事件傳遞的方法.

> <a name="q-2"></a> Eventemitter 的 emit 是同步還是非同步?

Node.js 中 Eventemitter 的 emit 是同步的. 在官方文件中有說明:

> The EventListener calls all listeners synchronously in the order in which they were registered. This is important to ensure the proper sequencing of events and to avoid race conditions or logic errors.

另外, 可以討論如下的執行結果是輸出 `hi 1` 還是 `hi 2`?

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('myEvent', () => {
  console.log('hi 1');
});

emitter.on('myEvent', () => {
  console.log('hi 2');
});

emitter.emit('myEvent');
```

或者如下情況是否會死迴圈?

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('myEvent', () => {
  console.log('hi');
  emitter.emit('myEvent');
});

emitter.emit('myEvent');
```

以及這樣會不會死迴圈?

```javascript
const EventEmitter = require('events');

let emitter = new EventEmitter();

emitter.on('myEvent', function sth () {
  emitter.on('myEvent', sth);
  console.log('hi');
});

emitter.emit('myEvent');
```

使用 emitter 處理問題可以處理比較複雜的狀態場景, 比如 TCP 的複雜狀態機, 做多項非同步操作的時候每一步都可能報錯, 這個時候 .emit 錯誤並且執行某些 .once 的操作可以將你從泥沼中拯救出來.

另外可以注意一下的是, 有些同學喜歡用 emitter 來監控某些類的狀態, 但是在這些類釋放的時候可能會忘記釋放 emitter, 而這些類的內部可能持有該 emitter 的 listener 的引用從而導致記憶體洩漏.

## 阻塞/非同步

> <a name="q-3"></a> 如何判斷介面是否非同步? 是否只要有回撥函數就是非同步?

開放性問題, 每個寫 node 的人都有一套自己的判斷方式.

* 看文件
* console.log 列印看看
* 看是否有 IO 操作

單純使用回撥函數並不會非同步, IO 操作才可能會非同步, 除此之外還有使用 setTimeout 等方式實現非同步.

> 有這樣一個場景, 你線上上使用 koa 搭建了一個網站, 這個網站項目中有一個你同事寫的介面 A, 而 A 介面中在特殊情況下會變成死迴圈. 那麼首先問題是, 如果觸發了這個死迴圈, 會對網站造成什麼影響?

Node.js 中執行 js 程式碼的過程是單執行緒的. 只有當前程式碼都執行完, 才會切入事件迴圈, 然後從事件佇列中 pop 出下一個回撥函數開始執行程式碼. 所以 ① 實現一個 sleep 函數, 只要通過一個死迴圈就可以阻塞整個 js 的執行流程. (關於如何避免坑爹的同事寫出死迴圈, 在後面的測試環節有寫到.)

> <a name="q-5"></a> 如何實現一個 sleep 函數? ①

```javascript
function sleep(ms) {
  var start = Date.now(), expire = start + ms;
  while (Date.now() < expire) ;
  return;
}
```

而非同步, 是使用 libuv 來實現的 (C/C++的同學可以參見 libev 和 libevent) 另一個執行緒裡的事件佇列.

如果線上上的網站中出現了死迴圈的邏輯被觸發, 整個程序就會一直卡在死迴圈中, 如果沒有多程序部署的話, 之後的網站請求全部會超時, js 程式碼沒有結束那麼事件佇列就會停下等待不會執行非同步, 整個網站無法響應.

> <a name="q-6"></a> 如何實現一個非同步的 reduce? (注:不是非同步完了之後同步 reduce)

需要了解 reduce 的情況, 是第 n 個與 n+1 的結果非同步處理完之後, 在用新的結果與第 n+2 個元素繼續依次非同步下去. 不貼答案, 期待諸君的版本.

## Timers

在筆者這裡將 Node.js 中的非同步簡單的劃分為兩種, 硬非同步和軟非同步.

硬非同步是指由於 IO 操作或者外部呼叫走 libuv 而需要非同步的情況. 當然, 也存在 readFileSync, execSync 等例外情況, 不過 node 由於是單執行緒的, 所以如果常規業務在普通時段執行可能比較耗時同步的 IO 操作會使得其執行過程中其他的所有操作都不能響應, 有點作死的感覺. 不過在啟動/初始化以及一些工具指令碼的應用場景下是完全沒問題的. 而一般的場景下 IO 操作都是需要非同步的.

軟非同步是指, 通過 setTimeout 等方式來實現的非同步. <a name="q-4"></a> 關於 nextTick, setTimeout 以及 setImmediate 三者的區別參見[該帖](https://cnodejs.org/topic/5556efce7cabb7b45ee6bcac)

**Event loop 示例**

```
   ┌───────────────────────┐
┌─>│        timers         │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     I/O callbacks     │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
│  │     idle, prepare     │
│  └──────────┬────────────┘      ┌───────────────┐
│  ┌──────────┴────────────┐      │   incoming:   │
│  │         poll          │<─────┤  connections, │
│  └──────────┬────────────┘      │   data, etc.  │
│  ┌──────────┴────────────┐      └───────────────┘
│  │        check          │
│  └──────────┬────────────┘
│  ┌──────────┴────────────┐
└──┤    close callbacks    │
   └───────────────────────┘
```

關於事件迴圈, Timers 以及 nextTick 的關係詳見官方文件 [The Node.js Event Loop, Timers, and process.nextTick() (英文)](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/) 以及阮一峰的 [JavaScript 執行機制詳解：再談Event Loop (中文)](http://www.ruanyifeng.com/blog/2014/10/event-loop.html) 等.

## 並行/併發

並行 (Parallel) 與併發 (Concurrent) 是兩個很常見的概念.

可以看 Erlang 作者 Joe Armstrong 的部落格 ([Concurrent and Parallel](http://joearms.github.io/2013/04/05/concurrent-and-parallel-programming.html))

![con_and_par](http://joearms.github.io/images/con_and_par.jpg)

併發 (Concurrent) = 2 佇列對應 1 咖啡機.

並行 (Parallel) = 2 佇列對應 2 咖啡機.

Node.js 通過事件迴圈來挨個抽取實踐佇列中的一個個 Task 執行, 從而避免了傳統的多執行緒情況下 `2個佇列對應 1個咖啡機` 的時候上線文切換以及資源爭搶/同步的問題, 所以獲得了高併發的成就.

至於在 node 中並行, 你可以通過 cluster 來再新增一個咖啡機.
