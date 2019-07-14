#	springmvc处理跨域请求的方式

##	@CrossOrigin注解

在Controller的方法或者类上添加@CrossOrigin注解即可自定义过滤器

````java
public class MyCorsFilter implements Filter {

	@Override
	public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
			throws IOException, ServletException {
		HttpServletRequest httpRequest = (HttpServletRequest)request;
		HttpServletResponse httpResponse = (HttpServletResponse)response;
		
		// 允许跨域请求
		httpResponse.addHeader("Access-Control-Allow-Origin", "*");
		httpResponse.addHeader("Vary", " Origin, Access-Control-Request-Method, Access-Control-Request-Headers");

		chain.doFilter(httpRequest, response);
	
	}
}
````



##	配置springmvc.xml

````JAVA
	<!-- 跨域请求 -->
	<mvc:cors>
		<mvc:mapping path="/*" />
	</mvc:cors>

````



##	springmvc的java配置类中定义

```JAVA
@Configuration
@EnableWebMvc
public class WebConfig implements WebMvcConfigurer {

    @Override
    public void addCorsMappings(CorsRegistry registry) {

        registry.addMapping("/api/**")
            .allowedOrigins("http://domain2.com")
            .allowedMethods("PUT", "DELETE")
            .allowedHeaders("header1", "header2", "header3")
            .exposedHeaders("header1", "header2")
            .allowCredentials(true).maxAge(3600);

        // Add more mappings...
    }
}
```

