# 从编程语言访问数据库
- SQL语言是简单的声明式语言，无法处理分支/循环等复杂流程控制，也无法直接和用户交互
- 数据库系统的架构：
	- 示意图：![[Pasted image 20260612153732.png]]
		- 两层架构（Two-tier）：应用程序直接和数据库交互，客户端同时处理用户页面和业务逻辑，数据库服务器只负责存储和执行SQL语句
		- 三层架构（Three-tier）：引入应用服务器，客户端只负责用户界面，应用服务器处理业务逻辑和数据库交互，数据库服务器专注于数据存储和查询执行
	- 数据库的交互通信：数据库的交互必须按照以下流程：
		1. 建立连接：用户通过网络建立会话，验证身份，创建通信通道
		2. 发送SQL命令：用户输入SQL语句，客户端将其发送到数据库服务器
		3. 处理返回结果：数据库服务器执行SQL语句，生成结果集，返回给客户端
		- 数据库的交互通信必须遵循一套API规范，确保不同编程语言和数据库系统之间的兼容性和互操作性
- 数据库的访问机制：
	- JDBC（Java Database Connectivity）：Java语言访问数据库的标准API，提供统一的接口和驱动机制，支持多种数据库系统
		- 特点：
			- 跨数据库兼容：只要数据库提供了对应的JDBC驱动，就可以用相同的API连接不同的数据库
			- 面向对象的API：用对象封装了数据库操作，方便调用
		- 流程：
			1. 建立数据库连接：
				```java
				Connection conn = DriverManager.getConnection(
					"jdbc:postgresql://localhost:5432/postgres", 
					"postgres", 
					"pawd" );
				```
				- 连接URL格式：`jdbc:<数据库类型>://<主机>:<端口>/<数据库名>`
				- 后续参数是数据库的用户名和密码，用于用户身份验证
				- 实际项目中`DriveManager`使用相对较少，一般使用连接池来管理数据库连接，提升性能和资源利用率
>连接池：连接池是一种数据库连接管理机制，预先创建一定数量的数据库连接，并在应用程序需要时分配和回收这些连接，避免频繁建立和关闭连接带来的性能开销
>
			2. 创建Statement对象，执行SQL：
				-  更新操作（增删改）：
                ```java
                stmt.executeUpdate( 
	                "insert into instructor values('77987', 'Tom', 'Physics', 98000)" 
	            );
                ```
				- 查询操作：
					```java
					ResultSet rset = stmt.executeQuery( 
						"select dept_name, avg(salary) from instructor group by dept_name" 
					);
		            ```
		    3. 处理查询结果（ResultSet）：
					```java
					while (rset.next()) { 
						// 方式1：按列名读取，可读性好 
						String dept = rset.getString("dept_name"); 
						// 方式2：按列索引读取（索引从1开始，对应SELECT语句的列顺序） 
						float avgSalary = rset.getFloat(2); 
						System.out.println(dept + " " + avgSalary); 
					}
					```
					- `ResultSet`返回的结果指向查询结果的每一行，需要用`next()`方法移动指针来访问每一行数据
					- 空值处理：如果列的值为`NULL`，使用`getInt()`等方法会返回默认值（如0），需要调用`wasNull()`方法来判断是否真的为`NULL`
						```java
						int a = rset.getInt("a"); 
						if (rset.wasNull()) { 
							System.out.println("Got null value"); 
						}
						```
			4. 关闭连接：
				```java
				stmt.close(); 
				conn.close();
				```
				必须关闭连接释放数据库资源，否则会导致连接泄漏问题
		- `PreparedStatement`：安全高效的执行SQL
			- 问题：
				- **SQL注入漏洞**：直接拼接用户输入，可能被恶意用户利用来执行任意SQL命令，造成数据泄露或破坏
					- 例：`String sql = "SELECT * FROM users WHERE username = '" + username + "' AND password = '" + password + "'";`如果`username`或`password`包含恶意SQL代码，就可能被注入攻击
				- 性能开销：每一条拼接的SQL语句都需要数据库解析和编译，效率较低
			- 解决：使用`PreparedStatement`语句预编译SQL模板，使用占位符`?`来代替参数，执行时再绑定参数值
				```java
				// 预编译SQL，用?作为占位符 
				PreparedStatement pStmt = conn.prepareStatement( 
					"insert into instructor values(?,?,?,?)" 
				); // 给占位符赋值 
				pStmt.setString(1, "88877"); 
				pStmt.setString(2, "Perry"); 
				pStmt.setString(3, "Finance"); 
				pStmt.setInt(4, 125000); 
				// 执行更新 
				pStmt.executeUpdate();
				```
				- `PreparedStatement`会预编译SQL模板，提升执行效率，并且自动处理参数值的转义，防止SQL注入攻击
		- JDBC元数据：
			- `ResultSet`元数据：通过`ResultSetMetaData`对象可以获取查询结果的列数、列名、列类型等信息，方便动态处理查询结果
				```java
				ResultSetMetaData rsmd = rset.getMetaData(); 
				for(int i = 1; i <= rsmd.getColumnCount(); i++) { 
					System.out.println("列名：" + rsmd.getColumnName(i)); 
					System.out.println("类型：" + rsmd.getColumnTypeName(i)); 
				}
				```
			- 数据库元数据：通过`DatabaseMetaData`对象可以获取数据库的版本、支持的SQL功能、表和列的信息等，方便编写兼容不同数据库的代码
				```java
				DatabaseMetaData dbmd = conn.getMetaData(); 
				// 获取instructor表的所有列信息 
				ResultSet rs = dbmd.getColumns(null, "public", "instructor", "%"); 
				while(rs.next()) { 
					System.out.println("列名：" + rs.getString("COLUMN_NAME")); 
					System.out.println("类型：" + rs.getString("TYPE_NAME")); 
				}
				```
		- 事务控制：默认每一个SQL语句都是一个独立事务，且是自动提交的
			- 手动事务控制：多个SQL组成事务，需要自动提交并手动控制事务边界
				```java
				conn.setAutoCommit(false); // 关闭自动提交 
				try { 
					// 执行多个SQL操作（比如转账的扣钱、加钱） 
					conn.commit(); // 所有操作成功，提交事务 
				} catch (SQLException e) { 
					conn.rollback(); // 出现异常，回滚所有操作 
				}
				```
		- 调用存储过程/函数：使用`CallableStatement`可以调用数据库中定义好的存储过程或函数
			```java
			// 调用带返回值的函数 
			CallableStatement cStmt1 = conn.prepareCall("{? = call some_function(?)}"); // 调用存储过程 
			CallableStatement cStmt2 = conn.prepareCall("{call some_procedure(?,?)}");
			```
	- ODBC（Open Database Connectivity）：一种跨平台的数据库访问API，提供统一的接口和驱动机制，支持多种数据库系统
		- 支持多种语言
		- 目前几乎所有的数据库也都支持ODBC接口，尤其在Windows平台上，ODBC是非常常用的数据库访问方式
	- *嵌入式SQL（Embedded SQL）：在编程语言中直接嵌入SQL语句，通过预处理器将SQL语句转换为相应的API调用，适用于一些特定的编程环境和数据库系统*
		- *例如在C语言中，可以使用嵌入式SQL来直接在代码中写SQL语句，编译时通过预处理器转换为C函数调用*
		- *这种方式较为古老，目前使用较少，但在某些特定场景下仍然有应用价值*
