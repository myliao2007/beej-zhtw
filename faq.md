# 8. 常見的問題

## 我可以從哪邊取得那些 header 檔案呢?

如果你的系統還沒有這些檔案，你可能就不需要它們。檢查你平台的使用手冊。若你在 Windows 上開發，那麼你只需要 `#include <winsock.h>`。

## 在 bind() 回報＂Address already in use＂（位址已經在使用中）時，我該怎麼辦呢？

你必須使用 setsockopt() 對 listen 的 socket 設定 SO\_REUSEADDR 選項。請參考 bind() 及 select() 章節的範例。

## 我該如何取得系統上已經開啟的 sockets 清單呢？

使用 netstat。細節請參考 man 使用手冊，不過你應該只要輸入下列的指令就能取得一些不錯的資訊：

```
$ netstat
```

## 我該如何檢視 routing table（路由表）呢？

執行 route 指令（多數的 Linux 系統是在 /sbin 底下），或者 netstat -r 指令。

## 如果我只有一台電腦，我該如何執行 client（客戶端）與 server（伺服器）程式呢？我需要網路來寫網路程式嗎？

你很幸運，全部的系統都有實作一個 loopback（繞迴）虛擬網路＂裝置＂，這個裝置位於 kernel 中，並假裝是張網路卡［這個介面就是 routing table 中所列出的 ＂lo＂］。

假裝你已經登入一個名為＂goat＂的系統，在一個視窗中執行 client，並在另一個視窗執行 server。

或者可以在背景啟動 server（server &），並在同樣的視窗執行 client。

loopback 裝置的功能是你可以執行 client goat 或 client localhost（因為 localhost 應該已經定義在你的 /etc/hosts 檔案），而你可以讓 client 與 server 溝通而不需要網路。

簡而言之，不需要改變任何的程式碼，就可以讓程式在無網路的單機系統上執行！好耶！

## 我該怎麼知道對方已經關閉連線呢？

你可以辨別出來，因為 recv() 會傳回 0。

## 我該如何實作一個＂ping＂工具呢？什麼是 ICMP 呢？我可以在哪裡找到更多關於 raw socket 與 SOCK\_RAW 的資料呢？

你對 raw socket 的全部疑問都可以在 W. Richard Stevens 的 UNIX Network Programming 書本上找到答案。還有，研究 Stevens 的 UNIX Network Programming 程式碼的 ping 子目錄，可以從線上下載 \[37]。

## 我該如何改變或縮短呼叫 connect() 的逾期時間呢？

我不想跟你說一樣的答案：「W. Richard Stevens 會告訴你」，我只能建議你參考 UNIX Network Programming 原始程式碼 \[38] 中的 /lib/connect\_nonb.c。

主要是你要用 socket() 建立一個 socket descriptor，將它設定為 non-blocking（非阻塞式），呼叫 connect()，而如果一切順利，connect() 會立刻傳回 -1，並將 errno 設定為 EINPROGRESS。接著你要呼叫 select() 並設定你想要的 timeout 時間，傳遞讀取及寫入集合（read and write sets）的 socket descriptor。如果 select() 沒有發生 timeout，這表示 connect() call 已經完成。此時，你必須使用 getsockopt() 設定 SO\_ERROR 選項以取得 connect() call 的傳回值，在沒有錯誤時，這個值應該是零。

最後，在你開始透過 socket 傳輸資料以前，你可能想要再將它設定回 blocking（阻塞）。

要注意的是，這樣做的好處是讓你的程式在連線（connecting）期間也可以另外做點事情。比如：你可以將 timeout 時間設定為類似 500 毫秒，並在每次 timeout 發生時更新螢幕畫面，然後再次呼叫 select()。當你已經呼叫了 select() 時，並且 timeout 了，像這樣重複了 20 次，你就會知道應該放棄這個連線了。

如我所述的，請參考 Stevens 的既完美又優秀的範例程式碼。

