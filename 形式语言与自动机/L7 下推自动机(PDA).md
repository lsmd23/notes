# 基本概念
- 将NFA与一个栈结合，即形成PDA
	- 每次接受一个输入字符，对栈进行一次操作（一次弹出与压入）
		- 只能弹出一个字符，但压入可以压入一个字符串
	- 栈不可为空，对空栈进行任意的操作都将导致自动机停机
	- 类似NFA，同一状态可以对应两种不同的转移（即两次不同的栈操作），同样也可以有空串$\epsilon$输入（即$\epsilon$-转移），这样的自动机是NPDA
- 定义：
	- 终态型PDA是七元组：$P=(Q,\Sigma,\Gamma,\delta,q_0,Z_0,F)$
		- $Q$：有限状态集合
		- $\Sigma$：有限输入字母表
		- $\Gamma$：有限栈字符
		- $\delta$：转移函数，$\delta:Q\times (\Sigma\cup\{\epsilon\})\times\Gamma\rightarrow 2^{Q\times\Gamma^*}$
		- $q_0$：初态（唯一）
		- $Z_0$：栈初始符号（唯一）
		- $F$：终态集合
	- 空栈型PDA是六元组：$P=(Q,\Sigma,\Gamma,\delta,q_0,Z_0)$
		- 没有终态集合
	- 如无特殊说明，默认PDA是NPDA，且为终态型PDA
	- 接受与拒绝：
		- 接受：一个输入串被PDA接受，如果所有的字符都被输入完毕且落在PDA的一个终态（与栈无关）。被PDA接受的串的集合即**PDA的语言**
		- 拒绝：一个输入串被PDA拒绝，如果所有的字符不能读完或最后不能落在PDA的一个终态（与栈无关）
	- 例：
		- ![[Pasted image 20250416110623.png]]
		- ![[Pasted image 20250416111910.png]]
		- ![[Pasted image 20250416111958.png]]
# PDA的即时描述
- 定义：三元组：$(q,w,\gamma)$
	- $q$：当前状态
	- $w$：剩余输入串
	- $\gamma$：栈中当前符号串
- 传递：
	- 定义：设PDA：$P=(Q,\Sigma,\Gamma,\delta,q_0,Z_0,F)$，$\vdash_P$或$\vdash$满足：若$(p,\alpha)\in\delta(q,a,X)$，则$(q,aw,X\beta)\vdash(p,w,\alpha\beta)$，其中$p,q\in Q,a\in \Sigma,w\in \Sigma^*,X\in \Gamma.\alpha,\beta\in\Gamma^*$
		- 例：![[Pasted image 20250416120328.png]]![[Pasted image 20250416120334.png]]则有传递：$(q_1,bbb,aaaz_0)\vdash(q_2,bb,aaz_0)$
	- 传递闭包：$\vdash_P^*$或$\vdash^*$定义为：对任意即时描述$I$，有$I\vdash^*I$；若$I\vdash K,K\vdash^*J$，则有：$I\vdash^*J$
- 定理：设PDA：$P=(Q,\Sigma,\Gamma,\delta,q_0,Z_0,F)$，若$(q,x,\alpha)\vdash^*(p,y,\beta)$，则对任意的$w\in\Sigma^*,\gamma\in\Gamma^*$，有：$(q,xw,\alpha\gamma)\vdash^*(p,yw,\beta\gamma)$
	- 也即：传递之间是相互独立的
# PDA的语言
- 终态型PDA的语言：设NPDA：$P=(Q,\Sigma,\Gamma,\delta,q_0,Z_0,F)$，语言定义为：$L(M)=\{w|w\in\Sigma^*,(q_0,w,z_0)\vdash_M^*(q_f,\epsilon,u),u\in\Gamma^*\}$其中$(q_0,w,z_0)$是初始状态，$(q_f,\epsilon,u)$是最终状态
- 空栈型PDA的语言：设NPDA：$P=(Q,\Sigma,\Gamma,\delta,q_0,Z_0)$，语言定义为$L(M)=\{w|(q_0,w,z_0)\vdash^*(q,\epsilon,\epsilon),q\in Q\}$（也即可以使栈清空的串）
- 关系：
	- 对空栈型PDA：$P_N=(Q,\Sigma,\Gamma,\delta,q_0,Z_0),L=L(P_N)$，存在一个终态型PDA：$P_F$，使得$L=L(P_F)$
		- 证明：![[Pasted image 20250416130348.png]]
	- 对终态型PDA：$P_F=(Q,\Sigma,\Gamma,\delta,q_0,Z_0,F),L=L(P_F)$，存在一个空栈型PDA：$P_N$，使得$L=L(P_N)$
		- 证明：![[Pasted image 20250416130448.png]]
