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
		- 状态位： ^235954
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

- 直接内存操作数：对内存中存储位置的命名引用
	- 例：
		```asm
		 .data 
		 var1 BYTE 10h 
		 
		 .code 
		 mov al,var1 ; AL = 10h，直接使用标签 
		 mov al,[var1] ; AL = 10h，方括号为另一种格式，效果相同
		```
- `MOV`指令：
	- 语法：`MOV destination, source`
	- 要求：
		1. 不允许有超过一个内存操作数
		2. `CS`、`EIP`、`IP`寄存器不能作为目的操作数
		3. 不允许立即数传送到段寄存器
		- 例：
			```asm
			 .data 
			 count BYTE 100 
			 wVal WORD 2 
			 .code 
			 mov al,wVal ; 错误，操作数位数不匹配 
			 mov ax,count ; 错误，操作数位数不匹配 
			 mov eax,count ; 错误，操作数位数不匹配
			```
- 零扩展与符号扩展
	1. 零扩展（Zero Extension）：
		- 将较小的数据类型扩展为较大的数据类型时，在高位填充零
		- 适用于无符号数据类型
		- 例：
			```asm
			 mov bl,10001111b 
			 movzx ax,bl ; 执行后AX的高8位为0，低8位为10001111b
			```
	2. 符号扩展（Sign Extension）：
		- 将较小的数据类型扩展为较大的数据类型时，保持符号位不变
		- 适用于有符号数据类型
		- 例：
			```asm
			 mov bl,10001111b 
			 movsx ax,bl ; 执行后AX的高8位与bl的最高位（1）相同，即11111111b，低8位为10001111b
			```
- `XCHG`指令：
	- 语法：`XCHG destination, source`
	- 功能：交换两个操作数的值
	- 要求：
		1. 不允许有超过一个内存操作数
		2. `CS`、`EIP`、`IP`寄存器不能作为操作数
	- 例：
		```asm
		 .data 
		 val1 DWORD 12345678h 
		 val2 DWORD 9ABCDEF0h 
		 
		 .code 
		 mov eax,val1 ; EAX = 12345678h 
		 mov ebx,val2 ; EBX = 9ABCDEF0h 
		 xchg eax,ebx ; 交换EAX和EBX的值 
		 ; 执行后，EAX = 9ABCDEF0h，EBX = 12345678h
		```
- 直接偏移操作数：
	- 语法：`[base + index * scale + displacement]`
	- 组成部分：
		- `base`：基址寄存器，如`EAX`、`EBX`等
		- `index`：变址寄存器，如`ESI`、`EDI`等
		- `scale`：缩放因子，取值为1、2、4或8
		- `displacement`：位移量，可以是立即数或符号常量
	- 例：
		```asm
		 .data 
		 arrayB BYTE 10h,20h,30h,40h 
		 .code 
		 mov al,arrayB+1 ; AL = 20h，arrayB+1为有效地址 
		 mov al,[arrayB+1] ; 另一种格式，效果相同
		```
## 加减运算指令
- `INC`和`DEC`指令：
	- `INC`指令：`INC operand`
		- 功能：将操作数加1
		- 操作数可以为寄存器或内存地址
	- `DEC`指令：`DEC operand`
		- 功能：将操作数减1
		- 操作数可以为寄存器或内存地址
	- 例：
		```asm
		 .data 
		 myWord WORD 1000h 
		 myDword DWORD 10000000h 
		 
		 .code 
		 dec myWord ; myWord = 0FFFh 
		 inc myWord ; myWord = 1000h 
		 inc myDword ; myDword = 10000001h 
		 mov ax,00FFh 
		 inc ax ; AX = 0100h 
		 mov ax,00FFh 
		 inc al ; AX = 0000h（al加1后溢出为0，ax高8位不变）
		```
- `ADD`和`SUB`指令：
	- `ADD`指令：`ADD destination, source`
		- 功能：将源操作数加到目的操作数上，结果存储在目的操作数中
		- 操作数可以为寄存器或内存地址
	- `SUB`指令：`SUB destination, source`
		- 功能：将源操作数从目的操作数中减去，结果存储在目的操作数中
		- 操作数可以为寄存器或内存地址
	- 例：
		```asm
		 .data 
		 var1 DWORD 10000h 
		 var2 DWORD 20000h 
		 .code 
		 mov eax,var1 ; EAX = 00010000h 
		 add eax,var2 ; EAX = 00030000h 
		 add ax,0FFFFh ; EAX = 0003FFFFh 
		 add eax,1 ; EAX = 00040000h 
		 sub ax,1 ; EAX = 0003FFFFh
		```
