# 7.6. 廣播封包：Hello World！

到了這裡，本文已經談了如何將資料從一台主機傳送到另一台主機。但是，我堅持你可能會需要絕對的權力，同時將資料送給多個主機！

用 UDP（只能用 UDP，TCP 不行）與標準 IPv4，可以透過一種叫作廣播（broadcasting）的機制達成。IPv6 不支援廣播，所以你必須要採用比較高級的技術－群播（multicasting），很遺憾地，我現在不會討論這個，我受夠了異想天開的未來，我們現在還停留在 32-bit 的 IPv4 世界呢！

可是，請等一下！不管你願不願意，你別離開呀，開始說說廣播吧。

你必須在將廣播封包送到網路之前，先設定 SO\_BROADCAST socket 選項。這類似一個推送導彈開關的小塑膠蓋！就只是你的手上掌握了多少的權力。

不過認真說來，使用廣播封包是很危險的，因為每個收到廣播封包的系統都要撥開一層層的資料封裝，直到系統知道這筆資料是要送給哪個 port 為止。然後系統會開始處理這筆資料或者丟掉它。在另一種情況，對每部收到廣播封包的機器而言這很費工，因為他們都在同一個區域網路（local network），這樣會讓很多電腦做不少多餘的工作。當 Doom 遊戲出現時，就有人在說它的網路程式寫的不好。

現在，有很多方法可以解決這個問題 ...

等一下，真的有很多方法嗎？

那是什麼表情阿？哎呀，一樣阿，送廣播封包的方法很多。所以重點就是：你該如何指定廣播訊息的目地位址呢？

有兩種常見的方法：

1. 將資料送給子網路（subnet）的廣播位址，就是將 subnet's network（子網路網段）的 host（主機）那部分全部填 1，舉例來說，我家裡的網路是 192.168.1.0，而我的 netmask（網路遮罩）是 255.255.255.0，所以位址的最後一個 byte 就是我的 host number（因為依據 netmask，前三個 bytes 是 network number）。所以我的廣播位址就是 192.168.1.255。在 Unix 底下，ifconfig指令實際上都會給你這些資料。（如果你有興趣，取得你廣播位址的邏輯運算方式是 network\_number OR (Not netmask) ）。你可以用跟區域網路一樣的方式，將這類型的廣播封包送到遠端網路（remote network），不過風險是封包可能會被目地端的 router（路由器）丟棄。（如果 router 沒有將封包丟棄，那麼有個隨機的藍色小精靈會開始用廣播流量對它們的區域網路造成水災。）
2. 將資料送給 ＂global（全域的）＂廣播位址，255.255.255.255，又稱為 INADDR\_BROADCAST，很多機器會自動將它與你的 network number 進行 AND 位元運算，以轉換為網路廣播位址，但是有些機器不會這樣做。Routers 不會將這類的廣播封包轉送（forward）出你的區域網路，夠諷刺的。

所以如果你想要將資料送到廣播位址，但是沒有設定 SO\_BROADCAST socket 選項時會怎樣呢？好，我們用之前的 talker 與listener 來炒冷飯，然後看看會發生什麼事情。

```shell
$ talker 192.168.1.2 foo
sent 3 bytes to 192.168.1.2
$ talker 192.168.1.255 foo
sendto: Permission denied
$ talker 255.255.255.255 foo
sendto: Permission denied
```

是的，沒有很順利 ... 因為我們沒有設定 SO\_BROADCAST socket 選項，設定它，然後現在你就可以用 sendto() 將資料送到你想送的地方了！

事實上，這就是 UDP 應用程式能不能廣播的差異點。所以我們改一下舊的 talker 應用程式，設定 SO\_BROADCAST socket 選項。這樣我們就能呼叫 broadcaster.c 程式了 \[36]：

\[36] [http://beej.us/guide/bgnet/examples/broadcaster.c](http://beej.us/guide/bgnet/examples/broadcaster.c)

```c
/*
** broadcaster.c -- 一個類似 talker.c 的 datagram "client"，
** 差異在於這個可以廣播
*/
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>

#define SERVERPORT 4950 // 所要連線的 port

int main(int argc, char *argv[])
{
  int sockfd;
  struct sockaddr_in their_addr; // 連線者的位址資訊
  struct hostent *he;
  int numbytes;
  int broadcast = 1;
  //char broadcast = '1'; // 如果上面這行不能用的話，改用這行

  if (argc != 3) {
    fprintf(stderr,"usage: broadcaster hostname message\n");
    exit(1);
  }

  if ((he=gethostbyname(argv[1])) == NULL) { // 取得 host 資訊
    perror("gethostbyname");
    exit(1);
  }

  if ((sockfd = socket(AF_INET, SOCK_DGRAM, 0)) == -1) {
    perror("socket");
    exit(1);
  }

  // 這個 call 就是要讓 sockfd 可以送廣播封包
  if (setsockopt(sockfd, SOL_SOCKET, SO_BROADCAST, &broadcast,
    sizeof broadcast) == -1) {
    perror("setsockopt (SO_BROADCAST)");
    exit(1);
  }

  their_addr.sin_family = AF_INET; // host byte order
  their_addr.sin_port = htons(SERVERPORT); // short, network byte order
  their_addr.sin_addr = *((struct in_addr *)he->h_addr);
  memset(their_addr.sin_zero, '\0', sizeof their_addr.sin_zero);

  if ((numbytes=sendto(sockfd, argv[2], strlen(argv[2]), 0,
          (struct sockaddr *)&their_addr, sizeof their_addr)) == -1) {
    perror("sendto");
    exit(1);
    }

  printf("sent %d bytes to %s\n", numbytes,
      inet_ntoa(their_addr.sin_addr));

  close(sockfd);

  return 0;
}
```

這個跟 ＂一般的＂ UDP client/server 有什麼不同呢？

沒有！［除了 client 可以送出廣播封包］

同樣地，我們繼續，並在其中一個視窗執行舊版的 UDP listener 程式，然後在另一個視窗執行 broadcaster，你應該可以順利執行了。

```shell
$ broadcaster 192.168.1.2 foo
sent 3 bytes to 192.168.1.2
$ broadcaster 192.168.1.255 foo
sent 3 bytes to 192.168.1.255
$ broadcaster 255.255.255.255 foo
sent 3 bytes to 255.255.255.255
```

而你應該會看到 listener 回應說它已經收到封包。（如果 listener 沒有回應，可能是因為它綁到 IPv6 位址了，試著將 listener.c 中的 AF\_UNSPEC 改成 AF\_INET，強制使用 IPv4）。

好，真令人興奮，可是現在要在同一個網路上的另一台電腦執行 listener，所以你會有兩個複本正在執行，每個機器上各有一個，然後再次用你的廣播位址來執行 broadcaster ... 嘿！你只有呼叫一次 sendto()，但是兩個 listeners 都收到了你的封包。酷喔！

如果 listener 收到你直接送給它的資料，但不是在廣播位址沒有資料，可能是因為你本機（local machine）上有防火牆（firewall）封鎖了這些封包。（是的，謝謝 Pat 與 Bapper 的說明，讓我知道為什麼我的範例程式無法運作。我跟你們說過我會在文件中提到你們，就是這裡了，感恩。）

再次提醒，使用廣播封包一定要小心，因為 LAN 上面的每台電腦都會被迫處理這類封包，無論它們有沒有用 recvfrom() 接收，這類封包會造成整個電腦網路相當大的負擔，所以一定要謹慎、適當地使用廣播。
