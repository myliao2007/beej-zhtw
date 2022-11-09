# 5.3. bind()－ 我在哪個 port？

一旦你有了一個 socket，你會想要將這個 socket 與你本機上的 port 進行關聯（如果你正想要 listen() 特定 port 進入的連線，通常都會這樣做，比如：多人網路連線遊戲在它們告訴你＂連線到 192.168.5.10 port 3490＂時這麼做）。port number 是用來讓 kernel 可以比對出進入的封包是屬於哪個 process 的 socket descriptor。如果你只是正在進行 connect()（因為你是 client，而不是 server），這可能就不用。不過還是可以讀讀，好玩嘛。

```c
#include <sys/types.h>
#include <sys/socket.h>

int bind(int sockfd, struct sockaddr *my_addr, int addrlen);
```

sockfd 是 socket() 傳回的 socket file descriptor。my\_addr 是指向包含你的位址資訊、名稱及 IP address 的 struct sockaddr 之指標。addrlen 是以 byte 為單位的位址長度。

呼！有點比較好玩了。我們來看一個範例，它將 socket bind（綁定）到執行程式的主機上，port 是 3490：

```c
struct addrinfo hints, *res;
int sockfd;

// 首先，用 getaddrinfo() 載入位址結構：

memset(&hints, 0, sizeof hints);
hints.ai_family = AF_UNSPEC; // use IPv4 or IPv6, whichever
hints.ai_socktype = SOCK_STREAM;
hints.ai_flags = AI_PASSIVE; // fill in my IP for me

getaddrinfo(NULL, "3490", &hints, &res);

// 建立一個 socket：

sockfd = socket(res->ai_family, res->ai_socktype, res->ai_protocol);

// 將 socket bind 到我們傳遞給 getaddrinfo() 的 port：

bind(sockfd, res->ai_addr, res->ai_addrlen);
```

使用 AI\_PASSIVE 旗標，我可以跟程式說要 bind 它所在主機的 IP。如果你想要 bind 到指定的本地 IP address，捨棄 AI\_PASSIVE，並改放一個位址到 getaddrinfo() 的第一個參數。

bind() 在錯誤時也會傳回 -1，並將 errno 設定為該錯誤的值。

許多舊程式都在呼叫 bind() 以前手動封裝 struct sockaddr\_in。很顯然地，這是 IPv4 才有的，可是真的沒有辦法阻止你在 IPv6 做一樣的事情，一般來說，使用 getaddrinfo() 會比較簡單。總之，舊版的程式看起來會像這樣：

```c
// !!! 這 是 老 方 法 !!!

int sockfd;
struct sockaddr_in my_addr;

sockfd = socket(PF_INET, SOCK_STREAM, 0);

my_addr.sin_family = AF_INET;
my_addr.sin_port = htons(MYPORT); // short, network byte order
my_addr.sin_addr.s_addr = inet_addr("10.12.110.57");
memset(my_addr.sin_zero, '\0', sizeof my_addr.sin_zero);

bind(sockfd, (struct sockaddr *)&my_addr, sizeof my_addr);
```

在上列的程式碼中，如果你想要 bind 到你本機的 IP address（就像上面的 AI\_PASSIVE 旗標），你也可以將 INADDR\_ANY 指定給 s\_addr 欄位。INADDR\_ANY 的 IPv6 版本是一個 in6addr\_any 全域變數，它會被指定給你的 struct sockaddr\_in6 的 sin6\_addr 欄位。

「也有一個你能用於變數初始器（variable initializer）的 IN6ADDR\_ANY\_INIT macro（巨集）」

另一件呼叫 bind() 時要小心的事情是：不要用太小的 port number。全部 1024 以下的 ports 都是保留的（除非你是系統管理員）！你可以使用任何 1024 以上的 port number，最高到 65535（提供尚未被其它程式使用的）。

你可能有注意到，有時候你試著重新執行 server，而 bind() 卻失敗了，它聲稱＂Address already in use.＂（位址使用中）。這是什麼意思呢？很好，有些連接到 socket 的連線還懸在 kernel 裡面，而它佔據了這個 port。你可以等待它自行清除（一分鐘之類），或者在你的程式中新增程式碼，讓它重新使用這個 port，類似這樣：

```c
int yes=1;
//char yes='1'; // Solaris 的使用者用這個

// 可以跳過 "Address already in use" 錯誤訊息

if (setsockopt(listener,SOL_SOCKET,SO_REUSEADDR,&yes,sizeof(int)) == -1) {
  perror("setsockopt");
  exit(1);
}
```

最後一個對 bind() 的額外小提醒：在你不願意呼叫 bind() 時。若你正使用 connect() 連線到遠端的機器，你可以不用管 local port 是多少（以 telnet 為例，你只管遠端的 port 就好），你可以單純地呼叫 connect()，它會檢查 socket 是否尚未綁定（unbound），並在有需要的時候自動將 socket bind() 到一個尚未使用的 local port。
