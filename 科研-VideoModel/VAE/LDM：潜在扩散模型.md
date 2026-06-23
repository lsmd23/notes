- 原文：[[High-Resolution Image Synthesis with Latent Diffusion Models.pdf]]
# 文献内容
## 1. 前置背景与核心挑战
- **扩散模型 (Diffusion Models, DM)**：通过逆转高斯噪声的过程来学习数据分布 。虽然在图像合成上达到了 SOTA，但由于在 **像素空间 (Pixel Space)** 运行，训练和推理极其昂贵（常需数百个 GPU 天） 。
- **压缩权衡**：似然模型（如 DM）的学习通常分为两阶段：
    1. **感知压缩 (Perceptual Compression)**：去除高频细节，学习很少的语义信息。
    2. **语义压缩 (Semantic Compression)**：学习数据的语义和概念组成。
- **解决问题**：LDM 旨在将 DM 的训练从像素空间转移到 **潜在空间 (Latent Space)**，在保留视觉忠实度的同时大幅降低计算成本 。
## 2. 核心架构设计
LDM 将生成过程分解为：**第一阶段自编码器 (Autoencoder)** 提供潜在空间，**第二阶段扩散模型 (DM)** 在该空间内学习分布 。
### 2.1 系统架构图 (Mermaid)
![[Pasted image 20260331135806.png]]
### 2.2 核心组件功能说明
1. **Encoder ($\mathcal{E}$)**：将 RGB 图像 $x \in \mathbb{R}^{H \times W \times 3}$ 降采样到低维潜在空间 $z \in \mathbb{R}^{h \times w \times c}$，降采样因子 $f=H/h=W/w$ 。
2. **Decoder ($\mathcal{D}$)**：从潜在表示中解码重建图像。实验表明 $f=4$ 到 $16$ 能在效率与画质间取得平衡 。
3. **Denoising UNet ($\epsilon_{\theta}$)**：在潜在空间处理噪声。利用 2D 卷积层的归纳偏置处理空间结构数据 。
4. **Cross-Attention 机制**：通过交叉注意力层处理各类模态调节输入 $y$ 。
## 3. 关键数学推导与训练目标
### 3.1 潜在扩散目标函数
LDM 的核心训练目标是在潜在空间最小化噪声预测误差：
$$L_{LDM} := \mathbb{E}_{\mathcal{E}(x), \epsilon \sim \mathcal{N}(0,1), t} \left[ \| \epsilon - \epsilon_{\theta}(z_t, t) \|_2^2 \right]$$
- **$z_t$**：由编码后的潜在表示 $z$ 在时间步 $t$ 加入噪声得到。
- **$\epsilon$**：实际加入的噪声，服从标准正态分布 $\mathcal{N}(0,1)$。
- **$\epsilon_{\theta}(z_t, t)$**：UNet 模型预测的噪声。
### 3.2 引入条件调节 (Conditioning)
为了实现文本到图像等控制，加入调节因子 $y$ 和编码器 $\tau_{\theta}$：
$$L_{LDM} := \mathbb{E}_{\mathcal{E}(x), y, \epsilon \sim \mathcal{N}(0,1), t} \left[ \| \epsilon - \epsilon_{\theta}(z_t, t, \tau_{\theta}(y)) \|_2^2 \right]$$
**Cross-Attention 计算过程**： UNet 的中间表示 $\varphi_i(z_t)$ 作为查询 (Query)，调节输入作为键 (Key) 和值 (Value) ：
- $Q = W_Q^{(i)} \cdot \varphi_i(z_t)$
- $K = W_K^{(i)} \cdot \tau_{\theta}(y)$
- $V = W_V^{(i)} \cdot \tau_{\theta}(y)$
- $\text{Attention}(Q,K,V) = \text{softmax}\left(\frac{QK^T}{\sqrt{d}}\right) \cdot V$
## 4. 实验结果与启示
### 4.1 主要实验结论
- **效率提升**：相比像素级 DM，LDM-4 到 LDM-8 在训练和推理速度上大幅领先（采样吞吐量提升显著），且 FID 得分更优 。
- **任务通用性**：LDM 在多项任务中达到或接近 SOTA：
    - **无条件生成**：在 CelebA-HQ 上达到 5.11 的新 SOTA FID 。
    - **修复 (Inpainting)**：比 LaMa 模型更快，且生成结果更具多样性 。
    - **超分辨率 (Super-Resolution)**：在 FID 性能上超过了 SR3 。
    - **文本到图像**：通过交叉注意力，模型能理解复杂的文本提示 (Text Prompts) 。
### 4.2 关键启示
1. **压缩因子的选择 (f)**：过小的 $f$ 导致训练慢；过大的 $f$（如 $f=32$）由于第一阶段信息损失太严重，会导致图像质量停滞 。
2. **两阶段分离的优势**：先训练通用的感知压缩模型（第一阶段），可以复用于多个不同的 DM 训练，极大地节省了计算资源 。
3. **全卷积推理**：对于空间调节任务（如语义合成），LDM 可以以卷积方式生成超过训练分辨率（如 $1024^2$）的大图 。
## 5. 局限性与社会影响
- **采样速度**：虽然比像素 DM 快，但作为序列采样模型，仍慢于 GAN 。
- **重构瓶颈**：在需要极高精度的像素级任务中，自编码器的重构损失可能成为上限瓶颈 。
- **社会影响**：降低了创作门槛，但也增加了“深度伪造 (Deepfakes)”和滥用敏感数据的风险 。
# 关键启示
- 在扩散模型中，对复杂的数据可以进行**适度压缩**，采用合适的编码解码结构，可以**优化训练复杂度**，同时可以提高生成的**分辨率**和**质量**。
	- 压缩的比例或许需要实验来验证和调整
- 文献在**latent**上做训练和生成，而不是直接在像素空间上进行训练和生成，这样可以**降低计算资源的需求**，同时保持生成图像的质量。
    - 这种方法可能适用于其他类型的数据，如音频、文本等
- *对气象场建模，是否需要对不同维度的预测进行不同维度的压缩，还是一个简单的先后顺序可以解决的问题？*