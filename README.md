# TuMeNiche

TuMeNiche 是一个用于从空间转录组数据中识别肿瘤代谢生态位的层级化计算框架。该框架整合空间转录组数据、肿瘤恶性程度估计、代谢先验知识、图表示学习和空间代谢耦合分析，用于刻画肿瘤区域内部复杂的代谢异质性。

## 项目简介

TuMeNiche 旨在分析空间转录组数据，并识别肿瘤组织中具有空间组织结构的代谢生态位。整体流程主要包括：

1. 空间转录组数据预处理
2. 肿瘤区与非肿瘤区识别
3. 基于 SpaCET 的细胞类型反卷积
4. 代谢相关先验知识构建
5. 基于 NicheCompass 的图表示学习
6. 空间代谢耦合强度计算
7. 层级化肿瘤代谢生态位注释

## 仓库结构

```text
TuMeNiche/
├── README.md
├── data/
│   └── pre_data/
├── src/
│   ├── nichecompass/
└── Tutorials/
```

其中：

- `src/`：NicheCompass代码
- `Tutorials/`：教程 notebook 和示例分析流程
- `data/`：元数据信息
- `README.md`：项目说明文档



# 环境配置

TuMeNiche 同时依赖 Python 环境和 R 环境。其中，NicheCompass 主要用于图表示学习和空间生态位识别，SpaCET 主要用于空间转录组数据的细胞类型反卷积和恶性程度估计。

由于 NicheCompass 和 SpaCET 依赖环境不同，建议分别创建两个独立环境：

```text
tume_nichecompass：用于运行 NicheCompass 和 TuMeNiche 主流程
tume_spacet：用于运行 SpaCET 反卷积分析
```

------

# 1. NicheCompass 环境配置

NicheCompass 用于构建空间邻域图、学习空间表示，并辅助识别空间生态位。

## 创建 Conda 环境

```bash
conda create -n tume_nichecompass python=3.9
conda activate tume_nichecompass
```

## 安装基础 Python 依赖

```bash
pip install numpy pandas scipy scikit-learn matplotlib seaborn
pip install scanpy anndata squidpy
pip install torch torchvision torchaudio
```

## 安装 NicheCompass

推荐按照 NicheCompass 官方说明进行安装。

如果可以直接通过 pip 安装：

```bash
pip install nichecompass
```

如果需要从源码安装：

```bash
git clone https://github.com/Lotfollahi-lab/nichecompass.git
cd nichecompass
pip install -e .
```

## TuMeNiche 常用 Python 包

TuMeNiche 分析流程中常用的 Python 包包括：

```bash
pip install numpy
pip install pandas
pip install scipy
pip install scikit-learn
pip install matplotlib
pip install seaborn
pip install scanpy
pip install anndata
pip install squidpy
pip install networkx
pip install tqdm
pip install statsmodels
```

------

# 2. SpaCET 环境配置

SpaCET 用于空间转录组数据的细胞类型反卷积，并可用于估计恶性细胞、免疫细胞和基质细胞等成分。

SpaCET 是基于 R 的工具，因此建议单独创建 R 环境。

## 创建 R 环境

```bash
conda create -n tume_spacet r-base=4.2
conda activate tume_spacet
```

## 安装 R 依赖包

进入 R：

```bash
R
```

然后安装常用依赖：

```r
install.packages(c(
  "devtools",
  "remotes",
  "Matrix",
  "ggplot2",
  "dplyr",
  "data.table",
  "Seurat",
  "patchwork"
))
```

## 安装 SpaCET

推荐按照 SpaCET 官方说明进行安装。

通常可以使用：

```r
devtools::install_github("data2intelligence/SpaCET")
```

或者：

```r
remotes::install_github("data2intelligence/SpaCET")
```

## 检查 SpaCET 是否安装成功

```r
library(SpaCET)
```

如果没有报错，说明 SpaCET 已经安装成功。

