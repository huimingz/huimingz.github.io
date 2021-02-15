---
title: GolangでUTF-8の文字列を扱う
author: huimingz
date: 2021-02-10 18:50:06 +0800
category: [Golang]
tags: [Golang]
pin: false
---

転載元：[https://qiita.com/masakielastic/items/01a4fb691c572dd71a19](https://qiita.com/masakielastic/items/01a4fb691c572dd71a19)

## 文字列を表示する

```go
package main

func main() {
    str := "あいうえお"
    buf := []byte("あいうえお")
    runes := []rune("あいうえお")

    println(str)
    println(string(buf))
    println(string(runes))
}
```

## 文字列リテラルを使う
```go
package main

func main() {
    str := "か\u3099き\u3099く\u3099け\u3099こ\u3099"
    str2 := "DOG FACE \U0001F436"

    println(str)
    println(str2)
}
```

文字列リテラルを使う場合、次のとおり。

```go
package main

func main() {
    str := "\xe3\x81\x82\xe3\x81\x84\xe3\x81\x86\xe3\x81\x88\xe3\x81\x8a"
    println("あいうえお" == str)
}
```

## 数値から文字列を生成する

次のようにコードポイントをあらわす数値から rune スライスをつくることができる。

```go
package main

func main() {
    runes := []rune{0x3042, 0x3044, 0x3046, 0x3048, 0x304A}
    println(string(runes))
}
```

バイト列をあらわす数値から byte スライスを次のようにつくることができる。

```go
package main

func main() {
    buf := []byte{0xe3, 0x81, 0x82, 0xe3, 0x81, 0x84, 0xe3, 0x81, 0x86, 0xe3, 0x81, 0x88, 0xe3, 0x81, 0x8a}
    println(string(buf))
}
```

## Go の仕様

### string、byte、rune の相互変換ルール

Go の仕様ドキュメントの [Conversions to and from a string type](https://golang.org/ref/spec#Conversions_to_and_from_a_string_type) にまとめられています。

符号つきもしくは符号なしの整数値を string 型に変換すると、整数の UTF-8 表現を含む文字列が生成されます。有効な Unicode コードポイントの範囲の外にある値は "\\uFFFD" に変換されます。

```go
string('a')       // "a"
string(-1)        // "\ufffd" == "\xef\xbf\xbd"
string(0xf8)      // "\u00f8" == "ø" == "\xc3\xb8"
type MyString string
MyString(0x65e5)  // "\u65e5" == "日" == "\xe6\x97\xa5"
```

byte のスライスを string 型に変換すると、スライスの要素が連続したバイト列である文字列が生成されます。

```go
string([]byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'})   // "hellø"
string([]byte{})                                     // ""
string([]byte(nil))                                  // ""

type MyBytes []byte
string(MyBytes{'h', 'e', 'l', 'l', '\xc3', '\xb8'})  // "hellø"
```

rune のスライスを string 型に変換すると、それぞれの rune の値を文字に変換して連結した文字列が生成されます。

```go
string([]rune{0x767d, 0x9d6c, 0x7fd4})   // "\u767d\u9d6c\u7fd4" == "白鵬翔"
string([]rune{})                         // ""
string([]rune(nil))                      // ""

type MyRunes []rune
string(MyRunes{0x767d, 0x9d6c, 0x7fd4})  // "\u767d\u9d6c\u7fd4" == "白鵬翔"
```

string 型の値を byte のスライスに変換すると、連続した要素が文字列のバイトであるスライスが生成されます。

```go
[]byte("hellø")   // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}
[]byte("")        // []byte{}

MyBytes("hellø")  // []byte{'h', 'e', 'l', 'l', '\xc3', '\xb8'}
```

string 型の値を rune のスライスに変換すると、それぞれの文字の Unicode コードポイントを保有するスライスが生成されます。

```go
[]rune(MyString("白鵬翔"))  // []rune{0x767d, 0x9d6c, 0x7fd4}
[]rune("")                 // []rune{}

MyRunes("白鵬翔")           // []rune{0x767d, 0x9d6c, 0x7fd4}
```

## バイト数を求める

```go
package main

func main() {
    str := "あいうえお"
    buf := []byte("あいうえお")
    runes := []rune("あいうえお")

    println(15 == len(str))
    println(15 == len(buf))
    println(15 == len(string(runes))) 
}
```

## 文字数を求める

```go
package main

import "unicode/utf8"

func main() {
    str := "あいうえお"
    buf := []byte("あいうえお")
    runes := []rune("あいうえお")

    println(5 == utf8.RuneCountInString(str))
    println(5 == utf8.RuneCount(buf))
    println(5 == len(runes))
}
```
package  main  import  "unicode/utf8"  func  main()  {  str  :=  "あいうえお"  buf  :=  \[\]byte("あいうえお")  runes  :=  \[\]rune("あいうえお")  println(5  \==  utf8.RuneCountInString(str))  println(5  \==  utf8.RuneCount(buf))  println(5  \==  len(runes))  }  

## 1文字ずつアクセスする

```go 
package main

func main() {
    str := "あいうえお"

    for index, runeValue := range str {
        println("位置:", index, "文字:", string([]rune{runeValue}))
    }
}
```

rune 型のスライスの場合、次のようになる。

```go
package main

import (
    "fmt"
)

func main() {
    runes := []rune("あいうえお")

    for i := range runes {
        fmt.Printf("%c\n", runes[i])
    }
}
```

`utf8.DecodeRuneInString` を使う場合、次のように書ける。

```go
package main

import (
    "fmt"
    "unicode/utf8"
)

func main() {
    str := "あいうえお"

    for len(str) > 0 {
        r, size := utf8.DecodeRuneInString(str)
        fmt.Printf("%c\n", r)

        str = str[size:]
    }
}
```

## 不正なバイト列

### 不正なバイト列が含まれるかチェックする

```go
package main

import "unicode/utf8"

func main() {
    str := "あいうえお\x80"
    buf := []byte("あいうえお\x80")

    println(false == utf8.ValidString(str))
    println(false == utf8.Valid(buf))
}
```

### 不正なバイト列を代替文字の U+FFFD に置き換える

```go
package main

func main() {
    str := "あいうえお\x80"
    buf := []byte("")

    for _, runeValue := range str {
        buf = append(buf, string(runeValue)...)
    }

    println(string(buf))
}
```

## 東アジアの文字幅

### 文字幅の種類を調べる

文字幅の種類をあらわす定数として `EastAsianAmbiguous`、`EastAsianWide`、`EastAsianNarrow`、`EastAsianFullwidth`、`EastAsianHalfwidth` が定義される。

```go
package main

import (
    "golang.org/x/text/width"
)

func main() {
    s := "あいうえお"
    p, _ := width.LookupString(s)

    println(width.EastAsianWide == p.Kind())
}
```

### 全角文字と半角文字の相互変換

```go
package main

import (
    "golang.org/x/text/width"
    "fmt"
)

func main() {
    s := "ﾊﾝｶｸｶﾀｶﾅ"
    w := width.Widen.String(s)
    fmt.Printf("%U: %s\n", []rune(s), s)
    fmt.Printf("%U: %s\n", []rune(w), w)
}
```

```go
package main

import (
    "golang.org/x/text/width"
    "fmt"
)

func main() {
    s := "ゼンカクカタカナ"
    w := width.Narrow.String(s)
    fmt.Printf("%U: %s\n", []rune(s), s)
    fmt.Printf("%U: %s\n", []rune(w), w)
}
```


## Unicode 照合
```go
package main

import (
    "golang.org/x/text/collate"
    "golang.org/x/text/language"
)

func main() {
    c := collate.New(language.Japanese)

    println(0 == c.CompareString("a", "a"))
    println(-1 == c.CompareString("a", "A"))
    println(1 == c.CompareString("A", "a"))

    println(0 == c.CompareString("が", "か\u3099"))
}
```

## Unicode 正規化

```go
package main

import "golang.org/x/text/unicode/norm"

func main() {
    buf := norm.NFD.Bytes([]byte("がぎぐげご"))
    str := norm.NFD.String("がぎぐげご")
    expected := "か\u3099き\u3099く\u3099け\u3099こ\u3099"

    println(expected == string(buf))
    println(expected == str)
}
```

U+FDFA に NFKD を適用すると18文字になる。Unicode 正規化を適用して長くなる文字の例は 「[What are the maximum expansion factors for the different normalization forms?](http://www.unicode.org/faq/normalization.html#12)」(Unicode.org)の記事を参照。

```go
package main

import (
    "golang.org/x/text/unicode/norm"
    "unicode/utf8"
)

func main() {
    str := norm.NFKD.String("\uFDFA")

    println(18 == utf8.RuneCountInString(str))
}
```

基底文字の後に続く装飾記号の個数が31以上である場合、正規化は適用されない。

```go
package main

import (
    "strings"
    "golang.org/x/text/unicode/norm"
    "unicode/utf8"
)

func main() {
    str := "か" + strings.Repeat("\u3099", 30)
    ret := norm.NFC.String(str)

    println(30 == utf8.RuneCountInString(ret))

    str2 := "か" + strings.Repeat("\u3099", 32)
    ret2 := norm.NFC.String(str2)

    println(33 == utf8.RuneCountInString(ret2))
}
```