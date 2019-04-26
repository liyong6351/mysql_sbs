<!-- TOC -->

- [Explain概览](#explain%E6%A6%82%E8%A7%88)
  - [EXPLAIN Output Columns(Explain 输出列)](#explain-output-columnsexplain-%E8%BE%93%E5%87%BA%E5%88%97)
    - [1. id](#1-id)
    - [2. select_type](#2-selecttype)
    - [3. table (JSON_NAME select_table)](#3-table-jsonname-selecttable)
    - [4. partitions](#4-partitions)
    - [5. type](#5-type)
    - [6. possible_keys](#6-possiblekeys)
    - [7. key](#7-key)
    - [8. key_len](#8-keylen)
    - [9. ref](#9-ref)
    - [10. rows](#10-rows)
    - [11. filtered](#11-filtered)
    - [12. Extra](#12-extra)
  - [EXPLAIN Join Types(Explain 连接类型)](#explain-join-typesexplain-%E8%BF%9E%E6%8E%A5%E7%B1%BB%E5%9E%8B)
    - [1. system](#1-system)
    - [2. const](#2-const)
    - [3. eq_ref](#3-eqref)
    - [4. ref](#4-ref)
    - [5. fulltext](#5-fulltext)
    - [6. ref_or_null](#6-refornull)
    - [7. index_merge](#7-indexmerge)
    - [8.unique_subquery](#8uniquesubquery)
    - [9. index_subquery](#9-indexsubquery)
    - [10. range](#10-range)
    - [11. index](#11-index)
    - [12.ALL](#12all)
  - [EXPLAIN Extra Information(Explain 其他类型)](#explain-extra-informationexplain-%E5%85%B6%E4%BB%96%E7%B1%BB%E5%9E%8B)
  - [EXPLAIN Output Interpretation](#explain-output-interpretation)

<!-- /TOC -->

# Explain概览

https://dev.mysql.com/doc/refman/8.0/en/explain-output.html

explain可以打印出当前的语句在SQL中如何执行的信息。支持的关键字包括:select、delete、insert、replace和update

EXPLAIN为SELECT语句中使用的每个表返回一行信息。他会按照Mysql执行的顺序输出执行信息。MySQL使用嵌套循环连接方法解析所有连接。意思就是说mysql从驱动表读取一行，然后到第二张第三张表获取到匹配的数据。当所有的表都被执行之后，MySQL通过表列表输出所选列和回溯，直到找到有更多匹配行的表。 从该表中读取下一行，并继续下一个表。

## EXPLAIN Output Columns(Explain 输出列)

本小节讨论explain输出列的信息，后面两节讨论输出列的类型和扩展信息。  
EXPLAIN的每个输出行都提供有关一个表的信息。每一行都包含8.1表中的列。列名显示在表的第一列中; 第二列提供使用FORMAT = JSON时输出中显示的等效属性名称。

8.1表
|Column|JSON Name|Meaning|
|--|--|--|
|id|select_id|Select 的标识符|
|select_type|None|Select的类型|
|table|table_name|输出的表名|
|partitions|partitions|匹配的分区|
|type|access_type|连接的类型|
|possible_keys|possible_keys|可能选择的索引|
|key|key|实际选择的索引|
|key_len|key_length|索引的长度|
|ref|ref|与连接进行比较的列|
|rows|rows|估计进行计算的列数|
|filtered|filtered|按表条件过滤的行的百分比|
|Extra|None|额外的信息|

说明

### 1. id

Select的标识符，表示在sql执行中的顺序。如果行引用其他行的并集结果，则该值可以为NULL。比如在<unionM,N>这种情况下，表示指向集合的N或者M的id

### 2. select_type

Select 的类型，见下表

|Select_Type Value|JSON Name|meaning|
|----|----|----|
|SIMPLE|None|simple select(没有union和子查询)|
|Primary|None|最外面的Select|
|Union|None|在Union中第二个|
|DEPENDENT UNION|dependent (true)|UNION中的第二个或更高的SELECT语句，取决于外部查询|
|UNION RESULT|union_result|Union结果集(两次查询之后合并)|
|SUBQUERY|None|第一个子查询|
|DEPENDENT SUBQUERY|dependent (true)|依赖于外部查询的第一个子查询|
|DERIVED|None|临时表|
|DEPENDENT DERIVED|dependent (true)|依赖于外部表的临时表|
|MATERIALIZED|materialized_from_subquery|被物化的子查询|
|UNCACHEABLE SUBQUERY|cacheable (false)|未被缓存的子查询|
|UNCACHEABLE UNION|cacheable (false)|未被缓存的union数据集|

* DEPENDENT SUBQUERY评估与UNCACHEABLE SUBQUERY评估不同。 对于DEPENDENT SUBQUERY，子查询仅针对来自其外部上下文的变量的每组不同值重新评估一次。 对于UNCACHEABLE SUBQUERY，将为外部上下文的每一行重新评估子查询。

### 3. table (JSON_NAME select_table)

数据来源的表名，有以下几种来源

|来源类型|说明|
|--|--|
|<union M,N>|数据从Union的两张表中获取，M、N|
|derived M|数据来源是派生表M，派生表可能是一个子查询|
|subqueryN|数据来源是子查询|

### 4. partitions

命中的哪一个数据分区

### 5. type

连接的类型，我们将有专门的一个小节讲解select_type

### 6. possible_keys

在Mysql表中可能使用到的索引，请注意，此列完全独立于EXPLAIN输出中显示的表的顺序。  
如果此列的值为null，那么可能并不存在可以使用的索引，这个时候就需要检查where子句，以保证使用到索引。如果有必要的话需要创建索引，并且再次执行explain命令。

查看表索引情况的查询语句为 show index from tabelName

### 7. key

Mysql在执行过程中实际使用的索引。如果Mysql决定使用possible_key中的某一种类型，那么在此列中会列举出来。  
最终的查询可能会命名一个不存在于possible_keys值中的索引。 如果所有possible_keys索引都不适合查找行，则会发生这种情况，但查询选择的所有列都是其他索引的列。 也就是说，命名索引覆盖了所选列，因此虽然它不用于确定要检索的行，但索引扫描比数据行扫描更有效。

对于InnoDB，即使查询还选择主键，二级索引也可能覆盖所选列，因为InnoDB将主键值与每个辅助索引一起存储。 如果key为NULL，则MySQL找不到用于更有效地执行查询的索引。

如果要强行使用某个索引或者强行忽略某个索引，使用sql hint: force index | use index | ignore index  

对于MyIsam表的话，使用Analyze table可以查找是否有更适合的索引

### 8. key_len

使用的index的长度，由于key存储格式，对于可以为NULL的列而言，key_len比对于NOT NULL列更大。

### 9. ref

显示索引的哪一列被使用了，有时候会是一个常量：表示哪些列或常量被用于用于查找索引列上的值

### 10. rows

rows列表示MySQL认为必须检查以执行查询的行数。

### 11. filtered

filtered列指示将按表条件筛选的表行的估计百分比。最大值为100，这意味着不会对行进行过滤。值从100开始减少表示过滤量增加。rows显示检查的估计行数，rows×filtered显示将与下表连接的行数。  
例如，如果行为1000且过滤为50.00（50％），则使用下表连接的行数为1000×50％= 500。

### 12. Extra

此列包含有关MySQL如何解析查询的其他信息。将有专门的小节讲解这个问题  

## EXPLAIN Join Types(Explain 连接类型)

type字段说明的是数据如何join的。下面将从最好到最坏的情况讲解type类型:

### 1. system

表中只有一列，这是常量join的特例

### 2. const

该表最多只有一个匹配行，在查询开头读取。 因为只有一行，所以优化器的其余部分可以将此行中列的值视为常量。 const表非常快，因为它们只读一次。

<pre>
SELECT * FROM tbl_name WHERE primary_key=1;

SELECT * FROM tbl_name
  WHERE primary_key_part1=1 AND primary_key_part2=2;
</pre>

### 3. eq_ref

对于前面表格中的每个行组合，从该表中读取一行。 除了system和const类型之外，这是最好的连接类型。 当连接使用索引的所有部分并且索引是PRIMARY KEY或UNIQUE NOT NULL索引时使用它。

<pre>
SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column=other_table.column;

SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column_part1=other_table.column
  AND ref_table.key_column_part2=1;
</pre>

### 4. ref

不像eq_ref那样要求连接顺序，也没有主键和唯一索引的要求，只要使用相等条件检索时就可能出现，常见与辅助索引的等值查找。或者多列主键、唯一索引中，使用第一个列之外的列作为等值查找也会出现，总之，返回数据不唯一的等值查找就可能出现。

<pre>
SELECT * FROM ref_table WHERE key_column=expr;

SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column=other_table.column;

SELECT * FROM ref_table,other_table
  WHERE ref_table.key_column_part1=other_table.column
  AND ref_table.key_column_part2=1;
</pre>

### 5. fulltext

使用全文检索的连接类型

### 6. ref_or_null

这种连接类型类似于ref,但是区别在于本类型可以检查null，

<pre>
SELECT * FROM ref_table
  WHERE key_column=expr OR key_column IS NULL;
</pre>

### 7. index_merge

此连接类型表示使用了索引合并优化。 在这种情况下，输出行中的键列包含使用的索引列表，key_len包含所用索引的最长键部分列表。支持优化

### 8.unique_subquery

此类型替换以下形式的某些IN子查询的eq_ref：

<pre>
value IN (SELECT primary_key FROM single_table WHERE some_expr)
</pre>
unique_subquery只是一个索引查找函数，它可以完全替换子查询以提高效率。

### 9. index_subquery

此连接类型类似于unique_subquery。 它取代了IN子查询，但它适用于以下形式的子查询中的非唯一索引：

<pre>
value IN (SELECT key_column FROM single_table WHERE some_expr)
</pre>

### 10. range

仅检索给定范围内的行，使用索引选择行。 输出行中的键列指示使用哪个索引。 key_len包含使用的最长的关键部分。 对于此类型，ref列为NULL。

以下关键字都是范围查找: =, <>, >, >=, <, <=, IS NULL, <=>, BETWEEN, LIKE, 或者 IN()

<pre>
SELECT * FROM tbl_name
  WHERE key_column = 10;

SELECT * FROM tbl_name
  WHERE key_column BETWEEN 10 and 20;

SELECT * FROM tbl_name
  WHERE key_column IN (10,20,30);

SELECT * FROM tbl_name
  WHERE key_part1 = 10 AND key_part2 IN (10,20,30);
</pre>

### 11. index

索引连接类型与ALL相同，只是扫描索引树。 这有两种方式：

* 扫描索引树，这种比全表扫描效率高，extra中显示为Using index
* 全表扫描,当查询仅使用属于单个索引的列时，MySQL可以使用此连接类型。
  
### 12.ALL

全表扫描，效率最低，考虑优化

## EXPLAIN Extra Information(Explain 其他类型)

## EXPLAIN Output Interpretation

通过获取EXPLAIN输出的rows列中的值的乘积，可以很好地指示连接的好坏程度。这可以在一定情况下告诉你SQL检查的数据量。如果你在查询的时候使用max_join_size这个系统变量，他会告诉你多表查询的时候哪些被执行了哪些被丢弃掉了

以下示例显示如何根据EXPLAIN提供的信息逐步优化多表连接。
