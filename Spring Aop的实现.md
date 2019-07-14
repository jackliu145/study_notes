#	Spring Aop的实现

##	ProxyFactoryBean

​	该类用于创建一个代理对象。该类需要注入interceptorNames属性、 targetName 属性。。



targetName属性是目标对象的bean名称。

InterceptorNames是一个数组，该数组中的元素是org.aopalliance.intercept.MethodInterceptor、org.springframework.aop.Advisor的实现类。

两个不同的实现，对应的切入点的粒度不同。！！！！！！



Advisor是一个委托者，用于管理切面注入的粒度。需要注入一个PointCut和一个Advice属性。

PointCut属性需要注入ClassFilter和MethodFilter的对象。

