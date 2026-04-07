- SQL是一种综合型的数据库语言，功能包括：
	- 数据定义
	- 数据查询
	- 数据操作
	- 数据控制
	- ……
# 数据定义语言(DDL)
- 功能：定义关系表的**模式名、属性、属性域、约束**（$R = (A, D, dom, F)$）
	- $R$：关系名
	- $A$：属性集
	- $D$：属性域
	- $dom$：从$A$到$D$的映射
	- $F$：约束集
- 典型数据类型：
	- `char(n)`：定长字符串，长度为n
	- `varchar(n)`：变长字符串，长度不超过n
	- `int`：整数
	- `smallint`：小整数，占用空间较小
	- `numeric(p,d)`：定点数，p为总位数，d为小数位数
	- `real/double precision`：浮点数，精度较高
	- `float(n)`：浮点数，n为精度
	- `date`：日期
	- ……
- 建表语句：`CREATE TABLE`
	- 基本语法：
		```sql
		create table r ( 
			A₁ D₁, A₂ D₂, ..., Aₙ Dₙ,
			(integrity-constraint₁), 
			..., 
			(integrity-constraintₖ) 
		)
		```
		- `r`：关系名，也即表名
		- `A₁, A₂, ..., Aₙ`：属性名（列名）
		- `D₁, D₂, ..., Dₙ`：属性的数据类型
		- `integrity-constraint`：完整性约束，常见约束有：
			- `NOT NULL`：属性值不能为空
			- `UNIQUE`：属性值必须唯一
			- `PRIMARY KEY (A₁, A₂, ..., Aₖ)`：指定一个或多个属性作为主键，主键值必须唯一且不能为空
			- `FOREIGN KEY (A₁, A₂, ..., Aₖ) REFERENCES r₂(B₁, B₂, ..., Bₖ)`：指定一个或多个属性作为外键，外键值必须在另一个表中存在
	- 例：
		- 简单表：
			```sql
			create table department (
				dept_name char(20),
				building char(15), 
				budget numeric(12,2) 
			);
			```
		- 包含约束的表：
			```sql
			-- 教师表 
			create table instructor ( 
				ID char(5), 
				name varchar(20) not null, 
				dept_name varchar(20), 
				salary numeric(8,2), 
				primary key (ID), 
				foreign key (dept_name) references department 
			); 
			-- 学生表 
			create table student ( 
				ID varchar(5), 
				name varchar(20) not null, 
				dept_name varchar(20), 
				tot_cred numeric(3,0), 
				primary key (ID), 
				foreign key (dept_name) references department 
			); 
			-- 选课表（复合主键） 
			create table takes ( 
				ID varchar(5), 
				course_id varchar(8), 
				sec_id varchar(8), 
				semester varchar(6), 
				year numeric(4,0), 
				grade varchar(2), 
				primary key (ID, course_id, sec_id, semester, year), 
				foreign key (ID) references student, 
				foreign key (course_id, sec_id, semester, year) references section 
			);
			```
- 删表与改表语句：
	- 删表语句：`DROP TABLE r`（`r`为表名）
		- 注：`DROP`语句会**删除整个表的数据以及表的结构**，而`DELETE`语句只会删除表中的数据，表结构仍然存在
	- 改表语句：
		- 新增列：`ALTER TABLE r ADD A D`
			- `A`：新增的属性名
			- `D`：新增属性的数据类型
			- 注：此时，表中现有元组的新列默认值为`NULL`
		- 删除列：`ALTER TABLE r DROP A`
			- `A`：要删除的属性名
			- 有些数据库不支持删除列，因为列可能有约束或依赖关系