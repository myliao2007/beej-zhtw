# 7.1. Blocking（阻塞）

你聽過 blocking，只是它在這裡代表什麼鬼東西呢？簡而言之，"block" 就是 "sleep（休眠）" 的技術術語。你以前執行 listener 時可能有注意到，它只是一直在那邊等待，直到有封包抵達才繼續動作。

很多函式都會 block，accept() 會 block，全部的 recv() 函式都會 block。原因是它們有權這麼做。

當你先用 socket() 建立 socket descriptor 時，kernel（核心）會將它設定為 blocking。若你不想要 blocking socket，你必須呼叫 fcntl()：

```c
#include <unistd.h>
#include <fcntl.h>
.
.
.
sockfd = socket(PF_INET, SOCK_STREAM, 0);
fcntl(sockfd, F_SETFL, O_NONBLOCK);
.
.
.
```

將 socket 設定為 non-blocking（非阻塞），你就能 "poll（輪詢）" socket 以取得資訊。如果你試著讀取 non-blocking socket，而 socket 沒有資料時，函式就不會發生 block，而是傳回 -1，並將 errno 設定為 EWOULDBLOCK。

然而，一般來說，這樣 polling 是不好的想法。如果你讓程式一直忙著查 socket 上是否有資料，則會浪費 CPU 的時間，這樣是不合適的。比較漂亮的解法是利用下一節的 select() 來檢查 socket 是否有資料需要讀取。
