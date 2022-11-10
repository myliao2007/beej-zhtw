# 7.4. Serialization：如何封裝資料

你已經知道要將文字資料透過網路傳送很簡單，不過如果你想要送一些 「二進制」 的資料，如 int 或 float，會發生什麼事情呢？這裡有一些選擇。

1. 將數字轉換為文字，使用如 sprintf() 的函式，接著傳送文字。接收者會使用如 strtol() 函式解析文字，並轉換為數字。
2. 直接以原始資料傳送，將指向資料的指標傳遞給 send()。
3. 將數字編碼（encode）為可移植的二進制格式，接收者會將它解碼（decode）。

先睹為快！只在今晚！

［序幕］ Beej 說：＂我偏好上面的第三個方法！＂ ［結束］

（在我開始熱血介紹本章節之前，我應該要跟你說有現成的函式庫可以做這件事情，而要自製個可移植及無錯誤的作品會是相當大的挑戰。所以在決定要自己實作這部分時，可以先四處看看，並做完你的家庭作業。我在這裡引用些類似這個作品的有趣的資訊。）

實際上，上面全部的方法都有它們的缺點與優點，但是如我所述，通常我偏好第三個方法。首先，咱們先談談另外兩個的優缺點。

第一個方法，在傳送以前先將數字編碼為文字，優點是你可以很容易印出及讀取來自網路的資料。有時，人類易讀的協定比較適用於頻寬不敏感（non-bandwidth-intensive）的情況，例如：Internet Relay Chat（IRC）\[27]。然而，缺點是轉換耗時，且總是需要比原本的數字使用更多的空間。

第二個方法：傳送原始資料（raw data），這個方法相當簡單［但是危險！］：只要將資料指標提供給 send()。

```c
double d = 3490.15926535;

send(s, &d, sizeof d, 0); /* 危險，不具可移植性！ */
```

接收者類似這樣接收：

```c
double d;

recv(s, &d, sizeof d, 0); /* 危險，不具可移植性！ */
```

快速又簡單，那有什麼不好的呢？

好的，事實證明不是全部的架構都能表示 double（或 int）。（嘿！或許你不需要可移植性，在這樣的情況下這個方法很好，而且快速。）

當封裝整數型別時，我們已經知道 htons() 這類的函式如何透過將數字轉換為 Network Byte Order（網路位元組順序），來讓東西可以移植。可惜的是，沒有類似的函式可以供 float 型別使用。

全部的希望都落空了嗎？

別怕！（你有擔心了一會兒嗎？沒有嗎？一點都沒有嗎？）

我們可以做件事情：我們可以將資料封裝為接收者已知的二進位格式，讓接收著可以在遠端解壓縮。

我所謂的 「已知二進位格式」是什麼意思呢？

好的，我們已經看過了 htons() 範例了，不是嗎？它將數字從 host 格式改變（或是 ＂編碼＂）為 Network Byte Order 格式；如果要反轉「解碼」這個數字，接收端會呼叫 ntohs()。

可是我不是才剛說過，沒有這樣的函式可供非整數型別使用嗎？

是的，我說過。而且因為 C 語言並沒有規範標準的方式來做，所以這有點麻煩［that a gratuitous pun there for you Python fans］。

要做的事情是將資料封裝到已知的格式，並透過網路送出。例如：封裝 float，這裡的東西有很大的改善空間：\[28]

```c
#include <stdint.h>

uint32_t htonf(float f)
{
  uint32_t p;
  uint32_t sign;

  if (f < 0) { sign = 1; f = -f; }
  else { sign = 0; }

  p = ((((uint32_t)f)&0x7fff)<<16) | (sign<<31); // whole part and sign
  p |= (uint32_t)(((f - (int)f) * 65536.0f))&0xffff; // fraction

  return p;
}

float ntohf(uint32_t p)
{
  float f = ((p>>16)&0x7fff); // whole part
  f += (p&0xffff) / 65536.0f; // fraction

  if (((p>>31)&0x1) == 0x1) { f = -f; } // sign bit set

  return f;
}
```

上列的程式碼是一個 native（原生的）實作，將 float 儲存為 32-bit 的數字。High bit（高位元）（31）用來儲存數字的正負號（'1' 表示負數），而接下來的七個位元（30-16）是用來儲存 float 整個數字的部分。最後，剩下的位元（15-0）用來儲存數字的小數（fractional portion）部分。

