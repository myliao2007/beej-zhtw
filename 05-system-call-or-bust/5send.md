# 5.7. send() 與 recv()－ 寶貝，我們來聊天！

這兩個用來通信的函式是透過 stream socket 或 connected datagram socket。若你想要使用常規的 unconnected datagram socket，你會需要參考底下的 sendto() 及 recvfrom() 的章節。

send() call：

```
int send(int sockfd, const void *msg, int len, int flags);
```

sockfd 是你想要送資料過去的 socket descriptor（無論它是不是 socket() 傳回的，或是你用 accept() 取得的）。msg 是一個指向你想要傳送資料之指標，而 len 是以 byte 為單位的資料長度。而 flags 設定為 0 就好。（更多相關的旗標資訊請見 send() man 使用手冊）。

一些範例程式如下：

```c
char *msg = "Beej was here!";
int len, bytes_sent;
.
.
.
len = strlen(msg);
bytes_sent = send(sockfd, msg, len, 0);
.
.
.
```

send() 會傳回實際有送出的 byte 數目，這可能會少於你所要傳送的數目！有時候你告訴 send() 要送整筆的資料，而它就是無法處理這麼多資料。它只會盡量將資料送出，並認為你之後會再次送出剩下沒送出的部分。

要記住，如果 send() 傳回的值與 len 的值不符合的話，你就需要再送出字串剩下的部分。好消息是：如果封包很小（比 1K 還要小這類的），或許有機會一次就送出全部的東西。

一樣，錯誤時會傳回 -1，並將 errno 設定為錯誤碼（error number）。

recv() 呼叫在許多地方都是類似的：

```c
int recv(int sockfd, void *buf, int len, int flags);
```

sockfd 是要讀取的 socket descriptor，buf 是要記錄讀到資訊的緩衝區（buffer），len 是緩衝區的最大長度，而 flags 可以再設定為 0。［關於旗標資訊的細節請參考 recv() 的 man 使用手冊］。

recv() 傳回實際讀到並寫入到緩衝區的 byte 數，而錯誤時傳回 -1（並設定相對的 errno）。

等等！ recv() 會傳回 0，這只能表示一件事情：遠端那邊已經關閉了你的連線！recv() 傳回 0 的值是讓你知道這件事情。

這樣很簡單，不是嗎？你現在可以送回資料，並往 stream sockets 邁進！嘻嘻！你是 UNIX 網路程式設計師了。
