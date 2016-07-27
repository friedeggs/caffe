# Fork

Adds a `compress` command that roughly implements the recent NIPS 2015 paper _Learning both Weights and Connections for Efficient Neural Networks_ by Han et al.

**In development.**

## Usage

`build/tools/caffe compress -solver <path to solver prototxt file> -weights <path to trained model> -layertype [InnerProduct | Convolution]`

#### Flags
Supports either `-layertype` (for all layers of given type) or `-layername` (for a specific layer). Exactly one of these options must be provided.

__Note:__ Convolution is not yet implemented.


## Example

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
...
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

    @inproceedings{han2015learning,
      title={Learning both weights and connections for efficient neural network},
      author={Han, Song and Pool, Jeff and Tran, John and Dally, William},
      booktitle={Advances in Neural Information Processing Systems},
      pages={1135--1143},
      year={2015}
    }


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
