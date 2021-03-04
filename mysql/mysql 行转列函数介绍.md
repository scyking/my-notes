# mysql 行转列函数介绍

标签（空格分隔）： mysql

---

[toc]

## [官网介绍](https://dev.mysql.com/doc/refman/5.7/en/aggregate-functions.html)

## 函数`GROUP_CONCAT(expr)`语法

```
GROUP_CONCAT([DISTINCT] expr [,expr ...]
             [ORDER BY {unsigned_integer | col_name | expr}
                 [ASC | DESC] [,col_name ...]]
             [SEPARATOR str_val])
```

## 示例

```
mysql> SELECT student_name,
         GROUP_CONCAT(test_score)
       FROM student
       GROUP BY student_name;
```

or:

```
mysql> SELECT student_name,
         GROUP_CONCAT(DISTINCT test_score
                      ORDER BY test_score DESC SEPARATOR ' ')
       FROM student
       GROUP BY student_name;
```






