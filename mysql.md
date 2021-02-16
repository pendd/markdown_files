## 索引

`索引`是帮助MySQL高效获取数据的`数据结构`，实现了高级查找`算法`的数据结构，索引一般以文件形式存储在`磁盘`上

> EXPLAIN

> 概要描述：

```mysql
id:选择标识符
select_type:表示查询的类型。
table:输出结果集的表
partitions:匹配的分区
type:表示表的连接类型
possible_keys:表示查询时，可能使用的索引
key:表示实际使用的索引
key_len:索引字段的长度
ref:列与索引的比较
rows:扫描出的行数(估算的行数)
filtered:按表条件过滤的行百分比
Extra:执行情况的描述和说明
```

知乎介绍：https://zhuanlan.zhihu.com/p/127948040

> 查看索引

```mysql
SHOW INDEXES IN 表名;
```

// 缺个图片  

```mysql
Collation：表示排序 A 升序  B 降序
Cardinality： 表示索引中唯一值的数量  近视值
```

> ANALYZE

```mysql
ANALYZE TABLE 表名;
```

> 创建索引

```mysql
CREATE INDEX idx_ ON 表名(字段1，字段2);
```

> 找出最佳size 前置字符

```mysql
CREATE INDEX idx_lastname ON customers (last_name(size));

SELECT
	COUNT(DISTINCT LEFT(last_name, 1)),  -- 25
	COUNT(DISTINCT LEFT(last_name, 5)),  -- 966
	COUNT(DISTINCT LEFT(last_name, 10))  -- 996
FROM customers;

-- 所以上面的size 取5
```

> 全文检索

```mysql
-- 创建全文检索索引
CREATE FULLTEXT INDEX idx_title_body ON posts (title, body);

-- 检索文章标题或正文中含react redux中某个词   自然模式
-- MATCH(title, body) AGAINST('react redux') 表示关联度
SELECT *, MATCH(title, body) AGAINST('react redux')
FROM posts
WHERE MATCH(title, body) AGAINST('react redux');

-- BOOLEAN MODE 
-- 检索标题或正文中含有react 且没有redux 必须有form的文章
SELECT *
FROM posts
WHERE MATCH(title, body) AGAINST('react -redux +form' IN BOOLEAN MODE);

-- 精确查找  加""
SELECT *
FROM posts
WHERE MATCH(title, body) AGAINST('"Handling a Form"');
```

> 索引优先级

```mysql
-- 优先级：最常用属性 > 
CREATE INDEX idx_state ON customers (state);
CREATE INDEX idx_points ON customers(points);

EXPLAIN 
	SELECT customer_id FROM customers
    WHERE state = 'CA' 
    or points > 1000;
    
-- 可以优化为下面   

EXPLAIN
	SELECT customer_id FROM customers
    WHERE state = 'CA' 
    UNION
    SELECT customer_id FROM customers
    WHERE points > 1000;
```

> 查询上个查询的消耗

```mysql
SHOW STATUS;  -- 查询系统变量

SHOW STATUS LIKE 'last_query_cost';  -- 展示上个查询的消耗
```

> 索引 -- 排序

```mysql
-- 存在索引 （a,b)
-- 则下面几种情况排序是用到索引的，所以效率会比较高
-- 1. a
-- 2. a, b
-- 3. a DESC, b DESC
-- 升序和降序混着来是不ok的
-- b 单独的b是不可以的 但是可以修改为以下
-- WHERE a = '' ORDER BY b 这种情况下是用到了索引（a,b）
```

## Mysql 逻辑分层

```mysql
-- 服务端自上而下分为四层

-- 1. 连接层 
		-- 提供与客户端连接的服务
-- 2. 服务层
		-- 提供各种用户使用的接口 （select、update ...)
		-- 提供SQL优化器 （MySQL Query Optimizer）
-- 3. 引擎层
    -- 提供了各种存储数据的方式（InnoDB MyISAM）
-- 4. 存储层
    -- 存储数据
```



### 存储引擎

InnoDB ：事务优先 （适合高并发操作 行锁）

MyISAM：性能优先 （表锁）

## SQL优化

> SQL 优化主要就是 **优化索引**

索引：是帮助MySQL高效获取数据的数据结构。

索引是数据结构（树：B树（默认）、Hash树）

```mysql
-- 编写过程
-- select distinct from join on where group by having order by limit  
-- 执行过程
-- from on join where group by having select distinct order by limit
```

> 索引的弊端

```mysql
-- 1. 索引本身很大， 可以存放在内存/硬盘（通常为硬盘）
-- 2. 索引不是所有情况均适用：
		-- a. 少量数据
		-- b. 频繁更新的字段
		-- c. 不常用的字段
-- 3. 索引会降低增删改的效率（因为增删改数据的同时要把索引树中的数据给更新掉）
```

> 索引的优势

```mysql
-- 1. 提高查询效率（降低IO使用率）
-- 2. 降低CPU使用率
```

![](https://github.com/pendd/picture/raw/master/Mysql/索引.png)

> 索引的分类

```mysql
-- 1. 单值索引：单列
-- 2. 唯一索引： 
-- 3. 复合索引：多列构成
```

> 创建索引

```mysql
-- 方式一：
-- 单值索引
CREATE INDEX 索引名 ON 表名（字段）
-- 唯一索引
CREATE UNIQUE 索引名 ON 表名（字段）
-- 复合索引
CREATE INDEX 索引名 ON 表名（字段1，字段2）
```

```mysql
-- 方式二：
ALTER TABLE 表名 ADD 索引类型 索引名（字段）
```

### 如何才能使用索引，提高索引命中率

**索引是为了提高检索数据的速度，一般查询的时候会根据我们where后的条件字段是否存在索引来触发索引的使用**

> 索引生效不生效案例

```mysql
-- 不会使用索引，索引列参与了计算
  SELECT filename FROM projectfile WHERE id + 29 = 30;
-- 不会使用索引，因为使用了函数运算
	SELECT filename FROM projectfile WHERE LEFT(data,4) < 1990;
	
-- 走索引
SELECT * FROM projectfile WHERE filename LIKE 'php%';
-- 不走索引
SELECT * FROM projectfile WHERE filename LIKE '%php%';

-- 正则表达式不使用索引，这也是为什么SQL中很难见到regexp关键字的原因
```

```mysql
-- 字符串与数字比较不使用索引  a char(10)
SELECT * FROM a WHERE a = "1";  -- 走索引

SELECT * FROM a WHERE a = 1;   -- 不走索引 因为转换用到了函数计算
```

如果where条件中含有or，即使其中有条件带索引也不会使用，所以应该避免使用or关键字。

MySQL 内部优化器会对SQL语句进行优化，如果优化器预估使用全表扫描要比使用索引快，则就会不使用索引。

> 组合索引遵循“最左前缀”原则

> 有组合索引（name，city，age）

```mysql
-- 下面两个用到索引
SELECT * FROM myIndex WHERE name = "erquan" AND city = "郑州";
SELECT * FROM myIndex WHERE name = "erquan";
-- 下面两个则用不到索引
SELECT * FROM myIndex WHERE age = 20 AND city = "郑州";
SELECT * FROM myIndex WHERE city = "郑州";
```

也就是说（name，city，age）从左到右进行索引，如果查询条件没有能够匹配第一个则不执行索引查询。