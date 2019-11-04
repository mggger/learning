# KTable

KStream 定义为无限的事件序列

KTable 可以定义为KStraem的更新流

比如KStream

|offset|name|value|
|:--|:--|:--|
|1|wang|11|
|2|wang|12|
|3|li|13|
|4|zhang|14|
|5|li|22|

对应的KTable就是:

|offset|name|value|
|:--|:--|:--|
|2|wang|12|
|4|zhang|14|
|5|li|22|

KTable 记录存储在本地状态， 它是通过一部分缓存和新数据对比， 将更新记录发送到下游.

```cache.max.bytes.buffering``` 为本地缓存， 默认10M
```commit.interval.ms``` 提交间隔， 默认值30s

## 聚合
KStream 通过groupbykey 或者 grouoby 生成KgroupedStream 会生成KGroupedStream， 聚合之后生成KTable


## 开窗
窗口类型:
- 会话窗口
- 翻转窗口
- 滑动 / 跳跃窗口

选择那种类型的窗口取决于业务需求。 翻转和跳跃窗口是有时间限制的， 而会话窗口偏重于用户活跃度， 会话的长度取决于用户的活跃度。 对于所有类型窗口的一个关键点事它们都是基于记录中的时间戳.


### 会话窗口

api: ```xxx.windowdBy(SessionWindows.with(20).until(900)) ```

上面设置了一个会话窗口 不活跃的时间为20s， 保留时间为15分钟

#### 示例
现在有以下事件流：

|key|value|time|
|:--|:--|:--|
|A|1|10|
|A|2|15|
|A|3|40|

以上有2个会话窗口，  时间10， 15为一个， 40为另外一个，

这时候有1条数据为 ```key: A, value: 2, time 30 ```, 这时候会合成一个会话窗口

***

### 翻转窗口
翻转窗口是最常用的场景， 统计每20秒内统计公司的一个交易量. ```TimeWindows.of(20) ```

### 滑动 / 跳跃窗口

滑动或者跳跃窗口和翻转窗口有点类似，  比如每20秒内统计公司的一个交易量， 但是每5s更新一次。 ```TimeWindows.of(20).advanceBy(5) ```

## GlobalKTable
在某种情况下，你希望链接的查找数据尽可能相对比较下， 并且查找数据的整个副本可以在每个节点上本地匹配，这个时候就可以使用GlobalKtable



