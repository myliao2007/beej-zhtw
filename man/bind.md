# 9.2. bind()

將一個 socket 關聯到一個 IP address 及 port number

## 函式原型

```c
#include <sys/types.h>
#include <sys/socket.h>
int bind(int sockfd, struct sockaddr *my_addr, socklen_t addrlen);
```

## 說明

當遠端機器想要連線到你的伺服器程式時，他需要兩項資訊：IP address 及 port number。 而 bind() call 可以讓你做這件事。

首先你要先呼叫 getaddrinfo() 載入 struct sockaddr，以取得 destination address（目地位址）與 port 的資訊。然後你呼叫 socket() 以取得一個 socket descriptor，接著將這個 socket 與 address 傳遞給 bind()，然後 IP address 跟 port 就會很神奇的跟 socket 綁在一起了（使用真的魔法）！

如果你不知道你電腦的 IP address、或你知道你的電腦只有一個 IP address、或你不在意要用電腦上的哪個 IP address 時，你可以單純地將 getaddrinfo() 的 hints.ai\_flags 參數設定為 AI\_PASSIVE，這裡所做的事情就是將特別的值填入 struct sockaddr 的 IP address 欄位，以告訴 bind() 說它應該要自動填入這個主機的 IP address。

什麼？將什麼特別的值填入 struct sockaddr 的 IP address 欄位就可以讓它自動填上目前主機的 address 呢？

我會告訴你的，但是請記住這只適用於你手動填入 struct sockaddr 時，如果不是，請依據上述方式，使用 getaddrinfo() 傳回的結果。在 IPv4 中， struct sockaddr\_in 結構的 sin\_addr.s\_addr 會設定為 INADDR\_ANY；而在 IPv6 中， struct sockaddr\_in6 結構的 sin6\_addr 欄位會從全域變數 in6addr\_any 載入。或者，若你正宣告一個新的 struct in6\_addr，你可以將它初始化為 IN6ADDR\_ANY\_INIT。

最後，addrlen 參數應該設定為 my\_addr 的大小。

## 傳回值

成功時傳回零，或者錯誤時傳回 -1（且並依據錯誤設定 errno）

## 範例

```c
// 用現代化的 getaddrinfo() 方式：

struct addrinfo hints, *res;
int sockfd;

// 首先，用 getaddrinfo() 載入位址結構資料：

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC; // 使用 IPv4 或 IPv6，兩者皆可
hints.ai_socktype = SOCK_STREAM;
hints.ai_flags = AI_PASSIVE; // 幫我填上我的 IP

getaddrinfo(NULL, "3490", &hints, &res);

// 建立 socket：
//（這邊是簡化版，照理你應該要查看 "res" 鏈結串列的每個成員，並進行錯誤檢查！）

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

// 將 sockfd 綁定（bind）到我們傳遞給 getaddrinfo() 的那個 port：

bind(sockfd, res->ai_addr, res->ai_addrlen);

// 手動封裝 struct 的範例，IPv4

struct sockaddr_in myaddr;
int s;

myaddr.sin_family = AF_INET;
myaddr.sin_port = htons(3490);

// 你可以指定一個 IP address:
inet_pton(AF_INET, "63.161.169.137", &(myaddr.sin_addr));

// 或者你可以讓系統自動選一個 IP address：
myaddr.sin_addr.s_addr = INADDR_ANY;

s = socket(PF_INET, SOCK_STREAM, 0);
bind(s, (struct sockaddr*)&myaddr, sizeof myaddr);
```

## 參考

getaddrinfo(), socket(), struct sockaddr\_in, struct in\_addr
