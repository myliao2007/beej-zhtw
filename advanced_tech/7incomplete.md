# 7.3. 不完整傳送的後續處理

還記得前面的 send() 章節嗎？當時我不是提過 send() 可能不會將你所要求的資料全部送出嗎？也就是說，雖然你想要送出 512 bytes，但是 send() 只送出 412 bytes。那剩下的 100 個 bytes 到哪去了呢？

好的，其實它們還在你的緩衝區裡。因為環境不是你能控制的，kernel 會決定要不要用一個 chunk 將全部的資料送出，而現在，我的朋友，你可以決定要如何處理緩衝區中剩下的資料。

你可以寫一個像這樣的函式：

```c
#include <sys/types.h>
#include <sys/socket.h>

int sendall(int s, char *buf, int *len)
{
  int total = 0; // 我們已經送出多少 bytes 的資料
  int bytesleft = *len; // 我們還有多少資料要送
  int n;

  while(total < *len) {
    n = send(s, buf+total, bytesleft, 0);
    if (n == -1) { break; }
    total += n;
    bytesleft -= n;
  }

  *len = total; // 傳回實際上送出的資料量

  return n==-1?-1:0; // 失敗時傳回 -1、成功時傳回 0
}
```

在這個例子裡，s 是你想要傳送資料的 socket，buf 是儲存資料的緩衝區，而 len 是一個指標，指向一個 int 型別的變數，記錄了緩衝區中的資料數量。

函式在錯誤時傳回 -1（而 errno 仍然從呼叫 send() 設定）。還有，實際送出的資料數量會在 len 中回傳，除非有錯誤發生，不然這會跟你所要求要傳送的資料量相同。sendall() 會盡力將資料送出，不過如果有錯誤發生時，它就會立刻回傳給你。

為了完整性，這邊有一個呼叫函式的範例：

```c
char buf[10] = "Beej!";
int len;

len = strlen(buf);
if (sendall(s, buf, &len) == -1) {
  perror("sendall");
  printf("We only sent %d bytes because of the error!\n", len);
}
```

當封包的一部分抵達接收端（receiver end）時會發生什麼事情呢？如果封包的長度是會變動的（variable），接收端要如何知道另一端的封包何時開始與結束呢？

是的，你或許必須封裝（encapsulate）「還記得資料封裝（data encapsulation）這節的開頭那邊嗎？那邊有詳細說明」。
