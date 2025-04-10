# 基本概念
- 定义：四元组$G=(V,T,S,P)$
	- $V$：变量的集合
	- $T$：终结符的集合
	- $S$：开始变量
	- $P$：产生式的集合
	- 关系：$V\cap T=\emptyset,S\in V$；对产生式$P$：$A\rightarrow\alpha,A\in V,\alpha\in(V\cup T)^*$
	- *巴克斯-瑙尔范式：计算机中产生式的$\rightarrow$常用$::=$表示*
	- 所有的推导过程只和变量与产生式有关，而和上下文无关，因此被称为上下文无关文法
# 归约与推导
- 用于判断字符串是否属于文法的语言
- 归约过程：将产生式的右半部分替换为左半部分
	- 自下而上的推理
	- 例：![[Pasted image 20250402143420.png]]
- 推导过程：将产生式的左半部分替换为右半部分
	- 对CFG：$G=(V,T,S,P)$，设$\alpha,\beta\in(V\cup T)^*$，$A\rightarrow \gamma$是一个产生式，则定义推导过程$\alpha A\beta\Rightarrow_G\alpha\gamma\beta$，若G在上下文中是明确的，则简记为$\alpha A\beta\Rightarrow\alpha\gamma\beta$
	- 自上而下的推理
	- 例：![[Pasted image 20250402143531.png]]
	- 推导传递闭包：$\Rightarrow_G^*$，归纳定义
		- 基础：对任何$\alpha\in(V\cup T)^*$，有$\alpha\Rightarrow_G^*\alpha$
		- 归纳：$\alpha,\beta,\gamma\in(V\cup T)^*$，若$\alpha\Rightarrow_G^*\beta,\beta\Rightarrow_G^*\gamma$，则$\alpha\Rightarrow_G^*\gamma$
- 最左推导：
	- 若推导过程的每一步总是替换出现在最左边的非终结符，则称这样的推导为最左推导，用$\Rightarrow_{lm}$表示，其传递闭包用$\Rightarrow_{lm}^*$表示
	- 例：![[Pasted image 20250402144449.png]]
- 最右推导：
	- 若推导过程的每一步总是替换出现在最右边的非终结符，则称这样的推导为最右推导，用$\Rightarrow_{rm}$表示，其传递闭包用$\Rightarrow_{rm}^*$表示
	- 例：![[Pasted image 20250402144550.png]]
- 句型：对CFG：$G=(V,T,S,P)$，称$\alpha\in(V\cup T)^*$为$G$的一个句型
	- 若$S\Rightarrow_{lm}^*\alpha$，则$\alpha$是一个左句型
	- 若$S\Rightarrow_{rm}^*\alpha$，则$\alpha$是一个右句型
	- 若句型$\alpha\in T^*$，则称$\alpha$为一个句子
# 语法分析树
- 归约和推导过程，都是在构建语法分析树
	- ![[Pasted image 20250409111809.png]]
	- ![[Pasted image 20250409111752.png]]
- 定义：对上下文无关文法$G=(V,T,S,P)$，其语法分析树满足以下条件：
	- 每个内部节点都由一个非终结符标记
	- 每个叶节点为一个变量，或终结符，或$\epsilon$标记（当$\epsilon$标记时，其一定为父节点的唯一子节点）
	- 若内部节点标记为A，而子节点从左至右标记为$X_1,X_2,\cdots,X_k$，则$A\rightarrow X_1X_2\cdots X_k$是一个产生式
	- 例：![[Pasted image 20250409112206.png]]
- 分析树的产物：
	- 将语法分析树的每个叶结点按照从左至右的次序连接起来，得到一个$(V\cup T)^*$中的字符串，称为该语法树的产物
	- G的每个句型都是某个根结点为S的分析树的产物；这些分析树中，有些树的产物为句子，所有这些分析树的产物构成了G的语言
