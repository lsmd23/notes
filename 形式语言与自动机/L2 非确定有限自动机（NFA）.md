# NFA的概念
- 若自动机中，存在两个变迁选择的转移函数，或存在有没有迁移的状态，则为非确定有限自动机
- NFA及其语言：
	- 字符串NFA**路径接受**：NFA的**一条路径中**，字符串w输入完，处于NFA的终态
	- 字符串NFA接受：NFA中**至少存在一条接受字符串的路径**
	- 字符串w被一条路径拒绝：
		- 字符串输完，但不在终态
		- 字符串无法输完
	- 字符串w被NFA拒绝：NFA中**没有路径可以接受字符串w**
	- NFA的语言：NFA接受字符串的集合
# $\epsilon$-转移
- 状态转移函数中允许空串作为输入，转移图上有用$\epsilon$标记的边，即为$\epsilon$-转移
- 也即，可以在无输入的时候，发生状态的迁移
- 例：
	- ![[Pasted image 20250305105202.png]]
	- ![[Pasted image 20250305105406.png]]
	- ![[Pasted image 20250305105506.png]]
# NFA的定义
- 非确定有限自动机DFA是五元组：
	- $Q$：有限状态集
	- $\Sigma$：输入符号集
	- $\delta$：转移函数：$Q\times \Sigma\cup\{\epsilon\}\rightarrow2^Q$
	- $q_0$：开始状态
	- $F$：终态集合
- 状态转移表：状态转移函数也可以用表表示
	- 例：![[Pasted image 20250305110524.png]]
- $\epsilon$-闭包：
	- 状态q的$\epsilon$-闭包是q包括q资深的$\epsilon$路径到达的所有状态，记为$EC(q)$
	- 归纳定义：$NFA,q\in Q,EC(q):$
		- $q\in EC(q)$
		- 若$p\in EC(q)$且$r\in \delta(p,\epsilon)$，则$r\in EC(q)$
	- 集合S的$\epsilon$-闭包$EC(S)$定义为$EC(S)=\bigcup_{q\in S}EC(q)$
	- 例：![[Pasted image 20250305111558.png]]
# 扩展转移函数
- 除去DFA中基本的定义外，在NFA中，应当对整个$\epsilon$-闭包作扩展
- 定义：$q_{j_k} \in \delta^*(q_i,w)$，则存在一条从$q_i$到$q_j$的路径，且$q_{j_k} \in EC\{q_j\}$![[Pasted image 20250305113147.png]]
	- 例：![[Pasted image 20250305113708.png]]
- NFA的语言：
	- $L(M)=\{w\in \Sigma^*|\delta^*(q_0,w)\cap F\neq \emptyset\}$
	- $L(M)=\{w_1,w_2,w_3,...\}$，满足$\delta^*(q_0,w_m)=\{q_i,q_j,...,q_k,...\},q_k\in F$
	- 例：![[Pasted image 20250305113652.png]]
# 等价性证明
- 两个自动机称为等价，如果两个自动机接受的语言相同
- 定理：**NFA与DFA有着相同的表达能力**
	- DFA到NFA：显然
	- NFA到DFA的算法：给定NFA，$M=(Q,\Sigma,\delta,q_o,F)$
		1. DFA的状态集选取NFA的幂集：$2^Q=\{\emptyset,[q_0],[q_1],[q_1q_2]\}$
		2. 若NFA的初始状态为$q_0$，则DFA的初始状态是$[EC(q_0)]$
		3. 对DFA状态$[q_i,q_j,...,q_m]$，如果在NFA中有迁移：$$\left.\begin{array}{l}&\delta^*(q_i,a),)\\&\delta^*(q_i,a),)\right\}=\{q_i',q_j',...,q_m'\}$$则在DFA中添加迁移：$\delta([q_iq_j...q_m],a)=[q_i'q_j'...q_m']$
		4. 对字母表中每个字符重复上述步骤，直到没有新的转移