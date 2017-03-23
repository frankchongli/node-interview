# Javascript 基礎問題

* [`[Basic]` 類型判斷](https://github.com/ElemeFE/node-interview/blob/master/sections/js-basic.md#類型判斷)
* [`[Basic]` 作用域](https://github.com/ElemeFE/node-interview/blob/master/sections/js-basic.md#作用域)
* [`[Basic]` 引用傳遞](https://github.com/ElemeFE/node-interview/blob/master/sections/js-basic.md#引用傳遞)
* [`[Basic]` 記憶體釋放](https://github.com/ElemeFE/node-interview/blob/master/sections/js-basic.md#記憶體釋放)
* [`[Basic]` ES6 新特性](https://github.com/ElemeFE/node-interview/blob/master/sections/js-basic.md#es6-新特性)


## 簡述

與前端 Js 不同, 後端方面除了SSR/爬蟲之外很少會接觸 DOM, 所以關於 DOM 方面的各種知識基本不會討論. 前端很少碰到記憶體問題, 但是後端幾乎是直面伺服器記憶體的, 更加偏向記憶體方面, 對於一些更基礎的問題也會更加關注.

不過由於 Js 方面的知識點是在太多, 《Javascript 權威指南》的厚度完全可以說明問題, 所以本教程並不會完整的帶大家過一遍 Js 的基礎問題, 只是簡單列舉一些餓了麼在面試 Node.js 程式的時候通常會問的一些 Js 基礎問題, 有的詳細的地方會直接留下書名或者博文連結, 以供大家深入瞭解, 這裡就不贅述了.

> 希望大家更多的是帶著本文拋出的問題去學習, 而不是期待本文把所有答案列出來.

## 類型判斷

Javascript 的類型判斷其實是個挺折磨人的話題, 不然也不會有 Typescript 出現了. 在類型判斷的問題上, 基礎上 推薦閱讀 [lodash](https://github.com/lodash/lodash) 的原始碼.

這類問題一般只是簡單的開場, 不會因為說你不知道 `undefined == null` 的結果是 `true` 就一票否決一個人. 只是根據個人經驗看來，這個問題答不清楚的有不小的概率屬於基礎較差. 如果你對這種問題沒有任何概念, 也許要反思一下是不是該找本書過一下 Js 的基礎了.

另外在這個問題上, 對使用 TypeScript 以及 flow 同學會有一定的加分.

## 作用域

在面試時, 作用域並不是一個很好問的知識點, 一般會問的是 `es6 中 let 與 var 的區別`, 或者列舉程式碼, 然後通過對程式碼的解讀來看你對作用域的掌握比較方便.

印象中那本 [《你不知道的 Javascript》](https://book.douban.com/subject/26351021/) 講的很好了, 有興趣可以去看那本書, 以下是該書的部分目錄:

* 第1章 作用域是什麼
* 第2章 詞法作用域
* 第3章 函數作用域和塊作用域
* 第4章 提升
* 第5章 作用域閉包
* ...

## 引用傳遞

> <a name="q-value"></a> js 中什麼類型是引用傳遞, 什麼類型是值傳遞? 如何將值類型的變數以引用的方式傳遞?

簡單點說, 物件是引用傳遞, 基礎類型是值傳遞, 通過將基礎類型包裝 (boxing) 可以以引用的方式傳遞.(複雜見注①)

引用傳遞和值傳遞是一個非常簡單的問題, 也是理解 Javascript 中的記憶體方面問題的一個基礎. 如果不瞭解引用可能很難去看很多問題.

面試寫程式碼的話, 可以通過 `如何編寫一個 json 物件的拷貝函數` 等類似的問題來考察對引用的瞭解.
不過筆者偶爾會有惡趣味, 喜歡先問應聘者對於 `==` 的 `===` 的區別的瞭解. 然後再問 `[1] == [1]` 是 `true` 還是 `false`. 如果基礎不好的同學可能會被自己對於 `==` 和 `===` 的結論影響然後得出錯誤的結論.

注①: 對於技術好的, 希望能直接反駁這個問題本身是有問題的, 比如講清楚 Javascript 中沒有引用傳遞只是傳遞引用. 參見 [Is JavaScript a pass-by-reference or pass-by-value language?](http://stackoverflow.com/questions/518000/is-javascript-a-pass-by-reference-or-pass-by-value-language). 雖然說是複雜版, 但是這些知識對於 3年經驗的同學真的應該是很簡單的問題了.

另外如果簡歷中有寫 C++, 則必問 `指針與引用的區別`.

## 記憶體釋放

> <a name="q-mem"></a> Javascript 中不同類型以及不同環境下變數的記憶體都是何時釋放?

引用類型是在沒有引用之後, 通過 v8 的 GC 自動回收, 值類型如果是處於閉包的情況下, 要等閉包沒有引用才會被 GC 回收, 非閉包的情況下等待 v8 的新生代 (new space) 切換的時候回收.

與前端 Js 不同, 2年以上經驗的 Node.js 一定要開始注意記憶體了, 不說對 v8 的 GC 有多瞭解, 基礎的記憶體釋放一定有概念了, 並且要開始注意記憶體洩漏的問題了.

你需要了解哪些操作一定會導致記憶體洩漏, 或者可以崩掉記憶體. 比如如下程式碼能否爆掉 V8 的記憶體?

```javascript
let arr = [];
while(true)
  arr.push(1);
```

然後上述程式碼與下方的情況有什麼區別?

```javascript
let arr = [];
while(true)
  arr.push();
```

如果 push 的是 `Buffer` 情況又會有什麼區別?

```javascript
let arr = [];
while(true)
  arr.push(new Buffer(1000));
```

思考完之後可以嘗試找找別的情況如何爆掉 V8 的記憶體. 以及來聊聊記憶體洩漏?

```javascript
var theThing = null
var replaceThing = function () {
  var originalThing = theThing
  var unused = function () {
    if (originalThing)
      console.log("hi")
  }
  theThing = {
    longStr: new Array(1000000).join('*'),
    someMethod: function () {
      console.log(someMessage)
    }
  };
};
setInterval(replaceThing, 1000)
```

比如上述情況中 `unused` 的函數中持有了 `originalThing` 的引用, 使得每次舊的物件不會釋放從而導致記憶體洩漏 (例子出自[《Node.js 垃圾回收》](https://eggggger.xyz/2016/10/22/node-gc/))

當然對於一些高水平的同學, 要求能清楚的瞭解 v8 記憶體 GC 的機制, 懂得記憶體快照等 (之後會在`偵錯/優化`的小結中討論) 了. 比如 V8 中不同類型的資料儲存的位置, 在記憶體釋放的時候不同區域的不同策略等等.

## ES6 新特性

推薦閱讀阮一峰的 [《ECMAScript 6 入門》](http://es6.ruanyifeng.com/)

比較簡單的會問 `let` 與 `var` 的區別, 以及 `箭頭函數` 與 `function` 的區別等等.

深入的話, es6 有太多細節可以深入了. 比如結合 `引用` 的知識點來詢問 `const` 方面的知識. 結合 `{}` 的使用與缺點來談 `Set, Map` 等. 比如私有化的問題與 `symbol` 等等.

其他像是 `閉包是什麼?` 這種問爛了問題已經感覺沒必要問了, 取而代之的是詢問閉包應用的場景更加合理. 比如說, 如果回答者通常使用閉包實現資料的私有, 那麼可以接著問 es6 的一些新特性 (例如 `class`, `symbol`) 能否實現私有, 如果能的話那為什麼要用閉包? 亦或者是什麼閉包中的資料/私有化的資料的記憶體什麼時候釋放? 等等.

`...` 的使用上, 如何實現一個陣列的去重 (使用 Set 可以加分).

> <a name="q-const"></a> const 定義的 Array 中間元素能否被修改? 如果可以, 那 const 修飾物件有什麼意義?

其中的值可以被修改. 意義上, 主要保護引用不被修改 (如用 [Map](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Map) 等介面對引用的變化很敏感, 使用 const 保護引用始終如一是有意義的), 也適合用在 immutable 的場景.

暫時寫上這些, 之後會慢慢整理, 如果內容比較多可能單獨歸一類來討論.
