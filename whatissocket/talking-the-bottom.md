# 2.2 底層漫談與網路理論

因為我只著重於協定的分層，該是談談網路是如何真正的運作的時候了，並呈現一些如何打造 SOCK\_DGRAM 封包的例子。就實務面，你或許可以跳過這一節，不過，這一節有很好的觀念背景，所以需要的人可以讀一讀。

![](../.gitbook/assets/encapsulation\_packet.png)

嘿！孩子們，該是學習資料封裝（Data Encapsulation）的時候了。這很重要，它就是如此重要，即使你是在加州這裡上的網路課程，也只能學到皮毛。

基本上我們會講到這些內容：

封包的誕生、將封包打包［封裝］到第一個協定［所謂的 TFTP 協定］的 header 中［幾乎是最底層了］，接著將全部的東西［包含 TFTP header］封裝到下一個協定中［所謂的 UDP］，接著下一個協定［IP］，最後銜接到硬體［實體］層上面的通訊協定［所謂的 Ethernet，乙太網路］。

當另一台電腦收到封包時，硬體會解開 Ethernet header，而 kernel 會解開 IP 與 UDP header，再來由 TFTP 程式解開 TFTP header，最後程式可以取得資料。

現在我最後要談個聲名狼藉的分層網路模型（Layered Network Model），亦稱 "ISO/OSI"。這個網路模型介紹了一個網路功能系統，有許多其它模型的優點。例如，你可以寫剛好一樣的 socket 程式，而不用管資料在實體上是怎麼傳送的［Serial、thin Ethernet、AUI 之類］。因為在底層的程式會幫你處理這件事。真正的網路硬體與拓樸對 socket 程式設計師而言是透明的。

不囉嗦，我將介紹這個成熟模型的分層。為了網路課程的測驗，要記住這些。

Application（應用層） Presentation（表現層） Session（會談層） Transport（傳輸層） Network（網路層） Data Link（資料鏈結層） Physical（實體層）

實體層就是硬體（serial、Ethernet 等）。而應用層你可以盡可能的想像，這是個使用者與網路互動的地方。

現在這個模型已經很普及，所以你如果願意的話，或許可以將它當作一本汽車修理指南來用。與 Unix 比較相容的分層模型有：

應用層（Application layer：telnet、ftp 等） 主機到主機的傳輸層（Transport layer：TCP、UDP） 網際網路層（Internet layer：IP 與路由遶送） 網路存取層（Network Access Layer：Ethernet、wi-fi、諸如此類）

此時，你或許能知道這幾層是如何對應到原始資料的封裝。

看看在打造一個簡單的封包需要多少工作呢？

天阿！你得自己用 "cat" 將資訊填入封包的 header 裡！

開玩笑的啦。

你對 stream socket 需要做的只有用 send() 將資料送出。而在 datagram socket 需要你做的是，用你所選擇的方式封裝該封包，並且用 sendto() 送出。Kernel 會自動幫你建立傳輸層與網路層，而硬體處理網路存取層。啊！真現代化的技術。

所以該結束我們短暫的網路理論之旅了。

喔！對了，我忘記告訴你我想要談談 routing（路由遶送）了。恩，沒事！沒關係，我不打算全部講完。

Router（路由器）會解開封包的 IP header，參考自己的 routing table（路由表）…。如果你真的很想知道，你可以讀 IP RFC \[9]。如果你永遠都不想碰它，其實你也可以過得很好。

\[9] [http://tools.ietf.org/html/rfc791](http://tools.ietf.org/html/rfc791)