# 函数与过程构造
- 如果所有过程都使用高级语言与SQL交互，中间需要多次访问网络和数据库，效率低下，且复用较少，同时数据一致性也难以保证
- 存储过程（Stored Procedure）和函数（Function）：以PostgreSQL为例
	- 存储过程：语法示例：
		```sql
		CREATE OR REPLACE PROCEDURE 
		get_instructor(IN p_id INT, OUT deptname VARCHAR(20)) 
		AS $$ 
		BEGIN -- READS SQL DATA 
			SELECT dept_name INTO deptname FROM instructor 
			WHERE id = p_id; 
		END $$ LANGUAGE plpgsql;
		```
		- `CREATE OR REPLACE PROCEDURE`：创建或替换存储过程
		- `IN p_id INT`：输入参数，类型为整数
		- `OUT deptname VARCHAR(20)`：输出参数，类型为字符串
		- `LANGUAGE plpgsql`：指定使用PL/pgSQL语言编写存储过程
			- PostgreSQL的过程化SQL语言
			- 其他的数据库一般也有类似的过程化语言，如Oracle的PL/SQL、SQL Server的T-SQL等
		- 调用：使用`CALL`语句调用
			```sql
			CALL get_instructor(77987, deptname);
			```
	- SQL函数：语法示例：
		```sql
		create or replace function dept_count (department_name varchar(20)) 
		returns integer as $$ 
		declare d_count integer; 
		begin -- READS SQL DATA 
			select count (*) into d_count 
			from instructor 
			where instructor.dept_name = department_name; 
			return d_count; 
		end $$ language plpgsql;
		```
		- `returns integer`：指定函数的返回类型为整数
		- `declare d_count integer;`：声明局部变量
		- `return d_count;`：**函数必须有返回值，过程则不需要返回值**
		- 调用：直接嵌入语句中即可（`SELECT`/`WHERE`等）
			```sql
			SELECT dept_count('Physics');
			```
	- 函数与存储事务的事务控制：
		- 存储过程可以控制事务（提交/回滚）
		- 函数不能控制事务，必须在一个已经存在的事务中执行