- 归约、推导与语法分析树：
	- 对上下文无关文法$G=(V,T,S,P)$，以下命题相互等价
		- 字符串$w\in T^*$可以归约到非终结符$A$
		- $A\Rightarrow^* w$
		- $A\Rightarrow^*_{lm} w$
		- $A\Rightarrow^*_{rm} w$
		- 存在一棵根节点为$A$的分析树，产物为$w$
	- 证明：
		- ![[Pasted image 20250409112808.png]]
		- ![[Pasted image 20250409112815.png]]
		- ![[Pasted image 20250409112822.png]]
		- ![[Pasted image 20250409112831.png]]
		- ![[Pasted image 20250409112837.png]]
# 上下文无关语言
- 定义：上下文无关文法$G=(V,T,S,P)$，其语言为：$L(G)=\{w|w\in T^*,S\Rightarrow^*_G w\}$，称为上下文无关语言
- 证明给定语言对应的文法：
	- 方法：双向推导，对给定语言，根据字符串长度$|w|$作归纳；对给定文法，对推导的步骤作归纳
	- 例：![[Pasted image 20250409112101.png]]![[Pasted image 20250409112107.png]]
# 文法的二义性
- 二义文法：
	- 上下文无关文法$G=(V,T,S,P)$，若对某个$w\in T^*$，存在根节点都为开始符号$S$的两棵不同的语法分析树，且产物都是$w$，则称该文法为二义的
	- 反之，若对每一个$w\in T^*$，仅存在一棵这样的分析树，称该文法为无二义的
	- 定理：上下文无关文法$G=(V,T,S,P)$与$w\in T^*$，$w$有两棵不同的语法分析树，当且仅当存在两个不同的从$S$开始到$w$最左推导
- 文法是否有二义性是一个**不可判定的问题**，也没有通用的算法消除文法的二义性，但对特定意义的文法可以**消除二义性**
	- 例：简单运算的文法的二义性消除——算符优先级法：调整运算符的优先级对应的文法
		- 变换：![[Pasted image 20250409115102.png]]但不能解决串$v+v+d$对应的二义性
		- 变换——左结合法：将对称的表示式拆分，向一侧结合![[Pasted image 20250409115137.png]]
	- 例：悬挂else二义性
		- ![[Pasted image 20250409115456.png]]
		- 变换——最近嵌套法：拆分表示式为非对称的![[Pasted image 20250409115849.png]]
	- 固有二义语言：有的语言的所有文法都是二义的，称这类语言是固有二义的，它们只有二义文法
		- 例：![[Pasted image 20250409115344.png]]对串$a^nb^nc^n$一定有两棵语法分析树![[Pasted image 20250409115434.png]]
# CFG的构造
- 不存在通用的CFG构造算法
	- 对正则语言，只需构造线性文法即可
	- 对非正则语言，优先考虑文法中的递归过程，保证递归的循环和完结



# CFG的应用
- 正则表达式：匹配字符串
	- 元字符：![[Pasted image 20250409095957.png]]![[Pasted image 20250409100004.png]]
	- 缩写字符：![[Pasted image 20250409100021.png]]
	- 正则表达式的范式可以通过CFG生成
	- 类似的，其他规范性的语句也可通过CFG生成
		- 命题逻辑中的语法
		- 一阶谓词逻辑中的语法
		- 线性时序逻辑中的文法
- 程序设计语言的语法描述与分析——Yacc语法规则
	- 程序的编译时，需要分析程序的语法和逻辑，借助CFG可以得到词法分析器的生成器和语法解析器的生成器
	- Yacc源程序式样：
```
# 声明节：定义文法中的单词(%token命令)以及c代码相关声明

%%
# 语法规则节：定义语法规则以及语义动作
# 格式：非终结符:符号串 {C语句表示的语义动作}

%%
# 支撑函数节：用到的局部C函数定义
```
- 标记语言：标记语言绝大多数都可以用CFG解释
	- HTML：网页的描述语言
	- SGML：国际标准标记语言
	- JSON：更适合描述各类数据的正则语言
		- 例：![[Pasted image 20250409104517.png]]
	- XML：SGML的一个子集，可以通过文档定义DTD自定义tag的类型
		- DTD的格式：DTD实质上是一个正则表示
```
<!DOCTYPE name-of-DTD
	[list of element definitions]>
	<!ELEMENTelement-name
	(description of the element)>
```
# CFG的转换
