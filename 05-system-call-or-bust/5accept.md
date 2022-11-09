# 5.6. accept()－ 謝謝你 call 3490 port

準備好，accept() call 是很奇妙的！會發生的事情就是：很遠的人會試著 connect() 到你的電腦正在 listen() 的 port。他們的連線會排隊等待被 accept()。你呼叫 accept()，並告訴它要取得擱置的（pending）連線。它會傳回給你專屬這個連線的一個新 socket file descriptor！那是對的，你突然有了兩個 socket file descriptor！原本的 socket file descriptor 仍然正在 listen 後續的連線，而新建立的 socket file descriptor 則是在最後要準備給 send() 與 recv() 用的。

呼叫如下：

```c
#include <sys/types.h>
#include <sys/socket.h>

int accept(int sockfd, struct sockaddr *addr, socklen_t *addrlen);
```

sockfd 是正在進行 listen() 的 socket descriptor。很簡單，addr 通常是一個指向 local struct sockaddr\_storage 的指標，關於進來的連線將往哪裡去的資訊（而你可以用它來得知是哪一台主機從哪一個 port 呼叫你的）。addrlen 是一個 local 的整數變數，應該在將它的位址傳遞給 accept() 以前，將它設定為 sizeof(struct sockaddr\_storage)。accept() 不會存放更多的 bytes（位元組）到addr。若它存放了較少的 bytes 進去，它會改變 addrlen 的值來表示。

有想到嗎？accept() 在錯誤發生時傳回 -1 並設定 errno。不過 BetCha 不這麼認為。

跟以前一樣，用一段程式範例會比較好吸收，所以這裡有一段範例程供你細讀：

```c
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>

#define MYPORT "3490" // 使用者將連線的 port
#define BACKLOG 10 // 在佇列中可以有多少個連線在等待

int main(void)
{
  struct sockaddr_storage their_addr;
  socklen_t addr_size;
  struct addrinfo hints, *res;
  int sockfd, new_fd;

  // !! 不要忘了幫這些呼叫做錯誤檢查 !!

  // 首先，使用 getaddrinfo() 載入 address struct：

  memset(&hints, 0, sizeof hints);
  hints.ai_family = AF_UNSPEC; // 使用 IPv4 或 IPv6，都可以
  hints.ai_socktype = SOCK_STREAM;
  hints.ai_flags = AI_PASSIVE; // 幫我填上我的 IP 

  getaddrinfo(NULL, MYPORT, &hints, &res);

  // 產生一個 socket，bind socket，並 listen socket：

  sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);
  bind(sockfd, res->ai_addr, res->ai_addrlen);
  listen(sockfd, BACKLOG);

  // 現在接受一個進入的連線：

  addr_size = sizeof their_addr;
  new_fd = accept(sockfd, (struct sockaddr *)&their_addr, &addr_size);

  // 準備好與 new_fd 這個 socket descriptor 進行溝通！
  .
  .
  .
```

一樣，我們會將 new\_fd socket descriptor 用於 send() 與 recv() 呼叫。若你只是要取得一個連線，你可以用 close() 關閉正在 listen 的 sockfd，以避免有更多的連線進入同樣的 port，若你有這個需要的話。
