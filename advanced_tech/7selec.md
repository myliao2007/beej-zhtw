# 7.2. select()：同步 I/O 多工

這個函式有點特別，不過它很好用。看看下面這個情況：如果你是一個 server，而你想要 listen 正在進來的連線，如同不斷讀取已建立的連線 socket 一樣。

你說：沒問題，只要用 accept() 及一對 recv() 就好了。

慢點，老兄！如果你在 accept() call 時發生了 blocking 該怎麼辦呢？你要如何同時進行 recv() 呢？

「那就使用 non-blocking socket！」

不行！你不會想成為浪費 CPU 資源的罪人吧。

嗯，那有什麼好方法嗎？

select() 授予你同時監視多個 sockets 的權力，它會告訴你哪些 sockets 已經有資料可以讀取、哪些 sockets 已經可以寫入，如果你真的想知道，還可以告訴你哪些 sockets 觸發了例外。

即使 select() 有相當好的可移植性，不過卻是最慢的監視 sockets 方法。一個比較可行的替代方案是 libevent \[24] 或者其它類似的方法，將全部的系統相依要素封裝起來，用在取得 socket 的通知。

好了，不囉唆，下面我提供了 select() 的原型：

```c
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>

int select(int numfds, fd_set *readfds, fd_set *writefds,
           fd_set *exceptfds, struct timeval *timeout);
```

這個函式以 readfds、writefds 及 exceptfds 分別表示要監視的那一組 file descriptor set（檔案描述符集合）。

如果你想要知道你是否能讀取 standard input（標準輸入）及某個 sockfd socket descriptor，只要將 file descriptor 0 與 sockfd 新增到 readfds set 中。numfds 參數應該要設定為 file descriptor 的最高值加 1。在這個例子中，應該要將 numfds 設定為 sockfd+1，因為它必定大於 standard input（0）。

當 select() 返回時，readfds 會被修改，用來反映你所設定的 file descriptors 中，哪些已經有資料可以讀取，你可以用下列的FD\_ISSET() macro（巨集）來取得這些就緒可讀的 file descriptors。

在繼續談下去以前，我想要說說該如何控制這些 sets。

每個 sets 的型別都是 fd\_set，下列是用來控制這個型別的 macro：

```c
FD_SET(int fd, fd_set *set);     將 fd 新增到 set。
FD_CLR(int fd, fd_set *set);     從 set 移除 fd。
FD_ISSET(int fd, fd_set *set);   若 fd 在 set 中，傳回 true。
FD_ZERO(fd_set *set);            將 set 整個清為零。
```

最後，這個令人困惑的 struct timeval 是什麼東西呢？

好，有時你不想要一直花時間在等人家送資料給你，或者明明沒什麼事，卻每 96 秒就要印出 ＂執行中 ...＂ 到終端機（terminal），而這個 time structure 讓你可以設定 timeout 的週期。

如果時間超過了，而 select() 還沒有找到任何就緒的 file descriptor 時，它就會返回，讓你可以繼續做其它事情。

```c
struct timeval 的欄位如下：
struct timeval {
  int tv_sec; // 秒（second）
  int tv_usec; // 微秒（microseconds）
};
```

只要將 tv\_sec 設定為要等待的秒數，並將 tv\_usec 設定為要等待的微秒數。是的，就是微秒，不是毫秒。一毫秒有 1,000 微秒，而一秒有 1,000 毫秒。所以，一秒就有 1,000,000 微秒。

為什麼要用 ＂usec（微秒）＂ 呢？

＂u＂看起來很像我們用來表示 ＂micro（微）＂的希臘字母 μ（Mu）。還有，當函式返回時，會更新 timeout，用以表示還剩下多少時間。這個行為取決於你所使用的 Unix 而定。

譯註：\
因為有些系統平台的 select() 會修改 timeout 的值，而有些系統不會，所以如果要重複呼叫 select() 的話，每次呼叫之前都應該要重新指定 timeout 的值，以確保程式的行為在各平台都可以有預期的行為。

哇！我們有微秒精度的計時器了！

是的，不過別依賴它。無論你將 struct timeval 設定的多小，你可能還要等待一小段的 standard Unix timeslice（標準 Unix 時間片段）。

另一件有趣的事：如果你將 struct timeval 的欄位設定為 0，select() 會在輪詢過 sets 中的每個 file descriptors 之後，就馬上 timeout。如果你將 timeout 參數設定為 NULL，它就永遠不會 timeout，並且陷入等待，直到至少一個 file descriptor 已經就緒（ready）。如果你不在乎等待時間，就在呼叫 select() 時將 timeout 參數設定為 NULL。

下列的程式碼片段 \[25] 等待 2.5 秒後，就會出現 standard input（標準輸入）所輸入的東西：

