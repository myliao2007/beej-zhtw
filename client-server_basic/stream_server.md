# 6.1. 簡單的 Stream Server

這個 server 所做的事情就是透過 stream connection（串流連線）送出 "Hello, World!\n" 字串。你所需要做就是用一個視窗來測試執行 server，並用另一個視窗來 telnet 到 server：

```shell
$ telnet remotehostname 3490
```

這裡的 remotehostname 就是你執行 server 的主機名稱。

Server的程式碼如下 \[20]：

```c
/*
** server.c – 展示一個stream socket server
*/
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <errno.h>
#include <string.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <netdb.h>
#include <arpa/inet.h>
#include <sys/wait.h>
#include <signal.h>
#define PORT "3490" // 提供給使用者連線的 port
#define BACKLOG 10 // 有多少個特定的連線佇列（pending connections queue）

void sigchld_handler(int s)
{
  while(waitpid(-1, NULL, WNOHANG) > 0);
}

// 取得sockaddr，IPv4或IPv6：
void *get_in_addr(struct sockaddr *sa)
{
  if (sa->sa_family == AF_INET) {
    return &(((struct sockaddr_in*)sa)->sin_addr);
  }
  return &(((struct sockaddr_in6*)sa)->sin6_addr);
}

int main(void)
{
  int sockfd, new_fd; // 在 sock_fd 進行 listen，new_fd 是新的連線
  struct addrinfo hints, *servinfo, *p;
  struct sockaddr_storage their_addr; // 連線者的位址資訊 
  socklen_t sin_size;
  struct sigaction sa;
  int yes=1;
  char s[INET6_ADDRSTRLEN];
  int rv;

  memset(&hints, 0, sizeof hints);
  hints.ai_family = AF_UNSPEC;
  hints.ai_socktype = SOCK_STREAM;
  hints.ai_flags = AI_PASSIVE; // 使用我的 IP

  if ((rv = getaddrinfo(NULL, PORT, &hints, &servinfo)) != 0) {
    fprintf(stderr, "getaddrinfo: %s\n", gai_strerror(rv));
    return 1;
  }

  // 以迴圈找出全部的結果，並綁定（bind）到第一個能用的結果
  for(p = servinfo; p != NULL; p = p->ai_next) {
    if ((sockfd = socket(p->ai_family, p->ai_socktype,
      p->ai_protocol)) == -1) {
      perror("server: socket");
      continue;
    }

    if (setsockopt(sockfd, SOL_SOCKET, SO_REUSEADDR, &yes,
      sizeof(int)) == -1) {
      perror("setsockopt");
      exit(1);
    }

    if (bind(sockfd, p->ai_addr, p->ai_addrlen) == -1) {
      close(sockfd);
      perror("server: bind");
      continue;
    }

    break;
  }

  if (p == NULL) {
    fprintf(stderr, "server: failed to bind\n");
    return 2;
  }

  freeaddrinfo(servinfo); // 全部都用這個 structure

  if (listen(sockfd, BACKLOG) == -1) {
    perror("listen");
    exit(1);
  }

  sa.sa_handler = sigchld_handler; // 收拾全部死掉的 processes
  sigemptyset(&sa.sa_mask);
  sa.sa_flags = SA_RESTART;

  if (sigaction(SIGCHLD, &sa, NULL) == -1) {
    perror("sigaction");
    exit(1);
  }

  printf("server: waiting for connections...\n");

  while(1) { // 主要的 accept() 迴圈
    sin_size = sizeof their_addr;
    new_fd = accept(sockfd, (struct sockaddr *)&their_addr, &sin_size);
    if (new_fd == -1) {
      perror("accept");
      continue;
    }

    inet_ntop(their_addr.ss_family,
      get_in_addr((struct sockaddr *)&their_addr),
      s, sizeof s);
    printf("server: got connection from %s\n", s);

    if (!fork()) { // 這個是 child process
      close(sockfd); // child 不需要 listener

      if (send(new_fd, "Hello, world!", 13, 0) == -1)
        perror("send");

      close(new_fd);

      exit(0);
    }

    close(new_fd); // parent 不需要這個
  }

  return 0;
}
```

趁著你對這個例子還感到很好奇，我為了讓句子比較清楚（我個人覺得），所以將程式碼放在一個大的 main() 函式中，如果你覺得將它分成幾個小一點的函式會比較好的話，可以儘管去做。

「還有，sigaction() 這個東西對你而言應該是蠻陌生的。沒有關係，這個程式碼是用來清理 zombie process（殭屍行程），當 parent process 所 fork() 出來的 child process 結束時，且 parent process 沒有取得 child process 的離開狀態時，就會出現 zombie process。如果你產生了許多 zombie，但卻無法清除他們時，你的系統管理員就會開始焦慮不安了。」

你可以利用下一節所列出的 client，來取得 server 的資料。

\[20] [http://beej.us/guide/bgnet/examples/server.c](http://beej.us/guide/bgnet/examples/server.c)
