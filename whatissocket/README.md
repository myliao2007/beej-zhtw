# 2. 何謂 Socket

你一直聽到人家在講 "sockets"，你可能也想知道這些是什麼東西。

好的，其實它們就是：「利用標準 UNIX file descriptors（檔案描述符）與其它程式溝通的一種方式」。

「什麼？」

OK，你可能有聽過有些駭客（hacker）說過：「我的天呀！在 UNIX 系統中的任何東西都可以視為檔案！」

他說的是真的，不是假的。當 UNIX 程式要做任何類型的 I/O 時，它們會讀寫 file descriptor。

File descriptor 單純是與已開啟檔案有關的整數。只是「關鍵在於」該檔案可以是一個網路連線、FIFO、pipe（管線）、terminal（終端機）、真實的磁碟檔案、或只是相關的東西。

在 UNIX 所見都是檔案！

所以當你想要透過 Internet（網際網路）跟其它的程式溝通時，你需要透過一個 file descriptor 來達成，這點你一定要相信。

「那麼，Smarty-Pants 先生，我在哪裡可以取得這個用在網路通信的 file descriptor 呢？」

這可能是你現在心裡的問題，我會跟你說的：你可以呼叫 socket() system routine（系統常式）。它會傳回 socket descriptor，你可以用精心設計的 send() 與 recv() socket calls（man send、man recv）來透過 socket descriptor 進行通信。

「不過，嘿嘿！」

現在你可能在想：「既然只是個 file descriptor，為什麼我不能用一般的 read() 與 write() call 透過 socket 進行通信，而要用這什麼鬼東西阿？」

簡而言之：「可以！」

完整說來就是：「可以，不過 send() 與 recv() 讓你能對資料傳輸有更多的控制權」。

「接下來呢？」

這麼說吧：有很多種 sockets，如 DARPA Internet Sockets（網際網路位址）、本機端上的路徑名稱（path names on a local node，UNIX Sockets）、CCITT X.25 位址（你可以放心忽略 X.25 Sockets），可能還有其它的，要看你用的是哪種 UNIX 系統。在這裡我們只討論第一種：Internet Sockets。