- `NEG`指令：
	- 语法：`NEG operand`
	- 功能：对操作数取负值（即计算其二进制补码）
	- 操作数可以为寄存器或内存地址
	- 例：
		```asm
		 .data 
		 valB BYTE -1 
		 valW WORD +32767 
		 .code 
		 mov al,valB ; AL = -1 
		 neg al ; AL = +1 
		 neg valW ; valW = -32767
		```
- 算术表达式实现：通过上述指令组合实现复杂的算术表达式
	- 例：计算`Rval = -Xval + (Yval – Zval)`
		```asm
		 .data 
		 Rval DWORD ? 
		 Xval DWORD 26 
		 Yval DWORD 30 
		 Zval DWORD 40 
		 
		 .code 
		 mov eax,Xval 
		 neg eax ; EAX = -26（得到-Xval） 
		 mov ebx,Yval 
		 sub ebx,Zval ; EBX = -10（得到Yval - Zval） 
		 add eax,ebx ; EAX = -36（计算-Xval + (Yval - Zval)） 
		 mov Rval,eax ; 将结果存入Rval，Rval = -36
		```
	* 标志位的影响：算术逻辑指令会改变标志寄存器中的标志位：

		|标志位|含义|设置 / 清除条件|
		|---|---|---|
		|零标志（ZF）|目标操作数等于零时设置|结果为 0 时 ZF=1，否则 ZF=0|
		|符号标志（SF）|目标操作数为负时设置|结果为负时 SF=1（最高位为 1），为正时 SF=0（最高位为 0）|
		|进位标志（CF）|无符号值超出范围时设置|无符号数运算结果超出目标操作数位数范围时 CF=1，否则 CF=0|
		|溢出标志（OF）|有符号值超出范围时设置|有符号数运算结果超出目标操作数位数表示的有符号数范围时 OF=1，否则 OF=0|

		- 示例：
			1. 零标志（ZF）：
				```asm
				mov cx,1 
				sub cx,1 ; CX = 0，ZF = 1 
				mov ax,0FFFFh 
				inc ax ; AX = 0，ZF = 1 
				inc ax ; AX = 1，ZF = 0
				```
			2. 符号标志（SF）：
				```asm
				mov cx,0 
				sub cx,1 ; CX = -1，SF = 1 
				add cx,2 ; CX = 1，SF = 0 
				mov al,0 
				sub al,1 ; AL = 11111111b（-1），SF = 1 
				sub al,1 ; AL = 11111110b（-2），SF = 1 
				add al,2 ; AL = 00000000b（0），SF = 0 
				add al,2 ; AL = 00000010b（2），SF = 0
				```
			3. 进位标志（CF）：
				```asm
				mov al,0FFh 
				add al,1 ; CF = 1，AL = 00（无符号数0FFh+1=100h，超出8位范围） 
				mov al,0 
				sub al,1 ; CF = 1，AL = FF（无符号数0-1=-1，超出8位无符号数范围）
				```
			4. 溢出标志（OF）：
				```asm
				mov al,+127 
				add al,1 ; OF = 1，AL = 80h（有符号数127+1=128，超出8位有符号数最大范围127） 
				mov al,7Fh（即+127） 
				add al,1 ; OF = 1，AL = 80h（与上例二进制层面相同，结果超出范围）
				```
## 数据相关运算符和指令
- `OFFSET`运算符：
	- 语法：`OFFSET name`
	- 功能：返回变量或标签在数据段中的偏移地址，保护模式下为32位，实地址模式下为16位
	- 例：
		```asm
		 .data 
		 bVal BYTE ? 
		 wVal WORD ? 
		 dVal DWORD ? 
		 dVal2 DWORD ? 
		 
		 .code 
		 mov esi,OFFSET bVal ; ESI = 00404000（bVal在段起始处） 
		 mov esi,OFFSET wVal ; ESI = 00404001（bVal占1字节，wVal在其之后） 
		 mov esi,OFFSET dVal ; ESI = 00404003（wVal占2字节，dVal在其之后） 
		 mov esi,OFFSET dVal2 ; ESI = 00404007（dVal占4字节，dVal2在其之后）
		```
