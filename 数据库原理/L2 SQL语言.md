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
- 表结构的存储：
	- 数据字典：数据库的元数据中枢，用于存储所有数据库对象的结构信息，也被称为系统目录（system catalog）或系统表（system tables）
		- 在PostgreSQL中，可以通过路径：`Catalogs -> pg_catalog -> Tables`查找系统表，同时提供标准的`information_schema`视图来查询表结构信息，包括：
			- `pg_class`：存储表、视图、序列等关系对象的元数据
			- `pg_attribute`：存储表的属性（列）的元数据
			- `pg_database`：存储所有数据库的全局信息
			- `pg_namespace`：存储命名空间（schema）的信息
	- 数据库表的层次结构：
		- 以PostgreSQL为例，数据库对象的层次结构如下：![[Pasted image 20260408154914.png]]
            - DBMS Instance：数据库管理系统实例，包含多个数据库，全局角色和全局表空间等
            - Catalog：数据库，实例下的独立逻辑单元，不同catalog下的数据库是物理隔离的
            - Schema：Database下的逻辑分组，用于隔离用户对象
            - Relation：Schema下的表、视图等关系对象
    - 数据库和表目录操作
	    - 数据库操作：
		    - 创建数据库：`CREATE DATABASE db_name`
            - 删除数据库：`DROP DATABASE db_name`
        - 表目录操作：
	        - 创建模式：`CREATE SCHEMA [schema_name] [AUTHORIZATION user_name]`
            - 删除模式：`DROP SCHEMA [schema_name] [CASCADE | RESTRICT]`
# SQL查询基本结构
- 数据操作语言（DML）：用于访问和操作数据库中数据的语言，也叫查询语言（query language）
	- 过程式查询语言（Procedural Query Language）：需要用户指定访问数据的具体步骤和方法
	- 非过程式查询语言（Non-Procedural Query Language）：用户只需指定需要什么数据，数据库系统会自动决定如何获取数据
- 基本结构：
    ```sql
    SELECT A₁, A₂, ..., Aₙ 
    FROM R₁, R₂, ..., Rₘ
    WHERE P;
    ```
    - 说明：
	    - `A₁, A₂, ..., Aₙ`：要查询的属性列表，可以是具体属性，也可以是`*`表示所有属性
	    - `R₁, R₂, ..., Rₘ`：要查询的关系（表）列表，可以是一个或多个关系
	    - `P`：查询条件，使用布尔表达式来指定筛选数据
	    - 对应的关系代数：$\Pi_{A₁, A₂, ..., Aₙ}(\sigma_P(R₁ \times R₂ \times ... \times Rₘ))$
		- 注：SQL语句中的关系指**多重集合**，而关系代数中的关系指**集合**，因此SQL查询结果可能包含重复的元组，而关系代数查询结果不包含重复的元组
			- 例：![[Pasted image 20260408160732.png]]
	- `SELECT`子句：对应投影操作，指定查询结果需要的列
		- 基础：`SELECT name FROM student;`，投影学生表中的`name`列
		- 去重：`SELECT DISTINCT name FROM student;`，去重后投影学生表中的`name`列
		- 留重：`SELECT ALL name FROM student;`，保留重复值投影学生表中的`name`列（默认行为）
		- 通配符`*`：`SELECT * FROM student;`，投影学生表中的所有列
		- 常量：
			- 无`FROM`：`SELECT 'Hello World' AS greeting;`，查询结果为一行一列，列名为`greeting`，值为`Hello World`
            - 有`FROM`：`SELECT 'Hello World' AS greeting FROM student;`，查询结果为与学生表中行数相同的多行，每行的`greeting`列值为`Hello World`
    - `FROM`子句：对应笛卡尔积操作，指定查询涉及的表
	    - `SELECT * FROM instructor, teaches`，查询`instructor`表和`teaches`表的笛卡尔积
	    - 同名属性需要用表名限定：`SELECT instructor.name, teaches.name FROM instructor, teaches;`，查询`instructor`表的`name`列和`teaches`表的`course_id`列
	- `WHERE`子句：对应选择操作，指定查询的筛选条件
        - 基础：`SELECT * FROM student WHERE dept_name = 'CS';`，查询学生表中`dept_name`为`CS`的所有列
        - 复杂条件：`SELECT * FROM student WHERE dept_name = 'CS' AND tot_cred > 30;`，查询学生表中`dept_name`为`CS`且`tot_cred`大于30的所有列
# 附加操作与空值
- 换名操作（Renaming）：