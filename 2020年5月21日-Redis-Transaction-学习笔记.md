## Redis Transaction

**MULTI（开始事务）**

- 标记一个事务块的开始。
- 事务块内的多条命令会按照先后顺序被放进一个队列当中，最后由 [*EXEC*](http://doc.redisfans.com/transaction/exec.html#exec) 命令原子性(atomic)地执行。

**DISCARD（回滚）**

- 取消事务，放弃执行事务块内的所有命令。
- 如果正在使用 [*WATCH*](http://doc.redisfans.com/transaction/watch.html#watch) 命令监视某个(或某些) key，那么取消所有监视，等同于执行命令 [*UNWATCH*](http://doc.redisfans.com/transaction/unwatch.html#unwatch) 。

**EXEC（提交）**

- 执行所有事务块内的命令。
- 假如某个(或某些) key 正处于 [*WATCH*](http://doc.redisfans.com/transaction/watch.html#watch) 命令的监视之下，且事务块中有和这个(或这些) key 相关的命令，那么 [EXEC](http://doc.redisfans.com/transaction/exec.html#exec) 命令只在这个(或这些) key 没有被其他命令所改动的情况下执行并生效，否则该事务被打断(abort)。

> **回滚验证**
>
> ```
> 127.0.0.1:6379> MULTI
> OK
> 127.0.0.1:6379> set a a
> QUEUED
> 127.0.0.1:6379> set b b
> QUEUED
> 127.0.0.1:6379> DISCARD
> OK
> 127.0.0.1:6379> get a
> (nil)
> 127.0.0.1:6379> get b
> (nil)
> ```
>
> **提交验证**
>
> ```
> 127.0.0.1:6379> MULTI
> OK
> 127.0.0.1:6379> set a a
> QUEUED
> 127.0.0.1:6379> set b b
> QUEUED
> 127.0.0.1:6379> exec
> 1) OK
> 2) OK
> 127.0.0.1:6379> get a
> "a"
> ```
>
> **提交过程中出错，全部回滚**
>
> ```
> 127.0.0.1:6379> MULTI
> OK
> 127.0.0.1:6379> set a a
> QUEUED
> 127.0.0.1:6379> set b b
> QUEUED
> 127.0.0.1:6379> EXEC -
> (error) ERR unknown command 'ex'
> 127.0.0.1:6379> exec
> (error) EXECABORT Transaction discarded because of previous errors.
> ```



**WATCH key [key ...]**

- 监视一个(或多个) key ，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

**UNWATCH**

- 取消 [*WATCH*](http://doc.redisfans.com/transaction/watch.html#watch) 命令对所有 key 的监视。
- 如果在执行 [*WATCH*](http://doc.redisfans.com/transaction/watch.html#watch) 命令之后， [*EXEC*](http://doc.redisfans.com/transaction/exec.html#exec) 命令或 [*DISCARD*](http://doc.redisfans.com/transaction/discard.html#discard) 命令先被执行了的话，那么就不需要再执行 [UNWATCH](http://doc.redisfans.com/transaction/unwatch.html#unwatch) 了。
- 因为 [*EXEC*](http://doc.redisfans.com/transaction/exec.html#exec) 命令会执行事务，因此 [*WATCH*](http://doc.redisfans.com/transaction/watch.html#watch) 命令的效果已经产生了；而 [*DISCARD*](http://doc.redisfans.com/transaction/discard.html#discard) 命令在取消事务的同时也会取消所有对 key 的监视，因此这两个命令执行之后，就没有必要执行 [UNWATCH](http://doc.redisfans.com/transaction/unwatch.html#unwatch) 了。

> **WATCH 不允许放在 multi里面**
>
> ```
> 127.0.0.1:6379> MULTI
> OK
> 127.0.0.1:6379> WATCH a
> (error) ERR WATCH inside MULTI is not allowed
> ```
>
> ****
>
> **客户端1添加 a**
>
> ```
> 127.0.0.1:6379> flushdb
> OK
> 127.0.0.1:6379> set a a
> ```
>
> **客户端2监视 a，并且开启事务**
>
> ```
> 127.0.0.1:6379> WATCH a
> OK
> 127.0.0.1:6379> MULTI
> OK
> ```
>
> **客户端1 修改a的value**
>
> ```
> 127.0.0.1:6379> set a b
> OK
> ```
>
> **客户端2 修改a的value，最后发现结果丢失，因为a被其他客户端修改过了**
>
> ```
> 127.0.0.1:6379> set a c
> QUEUED
> 127.0.0.1:6379> set b b
> QUEUED
> 127.0.0.1:6379> exec
> (nil)
> 127.0.0.1:6379> get a
> "b"
> 127.0.0.1:6379> get b
> (nil)
> ```

