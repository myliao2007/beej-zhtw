# 3.1. IPv4 與 IPv6

在 Ben Kenobi 還是叫 Obi Wan Kenobi 的那段過去的美好時光，有個很棒的 network routing system（網路路由系統），稱為 Internet Protocol Version 4（網際網路協定第四版），又稱為 IPv4。它的位址是由四個 bytes 組成（亦稱為四個＂octets＂），而格式是由句點與數字組成，像是這樣：192.0.2.111。

你或許曾經看過。

實際上，在撰寫本文時，幾乎整個 Internet（網際網路）的每個網站都還是使用 IPv4。

每個人跟 Obi Wan 都很開心，一切都是如此美好，直到某個名為 Vint Cerf 的人提出質疑，警告所有人 IPv4 address 即將耗盡。

Vint Cerf \[10] 除了提出即將到來的 IPv4 危機警告，他本身還是有名的 Internet 之父，所以我真的沒資格能評論他的判斷。

你說的是耗盡 address 嗎？會發生什麼事呢？其實我的意思是，32-bit 的 IPv4 address 有幾十億個 IP address，我們真的有幾十億台的電腦在用嗎？

是的。在一開始大家也是認為這樣就夠用了，因為當時只有一些電腦，而且每個人認為幾十億是不可能用完的大數目，還很慷慨的分給某些大型組織幾百萬個 IP address 供他們自己使用（例如：Xerox、MIT、Ford、HP、IBM、GE、AT\&T 及某個名為 Apple 的小公司，族繁不及備載）。

不過現實狀況是，如果不是有些變通的方法，我們早就用光 IPv4 位址了。

我們現在生活於每個人、每台電腦、每部計算機、每隻電話、每部停車計時收費器、以及每條小狗（為什麼不行？）都有一個 IP address 的年代，因此，IPv6 誕生了。

因為 Vint Cerf 可能是不朽的（即使他的軀殼終究該回歸自然，我也希望永遠不會發生，不過他的精神或許已經以某種超智慧的 ELIZA \[11] 程式存在於 Internet2 的核心），應該沒有人想要因為下一代的網際網路協定沒有足夠的位址，然後又聽到他說：「我要告訴你們一件事 ...」。

那你有什麼建議嗎？

我們需要更多的位址，我們需要不止兩倍以上的位址、不止幾十億倍、千兆倍以上，而是 79 乘以 百萬 乘以 十億 乘以 兆倍以上的可用位址！你們大家將會見識到的。

你說：「Beej，真的嗎？我還是有許多可以質疑這個大數字的理由。」

好的，32 bits 與 128 bits 的差異聽起來似乎不是很多；它只多了 96 個 bits 而已，不是嗎？不過請記得，我們所談的是等比級數；32 bits 表示個 40 億的數字［2 的 32 次方］，而128 bits 表示的大約是 340 個兆兆兆的數字［2 的 128 次方］，這相當於宇宙中的每顆星星都能擁有一百萬個 IPv4 Internets。

大家順便忘了 IPv4 的句號與數字的長相吧；現在我們有十六進制的表示法，每兩個 bytes 間以冒號分隔，類似這樣：

2001:0db8:c9d2:aee5:73e3:934a:a5ae:9551。

這還不是全部呢！大部分的時候，你的 IP address 裡面會有很多個零，而你可以將它們壓縮到兩個冒號間，你也可以去掉每個 byte pair（位元組對）裡開頭的零。例如，這些成對位址中的兩個位址是相等的：

|                                         |
| --------------------------------------- |
| 2001:0db8:c9d2:0012:0000:0000:0000:0051 |
| 2001:db8:c9d2:12::51                    |
|                                         |
| 2001:0db8:ab00:0000:0000:0000:0000:0000 |
| 2001:db8:ab00::                         |
|                                         |
| 0000:0000:0000:0000:0000:0000:0000:0001 |
| ::1                                     |

\[10] [http://en.wikipedia.org/wiki/Vinton\_Cerf](http://en.wikipedia.org/wiki/Vinton\_Cerf)

\[11] [http://en.wikipedia.org/wiki/ELIZA](http://en.wikipedia.org/wiki/ELIZA)

位址 ::1 是個 loopback（遶回）位址，它永遠只代表「我現在執行的這台電腦」，在 IPv4 中，loopback 位址是 127.0.0.1。

最後，你可能會遇到 IPv6 與 IPv4 相容的模式。例如，如果你願意的話，你可以將 IPv4 address 192.0.2.33 以 IPv6 位址表示，可以使用如下的符號：「::ffff:192.0.2.33」。

因為所謂的自信，所以 IPv6 的發明人很有把握的保留了兆來兆去的位址，不過說實在的，我們有這麼多位址，誰能算清楚呢？

還剩下很多位址可以分配給星系中每個行星的每個男人、女人、小孩、小狗跟停車計時收費器。相信我，星系中的每個行星都有行車計時收費器。你明白這是真的。
