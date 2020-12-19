# 										Spring容器创建过程

# 1、入口

本文测试案例通过创建AnnotationConfigApplicationContext这个实例,在构造函数里传入配置类的方式在拉起容器的

ExtConfig配置类如下：

```java
@ComponentScan("com.snail.ext")
@Configuration
public class ExtConfig {
    @Bean
    public Blue blue() {
        System.out.println("创建blue...");
        return new Blue();
    }
}
```



```java
AnnotationConfigApplicationContext annotationConfigApplicationContext =
                 new AnnotationConfigApplicationContext(ExtConfig.class);
```

在AnnotationConfigApplicationContext的构造方法里完成了容器的创建工作。

```java
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
		this();
		register(componentClasses);
		refresh();
	}
```

# 2、this

这个方法主要是调用父类的构造函数(GenericApplicationContext)初始化beanFactory
  this.beanFactory = new DefaultListableBeanFactory();

# 3、register

register(componentClasses);//主要是把配置类ExtConfig.class注册到容器的BeanDefinitionRegistry

# 4、refresh



```java
2.1、prepareRefresh

        initPropertySources（）初始化一些属性设置，子类自定义个性化的属性设置方式
        getEnvironment().validateRequiredProperties()校验属性的合法
        this.earlyApplicationEvents ，this.applicationListeners 构造
        //设置beanFactory
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory()
        prepareBeanFactory(beanFactory);
        2.2、
        postProcessBeanFactory//BeanFactory准备工作完成后进行后置处理工作，子类通过重写这个方法在，做一些设置
        invokeBeanFactoryPostProcessors//执行BeanFactoryPostProcessor
        2.3
        // Register bean processors that intercept bean creation.
        registerBeanPostProcessors(beanFactory);

        2.4initMessageSource  //国际化，等
        2.5initApplicationEventMulticaster 初始化事件发送多波器
        2.6onRefresh 子类重写给容器注入组件等
        2.7 registerListeners 从容器中拿到所有的applicationListener组件注册到事件分派多波器中
        2.8 finishBeanFactoryInitialization //初始化所有的剩下的单实例bean
        获取所有的ban定义信心，通过getBean创建bean
        主要方法preInstantiateSingletons（）
        {
            // Trigger initialization of all non-lazy singleton beans...
                for (String beanName : beanNames) {
                    RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
                    if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
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
        }

        实例化完毕后，检查所有的beans会否是SmartInitializingSingleton接口，如果是就来执行afterSingletonsInstantiated方法


        2.9：finishRefresh 完成刷新 初始化生命周期处理器、发送事件等


        总结
        1.Spring容器在启动时，贤惠保存所有注册进来的Bean定义信息
        1)xml bean
        2)注解bean @Service,@Component @Bean 等

        2.Spring容器会在合适的时候创建这些bean
        1)、用到这些bean的时候，利用getBean创建bean,创建好后保存在容器中
        2）、统一创建剩下的bean,finishBeanFactoryInitialization()
        3.后置处理器(BeanPostProcessor)，每一个bean创建完成都会使用各种后置处理器进行处理,增强bean的功能或者创建代理
            AutowiredAnnotationBeanPostProcessor处理@AutoWired 注解自动注入
            AnnotationAwareAspectJAutoProxyCreator处理AOP
            xxx

        3、事件驱动模型
        ApplicationListener:事件监听

```