使用方式相當直覺：

```c
#include <stdio.h>

int main(void)
{
  float f = 3.1415926, f2;
  uint32_t netf;

  netf = htonf(f); // 轉換為 "network" 形式
  f2 = ntohf(netf); // 轉回測試

  printf("Original: %f\n", f); // 3.141593
  printf(" Network: 0x%08X\n", netf); // 0x0003243F
  printf("Unpacked: %f\n", f2); // 3.141586

  return 0;
}
```

好處是：它很小、很簡單且快速，缺點是：它在空間的使用沒有效率，而且對範圍有嚴格的限制－試著在那邊儲存一個大於 32767 的數，它就會不爽！

你也可以在上面的例子看到，最後一對的十進位空間並沒有正確保存。

我們該怎麼改呢？

好的，用來儲存浮點數（float point number）的標準方式是已知的 IEEE-754 \[29]。多數的電腦會在內部使用這個格式做浮點運算，所以在這些例子裡，嚴格說來，不需要做轉換。但是如果你想要你的程式碼具可移植性，就要假設你不需要轉換。（換句話說，如果你想要讓程式很快，你應該要在不需要做轉換的平台上進行最佳化！這就是 htons() 與它的家族使用的方法。）

這邊有段程式碼可以將 float 與 double 編碼為 IEEE-754 格式 \[30]。（主要的功能，它不會編碼 NaN 或 Infinity，只要作點修改就可以了。）

```c
#define pack754_32(f) (pack754((f), 32, 8))
#define pack754_64(f) (pack754((f), 64, 11))
#define unpack754_32(i) (unpack754((i), 32, 8))
#define unpack754_64(i) (unpack754((i), 64, 11))

uint64_t pack754(long double f, unsigned bits, unsigned expbits)
{
  long double fnorm;
  int shift;
  long long sign, exp, significand;
  unsigned significandbits = bits - expbits - 1; // -1 for sign bit

  if (f == 0.0) return 0; // get this special case out of the way

  // 檢查正負號並開始正規化
  if (f < 0) { sign = 1; fnorm = -f; }
  else { sign = 0; fnorm = f; }

  // 取得 f 的正規化型式並追蹤指數
  shift = 0;
  while(fnorm >= 2.0) { fnorm /= 2.0; shift++; }
  while(fnorm < 1.0) { fnorm *= 2.0; shift--; }
  fnorm = fnorm - 1.0;

  // 計算有效位數資料的二進位格式（非浮點數）
  significand = fnorm * ((1LL<<significandbits) + 0.5f);

  // get the biased exponent
  exp = shift + ((1<<(expbits-1)) - 1); // shift + bias

  // 傳回最後的解答
  return (sign<<(bits-1)) | (exp<<(bits-expbits-1)) | significand;
}

long double unpack754(uint64_t i, unsigned bits, unsigned expbits)
{
  long double result;
  long long shift;
  unsigned bias;
  unsigned significandbits = bits - expbits - 1; // -1 for sign bit

  if (i == 0) return 0.0;

  // pull the significand

  result = (i&((1LL<<significandbits)-1)); // mask
  result /= (1LL<<significandbits); // convert back to float
  result += 1.0f; // add the one back on

  // deal with the exponent
  bias = (1<<(expbits-1)) - 1;
  shift = ((i>>significandbits)&((1LL<<expbits)-1)) - bias;
  while(shift > 0) { result *= 2.0; shift--; }
  while(shift < 0) { result /= 2.0; shift++; }

  // sign it
  result *= (i>>(bits-1))&1? -1.0: 1.0;

  return result;
}
```

我在那裡的頂端放一些方便的 macro（巨集），用來封裝與解封裝 32-bit（可能是 float）與 64-bit（可能是 double）的數字，但是 pack754() 函式可以直接呼叫，並告知編碼幾個位元的資料（expbits 的哪幾個位元要保留給正規化數值的指數。）

這裡是使用範例：

```c
#include <stdio.h>
#include <stdint.h> // 定義 uintN_t 型別
#include <inttypes.h> // 定義 PRIx macros

int main(void)
{
  float f = 3.1415926, f2;
  double d = 3.14159265358979323, d2;
  uint32_t fi;
  uint64_t di;

  fi = pack754_32(f);
  f2 = unpack754_32(fi);

  di = pack754_64(d);
  d2 = unpack754_64(di);

  printf("float before : %.7f\n", f);
  printf("float encoded: 0x%08" PRIx32 "\n", fi);
  printf("float after : %.7f\n\n", f2);

  printf("double before : %.20lf\n", d);
  printf("double encoded: 0x%016" PRIx64 "\n", di);
  printf("double after : %.20lf\n", d2);

  return 0;
}
```

