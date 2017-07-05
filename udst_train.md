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
時間を計測  

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
```
現在のエポック、現在までのテスト結果を表示  
時間を再計測  

```cpp
  auto on_enumerate_minibatch = [&]() { disp += n_minibatch; };
```
バッチサイズの調整  
```cpp
  // training
  nn.train<mse>(optimizer, train_images, train_labels, n_minibatch,
                n_train_epochs, on_enumerate_minibatch, on_enumerate_epoch);
```
学習を行う  
```cpp
  std::cout << "end training." << std::endl;

  // test and show results
  nn.test(test_images, test_labels).print_detail(std::cout);
  // save network model & trained weights
  nn.save("LeNet-model");
}
```
学習が終わったらテストを行い結果を表示・モデルの保存  

## バックエンドの指定  
```cpp
static core::backend_t parse_backend_name(const std::string &name) {
  const std::array<const std::string, 5> names = {
    "internal", "nnpack", "libdnn", "avx", "opencl",
  };
  for (size_t i = 0; i < names.size(); ++i) {
    if (name.compare(names[i]) == 0) {
      return static_cast<core::backend_t>(i);
    }
  }
  return core::default_engine();
}
```
バックエンドの手法の指定をする関数らしい

## 引数の指定  
```cpp
static void usage(const char *argv0) {
  std::cout << "Usage: " << argv0 << " --data_path path_to_dataset_folder"
            << " --learning_rate 1"
            << " --epochs 30"
            << " --minibatch_size 16"
            << " --backend_type internal" << std::endl;
}
```
与える引数の指定を行う関数  

## メイン関数  
```cpp
int main(int argc, char **argv) {
  double learning_rate         = 1;
  int epochs                   = 30;
  std::string data_path        = "";
  int minibatch_size           = 16;
  core::backend_t backend_type = core::default_engine();

  if (argc == 2) {
    std::string argname(argv[1]);
    if (argname == "--help" || argname == "-h") {
      usage(argv[0]);
      return 0;
    }
  }
  for (int count = 1; count + 1 < argc; count += 2) {
    std::string argname(argv[count]);
    if (argname == "--learning_rate") {
      learning_rate = atof(argv[count + 1]);
    } else if (argname == "--epochs") {
      epochs = atoi(argv[count + 1]);
    } else if (argname == "--minibatch_size") {
      minibatch_size = atoi(argv[count + 1]);
    } else if (argname == "--backend_type") {
      backend_type = parse_backend_name(argv[count + 1]);
    } else if (argname == "--data_path") {
      data_path = std::string(argv[count + 1]);
    } else {
      std::cerr << "Invalid parameter specified - \"" << argname << "\""
                << std::endl;
      usage(argv[0]);
      return -1;
    }
  }
  if (data_path == "") {
    std::cerr << "Data path not specified." << std::endl;
    usage(argv[0]);
    return -1;
  }
  if (learning_rate <= 0) {
    std::cerr
      << "Invalid learning rate. The learning rate must be greater than 0."
      << std::endl;
    return -1;
  }
  if (epochs <= 0) {
    std::cerr << "Invalid number of epochs. The number of epochs must be "
                 "greater than 0."
              << std::endl;
    return -1;
  }
  if (minibatch_size <= 0 || minibatch_size > 60000) {
    std::cerr
      << "Invalid minibatch size. The minibatch size must be greater than 0"
         " and less than dataset size (60000)."
      << std::endl;
    return -1;
  }
  std::cout << "Running with the following parameters:" << std::endl
            << "Data path: " << data_path << std::endl
            << "Learning rate: " << learning_rate << std::endl
            << "Minibatch size: " << minibatch_size << std::endl
            << "Number of epochs: " << epochs << std::endl
            << "Backend type: " << backend_type << std::endl
            << std::endl;
  try {
    train_lenet(data_path, learning_rate, epochs, minibatch_size, backend_type);
  } catch (tiny_dnn::nn_error &err) {
    std::cerr << "Exception: " << err.what() << std::endl;
  }
  return 0;
}
```
