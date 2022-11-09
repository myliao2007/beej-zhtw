# 5.4. connect()，嘿！你好。

咱們用幾分鐘的時間假裝你是個 telnet 應用程式，你的使用者命令你（就像 TRON 電影裡那樣）取得一個 socket file descriptor。你執行並呼叫 socket()。接著使用者告訴你連線到 "10.12.110.57" 的 port 23（標準 telnet port）。喲！你現在該做什麼呢？

你是很幸運的程式，你現在可以細讀 connect() 的章節，如何連線到遠端主機。所以努力往前讀吧！刻不容緩！

connect() call 如下：

```c
#include <sys/types.h>
#include <sys/socket.h>

int connect(int sockfd, struct sockaddr *serv_addr, int addrlen);
```

sockfd 是我們的好鄰居 socket file descriptor，如同 socket() 呼叫所傳回的，serv\_addr 是一個 struct sockaddr，包含了目的 port 及 IP 位址，而 addrlen 是以 byte 為單位的 server 位址結構之長度。

全部的資訊都可以從 getaddrinfo() 呼叫中取得，它很棒。

這樣有開始比較有感覺了嗎？我在這裡沒辦法知道，所以我只能希望是這樣沒錯。

我們有個範例，這邊我們用 socket 連線到 "www.example.com" 的 port 3490：

```c
struct addrinfo hints, *res;
int sockfd;

// 首先，用 getaddrinfo() 載入 address structs：

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC;
hints.ai_socktype = SOCK_STREAM;

getaddrinfo("www.example.com", "3490", &hints, &res);

// 建立一個 socket：

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

// connect!
connect(sockfd, res->ai_addr, res->ai_addrlen);
```

老學校的程式再次填滿了它們自己的 struct sockaddr\_ins 並傳給 connect()。如果你願意的話，你可以這樣做。請見上面 bind() 章節中類似的提點。

要確定有檢查 connect() 的回傳值，它在錯誤時會傳回 -1，並設定 errno 變數。

還要注意的是，我們不會呼叫 bind()。基本上，我們不用管我們的 local port number；我們只在意我們的目地（遠端 port）。Kernel 會幫我們選擇一個 local port，而我們要連線的站台會自動從我們這裡取得資訊，不用擔心。