上面的程式碼會產生下列的輸出：

```c
float before : 3.1415925
float encoded: 0x40490FDA
float after  : 3.1415925

double before : 3.14159265358979311600
double encoded: 0x400921FB54442D18
double after  : 3.14159265358979311600
```

你可能遭遇的另一個問題是你該如何封裝 struct 呢？

對你來說沒有問題的，編譯器會自動將一個 struct 中的全部空間填入。［你不會病到聽成 ＂不能這樣做＂、＂不能那樣做＂？抱歉！引述一個朋友的話：＂當事情出錯了，我都會怪給 Microsoft。＂這次固然可能不是 Microsoft 的錯，不過我朋友的陳述完全符合事實。］

回到這邊，透過網路送出 struct 的最好方式是將每個欄位獨立封裝，並接著在它們抵達另一端時，將它們解封裝到 struct。

你正在想，這樣要做很多事情。

是的，的確是。你能做的一件事情是寫個好用的函式來幫你封裝資料，這很好玩！真的！

在 Kernighan 與 Pike 著作的 "The Practice of Programming" \[31] 這本書，他們實作類似 printf() 的函式，名為 pack() 與 unpack()，可以完全做到這件事。我想要連結到這些函式，但是這些函式顯然地無法從網路上取得。

（The Practice of Programming 是值得閱讀的好書，Zeus saves a kitten every time I recommend it。）

此時，我正打算捨棄一個指標（pointer），它指向我從未用過的 BSD 授權類型參數語言 C API（BSD-licensed Typed Parameter Language C API）\[32]，可是這看起來整個很可敬。Python 與 Perl 程式設計師想找出他們語言裡的 pack() 與 unpack() 函式，用來完成同樣的事情。而 Java 有一個能用於相同用途的 big-ol' Serializable interface。

不過，如果你想要用 C 寫自己的封裝工具，K\&P 的技巧是使用變動參數列（variable argument list），用類似 printf() 的函式建立封包。我自己編寫的版本 \[33] 希望能足以幫助你瞭解這樣的東西是如何運作的。

「這段程式碼參考到上面的 pack754() 函式，packi\*() 函式的運作方式類似 htons() 家族，除非它們是封裝到一個 char 陣列（array）而不是另一個整數。」