- `PTR`运算符：
	- 语法：`PTR TYPE name`
	- 功能：用于覆盖标签（变量）的默认类型，灵活访问其内容
	- 例：一个双字变量的不同访问方式
		```asm
		 .data 
		 myDouble DWORD 12345678h（小端存储时，内存地址0000存78h，0001存56h，0002存34h，0003存12h） 
		 .code 
		 mov ax,myDouble ; 错误，myDouble默认是双字（4字节），ax是字（2字节），类型不匹配 
		 mov ax,WORD PTR myDouble ; 正确，加载myDouble的低2字节，AX = 5678h 
		 mov WORD PTR myDouble,4321h ; 正确，将4321h存入myDouble的低2字节 
		 mov al,BYTE PTR myDouble ; AL = 78h（访问myDouble的第1字节） 
		 mov al,BYTE PTR [myDouble+1] ; AL = 56h（访问myDouble的第2字节） 
		 mov al,BYTE PTR [myDouble+2] ; AL = 34h（访问myDouble的第3字节） 
		 mov ax,WORD PTR [myDouble+2] ; AX = 1234h（访问myDouble的高2字节）
		```
- `TYPE`运算符：
	- 语法：`TYPE name`
	- 功能：返回变量或数据类型的大小（以字节为单位）
	- 例：
		```asm
		 .data 
		 var1 BYTE ? 
		 var2 WORD ? 
		 var3 DWORD ? 
		 var4 QWORD ? 
		 
		 .code 
		 mov eax,TYPE var1 ; EAX = 1（BYTE类型占1字节） 
		 mov eax,TYPE var2 ; EAX = 2（WORD类型占2字节） 
		 mov eax,TYPE var3 ; EAX = 4（DWORD类型占4字节） 
		 mov eax,TYPE var4 ; EAX = 8（QWORD类型占8字节）
		```
- `LENGTHOF`运算符：
	- 语法：`LENGTHOF name`
	- 功能：返回数组中元素的数量
	- 例：
		```asm
		 .data 
		 byte1 BYTE 10,20,30 ; 3个元素，LENGTHOF byte1 = 3 
		 array1 WORD 30 DUP(?),0,0 ; 30个空元素加2个0，共32个元素，LENGTHOF array1 = 32 
		 array2 WORD 5 DUP(3 DUP(?)) ; 5组，每组3个空元素，共15个元素，LENGTHOF array2 = 15 
		 array3 DWORD 1,2,3,4 ; 4个元素，LENGTHOF array3 = 4 digitStr BYTE "12345678",0 ; 8个字符加1个结束符，共9个元素，LENGTHOF digitStr = 9 
		 .code 
		 mov ecx,LENGTHOF array1 ; ECX = 32
		```
- `SIZEOF`运算符：
	- 语法：`SIZEOF name`
	- 功能：返回变量或数组所占的总字节数，等于`TYPE`乘以`LENGTHOF`
	- 例：
		```asm
		 .data 
		 byte1 BYTE 10,20,30 ; 3个字节，SIZEOF byte1 = 3 
		 array1 WORD 30 DUP(?),0,0 ; 32个字，每个2字节，SIZEOF array1 = 64 
		 array2 WORD 5 DUP(3 DUP(?)) ; 15个字，每个2字节，SIZEOF array2 = 30 
		 array3 DWORD 1,2,3,4 ; 4个双字，每个4字节，SIZEOF array3 = 16 
		 digitStr BYTE "12345678",0 ; 9个字节，SIZEOF digitStr = 9 
		 .code 
		 mov ecx,SIZEOF array1 ; ECX = 64
		```
- 跨多行数据声明：
	1. 若声明除最后一行外每行末尾都有逗号，则表示数据在继续声明，`LENGTHOF`和`SIZEOF`会计算所有行的数据
		```asm
		.data 
		array WORD 10,20, 
				30,40, 
				50,60 ; 共6个元素，总字节数12 
		.code 
		mov eax,LENGTHOF array ; EAX = 6 
		mov ebx,SIZEOF array ; EBX = 12
		```
	2. 若每行都是独立的声明，则每行视为单独的变量，`LENGTHOF`和`SIZEOF`只计算指定行的数据
		```asm
		.data 
		array WORD 10,20 ; 标签array仅对应这一行 
		WORD 30,40 
		WORD 50,60 
		
		.code 
		mov eax,LENGTHOF array ; EAX = 2（仅第一个WORD声明的元素数） 
		mov ebx,SIZEOF array ; EBX = 4（2×2，仅第一个WORD声明的总字节数）
		```
