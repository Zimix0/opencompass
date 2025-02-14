# 数据集准备和选择

本节教程主要关注如何准备 OpenCompass 已支持的数据集，并构建需要的配置文件完成数据集的选择。

## 数据集配置文件目录结构

首先简单介绍一下 OpenCompass `configs/datasets` 目录下的结构，如下所示：

```
configs/datasets/
├── agieval
├── apps
├── ARC_c
├── ...
├── CLUE_afqmc  # 数据集
│   ├── CLUE_afqmc_gen_901306.py  # 不同版本数据集配置文件
│   ├── CLUE_afqmc_gen.py
│   ├── CLUE_afqmc_ppl_378c5b.py
│   ├── CLUE_afqmc_ppl_6507d7.py
│   ├── CLUE_afqmc_ppl_7b0c1e.py
│   └── CLUE_afqmc_ppl.py
├── ...
├── XLSum
├── Xsum
└── z_bench
```

在 `configs/datasets` 目录结构下，我们直接展平所有数据集，在各个数据集对应的文件夹下存在多个数据集配置。

数据集配置文件名由以下命名方式构成 `{数据集名称}_{评测方式}_{prompt版本号}.py`，以 `CLUE_afqmc/CLUE_afqmc_gen_db509b.py` 为例，该配置文件则为中文通用能力下的 `CLUE_afqmc` 数据集，对应的评测方式为 `gen`，即生成式评测，对应的prompt版本号为 `db509b`；同样的， `CLUE_afqmc_ppl_00b348.py` 指评测方式为`ppl`即判别式评测，prompt版本号为 `00b348` 。

除此之外，不带版本号的文件，例如： `CLUE_afqmc_gen.py` 则指向该评测方式最新的prompt配置文件，通常来说会是精度最高的prompt。

## 数据集准备

OpenCompass 支持的数据集主要包括两个部分：

1. Huggingface 数据集

[Huggingface Dataset](https://huggingface.co/datasets) 提供了大量的数据集。OpenCompass 已经支持了大多数常用于性能比较的数据集，具体支持的数据集列表请直接在 `configs/dataset` 下进行查找。

2. 第三方数据集

除了支持 Huggingface 已有的数据集， OpenCompass 还提供了一些第三方数据集及自建CN数据集。运行以下命令，将数据集统一下载并放置在`./data`目录下即可完成数据集准备。

```bash
# 在 OpenCompass 目录下运行
wget https://github.com/InternLM/opencompass/releases/download/0.1.0/OpenCompassData.zip
unzip OpenCompassData.zip
```

需要注意的是，Repo中不仅包含自建的数据集，为了方便也加入了部分HF已支持的数据集方便测试。

## 数据集选择

在各个数据集配置文件中，数据集将会被定义在 `{}_datasets` 变量当中，例如下面 `CLUE_afqmc/CLUE_afqmc_gen_db509b.py` 中的 `afqmc_datasets`。

```python
afqmc_datasets = [
    dict(
        abbr="afqmc-dev",
        type=AFQMCDataset_V2,
        path="./data/CLUE/AFQMC/dev.json",
        reader_cfg=afqmc_reader_cfg,
        infer_cfg=afqmc_infer_cfg,
        eval_cfg=afqmc_eval_cfg,
    ),
]
```

以及 `CLUE_cmnli/CLUE_cmnli_ppl_b78ad4.py` 中的 `cmnli_datasets`。

```python
cmnli_datasets = [
    dict(
        type=HFDataset,
        abbr='cmnli',
        path='json',
        split='train',
        data_files='./data/CLUE/cmnli/cmnli_public/dev.json',
        reader_cfg=cmnli_reader_cfg,
        infer_cfg=cmnli_infer_cfg,
        eval_cfg=cmnli_eval_cfg)
]
```

以上述两个数据集为例， 如果用户想同时评测这两个数据集，可以在 `configs` 目录下新建一个配置文件，我们使用  `mmengine` 配置中直接import的机制来构建数据集部分的参数，如下所示：

```python
from mmengine.config import read_base

with read_base():
    from .datasets.CLUE_afqmc.CLUE_afqmc_gen_db509b import afqmc_datasets
    from .datasets.CLUE_cmnli.CLUE_cmnli_ppl_b78ad4 import cmnli_datasets

datasets = []
datasets += afqmc_datasets
datasets += cmnli_datasets
```

用户可以根据需要，选择不同能力不同数据集以及不同评测方式的配置文件来构建评测脚本中数据集的部分。

有关如何启动评测任务，以及如何评测自建数据集可以参考相关文档。
