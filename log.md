# 2017/07/01
## ディレクトリ作成
path(mac)
```html:path
/Users/yuma/github/BNNonFPGA
```  
## 環境構築(Mac)  
Dependencies  
>C++14 compiler (gcc 4.9+, clang 3.6+ or VS 2015+)  

#### gcc  
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

#### clang  
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

*tiny-dnnを使う準備おっけい！！*

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

#### mnist
datasetを[ここ](http://yann.lecun.com/exdb/mnist/)からdl  
`examples/mnist/`におく  
以下のコードで学習開始  
(mnistディレクトリ内のtrain.cppをさっきのmakeでbuildしたのを実行)
```
$ ./example_mnist_train --data_path mnist --learning_rate 1 --epochs 30 --minibatch_size 16 --backend_type internal
```
