#	MyBatis执行流程分析

##	分析一个insert流程

* SqlSessionFactoryBuilder根据配置文件构造一个DefaultSqlSessionFactory对象。

* DefaultSqlSessionFactory对象调用openSqlSession方法，返回要给DefaultSqlSession对象。

* openSqlSession方法会调用内部的openSessionFromDataSource方法，创建一个sqlSession对象。

  ```	java
    private SqlSession openSessionFromDataSource(ExecutorType execType, TransactionIsolationLevel level, boolean autoCommit) {
      Transaction tx = null;   
      try {
        final Environment environment = configuration.getEnvironment();
        final TransactionFactory transactionFactory = getTransactionFactoryFromEnvironment(environment);
          // 根据配置文件获取事务管理器。
        tx = transactionFactory.newTransaction(environment.getDataSource(), level, autoCommit);   
          // 创建执行器。
        final Executor executor = configuration.newExecutor(tx, execType);
          // 返回一个DefaultSqlSession对象。
        return new DefaultSqlSession(configuration, executor, autoCommit);
      } catch (Exception e) {
        closeTransaction(tx); // may have fetched a connection so lets call close()
        throw ExceptionFactory.wrapException("Error opening session.  Cause: " + e, e);
      } finally {
          // 一个保存错误上下文的单例对象
        ErrorContext.instance().reset();
      }
    }
  ```

* 通过sqlSession对象调用insert方法，insert方法调用内部update方法。

  ```	JAVA
    @Override
    public int update(String statement, Object parameter) {
      try {
        dirty = true;    // 该属性是sqlSession对象的私有成员，决定事务事务需要回滚。
        // 根据配置文件，查找sqlmapper配置文件中id所对象的sql语句。
        MappedStatement ms = configuration.getMappedStatement(statement);
        // 由执行器来执行jdbc操作。默认是executor是CacheExecutor， wrapCollection方法处理参入参数，将集合转化成map对象，普通对象直接返回。
        return executor.update(ms, wrapCollection(parameter));
      } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error updating database.  Cause: " + e, e);
      } finally {
        ErrorContext.instance().reset();
      }
    }
  ```

* 进入executor的update方法

  ````JAVA
    @Override
    public int update(MappedStatement ms, Object parameterObject) throws SQLException {
      flushCacheIfRequired(ms);  // ms中的configuration对象决定了是否刷新缓存。
      return delegate.update(ms, parameterObject);   // SimpleExecutor来执行Update操作。
    }
  	// 父类的一个实现方法。
      @Override
    public int update(MappedStatement ms, Object parameter) throws SQLException {
      ErrorContext.instance().resource(ms.getResource()).activity("executing an update").object(ms.getId());
      if (closed) {
        throw new ExecutorException("Executor was closed.");
      }
      clearLocalCache();   //  清空本地缓存。
      return doUpdate(ms, parameter);   // 执行重载的更新操作。
   
    }
  
  ````

* 继续跟踪doUpdate方法。

  ```	JAVA
    @Override
    public int doUpdate(MappedStatement ms, Object parameter) throws SQLException {
      Statement stmt = null;
      try {
        Configuration configuration = ms.getConfiguration();
          // 处理器，用于处理一个statement,该方法内部还回加载所有插件。RowBounds设置查询的最多条数。
        StatementHandler handler = configuration.newStatementHandler(this, ms, parameter, RowBounds.DEFAULT, null, null);
        // 该方法会生成一个preparedStatement对象。
        stmt = prepareStatement(handler, ms.getStatementLog());
        return handler.update(stmt);
      } finally {
        closeStatement(stmt);
      }
    }
  ```

* 继续跟踪handler.update方法。

  ```JAVA
    @Override
    public int update(Statement statement) throws SQLException {
      PreparedStatement ps = (PreparedStatement) statement;
        // 执行sql
      ps.execute();
      int rows = ps.getUpdateCount();
      Object parameterObject = boundSql.getParameterObject();
        
      KeyGenerator keyGenerator = mappedStatement.getKeyGenerator();
      keyGenerator.processAfter(executor, mappedStatement, ps, parameterObject);
      return rows;
    }
  
  ```

* sql语句执行完毕！！！

  

##	commit提交过程

* 执行sqlSession.commit();

  ```JAVA
   @Override
    public void commit() {
      commit(false);
    }
  
  	@Override
    public void commit(boolean force) {
      try {
        executor.commit(isCommitOrRollbackRequired(force));
        dirty = false;
      } catch (Exception e) {
        throw ExceptionFactory.wrapException("Error committing transaction.  Cause: " + e, e);
      } finally {
        ErrorContext.instance().reset();
      }
    }
  
  ```

* 跟踪isCommitOrRollbackRequired方法,如果没有开启自动提交，并且dirty为true时，executor.commit才会提交事务。

  ```JAVA
    private boolean isCommitOrRollbackRequired(boolean force) {
      return (!autoCommit && dirty) || force;
    }
  ```

* executor.commit方法内部会调用事务管理器进行事务提交

  ```JAVA
   @Override
    public void commit(boolean required) throws SQLException {
      if (closed) {
        throw new ExecutorException("Cannot commit, transaction is already closed");
      }
      clearLocalCache();
      flushStatements();
      if (required) {
        transaction.commit();
      }
    }
  ```

  ##	sqlSession.close()

* 该方法会判断dirty的值，来确定是否回滚事务。

  



