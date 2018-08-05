---
title: OrderBy字符串排序规则和在InnoDB中的性能优化
date: 2018-08-05 17:49:48
tags: MySQL
---
#OrderBy字符串排序规则和在InnoDB中的性能优化

## Order by字符串排序规则
对于字符串来讲，排序规则可以在配置文件中my.ini中指定，主要有以下几个种类的排序规则：
（1） **utf8_general_cs** 和 **utf8_general_ci** （后缀"_cs"或者"_ci"意思是区分大小写和不区分大小写（Case Sensitive & Case Insensitve））
（2） **utf8_bin** 规定每个字符串用二进制编码存储，区分大小写，可以直接存储二进制的内容
默认采取的utf8_general_ci **不区分大小写**，并且是按照**逐位比较**，如果相同再比较下一位的方法；如果想按照int方法排序，可以**order by varchar+0**。

## Order by性能优化
参考 [MySQL 5.7 Reference Manual::ORDER BY Optimization](https://dev.mysql.com/doc/refman/5.7/en/order-by-optimization.html)，假如有(key_part1,key_part2)的索引，在以下两种情况下MySQL会使用索引来避免排序操作：
- `SELECT * FROM t1   ORDER BY key_part1, key_part2;`
如果表项不只是key_part1,key_part2的话，那么MySQL优化器可能（不一定）不会通过索引来排序，因为扫描整个索引并且按照索引去查找出所有的行，花费的时间可能比直接查找出所有行再排序还慢。特别的，如果在InnoDB中，因为其特殊的索引存储结构（见聚簇索引），索引行和主键会存放在索引的B+树中，因此可以通过索引直接返回，极大提升排序速度：
`SELECT pk, key_part1, key_part2 FROM t1   ORDER BY key_part1, key_part2;`

- `SELECT * FROM t1   WHERE key_part1 = constant   ORDER BY key_part2;`
当key_part1是常量时，自然可以先通过key_part1索引找出对应的行，再利用key_part2进行排序（常量用前缀索引去检索，另一部分用来排序）。