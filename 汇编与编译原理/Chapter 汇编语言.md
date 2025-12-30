# 基本概念
- 汇编语言是连接硬件与高级编程语言的桥梁
	- 由指令、伪指令等构成
	- 需通过特定方法进行程序设计
	- 在实际场景中，汇编语言常用于底层编程、嵌入式系统开发等领域
- 虚拟机理论：
	- 每一台计算机直接执行的语言称为机器语言（L0语言），其是人类难以理解和使用的二进制代码
	- 在机器语言之上，可以设计一种更接近人类思维的低级语言，称为汇编语言（L1语言）
	- L1语言通过解释（逐条执行）或翻译（转换为机器语言后执行）的方式在机器上运行，二者之间是几乎一一对应的关系
- 数的表示和运算：参见[[L1.1 信息-整数的表示|计算机组成原理——整数的表示]]以及[[L1.2 信息-浮点数的表示|计算机组成原理——浮点数的表示]]
# x86处理器架构
- 基础处理器设计：参见计算机组成原理
	- [[L3.1 指令集架构|指令集架构]]
	- [[L3.2 电路逻辑设计|电路逻辑设计]]
	- [[L3.3 SEQ处理器设计|单周期顺序处理器设计]]
	- [[L3.4 流水线处理器设计|流水线处理器设计]]
