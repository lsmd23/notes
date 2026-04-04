- 原文：[[Scaling Rectified Flow Transformers for High-Resolution Image Synthesis.pdf]]
# 文献内容
## 解决的问题
本文致力于解决高分辨率文本到图像生成领域中，如何通过改进前向过程（Forward Process）和模型架构来提升生成质量与采样效率的问题。尽管Rectified Flow（RF）理论上可以通过直线连接数据与噪声分布，从而减少采样过程中的误差积累并加速采样，但它在实际的大规模生成任务中尚未稳定优于传统的扩散模型。同时，现有的基于交叉注意力（Cross-Attention）的文本条件注入方式并非最优，限制了模型对复杂文本的理解与空间推理能力。
## 核心数学推导与前置工作
- **流模型的建模和训练**：
	- 生成模型旨在通过常微分方程（ODE）建立从噪声分布$p_1$中的样本$x_1$到数据分布$p_0$中的样本$x_0$的映射：	$$dy_t = v_\Theta(y_t, t)dt$$其中，$v_\Theta$是神经网络参数化预测的速度场。为了高效训练，通过定义一个前向过程来直接回归生成概率路径的向量场：	$$z_t = a_t x_0 + b_t \epsilon \quad \text{where} \quad \epsilon \sim \mathcal{N}(0, I)$$为了建立$z_t$、$x_0$与$\epsilon$的关系，定义条件向量场$u_t(z|\epsilon) := \psi_t'(\psi_t^{-1}(z|\epsilon)|\epsilon)$，用于生成边缘概率路径的边缘向量场$u_t(z)$为： $$u_t(z) = \mathbb{E}_{\epsilon \sim \mathcal{N}(0, I)} u_t(z|\epsilon) \frac{p_t(z|\epsilon)}{p_t(z)}$$由于直接回归的边缘化计算困难，文献使用了等价的条件流匹配（CFM）目标： $$\mathcal{L}_{CFM} = \mathbb{E}_{t, p_t(z|\epsilon), p(\epsilon)} ||v_\Theta(z,t) - u_t(z|\epsilon)||_2^2$$结合信噪比（SNR）$\lambda_t := \log\frac{a_t^2}{b_t^2}$，可将条件向量场重参数化为噪声预测形式： $$\mathcal{L}_{CFM} = \mathbb{E}_{t, p_t(z|\epsilon), p(\epsilon)} \left(-\frac{b_t}{2}\lambda_t'\right)^2 ||\epsilon_\Theta(z,t) - \epsilon||_2^2$$
