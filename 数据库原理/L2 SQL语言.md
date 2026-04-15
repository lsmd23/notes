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
- 换名操作（Renaming）：`AS`语句
	- 用法：`[old_name] AS [new_name]`
	- `AS`语句用于给表、列更换别名，`AS`在语句中是可选的
		- 例：`instructor AS i`，给`instructor`表起别名`i`，也可直接写作`instructor i`
	- 常与`SELECT`操作结合使用，解决表名/列名冲突问题
        - 例：`SELECT i.name, t.course_id FROM instructor AS i, teaches AS t WHERE i.ID = t.ID;`，查询`instructor`表和`teaches`表的连接结果，使用别名`i`和`t`来区分同名属性
- 字符串运算（函数）：`LIKE`操作
	- `LIKE`用于在`WHERE`子句中进行字符串模式匹配
		- 通配符`%`：匹配任意长度的字符串（包括空字符串）
		- 通配符`_`：匹配任意单个字符
		- 转义字符：如果需要匹配`%`或`_`本身，可以使用转义字符（如`\`）进行转义
		- 模式匹配是**严格区分大小写**的
		- 其他操作：连接`||`，大小写转换函数`UPPER()`和`LOWER()`，取长度，取子串等
			- 例：`SELECT name FROM student WHERE name LIKE 'A%';`，查询学生表中`name`以`A`开头的所有名字
- 结果集中元组的顺序：`Order By`子句
	- 用法：`ORDER BY [column_name] [ASC|DESC]`
    - `ORDER BY`用于指定查询结果的排序方式，默认是升序（`ASC`），也可以指定为降序（`DESC`）
    - 可以指定多个排序列，按照优先级进行排序
        - 例：`SELECT name, salary FROM instructor ORDER BY salary DESC, name ASC;`，查询教师表中的名字和工资，并按照工资降序排序，如果工资相同则按照名字升序排序
    - 可以与`DISTINCT`一起使用，先去重再排序
        - 例：`SELECT DISTINCT dept_name FROM instructor ORDER BY dept_name;`，查询教师表中不同的部门名字，并按照部门名字升序排序
- `WHERE`子句的条件：`BETWEEN`与元组比较
	- `BETWEEN`操作：
		- `BETWEEN`用于指定一个范围
		- 用法：`[column_name] BETWEEN [lower_bound] AND [upper_bound]`
	    - `BETWEEN`包含边界值，即`lower_bound`和`upper_bound
	- 元组比较：
		- 多列同时匹配
		- 例：`select name, course_id  from instructor, teaches where (instructor.ID, dept_name) = (teaches.ID, 'Biology'); 50`，查询教师表和选课表中教师ID和部门名字同时满足条件的记录，即查询教授生物课程的教师名字和课程ID
- 空值：
	- 定义：空值（NULL）表示未知或不存在的值
	- 运算规则：
        - 任何与NULL进行的算术运算结果都是NULL
        - 任何与NULL进行的比较运算结果都是NULL（即未知）
    - 空值判断：必须用`IS NULL`或`IS NOT NULL`来判断是否为空值
        - 例：`SELECT name FROM student WHERE tot_cred IS NULL;`，查询学生表中`tot_cred`列值为NULL的学生名字
- 三值逻辑：
    - SQL中的布尔逻辑包括三种值：`TRUE`、`FALSE`和`UNKNOWN`（即NULL）
    - 逻辑运算：

		|运算|规则|
		|---|---|
		|OR|`unknown or true = true`；`unknown or false = unknown`；`unknown or unknown = unknown`|
		|AND|`true and unknown = unknown`；`false and unknown = false`；`unknown and unknown = unknown`|
		|NOT|`not unknown = unknown`|
    - 在`WHERE`子句中，只有当条件结果为`TRUE`时才会返回记录，`UNKNOWN`和`FALSE`都会被过滤掉
# 复杂SQL
## 集合操作
- 基础集合操作：操作结果自动去重
	- `UNION`：返回两个查询结果的并集
    - `INTERSECT`：返回两个查询结果的交集
    - `EXCEPT`：返回第一个查询结果中存在但第二个查询结果中不存在的记录
	- 例：`SELECT dept_name FROM instructor UNION SELECT dept_name FROM student;`，查询教师表和学生表中所有不同的部门名字
