# OS

* `[Doc]` TTY
* `[Doc]` OS (作業系統)
* `[Doc]` 命令列參數
* `[Basic]` 負載
* `[Point]` CheckList
* `[Basic]` 指標

## TTY

"tty" 原意是指 "teletype" 即打字機, "pty" 則是 "pseudo-teletype" 即偽打字機. 在 Unix 中, `/dev/tty*` 是指任何表現的像打字機的裝置, 例如終端 (terminal).

你可以通過 `w` 命令檢視當前登入的使用者情況, 你會發現每登入了一個視窗就會有一個新的 tty.

```shell
$ w
 11:49:43 up 482 days, 19:38,  3 users,  load average: 0.03, 0.08, 0.07
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
dev      pts/0    10.0.128.252     10:44    1:01m  0.09s  0.07s -bash
dev      pts/2    10.0.128.252     11:08    2:07   0.17s  0.14s top
root     pts/3    10.0.240.2       11:43    7.00s  0.04s  0.00s w
```

使用 ps 命令檢視程序資訊中也有 tty 的資訊:

```shell
$ ps -x
  PID TTY      STAT   TIME COMMAND
 5530 ?        S      0:00 sshd: dev@pts/3
 5531 pts/3    Ss+    0:00 -bash
11296 ?        S      0:00 sshd: dev@pts/4
11297 pts/4    Ss     0:00 -bash
13318 pts/4    R+     0:00 ps -x
23733 ?        Ssl    2:53 PM2 v1.1.2: God Daemon
```

