# Network

* [`[Doc]` Net (網路)](#net)
* [`[Doc]` UDP/Datagram](#udp)
* [`[Doc]` HTTP](#http)
* [`[Doc]` DNS (域名伺服器)](#dns)
* [`[Doc]` ZLIB (壓縮)](#zlib)
* [`[Point]` RPC](#rpc)


## Net

目前互聯化的核心是建立在 TCP/IP 協議的基礎上的, 這些協議將資料分割成小的資料包進行傳輸, 並且解決傳輸過程中各種各樣複雜的問題. 關於協議的具體細節推薦閱讀 W.Richard Stevens 的[《TCP/IP 詳解 卷1：協議》](https://www.amazon.cn/TCP-IP%E8%AF%A6%E8%A7%A3%E5%8D%B71-%E5%8D%8F%E8%AE%AE-W-Richard-Stevens/dp/B00116OTVS/), 本文不做贅述, 只是列舉一些常見的知識點, 新人推薦看[《圖解TCP/IP》](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00DMS9990/), 抓包工具推薦看[《Wireshark網路分析就這麼簡單》](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00PB5QQ84/).

### 粘包

預設情況下, TCP 連線會啟用延遲傳送演算法 (Nagle 演算法), 在資料傳送之前快取他們. 如果短時間有多個資料傳送, 會緩衝到一起作一次傳送 (緩衝大小見 `socket.bufferSize`), 這樣可以減少 IO 消耗提高效能.

如果是傳輸檔案的話, 那麼根本不用處理粘包的問題, 來一個包拼一個包就好了. 但是如果是多條訊息, 或者是別的用途的資料那麼久需要處理粘包.

可以參見網上流傳比較廣的一個例子, 連續呼叫兩次 send 分別傳送兩段資料 data1 和 data2, 在接收端有以下幾種常見的情況:

* A. 先接收到 data1, 然後接收到 data2 .
* B. 先接收到 data1 的部分資料, 然後接收到 data1 餘下的部分以及 data2 的全部.
* C. 先接收到了 data1 的全部資料和 data2 的部分資料, 然後接收到了 data2 的餘下的資料.
* D. 一次性接收到了 data1 和 data2 的全部資料.

其中的 BCD 就是我們常見的粘包的情況. 而對於處理粘包的問題, 常見的解決方案有:

* 1. 多次傳送之前間隔一個等待時間
* 2. 關閉 Nagle 演算法
* 3. 進行封包/拆包

***方案1***

只需要等上一段時間再進行下一次 send 就好, 適用於互動頻率特別低的場景. 缺點也很明顯, 對於比較頻繁的場景而言傳輸效率實在太低. 不過幾乎用做什麼處理.

***方案2***

關閉 Nagle 演算法, 在 Node.js 中你可以通過 [`socket.setNoDelay()`](https://nodejs.org/dist/latest-v6.x/docs/api/net.html#net_socket_setnodelay_nodelay) 方法來關閉 Nagle 演算法, 讓每一次 send 都不緩衝直接傳送.

該方法比較適用於每次傳送的資料都比較大 (但不是檔案那麼大), 並且頻率不是特別高的場景. 如果是每次傳送的資料量比較小, 並且頻率特別高的, 關閉 Nagle 純屬自廢武功.

另外, 該方法不適用於網路較差的情況, 因為 Nagle 演算法是在服務端進行的包合併情況, 但是如果短時間內客戶端的網路情況不好, 或者應用層由於某些原因不能及時將 TCP 的資料 recv, 就會造成多個包在客戶端緩衝從而粘包的情況. (如果是在穩定的機房內部通訊那麼這個概率是比較小可以選擇忽略的)

***方案3***

封包/拆包是目前業內常見的解決方案了. 即給每個資料包在傳送之前, 於其前/後放一些有特徵的資料, 然後收到資料的時候根據特徵資料分割出來各個資料包.

### 可靠傳輸

為每一個傳送的資料包分配一個序列號(SYN, Synchronise packet), 每一個包在對方收到後要返回一個對應的應答資料包(ACK, Acknowledgedgement),. 傳送方如果發現某個包沒有被對方 ACK, 則會選擇重發. 接收方通過 SYN 序號來保證資料的不會亂序(reordering), 傳送方通過 ACK 來保證資料不缺漏, 以此參考決定是否重傳. 關於具體的序號計算, 丟包時的重傳機制等可以參見閱讀陳皓的 [《TCP的那些事兒（上）》](http://coolshell.cn/articles/11564.html) 此處不做贅述.

### window

TCP 頭裡有一個 Window 欄位, 是接收端告訴傳送端自己還有多少緩衝區可以接收資料的. 傳送端就可以根據接收端的處理能力來傳送資料, 從而避免接收端處理不過來. 詳細參見陳皓的 [《TCP的那些事兒（下）》](http://coolshell.cn/articles/11609.html)

> window 是否設定的越大越好?

類似木桶理論, 一個木桶能裝多少水, 是由最短的那塊木板決定的. 一個 TCP 連線的 window 是由該連線中間一連串裝置中 window 最小的那一個裝置決定的.

### backlog

![圖片出處 http://www.cnxct.com/something-about-phpfpm-s-backlog/](/assets/socket-backlog.png)

關於該 backlog 的定義參見 [man](https://linux.die.net/man/2/listen) 手冊:

> The behavior of the backlog argument on TCP sockets changed with Linux 2.2. Now it specifies the queue length for completely established sockets waiting to be accepted, instead of the number of incomplete connection requests.

backlog 用於設定客戶端與服務端 `ESTABLISHED` 之後等待 accept 的佇列長圖 (如上圖中的 accept queue). 如果 backlog 過小, 在併發連線大的情況下容易導致 accept queue 裝滿之後斷開連線. 但是如果將這個佇列設定的特別大, 那麼假定連線數併發量是 65525, 以 php-fpm 的 qps 5000 為例, 處理完約耗時 13s, 而這段時間中連線可能早已被 nginx 或者客戶端斷開, 那麼我們去 accept 這個 socket 時只會拿到一個 broken pipe (該例子出處見 [PHP 源碼 Set FPM_BACKLOG_DEFAULT to 511](https://github.com/php/php-src/commit/ebf4ffc9354f316f19c839a114b26a564033708a)). 經過<del>我也不懂的</del>計算 backlog 的長度預設是 511.

另外提一句, 這個 backlog 是通過系統指定時是通過 `somaxconn` 參數來指定 accept queue 的. 而 `tcp_max_syn_backlog` 參數指定的是 SYN queue 的長度.

### 狀態機

![tcpfsm.png](/assets/tcpfsm.png)

關於網路連線的建立以及斷開, 存在著一個複雜的狀態轉換機制, 完整的狀態表參見 [《The TCP/IP Guide》](http://www.tcpipguide.com/free/t_TCPOperationalOverviewandtheTCPFiniteStateMachineF-2.htm)

state|簡述
-----|---
CLOSED|連線關閉, 所有連線的初始狀態
LISTEN|監聽狀態, 等待客戶端傳送 SYN
SYN-SENT|客戶端傳送了 SYN, 等待服務端回覆
SYN-RECEIVED|雙方都收到了 SYN, 等待 ACK
ESTABLISHED| SYN-RECEIVED 收到 ACK 之後, 狀態切換為連線已建立.
CLOSE-WAIT|被動方收到了關閉請求(FIN)後, 傳送 ACK, 如果有資料要傳送, 則傳送資料, 無資料傳送則回覆 FIN. 狀態切換到 LAST-ACK
LAST-ACK|等待對方 ACK 當前裝置的 CLOSE-WAIT 時傳送的 FIN, 等到則切換 CLOSED
FIN-WAIT-1|主動方傳送 FIN, 等待 ACK
FIN-WAIT-2|主動方收到被動方的 ACK, 等待 FIN
CLOSING|主動方收到了FIN, 卻沒收到 FIN-WAIT-1 時發的 ACK, 此時等待那個 ACK
TIME-WAIT|主動方收到 FIN, 返回收到對方 FIN 的 ACK, 等待對方是否真的收到了 ACK, 如果過一會又來一個 FIN, 表示對方沒收到, 這時要再 ACK 一次

> <a name="q-time-wait"></a> `TIME_WAIT` 是什麼情況? 出現過多的 `TIME_WAIT` 可能是什麼原因?

`TIME_WAIT` 是連線的某一方 (可能是服務端也可能是客戶端) 主動斷開連線時, 四次揮手等待被斷開的一方是否收到最後一次揮手 (ACK) 的狀態. 如果在等待時間中, 再次收到第三次揮手 (FIN) 表示對方沒收到最後一次揮手, 這時要再 ACK 一次. 這個等待的作用是避免出現連線混用的情況 (`prevent potential overlap with new connections` see [TCP Connection Termination](http://www.tcpipguide.com/free/t_TCPConnectionTermination.htm) for more).

出現大量的 `TIME_WAIT` 比較常見的情況是, 併發量大, 伺服器在短時間斷開了大量連線. 對應 HTTP server 的情況可能是沒開啟 `keepAlive`. 如果有開 `keepAlive`, 一般是等待客戶端自己主動斷開, 那麼`TIME_WAIT` 就只存在客戶端, 而服務端則是 `CLOSE_WAIT` 的狀態, 如果服務端出現大量 `CLOSE_WAIT`, 意味著當前服務端建立的連線大面積的被斷開, 可能是目標服務叢集重啟之類.


## UDP

> <a name="q-tcp-udp"></a> TCP/UDP 的區別? UDP 有粘包嗎?

協議|連線性|雙工性|可靠性|有序性|有界性|擁塞控制|傳輸速度|量級|頭部大小
---|---|---|---|---|---|---|---|---|---
TCP|面向連線<br>(Connection oriented)|全雙工(1:1)|可靠<br>(重傳機制)|有序<br>(通過SYN排序)|無, 有[粘包情況](#粘包)|有|慢|低|20~60位元組
UDP|無連線<br>(Connection less)|n:m|不可靠<br>(丟包後資料丟失)|無序|有訊息邊界, **無粘包**|無|快|高|8位元組

UDP socket 支援 n 對 m 的連線狀態, 在[官方文件](https://nodejs.org/dist/latest-v6.x/docs/api/dgram.html)中有寫到在 `dgram.createSocket(options[, callback])` 中的 option 可以指定 `reuseAddr` 即 `SO_REUSEADDR`標誌. 通過 `SO_REUSEADDR` 可以簡單的實現 n 對 m 的多播特性 (不過僅在支援多播的系統上才有).


### 常見的應用場景

<table>
  <tr><th>傳輸層協議</th><th>應用</th><th>應用層協議</th></tr>
  <tr><td rowspan="5">TCP</td><td>電子郵件</td><td>SMTP</td></tr>
  <tr><td>終端連線</td><td>TELNET</td></tr>
  <tr><td>終端連線</td><td>SSH</td></tr>
  <tr><td>全球資訊網</td><td>HTTP</td></tr>
  <tr><td>檔案傳輸</td><td>FTP</td></tr>
  <tr><td rowspan="8">UDP</td><td>域名解析</td><td>DNS</td></tr>
  <tr><td>簡單檔案傳輸</td><td>TFTP</td></tr>
  <tr><td>網路時間校對</td><td>NTP</td></tr>
  <tr><td>網路檔案系統</td><td>NFS</td></tr>
  <tr><td>路由選擇</td><td>RIP</td></tr>
  <tr><td>IP電話</td><td>-</td></tr>
  <tr><td>流式多媒體通訊</td><td>-</td></tr>
</table>

簡單的說, UDP 速度快, 開銷低, 不用封包/拆包允許丟一部分資料, 監控統計/日誌資料上報/流媒體通訊等場景都可以用 UDP. 目前 Node.js 的項目中使用 UDP 比較流行的是 [StatsD](https://github.com/etsy/statsd) 監控服務.


## HTTP

目前世界上執行最良好的分散式叢集, 莫過於當前的全球資訊網了 (http servers) 了. 目前前端工程師也都是靠 HTTP 協議吃飯的, 所以 2-3 年的前端同學都應該對 HTTP 有比較深的理解了, 所以這裡不做太多的贅述. 推薦書籍[《圖解HTTP》](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B00JTQK1L4/), 部落格[HTTP 協議入門](http://www.ruanyifeng.com/blog/2016/08/http.html).

另外最近幾年開始大家對 HTTP 的面試的考察也漸漸偏向[理解 RESTful 架構](http://www.ruanyifeng.com/blog/2011/09/restful.html). 簡單的說, RESTful 是把每個 URI 當做資源 (Resources), 通過 method 作為動詞來對資源做不同的動作, 然後伺服器返回 status 來得知資源狀態的變化 (State Transfer);

### method/status

因為 HTTP 的方法 (method) 與狀態碼 (status) 講解太常見, 你可以使用如下程式碼列印出來自己看 Node.js 官方定義的, 完整的就不列舉了.

```javascript
const http = require('http');

console.log(http.METHODS);
console.log(http.STATUS_CODES);
```

一個常見的 method 列表, 關於這些 method 在 RESTful 中的一些應用的詳細可以參見[Using HTTP Methods for RESTful Services](http://www.restapitutorial.com/lessons/httpmethods.html)

methods|CRUD|冪等|快取
---|---|---|---
GET|Read|✓|✓
POST|Create||
PUT|Update/Replace|✓
PATCH|Update/Modify|✓
DELETE|Delete|✓

> GET 和 POST 有什麼區別?

網上有很多講這個的, 比如從書籤, url 等前端的角度去看他們的區別這裡不贅述. 而從後端的角度看, 前兩年出來一個 《GET 和 POST 沒有區別》(出處不好考究, 就沒貼了) 的文章比較有名, 早在我剛學 PHP 的時候也有過這種疑惑, 剛學 Node 的時候發現不能像 PHP 那樣同時處理 GET 和 POST 的時候還很不適應. 後來接觸 RESTful 才意識到, 這兩個東西最根本的差別是語義, 引申了看, 協議 (protocol) 這種東西就是人與人之間協商的約定, 什麼行為是什麼作用都是"約定"好的, 而不是強制使用的, 非要把 GET 當 POST 這樣不遵守約定的做法我們也愛莫能助.

跑題了, 簡而言之, 討論這二者的區別最好從 RESTful 提倡的語義角度來講<del>比較符合當代程式設計師的逼格</del>比較合理.

> <a name="q-post-put"></a> POST 和 PUT 有什麼區別?

POST 是新建 (create) 資源, 非冪等, 同一個請求如果重複 POST 會新建多個資源. PUT 是 Update/Replace, 冪等, 同一個 PUT 請求重複操作會得到同樣的結果.


### headers

HTTP headers 是在進行 HTTP 請求的互動過程中互相支會對方一些資訊的主要欄位. 比如請求 (Request) 的時候告訴服務端自己能接受的各項參數, 以及之前就存在本地的一些資料等. 詳細各位可以參見 wikipedia:

* [Request fields](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Request_fields)
* [Response fields](https://en.wikipedia.org/wiki/List_of_HTTP_header_fields#Response_fields)

> <a name="q-cookie-session"></a> cookie 與 session 的區別? 服務端如何清除 cookie?

主要區別在於, session 存在服務端, cookie 存在客戶端. session 比 cookie 更安全. 而且 cookie 不一定一直能用 (可能被瀏覽器關掉). 服務端可以通過設定 cookie 的值為空並設定一個及時的 expires 來清除存在客戶端上的 cookie.

> <a name="q-cors"></a> 什麼是跨域請求? 如何允許跨域?

出於安全考慮, 預設情況下使用 XMLHttpRequest 和 Fetch 發起 HTTP 請求必須遵守同源策略, 即只能向相同域名請求. 向不同域名的請求被稱作跨域請求 (cross-origin HTTP request). 可以通過設定 [CORS headers](https://developer.mozilla.org/en-US/docs/Glossary/CORS) 即 `Access-Control-Allow-` 系列來允許跨域. 例如:

```
location ~* ^/(?:v1|_) {
  if ($request_method = OPTIONS) { return 200 ''; }
  header_filter_by_lua '
    ngx.header["Access-Control-Allow-Origin"] = ngx.var.http_origin; # 這樣相當於允許所有來源了
    ngx.header["Access-Control-Allow-Methods"] = "GET, POST, PUT, DELETE, PATCH, OPTIONS";
    ngx.header["Access-Control-Allow-Credentials"] = "true";
    ngx.header["Access-Control-Allow-Headers"] = "Content-Type";
  ';
  proxy_pass http://localhost:3001;
}
```

> `Script error.` 是什麼錯誤? 如何拿到更詳細的資訊?

接上題, 由於同源性策略 (CORS), 如果你引用的 js 指令碼所在的域與當前域不同, 那麼瀏覽器會把 onError 中的 msg 替換為 `Script error.` 要拿到詳細錯誤的方法, 處理配好 `Access-Control-Allow-Origin` 還有在引用指令碼的時候指定 `crossorigin` 例如:

```html
<script src="http://another-domain.com/app.js" crossorigin="anonymous"></script>
```

詳見 [Javascript Script Error.](https://sentry.io/answers/javascript-script-error/)


### Agent

Node.js 中的 `http.Agent` 用於池化 HTTP 客戶端請求的 socket (pooling sockets used in HTTP client requests). 也就是複用 HTTP 請求時候的 socket. 如果你沒有指定 Agent 的話, 預設用的是 `http.globalAgent`.

另外最近發現一個 Agent 坑爹的地方, 當 `keepAlive` 為 true 是, 由於 socket 複用, 之前的事件監聽如果忘了清除很容易導致重複監聽, 並且舊的監聽中的引用不會釋放從導致記憶體洩漏, 參見這個 [issue](https://github.com/nodejs/node/issues/9268). (本組的同學有在整理這方面的文章, 請期待)

### socket hang up

hang up 有結束通話的意思, socket hang up 也可以理解為 socket 被結束通話. 在 Node.js 中當你要 response 一個請求的時候, 發現該這個 socket 已經被 "結束通話", 就會就會報 socket hang up 錯誤.

[Node.js 中源碼的情況:](https://github.com/nodejs/node/blob/v6.x/lib/_http_client.js#L286)

```javascript
function socketCloseListener() {
  var socket = this;
  var req = socket._httpMessage;

  // Pull through final chunk, if anything is buffered.
  // the ondata function will handle it properly, and this
  // is a no-op if no final chunk remains.
  socket.read();

  // NOTE: It's important to get parser here, because it could be freed by
  // the `socketOnData`.
  var parser = socket.parser;
  req.emit('close');
  if (req.res && req.res.readable) {
    // Socket closed before we emitted 'end' below.
    req.res.emit('aborted');
    var res = req.res;
    res.on('end', function() {
      res.emit('close');
    });
    res.push(null);
  } else if (!req.res && !req.socket._hadError) {
    // This socket error fired before we started to
    // receive a response. The error needs to
    // fire on the request.
    req.emit('error', createHangUpError());  // <------------------- socket hang up
    req.socket._hadError = true;
  }

  // Too bad.  That output wasn't getting written.
  // This is pretty terrible that it doesn't raise an error.
  // Fixed better in v0.10
  if (req.output)
    req.output.length = 0;
  if (req.outputEncodings)
    req.outputEncodings.length = 0;

  if (parser) {
    parser.finish();
    freeParser(parser, req, socket);
  }
}
```

典型的情況是使用者使用瀏覽器, 請求的時間有點長, 然後使用者簡單的按了一下 F5 重新整理頁面. 這個操作會讓瀏覽器取消之前的請求, 然後導致服務端 throw 了一個 socket hang up.

詳見萬能的 stackoverflow: [NodeJS - What does “socket hang up” actually mean?](http://stackoverflow.com/questions/16995184/nodejs-what-does-socket-hang-up-actually-mean)


## DNS

早期可以用 TCP/IP 通訊之後, 有一個比較蛋疼的問題, 就是 ip 都是一串比較長的數字, 比較難記, 於是大家想了個辦法, 給每個 ip 取個好記一點的名字比如 `Alan -> 192.168.0.11` 這樣只需要記住好記的名字即可, 隨著這個名字的規範化最終變成了今天的域名 (Domain name), 而幫助別人記錄這個名字的服務就叫域名解析服務 (Domain Name Service).

DNS 服務主要基於 UDP, 這裡簡單介紹 Node.js 實現的介面中的兩個方法:

方法|功能|同步|網路請求|速度
---|---|---|---|---
.lookup(hostname[, options], cb)|通過系統自帶的 DNS 快取 (如 `/etc/hosts`)|同步|無|快
.resolve(hostname[, rrtype], cb)|通過系統配置的 DNS 伺服器指定的記錄 (rrtype指定)|非同步|有|慢

> DNS 模組中 .lookup 與 .resolve 的區別?

當你要解析一個域名的 ip 時, 通過 .lookup 查詢直接呼叫 `getaddrinfo` 來拿取地址, 速度很快, 但是如果本地的 hosts 檔案被修改了, .lookup 就會拿 hosts 檔案中的地方, 而 .resolve 依舊是外部正常的地址.

由於 .lookup 是同步的, 所以如果由於什麼不可控的原因導致 `getaddrinfo` 緩慢或者阻塞是會影響整個 Node 程序的, 參見[文件](https://nodejs.org/dist/latest-v6.x/docs/api/dns.html#dns_dns_lookup).

> hosts 檔案是什麼? 什麼叫 DNS 本地解析?

hosts 檔案是個沒有副檔名的系統檔案，其作用就是將網址域名與其對應的 IP 地址建立一個關聯“資料庫”，當使用者在瀏覽器中輸入一個需要登入的網址時，系統會首先自動從 hosts 檔案中尋找對應的IP地址。

當我們訪問一個域名時，實際上需要的是訪問對應的 IP 地址。這時候，獲取 IP 地址的方式，先是讀取瀏覽器快取，如果未命中 => 接著讀取本地 hosts 檔案，如果還是未命中 => 則向 DNS 伺服器傳送請求獲取。在向 DNS 伺服器獲取 IP 地址之前的行為，叫做 DNS 本地解析。

## ZLIB

在網路傳輸過程中, 如果網速穩定的情況下, 對資料進行壓縮, 壓縮比率越大, 那麼傳輸的效率就越高等同於速度越快了. zlib 模組提供了 Gzip/Gunzip, Deflate/Inflate 和 DeflateRaw/InflateRaw 等壓縮方法的類, 這些類接收相同的參數, 都屬於可讀寫的 Stream 例項.

TODO

## RPC

RPC (Remote Procedure Call Protocol) 基於 TCP/IP 來實現呼叫遠端伺服器的方法, 與 http 同屬應用層. 常用於構建叢集, 以及微服務 (推薦一本[《Node.js 微服務》](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B01MXY8ARP)<del>雖然我還沒看完</del>)

常見的 RPC 方式:

* [Thrift](http://thrift.apache.org/)
* HTTP
* MQ

### Thrift

> **Thrift**是一種[介面描述語言](https://zh.wikipedia.org/wiki/%E6%8E%A5%E5%8F%A3%E6%8F%8F%E8%BF%B0%E8%AF%AD%E8%A8%80 "介面描述語言")和二進位制通訊協議，它被用來定義和建立跨語言的服務。它被當作一個[遠端過程呼叫](https://zh.wikipedia.org/wiki/%E8%BF%9C%E7%A8%8B%E8%BF%87%E7%A8%8B%E8%B0%83%E7%94%A8 "遠端過程呼叫")（RPC）框架來使用，是由[Facebook](https://zh.wikipedia.org/wiki/Facebook "Facebook")為“大規模跨語言服務開發”而開發的。它通過一個程式碼生成引擎聯合了一個軟體棧，來建立不同程度的、無縫的[跨平臺](https://zh.wikipedia.org/wiki/%E8%B7%A8%E5%B9%B3%E5%8F%B0 "跨平臺")高效服務，可以使用[C#](https://zh.wikipedia.org/wiki/C%E2%99%AF "C♯")、[C++](https://zh.wikipedia.org/wiki/C%2B%2B "C++")（基於[POSIX](https://zh.wikipedia.org/wiki/POSIX "POSIX")相容系統）、Cappuccino、[Cocoa](https://zh.wikipedia.org/wiki/Cocoa "Cocoa")、[Delphi](https://zh.wikipedia.org/wiki/Delphi "Delphi")、[Erlang](https://zh.wikipedia.org/wiki/Erlang "Erlang")、[Go](https://zh.wikipedia.org/wiki/Go "Go")、[Haskell](https://zh.wikipedia.org/wiki/Haskell "Haskell")、[Java](https://zh.wikipedia.org/wiki/Java "Java")、[Node.js](https://zh.wikipedia.org/wiki/Node.js "Node.js")、[OCaml](https://zh.wikipedia.org/wiki/OCaml "OCaml")、[Perl](https://zh.wikipedia.org/wiki/Perl "Perl")、[PHP](https://zh.wikipedia.org/wiki/PHP "PHP")、[Python](https://zh.wikipedia.org/wiki/Python "Python")、[Ruby](https://zh.wikipedia.org/wiki/Ruby "Ruby")和[Smalltalk](https://zh.wikipedia.org/wiki/Smalltalk "Smalltalk")。雖然它以前是由Facebook開發的，但它現在是[Apache軟體基金會](https://zh.wikipedia.org/wiki/Apache%E8%BD%AF%E4%BB%B6%E5%9F%BA%E9%87%91%E4%BC%9A "Apache軟體基金會")的[開源](https://zh.wikipedia.org/wiki/%E5%BC%80%E6%BA%90 "開源")項目了。該實現被描述在2007年4月的一篇由Facebook發表的技術論文中，該論文現由Apache掌管。

### HTTP

使用 HTTP 協議來進行 RPC 呼叫也是很常見的, 相比 TCP 連線, 通過通過 HTTP 的方式效能會差一些, 但是在使用以及偵錯上會簡單一些. 近期比較有名的框架參見 [gRPC](http://www.grpc.io/):

> gRPC is an open source remote procedure call (RPC) system initially developed at Google. It uses HTTP/2 for transport, Protocol Buffers as the interface description language, and provides features such as authentication, bidirectional streaming and flow control, blocking or nonblocking bindings, and cancellation and timeouts. It generates cross-platform client and server bindings for many languages.

### MQ

使用訊息佇列 (Message Queue) 來進行 RPC 呼叫 (RPC over mq) 在業內有不少例子, 比較適合業務解耦/廣播/限流等場景.

TODO
