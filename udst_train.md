# tiny-dnnでのmnistのtrainコードの理解メモ  
c++でmachine leaningのcodingやったことないから精読してみました  
code全体は[こちら](https://github.com/yumfab-eeis/BNNonFPGA/blob/master/tiny-dnn/examples/mnist/train.cpp)

## 最初のおまじない
```c++
#include <iostream>
#include "tiny_dnn/tiny_dnn.h"
```

ヘッダファイルの読み込み  
iostreamは入出力のライブラリ  
tiny-dnnはヘッダとして読み込むだけで扱うことができる  

```cpp
using namespace tiny_dnn;  
using namespace tiny_dnn::activation;
```

tiny-dnnの名前空間を使用  

## モデルの構築
扱うNNのモデルの構築を行う  
```cpp
static void construct_net(network<sequential> &nn,
                          core::backend_t backend_type) {  
// connection table [Y.Lecun, 1998 Table.1]
#define O true
#define X false
  // clang-format off
    static const bool tbl[] = {
        O, X, X, X, O, O, O, X, X, O, O, O, O, X, O, O,
        O, O, X, X, X, O, O, O, X, X, O, O, O, O, X, O,
        O, O, O, X, X, X, O, O, O, X, X, O, X, O, O, O,
        X, O, O, O, X, X, O, O, O, O, X, X, O, X, O, O,
        X, X, O, O, O, X, X, O, O, O, O, X, O, O, X, O,
        X, X, X, O, O, O, X, X, O, O, O, O, X, O, O, O
    };
// clang-format on
#undef O
#undef X
```
static（静的）でローカルな関数であり、返り値はないのでvoid  
`static const bool tbl[]`はS2のFutureMapとC3のCnvLayerのconnection tableである  
3つのS2のFutureMapを横に並べたものとC3のCnvLayerとの結合を示す  
このtableを挟むことで構造がアシンメトリックになり、異なる特徴量が抽出できる（らしい）  
詳しくは論文のTable1を参照  

```c++
// construct nets
//
// C : convolution
// S : sub-sampling
// F : fully connected
// clang-format off
nn << convolutional_layer(32, 32, 5, 1, 6,   // C1, 1@32x32-in, 6@28x28-out
                          padding::valid, true, 1, 1, backend_type)
   << tanh_layer()
   << average_pooling_layer(28, 28, 6, 2)    // S2, 6@28x28-in, 6@14x14-out
   << tanh_layer()
   << convolutional_layer(14, 14, 5, 6, 16,  // C3, 6@14x14-in, 16@10x10-out
                          connection_table(tbl, 6, 16),
                          padding::valid, true, 1, 1, backend_type)
   << tanh_layer()
   << average_pooling_layer(10, 10, 16, 2)   // S4, 16@10x10-in, 16@5x5-out
   << tanh_layer()
   << convolutional_layer(5, 5, 5, 16, 120,  // C5, 16@5x5-in, 120@1x1-out
                          padding::valid, true, 1, 1, backend_type)
   << tanh_layer()
   << fully_connected_layer(120, 10, true,   // F6, 120-in, 10-out
                            backend_type)
   << tanh_layer();
// clang-format on
}
```
次に`nn << XXX`によってnnにレイヤーを追加していっている  
[LeNet5](http://yann.lecun.com/exdb/publis/pdf/lecun-98.pdf)という1998年のCNNの最初のモデルを実装している  
![LeNet5](https://cdn-ak.f.st-hatena.com/images/fotolife/n/ni4muraano/20170205/20170205221322.jpg)  
clang~~はフォーマットを整えるおまじない  

## 学習
```cpp
static void train_lenet(const std::string &data_dir_path,
                        double learning_rate,
                        const int n_train_epochs,
                        const int n_minibatch,
                        core::backend_t backend_type) {
```
関数の宣言、引数として、データのパス、学習率、エポック数、ミニバッチ、伝搬方法をとる

```cpp                          
  // specify loss-function and learning strategy
  network<sequential> nn;
  adagrad optimizer;

  construct_net(nn, backend_type);
```
使用するnnを宣言  
最適化手法としてadagradの使用を宣言  
nnを構築する関数に渡す  

```cpp  
  std::cout << "load models..." << std::endl;

  // load MNIST dataset
  std::vector<label_t> train_labels, test_labels;
  std::vector<vec_t> train_images, test_images;

  parse_mnist_labels(data_dir_path + "/train-labels.idx1-ubyte", &train_labels);
  parse_mnist_images(data_dir_path + "/train-images.idx3-ubyte", &train_images,
                     -1.0, 1.0, 2, 2);
  parse_mnist_labels(data_dir_path + "/t10k-labels.idx1-ubyte", &test_labels);
  parse_mnist_images(data_dir_path + "/t10k-images.idx3-ubyte", &test_images,
                     -1.0, 1.0, 2, 2);
```
学習・テストデータのローディング

```cpp
  std::cout << "start training" << std::endl;

  progress_display disp(train_images.size());
  timer t;
```
学習の進行具合を表示

```cpp
  optimizer.alpha *=
    std::min(tiny_dnn::float_t(4),
             static_cast<tiny_dnn::float_t>(sqrt(n_minibatch) * learning_rate));
```
学習率の最適化をしている  
詳しく知る必要がある場合はAdagradについて調べる

```cpp
  int epoch = 1;
  // create callback
  auto on_enumerate_epoch = [&]() {
    std::cout << "Epoch " << epoch << "/" << n_train_epochs << " finished. "
              << t.elapsed() << "s elapsed." << std::endl;
    ++epoch;
    tiny_dnn::result res = nn.test(test_images, test_labels);
    std::cout << res.num_success << "/" << res.num_total << std::endl;

    disp.restart(train_images.size());
    t.restart();
  };

  auto on_enumerate_minibatch = [&]() { disp += n_minibatch; };

  // training
  nn.train<mse>(optimizer, train_images, train_labels, n_minibatch,
                n_train_epochs, on_enumerate_minibatch, on_enumerate_epoch);
```

```cpp
  std::cout << "end training." << std::endl;

  // test and show results
  nn.test(test_images, test_labels).print_detail(std::cout);
  // save network model & trained weights
  nn.save("LeNet-model");
}
```
学習が終わったらテストを行い結果を表示・モデルの保存  
