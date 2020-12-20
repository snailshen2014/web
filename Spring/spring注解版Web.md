# 注解版Web

## 1、三大组件

web三大组件，Servlet,Filter,Listener,可以通过注解来实现这些这些组件分别是：@WebServlet，@WebFilter，@WebLIsterner。传统的servlet开发需要开发servlet,并且在web.xml里配置servlet.

## 2、Shared libraries/runtimes pluggability

容器在启动应用的时候，会扫描当前应用的每一个jar包里META-INF/services下制定的实现类(ServletContainerInitializer),通过这种方式实现运行时插件，共享库的能力。

## 3、使用ServletContext注册web组件(Servlet,Filter,LIstener)

```java
public class UserServlet extends HttpServlet {
  @Override
  protected void doGet(HttpServletRequest req,HttpServletResponse response) {
    response.getWriter().write("tomcat!!!");
  }
}
public class UserFilter extends Filter {
  @Override
  public void doFilter(...) {
    ...
  }
}
public class UserListener extends ServletContextListner {
  @Override
  public void contextDestroyed(...) {
    ...
  }
}

//通过ServletContext的方法来注册
```



## 4、Servlet和Spring MVC整合

### 4.1官方做法

```java
//实现WebApplicationInitializer会被servlet容器自动加载
public class MyWebApplicationInitializer implements WebApplicationInitializer {

    @Override
    public void onStartup(ServletContext servletCxt) {

        // Load Spring web application configuration
        AnnotationConfigWebApplicationContext ac = new AnnotationConfigWebApplicationContext();
        ac.register(AppConfig.class);
        ac.refresh();

        // Create and register the DispatcherServlet
        DispatcherServlet servlet = new DispatcherServlet(ac);
        ServletRegistration.Dynamic registration = servletCxt.addServlet("app", servlet);
        registration.setLoadOnStartup(1);
        registration.addMapping("/app/*");
    }
}
```

上面的做法类似配置web.xml

```xml
<web-app>

    <listener>
        <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
    </listener>

    <context-param>
        <param-name>contextConfigLocation</param-name>
        <param-value>/WEB-INF/app-context.xml</param-value>
    </context-param>

    <servlet>
        <servlet-name>app</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <init-param>
            <param-name>contextConfigLocation</param-name>
            <param-value></param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>

    <servlet-mapping>
        <servlet-name>app</servlet-name>
        <url-pattern>/app/*</url-pattern>
    </servlet-mapping>

</web-app>
```



### 4.2、注解版

#### 4.2.1、context hierarchy

![mvc context hierarchy](https://docs.spring.io/spring-framework/docs/5.0.20.RELEASE/spring-framework-reference/images/mvc-context-hierarchy.png)



#### 4.2.2、原理

* Servlet容器在启动的时候，会扫描每个jar包下的META-INF/services/javax.servlet.ServletContainerInitializer

* 加载这个文件制定的类SpringServletContainterInitializer
* Servlet容器启动会加载感兴趣的WebApplicationInitializer接口下的所有组件
* AbstractContextLoaderInitializer:创建跟容器，createRootApplicationContext()
* AbstractDispactherServletInitializer

创建一个web的IOC容器，createServletApplicationContext()

创建DispatchServlet:createDispatcherServlet()，将创建的DispatcherServlet添加到ServletContext中

* AbstractAnnotationConfigDispatcherServletInitializer注解方式配置的DispatcherServlet

创建根容器:createRootApplicationContext()  ,getRootCOnfigClasser()传入配置类

创建web的ioc容器:createServletApplicationContext()获取配置类getServletConfigClasses()

总结：

以注解方式启动SpringMvc,继承AbstractAnnotationConfigDispatcherServletInitializer，实现抽象方法指定DispatcherServlet

  4.2.3、例子

```java
public class MyWebAppInitializer extends AbstractAnnotationConfigDispatcherServletInitializer {

    @Override
    protected Class<?>[] getRootConfigClasses() {
        return new Class<?>[] { RootConfig.class };
    }

    @Override
    protected Class<?>[] getServletConfigClasses() {
        return new Class<?>[] { App1Config.class };
    }

    @Override
    protected String[] getServletMappings() {
        return new String[] { "/app1/*" };
    }
}
```



参考：

https://docs.spring.io/spring-framework/docs/5.0.20.RELEASE/spring-framework-reference/web.html#mvc-servlet-context-hierarchy















