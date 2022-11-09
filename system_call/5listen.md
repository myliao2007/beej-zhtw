# 5.5. listen()－有人會呼叫我嗎？

OK，是該改變步調的時候了。如果你不想要連線到一個遠端主機要怎麼做。

我說過，好玩就好，你想要等待進入的連線，並以某種方式處理它們。

這個過程有兩個步驟：你要先呼叫 listen()，接著呼叫 accept()（參考下一節）。

listen 呼叫相當簡單，不過需要一點說明：

```c
int listen(int sockfd, int backlog);
```

sockfd 是來自 socket() system call 的一般 socket file descriptor。backlog 是進入的佇列（incoming queue）中所允許的連線數目。這代表什麼意思呢？好的，進入的連線將會在這個佇列中排隊等待，直到你 accept() 它們（請見下節），而這限制了排隊的數量。多數的系統預設將這個數值限制為 20；你或許可以一開始就將它設定為 5 或 10。

再來，如同往常，listen() 會傳回 -1 並在錯誤時設定 errno。

好的，你可能會想像，我們需要在呼叫 listen() 以前呼叫 bind()，讓 server 可以在指定的 port 上執行。［你必須能告訴你的好朋友要連線到哪一個 port！］所以如果你正在 listen 進入的連線，你會執行的 system call 順序是：

```c
getaddrinfo();
socket();
bind();
listen();
/* accept() 從這裡開始 */
```

我只是留下範例程式的位置，因為它相當顯而易見。（在下面 accept() 章節中的程式碼會比較完整）。這整件事情真正需要技巧的部分是呼叫 accept()。
