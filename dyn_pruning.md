# 動的なPruningによる学習コスト削減の検討についてメモ
## やりたいこと  
従来のPruning = 通常通り学習　→　いらない枝を剪定  
これを学習中にいらない枝のPruningをしたらよくならないか？  

## 重みがどのように更新されて言ってるかをみてみた  
tensorflowのtutorialのmnistコードを実行  
```
$ python mnist_with_summaries.py
```  
引数でdataファイルを渡すんだけどよくわかんないからコードに直で指定しちゃった  

結果  
```
Extracting /tmp/tensorflow/mnist/input_data/train-images-idx3-ubyte.gz
Extracting /tmp/tensorflow/mnist/input_data/train-labels-idx1-ubyte.gz
Extracting /tmp/tensorflow/mnist/input_data/t10k-images-idx3-ubyte.gz
Extracting /tmp/tensorflow/mnist/input_data/t10k-labels-idx1-ubyte.gz
2017-07-26 15:59:45.712644: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.1 instructions, but these are available on your machine and could speed up CPU computations.
2017-07-26 15:59:45.712672: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use SSE4.2 instructions, but these are available on your machine and could speed up CPU computations.
2017-07-26 15:59:45.712678: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX instructions, but these are available on your machine and could speed up CPU computations.
2017-07-26 15:59:45.712684: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use AVX2 instructions, but these are available on your machine and could speed up CPU computations.
2017-07-26 15:59:45.712689: W tensorflow/core/platform/cpu_feature_guard.cc:45] The TensorFlow library wasn't compiled to use FMA instructions, but these are available on your machine and could speed up CPU computations.
Accuracy at step 0: 0.1078
Accuracy at step 10: 0.7437
Accuracy at step 20: 0.8363
Accuracy at step 30: 0.87
Accuracy at step 40: 0.8861
Accuracy at step 50: 0.8936
Accuracy at step 60: 0.9007
Accuracy at step 70: 0.9062
Accuracy at step 80: 0.9057
Accuracy at step 90: 0.8965
Adding run metadata for 99
Accuracy at step 100: 0.9079
Accuracy at step 110: 0.9178
Accuracy at step 120: 0.9223
Accuracy at step 130: 0.9178
Accuracy at step 140: 0.9228
Accuracy at step 150: 0.9254
Accuracy at step 160: 0.926
Accuracy at step 170: 0.9263
Accuracy at step 180: 0.9328
Accuracy at step 190: 0.9353
Adding run metadata for 199
Accuracy at step 200: 0.9323
Accuracy at step 210: 0.9269
Accuracy at step 220: 0.9353
Accuracy at step 230: 0.9379
Accuracy at step 240: 0.9378
Accuracy at step 250: 0.9388
Accuracy at step 260: 0.937
Accuracy at step 270: 0.9384
Accuracy at step 280: 0.9413
Accuracy at step 290: 0.9424
Adding run metadata for 299
Accuracy at step 300: 0.9389
Accuracy at step 310: 0.9439
Accuracy at step 320: 0.9426
Accuracy at step 330: 0.9434
Accuracy at step 340: 0.9463
Accuracy at step 350: 0.9463
Accuracy at step 360: 0.9472
Accuracy at step 370: 0.9461
Accuracy at step 380: 0.9486
Accuracy at step 390: 0.9497
Adding run metadata for 399
Accuracy at step 400: 0.9468
Accuracy at step 410: 0.9528
Accuracy at step 420: 0.9531
Accuracy at step 430: 0.9544
Accuracy at step 440: 0.9545
Accuracy at step 450: 0.9533
Accuracy at step 460: 0.9559
Accuracy at step 470: 0.9527
Accuracy at step 480: 0.9544
Accuracy at step 490: 0.9551
Adding run metadata for 499
Accuracy at step 500: 0.9558
Accuracy at step 510: 0.9565
Accuracy at step 520: 0.9567
Accuracy at step 530: 0.956
Accuracy at step 540: 0.9572
Accuracy at step 550: 0.9596
Accuracy at step 560: 0.9575
Accuracy at step 570: 0.9591
Accuracy at step 580: 0.9556
Accuracy at step 590: 0.9576
Adding run metadata for 599
Accuracy at step 600: 0.9594
Accuracy at step 610: 0.9596
Accuracy at step 620: 0.9613
Accuracy at step 630: 0.9592
Accuracy at step 640: 0.9615
Accuracy at step 650: 0.9614
Accuracy at step 660: 0.9604
Accuracy at step 670: 0.9625
Accuracy at step 680: 0.9633
Accuracy at step 690: 0.9629
Adding run metadata for 699
Accuracy at step 700: 0.9626
Accuracy at step 710: 0.9602
Accuracy at step 720: 0.9602
Accuracy at step 730: 0.9624
Accuracy at step 740: 0.9618
Accuracy at step 750: 0.9626
Accuracy at step 760: 0.9649
Accuracy at step 770: 0.9638
Accuracy at step 780: 0.9628
Accuracy at step 790: 0.964
Adding run metadata for 799
Accuracy at step 800: 0.965
Accuracy at step 810: 0.9651
Accuracy at step 820: 0.9644
Accuracy at step 830: 0.9637
Accuracy at step 840: 0.9632
Accuracy at step 850: 0.9665
Accuracy at step 860: 0.9676
Accuracy at step 870: 0.9668
Accuracy at step 880: 0.9658
Accuracy at step 890: 0.9663
Adding run metadata for 899
Accuracy at step 900: 0.9665
Accuracy at step 910: 0.9675
Accuracy at step 920: 0.966
Accuracy at step 930: 0.9684
Accuracy at step 940: 0.9698
Accuracy at step 950: 0.9657
Accuracy at step 960: 0.9684
Accuracy at step 970: 0.9689
Accuracy at step 980: 0.9696
Accuracy at step 990: 0.9673
Adding run metadata for 999
```

tensorboardを使って結果を可視化する  
```
➜  mnist git:(master) ✗ tensorboard --logdir=/Users/yuma/github/BNNonFPGA/dyn_pruning/code/data/train             
Starting TensorBoard 47 at http://0.0.0.0:6006
(Press CTRL+C to quit)
```
こんな感じでローカルに立ち上がるので確認すると、
![result](https://github.com/yumfab-eeis/BNNonFPGA/blob/master/dyn_pruning/img/mnist_train.png?raw=true)  
こんな感じ  
バイアスは変化が大きいけど、重みは線形変化しているきがするのでこれは途中で剪定できるのでは...  
