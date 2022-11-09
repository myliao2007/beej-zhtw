# 0.5. 進階資料

下列是譯者認為值得推薦的 Linux/UNIX socket 網路程式設計書籍：

* Unix Network Programming Vol. 1：
  * W. Richard Stevens 大師的網路經典著作之一，對 socket 網路程式設計有更深入的介紹（如 raw socket、data link layer socket 或新型的 SCTP 傳輸層協定）。
* Effective TCP/IP Programming: 44 Tips to Improve Your Network Programs （中文版 - 深入研究 TCP/IP 網路程式設計）
  * Jon C. Snader 說明如何設計高效能、高可靠度的 socket 網路程式。
* The Linux Programming Interface（[中文版](http://tlpi.netdpi.net) - The Linux Programming Interface 國際中文版）
  * 包羅萬象的 Linux/Unix 系統程式 API 開發手冊大全，要設計 multiple threads 網路程式可參考這本書，細節請參考作者網站。
* 基礎からわかるTCP/IPネットワーク実験プログラミング第2版（中文版 - TCP/IP網路程式實驗與設計）
  * 村山公保介紹 port scan、scan route、TCP hijack、TCP RST 攻擊、TCP SYN 攻擊、ARP spoofing、網路分析 network sniffer（raw socket 與 data link layer socket）等網路攻擊、網路分析的原理及技術，並有原始程式碼教導如何用 Linux 與 C 語言設計這類網路程式。

更多資訊請參考作者在 10.1 節推薦的書籍。 上述技術屬使用者空間（user space）的網路應用程式，對於作業系統如何在核心空間（kernel space）實作網路堆疊及資料結構設計，可參考下列書籍：

* W. Richard Stevens, TCP/IP Illustrated - The Implementation, Volume 2, Addison-Wesley Professional, 1995.
* Rami Rosen, Linux Kernel Networking: Implementation and Theory, first edition, Apress, 2013.
* Christian Benvenuti, Understanding Linux network internals, O'Reilly, 2006.（中文版 - Linux 網路原理）
* Klaus Wehrle et al., Linux Networking Architecture, Prentice Hall, 2004.
* Thomas Herbert, The Linux TCP/IP Stack: Networking for Embedded Systems, 2nd edition, Charles River Media, 2006.
