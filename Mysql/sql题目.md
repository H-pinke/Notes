##### 1.快件信息表（waybillinfo）中存储了快件的所有操作信息，请找出在中山公园网点，异常派送（optype='异常派件'）次数超过3次的快件(waybillno),

```
select waybillno from waybillinfo where zonecode='中山公园' and optype='异常派件' group by waybillno having count(*) > 3
```

##### 2.比赛结果result表内容如下：
Date           Win
2017-07-12        胜
2017-07-12        负
2017-07-15        胜
2017-07-15        负
如果要生成下列结果, 正确的sql语句是：（   ）
比赛日期      胜   负
2017-07-12     1   1
2017-07-15     1   1

```
select Date As 比赛日期, SUM(case when Win='胜' then 1 else 0 end) 胜, SUM(case when Win='负' then 1 else 0 end) 负 from result group by Date
```

##### 3.有一张学生成绩表sc（sno 学号，class 课程，score 成绩），请查询出每个学生的英语、数学的成绩（行转列，一个学生只有一行记录）。

```
select sno,
case when class='english' then score else 0  end ,
case when class='math' then score else 0 end
from sc
where class in('english','math')
```

##### 4.有一个名为app的MySQL数据库表，其建表语句如下：

```
CREATE TABLE `app` (
`app_id` int(10) DEFAULT '0',//应用ID
`version_code` int(10) DEFAULT '0',//应用的版本号
`download_count` int(10) DEFAULT '0'//当前版本的下载量
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4
```


当前表中数据记录如下，一条记录表示某个应用的某个版本的下载量记录：
+--------+--------------+----------------+
| app_id | version_code | download_count |
+--------+--------------+----------------+
|   1 |      10 |       90 |
|   1 |      11 |      100 |
|   1 |      10 |       20 |
|   2 |      15 |       10 |
|   2 |      16 |       15 |
|   2 |      17 |       30 |
|   2 |      16 |       5 |
|   3 |      2 |       50 |
+--------+--------------+----------------+

问： 下面那个MySQL语句可以查出每个应用中总下载量最大的版本号和次数（ ）？

```
select t.app_id, t.version_code, max(t.download_sum) from (select app_id, version_code, sum(download_count) download_sum from app
group by app_id, version_code order by download_sum desc) as t group by t.app_id;
```

## 百度面试题

写一个sql: student(sno, sname, classid, grade) 查出每个班成绩前三名的同学。

```
SELECT
	* 
FROM student t
where (SELECT COUNT(1)+1 FROM student where classid=t.classid and grade>t.grade) <=3

//解读
1. select count(1)+1 from student classid=1 and grade>100 //2
```

如果把后3名学生的成绩，`之需要`  `t.grade>grade` 就可以，题目还可以加姓别的判断。

==考虑一个情况并列呢==

```

```

