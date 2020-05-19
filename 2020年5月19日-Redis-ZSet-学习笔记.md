## Redis ZSet 学习

#### 1. **ZADD key score member [[score member] [score member] ...]**

- 将一个或多个 `member` 元素及其 `score` 值加入到有序集 `key` 当中。
- 如果某个 `member` 已经是有序集的成员，那么更新这个 `member` 的 `score` 值，并通过重新插入这个 `member` 元素，来保证该 `member` 在正确的位置上。
- `score` 值可以是整数值或双精度浮点数。
- 如果 `key` 不存在，则创建一个空的有序集并执行 [ZADD](http://doc.redisfans.com/sorted_set/zadd.html#zadd) 操作。
- 当 `key` 存在但不是有序集类型时，返回一个错误。
- 对有序集的更多介绍请参见 [sorted set](http://redis.io/topics/data-types#sorted-sets) 。

> **ZADD 根据 分数进行的排序**
>
> ```
> 127.0.0.1:6379> zadd a 1 haha
> (integer) 1
> 127.0.0.1:6379> zadd a 0 qqqq
> (integer) 1
> 127.0.0.1:6379> zadd a 9 wwww
> (integer) 1
> 127.0.0.1:6379> zadd a 5 eee
> (integer) 1
> 127.0.0.1:6379> zrange a 0 -1
> 1) "qqqq"
> 2) "haha"
> 3) "eee"
> 4) "wwww"
> ```
>
> **如果之前存在，则修改分数，排序**
>
> ```
> 127.0.0.1:6379> zadd a 10 aaaaa
> (integer) 1
> 127.0.0.1:6379> zadd a 11 bbbb
> (integer) 1
> 127.0.0.1:6379> ZRANGE a 0 -1
> 1) "aaaaa"
> 2) "bbbb"
> 127.0.0.1:6379> zadd a 12 aaaaa
> (integer) 0
> 127.0.0.1:6379> ZRANGE a 0 -1
> 1) "bbbb"
> 2) "aaaaa"
> ```



#### 2. **ZCARD key**

- 返回有序集 `key` 的基数。

> ```
> 127.0.0.1:6379> zadd a 1 haha
> (integer) 1
> 127.0.0.1:6379> zadd a 0 qqqq
> (integer) 1
> 127.0.0.1:6379> ZCARD a
> (integer) 2
> ```



#### 3. **ZCOUNT key min max**

- 返回有序集 `key` 中， `score` 值在 `min` 和 `max` 之间(默认包括 `score` 值等于 `min` 或 `max` )的成员的数量。
- 关于参数 `min` 和 `max` 的详细使用方法，请参考 [*ZRANGEBYSCORE*](http://doc.redisfans.com/sorted_set/zrangebyscore.html#zrangebyscore) 命令。

> ```
> 
> ```



#### 4. **ZINCRBY key increment member**

- 为有序集 `key` 的成员 `member` 的 `score` 值加上增量 `increment` 。
- 可以通过传递一个负数值 `increment` ，让 `score` 减去相应的值，比如 `ZINCRBY key -5 member` ，就是让 `member` 的 `score` 值减去 `5` 。
- 当 `key` 不存在，或 `member` 不是 `key` 的成员时， `ZINCRBY key increment member` 等同于 `ZADD key increment member` 。
- 当 `key` 不是有序集类型时，返回一个错误。
- `score` 值可以是整数值或双精度浮点数。

> **如果之前没有，相当于 ZADD**
>
> ```
> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> ZINCRBY a 5 aaa
> "5"
> ```
>
> **之前存在，相当于修改score分数，影响排序**
>
> ```
> 127.0.0.1:6379> zrange a 0 -1
> 1) "aaa"
> 127.0.0.1:6379> ZINCRBY a 5 aaa
> "10"
> ```



#### 5. **ZRANGE key start stop [WITHSCORES]**

- 返回有序集 `key` 中，指定区间内的成员。
- 其中成员的位置按 `score` 值递增(从小到大)来排序。
- 具有相同 `score` 值的成员按字典序([lexicographical order](http://en.wikipedia.org/wiki/Lexicographical_order) )来排列。
- 如果你需要成员按 `score` 值递减(从大到小)来排列，请使用 [*ZREVRANGE*](http://doc.redisfans.com/sorted_set/zrevrange.html#zrevrange) 命令。
- 下标参数 `start` 和 `stop` 都以 `0` 为底，也就是说，以 `0` 表示有序集第一个成员，以 `1` 表示有序集第二个成员，以此类推。
- 你也可以使用负数下标，以 `-1` 表示最后一个成员， `-2` 表示倒数第二个成员，以此类推。
- 超出范围的下标并不会引起错误。
- 比如说，当 `start` 的值比有序集的最大下标还要大，或是 `start > stop` 时， [ZRANGE](http://doc.redisfans.com/sorted_set/zrange.html#zrange) 命令只是简单地返回一个空列表。
- 另一方面，假如 `stop` 参数的值比有序集的最大下标还要大，那么 Redis 将 `stop` 当作最大下标来处理。
- 可以通过使用 `WITHSCORES` 选项，来让成员和它的 `score` 值一并返回，返回列表以 `value1,score1, ..., valueN,scoreN` 的格式表示。
- 客户端库可能会返回一些更复杂的数据类型，比如数组、元组等。

> **和 ZREVRANGE 相反**
>
> ```
> 127.0.0.1:6379> zadd a 1 1 2 2 3 3 4 4
> (integer) 4
> 127.0.0.1:6379> zrange a 0 -1
> 1) "1"
> 2) "2"
> 3) "3"
> 4) "4"
> ```



#### 6. **ZREVRANGE key start stop [WITHSCORES]**

- 返回有序集 `key` 中，指定区间内的成员。
- 其中成员的位置按 `score` 值递减(从大到小)来排列。
- 具有相同 `score` 值的成员按字典序的逆序([reverse lexicographical order](http://en.wikipedia.org/wiki/Lexicographical_order#Reverse_lexicographic_order))排列。
- 除了成员按 `score` 值递减的次序排列这一点外， [ZREVRANGE](http://doc.redisfans.com/sorted_set/zrevrange.html#zrevrange) 命令的其他方面和 [*ZRANGE*](http://doc.redisfans.com/sorted_set/zrange.html#zrange) 命令一样。

> **和 ZRANGE 相反**
>
> ```
> 127.0.0.1:6379> zadd a 1 1 2 2 3 3 4 4
> (integer) 4
> 127.0.0.1:6379> ZREVRANGE a 0 -1
> 1) "4"
> 2) "3"
> 3) "2"
> 4) "1"
> ```



#### 7. **ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]**

- 返回有序集 `key` 中，所有 `score` 值介于 `min` 和 `max` 之间(包括等于 `min` 或 `max` )的成员。有序集成员按 `score` 值递增(从小到大)次序排列。
- 具有相同 `score` 值的成员按字典序([lexicographical order](http://en.wikipedia.org/wiki/Lexicographical_order))来排列(该属性是有序集提供的，不需要额外的计算)。
- 可选的 `LIMIT` 参数指定返回结果的数量及区间(就像SQL中的 `SELECT LIMIT offset, count` )，注意当 `offset` 很大时，定位 `offset` 的操作可能需要遍历整个有序集，此过程最坏复杂度为 O(N) 时间。
- 可选的 `WITHSCORES` 参数决定结果集是单单返回有序集的成员，还是将有序集成员及其 `score` 值一起返回。
- 该选项自 Redis 2.0 版本起可用。

> **根据分数返回结果，和ZRANGE下标不一样**
>
> ```
> 127.0.0.1:6379> zadd a 1 1 2 2 3 3 4 4
> (integer) 4
> 127.0.0.1:6379> ZRANGEBYSCORE a 1 3
> 1) "1"
> 2) "2"
> 3) "3"
> ```



#### 8. **ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]**

- 返回有序集 `key` 中， `score` 值介于 `max` 和 `min` 之间(默认包括等于 `max` 或 `min` )的所有的成员。有序集成员按 `score` 值递减(从大到小)的次序排列。
- 具有相同 `score` 值的成员按字典序的逆序([reverse lexicographical order](http://en.wikipedia.org/wiki/Lexicographical_order) )排列。
- 除了成员按 `score` 值递减的次序排列这一点外， [ZREVRANGEBYSCORE](http://doc.redisfans.com/sorted_set/zrevrangebyscore.html#zrevrangebyscore) 命令的其他方面和 [*ZRANGEBYSCORE*](http://doc.redisfans.com/sorted_set/zrangebyscore.html#zrangebyscore) 命令一样。

> **根据分数返回结果，从大到小**
>
> ```
> 127.0.0.1:6379> zadd a 1 1 2 2 3 3 4 4
> (integer) 4
> 127.0.0.1:6379> ZREVRANGEBYSCORE a 3 1
> 1) "3"
> 2) "2"
> 3) "1"
> ```



#### 9. **ZRANK key member**

- 返回有序集 `key` 中成员 `member` 的排名。其中有序集成员按 `score` 值递增(从小到大)顺序排列。
- 排名以 `0` 为底，也就是说， `score` 值最小的成员排名为 `0` 。
- 使用 [*ZREVRANK*](http://doc.redisfans.com/sorted_set/zrevrank.html#zrevrank) 命令可以获得成员按 `score` 值递减(从大到小)排列的排名。

> **返回 value在zset中的排名**
>
> ```
> 127.0.0.1:6379> zadd a 1 1 3 3 4 4 5 5 6 6
> (integer) 5
> 127.0.0.1:6379> ZRANK a 3
> (integer) 1
> 127.0.0.1:6379> zrank a 1
> (integer) 0
> 127.0.0.1:6379> zrank a 6
> (integer) 4
> ```



#### 10. **ZREVRANK key member**

- 返回有序集 `key` 中成员 `member` 的排名。其中有序集成员按 `score` 值递减(从大到小)排序。
- 排名以 `0` 为底，也就是说， `score` 值最大的成员排名为 `0` 。
- 使用 [*ZRANK*](http://doc.redisfans.com/sorted_set/zrank.html#zrank) 命令可以获得成员按 `score` 值递增(从小到大)排列的排名。

> **和ZRANK 顺序相反**
>
> ```
> 127.0.0.1:6379> zadd a 1 1 3 3 4 4 5 5 6 6
> (integer) 5
> 127.0.0.1:6379> ZREVRANK a 5
> (integer) 1
> 127.0.0.1:6379> zrevrank a 6
> (integer) 0
> ```



#### 11. **ZREM key member [member ...]**

- 移除有序集 `key` 中的一个或多个成员，不存在的成员将被忽略。
- 当 `key` 存在但不是有序集类型时，返回一个错误。

> **和其他类型REM一样，都是移除**
>
> ```
> 127.0.0.1:6379> zadd a 1 1 3 3 4 4 5 5 6 6
> (integer) 5
> 127.0.0.1:6379> zrem a 1 3
> (integer) 2
> 127.0.0.1:6379> zrange a 0 -1
> 1) "4"
> 2) "5"
> 3) "6"
> ```



#### 12. **ZREMRANGEBYRANK key start stop**

- 移除有序集 `key` 中，指定排名(rank)区间内的所有成员。
- 区间分别以下标参数 `start` 和 `stop` 指出，包含 `start` 和 `stop` 在内。
- 下标参数 `start` 和 `stop` 都以 `0` 为底，也就是说，以 `0` 表示有序集第一个成员，以 `1` 表示有序集第二个成员，以此类推。
- 你也可以使用负数下标，以 `-1` 表示最后一个成员， `-2` 表示倒数第二个成员，以此类推。

> **根据下标范围移除**
>
> ```
> 127.0.0.1:6379> zadd a 1 1 2 2 3 3 4 4 5 5
> (integer) 3
> 127.0.0.1:6379> ZRANGE a 0 -1
> 1) "1"
> 2) "2"
> 3) "3"
> 4) "4"
> 5) "5"
> 6) "6"
> 127.0.0.1:6379> ZREMRANGEBYRANK a 2 4
> (integer) 3
> 127.0.0.1:6379> zrange a 0 -1
> 1) "1"
> 2) "2"
> 3) "6"
> ```



#### 13. **ZREMRANGEBYSCORE key min max**

- 移除有序集 `key` 中，所有 `score` 值介于 `min` 和 `max` 之间(包括等于 `min` 或 `max` )的成员。
- 自版本2.1.6开始， `score` 值等于 `min` 或 `max` 的成员也可以不包括在内，详情请参见 [*ZRANGEBYSCORE*](http://doc.redisfans.com/sorted_set/zrangebyscore.html#zrangebyscore) 命令。

> **根据分数移除**
>
> ```
> 127.0.0.1:6379> zadd a 1 a 2 b 3 c 4 d 5 e
> (integer) 5
> 127.0.0.1:6379> zremrangebyscore a 2 4
> (integer) 3
> 127.0.0.1:6379> zrange a 0 -1
> 1) "a"
> 2) "e"
> ```



#### 14. **ZUNIONSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]** （待研究）

- 计算给定的一个或多个有序集的并集，其中给定 `key` 的数量必须以 `numkeys` 参数指定，并将该并集(结果集)储存到 `destination` 。
- 默认情况下，结果集中某个成员的 `score` 值是所有给定集下该成员 `score` 值之 *和* 。
- **WEIGHTS**
- 使用 `WEIGHTS` 选项，你可以为 *每个* 给定有序集 *分别* 指定一个乘法因子(multiplication factor)，每个给定有序集的所有成员的 `score` 值在传递给聚合函数(aggregation function)之前都要先乘以该有序集的因子。
- 如果没有指定 `WEIGHTS` 选项，乘法因子默认设置为 `1` 。
- **AGGREGATE**
- 使用 `AGGREGATE` 选项，你可以指定并集的结果集的聚合方式。
- 默认使用的参数 `SUM` ，可以将所有集合中某个成员的 `score` 值之 *和* 作为结果集中该成员的 `score` 值；使用参数 `MIN` ，可以将所有集合中某个成员的 *最小* `score` 值作为结果集中该成员的 `score` 值；而参数 `MAX` 则是将所有集合中某个成员的 *最大* `score` 值作为结果集中该成员的 `score` 值。



#### 15. **ZINTERSTORE destination numkeys key [key ...] [WEIGHTS weight [weight ...]] [AGGREGATE SUM|MIN|MAX]** （待研究）

- 计算给定的一个或多个有序集的交集，其中给定 `key` 的数量必须以 `numkeys` 参数指定，并将该交集(结果集)储存到 `destination` 。
- 默认情况下，结果集中某个成员的 `score` 值是所有给定集下该成员 `score` 值之和.
- 关于 `WEIGHTS` 和 `AGGREGATE` 选项的描述，参见 [*ZUNIONSTORE*](http://doc.redisfans.com/sorted_set/zunionstore.html#zunionstore) 命令。



#### 16. **ZSCAN key cursor [MATCH pattern] [COUNT count]**  （待研究）

- 详细信息请参考 [*SCAN*](http://doc.redisfans.com/key/scan.html#scan) 命令。