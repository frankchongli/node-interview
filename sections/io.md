# IO

* [`[Doc]` Buffer](https://github.com/ElemeFE/node-interview/blob/master/sections/io.md#buffer)
* [`[Doc]` String Decoder (字元串解碼)](https://github.com/ElemeFE/node-interview/blob/master/sections/io.md#string-decoder)
* [`[Doc]` Stream (流)](https://github.com/ElemeFE/node-interview/blob/master/sections/io.md#stream)
* [`[Doc]` Console (控制檯)](https://github.com/ElemeFE/node-interview/blob/master/sections/io.md#console)
* [`[Doc]` File System (檔案系統)](https://github.com/ElemeFE/node-interview/blob/master/sections/io.md#file)
* [`[Doc]` Readline](https://github.com/ElemeFE/node-interview/blob/master/sections/io.md#readline)
* [`[Doc]` REPL](https://github.com/ElemeFE/node-interview/blob/master/sections/io.md#repl)

# 簡述

Node.js 是以 IO 密集型業務著稱. 那麼問題來了, 你真的瞭解什麼叫 IO, 什麼又叫 IO 密集型業務嗎?

## Buffer

Buffer 是 Node.js 中用於處理二進位制資料的類, 其中與 IO 相關的操作 (網路/檔案等) 均基於 Buffer. Buffer 類的例項非常類似整數陣列, ***但其大小是固定不變的***, 並且其記憶體在 V8 堆棧外分配原始記憶體空間. Buffer 類的例項建立之後, 其所佔用的記憶體大小就不能再進行調整.

在 Node.js v6.x 之後 `new Buffer()` 介面開始被廢棄, 理由是參數類型不同會返回不同類型的 Buffer 物件, 所以當開發者沒有正確校驗參數或沒有正確初始化 Buffer 物件的內容時, 以及不瞭解的情況下初始化  就會在不經意間向程式碼中引入安全性和可靠性問題.

介面|用途
---|---
Buffer.from()|根據已有資料生成一個 Buffer 物件
Buffer.alloc()|建立一個初始化後的 Buffer 物件
Buffer.allocUnsafe()|建立一個未初始化的 Buffer 物件

### TypedArray

Node.js 的 Buffer 在 ES6 增加了 TypedArray 類型之後, 修改了原來的 Buffer 的實現, 選擇基於 TypedArray 中 Uint8Array 來實現, 從而提升了一波效能.

使用上, 你需要了解如下情況:

```javascript
const arr = new Uint16Array(2);
arr[0] = 5000;
arr[1] = 4000;

const buf1 = Buffer.from(arr); // 拷貝了該 buffer
const buf2 = Buffer.from(arr.buffer); // 與該陣列共享了記憶體

console.log(buf1);
// 輸出: <Buffer 88 a0>, 拷貝的 buffer 只有兩個元素
console.log(buf2);
// 輸出: <Buffer 88 13 a0 0f>

arr[1] = 6000;
console.log(buf1);
// 輸出: <Buffer 88 a0>
console.log(buf2);
// 輸出: <Buffer 88 13 70 17>
```

## String Decoder

字元串解碼器 (String Decoder) 是一個用於將 Buffer 拿來 decode 到 string 的模組, 是作為 Buffer.function function toString() { [native code] }() { [native code] } 的一個補充, 它支援多位元組 UTF-8 和 UTF-16 字元. 例如

```javascript
const StringDecoder = require('string_decoder').StringDecoder;
const decoder = new StringDecoder('utf8');

const cent = Buffer.from([0xC2, 0xA2]);
console.log(decoder.write(cent)); // ¢

const euro = Buffer.from([0xE2, 0x82, 0xAC]);
console.log(decoder.write(euro)); // €
```

當然也可以斷斷續續的處理.

```javascript
const StringDecoder = require('string_decoder').StringDecoder;
const decoder = new StringDecoder('utf8');

decoder.write(Buffer.from([0xE2]));
decoder.write(Buffer.from([0x82]));
console.log(decoder.end(Buffer.from([0xAC])));  // €
```

## Stream

Node.js 內建的 `stream` 模組是多個核心模組的基礎. 但是流 (stream) 是一種很早之前流行的程式設計方式. 可以用大家比較熟悉的 C語言來看這種流式操作:

```c

int copy(const char *src, const char *dest)
{
    FILE *fpSrc, *fpDest;
    char buf[BUF_SIZE] = {0};
    int lenSrc, lenDest;

    // 開啟要 src 的檔案
    if ((fpSrc = fopen(src, "r")) == NULL)
    {
        printf("檔案 '%s' 無法開啟\n", src);
        return FAILURE;
    }

    // 開啟 dest 的檔案
    if ((fpDest = fopen(dest, "w")) == NULL)
    {
        printf("檔案 '%s' 無法開啟\n", dest);
        fclose(fpSrc);
        return FAILURE;
    }

    // 從 src 中讀取 BUF_SIZE 長的資料到 buf 中
    while ((lenSrc = fread(buf, 1, BUF_SIZE, fpSrc)) > 0)
    {
        // 將 buf 中的資料寫入 dest 中
        if ((lenDest = fwrite(buf, 1, lenSrc, fpDest)) != lenSrc)
        {
            printf("寫入檔案 '%s' 失敗\n", dest);
            fclose(fpSrc);
            fclose(fpDest);
            return FAILURE;
        }
        // 寫入成功後清空 buf
        memset(buf, 0, BUF_SIZE);
    }

    // 關閉檔案
    fclose(fpSrc);
    fclose(fpDest);
    return SUCCESS;
}
```

應用的場景很簡單, 你要拷貝一個 20G 大的檔案, 如果你一次性將 20G 的資料讀入到記憶體, 你的記憶體條可能不夠用, 或者嚴重影響效能. 但是你如果使用一個 1MB 大小的快取 (buf) 每次讀取 1Mb, 然後寫入 1Mb, 那麼不論這個檔案多大都只會佔用 1Mb 的記憶體.

而在 Node.js 中, 原理與上述 C 程式碼類似, 不過在讀寫的實現上通過 libuv 與 EventEmitter 加上了非同步的特性. 在 linux/unix 中你可以通過 `|` 來感受到流式操作.

### Stream 的類型


類|使用場景|重寫方法
---|---|---
[Readable](https://github.com/substack/stream-handbook#readable-streams)|只讀|_read
[Writable](https://github.com/substack/stream-handbook#writable-streams)|只寫|_write
[Duplex](https://github.com/substack/stream-handbook#duplex)|讀寫|_read, _write
[Transform](https://github.com/substack/stream-handbook#transform)|操作被寫入資料, 然後讀出結果|_transform, _flush


### 物件模式

通過 Node API 建立的流, 只能夠對字元串或者 buffer 物件進行操作. 但其實流的實現是可以基於其他的 Javascript 類型(除了 null, 它在流中有特殊的含義)的. 這樣的流就處在 "物件模式(objectMode)" 中.
在建立流物件的時候, 可以通過提供 `objectMode` 參數來生成物件模式的流. 試圖將現有的流轉換為物件模式是不安全的.

### 緩衝區

Node.js 中 stream 的緩衝區, 以開頭的 C語言 拷貝檔案的程式碼為模板討論, (拋開非同步的區別看) 則是從 `src` 中讀出資料到 `buf` 中後, 並沒有直接寫入 `dest` 中, 而是先放在一個比較大的緩衝區中, 等待寫入(消費) `dest` 中. 即, 在緩衝區的幫助下可以使讀與寫的過程分離.

Readable 和 Writable 流都會將資料儲存在內部的緩衝區中. 緩衝區可以分別通過 `writable._writableState.getBuffer()` 和 `readable._readableState.buffer` 來訪問. 緩衝區的大小, 由構造 stream 時候的 `highWaterMark` 標誌指定可容納的 byte 大小, 對於 `objectMode` 的 stream, 該標誌表示可以容納的物件個數.

#### 可讀流

當一個可讀例項呼叫 `stream.push()` 方法的時候, 資料將會被推入緩衝區. 如果資料沒有被消費, 即呼叫 `stream.read()` 方法讀取的話, 那麼資料會一直留在緩衝佇列中. 當緩衝區中的資料到達 `highWaterMark` 指定的閾值, 可讀流將停止從底層汲取資料, 直到當前緩衝的報備成功消耗為止.

#### 可寫流

在一個在可寫例項上不停地呼叫 writable.write(chunk) 的時候資料會被寫入可寫流的緩衝區. 如果當前緩衝區的緩衝的資料量低於 `highWaterMark` 設定的值, 呼叫 writable.write() 方法會返回 true (表示資料已經寫入緩衝區), 否則當緩衝的資料量達到了閾值, 資料無法寫入緩衝區 write 方法會返回 false, 直到 drain 事件觸發之後才能繼續呼叫 write 寫入.

```javascript
// Write the data to the supplied writable stream one million times.
// Be attentive to back-pressure.
function writeOneMillionTimes(writer, data, encoding, callback) {
  let i = 1000000;
  write();
  function write() {
    var ok = true;
    do {
      i--;
      if (i === 0) {
        // last time!
        writer.write(data, encoding, callback);
      } else {
        // see if we should continue, or wait
        // don't pass the callback, because we're not done yet.
        ok = writer.write(data, encoding);
      }
    } while (i > 0 && ok);
    if (i > 0) {
      // had to stop early!
      // write some more once it drains
      writer.once('drain', write);
    }
  }
}
```

#### Duplex 與 Transform

Duplex 流和 Transform 流都是同時可讀寫的, 他們會在內部維持兩個緩衝區, 分別對應讀取和寫入, 這樣就可以允許兩邊同時獨立操作, 維持高效的資料流. 比如說 net.Socket 是一個 Duplex 流, Readable 端允許從 socket 獲取、消耗資料, Writable 端允許向 socket 寫入資料. 資料寫入的速度很有可能與消耗的速度有差距, 所以兩端可以獨立操作和緩衝是很重要的.

### pipe

stream 的 `.pipe()`, 將一個可寫流附到可讀流上, 同時將可寫流切換到流模式, 並把所有資料推給可寫流. 在 pipe 傳遞資料的過程中, `objectMode` 是傳遞引用, 非 `objectMode` 則是拷貝一份資料傳遞下去.

pipe 方法最主要的目的就是將資料的流動緩衝到一個可接受的水平, 不讓不同速度的資料來源之間的差異導致記憶體被佔滿. 關於 pipe 的實現參見 David Cai 的 [通過源碼解析 Node.js 中導流（pipe）的實現](https://cnodejs.org/topic/56ba030271204e03637a3870)

## Console

[console.log 正常情況下是非同步的, 除非你使用 `new Console(stdout[, stderr])` 指定了一個檔案為目的地](https://nodejs.org/dist/latest-v6.x/docs/api/console.html#console_asynchronous_vs_synchronous_consoles). 不過一般情況下的實現都是如下 ([6.x 原始碼](https://github.com/nodejs/node/blob/v6.x/lib/console.js#L42)):

```javascript
// As of v8 5.0.71.32, the combination of rest param, template string
// and .apply(null, args) benchmarks consistently faster than using
// the spread operator when calling util.format.
Console.prototype.log = function(...args) {
  this._stdout.write(`${util.format.apply(null, args)}\n`);
};
```

自己實現一個 console.log 可以參考如下程式碼:

```javascript
let print = (str) => process.stdout.write(str + '\n');

print('hello world');
```

注意: 該程式碼並沒有處理多參數, 也沒有處理佔位符 (即 util.format 的功能).

### console.log.bind(console) 問題

```javascript
// 源碼出處 https://github.com/nodejs/node/blob/v6.x/lib/console.js
function Console(stdout, stderr) {
  // ... init ...

  // bind the prototype functions to this Console instance
  var keys = Object.keys(Console.prototype);
  for (var v = 0; v < keys.length; v++) {
    var k = keys[v];
    this[k] = this[k].bind(this);
  }
}
```

## File

“一切皆是檔案”是 Unix/Linux 的基本哲學之一, 不僅普通的檔案、目錄、字元裝置、塊裝置、套接字等在 Unix/Linux 中都是以檔案被對待, 也就是說這些資源的操作物件均為 fd (檔案描述符), 都可以通過同一套 system call 來讀寫. 在 linux 中你可以通過 ulimit 來對 fd 資源進行一定程度的管理限制.

Node.js 封裝了標準 POSIX 檔案 I/O 操作的集合. 通過 require('fs') 可以載入該模組. 該模組中的所有方法都有非同步執行和同步執行兩個版本. 你可以通過 fs.open 獲得一個檔案的檔案描述符.

### 編碼

// TODO

UTF8, GBK, es6 中對編碼的支援, 如何計算一個漢字的長度

BOM

### stdio

stdio (standard input output) 標準的輸入輸出流, 即輸入流 (stdin), 輸出流 (stdout), 錯誤流 (stderr) 三者. 在 Node.js 中分別對應 `process.stdin` (Readable), `process.stdout` (Writable) 以及 `process.stderr` (Writable) 三個 stream.

輸出函數是每個人在學習任何一門程式語言時所需要學到的第一個函數. 例如 C語言的 `printf("hello, world!");` python/ruby 的 `print 'hello, world!'` 以及 Javascript 中的 `console.log('hello, world!');`

以 C語言的虛擬碼來看的話, 這類輸出函數的實現思路如下:

```c
int printf(FILE *stream, 要列印的內容)
{
  // ...

  // 1. 申請一個臨時記憶體空間
  char *s = malloc(4096);

  // 2. 處理好要列印的的內容, 其值儲存在 s 中
  //      ...

  // 3. 將 s 上的內容寫入到 stream 中
  fwrite(s, stream);

  // 4. 釋放臨時空間
  free(s);

  // ...
}
```

我們需要了解的是第 3 步, 其中的 stream 則是指 stdout (輸出流). 實際上在 shell 上執行一個應用程式的時候, shell 做的第一個操作是 fork 當前 shell 的程序 (所以, 如果你通過 ps 去檢視你從 shell 上啟動的程序, 其父程序 pid 就是當前 shell 的 pid), 在這個過程中也把 shell 的 stdio 繼承給了你當前的應用程序, 所以你在當前程序裡面將資料寫入到 stdout, 也就是寫入到了 shell 的 stdout, 即在當前 shell 上顯示了.

輸入也是同理, 當前程序繼承了 shell 的 stdin, 所以當你從 stdin 中讀取資料時, 其實就獲取到你在 shell 上輸入的資料. (PS: shell 可以是 windows 下的 cmd, powershell, 也可以是 linux 下 bash 或者 zsh 等)

當你使用 ssh 在遠端伺服器上執行一個命令的時候, 在伺服器上的命令輸出雖然也是寫入到伺服器上 shell 的 stdout, 但是這個遠端的 shell 是從 sshd 服務上 fork 出來的, 其 stdout 是繼承自 sshd 的一個 fd, 這個 fd 其實是個 socket, 所以最終其實是寫入到了一個 socket 中, 通過這個 socket 傳輸你本地的計算機上的 shell 的 stdout.

如果你理解了上述情況, 那麼你也就能理解為什麼守護程序需要關閉 stdio, 如果切到後臺的守護程序沒有關閉 stdio 的話, 那麼你在用 shell 操作的過程中, 螢幕上會莫名其妙的多出來一些輸出. 此處對應[守護程序](https://github.com/ElemeFE/node-interview/blob/master/sections/process.md#守護程序)的 C 實現中的這一段:

```c
for (; i < getdtablesize(); ++i) {
   close(i);  // 關閉開啟的 fd
}
```

Linux/unix 的 fd 都被設計為整型數字, 從 0 開始. 你可以嘗試執行如下程式碼檢視.

```
console.log(process.stdin.fd); // 0
console.log(process.stdout.fd); // 1
console.log(process.stderr.fd); // 2
```

在上一節中的 [在 IPC 通道建立之前, 父程序與子程序是怎麼通訊的? 如果沒有通訊, 那 IPC 是怎麼建立的?](https://github.com/ElemeFE/node-interview/blob/master/sections/process.md#q-child) 中使用環境變數傳遞 fd 的方法, 這麼看起來就很直白了, 因為傳遞 fd 其實是直接傳遞了一個整型數字.

### 如何同步的獲取使用者的輸入?

如果你理解了上述的內容, 那麼放到 Node.js 中來看, 獲取使用者的輸入其實就是讀取 Node.js 程序中的輸入流 (即 process.stdin 這個 stream) 的資料.

而要同步讀取, 則是不用非同步的 read 介面, 而是用同步的 readSync 介面去讀取 stdin 的資料即可實現. 以下來自萬能的 stackoverflow:

```javascript
/*
 * http://stackoverflow.com/questions/3430939/node-js-readsync-from-stdin
 * @mklement0
 */
var fs = require('fs');

var BUFSIZE = 256;
var buf = new Buffer(BUFSIZE);
var bytesRead;

module.exports = function() {
  var fd = ('win32' === process.platform) ? process.stdin.fd : fs.openSync('/dev/stdin', 'rs');
  bytesRead = 0;

  try {
    bytesRead = fs.readSync(fd, buf, 0, BUFSIZE);
  } catch (e) {
    if (e.code === 'EAGAIN') { // 'resource temporarily unavailable'
      // Happens on OS X 10.8.3 (not Windows 7!), if there's no
      // stdin input - typically when invoking a script without any
      // input (for interactive stdin input).
      // If you were to just continue, you'd create a tight loop.
      console.error('ERROR: interactive stdin input not supported.');
      process.exit(1);
    } else if (e.code === 'EOF') {
      // Happens on Windows 7, but not OS X 10.8.3:
      // simply signals the end of *piped* stdin input.
      return '';
    }
    throw e; // unexpected exception
  }

  if (bytesRead === 0) {
    // No more stdin input available.
    // OS X 10.8.3: regardless of input method, this is how the end
    //   of input is signaled.
    // Windows 7: this is how the end of input is signaled for
    //   *interactive* stdin input.
    return '';
  }
  // Process the chunk read.

  var content = buf.function function toString() { [native code] }() { [native code] }(null, 0, bytesRead - 1);

  return content;
};
```

## Readline

`readline` 模組提供了一個用於從 Readble 的 stream (例如 process.stdin) 中一次讀取一行的介面. 當然你也可以用來讀取檔案或者 net, http 的 stream, 比如:

```javascript
const readline = require('readline');
const fs = require('fs');

const rl = readline.createInterface({
  input: fs.createReadStream('sample.txt')
});

rl.on('line', (line) => {
  console.log(`Line from file: ${line}`);
});
```

實現上, realine 在讀取 TTY 的資料時, 是通過 `input.on('keypress', onkeypress)` 時發現使用者按下了回車鍵來判斷是新的 line 的, 而讀取一般的 stream 時, 則是通過快取資料然後用正則 .test 來判斷是否為 new line 的.

PS: 打個廣告, 如果在編寫指令碼時, 不習慣這樣非同步獲取輸入, 想要同步獲取同步的使用者輸入可以看一看這個 Node.js 版本類 C語言使用的 [scanf](https://github.com/Lellansin/node-scanf/) 模組 (支援 ts).

## REPL

Read-Eval-Print-Loop (REPL)

整理中
