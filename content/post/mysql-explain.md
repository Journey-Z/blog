---
title: "Mysql explain 语句各个字段解释"
date: 2020-03-16T18:28:45+08:00
categories:
- Mysql
tags:
- Mysql
keywords:
- Mysql
thumbnailImagePosition: left
thumbnailImage: img/mysql-logo.png

---
学会使用 explain 来分析 mysql 查询语句，对 sql 语句进行优化...
<!--more-->


### MySQL Explain语句字段解释和作用
1. #### 字段解释
   | 字段名        | 解释                                                         |
   | ------------- | ------------------------------------------------------------ |
   | id            | SELECT 查询的标识符. 每个 SELECT 都会自动分配一个唯一的标识符. |
   | select_type   | SELECT 查询的类型                                            |
   | table         | 查询的是哪个表                                               |
   | type          | 类型                                                         |
   | possible_keys | 此次查询中可能选用的索引                                     |
   | key           | 此次查询中确切使用到的索引.                                  |
   | ref:          | 哪个字段或常数与 key 一起被使用                              |
   | rows          | 显示此查询一共扫描了多少行. 这个是一个估计值.                |
   | filtered      | 表示此查询条件所过滤的数据的百分比                           |
   | extra         | 额外的信息                                                   |

2. #### select_type 表示了查询的类型, 它的常用取值有
   | 值                   | 解释                                                         |
   | -------------------- | ------------------------------------------------------------ |
   | **SIMPLE**           | 简单查询，该查询不包含 UNION 或子查询                        |
   | **PRIMARY**          | 如果查询包含UNION 或子查询，则**最外层的查询**被标识为PRIMARY |
   | UNION                | 表示此查询是 UNION 中的第二个或者随后的查询                  |
   | DEPENDENT            | UNION 满足 UNION 中的第二个或者随后的查询，其次取决于外面的查询 |
   | **SUBQUERY**         | 子查询中的第一个select语句(该子查询不在from子句中)           |
   | DEPENDENT SUBQUERY   | 子查询中的 第一个 select，同时取决于外面的查询               |
   | **DERIVED**          | 包含在from子句中子查询(也称为派生表)                         |
   | UNCACHEABLE SUBQUERY | 满足是子查询中的第一个 select 语句，同时意味着 select 中的某些特性阻止结果被缓存于一个 Item_cache 中 |
   | UNCACHEABLE UNION    | 满足此查询是 UNION 中的第二个或者随后的查询，同时意味着 select 中的某些特性阻止结果被缓存于一个 Item_cache 中 |
     加粗的最为常见

3. ##### table

     该列显示了对应行正在访问哪个表(有别名就显示别名)。

     当from子句中有子查询时，table列是 `<derivenN>`格式，表示当前查询依赖 `id=N`的查询，于是先执行 `id=N` 的查询

4. ##### type 该列称为关联类型或者访问类型，它指明了MySQL决定如何查找表中符合条件的行，同时是我们判断查询是否高效的重要依据，以下为常见取值：
   | 值            | 解释                                                         |
   | ------------- | ------------------------------------------------------------ |
   | ALL           | **全表扫描**，这个类型是性能最差的查询之一。通常来说，我们的查询不应该出现 ALL 类型，因为这样的查询，在数据量最大的情况下，对数据库的性能是巨大的灾难 |
   | index         | **全索引扫描**，和 ALL 类型类似，只不过 ALL 类型是全表扫描，而 index 类型是扫描全部的索引，主要优点是避免了排序，但是开销仍然非常大。如果在 Extra 列看到 Using index，说明正在使用覆盖索引，只扫描索引的数据，它比按索引次序全表扫描的开销要少很多 |
   | range         | **范围扫描**，就是一个有限制的索引扫描，它开始于索引里的某一点，返回匹配这个值域的行。这个类型通常出现在 `=、<>、>、>=、<、<=、IS NULL、<=>、BETWEEN、IN()` 的操作中，key 列显示使用了哪个索引，当 type 为该值时，则输出的 ref 列为 NULL，并且 key_len 列是此次查询中使用到的索引最长的那个 |
   | ref           | 一种索引访问，也称索引查找，它返回所有匹配某个单个值的行。此类型通常出现在多表的 join 查询, 针对于非唯一或非主键索引, 或者是使用了最左前缀规则索引的查询 |
   | eq_ref        | 使用这种索引查找，最多只返回一条符合条件的记录。在使用唯一性索引或主键查找时会出现该值，非常高效 |
   | const、system | 该表至多有一个匹配行，在查询开始时读取，或者该表是系统表，只有一行匹配。其中 const 用于在和 primary key 或 unique 索引中有固定值比较的情形 |
   | NULL          | 在执行阶段不需要访问表                                       |

5. ##### possible_keys

     这一列显示查询**可能**使用哪些索引来查找

6. ##### key

     这一列显示MySQL**实际**决定使用的索引。如果没有选择索引，键是NULL

7. ##### key_len

     这一列显示了在索引里使用的字节数，当key列的值为 NULL 时，则该列也是 NULL

8. ##### ref

     这一列显示了哪些字段或者常量被用来和key配合从表中查询记录出来

9. ##### rows

     这一列显示了**估计**要找到所需的行而要读取的行数，这个值是个估计值，原则上值越小越好。

10. ##### extra 其他的信息，常见取值如下
   | 值              | 解释                                                         |
   | --------------- | ------------------------------------------------------------ |
   | **Using index** | 使用覆盖索引，表示查询索引就可查到所需数据，不用扫描表数据文件，往往说明性能不错 |
   | Using Where     | 在存储引擎检索行后再进行过滤，使用了where从句来限制哪些行将与下一张表匹配或者是返回给用户 |
   | Using temporary | 在查询结果排序时会使用一个临时表，一般出现于排序、分组和多表 join 的情况，查询效率不高，建议优化 |
   | Using filesort  | 对结果使用一个外部索引排序，而不是按索引次序从表里读取行，一般有出现该值，都建议优化去掉，因为这样的查询 CPU 资源消耗大 |

     