# 概览

本文档用于记录Mysql中的参数及其含义

## 1 wait_timeout

表示客户端和服务器端产生连接之后处于sleep状态的时长，如果超过这个时长，则自动断开服务端和客户端  
的连接，默认情况是8小时

## 2 query_cache_type

表示是否需要查询Mysql缓存。MySql 8.x之后已经不支持此参数，因为已经将缓存模块去掉了

## 3 innodb_flush_log_at_trx_commit

用于指定几次事务提交之后同步到磁盘中，如果设置为1，那么数据就不会丢失

## 4 sync_binlog

用于指定几次事务之后将binlog同步到磁盘，建议设置为1，可以保证binlog不丢失