## 我該如何寫 Windows 的網路程式呢？

首先，請刪除 Windows，並安裝 Linux 或 BSD。;-)。不是的，實際上，只要參考導讀章節中的（Windows程式設計師要注意的事情）就可以了。

## 我該如何在 Solaris/SunOS 上編譯程式呢？在我嘗試編譯時，一直遇到錯誤！

發生 Linker（連結器）錯誤是因為 Sun 系統在不會自動編入 socket 函式庫。請參考導讀中的（Solaris/SunOS 程式設計師要注意的事情），有如何處理這個問題的範例。

## 為什麼 select() 跟 signal 合不來呢？

Signal 試圖要讓 blocked system call 傳回 -1，並將 errno 設定為 EINTR。當你用 sigaction() 設定了一個 signal handler（訊號處理常式）時，你可以設定 SA\_RESTART 旗標，這可以在 system call 被中斷之後重新啟用它。

這自然不會每次都管用。

我最愛的解法是使用一個 goto，你明白這會讓你的教授很憤怒，所以放手去做吧！

```c
select_restart:
if ((err = select(fdmax+1, &readfds, NULL, NULL, NULL)) == -1) {
  if (errno == EINTR) {
    // 某個 signal 中斷了我們，所以重新啟動
    goto select_restart;
  }
  // 這裡處理真正的錯誤：
  perror("select");
}
```

當然，在這個例子裡，你不需使用 goto；你可以用其它的 structures 來控制，但是我認為用 goto 比較簡潔。

## 要怎麼樣我才能實作呼叫 recv() 的 timeout 呢？

使用 select()！它可以讓你對正在讀取的 socket descriptors 指定 timeout 的參數。或者你可以將整個功能包在一個獨立的函式中，類似這樣：

```c
#include <unistd.h>
#include <sys/time.h>
#include <sys/types.h>
#include <sys/socket.h>

int recvtimeout(int s, char *buf, int len, int timeout)
{
  fd_set fds;
  int n;
  struct timeval tv;

  // 設定 file descriptor set
  FD_ZERO(&fds);
  FD_SET(s, &fds);

  // 設定 timeout 的資料結構 struct timeval
  tv.tv_sec = timeout;
  tv.tv_usec = 0;

  // 一直等到 timeout 或收到資料
  n = select(s+1, &fds, NULL, NULL, &tv);
  if (n == 0) return -2; // timeout!
  if (n == -1) return -1; // error

  // 資料一定有在這裡，所以執行一般的 recv()
  return recv(s, buf, len, 0);
}
.
.
.
// 呼叫 recvtimeout() 的範例：
n = recvtimeout(s, buf, sizeof buf, 10); // 10 second timeout

if (n == -1) {
  // 發生錯誤
  perror("recvtimeout");
}
else if (n == -2) {
  // 發生 timeout
} else {
  // 從 buf 收到一些資料
}
.
.
.
```

請注意到，recvtimeout() 在 timeout 的例子中會傳回 -2，那為什麼不是傳回 0 呢？好的，如果你還記得，在呼叫 recv() 傳回 0 值時所代表的意思是對方已經關閉了連線。所以該傳回值已經用過了，而 -1 表示＂錯誤＂，所以我選擇 -2 做為我的 timeout 表示。

## 我該如何在將資料送給 socket 以前將資料加密或壓縮呢？

一個簡單的加密方法是使用 SSL（secure sockets layer），只是這超過本文件的範疇了［細節請參考 OpenSSL 專案 \[39]］。

不過假設你想要安插或實作你自己的壓縮器（compressor）或加密系統（encryption system），這只不過是將你的資料想成在兩端點間執行連續的步驟，每個步驟以同樣的方式改變資料：

1. server 從檔案讀取資料［或是什麼地方］
2. server 加密/壓縮資料［你新增這個部分］
3. server 用 send() 送出加密資料

而另一邊則是：

