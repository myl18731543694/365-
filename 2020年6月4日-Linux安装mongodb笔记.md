## Linux安装mongodb笔记

#### 1. 官网下载mongodb安装包

- www.mongodb.com
- 点击software，社区版或者企业版
- 选择对应的操作系统 -> 右边的all version binaries -> 选择自己想下载的版本



#### 2. 安装mongodb

2.1 解压安装包

```
tar -zxvf mongodb-linux-x86_64-4.0.1.tgz
```



2.2 将解压后的文件移动到/usr/local/目录下

```
mv -r mongodb-linux-x86_64-4.0.1 /usr/local/mongodb
```



2.3 创建db目录和日志文件

```
cd /usr/local/mongodb/
mkdir -p ./data/db
mkdir -p ./logs
touch ./logs/mongodb.log
```



2.4 创建mongodb.conf文件

```
vim mongodb.conf
```

```
#端口号
port=27017
#db目录
dbpath=/usr/local/mongodb/data/db
#日志目录
logpath=/usr/local/mongodb/logs/mongodb.log
#后台运行，以守护进程的方式运行
fork=true
#日志输出
logappend=true
#允许远程IP连接
bind_ip=0.0.0.0
```



#### 3. 启动mongodb

3.1 启动

```
cd /usr/local/mongodb/bin
./mongod -f mongodb.conf
```



3.2 检查是否启动成功

```
netstat -lanp | grep "27017"
```



#### 4. 连接mongodb数据库

```
cd /usr/local/mongodb/bin
./mongo
```



#### 5. 停止mongodb

```
cd /usr/local/mongodb/bin
./mongo
```

```
use admin
db.runCommand("shutdown")
```

或者

```
use admin
db.shutdownServer()
```