- 过程化控制结构：为SQL语言赋予流程控制能力
	- 复合语句（Compound Statement）：用`BEGIN...END`包裹一段SQL语句，形成一个复合语句块，可以在存储过程中使用，其内部可以声明局部变量，并且可以使用流程控制结构
	- 循环结构：
		- `WHILE`循环：当条件满足时重复执行一段代码
			```sql
			while boolean expression do 
				sequence of statements; 
			end while;
			```
		- `REPEAT-UNTIL`循环：至少执行一次代码块，直到条件满足时停止
			```sql
			repeat 
				sequence of statements; 
			until boolean expression 
			end repeat;
			```
		- `FOR`循环：对一个范围或集合中的每个元素执行代码块，例：
			```sql
			declare n integer default 0; 
			for r as 
				select budget from department 
			do 
				set n = n + r.budget 
			end for
			```
			此处`r`是循环变量，表示查询结果的每一行，通过`r.budget`访问当前行的`budget`列
	- 条件结构：
		- `IF-THEN-ELSE`：基础的条件判断结构
			```sql
			if boolean expression then 
			    sequence of statements; 
			elsif boolean expression then 
			    sequence of statements; 
			else 
			    sequence of statements; 
			end if;
			```
		- `CASE`表达式：更简洁的多分支条件表达式，适用于简单的条件判断
			```sql
			case 
			    when boolean expression then result 
			    when boolean expression then result 
			    else result 
			end case;
			```
	- **注：不同数据库系统的过程化语言在语法和功能上可能存在差异，具体使用时需要参考相应数据库的文档**
- 外部语言程序：SQL自带的过程化语言能力有限，因此允许使用外部语言编写过程，再在SQL中调用
	- 例：
		```sql
		create procedure dept_count_proc(in dept_name varchar(20), out count integer) 
		language C 
		external name '/usr/avi/bin/dept_count_proc' 
		create function dept_count(dept_name varchar(20)) 
		returns integer 
		language C 
		external name '/usr/avi/bin/dept_count'
		```
		- `language C`：指定使用C语言编写外部程序
		- `external name`：指定编译后可执行文件的路径，数据库将加载文件并执行其代码
	- 优点：性能高，适合复杂计算和特殊数据类型处理
	- 缺点：开发复杂，安全风险较高，需要额外的权限和配置
# 触发器
- 基本概念：
	- 触发器（Trigger）：数据库被修改时，由系统自动执行的语句
		- 触发条件（Conditions）：定义触发器何时被触发，如`BEFORE INSERT`、`AFTER UPDATE`等
		- 触发动作（Actions）：定义触发器被触发时执行的SQL语句
	- ECA规则（Event-Condition-Action Rules）：触发器的核心模型
		- 事件（Event）：触发器被触发的条件，如数据修改操作（INSERT、UPDATE、DELETE）
		- 条件（Condition）：触发器被触发时需要满足的条件，如特定列的值满足某个条件
		- 动作（Action）：当触发器被触发且条件满足时执行的SQL语句，如更新其他表、记录日志等
- 触发器的实现细节：
	- 触发时机：
		- `BEFORE`触发器：在修改操作执行前触发，用于**校验、修正、拦截数据**
			- 例：
				```sql
				create trigger setnull_trigger before update of takes 
				referencing new row as nrow 
				for each row 
				when (nrow.grade = ' ') 
				begin atomic 
					set nrow.grade = null; 
				end;
				```
				如果新的`grade`值为空格，则将其设置为`NULL`
		- `AFTER`触发器：在修改操作执行后触发，用于**同步数据，更新其他表，记录日志等**
			- 例：
				```sql
				create trigger credits_earned after update of takes on (grade) 
				referencing new row as nrow 
				referencing old row as orow 
				for each row 
				when nrow.grade <> ’F’ and nrow.grade is not null 
					and (orow.grade = ’F’ or orow.grade is null) 
				begin atomic 
					update student 
					set tot_cred= tot_cred + 
						(select credits 
						from course 
						where course.course_id= nrow.course_id) 
					where student.id = nrow.id; 
				end;
				```
				当学生的成绩更新到数据库后，如果新的成绩不是`F`且不为`NULL`，同时旧的成绩是`F`或`NULL`，则更新学生的总已修学分
	- 新旧数据的引用：`OLD`和`NEW`
		- 触发器执行时，可以访问修改前的数据（`OLD`）和修改后的数据（`NEW`），通过这些引用可以根据数据的变化来决定触发器的行为
		    - `INSERT`操作只能访问`NEW`数据
			- `DELETE`操作只能访问`OLD`数据
			- `UPDATE`操作可以访问`OLD`和`NEW`数据
		- 例：上例中，`nrow`引用了新的数据行，`orow`引用了旧的数据行，通过比较新旧成绩的变化来决定是否更新学生的总已修学分
	- `UPDATE`触发器的列级限制：
		- `UPDATE OF <column>`：指定触发器仅在特定列被更新时触发，避免不必要的触发器执行，提高性能
- 更细粒度的触发器：
	- 行级触发器（Row-level Trigger）：针对每一行数据的修改操作触发，适用于需要对每条数据进行处理的场景
		- 语法：`FOR EACH ROW`
		- 逻辑：每一条受影响的记录都会触发一次触发器
			- 例：`BEFORE UPDATE OF grade ON takes FOR EACH ROW`表示每次更新`takes`表的`grade`列时，针对每条记录触发一次
	- 语句级触发器（Statement-level Trigger）：针对整个SQL语句的修改操作触发，适用于需要对整个操作进行处理的场景
		- 语法：`FOR EACH STATEMENT`
		- 逻辑：无论SQL语句影响多少条记录，触发器只会被触发一次
		- 特性：可以引用`OLD TABLE`和`NEW TABLE`两个过渡表，方便批量处理数据