- `LABEL`指令：
	- 功能：为现有变量或标签指定新的数据类型，可以替代`PTR`运算符
	- 例：
		```asm
		 .data 
		 dwList LABEL DWORD ; 为intList分配DWORD类型的备用标签dwList 
		 wordList LABEL WORD ; 为intList分配WORD类型的备用标签wordList 
		 intList BYTE 00h,10h,00h,20h ; 原始存储位置，4字节 
		 
		 .code 
		 mov eax,dwList ; EAX = 20001000h（按DWORD类型访问intList） 
		 mov cx,wordList ; CX = 1000h（按WORD类型访问intList的前2字节） 
		 mov dl,intList ; DL = 00h（按BYTE类型访问intList的第1字节）
		```
## 间接寻址
- 间接操作数：存储变量的地址，类同指针，可以解引用
	- 例：
		```asm
		 .data 
		 val1 BYTE 10h,20h,30h 
		 
		 .code 
		 mov esi,OFFSET val1 ; 将val1的地址存入esi 
		 mov al,[esi] ; 解引用esi，AL = 10h（获取val1第一个元素） 
		 inc esi ; esi指向val1下一个元素地址 
		 mov al,[esi] ; AL = 20h（获取val1第二个元素） 
		 inc esi ; esi指向val1第三个元素地址 
		 mov al,[esi] ; AL = 30h（获取val1第三个元素）
		```
	- 注：需用`PTR`指定间接操作数的数据类型
		```asm
		 .data 
		 myCount WORD 0 
		 .code 
		 mov esi,OFFSET myCount 
		 inc [esi] ; 错误，无法确定操作数大小 
		 inc WORD PTR [esi] ; 正确，明确为WORD类型操作数
		```
- 示例：数组求和（使用间接寻址）
	```asm
	.data 
	arrayW WORD 1000h,2000h,3000h ; 16位数组 
	
	.code 
	mov esi,OFFSET arrayW ; esi指向数组起始地址 
	mov ax,[esi] ; 取第一个元素，AX = 1000h 
	add esi,2 ; 数组元素为WORD类型（2字节），esi加2指向第二个元素（也可写为add esi,TYPE arrayW） 
	add ax,[esi] ; 加第二个元素，AX = 3000h 
	add esi,2 ; esi加2指向第三个元素 
	add ax,[esi] ; 加第三个元素，AX = 6000h（数组总和）
	```
	- 若数组为双字`DWORD`类型，则每次加4
- 变址操作数：
	- 将常量和寄存器相加
	- 语法：`[label + reg]`或`label[reg]`
	- 例：
		```asm
		 .data 
		 arrayW WORD 1000h,2000h,3000h 
		 
		 .code 
		 mov esi,0 ; 初始索引为0 
		 mov ax,[arrayW + esi] ; AX = 1000h（取第一个元素） 
		 mov ax,arrayW[esi] ; 另一种格式，效果相同 
		 add esi,2 ; 索引加2（WORD类型），指向第二个元素 
		 add ax,[arrayW + esi] ; 加第二个元素，AX = 3000h ; 继续操作可完成数组遍历或计算
		```
- 索引比例：
	- 通过将索引乘以数组的`TYPE`，对间接或变址操作数进行比例调整，以获取数组元素的偏移量
	- 例：
		```asm
		 .data 
		 arrayB BYTE 0,1,2,3,4,5 ; 8位数组，TYPE=1 
		 arrayW WORD 0,1,2,3,4,5 ; 16位数组，TYPE=2 
		 arrayD DWORD 0,1,2,3,4,5 ; 32位数组，TYPE=4 
		 
		 .code 
		 mov esi,4 ; 索引为4 
		 mov al,arrayB[esi * TYPE arrayB] ; esi*1=4，AL = 4（取arrayB第5个元素） 
		 mov bx,arrayW[esi * TYPE arrayW] ; esi*2=8，BX = 0004h（取arrayW第5个元素） 
		 mov edx,arrayD[esi * TYPE arrayD] ; esi*4=16，EDX = 00000004h（取arrayD第5个元素）
		```
