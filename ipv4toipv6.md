# 4. 從 IPv4 移植為 IPv6

「可是我只想知道要怎麼改程式碼的哪些地方就可以支援 IPv6 了！快點告訴我！」

OK！OK！

我幾乎都是以過來人的身分來講這邊的每件事，這次不考驗各位的耐心，來個簡潔的短文。（當然有更多比這篇短的文章，不過我是跟自己的這份文件來比較）。

1. 首先，請試著用 getaddrinfo() 來取得 struct sockaddr 的資訊，取代手動填寫這個資料結構。這樣你就可以不用管 IP 的版本，而且能節省後續許多步驟。
2. 找出全部與 IP 版本相關的任何程式碼，試著用一個有用的函式將它們包起來（wrap up）。
3. 將 AF\_INET 更改為 AF\_INET6。
4. 將 PF\_INET 更改為 PF\_INET6。
5. 將 INADDR\_ANY 更改為 in6addr\_any，這裡有點不太一樣：

```c
struct sockaddr_in sa;
struct sockaddr_in6 sa6;

sa.sin_addr.s_addr = INADDR_ANY;   // 使用我的 IPv4 位址
sa6.sin6_addr = in6addr_any;  // 使用我的 IPv6 位址
```

還有，在宣告 struct in6\_addr 時，IN6ADDR\_ANY\_INIT 的值可以做為初始值，像這樣：

```c
struct in6_addr ia6 = IN6ADDR_ANY_INIT;
```

1. 使用 struct sockaddr\_in6 取代 struct sockaddr\_in，確定要將 "6" 新增到適當的欄位［參考上面的 structs］，但沒有 sin6\_zero 欄位。
2. 使用 struct in6\_addr 取代 struct in\_addr，要確定有將 "6" 新增到適當的欄位［參考上面的structs］。
3. 使用 inet\_pton() 取代 inet\_aton() 或 inet\_addr()。
4. 使用 inet\_ntop() 取代 inet\_ntoa()。
5. 使用很讚的 getaddrinfo() 取代 gethostbyname()。
6. 使用很讚的 getnameinfo() 取代 gethostbyaddr()［雖然 gethostbyaddr()在 IPv6 中也能正常運作］。
7. 不要用 INADDR\_BROADCAST 了，請愛用 IPv6 multicast 來替代。

就是這樣。

譯註： IPv6 可參考萩野純一郎, IPv6 網路程式設計, 博碩, 2004.
