#	Spring中的几个重要的接口

##	ImportBeanDefinitionRegistrar	

```	java
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);
	// 参数importingClassMatadata 表示正在导入ImportBeanDefinitionRegistrar实现类的类的注解。
	// 参数registry 当前bean定义的注册表，此时可以添加Bean定义 
}

```



##	ImportSelector







##	BeanPostProcessor





##	BeanFactory	





##	FactoryBean





##	InitializingBean





##	Condition	