- 指针：
	- 可以声明指针变量存储另一个变量的偏移量，从而实现间接寻址
	- 例：
		```asm
		 .data 
		 arrayW WORD 1000h,2000h,3000h 
		 ptrW DWORD arrayW ; ptrW存储arrayW的偏移量（也可写为ptrW DWORD OFFSET arrayW） 
		 
		 .code 
		 mov esi,ptrW ; 将ptrW存储的偏移量存入esi，即esi指向arrayW 
		 mov ax,[esi] ; AX = 1000h（取arrayW第一个元素）
		```
## 跳转与循环指令
- `JMP`指令：
	- 语法：`JMP label`
	- 功能：无条件跳转到指定标签处继续执行
	- 例：
		```asm
		 top: 
		 ; 此处可添加指令 
		 jmp top ; 无条件跳转到top标签，形成无限循环
		```
- `LOOP`指令：
	- 语法：`LOOP label`
	- 功能：将`ECX`寄存器减1，若结果不为0，则跳转到指定标签处继续执行
	- 例：计算倒序和`5 + 4 + 3 + 2 + 1`：
		```asm
		 .data 
		 sum DWORD 0 ; 存储结果的变量 
		 
		 .code 
		 mov ecx,5 ; 设置循环计数器为5 
		 mov eax,0 ; 初始化累加器EAX为0 
		 
		 sumLoop: 
		 add eax,ecx ; 将ECX的值加到EAX中 
		 loop sumLoop ; 循环直到ECX减到0 
		 
		 mov sum,eax ; 将结果存入sum变量
		```
		- 原理：每次执行`loop`指令时，`ECX`减1，直到`ECX`为0时跳出循环；由汇编器计算跳转偏移量
- 嵌套循环：
	- 如果循环发生嵌套，需要保存外层循环的`ECX`值
	- 方法：使用堆栈保存和恢复`ECX`
	- 例：
		```asm
		 .data 
		 count DWORD ? ; 用于保存外层循环计数器 
		 .code 
		 mov ecx,100 ; 设置外层循环次数为100 
		 L1: 
		 mov count,ecx ; 保存外层循环的ECX值 
		 mov ecx,20 ; 设置内层循环次数为20 
		 L2: 
		 ; 内层循环执行的指令 
		 loop L2 ; 内层循环，ECX自减1，非零则跳回L2 
		 mov ecx,count ; 恢复外层循环的ECX值 
		 loop L1 ; 外层循环，ECX自减1，非零则跳回L1
		```
	- 数组求和、字符串复制都是常见的嵌套循环应用
## 64位汇编指令集特性
- `MOV`指令特殊性：
	- 在64位模式下，`MOV`指令支持64位寄存器操作，也支持对应的32位、16位和8位寄存器
	- 低位数据传送到高位寄存器时，32位传输会将高32位清零，而16位和8位传输不会影响高位
- 其他指令特殊性：
	1. **MOVSXD 指令**：将 32 位值符号扩展为 64 位值，并存入 64 位目标寄存器。
	2. **OFFSET 运算符**：在 64 位模式下生成 64 位地址。
	3. **LOOP 指令**：使用 64 位 RCX 寄存器作为计数器（替代 32 位模式下的 ECX）。
	4. **索引寄存器**：RSI 和 RDI 是 64 位模式下访问数组最常用的 64 位索引寄存器。
	5. **加减运算与标志位**：ADD 和 SUB 指令对标志位的影响与 32 位模式相同。
	6. **索引比例**：64 位模式下，变址操作数仍可使用比例因子。
