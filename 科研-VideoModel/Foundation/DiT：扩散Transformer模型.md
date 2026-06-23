- 原文：[[Scalable diffusion models with Transformers.pdf]]
# 文献内容
## 一、 解决的问题
传统的扩散模型（Diffusion Models）在图像生成领域取得了巨大的成功，但它们几乎全都采用卷积U-Net作为事实上的骨干网络（Backbone）。本研究探索了一类基于Transformer架构的新型扩散模型（DiT），旨在打破U-Net的归纳偏置限制，并证明Transformer架构在扩散模型中同样具备极其优异的可扩展性（Scalability）、鲁棒性和效率，为未来的生成模型研究提供标准化的架构基准。
## 二、 必要的前置工作
本研究建立在去噪扩散概率模型（DDPMs）和潜在扩散模型（LDMs）的基础之上。
**1. 扩散模型的数学基础**：扩散模型包含一个前向加噪过程，它逐渐向真实数据$x_{0}$添加噪声： $$q(x_{t}|x_{0})=\mathcal{N}(x_{t};\sqrt{\overline{\alpha}_{t}}x_{0},(1-\overline{\alpha}_{t})I)$$ 其中$x_{0}$表示真实数据图像，$x_{t}$表示第$t$步加噪后的图像，$\overline{\alpha}_{t}$表示控制噪声调度的超参数常量。通过重参数化技巧，可以采样得到对应的状态： $$x_{t}=\sqrt{\overline{\alpha_{t}}}x_{0}+\sqrt{1-\overline{\alpha}_{t}}\epsilon_{t}$$ 其中$\epsilon_{t}\sim\mathcal{N}(0,I)$为标准高斯噪声。 模型需要学习逆向去噪过程来反转前向破坏过程： $$p_{\theta}(x_{t-1}|x_{t})=\mathcal{N}(\mu_{\theta}(x_{t}),\Sigma_{\theta}(x_{t}))$$ 其中$\mu_{\theta}$和$\Sigma_{\theta}$均由神经网络进行预测。为了简化变分下界的优化，$\mu_{\theta}$通常被重参数化为一个噪声预测网络$\epsilon_{\theta}$，训练的核心目标即计算预测噪声与真实采样的均方误差：

$$\mathcal{L}_{simple}(\theta)=||\epsilon_{\theta}(x_{t})-\epsilon_{t}||_{2}^{2}$$

如需训练具有学习逆向协方差$\Sigma_{\theta}$的扩散模型，则需要进一步优化完整的$\mathcal{D}_{KL}$。 
**2. 无分类器引导（Classifier-free guidance）**：条件扩散模型接收额外输入，逆过程变为$p_{\theta}(x_{t-1}|x_{t},c)$，其中$c$为类别标签。基于贝叶斯定理，可推导得出： $$\nabla_{x}\log p(c|x)\propto\nabla_{x}\log p(x|c)-\nabla_{x}\log p(x)$$ 通过将扩散模型输出解释为分数函数（Score function），可以引导DDPM采样过程偏向生成具有高条件概率$p(x|c)$的样本： $$\hat{\epsilon}_{\theta}(x_{t},c)=\epsilon_{\theta}(x_{t},\emptyset)+s\cdot(\epsilon_{\theta}(x_{t},c)-\epsilon_{\theta}(x_{t},\emptyset))$$ 其中$s>1$表示引导尺度，$\emptyset$代表训练中随机丢弃类别标签替换得到的学习到的“空”嵌入。无分类器引导技术能够大幅提升生成样本的视觉质量。
**3. 潜在扩散模型（LDMs）**：直接在高分辨率像素空间中训练成本过高，因此LDM采用两阶段法。首先利用一个冻结的预训练自编码器将图像压缩为较小的空间表示$z=E(x)$。其次，在潜在表示空间中训练扩散模型，最后通过解码器$x=D(z)$将生成的潜变量解码回图像像素。
## 三、 DiT的新架构建模与解析
DiT直接作用于潜在空间中，遵循Vision Transformer (ViT) 的结构设计，通过处理图像块序列（Patches）来执行扩散去噪任务。
![[Pasted image 20260404174444.png]]
**核心组件与设计空间说明：**
- **Patchify（分块化）**：将空间维度为$I\times I\times C$的潜在空间特征转换为包含$T=(I/p)^2$个隐维度为$d$的Token序列。其中超参数$p$决定了分块的大小，将其减半会使Token数量$T$增加4倍，从而导致Transformer的Gflops至少增加4倍。
- **条件注入机制与adaLN-Zero**：模型对多种条件注入策略进行了探索，最终确定带有零初始化的自适应层归一化（adaLN-Zero）模块性能最优且极其高效。该设计直接从$t$和$c$的嵌入求和中，回归出标准的缩放参数$\gamma$与偏移参数$\beta$，并额外回归出应用于残差连接内部的缩放因子$\alpha$。将输出$\alpha$的MLP初始化为零向量，可以使DiT块的初始状态等效于恒等函数（Identity function），极大地提高了训练稳定性。
- **Transformer Decoder（解码器）**：在所有DiT Block串联结束后，通过一个标准线性映射层对Token进行解码，将其投影成尺寸为$p\times p\times 2C$的张量形式。随后模型将这些解码块重新排列回原有的空间维度布局，进而同步输出对噪声分布以及对角协方差的预测结果。
**原文架构图**：![[Pasted image 20260404175003.png]]
## 四、 实验结果与启示
该研究在极其严苛的生成模型基准（ImageNet）上开展了深入扩展性（Scaling）分析：
- **计算量（Gflops）是提升生成质量的核心所在**：实验证明模型参数量无法准确反应架构复杂度，而DiT的理论计算前向开销（Gflops）与最终生成质量指标（FID）呈现出极强的负相关性（相关系数达-0.93）。通过不断增加Transformer网络的深度和宽度，或使用更小的块尺寸来增加输入Token数量，均能一致且稳定地改善模型的FID表现。
- **大规模DiT模型具备更高的计算效率**：在固定总体训练消耗算力（Training Compute）的情况下，尺寸更庞大的DiT网络会以更少的训练步数超越训练周期更长的小型DiT模型，展现出非凡的效率优势。
- **增加推理计算量无法替代模型本身容量**：分析证实，单纯在推理测试期依靠增加采样步数（例如将步数拉升至1000步），根本无法让小型DiT追平或逆转那些采样步数较少（如128步）的大型DiT模型的生成性能。
- **打破U-Net垄断并达成SOTA成就**：最强大的模型版本DiT-XL/2在类别条件ImageNet $256\times256$以及$512\times512$生成任务中，彻底击败了包含ADM、LDM在内的全部历史U-Net扩散网络，于$256\times256$分辨率上达成了2.27的惊人FID最佳记录。 
**研究启示**：长期存在于生成领域的U-Net卷积归纳偏置绝非扩散模型的刚需。通过将骨干网络替换为Transformer架构，生成任务成功继承了自然语言处理和图像识别领域中已验证的Scaling Law（缩放定律），预示着大语言模型的技术范式向图像生成领域的进一步深度统一。