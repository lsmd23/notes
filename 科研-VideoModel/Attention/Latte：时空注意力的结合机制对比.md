- 原文：[[Latte：Latent Diffusion Transformer for Video Generation.pdf]]
# 文献内容

## 1. 解决的问题与研究背景
- 相比于图像生成，生成高质量视频面临重大挑战，这主要归因于视频复杂且高维的特性，其高分辨率帧中蕴含着复杂的时空信息 。
- 过去在潜在扩散模型（LDMs）中，基于CNN的U-Net一直是主导架构 。尽管DiT证明了Transformer在图像扩散模型中表现优异，但基于Transformer的潜在扩散模型在视频生成领域的应用尚未得到充分探索 。
- 本文提出了Latte（潜在扩散Transformer），并探究了Transformer基视频扩散模型的最佳实践（Best Practices），以解答时空模块解耦程度等关键设计问题 。
## 2. 前置工作
- **潜在扩散模型（LDMs）**：通过在潜在空间而非像素空间中执行扩散过程来提高计算效率 。首先使用预训练变分自编码器（VAE）的编码器 $\mathcal{E}$ 将输入数据 $x$ 压缩为低维潜在代码 $z=\mathcal{E}(x)$ 。
- **Transformer 与 DiT**：Latte的构建很大程度上受到了视觉Transformer（ViT）和DiT（Diffusion Transformer）的启发，DiT证明了U-Net的归纳偏置对于LDMs并非不可或缺 。
## 3. 核心数学推导
- **扩散前向过程**：扩散过程在T个阶段的马尔可夫链中向潜在代码 $z$ 逐步引入高斯噪声，生成扰动样本 $z_{t}=\sqrt{\overline{\alpha}_{t}}z+\sqrt{1-\overline{\alpha}_{t}}\epsilon$ 。
    - 其中 $t$ 代表扩散时间步 。
    - $\overline{\alpha}_{t}$ 充当噪声调度器 。
    - $\epsilon\sim\mathcal{N}(0,1)$ 代表采样噪声 。
- **去噪逆向过程**：训练去噪过程以预测噪声较小的 $z_{t-1}$，公式为 $p_{\theta}(z_{t-1}|z_{t})=\mathcal{N}(\mu_{\theta}(z_{t}),\Sigma_{\theta}(z_{t}))$ 。
- **优化目标（损失函数）**：
    - 对数似然的变分下界简化为 $\mathcal{L}_{\theta}=-log~p(z_{0}|z_{1})+\sum_{t}D_{KL}((q(z_{t-1}|z_{t},z_{0})||p_{\theta}(z_{t-1}|z_{t}))$ 。
    - 简单目标函数 $\mathcal{L}_{simple} = \mathbb{E}_{z \sim p(z), \epsilon \sim \mathcal{N}(0,1), t} [||\epsilon - \epsilon_\theta(z_t, t)||^2_2]$（基于公式1与常规DDPM定义推演） 。
    - 为了使用学习到的逆过程协方差 $\Sigma_{\theta}$（由 $\epsilon_{\theta}$ 实现）来训练模型，需要优化完整的 $D_{KL}$ 项，即使用完整损失函数 $\mathcal{L}_{vlb}$。模型最终联合使用 $\mathcal{L}_{simple}$ 和 $\mathcal{L}_{vlb}$ 进行训练 。
- **S-AdaLN 机制（可扩展自适应层归一化）**：用于注入时间步或类别信息 $c$ 。
    - 基础AdaLN：$AdaLN(h,c)=\gamma_{c}LayerNorm(h)+\beta_{c}$，其中 $h$ 是Transformer块内的隐藏嵌入，$\gamma_{c}$ 和 $\beta_{c}$ 通过对 $c$ 进行线性回归得到 。
    - S-AdaLN引入了缩放因子 $\alpha_{c}$ 应用于残差连接（RCs）前：$RCs(h,c)=\alpha_{c}h+AdaLN(h,c)$ 。
## 4. Latte 模型架构与最佳实践
![[Pasted image 20260515194657.png]]
- ==**四种模型变体探索**==：
    - **Variant 1（最佳）**：交错使用专门的“空间Transformer块”和“时间Transformer块” 。
    - **Variant 2**：采用后期融合，前半部分全是空间块，后半部分全是时间块 。
    - **Variant 3**：在同一个Transformer块内，先进行空间多头注意力计算，再进行时间多头注意力计算 。
    - **Variant 4**：将多头注意力拆分，并行处理空间和时间维度，最后将特征相加融合 。
- **总结的最佳实践（Best Practices）**：
    - **模型架构**：==Variant 1 在各项配置中表现最好==，因其有效地解耦并交替处理时空信息 。
    - **视频片段Patch嵌入**：统一帧块嵌入（Uniform frame patch embedding）优于压缩帧块嵌入，后者容易丢失时空信号导致模型难以学习视频分布 。
    - **信息注入**：S-AdaLN（自适应层归一化）效果显著优于将时间步作为Token传入（All tokens），因为它能以更具适应性的方式将信息传播到整个网络，促进模型收敛 。
    - **位置编码**：绝对位置编码（Absolute position embedding）表现略优于相对位置编码（ROPE） 。
    - **学习策略**：图像-视频联合训练（Image-video joint training）能够增加训练批次中的样本多样性，显著提高生成视频的FVD和FID分数 。
## 5. 实验结果与关键启示
- **性能表现**：Latte 在四个标准视频生成基准测试（FaceForensics, SkyTimelapse, UCF101, Taichi-HD）上实现了最先进的（SOTA）性能，FVD和FID指标大幅领先此前基线模型 。
- **扩展性**：在文本到视频（T2V）生成任务上，Latte 取得了与近期主流 T2V 模型（如 VideoCrafter, CogVideo 等）相媲美的竞争力 。
- **模型缩放定律**：类似于图像生成领域的结论，增加 Latte 的模型参数量（从小规模的 Latte-S 32.48M 到超大规模的 Latte-XL 673.68M）能够显著提升模型性能 。
- **关于时空耦合深度的深刻洞见**：
    - 连续堆叠过多的空间注意力模块（如 Variant 2）会导致帧间余弦相似度不断下降且不均匀，严重破坏视频的连贯性 。
    - 直接将空间特征和时间特征通过并行结构简单相加（如 Variant 4）表现极差，因为这两种注意力输出的帧间特征分布差异巨大，直接融合会导致特征不匹配并破坏时间信息 。