## Mybatis insert源码流程

**sql xml**

```xml
<insert id="insertUser" parameterType="my.bean.User" useGeneratedKeys="true" keyProperty="id">
    INSERT INTO
    `user`(`name`) VALUES (#{user.name})
</insert>
```

**interface**

```java
package my.dao;

import org.apache.ibatis.annotations.Param;

import my.bean.User;

public interface UserMapper {

	/**
	 * 添加用户
	 * 
	 * @param user
	 * @return
	 */
	int insertUser(@Param("user") User user);
}
```

**mybatis启动类**

```java
public static void main(String[] args) {
		System.out.println("程序开始");

		String resource = "my/config/mybatis-config.xml";

		InputStream inputStream = null;
		try {
			inputStream = Resources.getResourceAsStream(resource);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);


		// 方式2
		try (SqlSession session = sqlSessionFactory.openSession()) {

			// 这里获取的 UserMapper 实际上是被 jdk动态代理的
			UserMapper mapper = session.getMapper(UserMapper.class);

//			<!-- 因为开启了事务，insert时需要执行commit -->
//			<transactionManager type="JDBC" />
			User user = new User();
			user.setName("王五");
            // 这里打断点进去
			int result = mapper.insertUser(user);
			session.commit();
			System.out.println(result + "\t" + user.toString());

		}

		System.out.println("程序结束");

	}

```

**进入mybatis创建的jdk动态代理UserMapper类**

```java
/**
 * 执行mapper接口时，真正执行的方法
 */
@Override
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
    try {
        if (Object.class.equals(method.getDeclaringClass())) {
            // 执行的是Object的方法则不走代理，直接放行
            return method.invoke(this, args);

        } else if (method.isDefault()) {
            if (privateLookupInMethod == null) {
                return invokeDefaultMethodJava8(proxy, method, args);
            } else {
                return invokeDefaultMethodJava9(proxy, method, args);
            }

        }

    } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);

    }

    // cachedMapperMethod(method)，获取map中缓存的MapperMethod方法，没有则创建一个
    final MapperMethod mapperMethod = cachedMapperMethod(method);

    // 这里执行对应的增删改查，并返回结果
    return mapperMethod.execute(sqlSession, args);
}
```

**mapperMethod.execute(sqlSession, args);**

- case INSERT:这里是insert的语句流程

```java
/**
 * 执行对应的增删改查
 * 
 * @param sqlSession
 * @param args
 * @return
 */
public Object execute(SqlSession sqlSession, Object[] args) {
    Object result;
    switch (command.getType()) {
        case INSERT: {
            Object param = method.convertArgsToSqlCommandParam(args);
			int insertResult = sqlSession.insert(command.getName(), param);
			result = rowCountResult(insertResult);
			break;
        }
        case UPDATE: {
            ...
        }
        case DELETE: {
            ...
        }
        case SELECT:
            ...
        case FLUSH:
            ...
        default:
            throw new BindingException("Unknown execution method for: " + command.getName());
    }
    if (result == null && method.getReturnType().isPrimitive() && !method.returnsVoid()) {
        throw new BindingException("Mapper method '" + command.getName()
                                   + " attempted to return null from a method with a primitive return type (" + method.getReturnType()
                                   + ").");
    }
    return result;
}
```

**sqlSession.insert(command.getName(), param)**

- 调用了update(statement, parameter);

```java
@Override
public int insert(String statement, Object parameter) {
    // statemengt:my.dao.UserMapper.insertUser
    // parameter:{user=User [id=0, name=王五], param1=User [id=0, name=王五]}
	return update(statement, parameter);
}
```

```java
@Override
public int update(String statement, Object parameter) {
    try {
        dirty = true;
        // 从一个hashmap中拿到MappedStatement，里面有当前需要的sql信息等等
        MappedStatement ms = configuration.getMappedStatement(statement);
        // 把穿进来的parameter，转成了两个hashmap
        // {user=User [id=0, name=王五], param1=User [id=0, name=王五]}
        Object object = wrapCollection(parameter);
        // 执行insert操作
        return executor.update(ms, object);
    } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error updating database.  Cause: " + e, e);
    } finally {
        ErrorContext.instance().reset();
    }
}
```

**executor.update(ms, object);**

```java
@Override
public int update(MappedStatement ms, Object parameterObject) throws SQLException {
    // 刷新缓存
    flushCacheIfRequired(ms);
    return delegate.update(ms, parameterObject);
}

private void flushCacheIfRequired(MappedStatement ms) {
    Cache cache = ms.getCache();
    if (cache != null && ms.isFlushCacheRequired()) {
        tcm.clear(cache);
    }
}
```

**delegate.update(ms, parameterObject);**

```java
@Override
public int update(MappedStatement ms, Object parameter) throws SQLException {
    // 指定错误信息
    ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
    if (closed) {
        throw new ExecutorException("Executor was closed.");
    }
    // 这里清除了一级缓存
    clearLocalCache();
    return doUpdate(ms, parameter);
}
```