- 留重集合操作：保留重复记录
	- `UNION ALL`：返回两个查询结果的并集，保留重复记录
    - `INTERSECT ALL`：返回两个查询结果的交集，保留重复记录
    - `EXCEPT ALL`：返回第一个查询结果中存在但第二个查询结果中不存在的记录，保留重复记录
    - 例：`SELECT dept_name FROM instructor UNION ALL SELECT dept_name FROM student;`，查询教师表和学生表中所有部门名字，包括重复的部门名字
## 聚合操作
- 对列的多重集合值进行统计计算的操作
- 核心聚合函数：
	- 
	|函数|含义|适用类型|特殊说明|
	|---|---|---|---|
	|`MAX()`|最大值|数值 / 字符串 / 日期|字符串按字典序、日期按时间序|
	|`MIN()`|最小值|数值 / 字符串 / 日期|同上|
	|`AVG()`|平均值|仅数值|自动忽略 `NULL`|
	|`SUM()`|求和|仅数值|自动忽略 `NULL`|
	|`COUNT()`|计数|任意类型|`COUNT(列)` 忽略 NULL；`COUNT(*)` 统计行数|
	- 例：
		```sql
		-- 1. 查音乐系教师平均工资 
		select avg(salary) from instructor where dept_name = 'Music'; 
		-- 2. 查2018春授课的教师总数（去重ID） 
		select count(distinct ID) from teaches where semester = 'Spring' and year = 2018; 
		-- 3. 查course表总记录数 
		select count(*) from course;
		```
	- 对`NULL`值的处理：
		- 除`COUNT(*)`外，自动忽略`NULL`值进行计算；`COUNT(*)`统计行数，包括`NULL`值所在的行
		- 若待聚合集合仅有`NULL`值：
			- `COUNT`函数返回0
			- 其他聚合函数返回`NULL`
- 分组统计：`GROUP BY`子句
    - 用法：`GROUP BY [column_name1], [column_name2], ...`
    - `GROUP BY`用于将查询结果按照一个或多个列进行分组，然后对每个分组应用聚合函数
	    - 例：按系别查找平均工资
            ```sql
			select dept_name, avg(salary) as avg_salary 
			from instructor 
			group by dept_name;
            ```
    * 规则：`SELECT`中非聚合函数的列，必须出现在`GROUP BY`子句中
        - 例：`SELECT dept_name, salary FROM instructor GROUP BY dept_name;`，错误示例，`salary`列既不是聚合函数的参数，也没有出现在`GROUP BY`子句中
- 分组后筛选：`HAVING`子句
    - 用法：`HAVING [condition]`
    - `HAVING`用于对分组后的结果进行筛选，条件通常涉及聚合函数
        - 例：查平均工资超过80000的系别
            ```sql
            select dept_name, avg(salary) as avg_salary 
            from instructor 
            group by dept_name 
            having avg(salary) > 80000;
            ```
## 嵌套子查询
- 嵌套在另一个SQL查询中的`SELECT FROM WHERE`子句表达式，也称为子查询（Nested Subquery）
### `WHERE`子句中的子查询
- 用于在筛选条件中执行集合相关操作判断
- 集合成员判断（`in`/`not in`）：
    - 功能：判断元素/元组是否属于查询返回的集合
    - 语法：
        ```sql
        [column_name] IN (SELECT [column_name] FROM [table_name] WHERE [condition])
        ```
    - 例：`SELECT name FROM student WHERE dept_name IN (SELECT dept_name FROM instructor WHERE salary > 80000);`，查询学生表中部门名字属于教师表中工资超过80000的部门名字的学生名字
- 集合比较（`some`/`all`）
	- `some`（等价于`any`）：与集合中至少一个元素满足比较关系
		- 逻辑：`F <comp> some r`$\Leftrightarrow \exists t \in r$使得`F <comp> t`，其中`<comp>`是比较运算符，如`<`、`>`、`=`等
		- 等价关系：
			- `= some`$\Leftrightarrow$`in`
			- `≠ some`不等价于`not in`，因为`not in`要求集合中没有任何元素满足条件，而`≠ some`只要求至少一个元素不满足条件
	- `all`：与集合中所有元素满足比较关系
        - 逻辑：`F <comp> all r`$\Leftrightarrow \forall t \in r$使得`F <comp> t`
        - 等价关系：
            - `= all`不等价于`in`，因为`in`要求至少一个元素满足条件，而`= all`要求所有元素都满足条件
            - `≠ all`$\Leftrightarrow$`not in`
    - 例：
	    - 查询薪资高于生物系**任意**教师的教师：`salary > some (select salary from instructor where dept_name='Biology')`
		- 查询资产高于布鲁克林**所有**支行的支行：`assets > all (select assets from branch where branch_city='Brooklyn')`
