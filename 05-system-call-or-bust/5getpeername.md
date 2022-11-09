# 5.10. getpeername()－你是誰？

這個函式很簡單。

它太簡單了，我幾乎不想給它一個獨立的章節，雖然還是給了。

getpeername() 函式會告訴你另一端連線的 stream socket 是誰，函式原型如下：

```c
#include <sys/socket.h>
int getpeername(int sockfd, struct sockaddr *addr, int *addrlen);
```

sockfd 是連線的 stream socket 之 descriptor，addr 是指向 struct sockaddr（或 struct sockaddr\_in）的指標，這個資料結構儲存了連線另一端的資訊，而 addrlen 則是指向 int 的指標，應該將它初始化為 sizeof \*addr 或 sizeof(struct sockaddr)。

函式在錯誤時傳回 -1，並設定相對的 errno。

一旦你取得了它們的位址，你就可以用 inet\_ntop()、getnameinfo() 或 gethostbyaddr() 印出或取得更多的資訊。不過你無法取得它們的登入帳號。

（好好好，如果另一台電腦執行的是 ident daemon 就可以）。然而，這個已經超出本文的範圍，更多資訊請參考 RFC 1413 \[19]。
