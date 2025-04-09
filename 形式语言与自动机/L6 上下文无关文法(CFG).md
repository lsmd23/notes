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
