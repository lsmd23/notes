# CFL的必要条件
- 由于Chomsky范式(CNF)有良好的文法结构，采用CNF进行语言的分析
- 推导：设CFG：$G=(V,T,S,P)$为CNF，且L为不含空串的无限CFL
	- 对语法分析树，记$|V|=m,n=2^m$，对$z\in L$，若最大路径的长度为$m$，则$|z|\leq 2^{m-1}<n$![[Pasted image 20250507100710.png]]
	- 反过来，如果$|z|\geq n=2^m$，则分析树的高度至少是$m+1$。从初始变量$S$开始的分析树最大路径为：$A_0A_1A_2\cdots A_ka$，则有$k\geq m$，但$|V|=m$，由鸽巢原理，其中一定有相同的变量，记为：$A_i=A_j(k-m\leq i<j\leq k)$
	- 将分析树进行分割，z表示为$z=uvwxy$，其中w是根节点为$A_j$子树的产物，vwx是根节点$A_i$的产物，v和x分别位于串w的左右。由于没有单一产生式，故$vx\neq \epsilon$。又有根节点为$A_i$的子树最高为$m+1$，则$|vwx|\leq 2^m=n$![[Pasted image 20250507102852.png]]
	- 可以得到”泵“性质：如图![[Pasted image 20250507110249.png]]也即对任意的$i>0,uv^iwx^iy\in L$

# CFL的泵引理
- 定理：设L是CFL，则存在常数$n$，对任意的$z\in L,|z|\geq n$，z可表示为$z=uvwxy$，且满足：$vx\neq \epsilon$；$|vwx|\leq n$；对任意$k\geq 0,uv^kwx^ky\in L$
	- 证明：
	- 应用，形式化为逻辑表示后，其否定型四用于判断一个语言不是CFL
- 证明语言不是CFL：
	1) 选取任意正整数n
	2) 找字符串$z\in L$，使得：
		1) $|z|\geq n$；
		2) 对满足以下条件的任意的$u,v,w,x,y:z=uvwxy,vx\neq \epsilon,|vwx|\leq n$
		3) 选择$k\geq 0$，使得$uv^kwx^ky\notin L$
	- 例：
		- ![[Pasted image 20250507101512.png]]
		- ![[Pasted image 20250507101604.png]]
# CFL的闭运算性质
- 定理：**CFL的并是CFL**
	- 证明：将变量集合加下标，相并，增加新的产生式：$S\rightarrow S_1|S_2$
- 定理：**CFL的\*闭包与+闭包都是CFL**
	- 证明：新的开始变量$S_1$，以及对应的产生式：$S_1\rightarrow SS_1|\epsilon$
- 定理：**CFL的连接是CFL**
	- 证明：将原语言的变量加下标区分，并增加新的初始变量$S$与对应的产生式：$S\rightarrow S_1S_2$
- 定理：**CFL的反转仍是CFL**
	- 证明：构造新的CFG：$G^R=(V,T,S,P^R)$，其中$P^R=\{A\rightarrow \alpha^R|A\rightarrow \alpha \in P\}$，证明对任意的$w$，都有$S \underset{G}{\stackrel{*}{  \Rightarrow}} w$对应于$S \underset{G_R}{\stackrel{*}{  \Rightarrow}} w^R$
# CFL的同态性质
- 定义：设$\Sigma$为字母表，$\mathscr L$为语言的集合，称映射$s:\Sigma\rightarrow \mathscr L$为$\Sigma$上的一个替换，也即对任意$a\in\Sigma$，$s(a)\in\mathscr L$为一个语言
	- 替换扩展：$s:\Sigma^*\rightarrow \mathscr L$，若$w=a_1a_2\cdots a_n\in\Sigma^*$，定义：$s(w)=s(a_1a_2\cdots a_n)=s(a_1)s(a_2)\cdots s(a_n)$
	- 设$L$为$\Sigma$上的语言，定义$s(L)=\bigcup_{w\in L}s(w)$
	- 例：![[Pasted image 20250507113442.png]]
	- 定理：设s为$\Sigma$上的替换，满足$\forall a\in \Sigma,s(a)$是CFL，若$L$是$\Sigma$上的CFL，则$s(L)$是一个CFL
- 同态与逆同态：参考[[L3 正则语言与正则表示#正则语言的同态|正则语言的同态]]
	- 定理：**CFL的同态和逆同态都是CFL**
# CFL的交运算
- CFL的交并不一定是CFL，因此其差、补等运算均不是封闭的
- 定理：（**正则闭性质**）$L$为CFL，$R$为正则语言，则$L\cap R$为CFL
	- 证明：构造对应的一个PDA：对DFA：$A=(Q_A,\Sigma,\delta_A,q_A,F_A)$，对PDA：$P=(Q_P,\Sigma,\Gamma,\delta_P,q_P,Z_0,F_P)$，新PDA：$P'=(Q_P\times Q_A,\Sigma,\Gamma,\delta,(q_p,q_A),Z_0,F_P\times F_A)$
		- 转移函数满足：$\delta((q,p),a,X)$包含所有的状态-栈字符对$((r,s),\gamma)$：$(r,\gamma)\in\delta_P(q,a,X),s=\delta_A^*(p,a),a\in\Sigma\cup\epsilon$
		- 直观来说，该自动机包含下列转移：
			- ![[Pasted image 20250507164756.png]]![[Pasted image 20250507164803.png]]
	- 应用：
		- 证明语言是CFL：![[Pasted image 20250507105940.png]]![[Pasted image 20250507105946.png]]
		- 反证法证明语言不是CFL
# CFL的判定性质
- 空语言问题：给定CFG，如何判定$L(G)=\emptyset$
	- 算法：判定CFG的开始变量否是无用的
- 无限语言问题：给定CFG，如何判定语言是无限的
	- 算法：消去单一产生式，$\epsilon$-产生式和无用符号，构造变量的依赖图，若有环，则语言是无限的
	- 例：![[Pasted image 20250507110626.png]]
- 语言元素问题：对给定CFG，如何判定$w\in L(G)$
	- 算法：CYK解析算法：设CFG：$G=(V,T,S,P)$为CNF，$w=a_1a_2\cdots a_n\in T^*$
		- 迭代计算下列满足条件的变量符号$X_{ij}(1\leq i\leq j\leq n)$：$X_{ij}\in V,A\in X_{ij} \text{   iff  } A\underset{G}{\stackrel{*}{  \Rightarrow}} a_ia_{i+1}\cdots a_j$
		- 判定：$w\in L(G)\text{ iff }S\in X_{1n}$
		- 迭代计算如下：
			- $j=i$，如果$A\rightarrow a_i\in P$，则$A\in X_{ii}$
			- $j>i$，$A∈X_{ij}$当且仅当存在 $k:i\leq k<j$，可以找到$B∈X_{ik}$和$C∈X_{(k+1)j}$，使得：$A→BC∈P$
		- 复杂度：$|w|=n$，复杂度为$O(n^3)$