**doUpdate(ms, parameter);**

```java
@Override
public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
    Statement stmt = null;
    try {
        // mybatis核心配置类，可以用这个拿到所有东西
        Configuration configuration = ms.getConfiguration();
        // 这里完成了sql和param值的保存生成了一个RoutingStatementHandler
        StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, null, null);
        // 获取了prepareStatement，里面有怎么获取jdbc连接方法，和解析sql方法
        stmt = prepareStatement(handler, ms.getStatementLog());
        // 通过handle执行修改
        return handler.update(stmt);
    } finally {
        closeStatement(stmt);
    }
}
```

**handler.update(stmt);**

```java
@Override
public int update(Statement statement) throws SQLException {
    return delegate.update(statement);
}
```

**delegate.update(statement);**

```java
@Override
public int update(Statement statement) throws SQLException {
    // jdbc的PreparedStatement
    PreparedStatement ps = (PreparedStatement) statement;
    // 执行insert sql
    ps.execute();
    // 获取了执行的结果，影响的行数
    int rows = ps.getUpdateCount();
    
    Object parameterObject = boundSql.getParameterObject();
    KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
    
    // 这里处理获取自增id的逻辑
    keyGenerator.processAfter(executor, mappedStatement, ps, parameterObject);
    return rows;
}
```

**keyGenerator.processAfter(executor, mappedStatement, ps, parameterObject);**

```java
@Override
public void processAfter(Executor executor, MappedStatement ms, Statement stmt, Object parameter) {
    processBatch(ms, stmt, parameter);
}
```

**processBatch(ms, stmt, parameter);**

```java
public void processBatch(MappedStatement ms, Statement stmt, Object parameter) {
    // keyProperties:[id]
    // 这里应该获取的是xml中配置的 keyProperty="id"
    final String[] keyProperties = ms.getKeyProperties();
    if (keyProperties == null || keyProperties.length == 0) {
        return;
    }
    
    try (ResultSet rs = stmt.getGeneratedKeys()) {
        final ResultSetMetaData rsmd = rs.getMetaData();
        // 获取mybatis核心配置类
        final Configuration configuration = ms.getConfiguration();
        // 这里判断指定的 keyProperties 配置的是不是超出给定类里面的filed数量
        if (rsmd.getColumnCount() < keyProperties.length) {
            // Error?
        } else {
            assignKeys(configuration, rs, rsmd, keyProperties, parameter);
        }
    } catch (Exception e) {
        throw new ExecutorException("Error getting generated key or setting result to parameter object. Cause: " + e, e);
    }
}
```

**assignKeys(configuration, rs, rsmd, keyProperties, parameter);**

```java
@SuppressWarnings("unchecked")
private void assignKeys(Configuration configuration, ResultSet rs, ResultSetMetaData rsmd, String[] keyProperties,
                        Object parameter) throws SQLException {
    if (parameter instanceof ParamMap || parameter instanceof StrictMap) {
        // Multi-param or single param with @Param
        assignKeysToParamMap(configuration, rs, rsmd, keyProperties, (Map<String, ?>) parameter);
    } else if (parameter instanceof ArrayList && !((ArrayList<?>) parameter).isEmpty()
               && ((ArrayList<?>) parameter).get(0) instanceof ParamMap) {
        ...
    } else {
        ...
    }
}
```

**assignKeysToParamMap(configuration, rs, rsmd, keyProperties, (Map<String, ?>) parameter);**

- 这个方法赋值给指定的属性里面了，目前看不懂。

```java
private void assignKeysToParamMap(Configuration configuration, ResultSet rs, ResultSetMetaData rsmd,
                                  String[] keyProperties, Map<String, ?> paramMap) throws SQLException {
    if (paramMap.isEmpty()) {
        return;
    }
    Map<String, Entry<Iterator<?>, List<KeyAssigner>>> assignerMap = new HashMap<>();
    for (int i = 0; i < keyProperties.length; i++) {
        Entry<String, KeyAssigner> entry = getAssignerForParamMap(configuration, rsmd, i + 1, paramMap, keyProperties[i],
                                                                  keyProperties, true);
        Entry<Iterator<?>, List<KeyAssigner>> iteratorPair = assignerMap.computeIfAbsent(entry.getKey(),
                                                                                         k -> entry(collectionize(paramMap.get(k)).iterator(), new ArrayList<>()));
        iteratorPair.getValue().add(entry.getValue());
    }
    long counter = 0;
    while (rs.next()) {
        for (Entry<Iterator<?>, List<KeyAssigner>> pair : assignerMap.values()) {
            if (!pair.getKey().hasNext()) {
                throw new ExecutorException(String.format(MSG_TOO_MANY_KEYS, counter));
            }
            Object param = pair.getKey().next();
            pair.getValue().forEach(x -> x.assign(rs, param));
        }
        counter++;
    }
}
```

