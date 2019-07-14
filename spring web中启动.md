

#	RootApplicationContext启动

​	在AbstractAnnotationConfigDispatcherServletInitializer类中，执行下列方法，创建了一个WebApplicaitonContext实例。

```	java
	@Override
	protected WebApplicationContext createRootApplicationContext() {
		Class<?>[] configClasses = getRootConfigClasses();   // 该方法由实现类提供
		if (!ObjectUtils.isEmpty(configClasses)) {
			AnnotationConfigWebApplicationContext rootAppContext = new AnnotationConfigWebApplicationContext();
			rootAppContext.register(configClasses);
			return rootAppContext;
		}
		else {
			return null;          // 返回空啊
		}
	}

```

​	跟踪该方法，发现其被AbstractContextLoaderInitializer的registerContextLoaderListener方法调用。

```	JAVA
	protected void registerContextLoaderListener(ServletContext servletContext) {
		WebApplicationContext rootAppContext = createRootApplicationContext();   // 
		if (rootAppContext != null) {
            // 创建监听器，并将applicaitonContext作为构造参数传入。
			servletContext.addListener(new ContextLoaderListener(rootAppContext)); 
		}
		else {
			logger.debug("No ContextLoaderListener registered, as " +
					"createRootApplicationContext() did not return an application context");
		}
	}

```

​	registerContextLoaderListener方法在AbstractContextLoaderInitializer类的startup方法中调用。AbstractContextLoaderInitializer抽象类

```	java
	@Override
	public void onStartup(ServletContext servletContext) throws ServletException {
		registerContextLoaderListener(servletContext);   //  注册监听器
	}

```

​	查看AbstractContextLoaderInitializer的父接口，为WebApplicationInitializer,该接口的实现类用于初始化Servlet容器。

````	java
public interface WebApplicationInitializer {

	/**
	 * Configure the given {@link ServletContext} with any servlets, filters, listeners
	 * context-params and attributes necessary for initializing this web application. See
	 * examples {@linkplain WebApplicationInitializer above}.
	 * @param servletContext the {@code ServletContext} to initialize
	 * @throws ServletException if any call against the given {@code ServletContext}
	 * throws a {@code ServletException}
	 */
	void onStartup(ServletContext servletContext) throws ServletException;

}

````

总结，

1.创建一个WebApplicationContext。

2.创建一个ContextLoaderListener实例，并将WebApplicationContext作为构造参数传入。

3.Servlet容器启动后。会调用ContextLoaderListener实例的contextInitialized生命周期方法。

```	java
	@Override
	public void contextInitialized(ServletContextEvent event) {
		initWebApplicationContext(event.getServletContext());
	}

	
```

4.initWebApplicationContext方法会判断是否存在ApplicationContext（spring应用上下文）,如果不存在（采用web.xml配置时，spring应用上下文由contextLoaderListener创建。），就会创建一个spring的应用上下文。

5.initWebApplicationContext 方法会调用configureAndRefreshWebApplicationContext方法，同时将applicaitonContext设置到ServletContext中，键值为：

```	
String ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = WebApplicationContext.class.getName() + ".ROOT";
```



#	ServeletApplicationContext  启动

​	在AbstractDispatcherServletInitializer类的onStartup方法中创建了registerDispatcherServlet

```	JAVA
	@Override
	public void onStartup(ServletContext servletContext) throws ServletException {
		super.onStartup(servletContext);

		registerDispatcherServlet(servletContext);
	}

```

​	跟踪registerDispatcherServlet方法，

```	JAVA
protected void registerDispatcherServlet(ServletContext servletContext) {
		String servletName = getServletName();
		Assert.hasLength(servletName, "getServletName() may not return empty or null");
		// 创建一个ApplicationContext上下文
		WebApplicationContext servletAppContext = createServletApplicationContext();
		Assert.notNull(servletAppContext,
				"createServletApplicationContext() did not return an application " +
				"context for servlet [" + servletName + "]");

		DispatcherServlet dispatcherServlet = new DispatcherServlet(servletAppContext);
		ServletRegistration.Dynamic registration = servletContext.addServlet(servletName, dispatcherServlet);
		Assert.notNull(registration,
				"Failed to register servlet with name '" + servletName + "'." +
				"Check if there is another servlet registered under the same name.");

		registration.setLoadOnStartup(1);
		registration.addMapping(getServletMappings());
		registration.setAsyncSupported(isAsyncSupported());

		Filter[] filters = getServletFilters();
		if (!ObjectUtils.isEmpty(filters)) {
			for (Filter filter : filters) {
				registerServletFilter(servletContext, filter);
			}
		}

		customizeRegistration(registration);
	}
```

​	跟踪createServletApplicationContext方法，发现AbstractAnnotationConfigDispatcherServletInitializer实现类提供了该方法的重载方法。

````JAVA
	@Override
	protected WebApplicationContext createServletApplicationContext() {
		AnnotationConfigWebApplicationContext servletAppContext = new AnnotationConfigWebApplicationContext();
		Class<?>[] configClasses = getServletConfigClasses();
		if (!ObjectUtils.isEmpty(configClasses)) {
			servletAppContext.register(configClasses);
		}
		return servletAppContext;
	}
````





