# 2017/07/01
## ディレクトリ作成
path(mac)
```html:path
/Users/yuma/github/BNNonFPGA
```  
## 環境構築(Mac)  
Dependencies  
>C++14 compiler (gcc 4.9+, clang 3.6+ or VS 2015+)  

### gcc  
gccっていうのはいろんなプログラミング言語のコンパイラパッケージのこと  
Macの標準gccはverが古いらしいのでuodateした  
最初からupdateしようとするとhomebrewをupdateする必要があるらしい  
homebrewのupdateにはchownする必要あり
```
$ sudo chown -R $(whoami) /usr/local
$ brew update
```
（10分くらい待つ）  
終わったらgcc4.9のinstall
```
$ brew tap homebrew/versions
$ brew install gcc49
```
終わったら前のgccと新しいgccのpathの置き換え  
```
$ sudo rm /usr/local/bin/gcc
$ sudo rm /usr/local/bin/g++
$ sudo ln -s /usr/local/bin/gcc-4.9 /usr/local/bin/gcc
$ sudo ln -s /usr/local/bin/g++-4.9 /usr/local/bin/g++
```
これで入った  
```
$ gcc --version
gcc (Homebrew GCC 4.9.4) 4.9.4
Copyright (C) 2015 Free Software Foundation, Inc.
This is free software; see the source for copying conditions.  There is NO
warranty; not even for MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.
```

### clang  
clangっていうのはC, C++, Objective-C向けのコンパイラフロントエンドのこと  
バックエンドのLLVMと組み合わさって、コンパイルを行う  
gccと双璧をなすコンパイラらしい（こっちのが早いとかなんとか）  

ver確認
```
$ clang --version
Apple LLVM version 8.1.0 (clang-802.0.42)
Target: x86_64-apple-darwin16.6.0
Thread model: posix
InstalledDir: /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin
```
どうやらver8.1.0が入っているようなのでOK

*準備おっけい！！*

## テストコードを動かしてみる
tiny-dnnのdevelopper曰く
>tiny-dnn is header-only, so there's nothing to build. If you want to execute sample program or unit tests, you need to install cmake and type the following commands:

チュートリアルに従ってやってみる  
```
$ cmake . -DBUILD_EXAMPLES=ON
$ make
```
10分くらいでmake完了  
Examplesに移動して再び`$ make`

### mnist
datasetを[ここ](http://yann.lecun.com/exdb/mnist/)からdlし`examples/mnist/`におく  
以下のコードで学習開始  
(mnistディレクトリ内のtrain.cppをさっきのmakeでbuildしたのを実行)
```
$ ./example_mnist_train --data_path mnist --learning_rate 1 --epochs 30 --minibatch_size 16 --backend_type internal
```
学習結果
```
accuracy:98.93% (9893/10000)
    *     0     1     2     3     4     5     6     7     8     9
    0   977     0     1     0     0     2     2     0     3     0
    1     0  1131     1     0     0     0     4     3     0     3
    2     0     1  1023     0     2     0     3     5     1     1
    3     0     1     1   999     0     2     0     2     2     0
    4     0     0     1     0   971     0     0     0     1     6
    5     1     0     0     3     0   885     2     0     1     5
    6     1     1     0     0     1     2   943     0     1     1
    7     1     1     3     5     1     0     0  1016     2     5
    8     0     0     2     2     1     1     4     0   961     1
    9     0     0     0     1     6     0     0     2     2   987
```
かかった時間は計測してないけど35m程度  
(1epochに70sくらいだったから)  

推論してみる  
テスト用画像を[ここ](https://raw.githubusercontent.com/wiki/tiny-dnn/tiny-dnn/4.bmp)からwgetして、
```
$ ./example_mnist_test 4.bmp
4,110.212
8,52.8517
7,50.9443
```
チュートリアルによるとこれは、4である確率が110.212%なんだけど、どうした  

## 次やること  
* C++でcodingできるようにmnistのsampleコード読む
* BNNのC++実装探してみる
* mnistをvivadoで高位合成できるか試してみる

# 2017/07/04  
こないだ動かしたmnistのLeNet5による学習コードを精読してみた  
詳しくは[ここ](https://github.com/yumfab-eeis/BNNonFPGA/blob/master/udst_train.md)  

# 2017/07/05  
こないだ動かしたmnistのLeNet5による学習コードを精読完了  

# 2017/07/26
動的Pruningについての検討をした
詳しくはここ 
