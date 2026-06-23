- 原文：[[REPA-E：Unlocking VAE for End-to-End Tuning with Latent Diffusion Transformers.pdf]]
# 文献内容
## 1. 必要的前置工作
- 传统的潜在扩散模型（Latent Diffusion Models, LDMs）通常采用两阶段训练法：首先使用重建损失训练变分自编码器（VAE）作为分词器；然后固定 VAE，使用扩散损失（Diffusion Loss）单独训练生成器网络（即扩散模型）。
- 近期研究表明，使用表示对齐（Representation Alignment, REPA）损失可以加速**扩散模型的生成学习**。
## 2. 其解决的问题
- **朴素的端到端（End-to-End）训练失效：** 如果直接将扩散损失反向传播给 VAE 进行端到端联合训练，会导致 VAE 的潜在空间发生坍塌（空间维度方差减小），使得扩散模型仅仅是在预测一个偏置项，这虽然简化了去噪任务，但会严重降低最终的图像生成质量。
- **潜在空间结构的次优性：** 现有的 VAE 主要针对重建误差进行优化，其潜在空间对于扩散模型的生成任务而言往往不是最优的（例如 SD-VAE 存在高频噪声，而 IN-VAE 则过度平滑）。
## 3. REPA-E 新架构与核心方法
- 核心思想是放弃使用扩散损失来更新 VAE，转而利用表示对齐（REPA）损失作为端到端训练的代理目标，联合微调 VAE 和扩散模型的特征。![[Pasted image 20260405204053.png]]
	- 插入一个Batch-Norm层，解决：
		- 传统LDM会对已经训练固定过的latent变量做归一化，但端到端训练时，latent在更新，不断归一化增大计算开销
		- 新增一个可微的归一化层，直接对潜变量进行归一化，避免了不断计算均值和方差的开销，同时保持了潜变量的稳定性
	- 保留Diffusion Loss，但只作用于Diffusion Model的参数更新（即带有Stop-Gradient，不更新 VAE 的参数）。
	- 保留一套VAE的正则化损失，确保不会因为端到端训练而损害 VAE 的重建性能，作者给出了：
		- $L_{MSE}$
		- $L_{LPIPS}$
		- $L_{GAN}$
		- $L_{KL}$​
		- 总公式：$$L_{\text{REG}}(\phi)=L_{MSE}+L_{LPIPS}+L_{GAN}+L_{KL}$$
## 4. 关键数学推导与目标函数
- **中间潜变量表示：** 在任意时间步 $t$，加噪的潜变量表示为 $z_{t}=\alpha_{t}z_{VAE}+\sigma_{t}\epsilon_{orig}$。如果直接用扩散损失优化 VAE，会使 $z_{VAE}$ 的空间方差降低，导致去噪目标退化。
- **端到端表示对齐损失 (REPA Loss)：**$$\mathcal{L}_{REPA}(\theta,\phi,\omega)=-\mathbb{E}_{x,\epsilon,t}[\frac{1}{N}\sum_{n=1}^{N}sim(y^{[n]},h_{\omega}(h_{t}^{[n]}))]$$
    - $\mathcal{V}_{\phi}$: 参数为 $\phi$ 的 VAE 模型。
    - $\mathcal{D}_{\theta}$: 参数为 $\theta$ 的扩散模型。
    - $f$: 固定的预训练感知模型（如 DINO-v2）。
    - $x$: 干净的输入图像。
    - $h_{t}$: 扩散 Transformer 的隐藏层状态输出。
    - $h_{\omega}$: 参数为 $\omega$ 的可训练投影层。
    - $y=f(x)$: 预训练感知模型提取的特征表示。
    - $N$: Patch 的数量。
    - $sim(<.,.>):$ 计算感知模型特征与投影后隐藏状态之间的 Patch 级余弦相似度。
- **VAE 正则化损失 (Regularization Loss)：** 为了维持 VAE 的重建性能，引入了正则化项，包括重建损失、对抗损失和 KL 散度：
$$\mathcal{L}_{REG}=\mathcal{L}_{KL}+\mathcal{L}_{MSE}+\mathcal{L}_{LPIPS}+\mathcal{L}_{GAN}$$
- **总体优化目标：**$$\mathcal{L}(\theta,\phi,\omega)=\mathcal{L}_{DIFF}(\theta)+\lambda\mathcal{L}_{REPA}(\theta,\phi,\omega)+\eta\mathcal{L}_{REG}(\phi)$$
    - 扩散损失 $\mathcal{L}_{DIFF}$ 仅作用于 $\theta$（即带有 Stop-Gradient，不更新 $\phi$）。
    - $\lambda$ 和 $\eta$ 为损失的权重系数。
## 5. 实验结果与得出的一些启示
- **极致的加速效果：** REPA-E 能够显著加快扩散模型的训练速度，相比于原始的 REPA 和普通的 LDM 训练方法，分别实现了超过 17 倍和 45 倍的加速。在 400K 步时，REPA-E 的 FID 即可达到 4.07。
- **刷新生成质量 SOTA：** 在 ImageNet $256\times256$ 数据集上，该方法创造了新的最优生成性能，在使用和不使用无分类器引导（CFG）的情况下，FID 分别达到了 1.12 和 1.69。
- **自动优化 VAE 潜在空间：** 端到端联合训练不仅提升了生成器，还自动改善了 VAE 本身的潜在空间结构（例如自动消除 SD-VAE 潜在空间中的高频噪声，或丰富 IN-VAE 过于平滑的特征细节）。
- **强大的泛化与迁移能力：** 经过 REPA-E 调优后的 VAE 展现出了优异的下游生成性能，甚至可以作为原始 VAE（如 SD-VAE）的直接平替（drop-in replacement），在不同的扩散模型架构（如 DiT-XL）中即插即用并带来性能提升。模型规模越大，REPA-E 带来的相对性能增益也越显著。
# 启示
- E2E的训练似乎可以更好的调整latent空间，可以使得latent中的高频噪声减少，或丰富其过于平滑的特征细节