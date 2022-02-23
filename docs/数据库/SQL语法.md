

## SELECT
```sql
SELECT column_name,column_name
FROM table_name;
```

## SELECT DISTINCT

在表中，一个列可能会包含多个重复值，`DISTINCT` 关键词用于返回唯一不同的值。


```sql
SELECT DISTINCT column_name,column_name
FROM table_name;
```

## WHERE 

WHERE 子句用于提取那些满足指定条件的记录。

```sql
SELECT column_name,column_name
FROM table_name
WHERE column_name operator value;
```

## AND & OR

如果第一个条件和第二个条件都成立，则 AND 运算符显示一条记录。

如果第一个条件和第二个条件中只要有一个成立，则 OR 运算符显示一条记录。

## ORDER BY 

用于对结果集按照一个列或者多个列进行排序,默认按照升序,降序使用`DESC`

```sql
SELECT column_name,column_name
FROM table_name
ORDER BY column_name,column_name ASC|DESC;
```

## INSERT INTO

INSERT INTO 语句用于向表中插入新记录。

INSERT INTO 语句可以有两种编写形式。

第一种形式无需指定要插入数据的列名，只需提供被插入的值即可：

```sql
INSERT INTO table_name
VALUES (value1,value2,value3,...);
```
第二种形式需要指定列名及被插入的值：

```sql
INSERT INTO table_name (column1,column2,column3,...)
VALUES (value1,value2,value3,...);
```

## 连接 join

用于把不同的表连接起来

* **INNER JOIN**：如果表中有至少一个匹配，则返回行(注释：INNER JOIN 与 JOIN 是相同的。)
* **LEFT JOIN**：即使右表中没有匹配，也从左表返回所有的行
* **RIGHT JOIN**：即使左表中没有匹配，也从右表返回所有的行
* **FULL JOIN**：只要其中一个表中存在匹配，则返回行

```sql
SELECT column_name(s)
FROM table_name1
INNER JOIN table_name2 
ON table_name1.column_name=table_name2.column_name
```


## Union

UNION 操作符用于合并两个或多个 SELECT 语句的结果集。

请注意，UNION 内部的 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每条 SELECT 语句中的列的顺序必须相同。(默认地，UNION 操作符选取不同的值。如果允许重复的值，请使用 UNION ALL)

另外，UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。

```sql
SELECT column_name(s) FROM table_name1
UNION
SELECT column_name(s) FROM table_name2
```

# Constraints(约束)

约束用于限制加入表的数据的类型。

可以在创建表时规定约束（通过 CREATE TABLE 语句），或者在表创建之后也可以（通过 ALTER TABLE 语句）。

## NOT NULL
NOT NULL 约束强制列不接受 NULL 值。
## UNIQUE
UNIQUE 约束唯一标识数据库表中的每条记录。

UNIQUE 和 PRIMARY KEY 约束均为列或列集合提供了唯一性的保证。

PRIMARY KEY 拥有自动定义的 UNIQUE 约束。

请注意，每个表可以有多个 UNIQUE 约束，但是每个表只能有一个 PRIMARY KEY 约束
## PRIMARY KEY
PRIMARY KEY 约束唯一标识数据库表中的每条记录。

主键必须包含唯一的值。

主键列不能包含 NULL 值。

每个表都应该有一个主键，并且每个表只能有一个主键。
## FOREIGN KEY
一个表中的 FOREIGN KEY 指向另一个表中的 PRIMARY KEY。

FOREIGN KEY 约束用于预防破坏表之间连接的动作。

FOREIGN KEY 约束也能防止非法数据插入外键列，因为它必须是它指向的那个表中的值之一
## CHECK
CHECK 约束用于限制列中的值的范围。

如果对单个列定义 CHECK 约束，那么该列只允许特定的值。

如果对一个表定义 CHECK 约束，那么此约束会在特定的列中对值进行限制。
## DEFAULT

DEFAULT 约束用于向列中插入默认值。

如果没有规定其他的值，那么会将默认值添加到所有的新记录。

# 函数

## GROUP BY

GROUP BY 语句用于结合合计函数，根据一个或多个列对结果集进行分组。

```sql
SELECT column_name, aggregate_function(column_name)
FROM table_name
WHERE column_name operator value
GROUP BY column_name
```

## HAVING

在 SQL 中增加 HAVING 子句原因是，WHERE 关键字无法与合计函数一起使用。


