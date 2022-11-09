# 5.11. gethostname()－我是誰？

比 getpeername() 更簡單的函式是 gethostname()，它會傳回你執行程式的電腦名稱，這個名稱之後可以用在 gethostbyname()，用來定義你本機電腦的 IP address。

有什麼更有趣的嗎？

我可以想到一些事情，不過這不適合 socket 程式設計，總之，下面是一段範例：

```c
#include <unistd.h>
int gethostname(char *hostname, size_t size);
```

參數很簡單：hostname 是指向字元陣列（array of chars）的指標，它會儲存函式傳回的主機名稱（hostname），而 size 是以 byte 為單位的主機名稱長度。

函式在成功執行時傳回 0，在錯誤時傳回 -1，並一樣設定 errno。
