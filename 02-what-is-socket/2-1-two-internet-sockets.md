# 2.1 兩種 Internet Sockets

這是什麼？有兩種 Internet socket 嗎？

是的，喔不，我騙你的啦。其實有很多種的 Internet socket，只是我不想嚇到你，所以這裡我只打算討論兩種。不過我還會告訴你 "Raw Socket"，這是很強大的東西，所以你應該要好好研究一下它們。

譯註： 一般的 socket 只能讀取傳輸層以上（不含）的資訊，raw socket 一般用在設計 network sniffer，可以讓應用程式取得網路封包底層的資訊（如 TCP 層、IP 層，甚至 link layer socket 可以讀取到 link layer 層），並用以分析封包資訊。這份文件不會談到這類的程式設計，有興趣的讀者可自行參考：Unix Network Programming Vol. 1、libpcap 或 TCP/IP 網路程式實驗與設計「內容包含介紹網路攻擊 port scan、scan route、TCP hijack、TCP RST 攻擊、TCP SYN 攻擊、ARP spoofing、網路分析 network sniffer（raw socket 與 data link layer socket）等原理及技術，並有原始程式碼教導如何用 Linux 與 C 語言設計這類網路攻擊及網路分析程式」。

好吧，不聊了。到底有哪兩種 Internet socket 呢？

其中一個是「Stream Socket（串流式 Socket）」，而另一個是「Datagram Socket（訊息式 Socket）」，之後我們分別以「SOCK\_STREAM」與「SOCK\_DGRAM」來表示。Datagram sockets 有時稱為「免連線的 sockets（connectionless socket）」（雖然它們也可以用 connect()，如果你想這麼做的話，請見後面章節的 connect()）。

Stream socket 是可靠的、雙向連接的通信串流。若你以 "1、2" 的順序將兩個項目輸出到 socket，它們在另一端則會以 "1、2" 的順序抵達。而且不會出錯。

哪裡會用到 stream socket 呢？

好的，你應該聽過 telnet 程式吧，不是嗎？它就是用 stream socket。你所輸入的每個字都需要按照你所輸入的順序抵達，不是嗎？

網站瀏覽器所使用的 HTTP 通訊協定也是用 stream socket 取得網頁。的確，若你以 port 80 telnet 到一個網站，並輸入 "GET / HTTP/1.0"，然後按兩下 Enter，它就會輸出 HTML 給你！

Stream socket 是如何達成如此高品質的資料傳送呢？

它們用所謂的 "The Transmission Control Protocol"（傳輸控制協定），就是常見的 "TCP"（TCP 的全部細節請參考RFC 793\[6]）。TCP 確保你的資料可以依序抵達而且不會出錯。你以前可能聽過 "TCP" 是 "TCP/IP" 裡比較好的部分，這邊的 "IP" 是指 "Internet Protocol"（網際網路協定，請見 RFC 791\[7]）。IP 主要處理 Internet routing（網際網路的路由遶送），通常不保障資料的完整性。

酷喔。那 Datagram socket 呢？為什麼它們號稱免連線呢？這邊有什麼好主意？為什麼它們是不可靠的？

好，這裡說明一下現況：如果你送出一個 datagram（訊息封包），它可能會順利到達、可能不會按照順序到達，而如果它到達了，封包中的資料就是正確的。

譯註： TCP 會在傳輸層對將上層送來的過大訊息分割成多個分段（TCP segments），而 UDP 本身不會，UDP 是訊息導向的（message oriented），若 UDP 訊息過大時（整體封包長度超過 MTU），則會由 host 或 router 在 IP 層對封包進行分割，將一個 IP packet 分割成多個 IP fragments。IP fragmention 的缺點是，接收端的系統需要做 IP 封包的重組，將多個 fragments 重組合併為原本的 IP 封包，同時也會增加封包遺失的機率。如將一個 IP packet 分裂成多個 IP fragments，只要其中一個 IP fragment 遺失了，接收端就會無法順利重組 IP 封包，因而造成封包的遺失，若是高可靠度的應用，則上層協定需重送整個 packet 的資料。

\[6] [http://tools.ietf.org/html/rfc793](http://tools.ietf.org/html/rfc793) \[7] [http://tools.ietf.org/html/rfc791](http://tools.ietf.org/html/rfc791)

Datagram sockets 也使用 IP 進行 routing（路由遶送），不過它們不用 TCP；而是用 "UDP，User Datagram Protocol"（使用者資料包協定，請見 RFC 768 \[8]）。

為什麼它們是免連線的？

好，基本上，這跟你在使用 stream socket 時不同，你不用維護一個開啟的連線，你只需打造封包、給它一個 IP header 與目的資訊、送出，不需要連線。通常用 datagram socket 的時機是在沒有可用的 TCP stack 時；或者當一些封包遺失不會造成什麼重大事故時。這類應用程式的例子有：tftp（trivial file transfer protocol，簡易檔案傳輸協定，是 FTP 的小兄弟），多人遊戲、串流音樂、影像會議等。

＂等一下！tftp 和 dhcpd 是用來在一台主機與另一台之間傳輸二進制的應用資料！你如果想要應用程式能在資料抵達時正常運作，那資料就不能遺失阿！這是什麼黑魔法？＂

好，我的人族好友，tftp 與類似的程式會在 UDP 的上層使用它們自己的協定。比如：tftp 協定會報告每個收送的封包，接收端必須送回一個封包表示：＂我收到了！＂［一個＂ACK＂回報封包］。若原本封包的傳送端在五秒內沒有收到回應，這表示它該重送這個封包，直到收到 ACK 為止。在實作可靠的 SOCK\_DGRAM 應用程式時，這個回報的過程很重要。

對於無需可靠度的（unreliable）應用程式，如遊戲、音效、或影像，你只需忽略遺失的封包，或也許能試著用技巧彌補回來。（雷神之鎚的玩家都知道的一個技術名詞的影響：accursed lag。在這個例子中，"accursed"（受詛咒）這個字代表各種低級的意思）。

為什麼你要用一個不可靠的底層協定？

有兩個理由：第一個理由是速度，第二個理由還是速度。直接忘了遺失的這個封包是比較快的方式，相較之下，持續追蹤全部的封包是否安全抵達，並確保依序抵達是比較慢的。如果你想要傳送聊天訊息，TCP 很讚；不過如果你想要替全世界的玩家，每秒送出 40 個位置更新的資訊，且若遺失一到兩個封包並不會有太大的影響時，此時 UDP 是一個好的選擇。

\[8] [http://tools.ietf.org/html/rfc768](http://tools.ietf.org/html/rfc768)

譯註： stream（串流式）socket 是指應用程式要傳輸的資料就如水流（串流）在水管中傳輸一般，經由這個 stream socket 流向目的，串流式 socket 是資料會由傳輸層負責處理遺失、依序送達等工作，以在傳輸層確保應用程式所送出的資料能夠可靠且依序抵達，而應用程式若對資料有可靠與依序的需求時，使用 stream socket 就不用自行處理這類的工作。

datagram（訊息式）socket 是基於訊息導向的方式傳送資料，應用程式送出的每筆資料會如平信的概念送出，由於遶送封包的路徑可能會隨著網路條件而改變，每筆資料抵達的順序不一定會按照送出的順序抵達，並且如平信般，信件可能在遞送過程遺失，而寄件人並無法知道是否遞送成功。

初步簡單知道應用這兩種 sockets 的時機：當需要資料能完整送達目地時，就使用 stream socket，若是部分資料遺失也無妨時，就可以使用 datagram socket。
