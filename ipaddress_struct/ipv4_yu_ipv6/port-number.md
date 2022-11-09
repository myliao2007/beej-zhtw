# 3.1.2. Port Number（連接埠號碼）

如果你還記得我之前跟你說過的分層網路模型（Layered Network Model），它將網路層（IP）與主機到主機間的傳輸層［TCP 與 UDP］分開。

我們要加快腳步了。

除了 IP address 之外［IP 層］，有另一個 TCP［stream socket］使用的位址，剛好 UDP［datagram socket］也用這個，就是 port number，這是一個 16-bit 的數字，就像是連線的本地端位址一樣。

將 IP address 想成飯店的地址，而 port number 就是飯店的房間號碼。這是貼切的比喻；或許以後我會用汽車工業來比喻。

你說想要有一台電腦能處理收到的電子郵件與網頁服務－你要如何在一台只有一個 IP address 的電腦上分辨這些封包呢？

好，Internet 上不同的服務都有已知的（well-known）port numbers。你可以在 Big IANA Port 清單 \[12] 中找到，如果你用的是 Unix 系統，你可以參考檔案 /etc/services。HTTP（網站）是 port 80、telnet 是 port 23、SMTP 是 port 25，而 DOOM 遊戲 \[13] 使用 port 666 等，諸如此類。Port 1024 以下通常是有特地用途的，而且要有作業系統管理員權限才能使用。

摁，這就是 port number 的介紹。

\[12] [http://www.iana.org/assignments/port-numbers](http://www.iana.org/assignments/port-numbers)

\[13] [http://en.wikipedia.org/wiki/Doom\_(video\_game\\](http://en.wikipedia.org/wiki/Doom\_\(video\_game/))
