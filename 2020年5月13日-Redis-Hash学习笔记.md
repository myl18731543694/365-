## Redis Hash类型命令

#### 1. **HSET key field value**

- 将哈希表 `key` 中的域 `field` 的值设为 `value` 。
- 如果 `key` 不存在，一个新的哈希表被创建并进行 `HSET` 操作。
- 如果域 `field` 已经存在于哈希表中，旧值将被覆盖。

> **不存在则创建**
>
> 127.0.0.1:6379> hset a 1 2
> (integer) 1
> 127.0.0.1:6379> hget a 1
> "2"
>
> **存在则直接覆盖**
>
> 127.0.0.1:6379> hset a 1 4
> (integer) 0
> 127.0.0.1:6379> hget a 1
> "4"



#### 2. **HGET key field**

- 返回哈希表 `key` 中给定域 `field` 的值。

> 127.0.0.1:6379> hget a 1
> "2"



#### 3. **HMSET key field value [field value ...]**

- 同时将多个 `field-value` (域-值)对设置到哈希表 `key` 中。
- 此命令会覆盖哈希表中已存在的域。
- 如果 `key` 不存在，一个空哈希表被创建并执行 [HMSET](http://doc.redisfans.com/hash/hmset.html#hmset) 操作。

> 127.0.0.1:6379> flushdb
> OK
>
> **批量添加**
>
> 127.0.0.1:6379> hmset a 1 1 2 2 3 3 4 4
> OK
> 127.0.0.1:6379> hget a 1
> "1"
> 127.0.0.1:6379> hgetall a
> 1) "1"
> 2) "1"
> 3) "2"
> 4) "2"
> 5) "3"
> 6) "3"
> 7) "4"
> 8) "4"



#### 4. **HMGET key field [field ...]**

- 返回哈希表 `key` 中，一个或多个给定域的值。
- 如果给定的域不存在于哈希表，那么返回一个 `nil` 值。
- 因为不存在的 `key` 被当作一个空哈希表来处理，所以对一个不存在的 `key` 进行 [HMGET](http://doc.redisfans.com/hash/hmget.html#hmget) 操作将返回一个只带有 `nil` 值的表。

> 127.0.0.1:6379> hmset a 1 1 2 2 3 3 4 4
> OK
>
> **批量获取**
>
> 127.0.0.1:6379> hmget a 1 2
> 1) "1"
> 2) "2"
>
> **某个值为空不报错返回null**
>
> 127.0.0.1:6379> hmget a 1 4 5
> 1) "1"
> 2) "4"
> 3) (nil)



#### 5. **HSETNX key field value**

- 将哈希表 `key` 中的域 `field` 的值设置为 `value` ，当且仅当域 `field` 不存在。
- 若域 `field` 已经存在，该操作无效。
- 如果 `key` 不存在，一个新哈希表被创建并执行 `HSETNX` 命令。

> 127.0.0.1:6379> hset a 1 1
> (integer) 1
>
> **修改失败，field1已经有值了**
>
> 127.0.0.1:6379> HSETNX a 1 2
> (integer) 0
> 127.0.0.1:6379> hget a 1
> "1"
>
> **成功了**
>
> 127.0.0.1:6379> hsetnx a 2 3
> (integer) 1
> 127.0.0.1:6379> hget a 2
> "3"



#### 6. **HGETALL key**

- 返回哈希表 `key` 中，所有的域和值。
- 在返回值里，紧跟每个域名(field name)之后是域的值(value)，所以返回值的长度是哈希表大小的两倍。

> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> hmset a 1 1 2 2 3 3
> OK
>
> **获取多有key value成功了**
>
> 127.0.0.1:6379> hgetall a
> 1) "1"
> 2) "1"
> 3) "2"
> 4) "2"
> 5) "3"
> 6) "3"
>
> **获取空值返回**
>
> 127.0.0.1:6379> hgetall b
> (empty list or set)



#### 7. **HDEL key field [field ...]**

- 删除哈希表 `key` 中的一个或多个指定域，不存在的域将被忽略。

> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> hmset a 1 1 2 2 3 3
> OK
>
> **删除1 3 4，不存在的会被忽略**
>
> 127.0.0.1:6379> hdel a 1 3 4
> (integer) 2
> 127.0.0.1:6379> hgetall a
> 1) "2"
> 2) "2"



#### 8. **HEXISTS key field**

- 查看哈希表 `key` 中，给定域 `field` 是否存在。

> 127.0.0.1:6379> FLUSHDB
> OK
> 127.0.0.1:6379> hset a 1 1
> (integer) 1
> **返回1 是存在**
> 127.0.0.1:6379> HEXISTS a 1
> (integer) 1
>
> **返回0，不存在**
>
> 127.0.0.1:6379> HEXISTS a 2
> (integer) 0