- **流轨迹的分析与设计**：
	- 为了统一分析不同的扩散模型公式与Rectified Flow，文献引入了时间相关的权重$w_t$，将目标函数写为统一的噪声预测形式：$$\mathcal{L}_w(x_0) = -\frac{1}{2}\mathbb{E}_{t \sim \mathcal{U}(t), \epsilon \sim \mathcal{N}(0,I)} [w_t \lambda'_t ||\epsilon_\Theta(z_t, t) - \epsilon||^2]$$其中，$\lambda_t = \log\frac{a_t^2}{b_t^2}$为信噪比（SNR），当权重$w_t = -\frac{1}{2}\lambda'_t b_t^2$时，该目标函数完全等价于条件流匹配（CFM）目标$\mathcal{L}_{CFM}$。基于此统一框架，文献重点探讨并对比了以下三种主要的流轨迹设计：
	1. **Rectified Flow (RF)**：RF定义了数据分布和标准正态分布之间的直线前向过程： $$z_t = (1-t)x_0 + t\epsilon$$ 在此定义下，当使用$\mathcal{L}_{CFM}$目标函数时，其对应的权重为： $$w_t^{RF} = \frac{t}{1-t}$$ 在该设定下，网络输出直接参数化预测速度$v_\Theta$。
		- 参考：[[Rectified Flow：校正直线流生成]]
	2. **EDM (Elucidating the Design Space)**：EDM使用的前向过程形式为：$$z_t = x_0 + b_t\epsilon$$其中噪声系数$b_t = \exp(F_N^{-1}(t | P_m, P_s^2))$，$F_N^{-1}$代表均值为$P_m$、方差为$P_s^2$的正态分布分位数函数。这种设定使得当$t \sim \mathcal{U}(0,1)$时，信噪比自然满足正态分布：$$\lambda_t \sim \mathcal{N}(-2P_m, (2P_s)^2)$$网络通过F-预测（F-prediction）进行参数化，其对应的损失权重可以写为：$$w_t^{EDM} = \mathcal{N}(\lambda_t | -2P_m, (2P_s)^2) (e^{\lambda_t} + 0.5^2)$$
	3. **Cosine Schedule**：Cosine调度定义的曲线前向过程为：$$z_t = \cos\left(\frac{\pi}{2}t\right)x_0 + \sin\left(\frac{\pi}{2}t\right)\epsilon$$其损失权重根据网络参数化的方式不同而有所区别：
		- 当结合$\epsilon$-参数化及对应损失时，等效权重为：$w_t = \text{sech}(\lambda_t / 2)$
		- 当结合$v$-预测（v-prediction）损失时，等效权重为：$w_t = e^{-\lambda_t / 2}$
- **SNR采样器的定制设计**：针对上述三种前向过程设计，文献提出了三种定制的采样分布，以进一步提升采样效率和生成质量：
	1. **Logit-Normal Sampling**：增加中间时间步的采样权重： $$\pi_{ln}(t; m, s) = \frac{1}{s\sqrt{2\pi}} \frac{1}{t(1-t)} \exp\left(-\frac{(\text{logit}(t) - m)^2}{2s^2}\right)$$ 其中$\text{logit}(t) = \log\frac{t}{1-t}$，$m$为位置参数（负值偏向数据，正值偏向噪声），$s$为尺度参数。 
	2. **Mode Sampling with Heavy Tails**：由于Logit-normal在端点0和1处密度消失，为研究其潜在负面影响，引入在$[0, 1]$上严格为正密度的采样分布。首先定义函数$f_{mode}$（其中$u \sim \mathcal{U}(0,1)$）： $$f_{mode}(u; s) = 1 - u - s \cdot \left(\cos^2\left(\frac{\pi}{2}u\right) - 1 + u\right)$$ 当尺度参数$s \in [-1, \frac{2}{\pi-2}]$时，该函数单调。其隐含密度为： $$\pi_{mode}(t; s) = \left| \frac{d}{dt}f_{mode}^{-1}(t) \right|$$ $s$为正时偏向中点，$s$为负时偏向端点，$s=0$时退化为均匀分布$\mathcal{U}(t)$。 
	3. **CosMap Sampling**：将Nichol等人的余弦调度适配到RF框架。寻找映射$t = f(u)$，使其log-SNR匹配余弦调度的log-SNR： $$2 \log\frac{\cos(\frac{\pi}{2}u)}{\sin(\frac{\pi}{2}u)} = 2 \log\frac{1-f(u)}{f(u)}$$ 解得$t = f(u) = 1 - \frac{1}{\tan(\frac{\pi}{2}u) + 1}$。求导得到其采样密度： $$\pi_{CosMap}(t) = \left| \frac{d}{dt}f^{-1}(t) \right| = \frac{2}{\pi - 2\pi t + 2\pi t^2}$$
## 对应的新架构 (MM-DiT)
文献提出了一种多模态扩散Transformer (MM-DiT)架构，为文本和图像模态使用独立的权重网络，在各自的表征空间内处理信息后，再通过序列拼接进行联合注意力计算。
![[Pasted image 20260404194825.png]]
原文架构图：![[Pasted image 20260404200007.png]]
解释：
- 文本编码：Caption输入三个文本编码器：
    - **CLIP-G/14文本编码器**
    - **CLIP-L/14文本编码器**
	    - 这两个编码器更偏重视觉语义对齐，因而被接入后续的pooled信息中，用于在全局调制中提供语义引导
    - **T5-XXL文本编码器**
	    - 该编码器更偏重语言理解，因而被接入序列条件向量中，用于在局部调制中提供语言细节指导
- 文本编码处理：一部分token形成池化信息（pooled），经过MLP和时间步编码，形成全局调制向量$y$；另一部分token经过简单线性层，形成序列条件向量$c$
- 图像编码处理：
	- 借助[[LDM：潜在扩散模型|LDM]]的思路，在加噪隐变量空间上进行处理，因而隐变量才是其输入
	- 进行patching，将其分成多个块，每个块被视为一个token输入Transformer
- MM-DiT块：
	- 由于文本和图像的token嵌入应当是高度不一致的，因而不能简单的进行concat操作。这里采用两条数据通路，借助交叉注意力机制进行多模态数据的融合
	- **Attention**：两种数据在这一步进行concat，之后共同参与注意力计算，以使得图像和文本token都能对彼此进行信息交互
	- **modulation**：这部分数据源自上述的全局调制向量$y$，通过**SiLU**激活函数后被分成数个部分，产生一组控制参数，控制每个block内部的归一化，整合，残差连接等操作
		- 此思路源自于[[DiT：扩散Transformer模型]]的建模思路
## 实验结果与启示
- **最优采样设计确立**：Logit-Normal时间步采样（特别是均值为0，尺度为1的参数配置）在CLIP分数和FID等生成指标上，一致且显著地优于均匀RF以及EDM、LDM等主流传统扩散调度的基线。
- **高频信息与高维Latent的价值**：提升预训练自编码器的潜在通道数（从4或8提升至16）加大了预测难度，但显著提升了重建质量的理论上限，并在拥有足够容量的Transformer骨干网络下兑现了更好的图像生成表现。
- **分离且联合的架构优势**：MM-DiT架构（双流设计）避免了早期合并文本和视觉特征造成的表征干涉，在验证损失收敛速度和质量上大幅度战胜了标准DiT和UViT网络。
- **数据质量的正向反馈**：利用现代VLM（CogVLM）生成的合成描述与原始数据以50/50混合后，有效改善了传统数据集描述简单、遗漏背景与构图细节的缺陷，极大增强了模型的Prompt Follow能力。
- **可预测的Scaling Law与推理冗余度**：在扩展至8B参数后，网络依然没有出现饱和迹象；此外，大型模型不仅本身质量高，且在少步采样（例如5步）时的表现衰减极其微小（轨迹更接近理想直线）。而在推理时即使将T5 XXL屏蔽，对于基础Prompt也几乎不损耗画质，证明了其部署的灵活性。
# 关键启示
- 多模态数据引导或许是训练时可以采用的idea
- 如果要继续使用这个架构，如何使用guidance性的数据很重要，MM-DiT中就使用了新型编码，以利用这里的文本模态数据