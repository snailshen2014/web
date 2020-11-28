# SpringMvc 自动配置

参考https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc

30.1.1 SpringMVC Auto-configuration

Spring Boot provides auto-configuration for Spring MVC that works well with most applications.

The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of `ContentNegotiatingViewResolver` and `BeanNameViewResolver` beans.
- Support for serving static resources, including support for WebJars (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-static-content))).
- Automatic registration of `Converter`, `GenericConverter`, and `Formatter` beans.
- Support for `HttpMessageConverters` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-message-converters)).
- Automatic registration of `MessageCodesResolver` (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-message-codes)).
- Static `index.html` support.
- Custom `Favicon` support (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-favicon)).
- Automatic use of a `ConfigurableWebBindingInitializer` bean (covered [later in this document](https://docs.spring.io/spring-boot/docs/2.1.18.RELEASE/reference/html/boot-features-developing-web-applications.html#boot-features-spring-mvc-web-binding-initializer)).



 ## 一、个性化配置MVC

If you want to keep Spring Boot MVC features and you want to add additional [MVC configuration](https://docs.spring.io/spring/docs/5.1.19.RELEASE/spring-framework-reference/web.html#mvc) (interceptors, formatters, view controllers, and other features), you can add your own `@Configuration` class of type `WebMvcConfigurer` but **without** `@EnableWebMvc`. If you wish to provide custom instances of `RequestMappingHandlerMapping`, `RequestMappingHandlerAdapter`, or `ExceptionHandlerExceptionResolver`, you can declare a `WebMvcRegistrationsAdapter` instance to provide such components.

If you want to take complete control of Spring MVC, you can add your own `@Configuration` annotated with `@EnableWebMvc`.

### 1、WebMvcConfigurer配置说明

WebMvcConfigurer配置类其实是`Spring`内部的一种配置方式，采用`JavaBean`的形式来代替传统的`xml`配置文件形式进行针对框架个性化定制，可以自定义一些Handler，Interceptor，ViewResolver，MessageConverter。基于java-based方式的spring mvc配置，需要创建一个**配置**类并实现**`WebMvcConfigurer`** 接口；

在Spring Boot 1.5版本都是靠重写**WebMvcConfigurerAdapter**的方法来添加自定义拦截器，消息转换器等。SpringBoot 2.0 后，该类被标记为@Deprecated（弃用）。官方推荐直接实现WebMvcConfigurer或者直接继承WebMvcConfigurationSupport，方式一实现WebMvcConfigurer接口（推荐），方式二继承WebMvcConfigurationSupport类，本文重点介绍第一种方式。

#### 1）、WebMvcConfigurer接口

```java
public interface WebMvcConfigurer {
    default void configurePathMatch(PathMatchConfigurer configurer) {
    }

    default void configureContentNegotiation(ContentNegotiationConfigurer configurer) {
    }

    default void configureAsyncSupport(AsyncSupportConfigurer configurer) {
    }

    default void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
    }

    default void addFormatters(FormatterRegistry registry) {
    }
		// 拦截器配置
    default void addInterceptors(InterceptorRegistry registry) {
    }
		//静态资源处理
    default void addResourceHandlers(ResourceHandlerRegistry registry) {
    }
		//跨域问题配置
    default void addCorsMappings(CorsRegistry registry) {
    }
		//视图跳转控制器
    default void addViewControllers(ViewControllerRegistry registry) {
    }

    default void configureViewResolvers(ViewResolverRegistry registry) {
    }

    default void addArgumentResolvers(List<HandlerMethodArgumentResolver> resolvers) {
    }

    default void addReturnValueHandlers(List<HandlerMethodReturnValueHandler> handlers) {
    }

    default void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    }

    default void extendMessageConverters(List<HttpMessageConverter<?>> converters) {
    }

    default void configureHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
    }

    default void extendHandlerExceptionResolvers(List<HandlerExceptionResolver> resolvers) {
    }

    @Nullable
    default Validator getValidator() {
        return null;
    }

    @Nullable
    default MessageCodesResolver getMessageCodesResolver() {
        return null;
    }
}
```

#### 2）、重要的接口使用说明

##### 2.1 addInterceptors:拦截器

* addInterceptor:需要一个实现了HandlerInterceptor接口的拦截器实例
* addPathPatterns:用于设置拦截器的过滤路径规则；`addPathPatterns("/**")`对所有请求都拦截
* excludePathPatterns：用于设置不需要拦截的过滤规则
* excludePathPatterns：用于设置不需要拦截的过滤规则

```java
@Override
public void addInterceptors(InterceptorRegistry registry) {
    super.addInterceptors(registry);
    registry.addInterceptor(new TestInterceptor()).addPathPatterns("/**").excludePathPatterns("/emp/toLogin","/emp/login","/js/**","/css/**","/images/**");
}
```



##### 2.2 addViewControllers：页面跳转

以前写SpringMVC的时候，如果需要访问一个页面，必须要写Controller类，然后再写一个方法跳转到页面，感觉好麻烦，其实重写WebMvcConfigurer中的addViewControllers方法即可达到效果了。

```java
    @Override
    public void addViewControllers(ViewControllerRegistry registry) {
        registry.addViewController("/toLogin").setViewName("login");
    }
```

值的指出的是，在这里重写addViewControllers方法，并不会覆盖**WebMvcAutoConfiguration**（Springboot自动配置）中的addViewControllers（在此方法中，Spring Boot将“/”映射至index.html），这也就意味着自己的配置和Spring Boot的自动配置同时有效，这也是我们推荐添加自己的MVC配置的方式。

##### 2.3 addResourceHandlers：静态资源

比如，我们想自定义静态资源映射目录的话，只需重写addResourceHandlers方法即可。

注：如果继承WebMvcConfigurationSupport类实现配置时必须要重写该方法，不在本文讨论范围内。

```java
@Configuration
public class MyWebMvcConfigurerAdapter implements WebMvcConfigurer {
    /**
     * 配置静态访问资源
     * @param registry
     */
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/my/**").addResourceLocations("classpath:/my/");
    }
}
```

* addResoureHandler：指的是对外暴露的访问路径
* addResourceLocations：指的是内部文件放置的目录

##### 2.4 configureDefaultServletHandling：默认静态资源处理器

```java
@Override
public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
        configurer.enable();
        configurer.enable("defaultServletName");
}
```

此时会注册一个默认的Handler：DefaultServletHttpRequestHandler，这个Handler也是用来处理静态文件的，它会尝试映射/。当DispatcherServelt映射/时（/ 和/ 是有区别的），并且没有找到合适的Handler来处理请求时，就会交给DefaultServletHttpRequestHandler 来处理。注意：这里的静态资源是放置在web根目录下，而非WEB-INF 下。
　　可能这里的描述有点不好懂（我自己也这么觉得），所以简单举个例子，例如：在webroot目录下有一个图片：1.png 我们知道Servelt规范中web根目录（webroot）下的文件可以直接访问的，但是由于DispatcherServlet配置了映射路径是：/ ，它几乎把所有的请求都拦截了，从而导致1.png 访问不到，这时注册一个DefaultServletHttpRequestHandler 就可以解决这个问题。其实可以理解为DispatcherServlet破坏了Servlet的一个特性（根目录下的文件可以直接访问），DefaultServletHttpRequestHandler是帮助回归这个特性的。

##### 2.5 configureViewResolvers：视图解析器

这个方法是用来配置视图解析器的，该方法的参数ViewResolverRegistry 是一个注册器，用来注册你想自定义的视图解析器等.

```java
/**
 * 配置请求视图映射
 * @return
 */
@Bean
public InternalResourceViewResolver resourceViewResolver()
{
	InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
	//请求视图文件的前缀地址
	internalResourceViewResolver.setPrefix("/WEB-INF/jsp/");
	//请求视图文件的后缀
	internalResourceViewResolver.setSuffix(".jsp");
	return internalResourceViewResolver;
}
 
/**
 * 视图配置
 * @param registry
 */
@Override
public void configureViewResolvers(ViewResolverRegistry registry) {
	super.configureViewResolvers(registry);
	registry.viewResolver(resourceViewResolver());
	/*registry.jsp("/WEB-INF/jsp/",".jsp");*/
}
```

##### 2.6 configureContentNegotiation：配置内容裁决的一些参数

##### 2.7 addCorsMappings：跨域

```java
@Override
public void addCorsMappings(CorsRegistry registry) {
    super.addCorsMappings(registry);
    registry.addMapping("/cors/**")
            .allowedHeaders("*")
            .allowedMethods("POST","GET")
            .allowedOrigins("*");
}
```

##### 2.8)configureMessageConverters：信息转换器

```java
 
/**
* 消息内容转换配置
 * 配置fastJson返回json转换
 * @param converters
 */
@Override
public void configureMessageConverters(List<HttpMessageConverter<?>> converters) {
    //调用父类的配置
    super.configureMessageConverters(converters);
    //创建fastJson消息转换器
    FastJsonHttpMessageConverter fastConverter = new FastJsonHttpMessageConverter();
    //创建配置类
    FastJsonConfig fastJsonConfig = new FastJsonConfig();
    //修改配置返回内容的过滤
    fastJsonConfig.setSerializerFeatures(
            SerializerFeature.DisableCircularReferenceDetect,
            SerializerFeature.WriteMapNullValue,
            SerializerFeature.WriteNullStringAsEmpty
    );
    fastConverter.setFastJsonConfig(fastJsonConfig);
    //将fastjson添加到视图消息转换器列表内
    converters.add(fastConverter);
 
}
```

## 二、Springboot默认的嵌入式的容器（tomcat,jetty,undertow)

![image-20201128101845795](/Users/yanjunshen/Library/Application Support/typora-user-images/image-20201128101845795.png)



### 1、如何定义和修改Servlet容器的相关配置

a、修改和server相关的配置(ServerProperties.class)

```pro
server.port=8081
server.contex-path=/crud
//通用Servlet容器设置
server.xxx
//Tomcat配置
server.tomcat.xxx
...

```

b、实现WebServerFactoryCustomizer的customize接口方法

```java
 //配置容器属性
    @Bean
    public WebServerFactoryCustomizer<ConfigurableWebServerFactory>  webServerFactoryWebServerFactoryCustomizer() {
        return new WebServerFactoryCustomizer<ConfigurableWebServerFactory>() {
            @Override
            public void customize(ConfigurableWebServerFactory factory) {
                factory.setPort(8085);
            }
        };
    }
```



### 2、配置嵌入式Servlet容器

支持的容器

* Tomcat
* Jetty(长连接)
* Undertow(不支持JSP)
* ReactiveWebServer

![image-20201128212352412](/Users/yanjunshen/Library/Application Support/typora-user-images/image-20201128212352412.png)



#### 2.1 替换servlet容器

只需要去除web里的tomcat,在引用其他容器的容器的jar包

```xm
<!-- 去掉web包里的tomcat,添加jetty依赖包-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-web</artifactId>
			<exclusions>
				<exclusion>
					<groupId>org.springframework.boot</groupId>
					<artifactId>spring-boot-starter-tomcat</artifactId>
				</exclusion>
			</exclusions>
		</dependency>
		<!-- 引入Jetty包-->
		<dependency>
			<groupId>org.springframework.boot</groupId>
			<artifactId>spring-boot-starter-jetty</artifactId>
		</dependency>
		
可以看到打印的日志
2020-11-28 21:30:23.701  INFO 4541 --- [           main] o.e.jetty.server.AbstractConnector       : Started ServerConnector@3624da92{HTTP/1.1, (http/1.1)}{0.0.0.0:8085}
2020-11-28 21:30:23.703  INFO 4541 --- [           main] o.s.b.web.embedded.jetty.JettyWebServer  : Jetty started on port(s) 8085 (http/1.1) with context path '/'
2020-11-28 21:30:23.707  INFO 4541 --- [           main] c.s.p.s.SpringbootDemoApplication        : Started SpringbootDemoApplication in 3.926 seconds (JVM running for 4.716)
```

#### 2.2、替换Servlet容器的原理

通过如下源码可以看到，通过@ConditionalOnClass来判断jar包里是否存在Tomcat.class，Server.class等Servlet容器的class来确定启动哪个容器

```java
@Configuration
@ConditionalOnWebApplication
@EnableConfigurationProperties(ServerProperties.class)
public class EmbeddedWebServerFactoryCustomizerAutoConfiguration {

	/**
	 * Nested configuration if Tomcat is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Tomcat.class, UpgradeProtocol.class })
	public static class TomcatWebServerFactoryCustomizerConfiguration {

		@Bean
		public TomcatWebServerFactoryCustomizer tomcatWebServerFactoryCustomizer(Environment environment,
				ServerProperties serverProperties) {
			return new TomcatWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

	/**
	 * Nested configuration if Jetty is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Server.class, Loader.class, WebAppContext.class })
	public static class JettyWebServerFactoryCustomizerConfiguration {

		@Bean
		public JettyWebServerFactoryCustomizer jettyWebServerFactoryCustomizer(Environment environment,
				ServerProperties serverProperties) {
			return new JettyWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

	/**
	 * Nested configuration if Undertow is being used.
	 */
	@Configuration
	@ConditionalOnClass({ Undertow.class, SslClientAuthMode.class })
	public static class UndertowWebServerFactoryCustomizerConfiguration {

		@Bean
		public UndertowWebServerFactoryCustomizer undertowWebServerFactoryCustomizer(Environment environment,
				ServerProperties serverProperties) {
			return new UndertowWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

	/**
	 * Nested configuration if Netty is being used.
	 */
	@Configuration
	@ConditionalOnClass(HttpServer.class)
	public static class NettyWebServerFactoryCustomizerConfiguration {

		@Bean
		public NettyWebServerFactoryCustomizer nettyWebServerFactoryCustomizer(Environment environment,
				ServerProperties serverProperties) {
			return new NettyWebServerFactoryCustomizer(environment, serverProperties);
		}

	}

}

```

#### 2.3 Servlet容器启动原理

什么时候创建嵌入式的Servlet容器工厂？待补充。。。。

### 3、注册Spring 三大组件Servlet、Filter、Listener

由于SpringBoot默认是以jar包的方式启动嵌入式的Servlet容器来启动SpringBoot的web应用，没有web.xml文件。所以如果注入servlet不能通过Spring的方式在xml里配置，SpringBoot提供了一下的方式注册自定义的Servlet,Filter,Listener的方式。

注册方式

* ServletRegistrationBean

  ```java
  @Bean
      public ServletRegistrationBean<MyServlet> myServlet() {
          ServletRegistrationBean<MyServlet> myServletServletRegistrationBean =
                  new ServletRegistrationBean<>(new MyServlet(),"/myservlet");
  
          return myServletServletRegistrationBean;
      }
  ```

  

* FilterRegistrationBean

  ```java
  @Bean
      public FilterRegistrationBean<MyFilter> myFilter() {
          FilterRegistrationBean<MyFilter> myFilterFilterRegistrationBean = new FilterRegistrationBean<>();
          myFilterFilterRegistrationBean.setFilter(new MyFilter());
          myFilterFilterRegistrationBean.setUrlPatterns(Arrays.asList("/filter","/hello"));
          return myFilterFilterRegistrationBean;
      }
  ```

  

* ServletListenerRegistrationBean

```
@Bean
    public ServletListenerRegistrationBean<MyListener> myListenerServletListenerRegistrationBean() {
        ServletListenerRegistrationBean<MyListener> servletListenerRegistrationBean =
                new ServletListenerRegistrationBean<>();
        servletListenerRegistrationBean.setListener(new MyListener());
        return  servletListenerRegistrationBean;

    }
```

SpringBoot帮我们自动配置SpringMVC的时候，自动注册SpringMVC的前端控制器 DispatcherServlet,参见代码

```java
//DispatcherServletAutoConfiguration.java
@Bean(name = DEFAULT_DISPATCHER_SERVLET_REGISTRATION_BEAN_NAME)
		@ConditionalOnBean(value = DispatcherServlet.class, name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
		public DispatcherServletRegistrationBean dispatcherServletRegistration(DispatcherServlet dispatcherServlet) {
			DispatcherServletRegistrationBean registration = new DispatcherServletRegistrationBean(dispatcherServlet,
					this.webMvcProperties.getServlet().getPath());
			registration.setName(DEFAULT_DISPATCHER_SERVLET_BEAN_NAME);
			registration.setLoadOnStartup(this.webMvcProperties.getServlet().getLoadOnStartup());
			if (this.multipartConfig != null) {
				registration.setMultipartConfig(this.multipartConfig);
			}
			return registration;
		}

@Bean(name = DEFAULT_DISPATCHER_SERVLET_BEAN_NAME)
		public DispatcherServlet dispatcherServlet() {
			DispatcherServlet dispatcherServlet = new DispatcherServlet();
			dispatcherServlet.setDispatchOptionsRequest(this.webMvcProperties.isDispatchOptionsRequest());
			dispatcherServlet.setDispatchTraceRequest(this.webMvcProperties.isDispatchTraceRequest());
			dispatcherServlet
					.setThrowExceptionIfNoHandlerFound(this.webMvcProperties.isThrowExceptionIfNoHandlerFound());
			dispatcherServlet.setEnableLoggingRequestDetails(this.httpProperties.isLogRequestDetails());
			return dispatcherServlet;
		}

```



# 三、时间监听机制

## 1、配置在META-INF/spring.factories下

* ApplicationContextInitializer

  ```java
  public class HelloApplicationContextInitializer implements ApplicationContextInitializer {
      @Override
      public void initialize(ConfigurableApplicationContext configurableApplicationContext) {
          System.out.println("HelloApplicationContextInitializer running " + configurableApplicationContext);
      }
  }
  ```

  ```properties
  org.springframework.context.ApplicationContextInitializer=\
  com.syj.prepare.springboot.listener.HelloApplicationContextInitializer
  
  
  ```

* SpringApplicationRunListener

  ```java
  public class HelloSpringApplicationRunListener implements SpringApplicationRunListener {
  
      public HelloSpringApplicationRunListener(SpringApplication application, String[] args){}
      @Override
      public void starting() {
          System.out.println("SpringApplicationListener ... starting....");
      }
  
      @Override
      public void environmentPrepared(ConfigurableEnvironment environment) {
          Object o = environment.getSystemProperties().get("os.name");
          System.out.println("SpringApplicationListener environmentPrepared " + o);
      }
  
      @Override
      public void contextPrepared(ConfigurableApplicationContext context) {
          System.out.println("SpringApplicationListener contextPrepared ");
      }
  
      @Override
      public void contextLoaded(ConfigurableApplicationContext context) {
          System.out.println("SpringApplicationListener contextLoaded ");
      }
  
      @Override
      public void started(ConfigurableApplicationContext context) {
          System.out.println("SpringApplicationListener started ");
      }
  
      @Override
      public void running(ConfigurableApplicationContext context) {
  
      }
  
      @Override
      public void failed(ConfigurableApplicationContext context, Throwable exception) {
  
      }
  }
  
  ```

  ```pro
  org.springframework.boot.SpringApplicationRunListener=\
  com.syj.prepare.springboot.listener.HelloSpringApplicationRunListener
  ```

  

  

## 2、只需要放在IOC容器里

* ApplicationRunner

  ```java
  @Component
  public class HelloApplicationRunner implements ApplicationRunner {
      @Override
      public void run(ApplicationArguments args) throws Exception {
          System.out.println("ApplicationRunner ...");
      }
  }
  ```

  

* CommandLineRunner

```java
@Component
public class HelloCommandLineRunner implements CommandLineRunner {
    @Override
    public void run(String... args) throws Exception {
        System.out.println("CommandLinrunner ..."+ Arrays.asList(args));
    }
}
```



