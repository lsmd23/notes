1. 
	1)  I
	2) 接受所有后缀为01或00的串
2. 
	1) 描述其状态集，始终态和转移函数如下：
		1) 状态集合$Q$：状态集合中的每个状态$q$表示一个字母表集合$S(S\subseteq \Sigma)$，记录当前出现过的字符，同时另有终态记为$F$
		2) 始终态：初态为$S_0=\emptyset$，表示没有字符出现过，唯一的终态为$F$
		3) 转移函数：对$\forall \sigma\notin S$，添加转移$\delta(S,\sigma)=S\cup \{\sigma\}$以及转移到终态的函数%$\delta(S,\sigma)=F$；反之，对$\forall \sigma\in S$，添加自环$\delta(S,\sigma)=S$
	2) 转移图如下：![[Pasted image 20250306214455.png]]
3. 
	1)  ![[Pasted image 20250306214922.png]]
	2) ![[Pasted image 20250306215113.png]]
	3) 转换结果如下：
4. 
	1) 如下：
		- $EC(p)=\{p,q,r\}$
		- $EC(q)=\{q\}$
		- $EC(r)=\{r\}$
	2) 如下：$\epsilon,a,b,c,aa,ab,ac,ba,bb,bc,ca,cb,cc,$
	3) DFA：
5. 