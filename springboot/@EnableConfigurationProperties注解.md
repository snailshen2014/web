# @EnableConfigurationProperties注解

## 一、作用

@EnableConfigurationProperties注解的作用是，使使用@ConfigurationProperties注解的类生效，如果一个配置类只配置@ConfigurationProperties注解，而没有使用@Component那么在IOC容器中是获取不到properties配置文件转换的bean,当`@EnableConfigurationProperties`注解应用到你的`@Configuration`时， 任何被`@ConfigurationProperties`注解的beans将自动被Environment属性配置。 这种风格的配置特别适合与SpringApplication的外部YAML配置进行配合使用。

主要作用:外部配置(properties,yml,配置中心配置)->@ConfigurationProperteis配置类->@EnableConfigurationPropertis配置类初始化依赖外部属性的类

## 二、使用方法

一般我们通过设置属性配置类，然后通过配置类动态初始化属性配置来进行bean的注入。举个例子

```java
//属性配置类
@ConfigurationProperties(prefix = TestProperties.PROPERTY_PREFIX)
public class TestProperties {
    protected static final String PROPERTY_PREFIX = "my.lock";
    //lock type
    private LockType lockType = LockType.REDIS;

    //zk properties
    private String lockPath;
    private String zkAddress;
  
    public String getZkAddress() {
        return zkAddress;
    }

    public void setZkAddress(String zkAddress) {
        this.zkAddress = zkAddress;
    }

    public LockType getLockType() {
        return lockType;
    }

    public void setLockType(LockType lockType) {
        this.lockType = lockType;
    }

    public String getLockPath() {
        return lockPath;
    }

    public void setLockPath(String lockPath) {
        this.lockPath = lockPath;
    }
}

//外部配置文件application.yml
my:
    lock:
        lockType: ZK
        lockPath: /lock
        zkAddress: localhost:2181
          
         
          
//bean自动配置类
@Configuration
@EnableConfigurationProperties(TestProperties.class)
public class LockAutoConfiguration {

    @Bean
    @ConditionalOnMissingBean(KooLock.class)
    @ConditionalOnProperty(name="my.lock.lockType",havingValue = "ZK")
    public KooLock kooLockZk(TestProperties properties) {
  
        properties.determineProperties();
        return new  	   LockZkImpl(properties.getLockPath(),properties.getZkAddress(),properties.getMaxWaitMillis(),
             properties.getAppName());
    }

   
}      
```

经过上述的配置，在resource/META-INF/spring.factories里添加

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.snail.autoconfigure.LockAutoConfiguration
```

后springboot在启动时会自动配置LockAutoConfiguration，从而实现注入TestProperties自动装载，并把配置映射到TestProperties bean上，在创建Bean时使用这些属性。

## 三、原理

* @EnableConfigurationProperties注解里通过@Import(EnableConfigurationPropertiesImportSelector.class)向容器中注册了两个bean,ConfigurationPropertiesBeanRegistrar和ConfigurationPropertiesBindingPostProcessorRegistrar

```java
class EnableConfigurationPropertiesImportSelector implements ImportSelector {

	private static final String[] IMPORTS = { ConfigurationPropertiesBeanRegistrar.class.getName(),
			ConfigurationPropertiesBindingPostProcessorRegistrar.class.getName() };

	@Override
	public String[] selectImports(AnnotationMetadata metadata) {
		return IMPORTS;
	}
  /...
}
```



* 其中ConfigurationPropertiesBeanRegistrar的方法registerBeanDefinitions里会拿到@EnableConfigurationProperties的value值，并注册成bean,即例子的@EnableConfigurationProperties(TestProperties.class)，会把TestProperties.class注册为bean,主要代码

```java
public static class ConfigurationPropertiesBeanRegistrar implements ImportBeanDefinitionRegistrar {

		@Override
		public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
			getTypes(metadata).forEach((type) -> register(registry, (ConfigurableListableBeanFactory) registry, type));
		}

		private List<Class<?>> getTypes(AnnotationMetadata metadata) {
			MultiValueMap<String, Object> attributes = metadata
					.getAllAnnotationAttributes(EnableConfigurationProperties.class.getName(), false);
			return collectClasses((attributes != null) ? attributes.get("value") : Collections.emptyList());
		}
  /...
}
```

* 属性配置bean注册到容器了，剩下就是外部的属性值绑定到配置文件类了，即例子中把yml的配置绑定到TestProperties的属性中。这个就是ConfigurationPropertiesBindingPostProcessorRegistrar这个bean的作用，下面分析一下。
* ConfigurationPropertiesBindingPostProcessorRegistrar里注册了两个bean,分别是ConfigurationPropertiesBindingPostProcessor和ConfigurationBeanFactoryMetadata，代码如下：

```java
public class ConfigurationPropertiesBindingPostProcessorRegistrar implements ImportBeanDefinitionRegistrar {

	@Override
	public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
		if (!registry.containsBeanDefinition(ConfigurationPropertiesBindingPostProcessor.BEAN_NAME)) {
			registerConfigurationPropertiesBindingPostProcessor(registry);
			registerConfigurationBeanFactoryMetadata(registry);
		}
	}

	private void registerConfigurationPropertiesBindingPostProcessor(BeanDefinitionRegistry registry) {
		GenericBeanDefinition definition = new GenericBeanDefinition();
		definition.setBeanClass(ConfigurationPropertiesBindingPostProcessor.class);
		definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(ConfigurationPropertiesBindingPostProcessor.BEAN_NAME, definition);

	}

	private void registerConfigurationBeanFactoryMetadata(BeanDefinitionRegistry registry) {
		GenericBeanDefinition definition = new GenericBeanDefinition();
		definition.setBeanClass(ConfigurationBeanFactoryMetadata.class);
		definition.setRole(BeanDefinition.ROLE_INFRASTRUCTURE);
		registry.registerBeanDefinition(ConfigurationBeanFactoryMetadata.BEAN_NAME, definition);
	}

}
```



* 其中ConfigurationPropertiesBindingPostProcessor负责把属性的值配置到属性的配置类中TestProperties

这个类实现了ApplicationContextAware接口可以拿到IOC容器,通过afterPropertiesSet()接口创建了ConfigurationPropertiesBinder，在postProcessBeforeInitialization（）里对属性值进行了绑定。关键代码

```java
public class ConfigurationPropertiesBindingPostProcessor
		implements BeanPostProcessor, PriorityOrdered, ApplicationContextAware, InitializingBean {
 @Override
	public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
		this.applicationContext = applicationContext;
	}

	@Override
	public void afterPropertiesSet() throws Exception {
		// We can't use constructor injection of the application context because
		// it causes eager factory bean initialization
		this.beanFactoryMetadata = this.applicationContext.getBean(ConfigurationBeanFactoryMetadata.BEAN_NAME,
				ConfigurationBeanFactoryMetadata.class);
		this.configurationPropertiesBinder = new ConfigurationPropertiesBinder(this.applicationContext,
				VALIDATOR_BEAN_NAME);
	}
  @Override
	public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		ConfigurationProperties annotation = getAnnotation(bean, beanName, ConfigurationProperties.class);
		if (annotation != null) {
			bind(bean, beanName, annotation);
		}
		return bean;
	}
}
```

