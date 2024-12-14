---
title: "SuperPoint模型训练"
date: 2024-12-14
tags: [SuperPoint, 深度学习, tensorflow]
---

开发环境搭建好，下载MS的MS-COCO 2014 数据集以及HPatches数据集，然后可以开始模型训练步骤。

1. 数据的准备：
settings.py 中 
```
DATA_PATH = '/home/dataset/SuperPointTrain'
EXPER_PATH = '/home/github/SuperPoint/results'
```
DATA_PATH 指向 数据集存放路径以及一些中间结果的保存路径
目录结构如下
```
$DATA_PATH
|-- COCO
|   |-- train2014
|   |   |-- file1.jpg
|   |   `-- ...
|   `-- val2014
|       |-- file1.jpg
|       `-- ...
`-- HPatches
|   |-- i_ajuntament
|   `-- ...
`-- synthetic_shapes  # will be automatically created
```
>EXPER_PATH 指向 训练结果路径，后面模型导出也会基于这个路径

2. 在合成的形状上训练MagicPoint
```
python experiment.py train configs/magic-point_shapes.yaml magic-point_synth
```
3. 对 MS-COCO  导出 检测，形成伪真值 特征点标注，
```
python export_detections.py configs/magic-point_coco_export.yaml magic-point_synth --pred_only --batch_size=5 --export_name=magic-point_coco-export1
```
4. MS-COCO 上训练MagicPoint
```
python experiment.py train configs/magic-point_coco_train.yaml magic-point_coco
```
第二步和第三步 可能需要重复进行多次

5. MS-COCO 上训练SuperPoint
```
python experiment.py train configs/superpoint_coco.yaml superpoint_coco
```