```c
/*
** select.c -- a select() demo
*/
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <unistd.h>
#define STDIN 0 // standard input 的 file descriptor
int main(void)
{
  struct timeval tv;
  fd_set readfds;

  tv.tv_sec = 2;
  tv.tv_usec = 500000;

  FD_ZERO(&readfds);
  FD_SET(STDIN, &readfds);

  // 不用管 writefds 與 exceptfds：
  select(STDIN+1, &readfds, NULL, NULL, &tv);

  if (FD_ISSET(STDIN, &readfds))
    printf("A key was pressed!\n");
  else
    printf("Timed out.\n");
  return 0;
}
```

如果你用一行緩衝區（buffer）的終端機，那麼你從鍵盤輸入資料後應該要盡快按下 Enter，否則程式就會發生 timeout。

你現在可能在想，這個方法用在需要等待資料的 datagram socket 上很棒，而且你是對的：應該是不錯的方法。

有些系統會用這個方式來使用 select()，而有些不行，如果你想要用它，你應該要參考你系統上的 man 使用手冊說明看是否會有問題。

有些系統會更新 struct timeval 的時間，用來反映 select() 原本還剩下多少時間 timeout；不過有些卻不會。如果你想要程式是可移植的，那就不要倚賴這個特性。（如果你需要追蹤剩下的時間，可以使用 gettimeofday()，我知道這很令人失望，不過事實就是這樣。）

如果在 read set 中的 socket 關閉連線，會怎樣嗎？

好的，這個例子的 select() 返回時，會在 socket descriptor set 中說明這個 socket 是 ＂ready to read（就緒可讀）＂的。而當你真的用 recv() 去讀取這個 socket 時，recv() 則會回傳 0 給你。這樣你就能知道是 client 關閉連線了。

再次強調 select() 有趣的地方：如果你正在 listen() 一個 socket，你可以將這個 socket 的 file descriptor 放在 readfds set 中，用來檢查是不是有新的連線。

朋友阿，這就是萬能 select() 函式的速成說明。

不過，應觀眾要求，這裡提供個有深度的範例，毫無疑問地，以前的簡單範例和這個範例的難易度會有顯著差距。不過你可以先看看，然後讀後面的解釋。

程式 \[26] 的行為是簡單的多使用者聊天室 server，在一個視窗中執行 server，然後在其它多個視窗使用 telnet 連線到 server（telnet hostname 9034）。當你在其中一個 telnet session 中輸入某些文字時，這些文字應該會在其它每個視窗上出現。