- 空关系测试（`exists`/`not exists`）：
    - 功能：判断子查询返回的关系是否为空
	    - `exists r`：子查询非空，返回`true`
	    - `not exists r`：子查询为空，返回`true`
    - 语法：
        ```sql
        EXISTS (SELECT [column_name] FROM [table_name] WHERE [condition])
        ```
    - 关联子查询（Correlated Subquery）：子查询中的条件引用了外层查询的列，子查询的结果依赖于外层查询的当前行
        - 例：`SELECT name FROM student s WHERE EXISTS (SELECT * FROM takes t WHERE t.ID = s.ID AND t.grade = 'A');`，查询学生表中至少选修过一门课程并且成绩为A的学生名字
- 重复元组测试（`unique`）
	- 功能：判断子查询返回的关系是否包含重复元组
    - 语法：
        ```sql
        UNIQUE (SELECT [column_name] FROM [table_name] WHERE [condition])
        ```
    - 例：`SELECT T.course_id FROM course as T WHERE UNIQUE (SELECT R.course_id FROM section AS R WHERE T.course_id = R.course_id AND R.year = 2017);`，查询课程表中在2017年至多开设过一次的课程ID
    - 注：目前`UNIQUE`并未被广泛支持，实际使用中可以通过`GROUP BY`和`HAVING COUNT(*) = 1`来实现类似的功能
### `FROM`子句中的子查询
- 功能：将子查询的结果作为临时表，参与外层查询的计算
	- **必须为子查询的结果定义一个别名**，以便在外层查询中引用
	- 例：
		- `having`写法：
			```sql
select dept_name, avg(salary) 
from instructor 
group by dept_name 
having avg(salary)>42000;
			```
		* 子查询写法：
            ```sql
select dept_name, avg_salary 
from (select dept_name, avg(salary) as avg_salary from instructor group by dept_name) as dept_avg 
where avg_salary > 42000;
            ```
- `lateral`子句（SQL：2003标准）：横向关联子句
	- 允许子查询引用外层查询中的列，即使子查询在`FROM`子句中
    - 语法：
        ```sql
        FROM [table_name] AS [alias], LATERAL (SELECT [column_name] FROM [table_name] WHERE [condition]) AS [subquery_alias]
        ```
    - 例：查询教师表中每个教师的名字、工资以及所在系的平均工资，使用`lateral`子查询来计算每个教师所在系的平均工资
			```sql
			select name, salary, avg_salary 
			from instructor I1, lateral (select avg(salary) as avg_salary 
			from instructor I2 
			where I2.dept_name= I1.dept_name);
			```
- `with`子句：公共子表达式（common table expression，CTE）
    - 普通`CTE`：用于定义仅在当前查询结果中有效的临时关系
	    - 语法：`with [cte_name] as (SELECT [column_name] FROM [table_name] WHERE [condition])`
	    - 例：查询预算最高的院系
        ```sql
        with max_budget(value) as (select max(budget) from department) 
        select dept_name from department, max_budget where department.budget = max_budget.value;
        ```
    - 递归`with`子句（SQL:1999标准）：处理层级递归数据
	    - 结构：
        ```sql
        WITH RECURSIVE [cte_name] AS (
            [base_query] -- 基础查询，提供递归的起始点
            UNION ALL
            [recursive_query] -- 递归查询，引用自身以继续递归
        )
        SELECT * FROM [cte_name];
        ```
	        - 锚点成员：递归的起点，提供初始数据集
	        - 递归成员：引用CTE自身，基于前一次递归的结果继续生成新的结果集
	        - 递归查询必须包含一个终止条件，以防止无限递归
	    - 例：查询员工表中所有员工及其直接或间接的上级员工
        ```sql
        WITH RECURSIVE EmployeeHierarchy AS (
            -- 锚点成员：选择没有上级的员工（即顶层员工）
            SELECT ID, name, manager_id, 1 AS level
            FROM employee
            WHERE manager_id IS NULL
            
            UNION ALL
            
            -- 递归成员：选择有上级的员工，并连接到上级员工的层次结构中
            SELECT e.ID, e.name, e.manager_id, eh.level + 1
            FROM employee e
            INNER JOIN EmployeeHierarchy eh ON e.manager_id = eh.ID
        )
        SELECT * FROM EmployeeHierarchy;
        ```