#### 9. **HKEYS key**

- 返回哈希表 `key` 中的所有域。

> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> hmset a 1 1 2 2 3 3 4 4
> OK
>
> **返回了多有的key，很像java里的keysets（）方法**
>
> 127.0.0.1:6379> hkeys a
> 1) "1"
> 2) "2"
> 3) "3"
> 4) "4"



#### 10. **HVALS key**

- 返回哈希表 `key` 中所有域的值。

> 127.0.0.1:6379> hmset a 1 v1 2 v2 3 v3 4 v4
> OK
>
> **value的列表**
>
> 127.0.0.1:6379> HVALS a
> 1) "v1"
> 2) "v2"
> 3) "v3"
> 4) "v4"



#### 11. **HLEN key**

- 返回哈希表 `key` 中域的数量。

> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> hmset a 1 1 2 2 3 3 4 4
> OK
> 127.0.0.1:6379> hkeys a
> 1) "1"
> 2) "2"
> 3) "3"
> 4) "4"
>
> **返回了key的数量**
>
> 127.0.0.1:6379> HLEN a
> (integer) 4



#### 12. **HINCRBY key field increment**

- 为哈希表 `key` 中的域 `field` 的值加上增量 `increment` 。
- 增量也可以为负数，相当于对给定域进行减法操作。
- 如果 `key` 不存在，一个新的哈希表被创建并执行 `HINCRBY` 命令。
- 如果域 `field` 不存在，那么在执行命令前，域的值被初始化为 `0` 。
- 对一个储存字符串值的域 `field` 执行 `HINCRBY` 命令将造成一个错误。
- 本操作的值被限制在 64 位(bit)有符号数字表示之内。

> 127.0.0.1:6379> flushdb
> OK
> **没有key value，会初始化为0，然后进行增加**
> 127.0.0.1:6379> HINCRBY a 1 2
> (integer) 2
> 127.0.0.1:6379> hget a 1
> "2"
>
> **是int类型可以直接增加**
>
> 127.0.0.1:6379> HINCRBY a 1 3
> (integer) 5
>
> **float类型就会报错**
>
> 127.0.0.1:6379> hset a 1 1.0
> (integer) 0
> 127.0.0.1:6379> hget a 1
> "1.0"
> 127.0.0.1:6379> HINCRBY a 1 3
> (error) ERR hash value is not an integer
>
> **char字符串等等也不可以**
>
> 127.0.0.1:6379> hset a 1 a
> (integer) 0
> 127.0.0.1:6379> hget a 1
> "a"
> 127.0.0.1:6379> HINCRBY a 1
> (error) ERR wrong number of arguments for 'hincrby' command



#### 13. **HINCRBYFLOAT key field increment**

- 为哈希表 `key` 中的域 `field` 加上浮点数增量 `increment` 。
- 如果哈希表中没有域 `field` ，那么 [HINCRBYFLOAT](http://doc.redisfans.com/hash/hincrbyfloat.html#hincrbyfloat) 会先将域 `field` 的值设为 `0` ，然后再执行加法操作。
- 如果键 `key` 不存在，那么 [HINCRBYFLOAT](http://doc.redisfans.com/hash/hincrbyfloat.html#hincrbyfloat) 会先创建一个哈希表，再创建域 `field` ，最后再执行加法操作。
- 当以下任意一个条件发生时，返回一个错误：
- - 域 `field` 的值不是字符串类型(因为 redis 中的数字和浮点数都以字符串的形式保存，所以它们都属于字符串类型）
  - 域 `field` 当前的值或给定的增量 `increment` 不能解释(parse)为双精度浮点数(double precision floating point number)
- [HINCRBYFLOAT](http://doc.redisfans.com/hash/hincrbyfloat.html#hincrbyfloat) 命令的详细功能和 [*INCRBYFLOAT*](http://doc.redisfans.com/string/incrbyfloat.html#incrbyfloat) 命令类似，请查看 [*INCRBYFLOAT*](http://doc.redisfans.com/string/incrbyfloat.html#incrbyfloat) 命令获取更多相关信息。

> **就是在12 HINCRBY key field increment 基础上加了float类型支持**
>
> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> hget a 1
> "4"
> 127.0.0.1:6379> hset a 1 1.1
> (integer) 0
> 127.0.0.1:6379> HINCRBYFLOAT a 1 3
> "4.1"
>
> **char，其他类型，依然会报错**
>
> 127.0.0.1:6379> hset a 1 a
> (integer) 0
> 127.0.0.1:6379> HINCRBYFLOAT a 1 1
> (error) ERR hash value is not a valid float 



#### 12. **HSCAN key cursor [MATCH pattern] [COUNT count]** （2020年5月13日 10:30:12暂时保留）

- 保留，暂时没看