其中為 `?` 的是沒有依賴 TTY 的程序, 即[守護程序](https://github.com/ElemeFE/node-interview/blob/master/sections/process.md#%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B).

在 Node.js 中你可以通過 stdio 的 isTTY 來判斷當前程序是否處於 TTY (如終端) 的環境.

```shell
$ node -p -e "Boolean(process.stdout.isTTY)"
true
$ node -p -e "Boolean(process.stdout.isTTY)" | cat
false
```

## OS

通過 OS 模組可以獲取到當前系統一些基礎資訊的輔助函數.

|屬性|描述|
|---|---|
|os.EOL|根據當前系統, 返回當前系統的 `End Of Line`|
|os.arch()|返回當前系統的 CPU 架構, 如 `'x86'` 和 `'x64'`|
|os.constants|返回系統常量|
|os.cpus()|返回 CPU 每個核的資訊|
|os.endianness()|返回 CPU 位元組序, 如果是大端位元組序返回 `BE`, 小端位元組序則 `LE`|
|os.freemem()|返回系統空閒記憶體的大小, 單位是位元組|
|os.homedir()|返回當前使用者的根目錄|
|os.hostname()|返回當前系統的主機名|
|os.loadavg()|返回負載資訊|
|os.networkInterfaces()|返回網卡資訊 (類似 `ifconfig`)|
|os.platform()|返回編譯時指定的平臺資訊, 如 `win32`, `linux`, 同 `process.platform()`|
|os.release()|返回作業系統的分發版本號|
|os.tmpdir()|返回系統預設的臨時資料夾|
|os.totalmem()|返回總記憶體大小(同記憶體條大小)|
|os.type()|根據 `[uname](https://en.wikipedia.org/wiki/Uname#Examples)` 返回系統的名稱|
|os.uptime()|返回系統的執行時間，單位是秒|
|os.userInfo([options])|返回當前使用者資訊|

> 不同作業系統的換行符 (EOL) 有什麼區別?

end of line (EOL) 同 newline, line ending, 以及 line break.

通常由 line feed (LF, `\n`) 和 carriage return (CR, `\r`) 組成. 常見的情況:

|符號|系統|
|---|---|
|LF|在 Unix 或 Unix 相容系統 (GNU/Linux, AIX, Xenix, Mac OS X, ...)、BeOS、Amiga、RISC OS|
|CR+LF|MS-DOS、微軟視窗作業系統 (Microsoft Windows)、大部分非 Unix 的系統|
|CR|Apple II 家族, Mac OS 至版本9|

如果不瞭解 EOL 跨系統的相容情況, 那麼在處理檔案的行分割/行統計等情況時可能會被坑.

### OS 常量

* 訊號常量 (Signal Constants), 如 `SIGHUP`, `SIGKILL` 等.
* POSIX 錯誤常量 (POSIX Error Constants), 如 `EACCES`, `EADDRINUSE` 等.
* Windows 錯誤常量 (Windows Specific Error Constants), 如 `WSAEACCES`, `WSAEBADF` 等.
* libuv 常量 (libuv Constants), 僅 `UV_UDP_REUSEADDR`.

## 命令列參數

命令列參數 (Command Line Options), 即對 CLI 使用上的一些文件. 關於 CLI 主要有 4 種使用方式:

* node [options] [v8 options] [script.js | -e "script"] [arguments]
* node debug [script.js | -e "script" | <host>:<port>] …
* node --v8-options
* 無參數直接啟動 REPL 環境

### Options

|參數|簡介|
|---|---|
|-v, --version|檢視當前 node 版本|
|-h, --help|檢視幫助文件|
|-e, --eval "script"|將參數字符串當做程式碼執行
|-p, --print "script"|列印 `-e` 的返回值
|-c, --check|檢查語法並不執行
|-i, --interactive|即使 stdin 不是終端也開啟 REPL 模式
|-r, --require module|在啟動前預先 `require` 指定模組
|--no-deprecation|關閉廢棄模組警告
|--trace-deprecation|列印廢棄模組的堆棧跟蹤資訊
|--throw-deprecation|執行廢棄模組時拋出錯誤
|--no-warnings|無視報警（包括廢棄警告）
|--trace-warnings|列印警告的 stack （包括廢棄模組）
|--trace-sync-io|只要檢測到非同步 I/O 出於 Event loop 的開頭就列印 stack trace
|--zero-fill-buffers|自動初始化(zero-fill) **Buffer** 和 **SlowBuffer**
|--preserve-symlinks|在解析和快取模組時指示模組載入程式儲存符號連結
|--track-heap-objects|為堆快照跟蹤堆物件的分配情況
|--prof-process|使用 v8 選項 `--prof` 生成 Profilling 報告
|--v8-options|顯示 v8 命令列選項
|--tls-cipher-list=list|指明替代的預設 TLS 加密器列表
|--enable-fips|在啟動時開啟 FIPS-compliant crypto
|--force-fips|在啟動時強制實施 FIPS-compliant
|--openssl-config=file|啟動時載入 OpenSSL 配置檔案
|--icu-data-dir=file|指定ICU資料載入路徑

### 環境變數

|環境變數|簡介|
|----|----|
|`NODE_DEBUG=module[,…]`|指定要列印偵錯資訊的核心模組列表
|`NODE_PATH=path[:…]`|指定搜尋目錄模組路徑的字首列表
|`NODE_DISABLE_COLORS=1`|關閉 REPL 的顏色顯示
|`NODE_ICU_DATA=file`|ICU (Intl object) 資料路徑
|`NODE_REPL_HISTORY=file`|持久化儲存REPL歷史檔案的路徑
|`NODE_TTY_UNSAFE_ASYNC=1`|設定為1時, 將同步操作 stdio (如 console.log 變成同步)
|`NODE_EXTRA_CA_CERTS=file`|指定 CA (如 VeriSign) 的額外證書路徑

## 負載

負載是衡量伺服器執行狀態的一個重要概念. 通過負載情況, 我們可以知道伺服器目前狀態是清閒, 良好, 繁忙還是即將 crash.

通常我們要檢視的負載是 CPU 負載, 詳細一點的情況你可以通過閱讀這篇部落格: [Understanding Linux CPU Load](http://blog.scoutapp.com/articles/2009/07/31/understanding-load-averages) 來了解.

命令列上可以通過 `uptime`, `top` 命令, Node.js 中可以通過 `os.loadavg()` 來獲取當前系統的負載情況:

```
load average: 0.09, 0.05, 0.01
```

其中分別是最近 1 分鐘, 5 分鐘, 15 分鐘內系統 CPU 的平均負載. 當 CPU 的一個核工作飽和的時候負載為 1, 有幾核 CPU 那麼飽和負載就是幾.

在 Node.js 中單個程序的 CPU 負載檢視可以使用 [pidusage](https://github.com/soyuka/pidusage) 模組.

除了 CPU 負載, 對於服務端 (偏維護) 還需要了解網路負載, 磁碟負載等.

## CheckList

> 有一個醉漢半夜在路燈下徘徊，路過的人奇怪地問他：“你在路燈下找什麼？”醉漢回答：“我在找我的KEY”,路人更奇怪了：“找鑰匙為什麼在路燈下?”，醉漢說：“因為這裡最亮！”。

很多服務端的同學在說到檢查伺服器狀態時只知道使用 `top` 命令, 其實情況就和上面的笑話一樣, 因為對於他們而言 `top` 是最亮的那盞路燈.

對於服務端程式設計師而言, 完整的伺服器 checklist 首推 [《效能之巔》](https://www.amazon.cn/%E5%9B%BE%E4%B9%A6/dp/B0140I5WPK) 第二章中講述的 [USE 方法](http://www.brendangregg.com/USEmethod/use-linux.html).

The USE Method provides a strategy for performing a complete check of system health, identifying common bottlenecks and errors. For each system resource, metrics for utilization, saturation and errors are identified and checked. Any issues discovered are then investigated using further strategies.

This is an example USE-based metric list for Linux operating systems (eg, Ubuntu, CentOS, Fedora). This is primarily intended for system administrators of the physical systems, who are using command line tools. Some of these metrics can be found in remote monitoring tools.

### Physical Resources

<table border="1" cellpadding="2" width="100%">
<tbody><tr><th>component</th><th>type</th><th>metric</th></tr>

<tr><td>CPU</td><td>utilization</td><td>system-wide: <tt>vmstat 1</tt>, "us" + "sy" + "st"; <tt>sar -u</tt>, sum fields except "%idle" and "%iowait"; <tt>dstat -c</tt>, sum fields except "idl" and "wai"; per-cpu: <tt>mpstat -P ALL 1</tt>, sum fields except "%idle" and "%iowait"; <tt>sar -P ALL</tt>, same as <tt>mpstat</tt>; per-process: <tt>top</tt>, "%CPU"; <tt>htop</tt>, "CPU%"; <tt>ps -o pcpu</tt>; <tt>pidstat 1</tt>, "%CPU"; per-kernel-thread: <tt>top</tt>/<tt>htop</tt> ("K" to toggle), where VIRT == 0 (heuristic). [1]</td></tr>
<tr><td>CPU</td><td>saturation</td><td>system-wide: <tt>vmstat 1</tt>, "r" &gt; CPU count [2]; <tt>sar -q</tt>, "runq-sz" &gt; CPU count; <tt>dstat -p</tt>, "run" &gt; CPU count; per-process: /proc/PID/schedstat 2nd field (sched_info.run_delay); <tt>perf sched latency</tt> (shows "Average" and "Maximum" delay per-schedule); dynamic tracing, eg, SystemTap schedtimes.stp "queued(us)" [3]</td></tr>
<tr><td>CPU</td><td>errors</td><td><tt>perf</tt> (LPE) if processor specific error events (CPC) are available; eg, AMD64's "04Ah Single-bit ECC Errors Recorded by Scrubber" [4]</td></tr>

<tr><td>Memory capacity</td><td>utilization</td><td>system-wide: <tt>free -m</tt>, "Mem:" (main memory), "Swap:" (virtual memory); <tt>vmstat 1</tt>, "free" (main memory), "swap" (virtual memory); <tt>sar -r</tt>, "%memused"; <tt>dstat -m</tt>, "free"; <tt>slabtop -s c</tt> for kmem slab usage; per-process: <tt>top</tt>/<tt>htop</tt>, "RES" (resident main memory), "VIRT" (virtual memory), "Mem" for system-wide summary</td></tr>
<tr><td>Memory capacity</td><td>saturation</td><td>system-wide: <tt>vmstat 1</tt>, "si"/"so" (swapping); <tt>sar -B</tt>, "pgscank" + "pgscand" (scanning); <tt>sar -W</tt>; per-process: 10th field (min_flt) from /proc/PID/stat for minor-fault rate, or dynamic tracing [5]; OOM killer: <tt>dmesg | grep killed</tt></td></tr>
<tr><td>Memory capacity</td><td>errors</td><td><tt>dmesg</tt> for physical failures; dynamic tracing, eg, SystemTap uprobes for failed malloc()s</td></tr>

<tr><td>Network Interfaces</td><td>utilization</td><td><tt>sar -n DEV 1</tt>, "rxKB/s"/max "txKB/s"/max; <tt>ip -s link</tt>, RX/TX tput / max bandwidth; /proc/net/dev, "bytes" RX/TX tput/max; nicstat "%Util" [6]</td></tr>
<tr><td>Network Interfaces</td><td>saturation</td><td><tt>ifconfig</tt>, "overruns", "dropped"; <tt>netstat -s</tt>, "segments retransmited"; <tt>sar -n EDEV</tt>, *drop and *fifo metrics; /proc/net/dev, RX/TX "drop"; nicstat "Sat" [6]; dynamic tracing for other TCP/IP stack queueing [7]</td></tr>
<tr><td>Network Interfaces</td><td>errors</td><td><tt>ifconfig</tt>, "errors", "dropped"; <tt>netstat -i</tt>, "RX-ERR"/"TX-ERR"; <tt>ip -s link</tt>, "errors"; <tt>sar -n EDEV</tt>, "rxerr/s" "txerr/s"; /proc/net/dev, "errs", "drop"; extra counters may be under /sys/class/net/...; dynamic tracing of driver function returns 76]</td></tr>

<tr><td>Storage device I/O</td><td>utilization</td><td>system-wide: <tt>iostat -xz 1</tt>, "%util"; <tt>sar -d</tt>, "%util"; per-process: iotop; <tt>pidstat -d</tt>; /proc/PID/sched "se.statistics.iowait_sum"</td></tr>
<tr><td>Storage device I/O</td><td>saturation</td><td><tt>iostat -xnz 1</tt>, "avgqu-sz" &gt; 1, or high "await"; <tt>sar -d</tt> same; LPE block probes for queue length/latency; dynamic/static tracing of I/O subsystem (incl. LPE block probes)</td></tr>
<tr><td>Storage device I/O</td><td>errors</td><td>/sys/devices/.../ioerr_cnt; <tt>smartctl</tt>; dynamic/static tracing of I/O subsystem response codes [8]</td></tr>

<tr><td>Storage capacity</td><td>utilization</td><td>swap: <tt>swapon -s</tt>; <tt>free</tt>; /proc/meminfo "SwapFree"/"SwapTotal"; file systems: "df -h"</td></tr>
<tr><td>Storage capacity</td><td>saturation</td><td>not sure this one makes sense - once it's full, ENOSPC</td></tr>
<tr><td>Storage capacity</td><td>errors</td><td><tt>strace</tt> for ENOSPC; dynamic tracing for ENOSPC; /var/log/messages errs, depending on FS</td></tr>

<tr><td>Storage controller</td><td>utilization</td><td><tt>iostat -xz 1</tt>, sum devices and compare to known IOPS/tput limits per-card</td></tr>
<tr><td>Storage controller</td><td>saturation</td><td>see storage device saturation, ...</td></tr>
<tr><td>Storage controller</td><td>errors</td><td>see storage device errors, ...</td></tr>

<tr><td>Network controller</td><td>utilization</td><td>infer from <tt>ip -s link</tt> (or /proc/net/dev) and known controller max tput for its interfaces</td></tr>
<tr><td>Network controller</td><td>saturation</td><td>see network interface saturation, ...</td></tr>
<tr><td>Network controller</td><td>errors</td><td>see network interface errors, ...</td></tr>

<tr><td>CPU interconnect</td><td>utilization</td><td>LPE (CPC) for CPU interconnect ports, tput / max</td></tr>
<tr><td>CPU interconnect</td><td>saturation</td><td>LPE (CPC) for stall cycles</td></tr>
<tr><td>CPU interconnect</td><td>errors</td><td>LPE (CPC) for whatever is available</td></tr>

<tr><td>Memory interconnect</td><td>utilization</td><td>LPE (CPC) for memory busses, tput / max; or CPI greater than, say, 5; CPC may also have local vs remote counters</td></tr>
<tr><td>Memory interconnect</td><td>saturation</td><td>LPE (CPC) for stall cycles</td></tr>
<tr><td>Memory interconnect</td><td>errors</td><td>LPE (CPC) for whatever is available</td></tr>

<tr><td>I/O interconnect</td><td>utilization</td><td>LPE (CPC) for tput / max if available; inference via known tput from iostat/ip/...</td></tr>
<tr><td>I/O interconnect</td><td>saturation</td><td>LPE (CPC) for stall cycles</td></tr>
<tr><td>I/O interconnect</td><td>errors</td><td>LPE (CPC) for whatever is available </td></tr>
</tbody></table>


### Software Resources

<table border="1" width="100%">
<tbody><tr><th>component</th><th>type</th><th>metric</th></tr>

<!--SW-START-->
<tr><td>Kernel mutex</td><td>utilization</td><td>With CONFIG_LOCK_STATS=y, /proc/lock_stat "holdtime-totat" / "acquisitions" (also see "holdtime-min", "holdtime-max") [8]; dynamic tracing of lock functions or instructions (maybe)</td></tr>
<tr><td>Kernel mutex</td><td>saturation</td><td>With CONFIG_LOCK_STATS=y, /proc/lock_stat "waittime-total" / "contentions" (also see "waittime-min", "waittime-max"); dynamic tracing of lock functions or instructions (maybe); spinning shows up with profiling (<tt>perf record -a -g -F 997 ...</tt>, <tt>oprofile</tt>, dynamic tracing)</td></tr>
<tr><td>Kernel mutex</td><td>errors</td><td>dynamic tracing (eg, recusive mutex enter); other errors can cause kernel lockup/panic, debug with kdump/<tt>crash</tt></td></tr>

<tr><td>User mutex</td><td>utilization</td><td><tt>valgrind --tool=drd --exclusive-threshold=...</tt> (held time); dynamic tracing of lock to unlock function time</td></tr>
<tr><td>User mutex</td><td>saturation</td><td><tt>valgrind --tool=drd</tt> to infer contention from held time; dynamic tracing of synchronization functions for wait time; profiling (oprofile, PEL, ...) user stacks for spins</td></tr>
<tr><td>User mutex</td><td>errors</td><td><tt>valgrind --tool=drd</tt> various errors; dynamic tracing of pthread_mutex_lock() for EAGAIN, EINVAL, EPERM, EDEADLK, ENOMEM, EOWNERDEAD, ...</td></tr>

<tr><td>Task capacity</td><td>utilization</td><td><tt>top</tt>/<tt>htop</tt>, "Tasks" (current); <tt>sysctl kernel.threads-max</tt>, /proc/sys/kernel/threads-max (max)</td></tr>
<tr><td>Task capacity</td><td>saturation</td><td>threads blocking on memory allocation; at this point the page scanner should be running (sar -B "pgscan*"), else examine using dynamic tracing</td></tr>
<tr><td>Task capacity</td><td>errors</td><td>"can't fork()" errors; user-level threads: pthread_create() failures with EAGAIN, EINVAL, ...; kernel: dynamic tracing of kernel_thread() ENOMEM</td></tr>

<tr><td>File descriptors</td><td>utilization</td><td>system-wide: <tt>sar -v</tt>, "file-nr" vs /proc/sys/fs/file-max; <tt>dstat --fs</tt>, "files"; or just /proc/sys/fs/file-nr; per-process: <tt>ls /proc/PID/fd | wc -l</tt> vs <tt>ulimit -n</tt></td></tr>
<tr><td>File descriptors</td><td>saturation</td><td>does this make sense?  I don't think there is any queueing or blocking, other than on memory allocation.</td></tr>
<tr><td>File descriptors</td><td>errors</td><td><tt>strace</tt> errno == EMFILE on syscalls returning fds (eg, open(), accept(), ...).</td></tr>
</tbody></table>

#### ulimit

ulimit 用於管理使用者對系統資源的訪問.

```
-a   顯示目前全部限制情況
-c   設定 core 檔案的最大值, 單位為區塊
-d   <資料節區大小> 程式資料節區的最大值, 單位為KB
-f   <檔案大小> shell 所能建立的最大檔案, 單位為區塊
-H   設定資源的硬性限制, 也就是管理員所設下的限制
-m   <記憶體大小> 指定可使用記憶體的上限, 單位為 KB
-n   <檔案描述符數目> 指定同一時間最多可開啟的 fd 數
-p   <緩衝區大小> 指定管道緩衝區的大小, 單位512位元組
-s   <堆疊大小> 指定堆疊的上限, 單位為 KB
-S   設定資源的彈性限制
-t   指定CPU使用時間的上限, 單位為秒
-u   <程序數目> 使用者最多可開啟的程序數目
-v   <虛擬記憶體大小> 指定可使用的虛擬記憶體上限, 單位為 KB
```

例如:

```
$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 127988
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 655360
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

注意, open socket 等資源拿到的也是 fd, 所以 `ulimit -n` 比較小除了檔案打不開, 還可能建立不了 socket 連結.
