# 连接表达式
- 参见[[L2 SQL语言#连接操作|SQL基础——连接操作]]
# 视图
- 数据库在面向用户时，不需要提供完整的逻辑视角。用户只关心与其相关的数据，数据库可以提供一个**视图**（View）来满足用户的需求
	- 示意图：![[Pasted image 20260611200417.png]]
	- 视图提供了数据的逻辑独立性，并且保证了对特定用户隐藏特定的数据
- 视图的语法：
	- 定义：`CREATE VIEW <view_name> AS <select_statement>`
		- 视图本身不存储数据，本质是后面的查询语句的一个别名
	- 创建与删除视图：
	    - 创建视图：使用`CREATE VIEW`语句创建视图
	    - 删除视图：使用`DROP VIEW <view_name>`语句删除视图
		    - 级联删除：如果一个视图被其他视图依赖，删除该视图时需要使用`CASCADE`选项，自动删除所有依赖于该视图的视图
	- 带聚合函数的视图：
		- 视图定义中可以包含聚合函数，如`SUM`、`AVG`、`COUNT`等，用于计算某个属性的总和、平均值、计数等统计信息
		- 例：创建一个部门工资总和视图：
			```sql
			CREATE VIEW departments_total_salary(dept_name, total_salary) AS 
			SELECT dept_name, SUM(salary) 
			FROM instructor 
			GROUP BY dept_name;
			```
	- 视图的嵌套：
		- 视图的定义中可以调用其它的视图，形成视图的嵌套
		- 例：基础视图筛选物理系2009年秋季课程：
			```sql
			CREATE VIEW physics_fall_2009 AS 
			SELECT course.course_id, sec_id, building, room_number 
			FROM course, section 
			WHERE course.course_id = section.course_id 
			  AND course.dept_name = 'Physics' 
			  AND section.semester = 'Fall' 
			  AND section.year = '2009';
			```
			再筛选在`Watson`教学楼的课程：
			```sql
			CREATE VIEW physics_fall_2009_watson AS 
			SELECT course_id, room_number 
			FROM physics_fall_2009 
			WHERE building = 'Watson';
			```
	- 视图的展开算法：数据库应用视图展开算法，将嵌套视图替换成查询语句
		- 算法逻辑：
			1. 从最内层的视图开始，逐层展开
			2. 将视图定义中的`SELECT`语句替换到调用该视图的查询中
			3. 重复上述步骤，直到所有视图都被展开成基本表的查询为止
		-  例：上述嵌套视图会被展开成为：
			```sql
			CREATE VIEW physics_fall_2009_watson AS 
			SELECT course_id, room_number 
			FROM ( 
				SELECT course.course_id, building, room_number 
				FROM course, section 
				WHERE course.course_id = section.course_id 
					AND course.dept_name = 'Physics' 
					AND section.semester = 'Fall' 
					AND section.year = '2009' 
			) AS temp 
			WHERE building = 'Watson';
			```
	- 视图更新：
		- 视图的自动更新：当基础表中的数据发生变化时，视图的内容会自动反映这些变化，无需手动更新视图
			- 例：创建视图如下：
				```sql
				CREATE VIEW faculty AS 
				SELECT ID, name, dept_name 
				FROM instructor;
				```
				插入数据：
				```sql
				INSERT INTO faculty VALUES ('30765', 'Green', 'Music');
				```
				若允许不在视图中的字段为`NULL`，则视图会自动更新
		- 大部分的DBMS只允许简单视图更新，需要满足以下条件
			1. `FROM`子句只有一个数据库关系
				-  如果视图本身是多表关系，则更新时数据库不知道应该更新哪个表
			2. `SELECT`子句只有属性名，没有表达式，聚合函数和`DISTINCT`关键字
                -  如果视图中包含表达式或聚合函数，更新时数据库无法确定如何计算这些值
                - `DISTINCT`去重后将会引起视图和底层表的名称不一致
            3. 未在`SELECT`中列出的属性都可空
                -  如果视图中没有列出某些属性，且这些属性在底层表中不可空，那么更新时可能会违反底层表的约束条件
            4. 没有`GROUP BY`、`HAVING`子句的复杂查询
                -  如果视图中包含`GROUP BY`或`HAVING`子句，更新时数据库无法确定如何处理分组和聚合的结果
            - 总结：**单表，无聚合，无分组，无去重**
# 事务
- 参见[[L7 事务#SQL中的事务|事务——SQL中的事务]]
- 隔离级别
	- 参见[[L7 事务#隔离级别|事务——隔离级别]]
	- 隔离级别的设置方法：
		- SQL标准提供了`SET TRANSACTION ISOLATION LEVEL <isolation_level>`语句来设置事务的隔离级别
		- `<isolation_level>`可以是以下四种隔离级别之一：
			1. `READ UNCOMMITTED`：允许读取未提交的数据，可能会出现脏读、不可重复读和幻读
			2. `READ COMMITTED`：只能读取已提交的数据，避免了脏读，但仍可能出现不可重复读和幻读
			3. `REPEATABLE READ`：保证在同一事务中多次读取同一数据时结果一致，避免了不可重复读，但仍可能出现幻读
			4. `SERIALIZABLE`：最高级别的隔离，完全串行化执行事务，避免了所有并发问题，但性能较低
		- 也可以在事务开启时直接指定：
			```sql
			BEGIN TRANSACTION ISOLATION LEVEL <isolation_level>;
			```
	- PostgreSQL中的事务实操：![[Pasted image 20260611204802.png]]
# 完整性约束
- 完整性约束：对数据库授权更改的一种约束，确保数据库的数据一致性来防止数据库遭受意外损坏
	- 目的是为了防止用户的意外操作/逻辑错误
- 单表约束：对单个关系的约束
	- `NOT NULL`约束：确保某个属性的值不能为空
		- 例：`branch_name char(15) not null`，银行的支行名称不能为空
		- 强制某个值非空，确保数据完整性
	- `PRIMARY KEY`约束：定义表的主键，用于唯一标识表的每一行数据
		- 特点：列值非空（`NOT NULL`）且列值全局唯一，不能重复
        - 例：`branch_name char(15) primary key`，也即银行的支行名称既不能为空又必须唯一
        - 一张表只能有**一个主键**
	- `UNIQUE`（唯一性）约束：确保某个属性的值在表中唯一，但允许空值
		- 例：
			```sql
			create table department ( 
				dept_name varchar(20), 
				building varchar(15), 
				budget numeric(12,2), 
				unique (dept_name) 
			);
			```
		在`dept_name`中加唯一性约束，也即部门名称不允许重复，但可为空
		- 空值允许重复，但非空值必须唯一
	- `CHECK`约束：定义一个布尔表达式，限制属性值必须满足的条件
		- 语法：`check P`，其中P是一个布尔表达式，可以涉及一个或多个属性
		- 例：
			```sql
			create table branch ( 
				branch_name char(15), 
				branch_city char(30), 
				assets integer, 
				primary key (branch_name), 
				check (assets >= 0) );
			```
        在`assets`属性上加检查约束，确保银行的资产不能为负数
        - 违反`check`约束的插入或更新操作会被拒绝，保证数据的有效性和合理性
- 参照完整性约束：负责处理多表间的约束，保证一张表中列的值在另一张表中存在
	- 基本概念：
		- 被引用的表称为父表或主表，被引用的列一般是父表的主键
		- 引用的表称为子表或从表，引用的列称为外键
		- 例：![[Pasted image 20260612134914.png]]
	- 外键的定义：
	    - 语法：`FOREIGN KEY (<foreign_key_column>) REFERENCES <parent_table>(<parent_key_column>)`
		-  例：
			```sql
            -- 父表：被引用的表，主键是branch_name 
            create table branch ( 
	            branch_name char(15), 
		        branch_city char(30), 
		        assets integer, 
		        primary key (branch_name) 
		    ); 
		    -- 子表：引用父表的account表，定义外键 
		    create table account ( 
			    account_number char(10), 
			    branch_name char(15), 
			    balance integer, 
			    primary key (account_number), 
			    foreign key (branch_name) references branch 
			);
			```
		- 说明：
			- 省略父表列名时默认引用主键列
			- 子表的外键列必须和父表的被引用列一致
		- 形式化定义：
			- 父表记为$r_1$，其主键记为$K_1$
			- 子表记为$r_2$，其外键记为$\alpha$
			- 形式化定义为：$\Pi_{\alpha}(r_2) \subseteq \Pi_{K_1}(r_1)$，即子表的外键列值必须是父表主键列值的子集，也叫**子集依赖**
	- 外键约束下的检测：
		- `INSERT`操作：
			- 必须保证新插入的外键值在父表的主键中存在，也即$t_2[\alpha] \in \Pi_{K_1}(r_1)$，$t_2$是子表中的新元组
		- `DELETE`操作：
			- 先查询有没有引用，也即执行：$\sigma_{\alpha = t_1[K_1]}(r_2)$，$t_1$是父表中的被删除元组，$K_1$是父表的主键
			- 如果有子表中的引用，则根据设置的删除规则进行处理：
				- `RESTRICT`：拒绝删除父表中的元组，保持数据完整性
				- `CASCADE`：级联删除，同时删除子表中引用该父表元组的所有子表元组
		- `UPDATE`操作：
			- 更新子表外键值：类似`INSERT`操作，保证新的外键值和父表主键值一致
			- 更新父表的主键值：类似`DELETE`操作，先查询有没有引用，如果有则根据设置的更新规则进行处理：
				- `RESTRICT`：拒绝更新父表中的主键值，保持数据完整性
				- `CASCADE`：级联更新，同时更新子表中引用该父表主键值的所有子表元组的外键值
	- 级联操作与置空：
		- 级联操作的声明：在建表时进行声明
			```sql
			create table account ( 
				... 
				foreign key(branch_name) references branch 
				on delete cascade -- 删除父表数据时，级联删除子表数据 
				on update cascade -- 更新父表主键时，级联更新子表外键 ... 
			);
			```
			- `on delete cascade`：删除父表数据时，级联删除子表数据
			- `on update cascade`：更新父表主键时，级联更新子表外键
			- 若不写明删除/更新规则，默认使用`RESTRICT`
		- 置空/置为默认值操作：当父表数据被删除或更新时，将子表中的外键值置为`NULL`（或置为默认值）
			- `on delete set null`：删除父表数据时，将子表外键值置为`NULL`
			- `on delete set default`：删除父表数据时，将子表外键值置为默认值
			- 注：外键的值为`NULL`是合法的数值
	- 自引用外键：
		- 例：
			```sql
			create table partners ( 
				player1_name char(15) primary key, 
				player2_name char(15), 
				gender int, 
				age int, 
				foreign key (player2_name) references partners 
			);
			```
			此处`player2_name`是一个自引用外键，引用同一表中的主键`player1_name`
# 数据类型与模式
- 内嵌时间数据类型：
    - `DATE`：表示日期，格式为`YYYY-MM-DD`
	    - 例：`2024-06-12`表示2024年6月12日
    - `TIME`：表示时间，格式为`HH:MM:SS`
	    - 例：`14:30:30.75`表示14点30分30秒75毫秒，最后一位表示毫秒
    - `TIMESTAMP`：表示日期和时间的组合，格式为`YYYY-MM-DD HH:MM:SS`
	    - 例：`2024-06-12 14:30:30.75`表示2024年6月12日14点30分30秒75毫秒
    - `INTERVAL`：表示时间间隔，格式为`[+|-]Y-M D H:M:S`
        - 例：`+2-6 10 5:30:00`表示正2年6个月10天5小时30分钟0秒的时间间隔
        - 时间间隔可以和日期/时间类型进行加减运算，得到新的日期/时间值“
	        - 两个`DATE`/`TIME`/`TIMESTAMP`类型的值相减得到一个`INTERVAL`类型的值
	        - `DATE`/`TIME`/`TIMESTAMP`类型的值加上一个`INTERVAL`类型的值得到一个新的`DATE`/`TIME`/`TIMESTAMP`类型的值
- 用户定义类型：
	- 语法：`CREATE TYPE <type_name> AS (<attribute_name> <data_type>, ...)`
		- 例：
			```sql
			-- 自定义一个金额类型Dollars，本质是numeric(12,2) 
			create type Dollars as numeric (12,2) final; 
			-- 在表中使用自定义类型 
			create table department ( 
				dept_name varchar (20), 
				building varchar (15), 
				budget Dollars -- 用Dollars代替直接写numeric(12,2) 
			);
			```
			- `final`关键字表示该类型不能被继承
			- `numeric(12,2)`表示该类型的值最多有12位数字，其中小数部分有2位
			- 自定义类型和别名有所不同，和一般的`numeric(12,2)`类型不能直接互相赋值
- 用户定义域：域（Domain）也可以由用户自定义，可以直接绑定约束
	- 语法：`CREATE DOMAIN <domain_name> AS <data_type> [constraint]`
		- 例：
			```sql
			-- 示例1：定义一个不允许为空的人名域 
			create domain person_name char(20) not null; 
			-- 示例2：定义一个学位等级域，限制只能取指定值 
			create domain degree_level varchar(10) 
			constraint degree_level_test check (value in ('Bachelors', 'Masters', 'Doctorate'));
			```
	- 特点：
		- 用户定义域可以直接复用其约束，简化表定义
		- 修改约束/数据类型时，只需修改域定义，所有使用该域的表都会自动更新
	- 带`CHECK`的域：域在定义时可以加入`CHECK`作约束校验
		- 例：
			```sql
			create domain hourly_wage numeric(5,2) 
			constraint value_test check(value >= 4.00);
			```
			- `constraint`后定义了约束名称，在报错时会直接显示该约束名称，方便定位问题
			- 域的`check`将自动应用于所有使用该域的属性上，确保数据的一致性和有效性
- 大对象类型（Large-Object Types）：用于存储大量数据，如文本、图像、音频等
    - `BLOB`（Binary Large Object）：用于存储二进制数据，如图像、音频、视频等
	    - 数据库不解析数据内容，将解析权交由外部应用解析
    - `CLOB`（Character Large Object）：用于存储大量文本数据，如文档、日志等
	    - 专门用于处理大文本，支持字符的各种编码
	- 对大对象的查询一般只会返回指针，而不是数据本身，应用程序可以按照指针读取数据，避免大量的I/O开销
# 授权
- 基本概念：
	- 授权（Authorization）是数据库管理员（DBA）授予用户访问数据库资源的权限，以控制用户对数据库的操作和访问
	- 用户（User）：授权的主体
		- 一个用户由`AuthID`（通常是用户名）标识，是授权的接收方
		- 一个用户可以有多个身份标识，多个用户也可以共享同一角色
		- 语法：
			```sql
			-- 创建用户test，密码为test 
			CREATE USER test PASSWORD 'test'; 
			-- 修改用户test的密码为new 
			ALTER USER test PASSWORD 'new'; 
			-- 删除用户test 
			DROP USER test;
			```
	- 数据库对象：授权的客体
		- 数据库、域、模式、表、视图、村粗过程等都属于授权客体
		- 不同客体的权限不同（如表有查询权限，索引有创建/删除权限）
		- **对象的创建者默认拥有所有权限，并可以对其他用户授权**
- 权限的类型：
	- 关系的权限：表/视图的权限
		- `SELECT`：查询权限，允许用户查询表中的数据
		- `INSERT`：插入权限，允许用户向表中插入新数据
		- `UPDATE`：修改权限，允许用户修改表中已有的数据
		- `DELETE`：删除权限，允许用户删除表中的数据
		- `REFERENCES`：引用权限，允许用户创建外键引用该表
	- 权限的执行：SQL语句必须保证用户具有执行该语句所需的权限，否则会被拒绝
		- 例：
			```sql
			insert into depositor 
			select customer-name, loan-number 
			from loan, borrower 
			where branch-name = 'Perryridge' 
			and loan.account-number = borrower.account-number
			```
			这条语句的执行需要`loan`和`borrower`表的查询权限，以及`depositor`表的插入权限
	- 模式授权：针对表结构/索引的模式操作权限
		- `Resources authorization`：允许创建新的关系表
		- `Alteration authorization`：允许修改表结构，如添加/删除属性、修改属性类型等
		- `Index authorization`：允许创建/删除索引
		- `Drop authorization`：允许删除表
	- 视图授权：
		- 通过定义视图，可以限制用户只能访问表中的部分数据或属性，从而实现更细粒度的权限控制
		- 例：对于银行数据库系统，创建一个只包含非敏感数据的视图
			```sql
			create view cust-loan as 
			select branch-name, customer-name 
			from borrower, loan 
			where borrower.loan-number = loan.loan-number
			```
			之后银行的职员只获取到`cust-loan`视图的访问权限，隐藏了贷款金额，账户号码等敏感信息
		- 特点：
			- 视图的授权不依赖基表的权限
			- 创建新的视图不需要建表的权限
			- 视图的权限不可能超过创建者的权限
			- 授权的检测：数据库会在查询之前，先检查权限，再解析视图的查询语句对基表进行查询
- 授权图（Grant Graph）：
    - 定义：授权图是一个有向图，用于表示用户之间的授权关系
	    - 节点：用户（`AuthID`）
	    - 边：权限的授予关系，从授权者指向被授权者
	    - 根节点：数据库管理员（DBA），默认拥有所有权限，是授权图的根节点
	    - 例：![[Pasted image 20260612145424.png]]
	- 规则：所有的权限必须有一条从DBA出发的路径，即所有权限都必须由DBA直接或间接授权
	- 权限的收回：当一个权限被收回时，需要同时收回所有通过该权限授权出去的权限，即收回该权限的子树
		- 例：![[Pasted image 20260612150246.png]]没有从DBA出发的路径的循环授权是不被允许的
- SQL授权语句：
	- `GRANT`语句：
		- 语法：
			```sql
			grant <privilege list> 
			on <relation name or view name> 
			to <user list>
			```
			- `<privilege list>`：权限列表，可以是`SELECT`、`INSERT`、`UPDATE`、`DELETE`等权限的组合，`all privileges`表示授予所有权限
			- `<user_list>`：用户列表，可以是一个或多个用户名的组合，`PUBLIC`表示授予所有用户权限
		- 例：`grant select on branch to U1, U2, U3;`表示授予用户U1、U2、U3对`branch`表的查询权限
	- `WITH GRANT OPTION`：允许被授权用户将权限继续授权给其他用户
		- 例：`grant select on branch to U1 with grant option;`表示授予用户U1对`branch`表的查询权限，并允许U1将该权限继续授权给其他用户
	- `REVOKE`语句：收回授权
	    - 语法：
			```sql
			revoke <privilege list> 
			on <relation name or view name> 
			from <user_list> 
			[restrict|cascade]
			```
			- `cascade`：级联收回权限，即同时收回所有通过该权限授权出去的权限
			- `restrict`：限制收回权限，如果有通过该权限授权出去的权限，则拒绝收回
			- `<privilege list>`：权限列表，可以是`SELECT`、`INSERT`、`UPDATE`、`DELETE`等权限的组合，`all privileges`表示收回所有权限
			- `<user_list>`：用户列表，可以是一个或多个用户名的组合，`public`表示收回所有用户的权限
		- 例：`revoke select on branch from U1 cascade;`表示收回用户U1对`branch`表的查询权限，并同时收回所有通过该权限授权出去的权限
---
[[L13 SQL高级]]