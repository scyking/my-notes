# mysql 全文索引

标签（空格分隔）： mysql

---

[toc]

## 概述
> 相比`like + %`，其用于检索大量的文本数据，以提升检索速度，但可能存在精度问题。

### 版本支持

- MySQL 5.6 以前的版本，只有 MyISAM 存储引擎支持全文索引；
- MySQL 5.6 及以后的版本，MyISAM 和 InnoDB 存储引擎均支持全文索引;
- 只有字段的数据类型为 `char`、`varchar`、`text` 及其系列才可以建全文索引。

## 创建

### 创建表时创建全文索引

```
create table fulltext_test (
    id int(11) NOT NULL AUTO_INCREMENT,
    content text NOT NULL,
    tag varchar(255),
    PRIMARY KEY (id),
    // 创建联合全文索引列
    FULLTEXT KEY content_tag_fulltext(content,tag)  
) ENGINE=MyISAM DEFAULT CHARSET=utf8;
```

### 在已存在的表上创建全文索引

```
create fulltext index content_tag_fulltext
    on fulltext_test(content,tag);
```

### 通过 SQL 语句 ALTER TABLE 创建全文索引

```
alter table fulltext_test
    add fulltext index content_tag_fulltext(content,tag);
```

## 删除

### 直接使用 DROP INDEX 删除全文索引

```
drop index content_tag_fulltext
    on fulltext_test;
```

### 通过 SQL 语句 ALTER TABLE 删除全文索引

```
alter table fulltext_test
    drop index content_tag_fulltext;
```

## 使用
> 与模糊匹配 `like + %` 不同，全文索引使用 `match` 和 `against` 关键字进行检索。

```
select * from fulltext_test 
    where match(content,tag) against('xxx xxx');
```

PS：`match()` 函数中指定的列必须和全文索引中指定的列完全相同，否则就会报错，无法使用全文索引，这是因为全文索引不会记录关键字来自哪一列。

## 注意事项

- 使用全文索引前，搞清楚版本支持情况；
- 全文索引比 like + % 快 N 倍，但是可能存在精度问题；
- 如果需要全文索引的是大量数据，建议先添加数据，再创建索引；
- 对于中文，可以使用 MySQL 5.7.6 之后的版本，或者第三方插件。