# PDA与CFG的关系
- 定理：上下文无关文法的语言和PDA的语言等价
- CFG到PDA：对CFG：$G=(V,T,S,P)$，可以构造空栈型PDA：$M=(\{q\},T,V\cup T,\delta,q,S)$
	- 定义转移函数：
		- 对每一个$A\in V$，有：$\delta(q,\epsilon,A)=\{(q,\beta)|\text{“}A\rightarrow \beta\text{”}\in P\}$
		- 对每一个$a\in T$，有：$\delta(q,a,a)=\{(q,\epsilon)\}$
	- 例：![[Pasted image 20250417175922.png]]![[Pasted image 20250417175940.png]]
	- 证明：采用归纳法证明
- PDA到CFG：对空栈型PDA：$P=(Q,\Sigma,\Gamma,\delta,q_0,Z_0)$，可以构造CFG：$G=(V,\Sigma,P,S)$，其中$V=\{S\}\cup\{[pX q]|p,q\in Q,X\in \Gamma\}$
	- 产生式集合定义如下：
		1. 对$\forall p\in Q$，G包含产生式$S\rightarrow [q_0Z_0p]$
		2. 变量$[pXq]$的产生式：
			1) 若$(q,\epsilon)\in \delta(p,a,X)$，则：$[pXq]\rightarrow a$
			2) 若$(q,X_1X_2\cdots X_k)\in \delta(p,a,X)$，则G包含产生式$[pXp_k]\rightarrow a[qX_1p_1][p_1X_2p_2]\cdots[p_{k-1}X_kp_k]$，其中$a\in \Sigma\cup\{\epsilon\};\forall p_i\in Q,i=1,2,\cdots,k$
	- 例：![[Pasted image 20250418153719.png]]![[Pasted image 20250418153737.png]]
# 确定下推自动机(DPDA)
## 基本概念
- 定义：对某PDA：$P=(Q,\Sigma,\Gamma,\delta,q_0,Z_0,F)$，其被称为确定下推自动机(DPDA)，若
	- 对$a\in \Sigma\vee a=\epsilon,X\in \Gamma-\{\epsilon\}$，转移函数$\delta(q,a,X)$最多包含一个元素
	- 对$a\in \Sigma\wedge a=\epsilon$，若$\delta(q,a,X)\neq \emptyset$，则$\delta(q,\epsilon,X)=\emptyset$
	- 含义：
- DPDA的语言与其它语言的关系：
	- 显然，DPDA的语言是CFG的语言的真子集
	- 定理：若L是正则语言，则存在DPDA：P使得$L(P)=L$
		- 证明：
## 空栈型DPDA
- 定义：
- 语言：
- 前缀性质：
- 定理：语言L是空栈型DPDA的语言，当且仅当L具有前缀性质且L是某终态型DPDA的语言
	- 空栈型DPDA的语言是终态型DPDA的语言的真子集
## DPDA与二义文法
- 定理：若语言L是空栈型DPDA的语言，则存在无二义的上下文无关文法G使得$L=L(G)$
	- 证明思路：前节中，从空栈型PDA构造等价CFG的方法，构造与DPDA等价的CFG；可证对于任何所接受的串w，此CFG有唯一的最左推导，因而是无二义文法
- 定理：若语言L是终态型DPDA的语言，则存在无二义的上下文无关文法G使得$L=L(G)$
	- 证明思路：
- 也即，**DPDA的语言是CFG非固有二义语言的真子集**
# CFG的化简与规范
- 每个转移函数占用计算机的资源，因此，对CFG进行化简是很有必要的
- 消去无用符号：
	- 有用符号：
- 消去$\epsilon$产生式
- 消去单一产生式
- CFG的化简与Chomsky范式