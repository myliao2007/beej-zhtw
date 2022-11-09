# 9.3. connect()

使用 socket 連線到伺服器

## 函式原型

```c
#include <sys/types.h>
#include <sys/socket.h>

int connect(int sockfd, const struct sockaddr *serv_addr,
            socklen_t addrlen);
```

## 說明

一旦你用 socket() call 建立了一個 socket descriptor 之後，你可以使用已知的 connect() system call 將這個 socket 連線到遠端伺服器。你該需做的是將 socket descriptor 與要連線的伺服器 address 傳遞給 connect()（喔，還有 address 的長度，這個通常會傳給這類的函式）。

通常這項資訊會透過呼叫 getaddrinfo() 取得，但是如果你願意，你也可以自行填寫 struct sockaddr。

若你尚未對這個 socket descriptor 呼叫 bind()，則會自動幫你綁定到你的 IP address 與一個隨機的 local port（本地連接埠）。如果你不是 伺服器，這樣還蠻不錯的，因為你真的可以不用管理的 local port 是多少了；你只要注意遠端的 port，這個可以在 serv\_addr 參數中設定。如果你有需要將 client socket 指定 IP address 與 port，那麼你可以選擇呼叫 bind() 來處理，不過這種情況是很少見的。

一旦 socket 已經用 connect() 完成連線，你就可以隨意使用這個 socket 來 send() 與 recv() 資料到你心裡想的地方。

特別注意：如果你用 connect() 與 SOCK\_DGRAM UDP socket 連線到遠端主機，若你願意，你也可以像使用 sendto() 與 recvfrom() 那樣來使用 send() 與 recv()。

## 傳回值

成功時傳回零，或者發生錯誤時傳回 -1（並設定相對應的 errno）。

## 範例

```c
// 連線到 www.example.com 的 port 80（http）

struct addrinfo hints, *res;
int sockfd;

// 首先，使用 getaddrinfo() 取得位址資訊：

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC; // use IPv4 or IPv6, whichever
hints.ai_socktype = SOCK_STREAM;

// 我們可以在下一行用 "http" 取代 "80"：
getaddrinfo("www.example.com", "http", &hints, &res);

// 建立一個 socket:

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

// 將 socket 連線到我們在 getaddrinfo() 裡指定的 address 與 port：

connect(sockfd, res->ai_addr, res->ai_addrlen);
```

## 參考

socket(), bind()