1. client 用 recv() 接收加密資料
2. client 解密/解壓縮資料［你新增這個部分］
3. client 寫資料到檔案［或是什麼地方］

如果你正要壓縮與加密，只要記得先壓縮。:-)

只要 client 適當地還原 server 所做的事情，資料在另一端就會完好如初，不論你在中間增加了多少步驟。

所以你用我的程式碼所需要做的只有：找出讀資料與透過網路傳送［使用 send()］這中間的段落，並在那裡加上編碼的程式碼。

我一直看到的 ＂PF\_INET＂是什麼呢？他跟 AF\_INET 有關係嗎？

是的，有關係，細節請參考 socket() 章節。

## 我該怎麼寫一個 server，可以接受來自 client 的 shell 指令並執行指令呢？

為了簡化，我們說 client 的連線用 connect()、send() 以及 close()［即為，沒有後續的 system calls，client 沒有再次連線。］

client 的處理過程是：

1. 用 connect() 連線到 server
2. send("/sbin/ls > /tmp/client.out")
3. 用 close() 關閉連線

此時，server 正在處理資料並執行指令：

1. accept() client 的連線
2. 使用 recv(str) 接收命令字串
3. 用 close() 關閉連線
4. 用 system(str) 執行指令

注意！server 會執行全部 client 所送的指令，就像是提供了遠端的 shell 存取權限，人們可以連線到你的 server 並用你的帳號做點事情。例如：若 client 送出 ＂rm -rf \~＂會怎麼樣呢？這會刪掉你帳號裡的全部資料，就是這樣！

所以你學聰明了，你會避免 client 使用任何危險的工具，比如 foobar 工具：

```
if (!strncmp(str, "foobar", 6)) {
  sprintf(sysstr, "%s > /tmp/server.out", str);
  system(sysstr);
}
```

可是這樣還是不安全，沒錯：如果 client 輸入 "foobar; rm -rf \~" 呢？

最安全的方式是寫一個小機制，將命令參數中的非字母數字字元前面放個［'\\'］字元［如果適合的話，要包括空白］。

如你所見，當 server 開始執行 client 送來的東西時，安全性（security）是個問題。

## 我正在傳送大量資料，可是當我 recv() 時，它一次只收到 536 bytes 或 1460 bytes。可是如果我在我本機上執行，它就會一次就收到全部的資料，這是怎麼回事呢？

你碰到的是 MTU，即實體媒介（physical medium）能處理的最大尺寸。在本機上，你用的是 loopback 裝置，它可以處理 8K 或更多資料也沒有問題。但是在 Ethernet（乙太網路），它只能處理 1500 bytes（有 header），你碰到這個限制。透過 modem 的話，MTU 是 576 bytes（一樣，有 header），你遇到比較低的限制。

你必須確認有送出全部的資料。（細節請參考 sendall() 函式的實作）。一旦你有確認，那麼你就需要在迴圈中呼叫 recv()，直到收到全部的資料。

對於使用多重呼叫 recv() 來接收完整資料封包的細節，請參考資料封裝（Son of Data Encapsulation）一節。

## 我用的是 Windows 系統，而且我沒有 fork() system call 或任何的 struct sigaction 可以用，該怎麼辦呢？

如果你問的是它們在哪裡，它們會在 POSIX 函式庫裡，這個會包裝在你的編譯器中。因為我沒有 Windows 系統，所以我真的無法回答你，不過我似乎記得 Microsoft 有一個 POSIX 相容層，那裏會有 fork()（而且甚至會有 sigaction）。

在 VC++ 的使用手冊搜尋 "fork" 或 "POSIX"，看它是否能給你什麼線索。

如果這樣一點都沒有用，拿掉 fork()/sigaction 這些東西，用 Win32 中等價的函式來取代：CreateProcess()。我不知道怎麼用CreateProcess()，它有多的數不清的參數，不過在 VC++ 的文件中應該可以找到怎麼使用它。

