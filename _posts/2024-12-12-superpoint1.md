---
title: "SuperPoint环境搭建"
date: 2024-12-12
tags: [SuperPoint, 深度学习, tensorflow]
---

MagicLeap开发了SuperPoint，放出了训练好的模型，有Pytorch版本，但并未开源训练过程。

https://github.com/rpautrat/SuperPoint rpautrat这位大佬用Tensorflow1.15复现了训练过程，效果上看差不多，所以用他这个代码的人貌似不少。但是，一个关键问题是用的Tensorflow1.15, 是个老版本了。现在显卡驱动、cuda版本都升级了很多，导致直接去Tensorflow官网下载Tensorflow1.15版本会发现和高版本的cuda版本不兼容了。

一开始自己尝试把这个代码升级到兼容Tensorflow2.x，但发现要改的代码越来越多，本身对Tensorflow也说不上很熟悉，觉得工作量会比较大，所以还是考虑基于Tensorflow1.15进行搭建。

又是一番谷歌、AI检索、折腾，发现英伟达已经考虑了这个问题。英伟达提供了高版本cuda下对tensorflow1.x系列的支持，算是功德一件。先装好anaconda，建立一个专用的虚拟环境，然后具体操作方法如下

```bash
pip install --user nvidia-pyindex
pip install --user nvidia-tensorflow[horovod]
```
装好后可以看到如下的一些包
```bash
(tf1.15) $ pip list | grep tensorflow
nvidia-tensorflow             1.15.5+nv23.3
tensorflow-estimator          1.15.1
tensorflow-io-gcs-filesystem  0.34.0
(tf1.15) $ pip list | grep nvidia
nvidia-cublas-cu12            12.4.5.8
nvidia-cuda-cupti-cu12        12.4.127
nvidia-cuda-nvcc-cu12         12.4.131
nvidia-cuda-nvrtc-cu12        12.4.127
nvidia-cuda-runtime-cu12      12.4.127
nvidia-cudnn-cu12             8.9.7.29
nvidia-cufft-cu12             11.2.1.3
nvidia-curand-cu12            10.3.5.147
nvidia-cusolver-cu12          11.6.1.9
nvidia-cusparse-cu12          12.3.1.170
nvidia-dali-cuda110           1.23.0
nvidia-dali-nvtf-plugin       1.23.0+nv23.3
nvidia-horovod                0.27.0+nv23.3
nvidia-ml-py                  12.535.133
nvidia-nccl-cu12              2.21.5
nvidia-nvjitlink-cu12         12.4.127
nvidia-pyindex                1.0.9
nvidia-tensorflow             1.15.5+nv23.3
```
把这些装好后，对前面的开源代码进行一些额外修改，比如yaml库升级，会导致相关API调用变化。然后就能跑起训练流程了。

https://github.com/eric-yyjau/pytorch-superpoint，有人用pytorch实现了下，但好像还不是很完善或者说那么易用，具体效果还没细看，但也是一个有价值的工作。

目前深度学习搞特征提取的很多，但真正落地的有哪些？SuperPoint应该是一个。