# 过程
- 参见[[L2.3 机器指令-过程|计算机组成原理——过程调用]]
- 运行时栈：
	- 参见[[L2.3 机器指令-过程#运行时栈的结构|计算机组成原理——运行时栈的结构]]
	- 核心指令：
		- `PUSH`：对32位，将栈指针减4，并将操作数存入栈顶
		- `POP`：对32位，将栈顶数据取出存入操作数，并将栈指针加4
		- `PUSHFD/POPFD`：将`EFLAGS`寄存器（也即标志位寄存器）内容压入/弹出栈
		- `PUSHAD/POPAD`：将所有通用寄存器内容压入/弹出栈
		- `PUSHA/POPA`：将16位模式下所有通用寄存器内容压入/弹出栈
	- 例：
		```asm
		 push esi ; 保存寄存器 
		 push ecx 
		 push ebx 
		 mov esi, OFFSET dwordVal ; 初始化操作 
		 mov ecx, LENGTHOF dwordVal 
		 mov ebx, TYPE dwordVal 
		 call DumpMem ; 调用过程 
		 pop ebx ; 恢复寄存器（逆序） 
		 pop ecx 
		 pop esi
		```
- 过程的创建与调用：
	- 创建过程：
		- 语法：
			```asm
			过程名 PROC ; 过程开始 
				; 过程执行代码 
				ret ; 返回指令，结束过程 
			过程名 ENDP ; 过程结束
			```
			需要包含功能描述，参数输入，返回值和过程的前置条件（寄存器、内存状态等）
		- 例：
			```asm
			; 过程定义 
			SumOf PROC 
			; 功能：计算3个32位整数的和，支持有符号/无符号数 
			; 输入：EAX、EBX、ECX（3个整数） 
			; 返回：EAX（和），状态标志（进位、溢出等）变化 
			; 前置条件：无 
				add eax, ebx ; 累加EBX到EAX 
				add eax, ecx ; 累加ECX到EAX 
				ret 
			SumOf ENDP 
			
			; 过程调用（准备参数后调用） 
			mov eax, 10 ; 第1个参数 
			mov ebx, 20 ; 第2个参数 
			mov ecx, 30 ; 第3个参数 
			call SumOf ; 调用过程 
			WriteDec ; 输出结果
			```
	- 调用和返回：`CALL`指令和`RET`指令
		- `CALL label`：将下一条指令地址压入栈，然后跳转到`label`处执行
		- `RET`：从栈顶弹出返回地址，跳转回调用点继续执行
	- 技术细节：
		- 嵌套过程：依靠栈的特性，支持多层嵌套调用
		- 局部标签和全局标签：过程内的标签默认为局部标签，过程外的标签为全局标签
		- 过程参数：通过寄存器或栈传递
		- `USES`运算符：用于声明过程使用的寄存器，自动保存和恢复
			```asm
			ArraySum PROC USES esi ecx ; 声明保存esi和ecx 
				mov eax, 0 ; 过程代码 
				; ...（省略循环逻辑） 
				ret 
			; MASM自动生成：push esi → push ecx（进入时）；pop ecx → pop esi（返回前） 
			ArraySum ENDP
			```
		- 注：**避免对存储返回值的寄存器执行`PUSH`或`POP`操作**，以免覆盖返回值
- 外部库链接：
	- 链接库：
		- 已编译为机器码的过程集合，可供程序调用（由`obj`文件构建）
		- 由`LIB`工具创建静态库文件（`.lib`）
	- 链接工作机制：
		- 程序需链接自定义库文件`Irvine32.lib`和系统库文件`kernel32.lib`
		- 系统库文件进一步会链接到`Kernel32.dll`动态链接库
	- 调用示例：使用`INCLUDE`指令
		```asm
		 INCLUDE Irvine32.inc ; 引入库原型 
		 .code 
		 mov eax, 1234h ; 准备输入参数（十六进制数） 
		 call WriteHex ; 调用库过程，显示EAX中的十六进制数 
		 call Crlf ; 调用库过程，换行
		```
- 64位过程调用：
	- `Irvine64`库为64位汇编程序提供常用过程
	- 调用约定：
		- 前四个整数参数通过`RCX`、`RDX`、`R8`、`R9`寄存器传递
		- 额外参数通过栈传递
		- 返回值通过`RAX`寄存器返回
		- 栈操作改为将栈指针`RSP`向下增长8字节，同时必须保证16字节对齐
# 条件处理
## 布尔量与比较指令
- [[Chapter 汇编语言#^235954|CPU的状态标志位]]
- 布尔运算指令
	- `AND`指令：
## 条件跳转


## 条件循环


## 条件控制流

# 整数运算

# 高级过程控制

# 字符串与数组

# 结构体与宏

# Windows API 编程