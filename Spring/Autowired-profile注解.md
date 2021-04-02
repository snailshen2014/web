#                   @Autowired & @Profile

## 一、@Autowired

​	自动装配，Spring利用依赖注入(DI),完成IOC容器中各个组件的自动赋值，下面解释这个注解的几条注意事项。

### 1、注意事项

* 默认优先按照类型去容器中找到对应的组件：applicationContext.getBean(xxx.class)

* 如果找到多个相同的组件，再将属性的名称作为组件的id去容器中找applicationContext.getBean("bookDao")

* @Qualifier 可以明确指定要装配的组件的id,而不是根据属性名

  ```java
  public class BookService {
       @Qualifier("bookDao")
       @Autowired
       private BookDao bookDao2;
  
       public void print() {
          System.out.println(bookDao2);
      }
  }
  ```

* 自动装配一定要将属性赋值好，必须要将组件定义好,否则组件依赖至少需要找到一个

*  @Autowired(required = false)，可以去掉上面的规则的限制，如果有则装配组件。

* @Primary:让Spring自动装配的时候，默认使用首选的组件

  ```java
      //指定是首选的组件
      @Primary
      @Bean("bookDao2")
      public BookDao bookDao() {
          BookDao bookDao = new BookDao();
          bookDao.setLable("2");
          return bookDao;
      }
  ```

  



### 2、Spring支持的Java标准的组件注入注解

Spring还支持使用@Resource(JSR250)和@Inject(JSR330)[java规范注解]
* @Resource:可以和@Autowired一样实现自动装配功能，默认按照组件名称装配
* 不支持@primary功能，可以根据name属性装配多个组件中的一个
* @Inject，需要安装依赖javax.inject的包和@Autowired功能一样，没有required=false选项

### 3、原理

AutowiredAnnotationBeanPostProcessor,后置处理器来处理这些注解来为组件赋值。

### 4、标注的位置

@Autowired可以标注在构造器、属性和方法上。

* 标注在方法位置,@Bean+方法参数，参数的值也从容器中获取，默认不写都能自动装配

*  标注在构造器上，如果构造器只有一个参数，那么@autowired可以省略
*  标注在参数位置

例如：

```java
//默认加载ioc容器中的组件，容器会调用无参数构造器创建对象，再进行初始化赋值等操作
@Component
public class Boss {
//    @Autowired
    public Boss(/*@Autowired*/ Car car) {
        this.car = car;
        System.out.println(car);
    }
//    @Autowired
    private Car car;

    public Car getCar() {
        return car;
    }
    //标注在方法上，Spring容器常见对象时，就会调用方法，完成变量赋值
    //方法中使用的参数，如果是自定义类型会从IOC容器中获取
//    @Autowired
    public void setCar(Car car) {
        this.car = car;
    }

    @Override
    public String toString() {
        return "Boss{" +
                "car=" + car +
                '}';
    }
}
```



### 5、如何获取Spring底层的组件

使用Spring容器底层的一些组件(ApplicationContext,BeanFactory)，可以通过实现xxxAware接口来获取,在创建

对象时会回调接口的方法，传入给对象spring底层的相关组件。实现原理是通过xxxAwareProcessor的后置处理器来实现的，比如:ApplicationContextAware->ApplicationContextAwareProcessor

例如：

```java
@Component
public class Red implements ApplicationContextAware, BeanNameAware, EmbeddedValueResolverAware {
    private ApplicationContext applicationContext;

    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        System.out.println("传入的IOC:"+ applicationContext);
        this.applicationContext = applicationContext;
    }

    public void setBeanName(String name) {
        System.out.println("当前Bean的名字："+name);
    }

    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        System.out.println(resolver.resolveStringValue("当前操作系统${os.name}"));
    }
}

```



## 二、@Profile

Spring为我们提供的可以根据当前环境，动态激活和切换组件的功能,比如：切换工程的环境，不同的环境注册不通的组件，比如：测试、开发、生成环境注册不同的数据源
官方解释：
```
Spring Profiles provide a way to segregate parts of your application configuration and make it only available in certain environments. 

Any @Component or @Configuration can be marked with @Profile to limit when it is loaded:

@Configuration
@Profile("production")
public class ProductionConfiguration {

    // ...

}
```

* 标注在配置类上，复合条件的组件和没有@Profile标注的组件都会注册

* 激活方法：

  1.通过运行时参数-Dspring.profiles.active=xxx

  2.通过容器获取Environment来设定,annotationConfigApplicationContext.getEnvironment().setActiveProfiles("dev");

例如：

```java
@Configuration
public class MainConfigOfProfile {

    @Profile("test")
    @Bean
    public DataSource dataSourceTest() {
        return new DataSource();
    }
    @Profile("dev")
    @Bean
    public DataSource dataSourceDev() {
        return new DataSource();
    }
    @Profile("pro")
    @Bean
    public DataSource dataSourcePro() {
        return new DataSource();
    }
}

```



