# mysql 存储过程

标签（空格分隔）： mysql

---

## 主要语法

### 创建存储过程

- 语法
    ```
    CREATE
        [DEFINER = { user | CURRENT_USER }]
    　PROCEDURE sp_name ([proc_parameter[,...]])
        [characteristic ...] routine_body
     
    proc_parameter:
        [ IN | OUT | INOUT ] param_name type
     
    characteristic:
        COMMENT 'string'
      | LANGUAGE SQL
      | [NOT] DETERMINISTIC
      | { CONTAINS SQL | NO SQL | READS SQL DATA | MODIFIES SQL DATA }
      | SQL SECURITY { DEFINER | INVOKER }
     
    routine_body:
    　　Valid SQL routine statement
     
    [begin_label:] BEGIN
    　　[statement_list]
    　　　　……
    END [end_label]
    ```
    
- 示例

    ```
    -- 存在则删除重新创建
	drop procedure if exists p_user_add_role_id;
	-- 创建存储过程 p_user_add_role_id
	CREATE PROCEDURE p_user_add_role_id()
	BEGIN 
	-- 变量声明
	-- 用于判断循环是否结束
	DECLARE done boolean DEFAULT false;
	DECLARE id varchar(16);
	DECLARE user_name varchar(64) ;
	-- 定义游标，用于循环查询结果
	DECLARE old_users CURSOR FOR SELECT t1.id,t1.user_name FROM contract.`user` t1;
	-- 声明当游标遍历完全部记录后将标志变量置成某个值
	DECLARE CONTINUE HANDLER FOR NOT FOUND SET done=true;
	-- 打开游标
	OPEN old_users;
	-- 开始循环
	REPEAT
	-- 获取游标中数据
		FETCH old_users INTO id,user_name;
		IF done = false THEN 
		    -- 业务逻辑实现
		    INSERT INTO test.t_user(`id`,`name`) VALUES (id,user_name);
		END IF;
			
	UNTIL done END REPEAT;
	-- 关闭游标
	CLOSE old_users;
	/*事务提交*/ 
	COMMIT; 
	END;
	```

- 调用

    ```
    -- 调用上面创建的存储过程
    CALL p_user_add_role_id();
    ```

### 变量
    
- 变量声明

    ```
    DECLARE variable_name [,variable_name...] datatype [DEFAULT value];
    ```
- 变量赋值

    ```
    SET 变量名 = 表达式值 [,variable_name = expression ...]
    ```
### 控制语句

- ` if-then-else`

    ```
    if <条件> then
    -- 处理
    end if;
    ```
- `case-when-then`

    ```
    case <变量> 
        when <变量值> then 
        -- 处理
        when <变量值> then
        -- 处理
        ...
        else
        -- 处理
    end case;
    ```
- `while-end while`
    
    ```
    while <条件> do
    -- 循环体
    end while;
    ```
- `repeat-end repeat`

    ```
    repeat
    -- 循环体
    until <条件>
    end repeat;
    ```

### 游标使用

可用于循环处理结果集。

- 定义

    ```
    -- 定义游标
    DECLARE <游标名> CURSOR FOR <select语句>;
    -- 定义 “设置循环结束标识done值（需在变量中声明）怎么改变” 的逻辑
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done=true;
    ```
- 打开游标

    ```
    OPEN <游标名>
    ```
- 关闭游标

    ```
    CLOSE <游标名>
    ```
- 使用游标

    ```
    -- 将游标中的值赋值给变量
    FETCH [NEXT | PRIOR | FIRST | LAST] FROM <游标名> INTO [变量名1,变量名2,变量名3,...]
    ```
    
## 问题

- 通过 `FETCH` 获取游标中的数据值为 `NULL`
    定义游标中的`select`语句，涉及的表需添加别名，通过别名查询数据。





