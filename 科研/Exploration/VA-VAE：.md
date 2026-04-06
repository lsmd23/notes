- 原文：[[Reconstruction vs. Generation：Taming Optimization Dilemma in Latent Diffusion Models.pdf]]
# 文献内容
## 一、 必要的前置工作
隐扩散模型（Latent Diffusion Models, LDM）通过连续值的变分自编码器（VAE）或视觉分词器（Visual Tokenizer）将高分辨率视觉信号压缩至潜在空间，从而大幅降低计算成本。在这一架构中，视觉分词器的压缩和重建能力对于生成系统的整体有效性至关重要。
## 二、 解决的问题（优化困境）
研究发现，在LDM的两阶段设计中存在“重建与生成”的优化困境：虽然增加视觉分词器的特征维度可以提升重建精度，但这会导致生成性能显著下降。当前的妥协方案往往表现不佳：要么为了强行训练高维特征而急剧增加扩散模型参数量和计算成本，要么为了生成模型易于收敛而故意限制分词器的重建能力（导致生成图像存在伪影和细节丢失）。论文指出，这一困境的根本原因在于模型极难从零开始学习并拟合一个无约束的高维潜在空间，导致高维特征分布不均匀。
## 三、 核心架构建模与机制
为了解决上述困境，论文提出了VA-VAE（Vision foundation model Aligned Variational AutoEncoder），即视觉基础模型对齐的变分自编码器。
![[Pasted image 20260405214640.png]]
## 四、 关键数学推导与损失函数设计

论文提出的核心是视觉基础模型对齐损失（VF Loss），作为即插即用的模块在训练分词器时使用，它包含以下数学推导： **1. 特征维度投影映射** 首先通过线性变换将基础模型的特征 $F$ 投影到与图像潜在特征 $Z$ 相同的维度：

$$F^{\prime}=WF+b$$

其中，$Z$ 为视觉分词器编码出的图像潜在特征，$F$ 为基础模型提取的视觉表示，$W \in \mathbb{R}^{d_{z} \times d_{f}}$ 和 $b \in \mathbb{R}^{d_{z}}$ 分别为权重矩阵和偏置项，得到的 $F^{\prime} \in \mathbb{R}^{d_{z}}$ 用于后续对齐计算。 **2. 边缘余弦相似度损失 (Marginal Cosine Similarity Loss)** 该损失用于强制执行点对点（全局与局部）的绝对结构对齐，缩小空间位置 $(i, j)$ 上对应特征的相似度差距：

$$\mathcal{L}_{mcos}=\frac{1}{h \times w} \sum_{i=1}^{h} \sum_{j=1}^{w} ReLU(1-m_{1}-\frac{z_{ij} \cdot f_{ij}^{\prime}}{||z_{ij}|| ||f_{ij}^{\prime}||})$$

其中，$h \times w$ 是特征网格的空间维度，$z_{ij}$ 和 $f_{ij}^{\prime}$ 代表各位置的特征向量，$m_{1}$ 为设定的边缘裕量（Margin），用于提供对齐的灵活性以防过度正则化（仅当特征相似度低于 $1-m_{1}$ 时才产生损失）。 **3. 边缘距离矩阵相似度损失 (Marginal Distance Matrix Similarity Loss)** 作为点对点绝对对齐的补充，该损失用于促使特征内部的相对分布距离矩阵尽可能相似：

$$\mathcal{L}_{mdms}=\frac{1}{N^{2}} \sum_{i,j} ReLU(|\frac{z_{i} \cdot z_{j}}{||z_{i}|| ||z_{j}||}-\frac{f_{i} \cdot f_{j}}{||f_{i}|| ||f_{j}||}|-m_{2})$$

其中，$N=h \times w$ 是展平后特征图的元素总数，$m_{2}$ 为放宽约束的裕量（Margin），仅当两个点对特征在不同空间下的余弦相似度绝对差值大于 $m_{2}$ 时才计入惩罚损失。 **4. 自适应权重调节 (Adaptive Weighting)** 为平衡不同量级的重建损失与VF对齐损失，在反向传播前基于编码器最后一层卷积的梯度计算自适应权重：

$$w_{adaptive}=\frac{||\nabla_{L_{L}} \mathcal{L}_{rec}||}{||\nabla_{L_{L}} \mathcal{L}_{vf}||}$$

最终结合超参数 $w_{hyper}$ 得到整体的VF Loss：

$$\mathcal{L}_{vf}=w_{hyper} * w_{adaptive}(\mathcal{L}_{mcos}+\mathcal{L}_{mdms})$$

## 五、 实验结果与启示

**1. 实验核心结果** 结合经过优化策略增强的扩散Transformer基线（LightningDiT），该集成系统在ImageNet 256x256图像生成任务上取得了SOTA的性能表现（FID得分为1.35）。在引入VF Loss后，LightningDiT在使用高维分词器时的训练速度提升了超过2.5倍；整个系统在仅仅64个Epoch内即达到了2.11的FID，相较于原始的DiT实现了超过21倍的惊人收敛加速。此外，DINOv2和MAE等自监督视觉基础模型在引导时展现出了最佳性能，在保持极高重建保真度的同时最大化改善了生成指标。 
**2. 重要启示** 解决高维分词器生成质量瓶颈的关键无需一味地强行扩大扩散模型的参数规模，而是应当从源头上优化底层潜在空间的特征分布。借助强大预训练视觉基础模型的先验表示能力来正则化和引导VAE的潜在空间，能够显著提升高维特征分布的均匀度，这也是打破“重建与生成”零和博弈的最有效途径。通过纯粹的架构改进与现代训练策略的整合（如Rectified Flow、Logit Normal Sampling等），证明了基础的DiT架构即便不引入复杂的掩码图像建模（MIM）依然具备释放极致性能和超快收敛速度的潜力。