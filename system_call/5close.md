# 5.9. close() 與 shutdown()－ 你消失吧！

呼！你已經整天都在 send() 與 recv()了。你正準備要關閉你 socket descriptor 的連線，這很簡單，你只要使用常規的 UNIX file descriptor close() 函式：

```c
close(sockfd);
```

這會避免對 socket 做更多的讀寫。任何想要對這個遠端的 socket 進行讀寫的人都會收到錯誤。

如果你想要能多點控制 socket 如何關閉，可以使用 shutdown() 函式。它讓你可以切斷單向的通信，或者雙向（就像是 close() 所做的），這是函式原型：

```c
int shutdown(int sockfd, int how);
```

sockfd 是你想要 shutdown 的 socket file descriptor，而 how 是下列其中一個值：

* 0 不允許再接收資料
* 1 不允許再傳送資料
* 2 不允許再傳送與接收資料（就像 close()）

shutdown() 成功時傳回 0，而錯誤時傳回 -1（設定相對的 errno）。

若你在 unconnected datagram socket 上使用 shutdown()，它只會單純的讓 socket 無法再進行 send() 與 recv() 呼叫（要記住你只能在有 connect() 到 datagram socket 的時候使用）。

重要的是 shutdown() 實際上沒有關閉 file descriptor，它只是改變了它的可用性。如果要釋放 socket descriptor，你還是需要使用 close()。

沒了。

（除了要記得的是，如果你用 Windows 與 Winsock，你應該要呼叫 closesocket() 而不是 close()）。