- 32位x86处理器：
	- 操作模式：
		- 保护模式（Protected Mode）：为程序分配独立的地址空间，支持多任务和内存保护，是现代操作系统的基础
		- 实地址模式（Real-address Mode）：仅支持单任务可以访问全部内存的模式，用于早期操作系统
		- 系统管理模式（System Management Mode）：用于处理系统级任务，如电源管理和硬件控制
		- 虚拟8086模式（Virtual 8086 Mode）：在保护模式下模拟8086处理器的运行环境，允许运行16位实模式程序
	- 基本执行环境：
		- 内存寻址：保护模式下使用32位地址，支持4GB地址空间；实地址模式和虚拟8086模式下使用20位地址，支持1MB地址空间
		- 通用寄存器：
			- 32位通用寄存器：参见[[L2.1 机器指令-基础#^a82ee7|计算机组成原理——通用寄存器]]
			- 16位段寄存器：`CS`（代码段寄存器）、`DS`（数据段寄存器）、`SS`（堆栈段寄存器）、`ES`（附加段寄存器）、`FS`、`GS`，另有`EFLAGS`（标志寄存器）用于存储条件码和控制位，`EIP`（指令指针寄存器）用于存储下一条要执行的指令地址
			- 寄存器也可以用不同的标志进行部分访问，如`AX`表示`EAX`的低16位，`AL`表示`EAX`的低8位，`AH`表示`EAX`的高8位
		- 索引与基址寄存器：仅下半部分有部分访问
			- `SI`（源变址寄存器）：用于字符串和数组操作，指向源数据地址，为`ESI`的低16位
			- `DI`（目的变址寄存器）：用于字符串和数组操作，指向目的数据地址，为`EDI`的低16位
			- `BP`（基指针寄存器）：指向当前栈帧的基地址，为`EBP`的低16位
			- `SP`（堆栈指针寄存器）：指向当前栈顶，为`ESP`的低16位
		- 状态位：
			- 进位标志（Carry）：无符号算术运算超出范围时置位
			- 溢出标志（Overflow）：有符号算术运算超出范围时置位
			- 符号标志（Sign）：运算结果为负数时置位
			- 零标志（Zero）：运算结果为零时置位
			- 辅助进位标志（Auxiliary Carry）：从第 3 位向第 4 位有进位时置位
			- 奇偶标志（Parity）：运算结果中 1 的个数为偶数时置位
		- **浮点、MMX、XMM 寄存器**：
		    - 8 个 80 位浮点数据寄存器：ST (0)~ST (7)，按栈结构排列，用于所有浮点算术运算
		    - 8 个 64 位 MMX 寄存器
		    - 8 个 128 位 XMM 寄存器，用于单指令多数据（SIMD）操作
	- 寻址模式：
		- 实地址模式：20位地址，无限制，仅支持单任务
			- 段地址计算：物理地址 = 段寄存器值 × 16 + 偏移地址
		- 保护模式：32位地址，支持多任务和内存保护
			- 线性地址计算：线性地址 = 段选择子 × 4 + 偏移地址
			- 物理地址计算：通过分页机制将线性地址映射到物理地址
- 64位x86处理器（x86-64）：
	- 扩展的寄存器集：
		- 16 个 64 位通用寄存器：`RAX`、`RBX`、`RCX`、`RDX`、`RSI`、`RDI`、`RBP`、`RSP`、`R8`~`R15`
		- 每个寄存器都可以按不同大小访问，如 `EAX`（低 32 位）、`AX`（低 16 位）、`AL`（低 8 位）、`AH`（高 8 位）
	- 扩展的寻址模式：
		- 支持 64 位地址空间，允许直接访问更大的内存
		- 引入了 RIP 相对寻址模式，使得代码位置无关
	- 新增的指令集扩展：
		- SSE、AVX 等 SIMD 指令集扩展，用于加速多媒体和科学计算
		- 支持更高效的浮点运算和并行处理
# 汇编语言基础
- 标准程序模板：
```asm
; Program Template (Template.asm) 
.386 ; 指定使用80386处理器指令集 
.model flat,stdcall ; 定义内存模型为flat，调用约定为stdcall 
.stack 4096 ; 分配4096字节的栈空间 ExitProcess PROTO, dwExitCode:DWORD ; 声明ExitProcess函数原型 

.data ; 数据段，用于声明变量 
; declare variables here 

.code ; 代码段，用于编写程序指令 
main PROC ; 主过程开始 
; write your code here 
INVOKE ExitProcess,0 ; 调用ExitProcess函数，结束程序 
main ENDP ; 主过程结束 
; (insert additional procedures here) 
END main ; 指定程序入口为main过程
```
- 基本元素：
	- 整数：
		- 可带有前导的+或-号
		- 支持不同进制，16进制以`h`结尾（以字母开头需加前导0，如`0Ah`），10进制无后缀，8进制以`o`结尾，2进制以`b`结尾
	- 整数表达式：运算遵循对应的优先级规则
	- 字符与字符串：
		- 字符用单引号或双引号括起，占1字节
		- 字符串也用单引号或双引号括起，每个字符占1字节
		- 可以嵌入引号
	- 保留字与标识符：
		- 保留字：汇编语言中具有特殊含义的词汇（如指令、伪指令等），不能用作变量名或过程名
		- 标识符：用于命名变量、过程等，自定义名称，以字母、下划线、@、或\$开头，后续字符可以是字母、数字、下划线、@、\$，不区分大小写
	- 伪指令：
		- 用于指导汇编器如何处理代码和数据，不生成机器代码
		- 例子：`.data`、`.code`、`.model`、`.stack`等
	- 指令：
		- 汇编语言的核心，用于执行具体操作
		- 包括数据传输指令、算术运算指令、逻辑运算指令、控制转移指令等
		- 结构：\[标签:\] 操作码 操作数(s) ; 注释\]
	- 标签：
		- 用于标识代码或数据的位置
		- 以冒号结尾，可作为跳转目标或数据引用
	- 助记符和操作数：
		- 指令助记符：指令的名称，如`MOV`、`ADD`、`JMP`等
		- 操作数：指令作用的对象，可以是寄存器、内存地址、立即数等
	- 注释：
		- 用于解释代码，增强可读性
		- 以分号（`;`）开头，直到行尾
		- 多行注释以`COMMENT`和用户选择的字符为开头和结尾
- 示例代码：
	```asm
	TITLE Add and Subtract (AddSubAlt.asm) 
	; This program adds and subtracts 32-bit integers. 
	.386 
	.MODEL flat,stdcall 
	.STACK 4096 
	ExitProcess PROTO, dwExitCode:DWORD 
	DumpRegs PROTO 
	
	.code 
	main PROC 
	mov eax,10000h ; EAX = 10000h（将10000h送入EAX寄存器） 
	add eax,40000h ; EAX = 50000h（EAX寄存器值加40000h） 
	sub eax,20000h ; EAX = 30000h（EAX寄存器值减20000h） 
	call DumpRegs ; 调用DumpRegs函数显示寄存器状态 
	INVOKE ExitProcess,0 ; 调用ExitProcess函数结束程序 
	main ENDP 
	END main
	```
	- 输出：![[Pasted image 20251230105622.png]]
- 程序的编译过程：
	- 示意图：![[Pasted image 20251230105820.png]]
	- 步骤：
		1. 编写汇编源代码文件（.asm）
		2. 使用汇编器（如MASM）将源代码汇编为目标文件（.obj）
		3. 使用链接器（如LINK）将目标文件与所需的库文件链接，生成可执行文件（.exe）
	- 列表文件：包含源码、机器码和地址信息的文件，便于调试

- 数据定义：
	- 数据类型：
		- 整数类型：

		| 数据类型   | 位数  | 类型  | 说明        |
		| ------ | --- | --- | --------- |
		| BYTE   | 8   | 无符号 | 8 位无符号整数  |
		| SBYTE  | 8   | 有符号 | 8 位有符号整数  |
		| WORD   | 16  | 无符号 | 16 位无符号整数 |
		| SWORD  | 16  | 有符号 | 16 位有符号整数 |
		| DWORD  | 32  | 无符号 | 32 位无符号整数 |
		| SDWORD | 32  | 有符号 | 32 位有符号整数 |
		| QWORD  | 64  | 整数  | 64 位整数    |
		| TBYTE  | 80  | 整数  | 80 位整数    |

		- 实数类型：
		
		| 数据类型   | 字节数 | 类型        | 说明              |
		| ------ | --- | --------- | --------------- |
		| REAL4  | 4   | IEEE 短实数  | 4 字节 IEEE 短实数   |
		| REAL8  | 8   | IEEE 长实数  | 8 字节 IEEE 长实数   |
		| REAL10 | 10  | IEEE 扩展实数 | 10 字节 IEEE 扩展实数 |

	- 定义语句：
		- 语法：`[name] DATA_TYPE initial_value [, initial_value ...]`
		- 例：`myByte BYTE 10`、`myArray WORD 1, 2, 3, 4`
	- 定义示例：
		- `BYTE`数据：
		```asm
		value1 BYTE 'A' ; 字符常量，存储'A'的ASCII码 
		value2 BYTE 0 ; 最小的无符号字节值 
		value3 BYTE 255 ; 最大的无符号字节值 
		value4 SBYTE -128 ; 最小的有符号字节值 
		value5 SBYTE +127 ; 最大的有符号字节值
		 value6 BYTE ? ; 未初始化的字节
		```
		- 字节数组：
		```asm
		list1 BYTE 10,20,30,40 ; 包含4个字节的数组 
		list2 BYTE 10,20,30,40 
			  BYTE 50,60,70,80 ; 续行定义数组 
			  BYTE 81,82,83,84 
		list3 BYTE ?,32,41h,00100010b ; 包含未初始化和不同进制值的数组 
		list4 BYTE 0Ah,20h,'A',22h ; 包含十六进制、ASCII码的数组
		```
		- 字符串：
		```asm
		str1 BYTE 'Hello, World!',0 ; 以空字符结尾的字符串
		str2 BYTE "Assembly Language",0 ; 使用双引号的字符串
		str3 BYTE 'A','E','I','O','U' ; 无空终止的字符数组
		greeting BYTE "Welcome to the Encryption Demo program "
		         BYTE "created by Kip Irvine.",0 ; 多行字符串
		menu BYTE "Checking Account",0dh,0ah,0dh,0ah, 
				  "1. Create a new account",0dh,0ah, 
				  "2. Open an existing account",0dh,0ah, 
				  "3. Credit the account",0dh,0ah, 
				  "4. Debit the account",0dh,0ah, 
				  "5. Exit",0ah,0ah, "Choice> ",0 ; 跨多行字符串
		str4 BYTE "Enter your name: ",0Dh,0Ah 
			 BYTE "Enter your address: ",0
			newLine BYTE 0Dh,0Ah,0 ; 定义换行符常量
		```
		- `DUP`运算符：用于分配存储空间，语法为`count DUP(value)`
		```asm
		var1 BYTE 20 DUP(0) ; 20个字节，均初始化为0 
		var2 BYTE 20 DUP(?) ; 20个未初始化的字节 
		var3 BYTE 4 DUP("STACK") ; 20个字节，存储"STACKSTACKSTACKSTACK" 
		var4 BYTE 10, 3 DUP(0), 20 ; 5个字节，值为10、0、0、0、20
		```
		- `WORD`和`SWORD`数据：
		```asm
		word1 WORD 65535 ; 最大的无符号16位整数 
		word2 SWORD –32768 ; 最小的有符号16位整数 
		word3 WORD ? ; 未初始化的无符号16位整数 
		word4 WORD "AB" ; 存储'A'和'B'的ASCII码（双字符） 
		myList WORD 1,2,3,4,5 ; 16位整数数组 
		array WORD 5 DUP(?) ; 未初始化的16位整数数组
		```
		- `DWORD`和`SDWORD`数据：
		```asm
		val1 DWORD 12345678h ; 无符号32位整数
		val2 SDWORD –2147483648 ; 最小的有符号32位整数 
		val3 DWORD 20 DUP(?) ; 未初始化的无符号32位整数数组 
		val4 SDWORD –3,–2,–1,0,1 ; 有符号32位整数数组
		```
		- `QWORD`等实数数据：
		```asm
		quad1 QWORD 1234567812345678h ; 64位整数 
		val1 TBYTE 1000000000123456789Ah ; 80位整数 
		rVal1 REAL4 -2.1 ; 4字节IEEE短实数 
		rVal2 REAL8 3.2E-260 ; 8字节IEEE长实数 
		rVal3 REAL10 4.6E+4096 ; 10字节IEEE扩展实数 
		ShortArray REAL4 20 DUP(0.0) ; 20个4字节IEEE短实数数组，初始化为0.0
		```
	- 字节顺序：参见[[L1.1 信息-整数的表示#^83b008|计算机组成原理——字节顺序]]
	- 例：添加变量定义的示例程序：
		```asm
		TITLE Add and Subtract, Version 2 (AddSub2.asm) 
		; This program adds and subtracts 32-bit unsigned 
		; integers and stores the sum in a variable. 
		INCLUDE Irvine32.inc 
		
		.data 
		val1 DWORD 10000h ; 定义无符号32位整数val1 
		val2 DWORD 40000h ; 定义无符号32位整数val2 
		val3 DWORD 20000h ; 定义无符号32位整数val3 
		finalVal DWORD ? ; 定义未初始化的无符号32位整数finalVal，用于存储结果 
		
		.code 
		main PROC 
		mov eax,val1 ; 将val1的值（10000h）送入EAX寄存器 
		add eax,val2 ; EAX寄存器值加val2的值（40000h），结果为50000h 
		sub eax,val3 ; EAX寄存器值减val3的值（20000h），结果为30000h 
		mov finalVal,eax ; 将EAX寄存器中的结果（30000h）存入finalVal 
		call DumpRegs ; 调用DumpRegs函数显示寄存器状态 exit ; 退出程序 
		main ENDP 
		END main
		```
- 符号常量：
	- 等号伪指令：
		- 语法：`name = expression`
		- 用于定义符号常量，表达式的值在汇编时计算
		- 例：
			```asm
			COUNT = 500 ; 定义符号常量COUNT为500 
			mov al,COUNT ; 将COUNT的值（500）送入AL寄存器
			```
	- 计算数组和字符串大小：
		- 字节数组大小计算：利用当前位置寄存器`$`与数组名的差值
		```asm
		list BYTE 10,20,30,40 ; 字节数组list 
		ListSize = ($ - list) ; 计算list的大小，结果为4
		```
		- 字数组大小计算：类似字节数组
		```asm
		list WORD 1000h,2000h,3000h,4000h ; 字数组list 
		ListSize = ($ - list) / 2 ; 计算list的大小，结果为4
		```
		- 双字数组大小计算：类似字节数组
		```asm
		list DWORD 1,2,3,4 ; 双字数组list 
		ListSize = ($ - list) / 4 ; 计算list的大小，结果为4
		```
	- `EQU`伪指令：
		- 语法：`name EQU expression`
		- 与等号伪指令类似，用于定义符号常量
		- 例：
			```asm
			PI EQU <3.1416> ; 定义符号PI为文本表达式3.1416 
			pressKey EQU <"Press any key to continue...",0> ; 定义符号pressKey为字符串表达式 
			.data 
			prompt BYTE pressKey ; 使用pressKey定义字符串变量prompt
			```
	- `TEXTEQU`伪指令：
		- 语法：`name TEXTEQU "string"`
		- 用于定义字符串常量
		- 例：
			```asm
			continueMsg TEXTEQU <"Do you wish to continue (Y/N)?"> ; 定义文本宏continueMsg 
			rowSize = 5 ; 定义符号常量rowSize为5 
			.data 
			prompt1 BYTE continueMsg ; 使用continueMsg定义字符串变量prompt1 
			count TEXTEQU %(rowSize * 2) ; 计算表达式rowSize * 2，定义为文本宏count 
			setupAL TEXTEQU <mov al,count> ; 定义文本宏setupAL为指令mov al,count 
			.code 
			setupAL ; 展开为mov al,10（因为count=5*2=10）
			```
- 64位系统编程：
	- 不允许使用`invoke`、`addr`等伪指令
	- 例：
		```asm
		; AddTwoSum_64.asm - Chapter 3 example. 
		ExitProcess PROTO ; 声明ExitProcess函数原型 
		
		.data 
		sum QWORD 0 ; 定义64位整数sum，初始化为0 
		
		.code 
		main PROC 
		mov rax,5 ; 将5送入64位寄存器rax 
		add rax,6 ; rax寄存器值加6，结果为11 
		mov sum,rax ; 将rax中的结果（11）存入sum 
		mov ecx,0 ; 将ecx寄存器置0 
		call ExitProcess ; 调用ExitProcess函数结束程序 
		main ENDP 
		END ; 指定程序入口（默认main）
		```
# 基本汇编伪指令
## 数据传输指令
- 操作数类型：
	- 立即数：常量值，如`5`、`0FFh`
	- 寄存器：如`EAX`、`EBX`
	- 内存地址：变量名或标签，如`myVar`、`array[4]`
- 指令操作数表示法：

	|符号|描述|
	|---|---|
	|r8|8 位通用寄存器，如 AH、AL、BH、BL 等|
	|r16|16 位通用寄存器，如 AX、BX、CX、DX 等|
	|r32|32 位通用寄存器，如 EAX、EBX、ECX、EDX 等|
	|reg|任意通用寄存器|
	|sreg|16 位段寄存器，如 CS、DS、SS、ES、FS、GS|
	|imm|8 位、16 位或 32 位立即数|
	|imm8|8 位立即字节值|
	|imm16|16 位立即字值|
	|imm32|32 位立即双字值|
	|r/m8|8 位操作数，可为 8 位通用寄存器或内存字节|
	|r/m16|16 位操作数，可为 16 位通用寄存器或内存字|
	|r/m32|32 位操作数，可为 32 位通用寄存器或内存双字|
	|mem|8 位、16 位或 32 位内存操作数|


