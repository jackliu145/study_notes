#	spring配置声明式事务

````
	<!-- 配置一个数据源 -->
	<bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="url" value="${jdbc.url}"/>
        <property name="username" value="${jdbc.username}"/>
        <property name="password" value="${jdbc.password}"/>
    </bean>
    
       <!--定义一个事务管理器-->
    <bean id="tx-manager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"></property>
    </bean>
    
     <!-- 定义事务的委托者 -->
    <tx:advice id="txAdvice" transaction-manager="tx-manager">
        <tx:attributes>
            <tx:method name="transfor*" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>
    
     <aop:config>
        <aop:advisor advice-ref="txAdvice" pointcut="execution(* com.lsh.service.impl.*.*(..))"></aop:advisor>
    </aop:config>
	
````

