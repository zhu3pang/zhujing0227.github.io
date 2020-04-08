---
layout: post
title: Mysql tips
categories: mysql
tags: [mysql]
comments: true
---


# Group By 是否允许 SELECT 非聚合列

先给出官方的结论： [slqmode\_only\_full\_group\_by](https://dev.mysql.com/doc/refman/5.7/en/sql-mode.html#sqlmode_only_full_group_by)，5.7.5 之后的版本默认是
「ONLY\_FULL\_GROUP\_BY」，即不允许非聚合列在 select 语句中；5.7.5 之前的版本默认不支
持「ONLY\_FULL\_GROUP\_BY」，即允许非聚合列在 select 语句中。当然，非要 select 非聚合列
也有办法的：<https://blog.csdn.net/Dax1n/article/details/86581472>

1.  「ONLY\_FULL\_GROUP\_BY」开启下，使用[ANY\_VALUE](https://dev.mysql.com/doc/refman/5.7/en/miscellaneous-functions.html#function_any-value)(非聚合列)进行查询；
2.  5.7.5 以后的版本可以将「ONLY\_FULL\_GROUP\_BY」属性关闭。


## 查看 sql\_mode 的变量值

    show variables like 'sql_mode';

    ONLY_FULL_GROUP_BY,STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION


## 表结构

    show create table TABLE_PARAMS;

    CREATE TABLE `TABLE_PARAMS` (
      `TBL_ID` bigint(20) NOT NULL,
      `PARAM_KEY` varchar(256) CHARACTER SET latin1 COLLATE latin1_bin NOT NULL,
      `PARAM_VALUE` mediumtext CHARACTER SET latin1 COLLATE latin1_bin,
      PRIMARY KEY (`TBL_ID`,`PARAM_KEY`),
      KEY `TABLE_PARAMS_N49` (`TBL_ID`),
      CONSTRAINT `TABLE_PARAMS_FK1` FOREIGN KEY (`TBL_ID`) REFERENCES `TBLS` (`TBL_ID`)
    ) ENGINE=InnoDB DEFAULT CHARSET=latin1


## 原数据

    select TBL_ID,PARAM_KEY from TABLE_PARAMS where PARAM_KEY in ('numFiles','numRows');

    
    | TBL_ID | PARAM_KEY |
    |--------|-----------|
    |      1 | numFiles  |
    |      1 | numRows   |
    |      2 | numFiles  |
    |      2 | numRows   |
    |      3 | numFiles  |
    |      3 | numRows   |


## GROUP BY 查询

    select TBL_ID,PARAM_KEY
    from TABLE_PARAMS
    where PARAM_KEY in ('numFiles','numRows')
    group by PARAM_KEY;

此时聚合查询直接报异常：

    [42000][1055] Expression #1 of SELECT list is not in GROUP BY clause and
    contains nonaggregated column 'hadoop_hive.TABLE_PARAMS.TBL_ID' which is not
    functionally dependent on columns in GROUP BY clause; this is incompatible with
    sql_mode=only_full_group_by 

-   去除 ONLY\_FULL\_GROUP\_BY
    
        set sql_mode = 'STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION';
-   再聚合查询
    
        select TBL_ID,PARAM_KEY
        from TABLE_PARAMS
        where PARAM_KEY in ('numFiles','numRows')
        group by PARAM_KEY;

查询结果实际上是分组后取第一条数据的非聚合列。

    | TBL_ID | PARAM_LEY |
    |--------|-----------|
    |      1 | numRows   |
    |      1 | numFiles  |

