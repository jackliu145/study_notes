##	Spring启动流程

 * 调用构造方法ClasspathXmlApplicationContext

   ```JAVA
   	public ClassPathXmlApplicationContext(
   			String[] configLocations, boolean refresh, @Nullable ApplicationContext parent)
   			throws BeansException {
   
   		super(parent);
   		setConfigLocations(configLocations);
   		if (refresh) {
   			refresh();
   		}
   	}
   
   ```

 * 内部调用refresh方法。该方法是线程安全的。

   ```JAVA
   @Override
   	public void refresh() throws BeansException, IllegalStateException {
   		synchronized (this.startupShutdownMonitor) {
   			// Prepare this context for refreshing.
   			prepareRefresh();
   
   			// Tell the subclass to refresh the internal bean factory.
   			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
   
   			// Prepare the bean factory for use in this context.
   			prepareBeanFactory(beanFactory);
   
   			try {
   				// Allows post-processing of the bean factory in context subclasses.
   				postProcessBeanFactory(beanFactory);
   
   				// Invoke factory processors registered as beans in the context.
   				invokeBeanFactoryPostProcessors(beanFactory);
   
   				// Register bean processors that intercept bean creation.
   				registerBeanPostProcessors(beanFactory);
   
   				// Initialize message source for this context.
   				initMessageSource();
   
   				// Initialize event multicaster for this context.
   				initApplicationEventMulticaster();
   
   				// Initialize other special beans in specific context subclasses.
   				onRefresh();
   
   				// Check for listener beans and register them.
   				registerListeners();
   
   				// Instantiate all remaining (non-lazy-init) singletons.
   				finishBeanFactoryInitialization(beanFactory);
   
   				// Last step: publish corresponding event.
   				finishRefresh();
   			}
   
   			catch (BeansException ex) {
   				if (logger.isWarnEnabled()) {
   					logger.warn("Exception encountered during context initialization - " +
   							"cancelling refresh attempt: " + ex);
   				}
   
   				// Destroy already created singletons to avoid dangling resources.
   				destroyBeans();
   
   				// Reset 'active' flag.
   				cancelRefresh(ex);
   
   				// Propagate exception to caller.
   				throw ex;
   			}
   
   			finally {
   				// Reset common introspection caches in Spring's core, since we
   				// might not ever need metadata for singleton beans anymore...
   				resetCommonCaches();
   			}
   		}
   	}
   ```

*	obtainFreshBeanFactory方法内部获取一个DefaultListableBeanFactory实例。默认的spring工厂。

*	finishBeanFactoryInitialization该方法内部代码如下：

  ````JAVA
  /**
  	 * Finish the initialization of this context's bean factory,
  	 * initializing all remaining singleton beans.
  	 */
  	protected void finishBeanFactoryInitialization(ConfigurableListableBeanFactory beanFactory) {
  		// Initialize conversion service for this context.
  		if (beanFactory.containsBean(CONVERSION_SERVICE_BEAN_NAME) &&
  				beanFactory.isTypeMatch(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class)) {
  			beanFactory.setConversionService(
  					beanFactory.getBean(CONVERSION_SERVICE_BEAN_NAME, ConversionService.class));
  		}
  
  		// Register a default embedded value resolver if no bean post-processor
  		// (such as a PropertyPlaceholderConfigurer bean) registered any before:
  		// at this point, primarily for resolution in annotation attribute values.
  		if (!beanFactory.hasEmbeddedValueResolver()) {
  			beanFactory.addEmbeddedValueResolver(strVal -> getEnvironment().resolvePlaceholders(strVal));
  		}
  
  		// Initialize LoadTimeWeaverAware beans early to allow for registering their transformers early.
  		String[] weaverAwareNames = beanFactory.getBeanNamesForType(LoadTimeWeaverAware.class, false, false);
  		for (String weaverAwareName : weaverAwareNames) {
  			getBean(weaverAwareName);
  		}
  
  		// Stop using the temporary ClassLoader for type matching.
  		beanFactory.setTempClassLoader(null);
  
  		// Allow for caching all bean definition metadata, not expecting further changes.
  		beanFactory.freezeConfiguration();
  
  		// Instantiate all remaining (non-lazy-init) singletons.
  		beanFactory.preInstantiateSingletons();
  	}
  ````

*	beanFactory.preInstanceiateSingletons方法, 该方法会注册所有的单例Bean，并执行

  ```JAVA
  @Override
  	public void preInstantiateSingletons() throws BeansException {
  		if (this.logger.isDebugEnabled()) {
  			this.logger.debug("Pre-instantiating singletons in " + this);
  		}
  
  		// Iterate over a copy to allow for init methods which in turn register new bean definitions.
  		// While this may not be part of the regular factory bootstrap, it does otherwise work fine.
  		List<String> beanNames = new ArrayList<>(this.beanDefinitionNames);
  
  		// Trigger initialization of all non-lazy singleton beans...
  		for (String beanName : beanNames) {
  			RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
  			if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
                  // 判断是否为FactoryBean,例如SqlSessionFactoryBean。。。
  				if (isFactoryBean(beanName)) {   
  					Object bean = getBean(FACTORY_BEAN_PREFIX + beanName);
  					if (bean instanceof FactoryBean) {
  						final FactoryBean<?> factory = (FactoryBean<?>) bean;
  						boolean isEagerInit;
  						if (System.getSecurityManager() != null && factory instanceof SmartFactoryBean) {
  							isEagerInit = AccessController.doPrivileged((PrivilegedAction<Boolean>)
  											((SmartFactoryBean<?>) factory)::isEagerInit,
  									getAccessControlContext());
  						}
  						else {
  							isEagerInit = (factory instanceof SmartFactoryBean &&
  									((SmartFactoryBean<?>) factory).isEagerInit());
  						}
  						if (isEagerInit) {
  							getBean(beanName);
  						}
  					}
  				}
  				else {
  					getBean(beanName);
  				}
  			}
  		}
  
  		// Trigger post-initialization callback for all applicable beans...
  		for (String beanName : beanNames) {
  			Object singletonInstance = getSingleton(beanName);
  			if (singletonInstance instanceof SmartInitializingSingleton) {
  				final SmartInitializingSingleton smartSingleton = (SmartInitializingSingleton) singletonInstance;
  				if (System.getSecurityManager() != null) {
  					AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
  						smartSingleton.afterSingletonsInstantiated();
  						return null;
  					}, getAccessControlContext());
  				}
  				else {
  					smartSingleton.afterSingletonsInstantiated();
  				}
  			}
  		}
  	}
  ```

  

