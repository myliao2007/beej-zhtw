# 3.3. 資料結構

很好，終於講到這裡了，該是談談程式設計的時間了。在本節，我會介紹 socket 介面的各種資料型別，因為它們有些會不太好理解。

首先是最簡單的：socket descriptor，型別如下：

```
int
```

就是一般的 int。

從這裡開始會有點不好理解，所以不用問太多，直接讀過就好。

我的第一個 StructTM －struct addrinfo，這個資料結構是最近的發明，用來準備之後要用的 socket 位址資料結構，也用在主機名稱（host name）及服務名稱（service name）的查詢。當我們之後開始實際應用時，才會開始覺得比較合理，現在只需要知道你在建立連線呼叫時會用到這個資料結構。

```c
struct addrinfo {
    int ai_flags; // AI_PASSIVE, AI_CANONNAME 等。
    int ai_family; // AF_INET, AF_INET6, AF_UNSPEC
    int ai_socktype; // SOCK_STREAM, SOCK_DGRAM
    int ai_protocol; // 用 0 當作 "any"
    size_t ai_addrlen; // ai_addr 的大小，單位是 byte
    struct sockaddr *ai_addr; // struct sockaddr_in 或 _in6
    char *ai_canonname; // 典型的 hostname
    struct addrinfo *ai_next; // 鏈結串列、下個節點
};
```

你可以載入這個資料結構，然後呼叫 getaddrinfo()。它會傳回一個指標，這個指標指向一個新的鏈結串列，這個串列有一些資料結構，而資料結構的內容記載了你所需的東西。

你可以在 ai\_family 欄位中設定強制使用 IPv4 或 IPv6，或者將它設定為 AF\_UNSPEC，AF\_UNSPEC 很酷，因為這樣你的程式就可以不用管 IP 的版本。

要注意的是，這是個鏈結串列：ai\_next 是指向下一個元素（element），可能會有多個結果讓你選擇。我會直接用它提供的第一個結果，不過你可能會有不同的個人考量；先生！我不是萬事通。

你會在 struct addrinfo 中看到 ai\_addr 欄位是一個指向 struct sockaddr 的指標。這是我們開始要了解 IP 位址結構中有哪些細節的地方。有時候，你需要的是呼叫 getaddrinfo() 幫你填好 struct addrinfo。然而，你必須查看這些資料結構，並將值取出，所以我在這邊會進行說明。

［還有，在發明 struct addrinfo 以前的程式碼都要手動填寫這些資料的每個欄位，所以你會看到很多 IPv4 的程式碼真的用很原始的方式去做這件事。你知道的，本文件在舊版也是這樣做］。

有些 structs 是 IPv4，而有些是 IPv6，有些兩者都是。我會特別註明清楚它們屬於哪一種。

總之，struct sockaddr 記錄了很多 sockets 類型的 socket 的位址資訊。

```c
struct sockaddr {
    unsigned short sa_family; // address family, AF_xxx
    char sa_data[14]; // 14 bytes of protocol address
};
```

sa\_family 可能是任何東西，不過在這份文件中我們會用到的是 AF\_INET［IPv4］或 AF\_INET6［IPv6］。sa\_data 包含一個 socket 的目地位址與 port number。這樣很不方便，因為你不會想要手動的將位址封裝到 sa\_data 裡。

為了處理 struct sockaddr，程式設計師建立了對等平行的資料結構：struct sockaddr\_in［"in" 是代表 "internet"］，可用在 IPv4。

而這邊有個重點：指向 struct sockaddr\_in 的指標可以轉型（cast）為指向 struct sockaddr 的指標，反之亦然。所以即使 connect() 需要一個 struct sockaddr \*，你也可以用 struct sockaddr\_in，並在最後的時候對它做型別轉換！

```c
// （IPv4 專用-- IPv6 請見 struct sockaddr_in6）
struct sockaddr_in {
    short int sin_family; // Address family, AF_INET
    unsigned short int sin_port; // Port number
    struct in_addr sin_addr; // Internet address
    unsigned char sin_zero[8]; // 與 struct sockaddr 相同的大小
};
```

這個資料結構讓它很容易可以參考（reference）socket 位址的成員。要注意的是 sin\_zero［這是用來將資料結構補足符合 struct sockaddr 的長度］，應該要使用 memset() 函式將 sin\_zero 整個清為零。還有，sin\_family 是對應到 struct sockaddr 中的 sa\_family，並應該設定為 "AF\_INET"。最後，sin\_port 必須是 Network Byte Order［利用 htons()］。

讓我們再更深入點！你可以在 struct in\_addr 裡看到 sin\_addr 欄位。

那是什麼？

好，別太激動，不過它是其中一個最恐怖的 union：

```c
// (僅限 IPv4 — Ipv6 請參考 struct in6_addr)
// Internet address (a structure for historical reasons)
struct in_addr {
    uint32_t s_addr; // that's a 32-bit int (4 bytes)
};
```

哇！好耶，它以前是 union，不過這個包袱現在似乎已經不見了。因此，若你已將 ina 宣告為 struct sockaddr\_in 的型別時，那麼 ina.sin\_addr.s\_addr 會參考到 4-byte 的 IP address（以 Network Byte Order）。要注意的是，如果你的系統仍然在 struct in\_addr 使用超恐怖的 union，你依然可以像我上面說的，精確地參考到 4-byte 的 IP address［這是因為有 #define］。

那麼 IPv6 會怎樣呢？

IPv6 也有提供類似的 struct，比如：

```c
// (IPv6 專用-- IPv4 請見 struct sockaddr_in 與 struct in_addr)
struct sockaddr_in6 {
    u_int16_t sin6_family; // address family, AF_INET6
    u_int16_t sin6_port; // port number, Network Byte Order
    u_int32_t sin6_flowinfo; // IPv6 flow 資訊
    struct in6_addr sin6_addr; // IPv6 address
    u_int32_t sin6_scope_id; // Scope ID
};

struct in6_addr {
    unsigned char s6_addr[16]; // IPv6 address
};
```

要注意到 IPv6 協定有一個 IPv6 address 與一個 port number，就像 IPv4 協定有一個 IPv4 address 與 port number 一樣。

我現在還不會介紹 IPv6 的流量資訊，或是 Scope ID 欄位 … 畢竟這只是一份入門文件嘛 :-)

最後要強調的一點，這個簡單的 struct sockaddr\_storage 設計用來足以儲存 IPv4 與 IPv6 structures 的 structure。 ［你看看，對於某些 calls，你有時無法事先知道它是否會使用 IPv4 或 IPv6 address 來填好你的 struct sockaddr。所以你用這個平行的 structure 來傳遞，它除了比較大以外，也很類似 struct sockaddr ，因而可以將它轉型為你所需的型別］。

```c
struct sockaddr_storage {
    sa_family_t ss_family; // address family
    // 這裡都是填充物（padding），依實作而定，請忽略它：
    char __ss_pad1[_SS_PAD1SIZE];
    int64_t __ss_align;
    char __ss_pad2[_SS_PAD2SIZE];
};
```

重點是你可以在 ss\_family 欄位看到位址家族（address family），檢查它是 AF\_INET 或 AF\_INET6（是 IPv4 或 IPv6）。之後如果你願意的話，你就可以將它轉型為 sockaddr\_in 或 struct sockaddr\_in6。
