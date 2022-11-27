# 6.3. Datagram Sockets

我們已經在討論 sendto() 與 recvfrom() 時涵蓋了 UDP datagram socket 的基礎，所以我會展示一對範例程式：talker.c 與 listener.c。

Listener 位於一台機器中，等待進入 port 4950 的封包。Talker 則從指定的機器傳送封包給這個 port，封包的內容包含使用者從命令列所輸入的資料。

這裡就是 listener.c 的原始程式碼 \[22]：

{% code lineNumbers="true" %}
```c
/*
** listener.c -- 一個 datagram sockets "server" 的 demo
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

#define MYPORT "4950" // 使用者所要連線的 port
#define MAXBUFLEN 100

// get sockaddr, IPv4 or IPv6:
void *get_in_addr(struct sockaddr *sa)
{
  if (sa->sa_family == AF_INET) {
    return &(((struct sockaddr_in*)sa)->sin_addr);
  }

  return &(((struct sockaddr_in6*)sa)->sin6_addr);
}

int main(void)
{
  int sockfd;
  struct addrinfo hints, *servinfo, *p;
  int rv;
  int numbytes;
  struct sockaddr_storage their_addr;
  char buf[MAXBUFLEN];
  socklen_t addr_len;
  char s[INET6_ADDRSTRLEN];

  memset(&hints, 0, sizeof hints);

  hints.ai_family = AF_UNSPEC; // 設定 AF_INET 以強制使用 IPv4
  hints.ai_socktype = SOCK_DGRAM;
  hints.ai_flags = AI_PASSIVE; // 使用我的 IP

  if ((rv = getaddrinfo(NULL, MYPORT, &hints, &servinfo)) != 0) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rv));
    return 1;
  }

  // 用迴圈來找出全部的結果，並 bind 到首先找到能 bind 的
  for(p = servinfo; p != NULL; p = p->ai_next) {

    if ((sockfd = socket(p->ai_family, p->ai_socktype,
      p->ai_protocol)) == -1) {
      perror("listener: socket");
      continue;
    }

    if (bind(sockfd, p->ai_addr, p->ai_addrlen) == -1) {
      close(sockfd);
      perror("listener: bind");
      continue;
    }

    break;
  }

  if (p == NULL) {
    fprintf(stderr, "listener: failed to bind socket\n");
    return 2;
  }

  freeaddrinfo(servinfo);
  printf("listener: waiting to recvfrom...\n");
  addr_len = sizeof their_addr;

  if ((numbytes = recvfrom(sockfd, buf, MAXBUFLEN-1 , 0, 
        (struct sockaddr *)&their_addr, &addr_len)) == -1) {

    perror("recvfrom");
    exit(1);
  }

  printf("listener: got packet from %s\n",

  inet_ntop(their_addr.ss_family,

  get_in_addr((struct sockaddr *)&their_addr), s, sizeof s));

  printf("listener: packet is %d bytes long\n", numbytes);

  buf[numbytes] = '\0';

  printf("listener: packet contains \"%s\"\n", buf);

  close(sockfd);

  return 0;
}
```
{% endcode %}

要注意的是，在我們呼叫 getaddrinfo()時，我們是使用 SOCK\_DGRAM。還要注意到，不需要 listen()或是 accept()，這是使用（免連線）datagram sockets 的一個好處！

接著是 talker.c 的程式碼 \[23]：

```c
/*
** talker.c -- 一個 datagram "client" 的 demo
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

#define SERVERPORT "4950" // 使用者所要連線的 port

int main(int argc, char *argv[])
{
  int sockfd;
  struct addrinfo hints, *servinfo, *p;
  int rv;
  int numbytes;

  if (argc != 3) {
    fprintf(stderr,"usage: talker hostname message\n");
    exit(1);
  }

  memset(&hints, 0, sizeof hints);
  hints.ai_family = AF_UNSPEC;
  hints.ai_socktype = SOCK_DGRAM;

  if ((rv = getaddrinfo(argv[1], SERVERPORT, &hints, &servinfo)) != 0) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rv));
    return 1;
  }

  // 用迴圈找出全部的結果，並產生一個 socket
  for(p = servinfo; p != NULL; p = p->ai_next) {
    if ((sockfd = socket(p->ai_family, p->ai_socktype,
      p->ai_protocol)) == -1) {
      perror("talker: socket");
      continue;
    }

    break;
  }

  if (p == NULL) {
    fprintf(stderr, "talker: failed to bind socket\n");
    return 2;
  }

  if ((numbytes = sendto(sockfd, argv[2], strlen(argv[2]), 0,
    p->ai_addr, p->ai_addrlen)) == -1) {

    perror("talker: sendto");
    exit(1);
  }

  freeaddrinfo(servinfo);
  printf("talker: sent %d bytes to %s\n", numbytes, argv[1]);
  close(sockfd);
  return 0;
}
```

全部就這些了！在某個機器上執行 listener，接著在另一台機器執行 talker。觀察它們的溝通！這真的超有趣。

這次你甚至不用執行 server！可以只執行 talker，而它只會很開心的將封包丟到網路上，如果另一端沒有人用 recvfrom()來接收的話，這些封包就只是消失而已。

要記得：使用 UDP datagram socket 傳送的資料是不會使命必達的！

我要再提之前提過無數次的小細節：connected datagram socket。我在這裡要再講一下，因為我們正在 datagram 這個章節。

我們說 talker 呼叫 connect()並指定 listener 的位址。從這開始，talker 就只能從 connect()所指定的位址進行傳送與接收。因此，你不用使用 sendto()與 recvfrom()，可以單純使用 send()與 recv() 就好。

\[22] [http://beej.us/guide/bgnet/examples/listener.c](http://beej.us/guide/bgnet/examples/listener.c)

\[23] [http://beej.us/guide/bgnet/examples/talker.c](http://beej.us/guide/bgnet/examples/talker.c)