```c
/*
** selectserver.c -- 一個 cheezy 的多人聊天室 server
*/

#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <netdb.h>

#define PORT "9034" // 我們正在 listen 的 port

// 取得 sockaddr，IPv4 或 IPv6：
void *get_in_addr(struct sockaddr *sa)
{
  if (sa->sa_family == AF_INET) {
    return &(((struct sockaddr_in*)sa)->sin_addr);
  }

  return &(((struct sockaddr_in6*)sa)->sin6_addr);
}

int main(void)
{
  fd_set master; // master file descriptor 清單
  fd_set read_fds; // 給 select() 用的暫時 file descriptor 清單
  int fdmax; // 最大的 file descriptor 數目

  int listener; // listening socket descriptor
  int newfd; // 新接受的 accept() socket descriptor
  struct sockaddr_storage remoteaddr; // client address
  socklen_t addrlen;

  char buf[256]; // 儲存 client 資料的緩衝區
  int nbytes;

  char remoteIP[INET6_ADDRSTRLEN];

  int yes=1; // 供底下的 setsockopt() 設定 SO_REUSEADDR
  int i, j, rv;

  struct addrinfo hints, *ai, *p;

  FD_ZERO(&master); // 清除 master 與 temp sets
  FD_ZERO(&read_fds);

  // 給我們一個 socket，並且 bind 它
  memset(&hints, 0, sizeof hints);
  hints.ai_family = AF_UNSPEC;
  hints.ai_socktype = SOCK_STREAM;
  hints.ai_flags = AI_PASSIVE;

  if ((rv = getaddrinfo(NULL, PORT, &hints, &ai)) != 0) {
    fprintf(stderr, "selectserver: %s\n", gai_strerror(rv));
    exit(1);
  }

  for(p = ai; p != NULL; p = p->ai_next) {
    listener = socket(p->ai_family, p->ai_socktype, p->ai_protocol);
    if (listener < 0) {
      continue;
    }

    // 避開這個錯誤訊息："address already in use"
    setsockopt(listener, SOL_SOCKET, SO_REUSEADDR, &yes, sizeof(int));

    if (bind(listener, p->ai_addr, p->ai_addrlen) < 0) {
      close(listener);
      continue;
    }

    break;
  }

  // 若我們進入這個判斷式，則表示我們 bind() 失敗
  if (p == NULL) {
    fprintf(stderr, "selectserver: failed to bind\n");
    exit(2);
  }
  freeaddrinfo(ai); // all done with this

  // listen
  if (listen(listener, 10) == -1) {
    perror("listen");
    exit(3);
  }

  // 將 listener 新增到 master set
  FD_SET(listener, &master);

  // 持續追蹤最大的 file descriptor
  fdmax = listener; // 到此為止，就是它了

  // 主要迴圈
  for( ; ; ) {
    read_fds = master; // 複製 master

    if (select(fdmax+1, &read_fds, NULL, NULL, NULL) == -1) {
      perror("select");
      exit(4);
    }

    // 在現存的連線中尋找需要讀取的資料
    for(i = 0; i <= fdmax; i++) {
      if (FD_ISSET(i, &read_fds)) { // 我們找到一個！！
        if (i == listener) {
          // handle new connections
          addrlen = sizeof remoteaddr;
          newfd = accept(listener,
            (struct sockaddr *)&remoteaddr,
            &addrlen);

          if (newfd == -1) {
            perror("accept");
          } else {
            FD_SET(newfd, &master); // 新增到 master set
            if (newfd > fdmax) { // 持續追蹤最大的 fd
              fdmax = newfd;
            }
            printf("selectserver: new connection from %s on "
              "socket %d\n",
              inet_ntop(remoteaddr.ss_family,
                get_in_addr((struct sockaddr*)&remoteaddr),
                remoteIP, INET6_ADDRSTRLEN),
              newfd);
          }

        } else {
          // 處理來自 client 的資料
          if ((nbytes = recv(i, buf, sizeof buf, 0)) <= 0) {
            // got error or connection closed by client
            if (nbytes == 0) {
              // 關閉連線
              printf("selectserver: socket %d hung up\n", i);
            } else {
              perror("recv");
            }
            close(i); // bye!
            FD_CLR(i, &master); // 從 master set 中移除

          } else {
            // 我們從 client 收到一些資料
            for(j = 0; j <= fdmax; j++) {
              // 送給大家！
              if (FD_ISSET(j, &master)) {
                // 不用送給 listener 跟我們自己
                if (j != listener && j != i) {
                  if (send(j, buf, nbytes, 0) == -1) {
                    perror("send");
                  }
                }
              }
            }
          }
        } // END handle data from client
      } // END got new incoming connection
    } // END looping through file descriptors
  } // END for( ; ; )--and you thought it would never end!

  return 0;
}
```

我說過在程式碼中有兩個 file descriptor set：master 與 read\_fds。前面的 master 記錄全部現有連線的 socket descriptor，與正在 listen 新連線的 socket descriptor 一樣。

我用 master 的理由是因為 select() 實際上會改變你傳送過去的 set，用來反映目前就緒可讀（ready for read）的 socket。因為我必須在在兩次的 select() calls 期間也能夠持續追蹤連線，所以我必須將這些資料安全地儲存在某個地方。最後，我再將 master 複製到 read\_fds，並接著呼叫 select()。

可是這不就代表每當有新連線時，我就要將它新增到 master set 嗎？是的！

而每次連線結束時，我們也要將它從 master set 中移除嗎？是的，沒有錯。

我說過，我們要檢查 listen 的 socket 是否就緒可讀，如果可讀，這代表我有一個待處理的連線，而且我要 accept() 這個連線，並將它新增到 master set。同樣地，當 client 連線就緒可讀且 recv() 傳回 0 時，我們就能知道 client 關閉了連線，而我必須將這個 socket descriptor 從 master set 中移除。

若 client 的 recv() 傳回非零的值，因而，我能知道 client 已經收到了一些資料，所以我收下這些資料，並接著到 master 清單，並將資料送給其它已連線的每個 clients。

我的朋友們，以上是萬能 select() 函式的概述，這真是一件不簡單的事情。

另外，這有個福利：一個名為 poll 的函式，它的行為與 select() 很像，但是在管理 file descriptor set 時是用不一樣的系統，你可以看看 poll()。

參考資料

\[24] [http://www.monkey.org/\~provos/libevent/](http://www.monkey.org/\~provos/libevent/)

\[25] [http://beej.us/guide/bgnet/examples/select.c](http://beej.us/guide/bgnet/examples/select.c)

\[26] [http://beej.us/guide/bgnet/examples/selectserver.c](http://beej.us/guide/bgnet/examples/selectserver.c)

譯者註：在 [The Linux Programming Interface](http://man7.org/tlpi/) 的第 63 章有更深入的說明。
