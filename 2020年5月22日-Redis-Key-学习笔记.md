## Redis Key 学习笔记

#### 1. **DEL key [key ...]**

- 删除给定的一个或多个 `key` 。
- 不存在的 `key` 会被忽略。

> ```
> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> del a
> (integer) 1
> 127.0.0.1:6379> get a
> (nil)
> ```



#### 2. **TYPE key** （返回key类型）

- 返回 `key` 所储存的值的类型。

> ```
> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> type a
> string
> ```



#### 3. **TTL key**（返回秒生存时间）

- 以秒为单位，返回给定 `key` 的剩余生存时间(TTL, time to live)。

> **-1是不会过期，-2不存在，正整数剩余存活时间**
>
> ```
> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> ttl a
> (integer) -1
> 127.0.0.1:6379> set b b ex 10
> OK
> 127.0.0.1:6379> ttl b
> (integer) 9
> 127.0.0.1:6379> ttl c
> (integer) -2
> ```



#### 4. **PTTL key **（返回毫秒生存时间）

- 这个命令类似于 [*TTL*](http://doc.redisfans.com/key/ttl.html#ttl) 命令，但它以毫秒为单位返回 `key` 的剩余生存时间，而不是像 [*TTL*](http://doc.redisfans.com/key/ttl.html#ttl) 命令那样，以秒为单位。



#### 5. **EXISTS key**

- 检查给定 `key` 是否存在。

> **给定key中有一个存在就返回1**
>
> ```
> 127.0.0.1:6379> set a a
> OK
> 127.0.0.1:6379> EXISTS a
> (integer) 1
> 127.0.0.1:6379> EXISTS b
> (integer) 0
> 127.0.0.1:6379> EXISTS a b
> (integer) 1
> ```



#### 6. **EXPIRE key seconds** （指定key存活多少秒）

- 为给定 `key` 设置生存时间，当 `key` 过期时(生存时间为 `0` )，它会被自动删除。
- 在 Redis 中，带有生存时间的 `key` 被称为『易失的』(volatile)。
- 生存时间可以通过使用 [*DEL*](http://doc.redisfans.com/key/del.html#del) 命令来删除整个 `key` 来移除，或者被 [*SET*](http://doc.redisfans.com/string/set.html#set) 和 [*GETSET*](http://doc.redisfans.com/string/getset.html#getset) 命令覆写(overwrite)，这意味着，如果一个命令只是修改(alter)一个带生存时间的 `key` 的值而不是用一个新的 `key` 值来代替(replace)它的话，那么生存时间不会被改变。
- 比如说，对一个 `key` 执行 [*INCR*](http://doc.redisfans.com/string/incr.html#incr) 命令，对一个列表进行 [*LPUSH*](http://doc.redisfans.com/list/lpush.html#lpush) 命令，或者对一个哈希表执行 [*HSET*](http://doc.redisfans.com/hash/hset.html#hset) 命令，这类操作都不会修改 `key` 本身的生存时间。
- 另一方面，如果使用 [*RENAME*](http://doc.redisfans.com/key/rename.html) 对一个 `key` 进行改名，那么改名后的 `key` 的生存时间和改名前一样。
- [*RENAME*](http://doc.redisfans.com/key/rename.html) 命令的另一种可能是，尝试将一个带生存时间的 `key` 改名成另一个带生存时间的 `another_key` ，这时旧的 `another_key` (以及它的生存时间)会被删除，然后旧的 `key` 会改名为 `another_key` ，因此，新的 `another_key` 的生存时间也和原本的 `key` 一样。
- 使用 [*PERSIST*](http://doc.redisfans.com/key/persist.html) 命令可以在不删除 `key` 的情况下，移除 `key` 的生存时间，让 `key` 重新成为一个『持久的』(persistent) `key` 。

> **设置时间为0时，相当于del**
>
> ```
> 127.0.0.1:6379> set a a
> OK
> 127.0.0.1:6379> EXPIRE a 30
> (integer) 1
> 127.0.0.1:6379> ttl a
> (integer) 28
> 127.0.0.1:6379> EXPIRE a 0
> (integer) 1
> 127.0.0.1:6379> get a
> (nil)
> ```



#### 7. **PEXPIRE key milliseconds** （指定key存活多少毫秒）

- 这个命令和 [*EXPIRE*](http://doc.redisfans.com/key/expire.html) 命令的作用类似，但是它以毫秒为单位设置 `key` 的生存时间，而不像 [*EXPIRE*](http://doc.redisfans.com/key/expire.html) 命令那样，以秒为单位。

#### 8. **EXPIREAT key timestamp** （指定 某个时间戳 过期，相当于设置过期日期）

- [EXPIREAT](http://doc.redisfans.com/key/expireat.html#expireat) 的作用和 [*EXPIRE*](http://doc.redisfans.com/key/expire.html) 类似，都用于为 `key` 设置生存时间。
- 不同在于 [EXPIREAT](http://doc.redisfans.com/key/expireat.html#expireat) 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。



#### 9. **PEXPIREAT key milliseconds-timestamp**（指定 某个毫秒时间戳 过期，相当于设置过期日期）

这个命令和 [*EXPIREAT*](http://doc.redisfans.com/key/expireat.html) 命令类似，但它以毫秒为单位设置 `key` 的过期 unix 时间戳，而不是像 [*EXPIREAT*](http://doc.redisfans.com/key/expireat.html) 那样，以秒为单位。



#### 10. **PERSIST key**

- 移除给定 `key` 的生存时间，将这个 `key` 从『易失的』(带生存时间 `key` )转换成『持久的』(一个不带生存时间、永不过期的 `key` )。

> ```
> 127.0.0.1:6379> set a a ex 20
> OK
> 127.0.0.1:6379> ttl a
> (integer) 17
> 127.0.0.1:6379> PERSIST a
> (integer) 1
> 127.0.0.1:6379> ttl a
> (integer) -1
> ```



#### 11. **KEYS pattern**

- 查找所有符合给定模式 `pattern` 的 `key` 。
- `KEYS *` 匹配数据库中所有 `key` 。
- `KEYS h?llo` 匹配 `hello` ， `hallo` 和 `hxllo` 等。
- `KEYS h*llo` 匹配 `hllo` 和 `heeeeello` 等。
- `KEYS h[ae]llo` 匹配 `hello` 和 `hallo` ，但不匹配 `hillo` 。
- 特殊符号用 `\` 隔开

> ```
> 127.0.0.1:6379> set a 1
> OK
> 127.0.0.1:6379> set b 2
> OK
> 127.0.0.1:6379> keys *
> 1) "b"
> 2) "a"
> 127.0.0.1:6379> keys a
> 1) "a"
> 127.0.0.1:6379> keys c
> (empty list or set)
> ```



#### 12. **RANDOMKEY** （随机返回key，不删除）

- 从当前数据库中随机返回(不删除)一个 `key` 。

> ```
> 127.0.0.1:6379> keys *
> 1) "b"
> 2) "a"
> 127.0.0.1:6379> RANDOMKEY
> "a"
> 127.0.0.1:6379> RANDOMKEY
> "b"
> 127.0.0.1:6379> RANDOMKEY
> "a"
> 127.0.0.1:6379> RANDOMKEY
> "b"
> ```



#### 13. **RENAME key newkey **（如果存在 newkey则会覆盖）

- 将 `key` 改名为 `newkey` 。
- 当 `key` 和 `newkey` 相同，或者 `key` 不存在时，返回一个错误。
- 当 `newkey` 已经存在时， [RENAME](http://doc.redisfans.com/key/rename.html#rename) 命令将覆盖旧值。

> ```
> 127.0.0.1:6379> set a a
> OK
> 127.0.0.1:6379> set b b
> OK
> 127.0.0.1:6379> RENAME a b
> OK
> 127.0.0.1:6379> get a
> (nil)
> 127.0.0.1:6379> get b
> "a"
> ```



#### 14. **RENAMENX key newkey** （当newkey不存在才会修改）

- 当且仅当 `newkey` 不存在时，将 `key` 改名为 `newkey` 。
- 当 `key` 不存在时，返回一个错误。

> ```
> 127.0.0.1:6379> set a a
> OK
> 127.0.0.1:6379> set b b
> OK
> 127.0.0.1:6379> renamenx a b
> (integer) 0
> 127.0.0.1:6379> get a
> "a"
> ```



#### 15. **MOVE key db**

- 将当前数据库的 `key` 移动到给定的数据库 `db` 当中。
- 如果当前数据库(源数据库)和给定数据库(目标数据库)有相同名字的给定 `key` ，或者 `key` 不存在于当前数据库，那么 `MOVE` 没有任何效果。
- 因此，也可以利用这一特性，将 [MOVE](http://doc.redisfans.com/key/move.html#move) 当作锁(locking)原语(primitive)。

> ```
> 127.0.0.1:6379> set a a
> OK
> 127.0.0.1:6379> move a 1
> (integer) 1
> 127.0.0.1:6379> get a
> (nil)
> 127.0.0.1:6379> select 1
> OK
> 127.0.0.1:6379[1]> get a
> "a"
> ```



#### 16. **SORT key [BY pattern] [LIMIT offset count] [GET pattern [GET pattern ...]] [ASC | DESC] [ALPHA] [STORE destination] **（2020年5月22日  暂不考虑）

- 返回或保存给定列表、集合、有序集合 `key` 中经过排序的元素。
- 排序默认以数字作为对象，值被解释为双精度浮点数，然后进行比较。

#### 17. **SCAN cursor [MATCH pattern] [COUNT count]** （暂不考虑）

- [*SCAN*](http://doc.redisfans.com/key/scan.html#scan) 命令及其相关的 [*SSCAN*](http://doc.redisfans.com/set/sscan.html#sscan) 命令、 [*HSCAN*](http://doc.redisfans.com/hash/hscan.html#hscan) 命令和 [*ZSCAN*](http://doc.redisfans.com/sorted_set/zscan.html#zscan) 命令都用于增量地迭代（incrementally iterate）一集元素（a collection of elements）：
- - [*SCAN*](http://doc.redisfans.com/key/scan.html#scan) 命令用于迭代当前数据库中的数据库键。
  - [*SSCAN*](http://doc.redisfans.com/set/sscan.html#sscan) 命令用于迭代集合键中的元素。
  - [*HSCAN*](http://doc.redisfans.com/hash/hscan.html#hscan) 命令用于迭代哈希键中的键值对。
  - [*ZSCAN*](http://doc.redisfans.com/sorted_set/zscan.html#zscan) 命令用于迭代有序集合中的元素（包括元素成员和元素分值）。
- 以上列出的四个命令都支持增量式迭代， 它们每次执行都只会返回少量元素， 所以这些命令可以用于生产环境， 而不会出现像 [*KEYS*](http://doc.redisfans.com/key/keys.html#keys) 命令、 [*SMEMBERS*](http://doc.redisfans.com/set/smembers.html#smembers) 命令带来的问题 —— 当 [*KEYS*](http://doc.redisfans.com/key/keys.html#keys) 命令被用于处理一个大的数据库时， 又或者 [*SMEMBERS*](http://doc.redisfans.com/set/smembers.html#smembers) 命令被用于处理一个大的集合键时， 它们可能会阻塞服务器达数秒之久。
- 不过， 增量式迭代命令也不是没有缺点的： 举个例子， 使用 [*SMEMBERS*](http://doc.redisfans.com/set/smembers.html#smembers) 命令可以返回集合键当前包含的所有元素， 但是对于 [*SCAN*](http://doc.redisfans.com/key/scan.html#scan) 这类增量式迭代命令来说， 因为在对键进行增量式迭代的过程中， 键可能会被修改， 所以增量式迭代命令只能对被返回的元素提供有限的保证 （offer limited guarantees about the returned elements）。
- 因为 [*SCAN*](http://doc.redisfans.com/key/scan.html#scan) 、 [*SSCAN*](http://doc.redisfans.com/set/sscan.html#sscan) 、 [*HSCAN*](http://doc.redisfans.com/hash/hscan.html#hscan) 和 [*ZSCAN*](http://doc.redisfans.com/sorted_set/zscan.html#zscan) 四个命令的工作方式都非常相似， 所以这个文档会一并介绍这四个命令， 但是要记住：
- - [*SSCAN*](http://doc.redisfans.com/set/sscan.html#sscan) 命令、 [*HSCAN*](http://doc.redisfans.com/hash/hscan.html#hscan) 命令和 [*ZSCAN*](http://doc.redisfans.com/sorted_set/zscan.html#zscan) 命令的第一个参数总是一个数据库键。
  - 而 [*SCAN*](http://doc.redisfans.com/key/scan.html#scan) 命令则不需要在第一个参数提供任何数据库键 —— 因为它迭代的是当前数据库中的所有数据库键。



#### 18. **OBJECT subcommand [arguments [arguments]] **（2020年5月22日 暂不考虑）

- [OBJECT](http://doc.redisfans.com/key/object.html#object) 命令允许从内部察看给定 `key` 的 Redis 对象。
- 它通常用在除错(debugging)或者了解为了节省空间而对 `key` 使用特殊编码的情况。
- 当将Redis用作缓存程序时，你也可以通过 [OBJECT](http://doc.redisfans.com/key/object.html#object) 命令中的信息，决定 `key` 的驱逐策略(eviction policies)。
- OBJECT 命令有多个子命令：
- - `OBJECT REFCOUNT ` 返回给定 `key` 引用所储存的值的次数。此命令主要用于除错。
  - `OBJECT ENCODING ` 返回给定 `key` 锁储存的值所使用的内部表示(representation)。
  - `OBJECT IDLETIME ` 返回给定 `key` 自储存以来的空转时间(idle， 没有被读取也没有被写入)，以秒为单位。
- 对象可以以多种方式编码：
- - 字符串可以被编码为 `raw` (一般字符串)或 `int` (用字符串表示64位数字是为了节约空间)。
  - 列表可以被编码为 `ziplist` 或 `linkedlist` 。 `ziplist` 是为节约大小较小的列表空间而作的特殊表示。
  - 集合可以被编码为 `intset` 或者 `hashtable` 。 `intset` 是只储存数字的小集合的特殊表示。
  - 哈希表可以编码为 `zipmap` 或者 `hashtable` 。 `zipmap` 是小哈希表的特殊表示。
  - 有序集合可以被编码为 `ziplist` 或者 `skiplist` 格式。 `ziplist` 用于表示小的有序集合，而 `skiplist` 则用于表示任何大小的有序集合。
- 假如你做了什么让 Redis 没办法再使用节省空间的编码时(比如将一个只有 1 个元素的集合扩展为一个有 100 万个元素的集合)，特殊编码类型(specially encoded types)会自动转换成通用类型(general type)。



#### 19. **DUMP**、**MIGRATE** 、**OBJECT**、**RESTORE** 、**SORT**  （之后研究）