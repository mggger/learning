# 关系代数

关系代数可以下面几组

- 一元关系操作
- 集合论中的关系代数运算
- 二元关系运算

## 一元关系操作

- select
- project
- rename 

## 集合论中的关系代数运算

- union
- intersection
- difference
- cartesian product

## 二元关系运算
- join
- division

### select
SELECT操作用于根据给定的选择条件选择元组的子集。（σ)符号表示它。它用作表达式来选择符合选择条件的元组。选择操作选择满足给定谓词的元组。

$$ \sigma_{p}(r) $$

σ 是谓语

r 表的名称

p 介词逻辑

```sql
select tuples from tutorials where topic = "database"
```
对应的关系代数为:

$$ \sigma_{topic} = database ^{(tutorials)} $$

### project
投影消除了输入关系的所有属性，但是在投影列表中提到了这些属性。投影方法定义包含Relation的垂直子集的关系。

这有助于提取指定属性的值以消除重复值。 （pi）用于从关系中选择属性的符号。此操作可帮助您保留关系中的特定列，并丢弃其他列。


```sql
select col1 from data1 
```

对应的关系代数为:

$$ \pi_{col1} (data1) $$

### Union
UNION用∪符号表示。它包括表A或表B中的所有元组

A UNION B 表示为:

```A U B ```

### Set Difference

```A - B``` 表示在A中的元组， 不在B中.

### Intersection
交叉由符号∩定义

``` A ∩ B ``` 表示即在A, 也在B中的元组

### Cartesian product
这种类型的操作有助于合并两个关系中的列。通常，笛卡尔积在单独执行时从不是有意义的操作。但是，当它跟随其他操作时，它变得有意义。

$$  \sigma_{column 2} = '1' ( A X B) $$  

### Join Operations
操作本质也是一个笛卡尔积， 后面是选择标准。 它用⋈表示。

**Inner join: 在内连接中，仅包括满足匹配条件的那些元组，而其余元组则被排除。 它具有以下类型**

- Theta join: JOIN操作的一般情况称为Theta join。它用符号θ表示 ``` A ⋈θ B ```, 
Theta join可以使用选择标准中的任何条件。 

- EQUI join:  当theta连接仅使用等价条件时，它将成为equi连接。

- Natural join:  只有在关系之间存在公共属性（列）时，才能执行自然连接

**OUTER JOIN: 在外连接中，以及满足匹配条件的元组，我们还包括一些或所有与标准不匹配的元组。**

- Left Outer Join: 在左外连接中，操作允许将所有元组保持在左关系中。但是，如果在右关系中找不到匹配的元组，则连接结果中的右关系属性将填充空值.

- Right Outer Join: 在右外连接中，操作允许将所有元组保持在正确的关系中。但是，如果在左关系中找不到匹配的元组，则连接结果中左关系的属性将填充空值。

- Full Outer Join: 在完全外连接中，无论匹配条件如何，来自两个关系的所有元组都包含在结果中。









