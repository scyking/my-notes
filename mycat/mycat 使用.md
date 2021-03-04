# mycat 使用

标签（空格分隔）： mycat mysql

---

> [mycat 源码地址](https://github.com/MyCATApache/Mycat-Server)

## mycat 概述

 > 是一个开源的分布式数据库系统，一个实现了 MySQL 协议的的 Server。前端用户可以把它看作是一个数据库代理，用 MySQL客户端工具和命令行访问，而其后端可以用 MySQL 原生（Native）协议与多个 MySQL服务器通信，也可以用 JDBC 协议与大多数主流数据库服务器通信。其核心功能是 **分表分库**，即将一个大表水平分割为 N 个小表，存储在后端 MySQL 服务器里或者其他数据库里。

### 数据切分

指通过某种特定的条件，将存放在同一个数据库中的数据分散存放到多个数据库（主机）上面，以达到分散单台设备负载的效果。

- 水平切分
按照某个字段的某种规则来分散到多个库之中，每个表中包含一部分数据。
- 垂直切分
按照业务将表进行分类，分布到不同的数据库上面，这样也就将数据或者说压力分担到不同的库上面。

### 应用场景

- 单纯的**读写分离**，此时配置最为简单
- 分表分库
- 多租户应用，每个应用一个库，但应用程序只连接mycat，从而不改造程序本身，实现多租户化
- 报表系统，借助于mycat的分表能力，处理大规模报表的统计
- 替代hbase，分析大数据
- 作为海量数据实时查询的一种简单有效方案

### 不适用场景

> [参考地址](https://blog.csdn.net/gaobudong1234/article/details/79581846)

- 非分片字段查询
mycat中的**路由**结果是通过**分片字段**和**分片方法**确定的。
- 分页排序
为处理有偏移量的排序分页，通过改写SQL解决结果集错误，资源消耗会随偏移量增大而增加。
- 任意表JOIN
两张表关联数据可能分布在不同的DB节点上，在单独分片DB中查询得不到正确结果。
- 分布式事务
Mycat并没有根据二阶段提交协议实现**XA事务**，而是只保证 **prepare** 阶段数据一致性的**弱XA事务**。

## 环境搭建

### windows下使用

> [github 参考地址](https://github.com/MyCATApache/Mycat-Server/wiki/2.0-Mycat%E5%AE%89%E8%A3%85%E4%B8%8E%E4%BD%BF%E7%94%A8)

1. [下载地址](https://github.com/MyCATApache/Mycat-download )，选择合适版本下载。
2. 直接运行 `startup_nowrap.bat`

### linux 下使用

暂略

### 三大配置文件

在安装目录中`conf`文件夹下存放。

1. `server.xml`：是mycat服务器参数调整和用户授权的配置文件。
2. `schema.xml`：管理着 MyCat 的逻辑库（`schema`）、表、分片规则、`DataNode` 以及 `DataSource`。
3. `rule.xml`：是分片规则的配置文件。分片规则的具体参数信息单独存放为文件，也在这个目录下，配置文件修改需要重启mycat。

## mycat 高可用与集群（主从 + 读写分离）

1. mysql节点开启主从复制的配置方案
    - 主数据库master修改
        - 修改`my.ini`（mysql配置文件）后，重启mysql。(mysql安装目录中没有，如果是windows系统则可能在C盘隐藏目录`ProgramData`下)
        
        ```
        [mysqld]
        log-bin=mysql-bin # 开启二进制日志
        server-id=1 # 设置server-id
        
        # 默认记录所有库操作
        # 不同步哪些数据库  
        binlog-ignore-db = mysql  
        binlog-ignore-db = test  
        binlog-ignore-db = information_schema  
  
        # 只同步哪些数据库，除此之外，其他不同步  
        binlog-do-db = game
        ```
        - 创建授权用户
        
        ```
        CREATE USER '用户名'@'从数据库Ip' IDENTIFIED BY '密码'; # 创建用户
        GRANT REPLICATION SLAVE ON *.* TO '用户名'@'从数据库IP'; # 分配权限
        flush privileges;   # 刷新权限
        ```
        - 查看master状态，记录二进制文件名和位置
        
        ```
        SHOW MASTER STATUS;
        ```
        
    - 从数据库slave修改
        - 修改`my.ini`（mysql配置文件）后，重启mysql。
        
        ```
        [mysqld]
        server-id=2 #设置server-id，必须唯一
        ```
        - 执行同步SQL语句(需要主服务器主机名，登陆凭据，二进制文件的名称和位置)
        
        ```
        CHANGE MASTER TO
        MASTER_HOST='主数据库IP',
        MASTER_USER='用户名',
        MASTER_PASSWORD='密码',
        MASTER_LOG_FILE='mysql-bin.000003', -- 由上，查看master状态得到
        MASTER_LOG_POS=73; -- 由上，查看master状态得到
        ```
        - 启动slave同步进程
        
        ```
        start slave; # 开启
        stop slave;  # 关闭
        ```
        - 查看slave状态
        
        ```
        show slave status\G;
        ```
        `Slave_IO_Running`、`Slave_SQL_Running`都为`YES`时表示主从同步设置成功。
        
    - 注意事项
        - 从库同步主库事务操作与二进制文件的名称和位置有关。
        - 初始化配置时，主库与从库数据需一致。（不一致数据不会自动同步，会造成同一查询得到不同结果）
        - 初始化配置时，主库与从库表结构需一致。（操作主库不同表结构时会引起从库同步异常，需重启从库同步）
    
1. 修改mycat配置文件`schema.xml`，配置逻辑库

    - 将**主节点**配置为mycat的`dataHost`里的`writeNode`
    - 将**从节点**配置为mycat的`dataHost`里的`readNode`
    
    ```
    <mycat:schema xmlns:mycat="http://io.mycat/">
	<!-- 逻辑库配置 -->
	<schema name="DB1" checkSQLschema="false" sqlMaxLimit="100" dataNode="dn1"/> 
	
	<!-- 节点配置 -->
	<dataNode name="dn1" dataHost="host01" database="testdb" />
	<!-- 数据库实例：读写分离的配置 -->
	<dataHost name="host01" maxCon="1000" minCon="10" balance="1" 
          writeType="0" dbType="mysql" dbDriver="native">
        <heartbeat>select user()</heartbeat> 
        <!-- 主库 master 配置，可多个-->
        <writeHost host="hostM1" url="192.168.20.113:3306" user="test" password="!QAZ2wsx"> 
            <!-- 从库 slave 配置，可多个 -->          
            <readHost host="hostS1" url="192.168.20.107:3306" user="test" password="!QAZ2wsx" />
        </writeHost> 
        <!--主故障，顶替写节点,主正常是分担读压力--> 
        <!-- 
            <writeHost host="hostS2" url="" user="root" password="" > </writeHost> 
        -->
    </dataHost> 
    </mycat:schema>
    ```
    
1. 修改mycat配置文件`server.xml`，添加 mycat 用户

    ```
    <user name="root" defaultAccount="true">
		<property name="password">123456</property>
		<property name="schemas">DB1</property>
	</user>

	<user name="user">
		<property name="password">user</property>
		<property name="schemas">DB1</property>
		<property name="readOnly">true</property>
	</user>
    ```
    
4. 使用完全与mysql相同。（PS：注意mycat默认端口号是`8066`） 

## 相关问题

- 连接时报错 `ERROR 1129 (00000): #HY000Host ‘192.168.31.242’ is blocked because of many connection errors; unblock with ‘mysqladmin flush-hosts’`

    原因是同一个`ip`在短时间内产生太多中断的数据库连接而导致阻塞。在mysql终端执行 `mysqladmin -u root -p flush-hosts`清除缓存后可进行连接。（导致原因需自行查找解决）
    
- 报错 `... connect error MySQLConnection ... Unknown charsetIndex:255 `

    修改mycat字符集配置文件`index_to_charset.properties`，添加`255=utf8mb4`，重启服务。
    
- 报错 `... Access denied for user ...`

    检查用户权限；检查 `dataNode` 上配置实体库（`database` 值）是否存在；
    
## 其它轻量级组件

- `sharding-jdbc`：朋友推荐的轻量级组件，需集成到程序中，待研究。








