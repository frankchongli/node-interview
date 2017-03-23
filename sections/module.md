# 模組

* [`[Basic]` 模組機制](https://github.com/ElemeFE/node-interview/blob/master/sections/module.md#模組機制)
* [`[Basic]` 熱更新](https://github.com/ElemeFE/node-interview/blob/master/sections/module.md#熱更新)
* [`[Basic]` 上下文](https://github.com/ElemeFE/node-interview/blob/master/sections/module.md#上下文)

## 常見問題


> <a name="q-hot"></a> 如何在不重啟 node 程序的情況下熱更新一個 js/json 檔案? 這個問題本身是否有問題?

可以清除掉 `require.cache` 的快取重新 `require(xxx)`, 視具體情況還可以用 VM 模組重新執行.

當然這個問題可能是典型的 [`X-Y Problem`](http://coolshell.cn/articles/10804.html), 使用 js 實現熱更新很容易碰到 v8 優化之後各地拿到快取的引用導致熱更新 js 沒意義. 當然熱更新 json 還是可以簡單一點比如用讀取檔案的方式來熱更新, 但是這樣也不如從 redis 之類的資料庫中讀取比較合理.

## 簡述

其他還有很多內容也是屬於很 '基礎' 的 Node.js 問題 (例如非同步/執行緒等等), 但是由於歸類的問題並沒有放在這個分類中. 所以這裡只簡單講幾個之後沒歸類的基礎問題.

### 模組機制

node 的基礎中毫無疑問的應該是有關於模組機制的方面的, 也即 `require` 這個內建功能的一些原理的問題.

關於模組互相引用之類的, 不瞭解的推薦先好好讀讀[官方文件](https://nodejs.org/dist/latest-v6.x/docs/api/modules.html).

其實官方文件已經說得很清楚了, 每個 node 程序只有一個 VM 的上下文, 不會跟瀏覽器相差多少, 模組機制在文件中也描述的非常清楚了:

```javascript
function require(...) {
  var module = { exports: {} };
  ((module, exports) => {
    // Your module code here. In this example, define a function.
    function some_func() {};
    exports = some_func;
    // At this point, exports is no longer a shortcut to module.exports, and
    // this module will still export an empty default object.
    module.exports = some_func;
    // At this point, the module will now export some_func, instead of the
    // default object.
  })(module, module.exports);
  return module.exports;
}
```

> <a name="q-global"></a> 如果 a.js require 了 b.js, 那麼在 b 中定義全局變數 `t = 111` 能否在 a 中直接列印出來?

① 每個 `.js` 能獨立一個環境只是因為 node 幫你在外層包了一圈自執行, 所以你使用 `t = 111` 定義全局變數在其他地方當然能拿到. 情況如下:

```javascript

// b.js
(function (exports, require, module, __filename, __dirname) {
  t = 111;
})();

// a.js
(function (exports, require, module, __filename, __dirname) {
  // ...
  console.log(t); // 111
})();
```

> <a name="q-loop"></a> a.js 和 b.js 兩個檔案互相 require 是否會死迴圈? 雙方是否能匯出變數? 如何從設計上避免這種問題?

② 不會, 先執行的匯出空物件, 通過匯出工廠函數讓對方從函數去拿比較好避免. 模組在匯出的只是 `var module = { exports: {} };` 中的 exports, 以從 a.js 啟動為例, a.js 還沒執行完 exports 就是 `{}` 在 b.js 的開頭拿到的就是 `{}` 而已.

另外還有非常基礎和常見的問題, 比如 module.exports 和 exports 的區別這裡也能一併解決了 exports 只是 module.exports 的一個引用. 沒看懂可以在細看我以前發的[帖子](https://cnodejs.org/topic/5734017ac3e4ef7657ab1215).

再晉級一點, 眾所周知, node 的模組機制是基於 [`CommonJS`](http://javascript.ruanyifeng.com/nodejs/module.html) 規範的. 對於從前端轉 node 的同學, 如果面試官想問的難一點會考驗關於 [`CommonJS`](http://javascript.ruanyifeng.com/nodejs/module.html) 的一些問題. 比如比較 `AMD`, `CMD`, [`CommonJS`](http://javascript.ruanyifeng.com/nodejs/module.html) 三者的區別, 包括詢問關於 node 中 `require` 的實現原理等.

### 熱更新

從面試官的角度看, `熱更新` 是很多程式常見的問題. 對客戶端而言, 熱更新意味著不用換包, 當然也包含著 md5 校驗/差異更新等複雜問題; 對服務端而言, 熱更新意味著服務不用重啟, 這樣可用性較高<del>同時也優雅和有逼格</del>. 問的過程中可以一定程度的暴露應聘程式設計師的水平.

從 PHP 轉 node 的同學可能會有些想法, 比如 PHP 的程式碼直接刷上去就好了, 並沒有所謂的重啟. 而 node 重啟看起來動作還挺大. 當然這裡面的區別, 主要是與同時有 PHP 與 node 開發經驗的同學可以討論, 也是很好的切入點.

在 Node.js 中做熱更新程式碼, 牽扯到的知識點可能主要是 `require` 會有一個 `cache`, 有這個 `cache` 在, 即使你更新了 `.js` 檔案, 在程式碼中再次 `require` 還是會拿到之前的編譯好快取在 v8 記憶體 (code space) 中的的舊程式碼. 但是如果只是單純的清除掉 `require` 中的 `cache`, 再次 `require` 確實能拿到新的程式碼, 但是這時候很容易碰到各地維持舊的引用依舊跑的舊的程式碼的問題. 如果還要繼續推行這種熱更新程式碼的話, 可能要推翻當前的架構, 從頭開始從新設計一下目前的框架.

不過熱更新 json 之類的配置檔案的話, 還是可以簡單的實現的, 更新 `require` 的 `cache` 可以實現, 不會有持有舊引用的問題, 可以參見我 2 年前寫著玩的[例子](https://www.npmjs.com/package/auto-reload), 但是如果舊的引用一直被持有很容易出現記憶體洩漏, 而要熱更新配置的話, 為什麼不存資料庫? 或者用 `zookeeper` 之類的服務? 通過更新檔案還要再發布一次, 但是存資料庫直接寫個介面配個介面多爽你說是不是?

所以這個問題其實本身其實是值得商榷的, 可能是典型的 [`X-Y Problem`](http://coolshell.cn/articles/10804.html), 不過聊起來確實是可以暴露水平.

### 上下文

如果你已經瞭解 ①② 那麼你也應該瞭解, 對於 Node.js 而言, 正常情況下只有一個上下文, 甚至於內建的很多方面例如 `require` 的實現只是在啟動的時候執行了[內建的函數](https://github.com/nodejs/node/tree/master/lib).

每個單獨的 `.js` 檔案並不意味著單獨的上下文, 在某個 `.js` 檔案中汙染了全局的作用域一樣能影響到其他的地方.

而目前的 Node.js 將 VM 的介面暴露了出來, 可以讓你自己建立一個新的 js 上下文, 這一點上跟前端 js 還是區別挺大的. 在執行外部程式碼的時候, 通過建立新的上下文沙盒 (sandbox) 可以避免上下文被汙染:

```javascript
'use strict';
const vm = require('vm');

let code =
`(function(require) {

  const http = require('http');

  http.createServer( (request, response) => {
    response.writeHead(200, {'Content-Type': 'text/plain'});
    response.end('Hello World\\n');
  }).listen(8124);

  console.log('Server running at http://127.0.0.1:8124/');
})`;

vm.runInThisContext(code)(require);
```

這種執行方式與 eval 和 Function 有明顯的區別. 關於 VM 更多的一些介面可以先閱讀[官方文件 VM (虛擬機器)](https://nodejs.org/dist/latest-v6.x/docs/api/vm.html)

講完這個知識點, 這裡留下一個簡單的問題, 既然可以通過新的上下文來避免汙染, 那麼`為什麼 Node.js 不給每一個 `.js` 檔案以獨立的上下文來避免作用域被汙染?` <del>(反應不過來的同學還是別投簡歷了, 微笑臉)</del>
