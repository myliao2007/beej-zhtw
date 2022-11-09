# 5.1. getaddrinfo()－準備開始！

這是個有很多選項的工作馬（workhorse）函式，但是卻相當容易上手。它幫你設定之後需要的 struct。

談點歷史：它前身是你用來做 DNS 查詢的 gethostbyname()。而當時你需要手動將資訊載入 struct sockaddr\_in，並在你的呼叫中使用。

感謝老天，現在已經不用了。［如果你想要設計能通用於 IPv4 與 IPv6 的程式也不用！］在現代，你有 getaddrinfo() 函式，可以幫你做許多事情，包含 DNS 與 service name 查詢，並填好你所需的 structs。

讓我們來看看！

```c
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>

int getaddrinfo(const char *node, // 例如： "www.example.com" 或 IP
                const char *service, // 例如： "http" 或 port number
                const struct addrinfo *hints,
                struct addrinfo **res);
```

你給這個函式三個輸入參數，結果它會回傳給你一個指向鏈結串列的指標 － res。

node 參數是要連線的主機名稱，或者一個 IP address（位址）。

下一個參數是 service，這可以是 port number，像是 "80"，或者特定服務的名稱［可以在你 UNIX 系統上的 IANA Port List \[17] 或 /etc/services 檔案中找到］，像是 "http" 或 "ftp" 或 "telnet" 或 "smtp" 諸如此類的。

最後，hints 參數指向一個你已經填好相關資訊的 struct addrinfo。

這裡是一個呼叫範例，如果你是一部 server（伺服器），想要在你主機上的 IP address 及 port 3490 執行 listen。要注意的是，這邊實際上沒有做任何的 listening 或網路設定；它只有設定我們之後要用的 structures 而已。

```c
int status;
struct addrinfo hints;c
struct addrinfo *servinfo; // 將指向結果

memset(&hints, 0, sizeof hints); // 確保 struct 為空
hints.ai_family = AF_UNSPEC; // 不用管是 IPv4 或 IPv6
hints.ai_socktype = SOCK_STREAM; // TCP stream sockets
hints.ai_flags = AI_PASSIVE; // 幫我填好我的 IP 

if ((status = getaddrinfo(NULL, "3490", &hints, &servinfo)) != 0) {
  fprintf(stderr, "getaddrinfo error: %s\n", gai_strerror(status));
  exit(1);
}

// servinfo 目前指向一個或多個 struct addrinfos 的鏈結串列

// ... 做每件事情，一直到你不再需要 servinfo  ....

freeaddrinfo(servinfo); // 釋放這個鏈結串列
```

注意一下，我將 ai\_family 設定為 AF\_UNSPEC，這樣代表我不用管我們用的是 IPv4 或 IPv6 address。如果你想要指定的話，你可以將它設定為 AF\_INET 或 AF\_INET6。

還有，你會在這裡看到 AI\_PASSIVE 旗標；這個會告訴 getaddrinfo() 要將我本機的位址（address of local host）指定給 socket structure。這樣很棒，因為你就不用把位址寫死了［或者你可以將特定的位址放在 getaddrinfo() 的第一個參數中，我現在寫 NULL 的那個參數］。

然後我們執行呼叫，若有錯誤發生時［getaddrinfo 會傳回非零的值］，如你所見，我們可以使用 gai\_strerror() 函式將錯誤印出來。若每件事情都正常運作，那麼 serinfo 就會指向一個 struct addrinfos 的鏈結串列，串列中的每個成員都會包含一個我們之後會用到的某種 struct sockaddr。

最後，當我們終於使用 getaddrinfo() 配置的鏈結串列完成工作後，我們可以［也應該］要呼叫 freeaddrinfo() 將鏈結串列全部釋放。

這邊有一個呼叫範例，如果你是一個想要連線到特定 server 的 client（客戶端），比如是："www.example.net" 的 port 3490。再次強調，這裡並沒有真的進行連線，它只是設定我們之後要用的 structure。

```c
int status;
struct addrinfo hints;
struct addrinfo *servinfo; // 將指向結果

memset(&hints, 0, sizeof hints); // 確保 struct 為空
hints.ai_family = AF_UNSPEC; // 不用管是 IPv4 或 IPv6
hints.ai_socktype = SOCK_STREAM; // TCP stream sockets

// 準備好連線
status = getaddrinfo("www.example.net", "3490", &hints, &servinfo);

// servinfo 現在指向有一個或多個 struct addrinfos 的鏈結串列

我一直說 serinfo 是一個鏈結串列，它有各種的位址資訊。讓我們寫一個能快速 demo 的程式，來呈現這個資訊。這個小程式 [18] 會印出你在命令列中所指定的主機之 IP address：
/*
** showip.c -- 顯示命令列中所給的主機 IP address
*/
#include <stdio.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netdb.h>
#include <arpa/inet.h>
#include <netinet/in.h>

int main(int argc, char *argv[])
{
  struct addrinfo hints, *res, *p;
  int status;
  char ipstr[INET6_ADDRSTRLEN];

  if (argc != 2) {
    fprintf(stderr,"usage: showip hostname\n");
    return 1;
  }

  memset(&hints, 0, sizeof hints);
  hints.ai_family = AF_UNSPEC; // AF_INET 或 AF_INET6 可以指定版本
  hints.ai_socktype = SOCK_STREAM;

  if ((status = getaddrinfo(argv[1], NULL, &hints, &res)) != 0) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(status));
    return 2;
  }

  printf("IP addresses for %s:\n\n", argv[1]);

  for(p = res;p != NULL; p = p->ai_next) {
    void *addr;
    char *ipver;

    // 取得本身位址的指標，
    // 在 IPv4 與 IPv6 中的欄位不同：
    if (p->ai_family == AF_INET) { // IPv4
      struct sockaddr_in *ipv4 = (struct sockaddr_in *)p->ai_addr;
      addr = &(ipv4->sin_addr);
      ipver = "IPv4";
    } else { // IPv6
      struct sockaddr_in6 *ipv6 = (struct sockaddr_in6 *)p->ai_addr;
      addr = &(ipv6->sin6_addr);
      ipver = "IPv6";
    }

    // convert the IP to a string and print it:
    inet_ntop(p->ai_family, addr, ipstr, sizeof ipstr);
    printf(" %s: %s\n", ipver, ipstr);
  }

  freeaddrinfo(res); // 釋放鏈結串列

  return 0;
}
```

如你所見，程式碼使用你在命令列輸入的參數呼叫 getaddrinfo()，它填好 res 所指的鏈結串列，並接著我們就能重複那行並印出東西或做點類似的事。

［有點不好意思！我們在討論 struct sockaddrs 它的型別差異是因 IP 版本而異之處有點鄙俗。我不確定是否有較優雅的方法。］

在下面執行範例！來看看大家喜歡看的執行畫面：

```c
$ showip www.example.net
IP addresses for www.example.net:

  IPv4: 192.0.2.88

$ showip ipv6.example.com
IP addresses for ipv6.example.com:

  IPv4: 192.0.2.101
  IPv6: 2001:db8:8c00:22::171
```

現在已經在我們的掌控之下，我們會將 getaddrinfo() 傳回的結果送給其它的 socket 函式，而且終於可以建立我們的網路連線了！

讓我們繼續看下去！

\[17] [http://www.iana.org/assignments/port-numbers](http://www.iana.org/assignments/port-numbers)

\[18] [http://beej.us/guide/bgnet/examples/showip.c](http://beej.us/guide/bgnet/examples/showip.c)

\[19] [http://tools.ietf.org/html/rfc1413](http://tools.ietf.org/html/rfc1413)
