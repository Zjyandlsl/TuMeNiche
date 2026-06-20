`src/nichecompass` 目录是 NicheCompass 模型的核心底层代码库，整体架构遵循典型的 PyTorch / PyTorch Geometric (PyG) 深度学习项目规范。为了方便理解如何在此基础上进行定制化开发，以下按模块对各个文件及目录的含义和作用进行详细拆解：

 1. 核心模型接口层 (`src/nichecompass/models/`)
这个目录定义了用户直接调用的高层 API。
 `nichecompass.py`: 这是整个包的主干入口，定义了 `NicheCompass` 类。它负责接收 AnnData 对象，初始化底层的神经网络模块，并提供 `train()`、`get_latent_representation()` 等面向用户的方法。在引入额外的节点特征（如通路活性评分、代谢酶表达量）时，通常需要首先在这里修改参数的接收与传递逻辑。
 `basemodelmixin.py` / `utils.py`: 包含模型的基础混入类（Mixin）和通用工具，通常负责模型的保存、加载（I/O 操作）以及权重的初始化管理。

 2. 网络架构抽象层 (`src/nichecompass/modules/`)
这里定义了图深度学习模型的整体拓扑结构，主要围绕变分图自编码器（VGAE）展开。
 `vgpgae.py`: 核心模块文件，定义了完整的变分图预测自编码器（Variational Graph Predictive Autoencoder）的计算图流向。它负责将编码器（Encoder）和解码器（Decoder）拼接在一起，并执行前向传播（Forward Pass）。
 `losses.py`: 极其关键的组件，定义了模型优化所需的损失函数（如 ELBO 损失、重构损失、KL 散度）。如果要引入额外的生物学因果约束（例如“驱动-执行”双重验证机制），通过自定义正则化惩罚项来剔除假阳性信号，相关逻辑需要在此处实现。
 `vgaemodulemixin.py` / `basemodulemixin.py`: 提供变分自编码器标准操作的底层封装，如重参数化技巧（Reparameterization Trick）。

 3. 神经网络算子层 (`src/nichecompass/nn/`)
这个目录包含了构成模块的具体深度学习算子和图卷积层，是重构信息传递机制的核心区域。
 `encoders.py` / `decoders.py`: 定义了对节点特征进行降维或重构的具体网络块（如多层感知机 MLP 或图卷积网络 GCN）。
 `layers.py` / `layercomponents.py`: 定义了底层的自定义神经网络层。
 `aggregators.py`: 定义了图神经网络的聚合机制（Message Passing 中的 Aggregation 阶段）。若要彻底重定义图的边权重，不再单纯依赖自学习的图注意力机制，而是强制注入基于有向通讯强度公式（Sender特征 × Receiver特征 × 距离衰减因子）的因果约束，必须深入 `aggregators.py` 和 `layers.py` 修改图张量的聚合逻辑。

 4. 数据管道层 (`src/nichecompass/data/`)
该模块负责将原始的单细胞/空间转录组数据转化为可供 PyTorch 训练的张量格式。
 `datasets.py`: 继承自 PyTorch 的 `Dataset` 类，负责定义单次获取数据的逻辑。
 `dataloaders.py`: 负责数据的批处理（Batching），针对图数据结构进行特定的采样（如邻居采样）和组装，以支持大规模图的训练。
 `dataprocessors.py`: 执行预处理步骤。在特征工程阶段，将外部计算得到的新型 Metadata（如通路激活程度、靶向基因打分）转化为模型可读的节点特征矩阵（Node Features）或边索引（Edge Index），主要由这里的脚本负责调度。
 `datareaders.py`: 处理从各类文件（如 `.h5ad`）中精准提取对应稀疏矩阵或坐标信息的逻辑。

 5. 训练与优化器引擎 (`src/nichecompass/train/`)
负责控制模型的训练生命周期。
 `trainer.py`: 定义了核心的 `Trainer` 类，包含完整的训练循环（Epoch Loop），负责执行梯度清零、反向传播（Backward Pass）、优化器更新（Optimizer Step）等标准化流程。
 `metrics.py`: 记录并追踪训练过程中的实时评估指标（如 Loss 下降趋势、准确率等）。
 `basetrainermixin.py` / `utils.py`: 提供早停机制（Early Stopping）、学习率衰减（LR Scheduling）等训练辅助工具。

 6. 实用工具与生物学先验 (`src/nichecompass/utils/`)
 `gene_programs.py`: 这是 NicheCompass 的一大特色文件，专门用于处理生物学先验知识库。它负责解析配体-受体数据库或代谢物-传感器网络，将这些生物学关系映射为模型中的“基因程序（Gene Programs）”。引入诸如 EGFR-PI3K/Akt 通路及特定代谢通讯“四元组”时，需要在此处扩展先验字典的解析逻辑。
 `analysis.py`: 提供下游分析函数，如聚类后各个微环境生态位的统计、可视化准备，或者网络拓扑特性的计算（如中心性评估）。
 `multimodal_mapping.py`: 处理多模态数据的对齐与映射工具。

 7. 性能基准测试 (`src/nichecompass/benchmarking/`)
用于评估模型表现的脚本集合，包含各类学术界常用的空间或单细胞评测指标（Metrics）。
 `metrics.py` / `utils.py`: 评测框架通用工具。
 `cas.py` / `clisis.py` / `nasw.py` / 等: 代表不同的具体评估指标算法，如 NASW（Niche Average Silhouette Width，生态位平均轮廓系数）用于衡量空间生态位聚类的纯度与分离度；其他文件则分别对应连通性、细胞类型关联等维度的打分系统。