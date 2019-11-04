# 分区

postgres的分区类型有:

- Table range partitioning
- Declarative partitioning
- Horizontal partitioning with PL/Proxy


示例表:

|orderid| orderdate | customerid| netamount| tax| totalamount|
|:--|:--|:--|:--|:--|:--|
|1|2004-01-01|7888|313.24|25.84|339.08|
|2|2004-02-01|1234|222.33|222.3|111.33|
|...|...|...|...|...|...|




## Table range partitioning



创建分区

```sql

CREATE TABLE orders_2004_01 (
      CHECK ( orderdate >= DATE '2004-01-01' and orderdate < DATE
   '2004-02-01')
   ) INHERITS (orders);

...
```



**创建触发器重定向insert请求**
```sql
CREATE OR REPLACE FUNCTION orders_insert_trigger()
   RETURNS TRIGGER AS $$
   BEGIN
       IF    ( NEW.orderdate >= DATE '2004-12-01' AND
             NEW.orderdate < DATE '2005-01-01' ) THEN
            INSERT INTO orders_2004_12 VALUES (NEW.*);
       ELSIF ( NEW.orderdate >= DATE '2004-11-01' AND
             NEW.orderdate < DATE '2004-12-01' ) THEN
            INSERT INTO orders_2004_11 VALUES (NEW.*);
       ...
       ELSE
             RAISE EXCEPTION 'Error in orders_insert_trigger():  date out of range';
       END IF;
       RETURN NULL;
   END;
```


```sql
CREATE TRIGGER insert_orders_trigger
       BEFORE INSERT ON orders
       FOR EACH ROW EXECUTE PROCEDURE orders_insert_trigger();
```


**还可以通过分区rule实现**


## Declarative partitioning
postgres 有内置的partition函数

**partition by list**

```sql
  create table orders_state
      (orderid integer not null,
      orderdate date not null,
      customerid integer not null,
      tax numeric(12,2) not null ,
      state char(2))
   partition by list (state);
```

接下来创建子分区
```sql
CREATE TABLE ORDERS_US PARTITION OF ORDERS_state FOR VALUES IN ('US');


CREATE TABLE ORDERS_state_EU PARTITION OF ORDERS_state FOR VALUES IN
   ('IT','FR','ES','GR','PT');
```

***

** partition by range **

```sql
create table orders_state
           (orderid integer not null,
           orderdate date not null,
           customerid integer not null,
           tax numeric(12,2) not null ,
           state char(2))
partition by  RANGE (orderdate);
```

接下来创建子分区

```sql
CREATE TABLE orders_state_q1 PARTITION OF orders_state FOR VALUES FROM
   ('2018-01-01') TO ('2018-03-01');
   
CREATE TABLE orders_state_q2 PARTITION OF orders_state FOR VALUES FROM
   ('2018-03-01') TO ('2018-06-01');

CREATE TABLE orders_state_q3 PARTITION OF orders_state FOR VALUES FROM
   ('2018-06-01') TO ('2018-09-01');

CREATE TABLE orders_state_q4 PARTITION OF orders_state FOR VALUES FROM
   ('2018-09-01') TO ('2019-01-01');
```

### 删除旧分区
```sql
alter table orders_state detach partition orders_state_q4
```

### 分区常见问题

- 没有开启 ```constraint_exclusion```
- 忘记将分区添加索引



## Horizontal partitioning with PL/Proxy
分片强调在每个节点上放置共享数据结构，以便它们中的每一个可以彼此独立地操作。 通常，您在每个系统上需要的数据通常被称为应用程序的维度，查找或事实表，这些数据对于每组数据都是通用的。我们的想法是，应该采用无共享架构，其中每个分片分区都是自给自足的，用于回答与其包含的数据相关的查询。






