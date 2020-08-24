## 索引

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