```c
#include <ctype.h>
#include <stdarg.h>
#include <string.h>
#include <stdint.h>
#include <inttypes.h>

// 供浮點數型別的變動位元
// 隨著架構而變動

typedef float float32_t;
typedef double float64_t;

/*
** packi16() -- store a 16-bit int into a char buffer (like htons())
*/
void packi16(unsigned char *buf, unsigned int i)
{
  *buf++ = i>>8; *buf++ = i;
}

/*
** packi32() -- store a 32-bit int into a char buffer (like htonl())
*/
void packi32(unsigned char *buf, unsigned long i)
{
  *buf++ = i>>24; *buf++ = i>>16;
  *buf++ = i>>8; *buf++ = i;
}

/*
** unpacki16() -- unpack a 16-bit int from a char buffer (like ntohs())
*/
unsigned int unpacki16(unsigned char *buf)
{
  return (buf[0]<<8) | buf[1];
}

/*
** unpacki32() -- unpack a 32-bit int from a char buffer (like ntohl())
*/
unsigned long unpacki32(unsigned char *buf)
{
  return (buf[0]<<24) | (buf[1]<<16) | (buf[2]<<8) | buf[3];
}

/*
** pack() -- store data dictated by the format string in the buffer
**
** h - 16-bit l - 32-bit
** c - 8-bit char f - float, 32-bit
** s - string (16-bit length is automatically prepended)
*/
int32_t pack(unsigned char *buf, char *format, ...)
{
  va_list ap;
  int16_t h;
  int32_t l;
  int8_t c;
  float32_t f;
  char *s;
  int32_t size = 0, len;

  va_start(ap, format);

  for(; *format != '\0'; format++) {
    switch(*format) {
    case 'h': // 16-bit
      size += 2;
      h = (int16_t)va_arg(ap, int); // promoted
      packi16(buf, h);
      buf += 2;
      break;

    case 'l': // 32-bit
      size += 4;
      l = va_arg(ap, int32_t);
      packi32(buf, l);
      buf += 4;
      break;

    case 'c': // 8-bit
      size += 1;
      c = (int8_t)va_arg(ap, int); // promoted
      *buf++ = (c>>0)&0xff;
      break;

    case 'f': // float
      size += 4;
      f = (float32_t)va_arg(ap, double); // promoted
      l = pack754_32(f); // convert to IEEE 754
      packi32(buf, l);
      buf += 4;
      break;

    case 's': // string
      s = va_arg(ap, char*);
      len = strlen(s);
      size += len + 2;
      packi16(buf, len);
      buf += 2;
      memcpy(buf, s, len);
      buf += len;
      break;
    }
  }

  va_end(ap);

  return size;
}
/*
** unpack() -- unpack data dictated by the format string into the buffer
*/
void unpack(unsigned char *buf, char *format, ...)
{
  va_list ap;
  int16_t *h;
  int32_t *l;
  int32_t pf;
  int8_t *c;
  float32_t *f;
  char *s;
  int32_t len, count, maxstrlen=0;

  va_start(ap, format);

  for(; *format != '\0'; format++) {
    switch(*format) {
    case 'h': // 16-bit
      h = va_arg(ap, int16_t*);
      *h = unpacki16(buf);
      buf += 2;
      break;

    case 'l': // 32-bit
      l = va_arg(ap, int32_t*);
      *l = unpacki32(buf);
      buf += 4;
      break;

    case 'c': // 8-bit
      c = va_arg(ap, int8_t*);
      *c = *buf++;
      break;

    case 'f': // float
      f = va_arg(ap, float32_t*);
      pf = unpacki32(buf);
      buf += 4;
      *f = unpack754_32(pf);
      break;

    case 's': // string
      s = va_arg(ap, char*);
      len = unpacki16(buf);
      buf += 2;
      if (maxstrlen > 0 && len > maxstrlen) count = maxstrlen - 1;
      else count = len;
      memcpy(s, buf, count);
      s[count] = '\0';
      buf += len;
      break;

    default:
      if (isdigit(*format)) { // track max str len
        maxstrlen = maxstrlen * 10 + (*format-'0');
      }
    }

    if (!isdigit(*format)) maxstrlen = 0;
  }

  va_end(ap);
}
```

不管你是自己寫的程式，或者用別人的程式碼，基於持續檢查 bugs 的理由，有組通用的資料封裝機制集合是個好主意，而且不用每次都手動封裝每個 bit（位元）。

封裝資料時，使用哪種格式會比較好呢？

好問題，很幸運地，RFC 4506 \[35]，the External Data Representation Standard 已經定義了一堆各類型的二進位格式，如：浮點數型別、整數型別、陣列、原始資料等。如果你打算自己寫程式來封裝資料，我建議要符合標準，雖然不會強制你一定要遵守規範，但是封包規則不會剛好是你家定義的，至少，我不認為。

無論如何，在你送出資料以前，用某種方法將資料編碼是正確的做事方法。

\[27] [http://en.wikipedia.org/wiki/Internet\_Relay\_Chat](http://en.wikipedia.org/wiki/Internet\_Relay\_Chat)

\[28] [http://beej.us/guide/bgnet/examples/pack.c](http://beej.us/guide/bgnet/examples/pack.c)

\[29] [http://en.wikipedia.org/wiki/IEEE\_754](http://en.wikipedia.org/wiki/IEEE\_754)

\[30] [http://beej.us/guide/bgnet/examples/ieee754.c](http://beej.us/guide/bgnet/examples/ieee754.c)

\[31] [http://cm.bell-labs.com/cm/cs/tpop/](http://cm.bell-labs.com/cm/cs/tpop/)

\[32] [http://tpl.sourceforge.net/](http://tpl.sourceforge.net/)

\[33] [http://beej.us/guide/bgnet/examples/pack2.c](http://beej.us/guide/bgnet/examples/pack2.c)

\[34] [http://beej.us/guide/bgnet/examples/pack2.c](http://beej.us/guide/bgnet/examples/pack2.c)

\[35] [http://tools.ietf.org/html/rfc4506](http://tools.ietf.org/html/rfc4506)
