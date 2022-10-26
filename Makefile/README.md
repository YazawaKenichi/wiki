# Makefile 備忘録

## Makefile の基本構造
Makefile の基本構造は以下の通り。

```
処理名 : 依存するファイルや処理名
    実行されるコマンド
```

より上に書かれた処理が優先される。

例えば下記は、`sample1` を `sample1.cpp sample1.h` から作り出すことになっている。

その方法が `g++ -o sample1 sample1.cpp` ということ。

`Makefile`
```
sample1 : sample1.cpp sample1.h
    g++ -o sample1 sample1.cpp
sample2 : sample2.cpp sample2.h
    g++ -o sample2 sample2.cpp
```

## 文法
### 変数
変数を使用するときは `$()` でくくる。

世間一般的によく使用する変数名

SRCFILES / SOURCES / DEPENDS / OBJS / OBJECTS / HEADERS / INCLUDES / CC / CFLAGS / CPPFLAGS / LDFLAGS / LDLIBS etc..

`Makefile`
```
CC = g++
CFLAGS = -g -Wall

Sample1: sample1.cpp sample1.h
    $(CC) $(CFLAGS) -o sample1 sample1.cpp
Sample2: sample2.cpp sample2.h
    $(CC) $(CFLAGS) -o sample2 sample2.cpp
```

### ネスト
処理の依存先に他の処理を指定することで、依存元の処理をする前に、依存先の処理をする。

例えば下記のようにすると、`All` という処理を指定したときに、先に `Sample1` と `Sample2` を実行する。

`All` には処理内容が記載されていないため、`Sample1` と `Sample2` を実行して終了。

`Makefile`
```
CC = g++
CFLAGS = -g -Wall

All: Sample1 Sample2

Sample1: sample1.cpp sample1.h
    $(CC) $(CFLAGS) -o sample1 sample1.cpp
Sample2: sample2.cpp sample1.h
    $(CC) $(CFLAGS) -o sample2 sample2.cpp
```

### 連続処理
ネストの応用。

複数の処理を連続して実行することが可能。

処理の順番は `Main` -> `Sub` -> `$(CC) $(CFLAG) -o main.o -c main.cpp` の順。

`Makefile`
```
CC = g++
CFLAGS = -g -Wall

All: Main Sub
    $(CC) $(CFLAGS) -o main main.o sub.0

Main: main.cpp main.h
    $(CC) $(CFLAGS) -o main.o -c main.cpp

Sub: sub.cpp sub.h
    $(CC) $(CFLAGS) -o sub.o -c sub.cpp
```

### 特殊文字

|文字|説明
|:---:|:---
|$@|処理名
|$<|依存関係の一番左の名前
|$^|依存関係にある名前全て

`Makefile`
```
CC = g++
CFLAGS = -g -Wall

all: main.o sub.o
    $(CC) $(CFLAGS) -o main main.o sub.0

main.o: main.cpp main.h
    $(CC) $(CFLAGS) -o $@ -c $<

sub.o: sub.cpp sub.h
    $(CC) $(CFLAGS) -o $@ -c $<
```

### 処理の共通化

`.cpp.o` をどの処理名よりも早く記述することで、依存ファイルが `.cpp` で処理名が `.o` の処理内容を共通化させることが可能。

最初の `.cpp.o` で処理内容が共通化されたので、それより下で `main.o` や `sub.o` の処理内容を記述する必要が無いため省く。

*自分の環境ではうまく行かなかった*

`Makefile`
```
CC = g++
CFLAGS = -g -Wall

.cpp.o:
    $(CC) $(CFLAGS) -o $@ -c $<

all: main.o sub.o
    $(CC) $(CFLAGS) -o main main.o sub.0

main.o: main.cpp main.h
sub.o: sub.cpp sub.h
```

## g++ の使い方

そもそも論として、gcc, g++ の使い方がわかっていないと理解できない部分がある。

役に立ちそうなオプションをまとめておく。

|オプション|説明
|:---:|:---
|-o|出力ファイル名を指定
|-c|オブジェクトファイル .o を作成
|-g|デバッグ情報の生成
 
