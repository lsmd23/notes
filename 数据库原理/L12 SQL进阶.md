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