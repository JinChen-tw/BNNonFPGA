# 2017/07/01
## ディレクトリ作成
path(mac)
```html:path
/Users/yuma/github/BNNonFPGA
```  
## 環境構築  
Dependencies  
* C++14 compiler (gcc 4.9+, clang 3.6+ or VS 2015+)  

Macの標準gccはverが古いらしいのでuodateした  
最初からupdateしようとするとhomebrewをupdateする必要があるらしい
homebrewのupdateにはchownする必要あり
```
$ sudo chown -R $(whoami) /usr/local
$ brew update
```
（10分くらい待つ）
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
