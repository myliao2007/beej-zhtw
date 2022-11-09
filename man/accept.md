# 9.1. accept()

接受從 listening socket 進來的連線

## 函式原型（Prototype）

```c
#include <sys/types.h>
#include <sys/socket.h>

int accept(int s, struct sockaddr *addr, socklen_t *addrlen);
```

## 說明

一旦你完成取得 SOCK\_STREAM socket 並將 socket 設定好可以用來 listen() 進來的連線，接著你就能呼叫 accept() 讓自己能取得一個新的 socket descriptor，做為後續與新連線 client 的溝通。

原本用來 listen 的 socket 仍然還是會留著，當有新連線進來時，一樣是用 accept() call 來接受新的連線。

* s：listen() 中的 socket descriptor。
* addr：這裡會填入連線到你這裡的 client 位址。
* addrlen：這裡會填入 addr 參數中傳回的資料結構大小。如果你確定你知道一定會收到的是 struct sockaddr\_in，你可以放心忽略這個參數，因為這就是你原本傳遞的 addr 型別。

accept() 通常會 block（阻塞），而你可以使用 select() 事先取得 listen 中的 socket descriptor 狀態，檢查 socket 是否就緒可讀（ready to read）。若為就緒可讀，則表示有新的連線正在等待被 accept()！另一個方式是將 listen 中的 socket 使用 fcntl() 設定 O\_NONBLOCK 旗標，然後 listen 中的 socket descriptor 就不會造成 block，而是傳回 -1，並將 errno 設定為 EWOULDBLOCK。

由 accept() 傳回的 socket descriptor 是如假包換的 socket descriptor，開啟並與遠端主機連線，如果你要結束與 client 的連線，必須用 close() 關閉。

## 傳回值

accept() 傳回新連線的 socket descriptor，錯誤時傳回 -1，並將 errno 設定適當的值。

## 範例

```c
struct sockaddr_storage their_addr;
socklen_t addr_size;
struct addrinfo hints, *res;
int sockfd, new_fd;

// 首先：使用 getaddrinfo() 填好位址結構：

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC; // 使用 IPv4 或 IPv6，都可以
hints.ai_socktype = SOCK_STREAM;
hints.ai_flags = AI_PASSIVE; // 幫我填好我的 IP

getaddrinfo(NULL, MYPORT, &hints, &res);

// 建立一個 socket、bind 它，並對它進行 listen：

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
bind(sockfd, res->ai_addr, res->ai_addrlen);
listen(sockfd, BACKLOG);

// 現在接受進入的連線：

addr_size = sizeof their_addr;
new_fd = accept(sockfd, (struct sockaddr *)&their_addr, &addr_size);

// 準備與 socket descriptor new_fd 溝通！
```

## 參考

socket(), getaddrinfo(), listen(), struct sockaddr\_in
