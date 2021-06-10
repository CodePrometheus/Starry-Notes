# SQL优化

[TOC]



## 通过explain查询和分析SQL的执行计划

```mysql
explain select .. from ...
```

![image-20210319193742715](images/68747470733a2f2f67697465652e636f6d2f73757065722d6a696d77616e672f696d672f7261772f6d61737465722f696d672f32303231303331393139333734322e706e67)



- type访问类型。比如唯一索引，非唯一索引，全表匹配之类的。
  1. SIMPLE   表示当前查询为最简单的查询，不包含任何子查询SUBQUERY和联合查询UNION
  2. PRIMARY   如果查询中包含任何复杂的部分，则最外层部分的查询过程被标记为PRIMARY
  3. SUBQUERY   不在FROM子句中的复杂子查询（如SELECT子句、WHERE子句等）都会被标记为SUBQUERY
  4. DERIVED   DERIVED值用来表示包含在FROM子句中的复杂子查询，MySQL会递归执行这些子查询，并将结果放置在临时表中
  5. UNION   对于联合查询UNION中，第二个及之后的SELECT查询都会被标记为UNION。如果UNION查询被FROM子句中的子查询包含，那么UNION查询中的第一个SELECT语句会被标记为DERIVED
  6. UNION RESULT   用来从UNION的匿名临时表中检索结果的SELECT被标记为UNION RESULT

- extra除了前几列之外的重要信息。比如using index，using where提示可以使用的优化。

- key_len使用了索引的长度，越短越好。



