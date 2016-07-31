# Fork

Adds a `compress` command that roughly implements the recent NIPS 2015 paper _Learning both Weights and Connections for Efficient Neural Networks_ by Han et al.

**In development.**

## Usage

`build/tools/caffe compress -solver <path to solver prototxt file> -weights <path to trained model> -layertype [InnerProduct | Convolution]`

#### Flags
Supports either `-layertype` (for all layers of given type) or `-layername` (for a specific layer). Exactly one of these options must be provided.

__Note:__ Convolution is not yet implemented.


## Example

First follow the [MNIST tutorial](http://caffe.berkeleyvision.org/gathered/examples/mnist.html).

Create a `lenet_compressor.prototxt` file as in `examples/mnist/` based off of `lenet_solver.prototxt`:
```
# The train/test net protocol buffer definition
net: "examples/mnist/lenet_train_test.prototxt"
# test_iter specifies how many forward passes the test should carry out.
# In the case of MNIST, we have test batch size 100 and 100 test iterations,
# covering the full 10,000 testing images.
test_iter: 100
# Carry out testing every 500 training iterations.
test_interval: 500
# The base learning rate, momentum and the weight decay of the network.
base_lr: 0.001 # retrain with lr 1/10 of the original
momentum: 0.9
weight_decay: 0.0005
# The learning rate policy
lr_policy: "inv"
gamma: 0.0001
power: 0.75
# Display every 100 iterations
display: 100
# The maximum number of iterations
max_iter: 1000
# snapshot intermediate results
snapshot: 5000
snapshot_prefix: "examples/mnist/lenet_compressed"
# solver mode: CPU or GPU
solver_mode: CPU
prune_threshold: 2
```

Note that `base_lr` is reduced by a factor of 10 and there is an additional `prune_threshold` parameter arbitrarily set to 2.

Run the compress command:

`build/tools/caffe compress -solver examples/mnist/lenet_compressor.prototxt -weights examples/mnist/lenet_iter_10000.caffemodel -layertype InnerProduct`

Output:
```
...
I0726 23:59:53.355252 1934443264 net.cpp:283] Network initialization done.
I0726 23:59:53.355311 1934443264 solver.cpp:60] Solver scaffolding done.
I0726 23:59:53.355366 1934443264 caffe.cpp:161] Finetuning from examples/mnist/lenet_iter_10000.caffemodel
I0726 23:59:53.362226 1934443264 caffe.cpp:320] Starting Pruning
I0726 23:59:53.362262 1934443264 solver.cpp:396] Pruning LeNet
I0726 23:59:53.362272 1934443264 solver.cpp:397] Threshold: 2
I0726 23:59:53.362283 1934443264 solver.cpp:534] Iteration 0, Testing net (#0)
I0726 23:59:56.325230 1934443264 solver.cpp:604]     Test net output #0: accuracy = 0.9905
I0726 23:59:56.325273 1934443264 solver.cpp:604]     Test net output #1: loss = 0.0305422 (* 1 = 0.0305422 loss)
I0726 23:59:56.325284 1934443264 solver.cpp:404] (Layer type)Data
I0726 23:59:56.325295 1934443264 solver.cpp:404] (Layer type)Convolution
I0726 23:59:56.325301 1934443264 solver.cpp:404] (Layer type)Pooling
I0726 23:59:56.325305 1934443264 solver.cpp:404] (Layer type)Convolution
I0726 23:59:56.325310 1934443264 solver.cpp:404] (Layer type)Pooling
I0726 23:59:56.325314 1934443264 solver.cpp:404] (Layer type)InnerProduct
I0726 23:59:56.325319 1934443264 solver.cpp:406] Pruning layer ip1
I0726 23:59:56.325323 1934443264 inner_product_layer.cpp:104] Calling Blob::Prune_cpu()
I0726 23:59:56.326490 1934443264 blob.cpp:67] Mask added
I0726 23:59:56.329623 1934443264 blob.cpp:277] ================ Computed Threshold is: 0.0513858, std is 0.0256929, threshold_param is 2
I0726 23:59:56.332329 1934443264 blob.cpp:300] ================ Pruned 98.7188% of connections away.
I0726 23:59:56.332376 1934443264 blob.cpp:301] ================ 5125 out of 400000 connections remaining.
I0726 23:59:56.332388 1934443264 solver.cpp:404] (Layer type)ReLU
I0726 23:59:56.332453 1934443264 solver.cpp:404] (Layer type)InnerProduct
I0726 23:59:56.332463 1934443264 solver.cpp:406] Pruning layer ip2
I0726 23:59:56.332473 1934443264 inner_product_layer.cpp:104] Calling Blob::Prune_cpu()
I0726 23:59:56.332489 1934443264 blob.cpp:67] Mask added
I0726 23:59:56.332510 1934443264 blob.cpp:277] ================ Computed Threshold is: 0.14272, std is 0.0713602, threshold_param is 2
I0726 23:59:56.332552 1934443264 blob.cpp:300] ================ Pruned 95.44% of connections away.
I0726 23:59:56.332559 1934443264 blob.cpp:301] ================ 228 out of 5000 connections remaining.
```

(Note that the statistics given are for the two inner product layers and not the entire net.)

The pruned net is then tested as is:

```
I0726 23:59:56.332564 1934443264 solver.cpp:404] (Layer type)SoftmaxWithLoss
I0726 23:59:56.332569 1934443264 solver.cpp:534] Iteration 0, Testing net (#0)
I0726 23:59:59.371148 1934443264 solver.cpp:604]     Test net output #0: accuracy = 0.6092
I0726 23:59:59.371181 1934443264 solver.cpp:604]     Test net output #1: loss = 1.41925 (* 1 = 1.41925 loss)
I0726 23:59:59.371191 1934443264 caffe.cpp:325] Pruning Done.
```

The network is then retrained:

```
I0726 23:59:59.371197 1934443264 caffe.cpp:326] Starting Optimization
I0726 23:59:59.371202 1934443264 solver.cpp:475] Solving LeNet
I0726 23:59:59.371206 1934443264 solver.cpp:476] Learning Rate Policy: inv
I0726 23:59:59.373217 1934443264 solver.cpp:534] Iteration 0, Testing net (#0)
I0727 00:00:02.515833 1934443264 solver.cpp:604]     Test net output #0: accuracy = 0.6092
I0727 00:00:02.515866 1934443264 solver.cpp:604]     Test net output #1: loss = 1.41925 (* 1 = 1.41925 loss)
I0727 00:00:02.566174 1934443264 solver.cpp:228] Iteration 0, loss = 1.30342
I0727 00:00:02.566236 1934443264 solver.cpp:244]     Train net output #0: loss = 1.30342 (* 1 = 1.30342 loss)
I0727 00:00:02.566260 1934443264 sgd_solver.cpp:106] Iteration 0, lr = 0.001
I0727 00:00:08.500051 1934443264 solver.cpp:228] Iteration 100, loss = 1.16216
I0727 00:00:08.500092 1934443264 solver.cpp:244]     Train net output #0: loss = 1.16216 (* 1 = 1.16216 loss)
I0727 00:00:08.500100 1934443264 sgd_solver.cpp:106] Iteration 100, lr = 0.000992565
...
I0727 00:00:26.892253 1934443264 solver.cpp:228] Iteration 400, loss = 0.690224
I0727 00:00:26.893460 1934443264 solver.cpp:244]     Train net output #0: loss = 0.690224 (* 1 = 0.690224 loss)
I0727 00:00:26.893476 1934443264 sgd_solver.cpp:106] Iteration 400, lr = 0.000971013
I0727 00:00:33.199570 1934443264 solver.cpp:534] Iteration 500, Testing net (#0)
I0727 00:00:36.463996 1934443264 solver.cpp:604]     Test net output #0: accuracy = 0.9163
I0727 00:00:36.464031 1934443264 solver.cpp:604]     Test net output #1: loss = 0.69147 (* 1 = 0.69147 loss)
I0727 00:00:36.573400 1934443264 solver.cpp:228] Iteration 500, loss = 0.784551
I0727 00:00:36.573436 1934443264 solver.cpp:244]     Train net output #0: loss = 0.784551 (* 1 = 0.784551 loss)
...
I0727 00:01:03.721237 1934443264 solver.cpp:244]     Train net output #0: loss = 0.515065 (* 1 = 0.515065 loss)
I0727 00:01:03.721254 1934443264 sgd_solver.cpp:106] Iteration 900, lr = 0.000937411
I0727 00:01:10.762639 1934443264 solver.cpp:658] Snapshotting to binary proto file examples/mnist/lenet_compressed_iter_1000.caffemodel
I0727 00:01:10.798032 1934443264 sgd_solver.cpp:273] Snapshotting solver state to binary proto file examples/mnist/lenet_compressed_iter_1000.solverstate
I0727 00:01:10.850066 1934443264 solver.cpp:514] Iteration 1000, loss = 0.601795
I0727 00:01:10.850101 1934443264 solver.cpp:534] Iteration 1000, Testing net (#0)
I0727 00:01:14.768610 1934443264 solver.cpp:604]     Test net output #0: accuracy = 0.9345
I0727 00:01:14.768648 1934443264 solver.cpp:604]     Test net output #1: loss = 0.446723 (* 1 = 0.446723 loss)
I0727 00:01:14.768661 1934443264 solver.cpp:519] Optimization Done.
```

To check that pruning has worked:

`build/tools/caffe test -model examples/mnist/lenet_train_test.prototxt -weights examples/mnist/lenet_compressed_iter_1000.caffemodel`

Output of tests:

```
I0727 00:01:43.715199 1934443264 net.cpp:283] Network initialization done.
I0727 00:01:43.718755 1934443264 caffe.cpp:358] Running for 50 iterations.
I0727 00:01:43.751853 1934443264 caffe.cpp:381] Batch 0, accuracy = 0.96
I0727 00:01:43.751885 1934443264 caffe.cpp:381] Batch 0, loss = 0.446991
I0727 00:01:43.783586 1934443264 caffe.cpp:381] Batch 1, accuracy = 0.96
I0727 00:01:43.783618 1934443264 caffe.cpp:381] Batch 1, loss = 0.462931
I0727 00:01:43.815724 1934443264 caffe.cpp:381] Batch 2, accuracy = 0.94
I0727 00:01:43.815757 1934443264 caffe.cpp:381] Batch 2, loss = 0.465818
I0727 00:01:43.844215 1934443264 caffe.cpp:381] Batch 3, accuracy = 0.95
I0727 00:01:43.844249 1934443264 caffe.cpp:381] Batch 3, loss = 0.50214
I0727 00:01:43.874644 1934443264 caffe.cpp:381] Batch 4, accuracy = 0.93
I0727 00:01:43.874686 1934443264 caffe.cpp:381] Batch 4, loss = 0.504952
I0727 00:01:43.906430 1934443264 caffe.cpp:381] Batch 5, accuracy = 0.92
I0727 00:01:43.906460 1934443264 caffe.cpp:381] Batch 5, loss = 0.542241
I0727 00:01:43.938120 1934443264 caffe.cpp:381] Batch 6, accuracy = 0.91
I0727 00:01:43.938150 1934443264 caffe.cpp:381] Batch 6, loss = 0.559257
I0727 00:01:43.968787 1934443264 caffe.cpp:381] Batch 7, accuracy = 0.93
I0727 00:01:43.968816 1934443264 caffe.cpp:381] Batch 7, loss = 0.472216
I0727 00:01:44.001312 1934443264 caffe.cpp:381] Batch 8, accuracy = 0.94
I0727 00:01:44.001341 1934443264 caffe.cpp:381] Batch 8, loss = 0.445917
I0727 00:01:44.029739 1934443264 caffe.cpp:381] Batch 9, accuracy = 0.92
I0727 00:01:44.029772 1934443264 caffe.cpp:381] Batch 9, loss = 0.480525
I0727 00:01:44.059865 1934443264 caffe.cpp:381] Batch 10, accuracy = 0.92
I0727 00:01:44.059898 1934443264 caffe.cpp:381] Batch 10, loss = 0.531594
I0727 00:01:44.088306 1934443264 caffe.cpp:381] Batch 11, accuracy = 0.92
I0727 00:01:44.088340 1934443264 caffe.cpp:381] Batch 11, loss = 0.543581
I0727 00:01:44.119846 1934443264 caffe.cpp:381] Batch 12, accuracy = 0.83
I0727 00:01:44.119877 1934443264 caffe.cpp:381] Batch 12, loss = 0.67841
I0727 00:01:44.150900 1934443264 caffe.cpp:381] Batch 13, accuracy = 0.94
I0727 00:01:44.150998 1934443264 caffe.cpp:381] Batch 13, loss = 0.471371
I0727 00:01:44.184655 1934443264 caffe.cpp:381] Batch 14, accuracy = 0.95
I0727 00:01:44.184686 1934443264 caffe.cpp:381] Batch 14, loss = 0.508547
I0727 00:01:44.214793 1934443264 caffe.cpp:381] Batch 15, accuracy = 0.92
I0727 00:01:44.214824 1934443264 caffe.cpp:381] Batch 15, loss = 0.507803
I0727 00:01:44.247390 1934443264 caffe.cpp:381] Batch 16, accuracy = 0.91
I0727 00:01:44.247419 1934443264 caffe.cpp:381] Batch 16, loss = 0.50551
I0727 00:01:44.276626 1934443264 caffe.cpp:381] Batch 17, accuracy = 0.9
I0727 00:01:44.276657 1934443264 caffe.cpp:381] Batch 17, loss = 0.547201
I0727 00:01:44.308115 1934443264 caffe.cpp:381] Batch 18, accuracy = 0.92
I0727 00:01:44.308146 1934443264 caffe.cpp:381] Batch 18, loss = 0.513329
I0727 00:01:44.337630 1934443264 caffe.cpp:381] Batch 19, accuracy = 0.92
I0727 00:01:44.337661 1934443264 caffe.cpp:381] Batch 19, loss = 0.560503
I0727 00:01:44.370326 1934443264 caffe.cpp:381] Batch 20, accuracy = 0.87
I0727 00:01:44.370358 1934443264 caffe.cpp:381] Batch 20, loss = 0.556425
I0727 00:01:44.400152 1934443264 caffe.cpp:381] Batch 21, accuracy = 0.84
I0727 00:01:44.400183 1934443264 caffe.cpp:381] Batch 21, loss = 0.614201
I0727 00:01:44.431962 1934443264 caffe.cpp:381] Batch 22, accuracy = 0.94
I0727 00:01:44.431995 1934443264 caffe.cpp:381] Batch 22, loss = 0.457587
I0727 00:01:44.461412 1934443264 caffe.cpp:381] Batch 23, accuracy = 0.92
I0727 00:01:44.461443 1934443264 caffe.cpp:381] Batch 23, loss = 0.555504
I0727 00:01:44.493445 1934443264 caffe.cpp:381] Batch 24, accuracy = 0.91
I0727 00:01:44.493516 1934443264 caffe.cpp:381] Batch 24, loss = 0.492875
I0727 00:01:44.523396 1934443264 caffe.cpp:381] Batch 25, accuracy = 0.94
I0727 00:01:44.523433 1934443264 caffe.cpp:381] Batch 25, loss = 0.471205
I0727 00:01:44.555320 1934443264 caffe.cpp:381] Batch 26, accuracy = 0.93
I0727 00:01:44.555371 1934443264 caffe.cpp:381] Batch 26, loss = 0.46078
I0727 00:01:44.584758 1934443264 caffe.cpp:381] Batch 27, accuracy = 0.91
I0727 00:01:44.584786 1934443264 caffe.cpp:381] Batch 27, loss = 0.516839
I0727 00:01:44.616545 1934443264 caffe.cpp:381] Batch 28, accuracy = 0.96
I0727 00:01:44.616575 1934443264 caffe.cpp:381] Batch 28, loss = 0.445957
I0727 00:01:44.645931 1934443264 caffe.cpp:381] Batch 29, accuracy = 0.88
I0727 00:01:44.645967 1934443264 caffe.cpp:381] Batch 29, loss = 0.6446
I0727 00:01:44.677723 1934443264 caffe.cpp:381] Batch 30, accuracy = 0.95
I0727 00:01:44.677752 1934443264 caffe.cpp:381] Batch 30, loss = 0.452647
I0727 00:01:44.707228 1934443264 caffe.cpp:381] Batch 31, accuracy = 0.9
I0727 00:01:44.707260 1934443264 caffe.cpp:381] Batch 31, loss = 0.548587
I0727 00:01:44.739732 1934443264 caffe.cpp:381] Batch 32, accuracy = 0.94
I0727 00:01:44.739761 1934443264 caffe.cpp:381] Batch 32, loss = 0.509797
I0727 00:01:44.769515 1934443264 caffe.cpp:381] Batch 33, accuracy = 0.94
I0727 00:01:44.769546 1934443264 caffe.cpp:381] Batch 33, loss = 0.499985
I0727 00:01:44.804766 1934443264 caffe.cpp:381] Batch 34, accuracy = 0.94
I0727 00:01:44.804795 1934443264 caffe.cpp:381] Batch 34, loss = 0.452719
I0727 00:01:44.833577 1934443264 caffe.cpp:381] Batch 35, accuracy = 0.84
I0727 00:01:44.833608 1934443264 caffe.cpp:381] Batch 35, loss = 0.628487
I0727 00:01:44.866924 1934443264 caffe.cpp:381] Batch 36, accuracy = 0.93
I0727 00:01:44.866955 1934443264 caffe.cpp:381] Batch 36, loss = 0.449822
I0727 00:01:44.895503 1934443264 caffe.cpp:381] Batch 37, accuracy = 0.81
I0727 00:01:44.895532 1934443264 caffe.cpp:381] Batch 37, loss = 0.656729
I0727 00:01:44.926841 1934443264 caffe.cpp:381] Batch 38, accuracy = 0.86
I0727 00:01:44.926874 1934443264 caffe.cpp:381] Batch 38, loss = 0.631623
I0727 00:01:44.958943 1934443264 caffe.cpp:381] Batch 39, accuracy = 0.95
I0727 00:01:44.958973 1934443264 caffe.cpp:381] Batch 39, loss = 0.479421
I0727 00:01:44.991243 1934443264 caffe.cpp:381] Batch 40, accuracy = 0.93
I0727 00:01:44.991273 1934443264 caffe.cpp:381] Batch 40, loss = 0.498626
I0727 00:01:45.019760 1934443264 caffe.cpp:381] Batch 41, accuracy = 0.95
I0727 00:01:45.019790 1934443264 caffe.cpp:381] Batch 41, loss = 0.52568
I0727 00:01:45.050673 1934443264 caffe.cpp:381] Batch 42, accuracy = 0.91
I0727 00:01:45.050703 1934443264 caffe.cpp:381] Batch 42, loss = 0.574733
I0727 00:01:45.080518 1934443264 caffe.cpp:381] Batch 43, accuracy = 0.9
I0727 00:01:45.080548 1934443264 caffe.cpp:381] Batch 43, loss = 0.556523
I0727 00:01:45.111312 1934443264 caffe.cpp:381] Batch 44, accuracy = 0.95
I0727 00:01:45.111343 1934443264 caffe.cpp:381] Batch 44, loss = 0.499967
I0727 00:01:45.140476 1934443264 caffe.cpp:381] Batch 45, accuracy = 0.87
I0727 00:01:45.140506 1934443264 caffe.cpp:381] Batch 45, loss = 0.537358
I0727 00:01:45.173477 1934443264 caffe.cpp:381] Batch 46, accuracy = 0.92
I0727 00:01:45.173516 1934443264 caffe.cpp:381] Batch 46, loss = 0.509583
I0727 00:01:45.204303 1934443264 caffe.cpp:381] Batch 47, accuracy = 0.91
I0727 00:01:45.204334 1934443264 caffe.cpp:381] Batch 47, loss = 0.453649
I0727 00:01:45.236830 1934443264 caffe.cpp:381] Batch 48, accuracy = 0.87
I0727 00:01:45.236860 1934443264 caffe.cpp:381] Batch 48, loss = 0.617736
I0727 00:01:45.266126 1934443264 caffe.cpp:381] Batch 49, accuracy = 0.91
I0727 00:01:45.266155 1934443264 caffe.cpp:381] Batch 49, loss = 0.528541
I0727 00:01:45.266161 1934443264 caffe.cpp:386] Loss: 0.521581
I0727 00:01:45.266175 1934443264 caffe.cpp:398] accuracy = 0.9154
I0727 00:01:45.266186 1934443264 caffe.cpp:398] loss = 0.521581 (* 1 = 0.521581 loss)
```

Partial hexdump:

`xxd -s 204800 -l 256 examples/mnist/lenet_compressed_iter_1000.caffemodel`

Output:

```
0032000: 0000 0080 0000 0000 0000 0000 0000 0080  ................
0032010: 0000 0080 0000 0080 0000 0080 0000 0080  ................
0032020: 0000 0000 0000 0000 0000 0000 0000 0080  ................
0032030: 0000 0080 0000 0000 0000 0080 0000 0080  ................
0032040: 0000 0080 0000 0000 0000 0000 0000 0000  ................
0032050: 0000 0080 0000 0000 0000 0000 0000 0080  ................
0032060: 0000 0000 0000 0000 0000 0080 0000 0000  ................
0032070: 0000 0000 0000 0080 0000 0000 0000 0000  ................
0032080: 0000 0000 0000 0080 0000 0000 0000 0080  ................
0032090: 0000 0000 0000 0080 0000 0080 be88 76bd  ..............v.
00320a0: 0000 0080 0000 0080 0000 0080 0000 0080  ................
00320b0: 0000 0080 0000 0080 0000 0080 0000 0000  ................
00320c0: 0000 0080 0000 0000 2c19 73bd 0000 0080  ........,.s.....
00320d0: 0000 0000 0000 0000 0000 0000 0000 0000  ................
00320e0: 0000 0080 0000 0000 0000 0080 0000 0080  ................
00320f0: 0000 0000 0000 0000 0000 0080 0000 0080  ................
```

## Citations

All acknowledgements go to the following research paper by Han et al. ([Learning both Weights and Connections for Efficient
Neural Networks](https://arxiv.org/pdf/1506.02626.pdf)):

```
  @article{DBLP:journals/corr/HanPTD15,
    author    = {Song Han and
                 Jeff Pool and
                 John Tran and
                 William J. Dally},
    title     = {Learning both Weights and Connections for Efficient Neural Networks},
    journal   = {CoRR},
    volume    = {abs/1506.02626},
    year      = {2015},
    url       = {http://arxiv.org/abs/1506.02626},
    timestamp = {Wed, 01 Jul 2015 15:10:24 +0200},
    biburl    = {http://dblp.uni-trier.de/rec/bib/journals/corr/HanPTD15},
    bibsource = {dblp computer science bibliography, http://dblp.org}
  }
```


# Caffe

[![Build Status](https://travis-ci.org/BVLC/caffe.svg?branch=master)](https://travis-ci.org/BVLC/caffe)
[![License](https://img.shields.io/badge/license-BSD-blue.svg)](LICENSE)

Caffe is a deep learning framework made with expression, speed, and modularity in mind.
It is developed by the Berkeley Vision and Learning Center ([BVLC](http://bvlc.eecs.berkeley.edu)) and community contributors.

Check out the [project site](http://caffe.berkeleyvision.org) for all the details like

- [DIY Deep Learning for Vision with Caffe](https://docs.google.com/presentation/d/1UeKXVgRvvxg9OUdh_UiC5G71UMscNPlvArsWER41PsU/edit#slide=id.p)
- [Tutorial Documentation](http://caffe.berkeleyvision.org/tutorial/)
- [BVLC reference models](http://caffe.berkeleyvision.org/model_zoo.html) and the [community model zoo](https://github.com/BVLC/caffe/wiki/Model-Zoo)
- [Installation instructions](http://caffe.berkeleyvision.org/installation.html)

and step-by-step examples.

[![Join the chat at https://gitter.im/BVLC/caffe](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/BVLC/caffe?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

Please join the [caffe-users group](https://groups.google.com/forum/#!forum/caffe-users) or [gitter chat](https://gitter.im/BVLC/caffe) to ask questions and talk about methods and models.
Framework development discussions and thorough bug reports are collected on [Issues](https://github.com/BVLC/caffe/issues).

Happy brewing!

## License and Citation

Caffe is released under the [BSD 2-Clause license](https://github.com/BVLC/caffe/blob/master/LICENSE).
The BVLC reference models are released for unrestricted use.

Please cite Caffe in your publications if it helps your research:

    @article{jia2014caffe,
      Author = {Jia, Yangqing and Shelhamer, Evan and Donahue, Jeff and Karayev, Sergey and Long, Jonathan and Girshick, Ross and Guadarrama, Sergio and Darrell, Trevor},
      Journal = {arXiv preprint arXiv:1408.5093},
      Title = {Caffe: Convolutional Architecture for Fast Feature Embedding},
      Year = {2014}
    }
