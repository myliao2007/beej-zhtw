# 3.4. IP 位址，續集

還好你運氣不錯，有一堆函式讓你能夠控制 IP address，而不需要親自用 long 與 << 運算符來處理它們。

咱們說，你有一個 struct sockaddr\_in ina，而且你有一個 "10.12.110.57" 或 "2001:db8:63b3:1::3490" 這樣的一個 IP address 要儲存。你想要使用 inet\_pton() 函式將 IP address 轉換為數值與句號的符號，並依照你指定的 AF\_INET 或 AF\_INET6 來決定要儲存在 struct in\_addr 或 struct in6\_addr。

（"pton" 的意思是 "presentation to network"，你可以稱之為 "printable to network"，如果這樣會比較好記的話）。

這樣的轉換可以用如下的方式：

```c
struct sockaddr_in sa; // IPv4
struct sockaddr_in6 sa6; // IPv6
inet_pton(AF_INET, "192.0.2.1", &(sa.sin_addr)); // IPv4
inet_pton(AF_INET6, "2001:db8:63b3:1::3490", &(sa6.sin6_addr)); // IPv6
```

（小記：原本的老方法是使用名為 inet\_addr() 或是 inet\_aton() 的函式；這些都過時了，而且不適合在 IPv6 中使用）。

目前上述的程式碼片段還不是很可靠，因為沒有錯誤檢查。inet\_pton() 在錯誤時會傳回 -1，而若位址被搞爛了，則會傳回 0。所以在使用之前要檢查，並確認結果是大於 0 的。

好了，現在你可以將 IP address 字串轉換為它們的二進位表示。

還有其它方法嗎？

如果你有一個 struct in\_addr 且你想要以數字與句號印出來的話呢？

（呵呵，或者如果你想要以＂十六進位與冒號＂印出 struct in6\_addr］。在這個例子中，你會想要使用 inet\_ntop()函式［"ntop" 意謂 "network to presentation"－如果有比較好記的話，你可以稱它為 "network to printable"），像是這樣：

```c
// IPv4:
char ip4[INET_ADDRSTRLEN]; // 儲存 IPv4 字串的空間
struct sockaddr_in sa; // 假裝這會由某個東西載入
inet_ntop(AF_INET, &(sa.sin_addr), ip4, INET_ADDRSTRLEN);
printf("The IPv4 address is: %s\n", ip4);
// IPv6:
char ip6[INET6_ADDRSTRLEN]; // 儲存 IPv6 字串的空間
struct sockaddr_in6 sa6; // 假裝這會由某個東西載入
inet_ntop(AF_INET6, &(sa6.sin6_addr), ip6, INET6_ADDRSTRLEN);
printf("The address is: %s\n", ip6);
```

當你呼叫它時，你會傳遞位址的型別（IPv4 或 IPv6），該位址是一個指向儲存結果的字串，與該字串的最大長度。「有兩個 macro（巨集）可以很方便地儲存你想儲存的最大 IPv4 或 IPv6 位址字串大小：INET\_ADDRSTRLEN 與 INET6\_ADDRSTRLEN」。

（另一個要再次注意的是以前的方法：以前做這類轉換的函式名為 inet\_ntoa()，它已經過期了，而也在 IPv6 中也不適用）。

最後，這些函式只能用在數值的 IP address 上，它們不需要 DNS nameserver 來查詢主機名稱，如 "www.example.com"。你可以使用 getaddrinfo() 來做這件事情，如同你稍後會看到的。
