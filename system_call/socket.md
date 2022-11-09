# 5.2. socket()－取得 File Descriptor！

我想可以不用再將 socket() 晾在旁邊了，我一定要講一下 socket() system call，這邊是程式片段：

```c
#include <sys/types.h>
#include <sys/socket.h>

int socket(int domain, int type, int protocol);
```

可是這些參數是什麼？

它們可以讓你設定想要的 socket 類型（IPv4 或 IPv6，stream 或 datagram 以及 TCP 或 UDP）。

以前的人得寫死這些值，而你也可以這樣做。（domain 是 PF\_INET 或 PF\_INET6，type 是 SOCK\_STREAM 或 SOCK\_DGRAM，而 protocol 可以設定為 0，用來幫給予的 type 選擇適當的協定。或者你可以呼叫 getprotobyname() 來查詢你想要的協定，"tcp" 或 "udp"）。

PF\_INET 就是你在初始化 struct sockaddr\_in 的 sin\_family 欄位會用到的，它是 AF\_INET 的親戚。實際上，它們的關係很近，所以其實它們的值也都一樣，而許多程式設計師會呼叫 socket()，並以 AF\_INET 取代 PF\_INET 來做為第一個參數傳遞。

現在，你可以去拿點牛奶跟餅乾，因為又是說故事時間了。

在很久很久以前，人們認為它應該是位址家族（address family），就是 "AF\_INET" 中的 "AF" 所代表的意思；而位址家族也要支援協定家族（protocol family）的幾個協定，這件事並沒有發生，而之後它們都過著幸福快樂的日子，結束。

所以最該做的事情就是在你的 struct sockaddr\_in 中使用 AF\_INET，而在呼叫 socket() 時使用 PF\_INET。

總之，這樣就夠了。你真的該做的只是使用呼叫 getaddrinfo() 得到的值，並將這個值直接餵給 socket()，像這樣：

```c
int s;
struct addrinfo hints, *res;

// 執行查詢
// [假裝我們已經填好 "hints" struct]
getaddrinfo("www.example.com", "http", &hints, &res);

// [再來，你應該要對 getaddrinfo() 進行錯誤檢查, 並走到 "res" 鏈結串列查詢能用的資料，
// 而不是假設第一筆資料就是好的［像這些範例一樣］
// 實際的範例請參考 client/server 章節。
s = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
```

socket() 單純傳回給你一個之後 system call 要用的 socket descriptor，錯誤時會回傳 -1。errno 全域變數會設定為該錯誤的值（細節請見 errno 的 man 使用手冊，而且你需要繼續閱讀並執行更多與它相關的 system call，這樣會比較有感覺）。
