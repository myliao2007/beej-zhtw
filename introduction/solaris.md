# 1.4. Solaris/SunOS 程式設計師該注意的事

當編譯 Solaris 或 SunOS 平台的程式時，你需要指定一些額外的命令列參數，以連結（link）正確的函式庫（library）。為了達到這個目的，可以在編譯指令後面簡單加上 "-lnsl -lsocket -lresolv"，類似這樣：

```shell
$ cc -o server server.c -lnsl -lsocket -lresolv
```

如果還是有錯誤訊息，你可以再加上一個 "-lnext" 到命令列的尾端。我不太清楚這樣做了什麼事，不過有些人是會這樣用。

你可能會遇到的另一個問題是呼叫 setsockopt()。這個原型與在我 Linux 系統上的不一樣，所以可以這樣取代：

```c
int yes=1;
```

輸入這行：

```c
char yes='1';
```

因為我沒有 Sun 系統，所以我無法測試上面的資訊，這只是有人用 email 跟我說的。