### `SELECT`子句中的子查询
- 返回单个标量值的子查询，用于替换`select`中的列
	- 要求：必须返回单行单列的结果，否则会导致错误
    - 例：查询每个教师的名字和所在系的平均工资
        ```sql
        select name, (select avg(salary) from instructor where dept_name = I.dept_name) as avg_salary 
        from instructor I;
        ```
        可以写作`group by`的形式：
        ```sql
select dept_name, avg(salary) as avg_salary 
from instructor 
group by dept_name;
        ```
# 数据库的修改操作
- 删除（Deletion）：
	- 清空全表数据：`DELETE FROM [table_name]`
		- 仅删除数据，表结构仍然存在
	- 条件删除：`DELETE FROM [table_name] WHERE [condition]`
        - 仅删除满足条件的数据行
    - 嵌套子查询删除：
        ```sql
        DELETE FROM [table_name] 
        WHERE [column_name] IN (SELECT [column_name] FROM [table_name] WHERE [condition]);
        ```
        - 例：删除学生表中选修过生物课程的学生记录
            ```sql
DELETE FROM student 
WHERE ID IN (SELECT ID FROM takes WHERE course_id IN (SELECT course_id FROM course WHERE dept_name = 'Biology'));
            ```
    - 基于聚合的删除：以删除工资低于教师平均工资的所有教师为例
        ```sql
DELETE FROM instructor 
WHERE salary < (SELECT AVG(salary) FROM instructor);
        ```
	    - 风险：若删除时动态重算工资，会导致后续删除出现问题
	    - 解决：先执行子查询计算，再删除
- 插入（Insertion）：
	- 单行插入：
		- 隐式列顺序插入：按表结构对应列顺序插入数据
            ```sql
            INSERT INTO [table_name] VALUES ([value1], [value2], ..., [valueN]);
            ```
            存在数据和表结构不匹配导致插入失败的风险
        - 显式列顺序插入：指定列名和对应值的顺序
            ```sql
            INSERT INTO [table_name] ([column_name1], [column_name2], ..., [column_nameN]) 
            VALUES ([value1], [value2], ..., [valueN]);
            ```
            更加安全，可读性强，不受表结构影响
        - 插入空值：使用`NULL`表示空值
            ```sql
            INSERT INTO [table_name] ([column_name1], [column_name2]) 
            VALUES ([value1], NULL);
            ```
    - 批量插入：`INSERT ... SELECT`语句
        ```sql
        INSERT INTO [table_name] ([column_name1], [column_name2], ..., [column_nameN]) 
        SELECT [column_name1], [column_name2], ..., [column_nameN] 
        FROM [source_table] 
        WHERE [condition];
        ```
        - 例：将教师表中工资高于80000的教师记录插入到一个新的表中
            ```sql
INSERT INTO high_salary_instructors (ID, name, dept_name, salary) 
SELECT ID, name, dept_name, salary 
FROM instructor 
WHERE salary > 80000;
            ```
- 更新（Updates）：
	- 条件更新：以给工资大于 10 万的教师涨 3%，其余涨 5% 为例
		- 分两次更新：
            ```sql
            UPDATE instructor 
            SET salary = salary * 1.03 
            WHERE salary > 100000;

            UPDATE instructor 
            SET salary = salary * 1.05 
            WHERE salary <= 100000;
            ```
        - 使用`CASE`表达式一次更新：
            ```sql
            UPDATE instructor 
            SET salary = CASE 
                WHEN salary > 100000 THEN salary * 1.03 
                ELSE salary * 1.05 
            END;
            ```
            解释：`CASE`语句根据条件对每行数据进行不同的计算，满足条件的行执行对应的更新操作
    - 标量子查询更新：以重新计算成绩非F且非空课程的学生总分为例
	    - 无课学生设置为`NULL`
        ```sql
        UPDATE student S
        SET tot_cred = (
	        SELECT SUM(credits) 
            FROM takes, course
            WHERE takes.course_id = course.course_id 
                AND takes.grade IS NOT NULL
                AND takes.grade <> 'F'
                AND takes.ID = student.ID
        );    
        ```
        - 无课学生设置为0
		    ```sql
UPDATE student S 
SET tot_cred = ( 
	SELECT CASE 
		WHEN SUM(credits) IS NOT NULL THEN SUM(credits) 
		ELSE 0 
	END 
	FROM takes, course 
	WHERE takes.course_id = course.course_id 
		AND S.ID = takes.ID 
		AND takes.grade <> 'F' 、
		AND takes.grade IS NOT NULL 
	);
		    ```
        解释：子查询计算每个学生的总学分，使用`CASE`表达式处理没有选课的学生，将其总学分设置为0
    - 更新的约束冲突：
	    - 例：
		    ```sql
		    -- 创建表，id为主键（唯一非空约束） 
		    CREATE TABLE myStudent (id INT PRIMARY KEY, name VARCHAR(20)); 
		    -- 插入测试数据 
		    INSERT INTO myStudent VALUES(1, 'Bob'); 
		    INSERT INTO myStudent VALUES(2, 'Alice'); 
		    -- 尝试将Alice的id改为1 
		    UPDATE myStudent SET id = 1 WHERE name = 'Alice';
		    ```
		    结果：更新失败，违反了主键约束，因为id为1已经存在于表中
	    - 原因：更新操作试图将一个属性值修改为一个已经存在的值，导致唯一约束被破坏