## 我在防火牆（firewall）後面，我該如何讓防火牆外面的人知道我的 IP 位址，讓他們可以連線到我的電腦呢？

毫無疑問地，防火牆的目的就是要防止防火牆外面的人連到防火牆裡面的電腦，所以你讓他們進來基本上會被認為是安全上的漏洞。

但也不是說完全不行，有一個方法，你仍然可以透過防火牆頻繁的進行 connect()，如果防火牆是使用某種偽裝（masquerading）或 NAT 或類似的方式。你只要讓程式一直在做初始化連線，那麼你有機會成功的。

如果這樣還不是很滿意，你可以要求系統管理員在防火牆開一個小洞（hole），讓人們可以連進你的電腦。防火牆可以透過 NAT 軟體或 proxy（代理）或類似的方法將封包轉送給你。

要留意，不要對防火牆中的一個小洞掉以輕心。你必須確保你不會放壞人進來存取內部網路；如果你是新手，做軟體安全是遠遠難於你的想像。

不要讓你的系統管理員對我發脾氣 ;-)

## 我該怎麼寫 packet sniffer 呢？我要怎麼將我的 Ethernet interface（網路卡）設定為 promiscuous mode（混雜模式）呢？

這些事情是在底層運作的，當網路卡設定為 "promiscuous mode" 時，它會轉送全部的封包給作業系統，而不只是位址屬於這台電腦的封包而已。［我們這裡談的是 Ethernet 層的位址，而不是 IP 位址，可是因為 ethernet 是在 IP 底層，所以全部的 IP 位址實際上都會轉送。細節請參考＂底層漫談與網路理論＂一節］。

這是 packet sniffer 如何運作的基礎，它將網路介面卡設定為 promiscuous mode，接著 OS 會收到經過網路線的每個封包，你會有一個可以用來讀取資料的某種型別 socket。

毫無疑問地，這個問題的答案依平台而異，不過如果你用 Google 搜尋，例如："windows promiscuous ioctl"，你或許會在某個地方找到，看起來跟 Linux Journal \[40] 中寫的一樣好的。

## 我該如何為 TCP 或 UDP socket 設定一個自訂的 timeout 值呢？

這個按照你的系統而定，你可以在網路搜尋 SO\_RCVTIMEO 與 SO\_SNDTIMEO（用在 setsockopt()），看看是否你的系統有支援這樣的功能。

Linux man 使用手冊建議使用 alarm() 或 setitimer() 作為替代品。

## 我要如何辨別哪些 ports 可以使用呢？有沒有＂官方＂的 port numbers 呢？

通常這不會有問題，如果你正在寫像 web server 這樣的程式，那麼在你的程式使用 port 80 是個好主意。如果你只是想要寫自己的 server，那麼隨機選擇一個 port［不過要大於 1023］，然後試試看。

如果 port 已經在使用中，你將會在嘗試 bind() 時遇到 "Address already in use" 錯誤。選擇另一個 port。［利用 config 組態檔或命令列參數設定，讓你的軟體使用者能指定 port 也是個不錯的想法］。

有一個官方的 port nubmer \[41] 清單，由 Internet Assigned Numbers Authority（IANA）所維護的。在清單中的 port（超過 1023）並不代表你就不能使用，比如，Id 軟體的 DOOM 跟 ＂mdqs＂ 用一樣的 port，不管那是什麼，最重要的是在同一台機器上沒有人用掉你要用的 port。

\[37] [http://www.unpbook.com/src.html](http://www.unpbook.com/src.html)

\[38] [http://www.unpbook.com/src.html](http://www.unpbook.com/src.html)

\[39] [http://www.openssl.org/](http://www.openssl.org/)

\[40] [http://interactive.linuxjournal.com/article/4659](http://interactive.linuxjournal.com/article/4659)

\[41] [http://www.iana.org/assignments/port-numbers](http://www.iana.org/assignments/port-numbers)