## [Makefile のテンプレート](https://github.com/YazawaKenichi/wiki/blob/main/Makefile/Makefile)

自分用に Makefile のテンプレートを作っておく。

前提として以下のようなファイル構造になっている。

```
Project
├── Makefile
├── bin
├── build
├── inc
│   ├── main.hpp
│   ├── sub1.h
│   └── sub2.h
└── src
    ├── main.cpp
    ├── sub1.cpp
    └── sub2.cpp
```

### 直接実行

ソースファイルから直接実行ファイルを作成するように記述しているため、わざわざオブジェクトファイルを経由する必要は無い。

` Makefile `
```
CC = g++
SRCS = ./src
INCS = ./inc
BINS = ./bin
BUILD = ./build
CFLAGS = -I$(INCS)

PROGRAM = program
LIBS = -lm
OBJS = 
CODES = $(SRCS)/main.cpp $(SRCS)/sub1.cpp $(SRCS)/sub2.cpp

$(PROGRAM): $(CODES)
    $(CC) $(CFLAGS) -o $(BUILD)/$@ $^

all: clean $(PROGRAM)

clean:
    rm -rf $(BUILD) $(BINS)
    mkdir $(BUILD) $(BINS)
```

### オブジェクトファイルから

[自分用に作ったテンプレート](https://github.com/YazawaKenichi/wiki/blob/main/Makefile/Makefile) はこっちの方法。

実用化してるだけあって扱いやすいように記述している。

#### Makefile
```
PROGRAM = Project

CC = g++
CONT = cpp
LIBS = -lm

MAIN = main
SUB1 = sub1
SUB2 = sub2

SRCS = ./src
INCS = ./inc
BINS = ./bin
BUILD = ./build

CFLAGS = -I$(INCS)

OBJS = $(BINS)/$(MAIN).o $(BINS)/$(SUB1).o $(BINS)/$(SUB2).o
CODES = $(SRCS)/$(MAIN).$(CONT) $(SRCS)/$(SUB1).$(CONT) $(SRCS)/$(SUB2).$(CONT)

# ↓こいつが機能してない
.cpp.o:
    $(CC) $(CFLAGS) -o $@ -c $<

$(PROGRAM): $(OBJS)
    $(CC) $(CFLAGS) -o $(BUILD)/$@ $^

all: clean $(PROGRAM)

clean:
    rm -rf $(BUILD) $(BINS)
    mkdir $(BUILD) $(BINS)

$(BINS)/$(MAIN).o: $(SRCS)/$(MAIN).$(CONT)
    $(CC) $(CFLAGS) -o $@ -c $<
$(BINS)/$(SUB1).o: $(SRCS)/$(SUB1).$(CONT)
    $(CC) $(CFLAGS) -o $@ -c $<
$(BINS)/$(SUB2).o: $(SRCS)/$(SUB2).$(CONT)
    $(CC) $(CFLAGS) -o $@ -c $<
```
#### 使い方
- `PROGRAM`, `MAIN`, `SUB*`, `LIBS`, `CC`, `CONT` は必要に応じて書き換える
    
    `SUB*` が増えたら...
    1. `OBJS` の末尾に ` $(BINS)/$(SUB*).o` を追記する
    1. `CODES` の末尾に `$(SRCS)/$(SUB*).$(CONT)` を追記する
    1. Makefile 末行に以下を追加する
    ```
    $(BINS)/$(SUB*).o: $(SRCS)/$(SUB*).$(CONT)
        $(CC) $(CFLAGS) -o $@ -c $<
    
    ```
    
## 有能まとめ記事
[特殊変数・自動変数 一覧](https://tex2e.github.io/blog/makefile/automatic-variables)

## ここに来るまでに参考にしたサイト
[Makefile の解説](http://omilab.naist.jp/~mukaigawa/misc/Makefile.html)
[Makefile の書き方（C 言語)](https://ie.u-ryukyu.ac.jp/~e085739/c.makefile.tuts.html)





