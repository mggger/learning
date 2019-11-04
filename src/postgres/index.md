# 索引

postgre索引类型有

- B-tree
- Hash
- GIN
- GIST



因为B-tree和Hash索引比较常见， 所以不多做介绍.


## GIN

广义倒置索引(GIN) 对于不同类型的组织非常有用. 

GIN 用一个列表来存储key， 这个key是一个列表， 包含涉及到的行。 一行可以出现在多个key中。 

GIN对于索引数组值很有用，它允许搜索数组列包含特定值的所有行，或者与要匹配的数组有一些共同的元素。这也是实现全文搜索的方法之一


## Gist


广义搜索树（GiST）提供了一种构建平衡树结构的方法，用于存储数据，例如内置B树，只需定义如何处理key。

PostgreSQL中包含的几何数据类型包括运算符，以允许索引按项之间的距离和它们是否相交来排序。


### 索引排序

B树索引按升序存储其条目。从PostgreSQL 8.3开始，空值也存储在其中，默认为在表中持续。您可以撤消这两个默认值，例如:

```sql
CREATE index i on t (v desc nulls first);
```

### 部分索引
可通过where来限制索引的创建, 使得不会对表所有的数据创建索引

```sql
create index accounts_interesting_index on accounts where interesting is true;
```


### 索引基于表达式

postgre还支持表达式来创建索引
```
create index i_lower_index on t (lower(name));
```



