# 3.2. Byte Order（位元組順序）

一直都以為有兩種 byte orderings，現在才知道，其實只有一種。

開玩笑的，但是其中一個的確比較受推崇 :-)

這真的不容易解釋，所以我只能胡扯：你的電腦可能背著你用相反的順序來儲存 bytes。我知道！以前沒人跟你說過。

Byte Order 其實就是，若你想要用兩個 bytes 的十六進制數字來表示資料，比如說 b34f，你可以將它以 b34f 的順序儲存，這件事在 Internet 世界的每個人一般都可以同意。這很合理，而且 Wilford Brimley \[14] 也會跟你說，這麼做是對的。由於這個數字是先儲存比較大的那一邊（big end），所以稱為 Big-Endian。

不幸的是，世界上的電腦那麼多，而 Intel 或 Intel 相容處理器就將 bytes 反過來儲存，所以 b34f 存在記憶體中的順序就是 4fb3，這樣的儲存方式稱為 Little-Endian。

不過，請等等。我還沒解釋名詞哩！

照理說，Big-Endian 又稱為 Network Byte Order，因為這個順序與我們網路類型順序一樣。

你的電腦會以 Host Byte Order 儲存數字，如果是 Intel 80x86，Host Byte Order 是 Little-Endian；若是 Motorola 68k，則 Host Byte Order 是 Big-Endian；若是 PowerPC，Host Byte Order 就是 … 恩，這要看你的 PowerPC 而定。

大多數當你在打造封包或填寫資料結構時，你需要確認你的 port number 跟 IP address 都是 Network Byte Order。只是如果你不知道本機的 Host Byte Order，那該怎麼做呢？

好消息是，我們可以直接假設，全部電腦的 Host Byte Order 都不是 Network Byte Order，然後每次都用函式將值轉換為 Network Byte Order。如果有必要，函式會發動魔法般的轉換，而這個方式會讓你的程式碼能更方便地移植到不同 endian 的機器。

你可以轉換兩種型別的數值：short［兩個 bytes］與 long［四個 bytes］。這些函式也可以用在 unsigned 變數。比如說，你想要將 short 從 Host Byte Order 轉換為 Network Byte Order， 'h' 代表 "host"， 'n' 代表 "network"，而 's' 代表 "short"，所以是：h-to-n-s，或者 htons()［讀做："Host to Network Short"］。

這真是太簡單了…

你可以用任何你想要的方式來組合 'n'、 'h'、's' 與 'l'，不過別用太蠢的組合，比如：沒有這樣的函式 stolh()［"Short to Long Host"］，沒有這種東西，不過有下面這些：

```c
htons() host to network short
htonl() host to network long
ntohs() network to host short
ntohl() network to host long
```

基本上，你需要在送出封包以前將數值轉換為 Network Byte Order，並在收到封包之後將數值轉回 Host Byte Order。

抱歉，我不知道 64-bit 的差異，如果你想要處理浮點數的話，可以參考 7.4 節。

\[14] [http://en.wikipedia.org/wiki/Wilford\_Brimley](http://en.wikipedia.org/wiki/Wilford\_Brimley)

如果我沒特別強調的話，本文中的數值預設是視為 Host Byte Order。
