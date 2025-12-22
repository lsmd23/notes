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
	- 如果可以定量计算该指标，就可以用梯度上升的办法完成强化学习：$$\pmb \theta_{t+1}=\pmb \theta_t + \alpha \nabla_{\pmb \theta} \hat{J(\pmb \theta_t)}$$
	- 对情节式任务（也即轨迹有终点），起始状态在当前策略下的状态价值函数的期望可以作为性能指标：$$J(\pmb \theta)=v_{\pi_\theta}(s_0)$$
	- 

## 策略梯度定理
- 内容：对于一个可微的策略$\pi(a|s;\pmb \theta)$，其性能指标$J(\pmb \theta)$的梯度可以表示为：$$\nabla_{\pmb \theta} J(\pmb \theta)=\mathbb{E}_{\pi}\left[q_{\pi}(S_t,A_t)\nabla_{\pmb \theta} \log \pi(A_t|S_t;\pmb \theta)\right]$$
	- 说明：性能指标$J(\pmb \theta)$是和环境有关的，但其梯度是和环境无关的，只依赖于策略$\pi$和动作价值函数$q_{\pi}$



## REINFORCE算法
- 背景：
	- 根据累计回报$G_t$的性质：$$\mathbb{E}_{\pi}[G_t|S_t=s,A_t=a]=q_{\pi}(s,a)$$也即$G_t$的期望就是动作价值函数$q_{\pi}(s,a)$，可以使用$G_t$的采样值来替换策略梯度定理$q_{\pi}(s,a)$
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
	- 思路：引入一个基线函数$b(s)$，减小估计的方差，但不改变估计的期望
	- 更新策略参数：$$\pmb \theta \leftarrow \pmb \theta + \alpha \gamma^t (G_t - b(S_t)) \nabla_{\pmb \theta} \log \pi(A_t|S_t;\pmb \theta)$$
	- 常用的基线函数：状态价值函数$v_{\pi}(s)$，可以通过蒙特卡洛方法或时序差分方法进行估计
## 演员-评委模型
- 背景：
- 兼容函数近似定理：
- 单步演员-评委算法：从TD(0)算法得到
	- 实现：![[Pasted image 20251222161230.png]]
		- 参考[[L7 强化学习(B)#^35f0e4]]
- 引入资格迹的演员-评委算法：从TD(λ)算法得到
	- 实现：![[Pasted image 20251222161512.png]]
		- 参考[[L7 强化学习(B)#^d3f5e2]]



# 强化学习的应用