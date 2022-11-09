# 3.1.1. Sub network (子網段)

為了結構化的理由，有時我們這樣宣告是很方便的：＂IP address 的前段是 IP address 的 network（網段），而後面的部分是 host（主機）。＂

例如：在 IPv4，你可能有 192.0.2.12，而我們可以說前面三個 bytes 是 network，而最後一個 byte 是 host。或者換個方式，我們能說 host 12 位在 network 192.0.2.0。［請參考我們如何將 host byte 清為零］。

接下來要講的是過時的資訊了！

真的嗎？

很久很久以前，有 subnets（子網路）的 "class"（分級），在這裡，位址的第一個、前二個或前三個 bytes 都是屬於 network 的一部分。如果你很幸運可以擁有一個 byte 的 network，而另外三個 bytes 是 host 位址，那在你的網路上，你有價值 24 bits 的 host number［大約兩千四百萬個位址左右］。這是一個 "Class A"（A 級）的網路；相對則是一個 "Class C"（C 級）的網路，network 有三個 bytes、而 host 只有一個 byte［256 個 hosts，而且還要再扣掉兩個保留的位址］。

所以，如同你所看到的，只有一些 Class A 網路，一大堆的 Class C 網路，以及一些中等的 Class B 網路。

IP address 的網段部分由 netmask（網路遮罩）決定，你可以將 IP address 與 netmask 進行 AND 位元運算，就能得到 network 的值。Netmask 一般看起來像是 255.255.255.0［如：若你的 IP 是 192.0.2.12，那麼使用這個 netmask 時，你的 network 就會是 192.0.2.12 AND 255.255.255.0 所得到的值：192.0.2.0］。

無庸置疑的，這樣的分級對於 Internet 的最終需求而言並不夠細膩；我們已經以相當快的速度在消耗 Class C 網路，這是我們都知道一定會耗盡的 Class，所以不用費心去想了。補救的方式是，要能接受任意個 bits 的 netmask，而不單純是 8、16 或 24 個而已。所以你可以有個 255.255.255.252 的 netmask，這個 netmask 能切出一個 30 個 bits 的 network 及 2 個 bits 的 host，這個 network 最多有四台 hosts［注意，netmask 的格式永遠都是：前面是一連串的 1，然後，後面是一連串的 0］。

不過一大串的數字會有點不好用，比如像 255.192.0.0 這樣的 netmask。首要是人們無法直觀地知道有多少個 bits 的 1；其次是這樣真的很不嚴謹。因此，後來的新方法就好多了。你只需要將一個斜線放在 IP address 後面，接著後面跟著一個十進制的數字用以表示 network bits 的數目，類似這樣：192.0.2.12/30。

或者在 IPv6 中，類似這樣：2001:db8::/32 或 2001:db8:5413:4028::9db9/64。
