
# 概览

EXPLAIN语句产生额外的（“扩展”）信息，这些信息不是EXPLAIN输出的一部分，但可以通过在EXPLAIN之后发出SHOW WARNINGS语句来查看。 从MySQL 8.0.12开始，扩展信息可用于SELECT，DELETE，INSERT，REPLACE和UPDATE语句。 在8.0.12之前，扩展信息仅适用于SELECT语句。

SHOW WARNINGS输出中的Message值显示优化程序如何限定SELECT语句中的表名和列名，SELECT应用重写和优化规则后的样子，以及可能有关优化过程的其他说明。

可以使用EXPLAIN之后的SHOW WARNINGS语句显示的扩展信息仅针对SELECT语句生成。 SHOW WARNINGS显示其他可解释语句的空结果（DELETE，INSERT，REPLACE和UPDATE）。

以下是输出样例：
<pre>
mysql > EXPLAIN
       SELECT t1.a, t1.a IN (SELECT t2.a FROM t2) FROM t1\G
*************************** 1. row ***************************
           id: 1
  select_type: PRIMARY
        table: t1
         type: index
possible_keys: NULL
          key: PRIMARY
      key_len: 4
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: Using index
*************************** 2. row ***************************
           id: 2
  select_type: SUBQUERY
        table: t2
         type: index
possible_keys: a
          key: a
      key_len: 5
          ref: NULL
         rows: 3
     filtered: 100.00
        Extra: Using index
2 rows in set, 1 warning (0.00 sec)

mysql> SHOW WARNINGS\G
*************************** 1. row ***************************
  Level: Note
   Code: 1003
Message: /* select#1 */ select `test`.`t1`.`a` AS `a`,
    <in_optimizer>(`test`.`t1`.`a`,`test`.`t1`.`a` in
    ( <materialize> (/* select#2 */ select `test`.`t2`.`a`
    from `test`.`t2` where 1 having 1 ),
    <primary_index_lookup>(`test`.`t1`.`a` in
    <temporary table> on <auto_key>
    where ((`test`.`t1`.`a` = `materialized-subquery`.`a`))))) AS `t1.a
    IN (SELECT t2.a FROM t2)` from `test`.`t1`
1 row in set (0.00 sec)
</pre>

由于SHOW WARNINGS显示的语句可能包含特殊标记以提供有关查询重写或优化程序操作的信息，因此该语句不一定是有效的SQL，也不打算执行。 输出还可能包含具有Message值的行，这些行提供有关优化程序所执行操作的其他非SQL解释性说明。

以下列表描述了可以出现在SHOW WARNINGS显示的扩展输出中的特殊标记：

* auto_key

为临时表自动创建的key

* cache

表达式（例如标量子查询）执行一次，结果值保存在内存中供以后使用。 对于由多个值组成的结果，可能会创建一个临时表，您将看到<临时表>。

* 