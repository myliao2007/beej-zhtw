# 5.8. sendto() 與 recvfrom()－ 來點 DGRAM

我聽到你說，「這全部都是上等的好貨，可是我該如何使用 unconnected datagram socket 呢？」

沒問題，朋友。我們正要講這件事。

因為 datagram socket 沒有連線到到遠端主機，猜猜看，我們在送出封包以前會需要哪些資訊呢？

對！目的位址！在這裡搶先看：

```c
sendto(int sockfd, const void *msg, int len, unsigned int flags,
       const struct sockaddr *to, socklen_t tolen);
```

如你所見，這個呼叫基本上與呼叫 send() 一樣，只是多了兩個額外的資訊。to 是一個指向 struct sockaddr（這或許是另一個你可以在最後轉型的 struct sockaddr\_in 或 struct sockaddr\_in6 或 struct sockaddr\_storage）的指標，它包含了目的 IP address 與 port。tolen 是一個 int，可以單純地將它設定為 sizeof \*to 或 sizeof(struct sockaddr\_storage)。

為了能自動處理目的位址結構（destination address structure），你或許可以用底下的 getaddrinfo() 或 recvfrom()，或者你也可以手動填上。

如同 send()，sendto() 會傳回實際已傳送的資料數量（一樣，可能會少於你要傳送的資料量！）而錯誤時傳回 -1。

recv() 與 recvfrom() 也是差不多的。recvfrom() 的對照如下：

```c
int recvfrom(int sockfd, void *buf, int len, unsigned int flags,
            struct sockaddr *from, int *fromlen);
```

一樣，它跟 recv() 很像，只是多了兩個欄位。from 是指向 local struct sockaddr\_storage 的指標，這個資料結構包含了封包來源的 IP address 與 port。fromlen 是指向 local int 的指標，應該要初始化為 sizeof \*from 或是 sizeof(struct sockaddr\_storage)。當函式傳回時，fromlen 會包含實際上儲存於 from 中的位址長度。

recvfrom() 傳回接收的資料數目，或在發生錯誤時傳回 -1［並設定相對的 errno］。

所以這裡有個問題：為什麼我們要用 struct sockaddr\_storage 做為 socket 的型別呢？為什麼不用 struct sockaddr\_in 呢？

因為你知道的，我們不想要讓自己綁在 IPv4 或 IPv6，所以我們使用通用的泛型 struct sockaddr\_storage，我們知道這樣有足夠的空間可以用在 IPv4 與 IPv6。

（所以 ... 這裡有另一個問題：為什麼不是 struct sockaddr 本身就可以容納任何位址呢？我們甚至可以將通用的 struct sockaddr\_storage 轉型為通用的 struct sockaddr！似乎沒什麼關係又很累贅啊。答案是，它就是不夠大，我猜在這個時候更動它會有問題，所以他們就弄了一個新的。）

記住，如果你 connect() 到一個 datagram socket，你可以在你全部的交易中只使用 send() 與 recv()。socket 本身仍然是 datagram socket，而封包仍然使用 UDP，但是 socket interface 會自動幫你增加目的與來源資訊。
