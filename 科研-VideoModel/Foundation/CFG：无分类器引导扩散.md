- 原文：[[CLASSIFIER-FREE DIFFUSION GUIDANCE.pdf]]
# 文献内容
## 一、必要的前置工作
- **扩散模型 (Diffusion Models)**：一类通过连续时间或离散时间马尔可夫过程将数据分布映射到高斯噪声，并通过训练神经网络反转该过程以生成数据的生成模型。
- **截断采样 (Truncation Sampling)**：在生成对抗网络 (GAN) 等模型中，通过降低采样时的潜在空间方差，牺牲样本多样性来提升单一样本的保真度 (Fidelity) 和质量。直接对扩散模型应用截断采样会导致样本模糊。
- **分类器引导 (Classifier Guidance)**：为了在扩散模型中实现类似截断采样的效果，先前的方法引入一个额外训练的图像分类器 $p_\theta(c|z_\lambda)$，利用其相对于噪声输入 $z_\lambda$ 的对数概率梯度来修改扩散模型的得分估计，引导生成过程偏向特定条件 $c$。
## 二、其解决的问题
- **复杂的训练流程**：分类器引导需要额外训练一个能在各种噪声尺度下工作的分类器，无法直接复用现成的预训练分类器，增加了训练成本和工程复杂度。
- **对抗攻击嫌疑**：引入分类器梯度进行采样，本质上类似于对分类器进行基于梯度的对抗攻击，这引发了关于模型生成质量提升是否仅仅是因为“欺骗”了评估指标（如 FID、IS 等依赖分类器的指标）的疑问。
- **核心目标**：提出无分类器引导 (Classifier-Free Guidance, CFG)，探索是否能完全脱离额外分类器，仅依靠纯生成模型实现高质量的条件生成，并在保真度与多样性之间取得权衡。
## 三、无分类器引导架构图
![[Pasted image 20260402205053.png|697]]
## 四、核心数学推导与表示（加噪/后验/去噪生成）
文献采用连续时间设定，定义 $\lambda=\log(\alpha_\lambda^2/\sigma_\lambda^2)$ 为对数信噪比 (Log SNR)。前向过程向减少 $\lambda$ 的方向运行。
**1. 前向加噪过程 (Forward Process)**
扩散模型的前向过程 $q(z|x)$ 是一个方差保留 (Variance-preserving) 的马尔可夫过程：
$$q(z_\lambda|x)=\mathcal{N}(\alpha_\lambda x, \sigma_\lambda^2 I)$$
其中 $\alpha_\lambda^2=1/(1+e^{-\lambda})$ 且 $\sigma_\lambda^2=1-\alpha_\lambda^2$。对于 $\lambda<\lambda'$ 的步进转移：
$$q(z_\lambda|z_{\lambda'})=\mathcal{N}\left(\frac{\alpha_\lambda}{\alpha_{\lambda'}}z_{\lambda'}, \sigma_{\lambda|\lambda'}^2 I\right), \quad \sigma_{\lambda|\lambda'}^2=(1-e^{\lambda-\lambda'})\sigma_\lambda^2$$
**2. 给定真实数据 $x$ 的后验去噪过程 (Posterior Process)**
基于贝叶斯定理，以真实数据 $x$ 为条件的逆向过程依然是高斯分布：
$$q(z_{\lambda'}|z_\lambda,x)=\mathcal{N}(\tilde{\mu}_{\lambda'|\lambda}(z_\lambda,x), \tilde{\sigma}_{\lambda'|\lambda}^2 I)$$
其中，后验均值 $\tilde{\mu}$ 和方差 $\tilde{\sigma}^2$ 可由前向公式推导得出：
$$\tilde{\mu}_{\lambda'|\lambda}(z_\lambda,x)=e^{\lambda-\lambda'}\frac{\alpha_{\lambda'}}{\alpha_\lambda}z_\lambda+(1-e^{\lambda-\lambda'})\alpha_{\lambda'}x$$
$$\tilde{\sigma}_{\lambda'|\lambda}^2=(1-e^{\lambda-\lambda'})\sigma_{\lambda'}^2$$
**3. 模型的去噪生成过程 (Generative Reverse Process)**
生成模型通过参数化网络 $x_\theta(z_\lambda)$ 逼近真实的 $x$，将其代入上述后验分布中进行采样：
$$p_\theta(z_{\lambda'}|z_\lambda)=\mathcal{N}(\tilde{\mu}_{\lambda'|\lambda}(z_\lambda,x_\theta(z_\lambda)), (\tilde{\sigma}_{\lambda'|\lambda}^2)^{1-v}(\sigma_{\lambda|\lambda'}^2)^v)$$
模型方差是对 $\tilde{\sigma}_{\lambda'|\lambda}^2$ 和 $\sigma_{\lambda|\lambda'}^2$ 的对数空间插值（超参数 $v$）。利用预测噪声 $\epsilon_\theta(z_\lambda)$，模型重参数化为：
$$x_\theta(z_\lambda)=\frac{z_\lambda-\sigma_\lambda\epsilon_\theta(z_\lambda)}{\alpha_\lambda}$$
训练目标为匹配真实噪声（等价于去噪得分匹配）：
$$\mathbb{E}_{\epsilon,\lambda}[||\epsilon_\theta(z_\lambda) - \epsilon||_2^2]$$
预测的 $\epsilon_\theta(z_\lambda)$ 本质上估计了缩放后的数据对数密度梯度：$\epsilon_\theta(z_\lambda)\approx-\sigma_\lambda\nabla_{z_\lambda}\log p(z_\lambda)$。
**4. 无分类器引导 (Classifier-Free Guidance) 的核心推导**
利用贝叶斯规则定义一个隐式分类器 $p^i(c|z_\lambda)\propto p(z_\lambda|c)/p(z_\lambda)$。其对数梯度为：
$$\nabla_{z_\lambda}\log p^i(c|z_\lambda)=-\frac{1}{\sigma_\lambda}[\epsilon^*(z_\lambda,c)-\epsilon^*(z_\lambda)]$$
将其代入带权重的引导得分公式 $\tilde{\epsilon}_\theta = \epsilon_\theta(z_\lambda,c) - w\sigma_\lambda\nabla_{z_\lambda}\log p(c|z_\lambda)$，得到最终 CFG 的采样更新方向：

$$\tilde{\epsilon}_\theta(z_\lambda,c)=(1+w)\epsilon_\theta(z_\lambda,c)-w\epsilon_\theta(z_\lambda)$$
其中 $w$ 为==引导强度==（**计算指标时取较小值，show case时较大**），$\epsilon_\theta(z_\lambda,c)$ 为条件得分，$\epsilon_\theta(z_\lambda)$ 为无条件得分。该公式通过将预测推向条件得分的同时远离无条件得分，增强了条件的影响。
**可参考的原始文献**：
- [[基于扩散的无监督学习：扩散模型的基本原理]]
- [[VDM：变分扩散模型]]
- [[SDE扩散：基于随机微分方程的流匹配扩散模型]]
## 五、实验结果与启示
- **有效平衡保真度与多样性**：通过调整引导权重 $w$，模型可以像 GAN 的截断机制一样，完美再现 IS (Inception Score，衡量图像清晰度) 和 FID (衡量分布距离) 之间的权衡曲线。小权重的 $w$ 即可达到最优 FID，强引导能极大提高 IS 并导致颜色和特征的饱和。
- **纯生成模型的能力验证**：实验证明，纯扩散生成模型（神经网络不一定构成保守矢量场）在没有额外分类器参与的情况下，依然能够极大优化基于分类器的评价指标。彻底排除了原先“对抗攻击欺骗指标”的猜想。
- **无条件训练的容量占比**：超参数消融实验指出，联合训练时的无条件丢弃概率 $p_{uncond}$ 取 0.1 或 0.2 时效果最佳，0.5 效果较差。这启示网络只需极小部分容量去拟合无条件分布，即可获得高度可靠的无条件得分指引。
- **计算成本的权衡**：CFG 的主要代价是采样阶段的计算量翻倍（每次步进需计算两次模型前向传播）。尽管如此，由于免除了训练和部署分类器的成本，其架构的极度简化使其成为了后续文本到图像生成模型的标准范式。
# 关键启示
- **条件生成模型的重要范式**，处理多模态数据时/生成对应的diffusion扩散数据时，可以通过引入一个无条件分支来实现对条件生成的引导，且不需要额外训练分类器
- 是一种基本性的采样增强机制，被广泛应用于各种条件生成任务，如文本到图像、图像到图像等

