[English](https://github.com/mindspore-lab/mindocr/blob/main/configs/rec/crnn/README.md) | 中文

# CRNN
<!--- Guideline: use url linked to abstract in ArXiv instead of PDF for fast loading.  -->

> [An End-to-End Trainable Neural Network for Image-based Sequence Recognition and Its Application to Scene Text Recognition](https://arxiv.org/abs/1507.05717)

## 模型描述
<!--- Guideline: Introduce the model and architectures. Cite if you use/adopt paper explanation from others. -->

卷积递归神经网络 (CRNN) 将 CNN 特征提取和 RNN 序列建模以及转录集成到一个统一的框架中。

如架构图（图 1）所示，CRNN 首先通过卷积层从输入图像中提取特征序列。由此一来，图像由提取的序列特征图表示，其中每个向量都与输入图像上的感受野相关联。 为了进一步处理特征，CRNN 采用循环神经网络层来预测每个帧的标签分布。为了将分布映射到文本字段，CRNN 添加了一个转录层，以将每帧预测转换为最终标签序列。 [<a href="#参考文献">1</a>]

<!--- Guideline: If an architecture table/figure is available in the paper, put one here and cite for intuitive illustration. -->

<p align="center">
  <img src="https://user-images.githubusercontent.com/26082447/224601239-a569a1d4-4b29-4fa8-804b-6690cb50caef.PNG" width=450 />
</p>
<p align="center">
  <em> 图1. CRNN架构图 [<a href="#参考文献">1</a>] </em>
</p>


## 配套版本

| mindspore  | ascend driver  |    firmware    | cann toolkit/kernel |
|:----------:|:--------------:|:--------------:|:-------------------:|
|   2.5.0    |    24.1.0      |   7.5.0.3.220  |     8.0.0.beta1     |



## 快速开始

### 安装

环境安装教程请参考MindOCR的 [installation instruction](https://github.com/mindspore-lab/mindocr#installation).

### 数据准备

#### 数据集下载

LMDB格式的训练及验证数据集可以从[这里](https://www.dropbox.com/sh/i39abvnefllx2si/AAAbAYRvxzRp3cIE5HzqUw3ra?dl=0) (出处: [deep-text-recognition-benchmark](https://github.com/clovaai/deep-text-recognition-benchmark#download-lmdb-dataset-for-traininig-and-evaluation-from-here))下载。连接中的文件包含多个压缩文件，其中:
- `data_lmdb_release.zip` 包含了**完整**的一套数据集，有训练集(training/），验证集(validation/)以及测试集(evaluation)。
    - `training.zip` 包括两个数据集，分别是 [MJSynth (MJ)](http://www.robots.ox.ac.uk/~vgg/data/text/) 和 [SynthText (ST)](https://academictorrents.com/details/2dba9518166cbd141534cbf381aa3e99a087e83c)
    - `validation.zip` 是多个单独数据集的训练集的一个合集，包括[IC13](http://rrc.cvc.uab.es/?ch=2), [IC15](http://rrc.cvc.uab.es/?ch=4), [IIIT](http://cvit.iiit.ac.in/projects/SceneTextUnderstanding/IIIT5K.html), 和 [SVT](http://www.iapr-tc11.org/mediawiki/index.php/The_Street_View_Text_Dataset)。
    - `evaluation.zip` 包含多个基准评估数据集，有[IIIT](http://cvit.iiit.ac.in/projects/SceneTextUnderstanding/IIIT5K.html), [SVT](http://www.iapr-tc11.org/mediawiki/index.php/The_Street_View_Text_Dataset), [IC03](http://www.iapr-tc11.org/mediawiki/index.php/ICDAR_2003_Robust_Reading_Competitions), [IC13](http://rrc.cvc.uab.es/?ch=2), [IC15](http://rrc.cvc.uab.es/?ch=4), [SVTP](http://openaccess.thecvf.com/content_iccv_2013/papers/Phan_Recognizing_Text_with_2013_ICCV_paper.pdf)和 [CUTE](http://cs-chan.com/downloads_CUTE80_dataset.html)
- `validation.zip`: 与 data_lmdb_release.zip 中的validation/ 一样。
- `evaluation.zip`: 与 data_lmdb_release.zip 中的evaluation/ 一样。

#### 数据集使用

解压文件后，数据文件夹结构如下：

``` text
data_lmdb_release/
├── evaluation
│   ├── CUTE80
│   │   ├── data.mdb
│   │   └── lock.mdb
│   ├── IC03_860
│   │   ├── data.mdb
│   │   └── lock.mdb
│   ├── IC03_867
│   │   ├── data.mdb
│   │   └── lock.mdb
│   ├── IC13_1015
│   │   ├── data.mdb
│   │   └── lock.mdb
│   ├── ...
├── training
│   ├── MJ
│   │   ├── MJ_test
│   │   │   ├── data.mdb
│   │   │   └── lock.mdb
│   │   ├── MJ_train
│   │   │   ├── data.mdb
│   │   │   └── lock.mdb
│   │   └── MJ_valid
│   │       ├── data.mdb
│   │       └── lock.mdb
│   └── ST
│       ├── data.mdb
│       └── lock.mdb
└── validation
    ├── data.mdb
    └── lock.mdb
```

在这里，我们使用 `training/` 文件夹下的数据集进行 **training**，并使用联合数据集 `validation/` 进行验证。训练后，我们使用 `evaluation/` 下的数据集来评估模型的准确性。

**Training:** (total 14,442,049 samples)
- [MJSynth (MJ)](http://www.robots.ox.ac.uk/~vgg/data/text/)
  - Train: 21.2 GB, 7224586 samples
  - Valid: 2.36 GB, 802731 samples
  - Test: 2.61 GB, 891924 samples
- [SynthText (ST)](https://academictorrents.com/details/2dba9518166cbd141534cbf381aa3e99a087e83c)
  - Train: 16.0 GB, 5522808 samples

**Validation:**
- Valid: 138 MB, 6992 samples

**Evaluation:** (total 12,067 samples)
- [CUTE80](http://cs-chan.com/downloads_CUTE80_dataset.html): 8.8 MB, 288 samples
- [IC03_860](http://www.iapr-tc11.org/mediawiki/index.php/ICDAR_2003_Robust_Reading_Competitions): 36 MB, 860 samples
- [IC03_867](http://www.iapr-tc11.org/mediawiki/index.php/ICDAR_2003_Robust_Reading_Competitions): 4.9 MB, 867 samples
- [IC13_857](http://rrc.cvc.uab.es/?ch=2): 72 MB, 857 samples
- [IC13_1015](http://rrc.cvc.uab.es/?ch=2): 77 MB, 1015 samples
- [IC15_1811](http://rrc.cvc.uab.es/?ch=4): 21 MB, 1811 samples
- [IC15_2077](http://rrc.cvc.uab.es/?ch=4): 25 MB, 2077 samples
- [IIIT5k_3000](http://cvit.iiit.ac.in/projects/SceneTextUnderstanding/IIIT5K.html): 50 MB, 3000 samples
- [SVT](http://www.iapr-tc11.org/mediawiki/index.php/The_Street_View_Text_Dataset): 2.4 MB, 647 samples
- [SVTP](http://openaccess.thecvf.com/content_iccv_2013/papers/Phan_Recognizing_Text_with_2013_ICCV_paper.pdf): 1.8 MB, 645 samples

### 配置说明

#### 模型训练的数据配置

如欲重现模型的训练，建议修改配置yaml如下：

```yaml
...
train:
  ...
  dataset:
    type: LMDBDataset
    dataset_root: dir/to/data_lmdb_release/                           # 训练数据集根目录
    data_dir: training/                                               # 训练数据集目录，将与`dataset_root`拼接形成完整训练数据集目录
    # label_files:                                                    # 训练数据集的标签文件路径，将与`dataset_root`拼接形成完整的训练数据的标签文件路径。当数据集为LMDB格式时无需配置
...
eval:
  dataset:
    type: LMDBDataset
    dataset_root: dir/to/data_lmdb_release/                           # 验证数据集根目录
    data_dir: validation/                                             # 验证数据集目录，将与`dataset_root`拼接形成完整验证数据集目录
    # label_file:                                                     # 验证数据集的标签文件路径，将与`dataset_root`拼接形成完整的验证或评估数据的标签文件路径。当数据集为LMDB格式时无需配置
  ...
```

#### 模型评估的数据配置

我们使用 `evaluation/` 下的数据集作为基准数据集。在**每个单独的数据集**（例如 CUTE80、IC03_860 等）上，我们通过将数据集的目录设置为评估数据集来执行完整评估。这样，我们就得到了每个数据集对应精度的列表，然后报告的精度是这些值的平均值。

如要重现报告的评估结果，您可以：
- 方法 1：对所有单个数据集重复评估步骤：CUTE80、IC03_860、IC03_867、IC13_857、IC131015、IC15_1811、IC15_2077、IIIT5k_3000、SVT、SVTP。然后取平均分。

- 方法 2：将所有基准数据集文件夹放在同一目录下，例如`evaluation/`，对应修改配置文件中`eval.dataset.data_dir`变量配置，并执行脚本`tools/benchmarking/multi_dataset_eval.py`。

1.评估一个特定的数据集

例如，您可以通过修改配置 yaml 来评估数据集“CUTE80”上的模型，如下所示：

```yaml
...
train:
  # 无需修改训练部分的配置，因验证或推理的时候不必使用该部分
...
eval:
  dataset:
    type: LMDBDataset
    dataset_root: dir/to/data_lmdb_release/                           # 训练数据集根目录
    data_dir: evaluation/CUTE80/                                      # 评估数据集目录，将与`dataset_root`拼接形成完整验证或评估数据集目录
    # label_file:                                                     # 评估数据集的标签文件路径，将与`dataset_root`拼接形成完整的评估数据的标签文件路径。当数据集为LMDB格式时无需配置
  ...
```

通过使用上述配置 yaml 运行 [模型评估](#模型评估) 部分中所述的`tools/eval.py`，您可以获得数据集 CUTE80 的准确度性能。


2. 对同一文件夹下的多个数据集进行评估

假设您已将所有 benckmark 数据集置于 evaluation/ 下，如下所示：

``` text
data_lmdb_release/
├── evaluation
│   ├── CUTE80
│   │   ├── data.mdb
│   │   └── lock.mdb
│   ├── IC03_860
│   │   ├── data.mdb
│   │   └── lock.mdb
│   ├── IC03_867
│   │   ├── data.mdb
│   │   └── lock.mdb
│   ├── IC13_1015
│   │   ├── data.mdb
│   │   └── lock.mdb
│   ├── ...
```

然后你可以通过如下修改配置yaml来评估每个数据集，并执行脚本`tools/benchmarking/multi_dataset_eval.py`。

```yaml
...
train:
  # NO NEED TO CHANGE ANYTHING IN TRAIN SINCE IT IS NOT USED
...
eval:
  dataset:
    type: LMDBDataset
    dataset_root: dir/to/data_lmdb_release/                           # Root dir of evaluation dataset
    data_dir: evaluation/                                   # Dir of evaluation dataset, concatenated with `dataset_root` to be the complete dir of evaluation dataset
    # label_file:                                                     # Path of evaluation label file, concatenated with `dataset_root` to be the complete path of evaluation label file, not required when using LMDBDataset
  ...
```

#### 检查配置文件

除了数据集的设置，请同时重点关注以下变量的配置：`system.distribute`, `system.val_while_train`, `common.batch_size`, `train.ckpt_save_dir`, `train.dataset.dataset_root`, `train.dataset.data_dir`, `train.dataset.label_file`,
`eval.ckpt_load_path`, `eval.dataset.dataset_root`, `eval.dataset.data_dir`, `eval.dataset.label_file`, `eval.loader.batch_size`。说明如下：

```yaml
system:
  distribute: True                                                    # 分布式训练为True，单卡训练为False
  amp_level: 'O3'
  seed: 42
  val_while_train: True                                               # 边训练边验证
  drop_overflow_update: False
common:
  ...
  batch_size: &batch_size 64                                          # 训练批大小
...
train:
  ckpt_save_dir: './tmp_rec'                                          # 训练结果（包括checkpoint、每个epoch的性能和曲线图）保存目录
  dataset_sink_mode: False
  dataset:
    type: LMDBDataset
    dataset_root: dir/to/data_lmdb_release/                           # 训练数据集根目录
    data_dir: training/                                               # 训练数据集目录，将与`dataset_root`拼接形成完整训练数据集目录
    # label_files:                                                    # 训练数据集的标签文件路径，将与`dataset_root`拼接形成完整的训练数据的标签文件路径。当数据集为LMDB格式时无需配置
...
eval:
  ckpt_load_path: './tmp_rec/best.ckpt'                               # checkpoint文件路径
  dataset_sink_mode: False
  dataset:
    type: LMDBDataset
    dataset_root: dir/to/data_lmdb_release/                           # 验证或评估数据集根目录
    data_dir: validation/                                             # 验证或评估数据集目录，将与`dataset_root`拼接形成完整验证或评估数据集目录
    # label_file:                                                     # 验证或评估数据集的标签文件路径，将与`dataset_root`拼接形成完整的验证或评估数据的标签文件路径。当数据集为LMDB格式时无需配置
  ...
  loader:
      shuffle: False
      batch_size: 64                                                  # 验证或评估批大小
...
```

**注意:**
- 由于全局批大小 （batch_size x num_devices） 是对结果复现很重要，因此当NPU卡数发生变化时，调整`batch_size`以保持全局批大小不变，或根据新的全局批大小线性调整学习率。


### 模型训练
<!--- Guideline: Avoid using shell script in the command line. Python script preferred. -->

* 分布式训练

使用预定义的训练配置可以轻松重现报告的结果。对于在多个昇腾910设备上的分布式训练，请将配置参数`system.distribute`修改为True，并运行：

```shell
# 在多个 Ascend 设备上进行分布式训练
# worker_num代表分布式总进程数量。
# local_worker_num代表当前节点进程数量。
# 进程数量即为训练使用的NPU的数量，单机多卡情况下worker_num和local_worker_num需保持一致。
msrun --worker_num=8 --local_worker_num=8 python tools/train.py --config configs/rec/crnn/crnn_resnet34.yaml

# 经验证，绑核在大部分情况下有性能加速，请配置参数并运行
msrun --bind_core=True --worker_num=8 --local_worker_num=8 python tools/train.py --config configs/rec/crnn/crnn_resnet34.yaml
```
**注意:** 有关 msrun 配置的更多信息，请参考[此处](https://www.mindspore.cn/docs/zh-CN/master/model_train/parallel/msrun_launcher.html).


* 单卡训练

如果要在没有分布式训练的情况下在较小的数据集上训练或微调模型，请将配置参数`system.distribute`修改为False 并运行：

```shell
# CPU/Ascend 设备上的单卡训练
python tools/train.py --config configs/rec/crnn/crnn_resnet34.yaml
```

训练结果（包括checkpoint、每个epoch的性能和曲线图）将被保存在yaml配置文件的`train.ckpt_save_dir`参数配置的目录下，默认为`./tmp_rec`。

### 模型评估

若要评估已训练模型的准确性，可以使用`eval.py`。请将yaml配置文件的参数`eval.ckpt_load_path`设置为模型checkpoint的文件路径，参数`eval.dataset.data_dir`设置为评估数据集目录，参数`system.distribute`设置为False，然后运行：

```
python tools/eval.py --config configs/rec/crnn/crnn_resnet34.yaml
```

类似的，可以修改yaml配置文件的`eval.ckpt_load_path`、`eval.dataset.data_dir`、`system.distribute`等参数，然后使用`multi_dataset_eval.py`评估多个数据集的模型准确性：

```
python tools/benchmarking/multi_dataset_eval.py --config configs/rec/crnn/crnn_resnet34.yaml
```

## 字符词典

### 默认设置

在数据处理时，真实文本会根据提供的字符字典转换为标签 ID，字典中键是字符，值是 ID。默认情况下，字典 **"0123456789abcdefghijklmnopqrstuvwxyz"**，这代表着id=0 将对应字符'0'。在默认设置下，字典只考虑数字和小写英文字符，不包括空格。


### 内置词典

Mindocr内置了一部分字典，均放在了 `mindocr/utils/dict/` 位置，可选择合适的字典使用。

- `en_dict.txt` 是一个包含94个字符的英文字典，其中有数字，常用符号以及大小写的英文字母。
- `ch_dict.txt` 是一个包含6623个字符的中文字典，其中有常用的繁简体中文，数字，常用符号以及大小写的英文字母。


### 自定义词典

您也可以自定义一个字典文件 (***.txt)， 放在 `mindocr/utils/dict/` 下，词典文件格式应为每行一个字符的.txt 文件。


如需使用指定的词典，请将参数 `common.character_dict_path` 设置为字典的路径，并将参数 `common.num_classes` 改成对应的数量，即字典中字符的数量 + 1。


**注意：**
- 您可以通过将配置文件中的参数 `common.use_space_char` 设置为 True 来包含空格字符。
- 请记住检查配置文件中的 `dataset->transform_pipeline->RecCTCLabelEncode->lower` 参数的值。如果词典中有大小写字母而且想区分大小写的话，请将其设置为 False。


## 中文识别模型训练

目前，CRNN模型支持中英文字识别并提供相应的预训练权重。详细内容如下

### 中文数据集准备及配置

我们采用公开的中文基准数据集[Benchmarking-Chinese-Text-Recognition](https://github.com/FudanVI/benchmarking-chinese-text-recognition)进行CRNN模型的训练和验证。

详细的数据准备和config文件配置方式, 请参考 [中文识别数据集准备](../../../docs/zh/datasets/chinese_text_recognition.md)


## 性能表现

### 通用泛化中文模型

在采用图模式的ascend 910*上实验结果，mindspore版本为2.5.0

| **model name** | **backbone** | **cards** | **batch size** | **language** | **jit level** | **graph compile** | **ms/step** | **img/s** | **scene** | **web** | **document** |                                                                               **recipe**                                                                                |                                                                                              **weight**                                                                                               |
| :------------: | :----------: | :-------: | :------------: | :----------: | :-----------: | :---------------: | :---------: | :-------: |:---------:|:-------:|:------------:|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------:| :---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------: |
|      CRNN      | ResNet34_vd  |     4     |      256       |   Chinese    |      O2       |     203.48 s      |    38.01    |   1180    |  60.71%   | 65.94%  |    97.67%    |                                    [yaml](https://github.com/mindspore-lab/mindocr/blob/main/configs/rec/crnn/crnn_resnet34_ch.yaml)                                    | [ckpt](https://download.mindspore.cn/toolkits/mindocr/crnn/crnn_resnet34_ch-7a342e3c.ckpt) \| [mindir](https://download.mindspore.cn/toolkits/mindocr/crnn/crnn_resnet34_ch-7a342e3c-105bccb2.mindir) |


> 链接中模型的MindIR导出时的输入Shape为`(1, 3, 32, 320)`.

### 细分领域模型

#### 训练性能表现

在采用图模式的ascend 910*上实验结果，mindspore版本为2.5.0

| **model name** | **backbone** | **train dataset** | **params(M)** | **cards** | **batch size** | **jit level** | **graph compile** | **ms/step** | **img/s** | **accuracy** |                                          **recipe**                                           |                                                                                           **weight**                                                                                           |
|:--------------:| :----------: | :---------------: | :-----------: | :-------: | :------------: | :-----------: |:-----------------:|:-----------:|:---------:|:------------:|:---------------------------------------------------------------------------------------------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------:|
|      CRNN      |     VGG7     |       MJ+ST       |     8.72      |     8     |       16       |      O2       |       59 s        |    15.47    |  8274.08  |    81.31%    |  [yaml](https://github.com/mindspore-lab/mindocr/blob/main/configs/rec/crnn/crnn_vgg7.yaml)   |                                               [ckpt](https://download-mindspore.osinfra.cn/toolkits/mindocr/crnn/crnn_vgg7-6faf1b2d-910v2.ckpt)                                                |
|      CRNN      | ResNet34_vd  |       MJ+ST       |     24.48     |     8     |       64       |      O2       |     120.41 s      |    60.86    |  8412.75  |    84.73%    |[yaml](https://github.com/mindspore-lab/mindocr/blob/main/configs/rec/crnn/crnn_resnet34.yaml) | [ckpt](https://download.mindspore.cn/toolkits/mindocr/crnn/crnn_resnet34-83f37f07.ckpt) \| [mindir](https://download.mindspore.cn/toolkits/mindocr/crnn/crnn_resnet34-83f37f07-eb10a0c9.mindir) |

在各个基准数据集（IC03，IC13，IC15，IIIT，SVT，SVTP，CUTE）上的准确率：

| **model name** | **backbone** | **cards** | **IC03_860** | **IC03_867** | **IC13_857** | **IC13_1015** | **IC15_1811** | **IC15_2077** | **IIIT5k_3000** | **SVT** | **SVTP** | **CUTE80** | **average** |
| :------------: | :----------: | :-------: |:------------:|:------------:|:------------:|:-------------:|:-------------:|:-------------:|:---------------:|:-------:|:--------:|:----------:|:-----------:|
|      CRNN      |     VGG7     |     1     |    93.72%    |    93.43%    |    91.83%    |    90.84%     |    70.84%     |    64.95%     |     84.40%      | 82.84%  |  72.87%  |   67.36%   |   81.31%    |
|      CRNN      | ResNet34_vd  |     1     |    95.35%    |    95.27%    |    93.70%    |    92.71%     |    75.65%     |    69.72%     |     87.30%      | 86.09%  |  78.60%  |   72.92%   |   84.73%    |





#### 推理性能表现

在采用图模式的ascend 310P上实验结果，mindspore lite版本为2.5.0

| model name |  backbone   | test dataset | params(M) | cards | batch size | **jit level** | **graph compile** | img/s  |
| :--------: | :---------: | :----------: | :-------: | :---: | :--------: | :-----------: | :---------------: | :----: |
|    CRNN    | ResNet34_vd |     IC15     |   24.48   |   1   |     1      |      O2       |      10.46 s      | 361.09 |
|    CRNN    | ResNet34_vd |     SVT      |   24.48   |   1   |     1      |      O2       |      10.31 s      | 274.67 |



### 注意
- 如需在其他环境配置重现训练结果，请确保全局批量大小与原配置文件保持一致。
- 模型所能识别的字符都是默认的设置，即所有英文小写字母a至z及数字0至9，详细请看[字符词典](#字符词典)
- 模型都是从头开始训练的，无需任何预训练。关于训练和测试数据集的详细介绍，请参考[数据准备](#数据准备)章节。
- CRNN_VGG7和CRNN_ResNet34_vd的MindIR导出时的输入Shape均为(1, 3, 32, 100)。



## 参考文献
<!--- Guideline: Citation format GB/T 7714 is suggested. -->

[1] Baoguang Shi, Xiang Bai, Cong Yao. An End-to-End Trainable Neural Network for Image-based Sequence Recognition and Its Application to Scene Text Recognition. arXiv preprint arXiv:1507.05717, 2015.
