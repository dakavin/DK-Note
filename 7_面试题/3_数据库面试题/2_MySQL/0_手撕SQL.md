
# 1、关于子查询的使用

## 1.1 题1

[185. 部门工资前三高的所有员工 - 力扣（LeetCode）](https://leetcode.cn/problems/department-top-three-salaries/description/)

1. 涉及两个表，所以需要外连接将两个表连接在一起
2. 因为需要前三，所以使用where中包括一个子查询
	- 查询员工表中的所有工资数量，使用COUNT
	- 与外查询同部门，且工资高于外查询的工资
	- 这样获得的数量少于3个，即为工资为部门前三的员工

## 1.2 题2

student表结构：id，name，class_id（班级id），score（成绩）

求各班前十名的同学

```sql
SELECT s1.id , s1.name , s1.score
FROM student s1
WHERE (
	SELECT score
	FROM student
	WHERE s1.class_id = class_id
	AND s1.score < score
) < 10
ORDER BY class_id , SCORE DESC;
```


# 2、关于多表连接使用

具体查看笔记：
- [3. SQL99语法实现多表查询](../../../../2_笔记/3_MySql、JDBC和Web/0_尚硅谷/0_MySQL基础内容/第06章_多表查询.md#3.%20SQL99语法实现多表查询)
- [5. 7种SQL JOINS的实现](../../../../2_笔记/3_MySql、JDBC和Web/0_尚硅谷/0_MySQL基础内容/第06章_多表查询.md#5.%207种SQL%20JOINS的实现)

## 2.1 题1

对于AB两张表，且都存在一样的字段tid

查询B表中为null，却在A表中的记录

```SQL
SELECT tid
FROM A LEFT JOIN B 
ON A.tid = B.tid
WHERE B.tid IS NUll;
```

解释：（假如是员工表和部门表）
1. 关联员工表和部门表，使用JOIN ON即可
2. 同时展示出员工表中没有部门的员工，使用LEFT JOIN ON
3. 只展示出员工表中没有部门的员工，使用WHERE