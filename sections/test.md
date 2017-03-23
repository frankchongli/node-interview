# 測試

* [`[Basic]` 測試方法](#測試方法)
* [`[Basic]` 單元測試](#單元測試)
* [`[Basic]` 基準測試](#整合測試)
* [`[Basic]` 整合測試](#基準測試)
* [`[Basic]` 壓力測試](#壓力測試)
* [`[Doc]` Assert (斷言)](#assert)

## 簡述

> <a name="q-why-write-test"></a> 為什麼要寫測試? 寫測試是否會拖累開發進度?

項目在多人合作的時候, 為了某個功能修改了某個模組的某部分程式碼, 實際的情況中修改一個地方可能會影響到別人開發的多個功能, 在自己不知情的情況下想要保證自己修改的程式碼不影響到其他功能, 最簡單的辦法是通過測試來保證.

```
A
  \
    E
  /   \
B       H
  \   /
    F
  /
C
  \
    G
  /
D
```

如上述情況, ABCD 是邏輯層, EFGH 等是更低一次層 (比如工具層等), 當你為了功能 A 的 BUG 修改了 H 的程式碼, 那麼實際受影響的功能除了 A 之外還有 BC, 如果你有針對每一個邏輯的測試, 那麼修改了 H 的程式碼之後, 跑一遍測試即可保證對 H 的修改不會影響到 BC (如果有影響, 那麼相應的測試會報錯). 利用這種特性, 你還可以基於測試去做重構, 在通過原有測試的情況下, 即表明新的重構版本可以替代原有的版本.

而這樣的效果, 只有當覆蓋率達到了一定程度 (通常是 80% 以上, 90% 以上為最理想) 才能實現, 如果測試的覆蓋率低, 無法覆蓋到多種情況, 那麼測試對你的項目可能是沒有用甚至起到反作用的 (讓你誤以為你的修改沒問題而釋出等).

寫測試是否會拖累開發進度要視具體情況而定. 需要考慮到, 開發進度包含功能和品質兩個方面, 單純寫程式碼的速度不能完全代表開發進度. 測試在適當的情況下可以保證項目的品質從而得到更好的開發進度.

如上述的例子, 在修改功能 A 的 BUG 的時候, 如果你不知道 H 會影響到 BC 又沒有測試的話, 那麼開發 BC 的同學可能會出現十分經典的 **"昨天還好好的, 今天怎麼就不能用了?"** 的情況.

當然寫測試拖累開發進度的情況也是客觀存在的, 通常是有以下幾種情況:

* 不會寫測試
* 過度測試, 不必要的測試
* 為了迎合測試, 而忽略了實際需求


> <a name="q-death-loop"></a> 測試是如何保證業務邏輯中不會出現死迴圈的?

你可以通過測試來避免坑爹的同事在某些邏輯中寫出死迴圈, 在通常的測試中加上超時的時間, 在覆蓋率足夠的情況下, 就可以通過跑出超時的測試來排查出現死迴圈以及低效能的情況.


## 測試方法

### 黑盒測試

黑盒測試 (Black-box Testing), 測試應用程式的功能, 而不是其內部結構或運作. 測試者不需瞭解程式碼、內部結構等, 只需知道什麼是應用應該做的事, 即當鍵入特定的輸入, 可得到一定的輸出. 測試者通過選擇`有效輸入`和`無效輸入`來驗證是否正確的輸出. 此測試方法可適合大部分的軟體測試, 例如整合測試 (Integration Testing) 以及系統測試 (System Testing).

### 白盒測試

白盒測試 (White-box Testing) 測試應用程式的內部結構或運作, 而不是測試應用程式的功能 (即黑盒測試). 在白盒測試時, 以程式語言的角度來設計測試案例. 白盒測試可以應用於單元測試 (Unit Testing)、整合測試 (Integration Testing) 和系統的軟體測試流程, 可測試在整合過程中每一單元之間的路徑, 或者主系統跟子系統中的測試.


## 單元測試

單元測試 (Unit Testing) 是白盒測試的一種, 用於針對程式模組進行正確性檢驗的測試工作. 單元 (Unit) 是指**最小可測試的部件**. 在過程化程式設計中, 一個單元就是單個程式、函數、過程等; 對於物件導向程式設計, 最小單元就是方法, 包括基類、抽象類、或者子類中的方法.

另外, 每次修改程式碼之後, 通過單元測試來驗證比把整個應用啟動/重啟驗證要更快/更簡單.

### 覆蓋率

測試覆蓋率 (Test Coverage) 是指程式碼中各項邏輯被測試覆蓋到的比率, 比如 90% 的覆蓋率, 是指程式碼中 90% 的情況都被測試覆蓋到了.

覆蓋率通常由四個維度貢獻:

* 行覆蓋率 (line coverage) 是否每一行都執行了？
* 函數覆蓋率 (function coverage) 是否每個函數都呼叫了？
* 分支覆蓋率 (branch coverage) 是否每個if程式碼塊都執行了？
* 語句覆蓋率 (statement coverage) 是否每個語句都執行了？

常用的測試覆蓋率框架 [istanbul](https://github.com/gotwarlost/istanbul).

當然覆蓋率並不完全是由單元測試貢獻, 在單元測試之上還有整合測試等. 更多關於覆蓋率的內容可以參見[測試覆蓋（率）到底有什麼用?](http://www.infoq.com/cn/articles/test-coverage-rate-role)

### Mock

Mock 主要用於單元測試中. 當一個測試的物件可能依賴其他 (也許複雜/多個) 的物件. 為了確保其行為不受其他物件的影響, 你可以通過模擬其他物件的行為來隔離你要測試的物件.

當你要測試的單元依賴了一些很難納入單元測試的情況時 (例如要測試的單元依賴資料庫/檔案操作/第三方服務 等情況的返回時), 使用 mock 是非常有用的. 簡而言之, Mock 是模擬其他依賴的 behaviour.

Mock 與 Stub 的區別參見: [Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)


### 常見測試工具

* [Mocha](https://github.com/mochajs/mocha)
* [ava](https://github.com/avajs/ava)
* [Jest](https://github.com/facebook/jest)


## 整合測試

整合測試也稱綜合測試、組裝測試、聯合測試, 將程式模組採用適當的整合策略組裝起來, 對系統的介面及整合後的功能進行正確性檢測的測試工作. 整合測試可以是黑盒的, 也可以是白盒的, 其主要目的是檢查軟體單位之間的介面是否正確, 而整合測試的物件是**已經經過單元測試的模組**.

例如你可以在本地將項目中的 web app 啟動, 並模擬介面呼叫:

```javascript
describe('Path API', () => {
  // ...

  describe('GET /v2/path/:_id', () => {
    it('should return 200 GET /v2/path/:_id', () => {
      return request
        .get('/v2/path/' + pathId)
        .set('Cookie', 'common_user=xxx')
        .expect(200);
    });
  });

  describe('POST /v2/path', () => {
    it('should return 412 POST /v2/path lost params path', () => {
      return request
        .post('/v2/path')
        .set('Cookie', 'common_user=xxx')
        .expect(412);
    });

    it('should return 409 POST /v2/path when path exist', () => {
      return request
        .post('/v2/path')
        .send({path: '/'})
        .set('Cookie', 'common_user=xxx')
        .expect(409);
    });

    it('should return 200 POST /v2/path successfully', () => {
      return request
        .post('/v2/path')
        .send({path: '/comment'})
        .set('Cookie', 'common_user=xxx')
        .expect(200);
    });
  });

  // ...
});
```

## 基準測試

目前 Node.js 中流行的白盒級基準測試工具是 [benchmark](https://benchmarkjs.com/docs).

```javascript
const Benchmark = require('benchmark');
const suite = new Benchmark.Suite;

suite.add('RegExp#test', function() {
    /o/.test('Hello World!');
})
.add('String#indexOf', function() {
    'Hello World!'.indexOf('o') > -1;
})
.on('cycle', function(event) {
    console.log(String(event.target));
})
.on('complete', function() {
    console.log('Fastest is ' + this.filter('fastest').map('name'));
})
// run async
.run({ 'async': true });
```

你可以將同一個功能的不同實現基於同一個標準來比較不同實現的速度, 從而得到最優解.

黑盒級別的基準測試, 則推薦 [Apache ab](https://httpd.apache.org/docs/2.4/programs/ab.html) 以及 [wrk](https://github.com/wg/wrk) 等, 例如執行:

```
ab -n 100 -c 10 https://ele.me/
```

可以得到如下的詳細資料:

```
Server Software:        Tengine/2.1.1
Server Hostname:        ele.me
Server Port:            443
SSL/TLS Protocol:       TLSv1.2,ECDHE-RSA-AES256-GCM-SHA384,2048,256

Document Path:          /
Document Length:        284 bytes

Concurrency Level:      10
Time taken for tests:   1.775 seconds
Complete requests:      100
Failed requests:        0
Non-2xx responses:      100
Total transferred:      62400 bytes
HTML transferred:       28400 bytes
Requests per second:    56.33 [#/sec] (mean)
Time per request:       177.511 [ms] (mean)
Time per request:       17.751 [ms] (mean, across all concurrent requests)
Transfer rate:          34.33 [Kbytes/sec] received

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:       88  116  26.0    104     234
Processing:    33   55  39.6     47     394
Waiting:       33   54  39.0     46     394
Total:        124  171  48.1    152     491

Percentage of the requests served within a certain time (ms)
  50%    152
  66%    184
  75%    193
  80%    199
  90%    224
  95%    242
  98%    288
  99%    491
 100%    491 (longest request)
```

與前者相比, ab 等工具可以設定規模以及併發情況. 在比規模不大/需求不復雜的情況下, ab 以及 wrk 也可以用於做壓力測試.


## 壓力測試

壓力測試 (Stress testing), 是保證系統穩定性的一種測試方法. 通過預估系統所需要承載的 QPS, TPS 等指標, 然後通過如 [Jmeter](http://jmeter.apache.org/) 等壓測工具模擬相應的請求情況, 來驗證當前應能能否達到目標.

對於比較重要, 流量較高或者後期業務量會持續增長的系統, 進行壓力測試是保證項目品質的重要環節. 常見的如負載是否均衡, 頻寬是否合理, 以及磁碟 IO 網路 IO 等問題都可以通過比較極限的壓力測試暴露出來.


## Assert

斷言 (Assert) 是快速判斷並對不符合預期的情況進行報錯的模組. 是將:

```javascript
if (condition) {
  throw new Error('Sth wrong');
}
```

寫成:

```javascript
assert(!condition, 'Sth wrong');
```

等等情況的一種簡化. 並且提供了豐富了 `equal` 判斷, 對於物件類型也有深度/嚴格判斷等情況支援.

Node.js 中內建的 `assert` 模組也是屬於斷言模組的一種, 但是官方在文件中有註明, 該內建模組主要是用於內建程式碼編寫時的基本斷言需求, 並不是一個通用的斷言庫 (**not intended to be used as a general purpose assertion library**)

### 常見斷言工具

* [Chai](https://github.com/chaijs/chai)
* [should.js](https://github.com/shouldjs/should.js)
