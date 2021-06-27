#### Mysql show profiles 分析sql性能

mysql 5.0.37 引入的这个功能

要开启的这个功能的命令

`set profling=1`

```
mysql> SHOW PROFILES;
+----------+------------+---------------------------------------------------------------------------------+
| Query_ID | Duration   | Query                                                                           |
+----------+------------+---------------------------------------------------------------------------------+
|        1 | 0.24123800 | SELECT * FROM employees.employees WHERE first_name='Eric' AND last_name='Anido' |
|        2 | 0.22307100 | SELECT * FROM employees.employees WHERE first_name='Eric'                       |
+----------+------------+---------------------------------------------------------------------------------+
```

根据query_id 查看某个查询的详细时间耗费

```
mysql> show profile for query 4;
+--------------------------------+----------+
| Status                         | Duration |
+--------------------------------+----------+
| starting                       | 0.001336 |
| checking permissions           | 0.000017 |
| checking permissions           | 0.000650 |
| init                           | 0.000030 |
| Opening tables                 | 0.000130 |
| setup                          | 0.001475 |
| creating table                 | 0.015118 |
| After create                   | 0.001915 |
| System lock                    | 0.000024 |
| preparing for alter table      | 0.014141 |
| altering table                 | 2.065239 |
| committing alter table to stor | 0.024199 |
| end                            | 0.000028 |
| query end                      | 0.000017 |
| closing tables                 | 0.000011 |
| freeing items                  | 0.000019 |
| cleaning up                    | 0.000024 |
+--------------------------------+----------+
17 rows in set, 1 warning (0.00 sec)
```

那么可不可以查看占用cpu、 io等信息呢

```
mysql> show profile block io,cpu for query 4;
+--------------------------------+----------+----------+------------+--------------+---------------+
| Status                         | Duration | CPU_user | CPU_system | Block_ops_in | Block_ops_out |
+--------------------------------+----------+----------+------------+--------------+---------------+
| starting                       | 0.001336 | 0.000139 |   0.000169 |            0 |             0 |
| checking permissions           | 0.000017 | 0.000008 |   0.000010 |            0 |             0 |
| checking permissions           | 0.000650 | 0.000020 |   0.000114 |            0 |             0 |
| init                           | 0.000030 | 0.000014 |   0.000015 |            0 |             0 |
| Opening tables                 | 0.000130 | 0.000077 |   0.000054 |            0 |             0 |
| setup                          | 0.001475 | 0.000117 |   0.000303 |            0 |             0 |
| creating table                 | 0.015118 | 0.000199 |   0.001955 |            0 |             0 |
| After create                   | 0.001915 | 0.000224 |   0.000602 |            0 |             0 |
| System lock                    | 0.000024 | 0.000018 |   0.000007 |            0 |             0 |
| preparing for alter table      | 0.014141 | 0.003117 |   0.006975 |            0 |             0 |
| altering table                 | 2.065239 | 2.064266 |   0.052163 |            0 |             0 |
| committing alter table to stor | 0.024199 | 0.011882 |   0.002949 |            0 |             0 |
| end                            | 0.000028 | 0.000012 |   0.000016 |            0 |             0 |
| query end                      | 0.000017 | 0.000017 |   0.000001 |            0 |             0 |
| closing tables                 | 0.000011 | 0.000009 |   0.000001 |            0 |             0 |
| freeing items                  | 0.000019 | 0.000008 |   0.000010 |            0 |             0 |
| cleaning up                    | 0.000024 | 0.000014 |   0.000010 |            0 |             0 |
+--------------------------------+----------+----------+------------+--------------+---------------+
17 rows in set, 1 warning (0.00 sec)
```

具体信息可以参考http://dev.mysql.com/doc/refman/5.0/en/show-profiles.html

