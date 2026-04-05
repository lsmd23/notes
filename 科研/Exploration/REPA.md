- 原文：[[Representation Alignment for Generation：Training Diffusion Transformers Is Easier Than You Think.pdf]]
# 文献内容
本文提出了一种名为REPA（REPresentation Alignment）的正则化方法，旨在通过将扩散模型（如DiT、SiT）的隐藏状态与外部预训练的视觉自监督表示进行对齐，来极大提高生成扩散模型的训练效率和生成质量。
## 1. 必要的前置工作
扩散模型和流匹配模型可以通过随机插值（Stochastic Interpolants）的统一视角进行描述。
- 考虑一个连续的时间依赖过程，包含数据 $x_*\sim p(x)$ 和高斯噪声 $\epsilon\sim\mathcal{N}(0,I)$，时间 $t\in[0,T]$：$$x_t=\alpha_tx_*+\sigma_t\epsilon$$其中 $\alpha_0=\sigma_T=1$，$\alpha_T=\sigma_0=0$。这里 $\alpha_t$ 和 $\sigma_t$ 分别是 $t$ 的递减和递增函数。
- 存在一个概率流常微分方程（PF ODE），其速度场（velocity field）为 $\dot{x}_t=v(x_t,t)$。
- 速度模型 $v_\theta(x_t,t)$ 的训练目标是最小化以下速度预测误差：$$\mathcal{L}_{velocity}(\theta):=\mathbb{E}_{x_*,\epsilon,t}[||v_\theta(x_t,t)-\dot{\alpha}_tx_*-\dot{\sigma}_t\epsilon||^2]$$
- 对于SiT模型，常采用线性插值（Linear Interpolant），即设定 $T=1$，$\alpha_t=1-t$ 且 $\sigma_t=t$。
## 2. 解决的核心问题
- **语义差距（Semantic Gap）**：虽然近期的扩散模型能够在其隐藏状态中学习到具有判别性的特征，但这种内部表示的质量仍然远远落后于最先进的自监督学习（SSL）方法（如DINOv2）。
- **对齐微弱**：扩散模型学习到的表示与DINOv2的表示之间的对齐程度很弱。
- **训练瓶颈**：作者假设，大规模生成式扩散模型训练的主要瓶颈在于如何有效地学习这些高质量表示。仅仅依赖扩散模型本身去从零开始隐式地学习表示是低效的。
## 3. 对应的新架构 (REPA: REPresentation Alignment)
为了解决上述问题，作者引入了外部高质量的视觉表示来指导模型的训练过程。
![[Pasted image 20260405211521.png]]
### 关键数学推导与损失函数
REPA通过最大化预训练表示和扩散模型隐藏状态之间的Patch级别相似度来实现对齐。
- 设 $y_*=f(x_*)\in\mathbb{R}^{N\times D}$ 为干净图像 $x_*$ 通过预训练编码器 $f$ 的输出表示，其中 $N$ 是Patch数量，$D$ 是嵌入维度。
- $h_t=f_\theta(z_t)$ 为扩散Transformer编码器在给定加噪输入下的隐藏状态，通过可训练投影头（MLP）得到投影表示 $h_\phi(h_t)\in\mathbb{R}^{N\times L}$。
- REPA损失函数定义如下：$$\mathcal{L}_{REPA}(\theta,\phi):=-\mathbb{E}_{x_*,\epsilon,t}\left[\frac{1}{N}\sum_{n=1}^N sim\left(y_*^{[n]},h_\phi(h_t^{[n]})\right)\right]$$
    其中 $n$ 是Patch索引，$sim(\cdot,\cdot)$ 是预定义的相似度函数（如负余弦相似度或NT-Xent）。
- 最终的总体训练目标是结合原有的去噪/速度预测损失与REPA正则化项：$$\mathcal{L}:=\mathcal{L}_{velocity}+\lambda\mathcal{L}_{REPA}$$
    其中 $\lambda>0$ 是控制去噪和表示对齐之间权衡的超参数。
## 4. 实验结果与启示
- **极大地提升了训练效率**：在应用于SiT模型时，REPA使得训练速度提高了17.5倍以上。在不到400K步的训练下，SiT-XL/2 + REPA即可匹配甚至超越原版SiT-XL模型7M步训练的性能（FID=7.9）。
- **生成质量突破（SOTA）**：结合无分类器引导（Classifier-free guidance）和引导区间调度策略，该方法在ImageNet基准上达到了最先进的 $FID=1.42$。
- **早期层对齐即充足**：实验表明，仅在Transformer的前几个Block（例如第8层）施加REPA正则化即可达到充分的对齐效果。这使得后续的层能够专注于在对齐的表示基础之上捕获高频细节。
- **目标表示的影响**：作为目标的预训练编码器质量越好（例如DINOv2-g），其对应的扩散Transformer的生成性能和判别性能提升越显著。
- **对不同噪声尺度的鲁棒性**：REPA在各个时间步（Timesteps/Noise scales）下都一致地缩小了扩散模型与外部视觉编码器之间的表示差距。