# 连接操作
- 示例表：
	- 银行数据库：
		- 
			|表名|字段|作用|
			|---|---|---|
			|`Branch`|`branch-name`, `branch-city`, `assets`|支行信息|
			|`Customer`|`id`, `customer-name`, `customer-street`, `customer-city`|客户信息|
			|`Account`|`account-number`, `branch-name`, `balance`|账户信息|
			|`Loan`|`loan-number`, `branch-name`, `amount`|贷款信息|
			|`Depositor`|`id`, `account-number`|客户 - 账户关联|
			|`Borrower`|`id`, `loan-number`|客户 - 贷款关联|
	- 大学数据库：核心表结构
		- |表名|核心字段|作用|
			|---|---|---|
			|`student`|`s_ID`, `name`, `dept_name`, `tot_cred`|学生信息|
			|`takes`|`s_ID`, `course_id`, `sec_id`, `semester`, `year`, `grade`|学生选课信息|
			|`course`|`course_id`, `title`, `dept_name`, `credits`|课程信息|
			|`prereq`|`course_id`, `prereq_id`|课程先修关系|
- 连接操作：本质是带匹配条件的笛卡尔积，通常作为子查询嵌套在`FROM`子句中
    - 内连接（Inner Join）：返回满足连接条件的元组
		- 自然连接（Natural Join）：自动基于两个表中同名的属性进行连接，连接条件为这些同名属性的值相等
			- 等价形式：`A natural inner join B`等价于`A inner join B using([common_attribute])`
			- 例：`loan natural join borrower`，连接贷款表和借款人表，自动使用`loan-number`作为连接条件，**悬浮元组（Dangling Tuple，即没有匹配的元组）会被过滤掉**
		- Theta连接（Theta Join）：用`ON`子句指定任意条件，保留所有列
			- 例：`loan inner join borrower on loan.loan-number = borrower.loan-number`，连接贷款表和借款人表，使用`loan-number`作为连接条件，**悬浮元组会被过滤掉**
		- 自连接（Self Join）：同一个表与自身进行连接，用于表内数据对比
			- 例：
				```sql
			select count(distinct i1.id) 
			from instructor as i1 inner join instructor as i2 
			on i1.salary > i2.salary;
			    ```
	- 外连接（Outer Join）：保留满足连接条件的元组以及不满足连接条件的元组（悬浮元组），对于不满足连接条件的元组，**缺失的属性值用`NULL`填充**
        - 左外连接（Left Outer Join）：保留左表中的所有元组
            - 例：`student natural left outer join takes`，连接学生表和选课表，保留学生表中的所有记录，对于没有匹配的选课记录，选课相关属性值为`NULL`
        - 右外连接（Right Outer Join）：保留右表中的所有元组
            - 例：`loan right outer join borrower on loan.loan-number = borrower.loan-number`，连接贷款表和借款人表，保留借款人表中的所有记录，对于没有匹配的贷款记录，贷款相关属性值为`NULL`
        - 全外连接（Full Outer Join）：保留两个表中的所有元组
            - 例：`course natural full outer join prereq`，连接课程表和先修关系表，保留两个表中的所有记录，对于没有匹配的记录，缺失的属性值为`NULL`