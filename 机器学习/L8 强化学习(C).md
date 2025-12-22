# 策略梯度方法
- 基于价值的参数化方法：参见[[L7 强化学习(B)#^2fcb18|价值函数近似方法]]
- 基于策略的参数化方法：直接使用参数对策略进行建模和优化
	- 建模：$$\pi(a|s;\pmb \theta)=\mathbb{P}[A_t=a|S_t=s,\pmb \theta]$$
	- 优势：
		- 适用于高维或连续动作空间
		- 相比基于价值的学习，直接对策略建模得到的策略具有更强的探索性，更容易收敛到最优解
	- 常用的建模方法：
		- 离散动作空间：使用Softmax函数$$\pi(a|s;\pmb \theta)=\frac{e^{h(s,a,\pmb \theta)}}{\sum_{a'}e^{h(s,a',\pmb \theta)}}$$其中$h(s,a,\pmb \theta)$为参数化的偏好函数：$$h(s,a,\pmb \theta)=\pmb \theta^T \pmb x(s,a)$$这里$\pmb x$为状态-动作对的特征向量
		- 连续动作空间：使用高斯分布$$\pi(a|s;\pmb \theta)=\frac{1}{\sigma(s,\pmb \theta)\sqrt{2\pi}}\exp\left(-\frac{(a-\mu(s,\pmb \theta))^2}{2\sigma(s,\pmb \theta)^2}\right)$$其中$\mu(s,\pmb \theta)$和$\sigma(s,\pmb \theta)$分别为参数化的均值和标准差函数，例如：$$\mu(s,\pmb \theta)=\pmb \theta_\mu^T \pmb x_\mu(s),\sigma(s,\pmb \theta)=\exp(\pmb \theta_\sigma^T \pmb x_\sigma(s))$$这里$\pmb x_\mu(s)$和$\pmb x_\sigma(s)$为状态的特征向量
- 策略梯度建模：
	- 建模一个性能指标$J(\pmb \theta)$，表示策略$\pi(a|s;\pmb \theta)$的好坏
	- 如果可以定量计算该指标，就可以用梯度上升的办法完成强化学习：$$\pmb \theta_{t+1}=\pmb \theta_t + \alpha \nabla_{\pmb \theta} \widehat{J(\pmb \theta_t)}$$
	- 对情节式任务（也即轨迹有终点），起始状态在当前策略下的状态价值函数的期望可以作为性能指标：$$J(\pmb \theta)=v_{\pi_\theta}(s_0)$$其中$s_0$为初始状态
	- 对持续式任务，需满足两个条件：
		- 平稳分布条件：$$\mu_{\pi_\theta}(s)=\lim_{t\to\infty}\mathbb{P}[S_t=s|A_{0:t-1}\sim \pi_\theta]$$也即当按照策略进行马尔可夫游走时，经过足够长时间后，状态分布收敛到一个平稳分布
		- 遍历性假设：也即对$\forall s'\in \mathcal{S}$，有：$$\sum_s\mu_{\pi_\theta}(s)\sum_a\pi_{\pmb \theta}(a|s)\mathcal{P}_{ss'}^a = \mu_{\pi_\theta}(s')$$也即从平稳分布出发，经过一步转移后，仍然保持在平稳分布上
		- 在满足上述条件下，可以定义性能指标为：$$J(\pmb \theta)=\sum_s \mu_{\pi_\theta}(s) \sum_a \pi_{\pmb \theta}(a|s) \mathcal{R}_s^a$$也即平均每一步的期望奖励
## 策略梯度定理
- 内容：对于一个可微的策略$\pi(a|s;\pmb \theta)$，其性能指标$J(\pmb \theta)$的梯度可以表示为：$$\nabla_{\pmb \theta} J(\pmb \theta)=\mathbb{E}_{\pi}\left[q_{\pi}(S_t,A_t)\nabla_{\pmb \theta} \log \pi(A_t|S_t;\pmb \theta)\right]$$
	- 说明：性能指标$J(\pmb \theta)$是和环境有关的，但其梯度是和环境无关的，只依赖于策略$\pi$和动作价值函数$q_{\pi}$
	- 证明：对情节式任务有证明，此处略去（可参考课件）
## REINFORCE算法
- 背景：
	- 根据累计回报$G_t$的性质：$$\mathbb{E}_{\pi}[G_t|S_t=s,A_t=a]=q_{\pi}(s,a)$$也即$G_t$的期望就是动作价值函数$q_{\pi}(s,a)$，可以使用$G_t$的采样值来替换策略梯度定理$q_{\pi}(s,a)$：$$\nabla J(\pmb \theta)=\mathbb{E}_{\pi}\left[G_t \nabla_{\pmb \theta} \log \pi(A_t|S_t;\pmb \theta)\right]$$
	- 由此，可以得到更新策略：$$\pmb \theta_{t+1} = \pmb \theta_t + \alpha G_t \nabla_{\pmb \theta} \log \pi(A_t|S_t;\pmb \theta_t)$$
- 蒙特卡洛策略梯度算法（REINFORCE）：
	- 实现：![[Pasted image 20251222155012.png]]
		- 输入：参数化策略$\pi(a|s;\pmb \theta)$，学习率$\alpha$
		- 初始化策略参数$\pmb \theta$
		- 对每一步迭代
			- 生成一个完整的轨迹：$S_0,A_0,R_1,S_1,A_1,R_2,\ldots,S_{T-1},A_{T-1},R_T\sim \pi(\cdot|\cdot,\pmb \theta)$
			- 对轨迹中的每个时间步$t=0,1,\ldots,T-1$
				- 计算累计回报：$G_t=\sum_{k=t+1}^{T}\gamma^{k-t-1}R_k$
				- 更新策略参数：$$\pmb \theta \leftarrow \pmb \theta + \alpha \gamma^t G_t \nabla_{\pmb \theta} \log \pi(A_t|S_t;\pmb \theta)$$
				- 这里$\gamma^t$是由于在策略梯度定理中引入折扣因子的结果
	- 问题：相当于使用蒙特卡洛估计方法对单步动作价值函数$q_{\pi}(s,a)$进行估计，方差较大，收敛较慢
- 带基线的蒙特卡洛策略梯度算法：
	- 思路：引入一个基线函数$b(s)$，减小估计的方差，但不改变估计的期望：$$\nabla J(\pmb \theta)\propto\mathbb{E}_{\pi}\left[(q_\pi(S_t,A_t) - b(S_t)) \nabla_{\pmb \theta} \log \pi(A_t|S_t;\pmb \theta)\right]$$
	- 数学基础：引入后的一项展开后期望为0：$$\begin{align*}\mathbb{E}_{\pi}\left[b(S_t)\nabla \log \pi(A_t|S_t, \boldsymbol{\theta})\right] &= \sum_{s} \mu(s) \sum_{a} b(s)\nabla \pi(a|s, \boldsymbol{\theta}) \\ &= \sum_{s} \mu(s)\, b(s) \nabla \sum_{a} \pi(a|s, \boldsymbol{\theta}) \\ &= \sum_{s} \mu(s) \, b(s) \nabla 1 = 0\end{align*}$$因此不会引入偏差
	- 常用的基线函数：状态价值函数$\hat v(S_t,\pmb w)$，可以通过蒙特卡洛方法或时序差分方法进行估计
	- 实现：![[Pasted image 20251222215708.png]]
		- 引入基线函数$\hat v(S_t,\pmb w)$，并初始化其参数$\pmb w$
		- 在采样完成计算累计回报$G_t$后，计算基线估计：$\delta\leftarrow G-\hat v(S_t,\pmb w)$
		- 使用估计值，更新基线函数的参数：$\pmb w \leftarrow \pmb w + \alpha^{\pmb w} \delta \nabla_{\pmb w} \hat v(S_t,\pmb w)$
		- 使用估计值，更新策略参数：$$\pmb \theta \leftarrow \pmb \theta + \alpha^{\pmb w} \gamma^t \delta \nabla_{\pmb \theta} \log \pi(A_t|S_t;\pmb \theta)$$
## 演员-评委模型
- 背景：蒙特卡洛方法是一个合理的价值估计，但方差过大，且只能在情节结束后才能更新
	- 时序差分方法可以在线更新，因此将TD学习合价值函数逼近的方法引入策略梯度中进行学习
	- 带入参数化价值的策略梯度：$$\nabla_{\pmb \theta} J(\pmb \theta)=\mathbb{E}_{\pi}\left[q_{\pi}(S_t,A_t)\nabla_{\pmb \theta} \log \pi(A_t|S_t;\pmb \theta)\right]$$其中，使用评委模型对动作价值函数进行参数化估计：$$q_{\pi}(s,a)\approx \hat q(s,a,\pmb w)$$使用演员模型更新策略梯度：$$\nabla J(\pmb \theta) = \hat{q}(S_t,A_t,\pmb w)\nabla\log\pi(A_t|S_t,\pmb \theta)$$
- 兼容函数近似定理：
	- 一般的策略函数近似会引入偏差
	- 定理：满足以下条件可以使得对策略的估计最终是一个无偏估计
		- 价值函数近似与策略兼容：也即梯度相等$$\nabla_{\pmb w} \hat q(s,a,\pmb w) = \nabla_{\pmb \theta} \log \pi(A_t|S_t;\pmb \theta)$$
		- 参数化的价值函数最小化均方误差：$$\epsilon= \mathbb{E}_{\pi_\theta}\left[\hat q(S_t,A_t,\pmb w)-q_{\pi_\theta}(S_t,A_t)\right]^2$$
- 单步演员-评委算法：从TD(0)算法得到
	- 实现：![[Pasted image 20251222161230.png]]
		- 参考[[L7 强化学习(B)#^6ef538|TD(0)算法的参数化更新]]
- 引入资格迹的演员-评委算法：从TD(λ)算法得到
	- 实现：![[Pasted image 20251222162143.png]]
		- 参考[[L7 强化学习(B)#^0149b2|Sarsa(λ)算法的参数化更新]]
- 



# 强化学习的最新应用
## 近端策略优化(PPO)
- 背景：传统的策略梯度方法在更新策略时，可能会导致策略发生剧烈变化，从而影响学习的稳定性和收敛性
- **替代损失**：用于替换传统的策略梯度定理
	- 用累计回报的期望定义性能指标：$$J(\pmb \theta)=\mathbb{E}_{\tau\sim\pi_\theta}G(\tau)$$
	- 重要性采样转换：将其用另一个策略$\pi_{\theta'}$下的轨迹分布表示：$$J(\pmb \theta)=\mathbb{E}_{\tau\sim\pi_{\theta'}}\left[\frac{\mathbb{P}(\tau|\pi_\theta)}{\mathbb{P}(\tau|\pi_{\theta'})}G(\tau)\right]$$
- 置信域策略优化(TRPO)：
	- 

- 近端策略优化(PPO)：
	- 引入自适应
	- 剪切概率比的替代损失函数：$$L^{CLIP}(\pmb \theta)=\mathbb{E}_t\left[\min\left(r_t(\pmb \theta)\hat A_t, \text{clip}(r_t(\pmb \theta),1-\epsilon,1+\epsilon)\hat A_t\right)\right]$$
		- 这里$r_t(\pmb \theta)=\frac{\pi(A_t|S_t;\pmb \theta)}{\pi(A_t|S_t;\pmb \theta_{old})}$为新旧策略的概率比，$\hat A_t$为优势函数的估计，$\epsilon$为一个小的超参数
	- 优化：使用随机梯度上升的方法对该损失函数进行优化
- 组相对策略优化(GRPO)：
	- 问题：使用参数化模型