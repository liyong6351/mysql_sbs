
# 概览

## Explain能告诉我们Mysql如何执行的信息

* explain 支持 select、insert、delete、replace和update关键字
* 当对一条语句使用explain时，他会打印出当前优化器最终的执行计划。也就是mysql如何执行这条语句的信息，包括表连接及执行的顺序。使用explain最典型的就是获取执行计划
* 当EXPLAIN与FOR CONNECTION connection_id一起使用而不是Sql语句时，它将显示在命名连接中执行的语句的执行计划。
* 对于Select的执行计划，可以使用show warning命令获取到执行计划的额外信息
* Explain还可以获取到对于分区表的执行计划
* FORMAT选项可用于选择输出格式。 TRADITIONAL以表格格式显示输出。 如果不存在FORMAT选项，则这是默认值。 JSON格式以JSON格式显示信息。

<pre>
使用explain你可以发现是否需要增加索引，用于提高sql语句的执行效率。也可以使用explain获取到表连接的时候是否是最优方案。要提示优化器使用与SELECT语句中命名表的顺序相对应的连接顺序，请使用SELECT STRAIGHT_JOIN而不是SELECT来开始语句。但是，STRAIGHT_JOIN可能会阻止使用索引，因为它会禁用半连接转换。  
优化器跟踪有时可以提供与EXPLAIN的信息互补的信息。 但是，优化程序跟踪格式和内容可能会在不同版本之间发生变化。
如果在您认为应该使用索引时遇到问题，请运行ANALYZE TABLE以更新可能影响优化程序所做选择的表统计信息，例如键的基数。
</pre>