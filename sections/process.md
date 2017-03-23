# 程序

* [`[Doc]` Process (程序)](https://github.com/ElemeFE/node-interview/blob/master/sections/process.md#process)
* [`[Doc]` Child Processes (子程序)](https://github.com/ElemeFE/node-interview/blob/master/sections/process.md#child-process)
* [`[Doc]` Cluster (叢集)](https://github.com/ElemeFE/node-interview/blob/master/sections/process.md#cluster)
* [`[Basic]` 程序間通訊](https://github.com/ElemeFE/node-interview/blob/master/sections/process.md#程序間通訊)
* [`[Basic]` 守護程序](https://github.com/ElemeFE/node-interview/blob/master/sections/process.md#守護程序)

## 簡述

關於 Process, 我們需要討論的是兩個概念, ①作業系統的程序, ② Node.js 中的 Process 物件. 操作程序對於服務端而言, 好比 html 之於前端一樣基礎. 想做服務端程式設計是不可能繞過 Unix/Linux 的. 在 Linux/Unix/Mac 系統中執行 `ps -ef` 命令可以看到當前系統中執行的程序. 各個參數如下:

|列名稱|意義|
|-----|---|
|UID|執行該程序的使用者ID|
|PID|程序編號|
|PPID|該程序的父程序編號|
|C|該程序所在的CPU利用率|
|STIME|程序執行時間|
|TTY|程序相關的終端類型|
|TIME|程序所佔用的CPU時間|
|CMD|建立該程序的指令|

關於程序以及作業系統一些更深入的細節推薦閱讀 APUE, 即《Unix 高階程式設計》等書籍來了解.

## Process

這裡來討論 Node.js 中的 `process` 物件. 直接在程式碼中通過 `console.log(process)` 即可列印出來. 可以看到 process 物件暴露了非常多有用的屬性以及方法, 具體的細節見[官方文件](https://nodejs.org/dist/latest-v6.x/docs/api/process.html), 已經說的挺詳細了. 其中包括但不限於:

* 程序基礎資訊
* 程序 Usage
* 程序級事件
* 依賴模組/版本資訊
* OS 基礎資訊
* 賬戶資訊
* 訊號收發
* 三個標準流

### process.nextTick

上一節已經提到過 `process.nextTick` 了, 這是一個你需要了解的, 重要的, 基礎方法.


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

`process.nextTick` 並不屬於 Event loop 中的某一個階段, 而是在 Event loop 的每一個階段結束後, 直接執行 `nextTickQueue` 中插入的 "Tick", 並且直到整個 Queue 處理完. 所以面試時又有可以問的問題了, 遞迴呼叫 process.nextTick 會怎麼樣? (doge

```javascript
function test() {
  process.nextTick(() => test());
}
```

這種情況與以下情況, 有什麼區別? 為什麼?

```javascript
function test() {
  setTimeout(() => test(), 0);
}
```

### 配置

配置是開發部署中一個很常見的問題. 普通的配置有兩種方式, 一是定義配置檔案, 二是使用環境變數.

![node-configuration](https://blog-assets.risingstack.com/2016/Sep/node-js-survey/node-js-survey-envvar-config-new.png)

你可以通過[設定環境變數](http://cn.bing.com/search?q=linux+%E8%AE%BE%E7%BD%AE%E7%8E%AF%E5%A2%83%E5%8F%98%E9%87%8F)來指定配置, 然後通過 `process.env` 來獲取配置項. 另外也可以通過讀取定義好的配置檔案來獲取, 在這方面有很多不錯的庫例如 `dotenv`, `node-config` 等, 而在使用這些庫來載入配置檔案的時候, 通常都會碰到一個當前工作目錄的問題.

> <a name="q-cwd"></a> 程序的當前工作目錄是什麼? 有什麼作用?

當前程序啟動的目錄, 通過 process.cwd() 獲取當前工作目錄 (current working directory), 通常是命令列啟動的時候所在的目錄 (也可以在啟動時指定), 檔案操作等使用相對路徑的時候會相對當前工作目錄來獲取檔案.

一些獲取配置的第三方模組就是通過你的當前目錄來找配置檔案的. 所以如果你錯誤的目錄啟動指令碼, 可能沒法得到正確的結果. 在程式中可以通過 `process.chdir()` 來改變當前的工作目錄.

### 標準流

在 process 物件上還暴露了 `process.stderr`, `process.stdout` 以及 `process.stdin` 三個標準流, 熟悉 C/C++/Java 的同學應該對此比較熟悉. 關於這幾個流, 常見的面試問題是問 **console.log 是同步還是非同步? 如何實現一個 console.log?**

如果簡歷中有出現 C/C++ 關鍵字, 一般都會問到如何實現一個同步的輸入 (類似實現C語言的 `scanf`, C++ 的 `cin`, Python 的 `raw_input` 等).

### 維護方面

熟悉與程序有關的基礎命令, 如 top, ps, pstree 等命令.

## Child Process

子程序 (Child Process) 是程序中一個重要的概念. 你可以通過 Node.js 的 `child_process` 模組來執行可執行檔案, 呼叫命令列命令, 比如其他語言的程式等. 也可以通過該模組來將 .js 程式碼以子程序的方式啟動. 比較有名的網易的分散式架構 [pomelo](https://github.com/NetEase/pomelo) 就是基於該模組 (而不是 `cluster`) 來實現多程序分散式架構的.

> <a name="q-fork"></a> child_process.fork 與 POSIX 的 fork 有什麼區別?

Node.js 的 `child_process.fork()` 不像 POSIX [fork(2)](http://man7.org/linux/man-pages/man2/fork.2.html) 系統呼叫, 不會拷貝當前父程序. 這裡對於其他語言轉過的同學可能比較誤導, 可以作為一個比較偏的面試題.

* spawn() 啟動一個子程序來執行命令
  * options.detached 父程序死後是否允許子程序存活
  * options.stdio 指定子程序的三個標準流
* spawnSync() 同步版的 spawn, 可指定超時, 返回的物件可獲得子程序的情況
* exec() 啟動一個子程序來執行命令, 帶回撥參數獲知子程序的情況, 可指定程序執行的超時時間
* execSync() 同步版的 exec(), 可指定超時, 返回子程序的輸出 (stdout)
* execFile() 啟動一個子程序來執行一個可執行檔案, 可指定程序執行的超時時間
* execFileSync() 同步版的 execFile(), 返回子程序的輸出, 如何超時或者 exit code 不為 0, 會直接 throw Error
* fork() 加強版的 spawn(), 返回值是 ChildProcess 物件可以與子程序互動

其中 exec/execSync 方法會直接呼叫 bash 來解釋命令, 所以如果有命令有外部參數, 則需要注意被注入的情況.

### child.kill 與 child.send

常見會問的面試題, 如 `child.kill` 與 `child.send` 的區別. 二者一個是基於訊號系統, 一個是基於 IPC.

> <a name="q-child"></a> 父程序或子程序的死亡是否會影響對方? 什麼是孤兒程序?

子程序死亡不會影響父程序, 不過子程序死亡時（執行緒組的最後一個執行緒，通常是“領頭”執行緒死亡時），會向它的父程序傳送死亡訊號. 反之父程序死亡, 一般情況下子程序也會隨之死亡, 但如果此時子程序處於可執行態、僵死狀態等等的話, 子程序將被`程序1`（init 程序）收養，從而成為孤兒程序. 另外, 子程序死亡的時候（處於“終止狀態”），父程序沒有及時呼叫 `wait()` 或 `waitpid()` 來返回死亡程序的相關資訊，此時子程序還有一個 `PCB` 殘留在程序表中，被稱作殭屍程序.

## Cluster

Cluster 是常見的 Node.js 利用多核的辦法. 它是基於 `child_process.fork()` 實現的, 所以 cluster 產生的程序之間是通過 IPC 來通訊的, 並且它也沒有拷貝父程序的空間, 而是通過加入 cluster.isMaster 這個標識, 來區分父程序以及子程序, 達到類似 POSIX 的 [fork](http://man7.org/linux/man-pages/man2/fork.2.html) 的效果.

```javascript
const cluster = require('cluster');            // | |
const http = require('http');                  // | |
const numCPUs = require('os').cpus().length;   // | |    都執行了
                                               // | |
if (cluster.isMaster) {                        // |-|-----------------
  // Fork workers.                             //   |
  for (var i = 0; i < numCPUs; i++) {          //   |
    cluster.fork();                            //   |
  }                                            //   | 僅父程序執行 (a.js)
  cluster.on('exit', (worker) => {             //   |
    console.log(`${worker.process.pid} died`); //   |
  });                                          //   |
} else {                                       // |-------------------
  // Workers can share any TCP connection      // |
  // In this case it is an HTTP server         // |
  http.createServer((req, res) => {            // |
    res.writeHead(200);                        // |   僅子程序執行 (b.js)
    res.end('hello world\n');                  // |
  }).listen(8000);                             // |
}                                              // |-------------------
                                               // | |
console.log('hello');                          // | |    都執行了
```

在上述程式碼中 numCPUs 雖然是全局變數但是, 在父程序中修改它, 子程序中並不會改變, 因為父程序與子程序是完全獨立的兩個空間. 他們所謂的共有僅僅只是都執行了, 並不是同一份.

你可以把父程序執行的部分當做 `a.js`, 子程序執行的部分當做 `b.js`, 你可以把他們想象成是先執行了 `node a.js` 然後 cluster.fork 了幾次, 就執行執行了幾次 `node b.js`. 而 cluster 模組則是二者之間的一個橋樑, 你可以通過 cluster 提供的方法, 讓其二者之間進行溝通交流.

### How It Works

worker 程序是由 child_process.fork() 方法建立的, 所以可以通過 IPC 在主程序和子程序之間相互傳遞伺服器控制代碼.

cluster 模組提供了兩種分發連線的方式.

第一種方式 (預設方式, 不適用於 windows), 通過時間片輪轉法（round-robin）分發連線. 主程序監聽埠, 接收到新連線之後, 通過時間片輪轉法來決定將接收到的客戶端的 socket 控制代碼傳遞給指定的 worker 處理. 至於每個連線由哪個 worker 來處理, 完全由內建的迴圈演算法決定.

第二種方式是由主程序建立 socket 監聽埠後, 將 socket 控制代碼直接分發給相應的 worker, 然後當連線進來時, 就直接由相應的 worker 來接收連線並處理.

使用第二種方式時, 多個 worker 之間會存在競爭關係, 產生一個老生常談的 "[驚群效應](https://www.google.com.hk/search?q=%E6%83%8A%E7%BE%A4%E6%95%88%E5%BA%94)" 從而導致效率變低的問題. 該問題常見於 Apache. 並且各自競爭的情況下無法控制一個新的連線由哪個程序來處理, 從而導致各 worker 程序之間的負載不均衡, 比如通常 70% 的連線僅被 8 個程序中的 2 個處理, 而其他程序比較清閒.

## 程序間通訊

IPC (Inter-process communication) 程序間通訊技術. 常見的程序間通訊技術列表如下:

類型|無連線|可靠|流控制|優先順序
---|-----|----|-----|-----
普通PIPE|N|Y|Y|N
命名PIPE|N|Y|Y|N
訊息佇列|N|Y|Y|N
訊號量|N|Y|Y|Y
共享儲存|N|Y|Y|Y
UNIX流SOCKET|N|Y|Y|N
UNIX資料包SOCKET|Y|Y|N|N

Node.js 中的 IPC 通訊是由 libuv 通過管道技術實現的,  在 windows 下由命名管道（named pipe）實現也就是上表中的最後第二個,  *nix 系統則採用 UDS (Unix Domain Socket) 實現.

普通的 socket 是為網路通訊設計的, 而網路本身是不可靠的, 而為 IPC 設計的 socket 則不然, 因為預設本地的網路環境是可靠的, 所以可以簡化大量不必要的 encode/decode 以及計算校驗等, 得到效率更高的 UDS 通訊.

如果瞭解 Node.js 的 IPC 的話, 可以問個比較有意思的問題

> <a name="q-ipc-fd"></a> 在 IPC 通道建立之前, 父程序與子程序是怎麼通訊的? 如果沒有通訊, 那 IPC 是怎麼建立的?

這個問題也挺簡單, 只是個思路的問題. 在通過 child_process 建立子程序的時候, 是可以指定子程序的 env (環境變數) 的. 所以 Node.js 在啟動子程序的時候, 主程序先建立 IPC 頻道, 然後將 IPC 頻道的 fd (檔案描述符) 通過環境變數 (`NODE_CHANNEL_FD`) 的方式傳遞給子程序, 然後子程序通過 fd 連上 IPC 與父程序建立連線.

最後於程序間通訊 (IPC) 的問題, 一般不會直接問 IPC 的實現, 而是會問什麼情況下需要 IPC, 以及使用 IPC 處理過什麼業務場景等.


## 守護程序

最後的守護程序, 是服務端方面一個很基礎的概念了. 很多人可能只知道通過 pm2 之類的工具可以將程序以守護程序的方式啟動, 卻不瞭解什麼是守護程序, 為什麼要用守護程序. 對於水平好的同學, 我們是希望能瞭解守護程序的實現的.

普通的程序, 在使用者退出終端之後就會直接關閉. 通過 `&` 啟動到後臺的程序, 之後會由於會話（session組）被回收而終止程序. 守護程序是不依賴終端（tty）的程序, 不會因為使用者退出終端而停止執行的程序.

```c
// 守護程序實現 (C語言版本)
void init_daemon()
{
    pid_t pid;
    int i = 0;

    if ((pid = fork()) == -1) {
        printf("Fork error !\n");
        exit(1);
    }

    if (pid != 0) {
        exit(0);        // 父程序退出
    }

    setsid();           // 子程序開啟新會話, 併成為會話首程序和組長程序
    if ((pid = fork()) == -1) {
        printf("Fork error !\n");
        exit(-1);
    }
    if (pid != 0) {
        exit(0);        // 結束第一子程序, 第二子程序不再是會話首程序
                        // 避免當前會話組重新與tty連線
    }
    chdir("/tmp");      // 改變工作目錄
    umask(0);           // 重設檔案掩碼
    for (; i < getdtablesize(); ++i) {
       close(i);        // 關閉開啟的檔案描述符
    }

    return;
}
```

[Node.js 編寫守護程序](https://cnodejs.org/topic/57adfadf476898b472247eac)
