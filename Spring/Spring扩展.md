#                            Spring扩展

## 一、BeanPostProcessor后置处理器

BeanPostProcessor:bean后置处理器,bean创建对象初始化前后进行拦截工作，BeanFactoryPostProcessor:beanFactory的后置处理器,Bean的类型信息已经加载了，但是没有创建实例时进行拦截。

### 1、BeanFactoryPostProcessor实现

```java
1.IOC容器创建
2.在refresh中调用invokeBeanFactoryPostProcessors->PostProcessorRegistrationDelegate.invokeBeanFactoryPostProcessors()
 在invokeBeanFactoryPostProcessors()方法中，找到BeanFactoryPostProcessor
  String[] postProcessorNames =
              beanFactory.getBeanNamesForType(BeanFactoryPostProcessor.class, true, false);
 
 * //先调用有优先级的，排序的等等，最后通过invokeBeanFactoryPostProcessors循环调用默认的BeanFactoryPostProcessor
      private static void invokeBeanFactoryPostProcessors(
         Collection<? extends BeanFactoryPostProcessor> postProcessors, ConfigurableListableBeanFactory beanFactory) {
 
       for (BeanFactoryPostProcessor postProcessor : postProcessors) {
           postProcessor.postProcessBeanFactory(beanFactory);
                 }
 
 
```

### 2、BeanDefinitionRegistryPostProcessor

BeanDefinitionRegistryPostProcessor,是BeanFactoryPostProcessor的子接口

 * 在所有bean信息将要被加载，初始化完成，bean的实例还未被创建,
 * 他的执行顺序优先于BeanFactoryPostProcessor,z



## 二、事件ApplicationListener

ApplicationListener:监听容器中事件的事件(ApplicationEvent)，回调通知

### 1、自定义事件：

 * 写一个监听器来监听某个事件，(ApplicationEvent及其子类）
 * 把监听器加入到容器
 * 通过容器发布事件，监听类就会收到回调通知
 * 发布自定义事件

```java
AnnotationConfigApplicationContext annotationConfigApplicationContext =

new AnnotationConfigApplicationContext(ExtConfig.class);

//发送自定义事件

annotationConfigApplicationContext.publishEvent(new ApplicationEvent(new String("我的事件！！！")) {

@Override

public Object getSource() {

return super.getSource();

}

});
```



### 2、原理：

 * ,容器创建init方法里调用refresh方法，通过initApplicationEventMulticaster方法创建的事件发送多波器
 * 在refresh方法里注册给多波器listener registerListeners();
     容器创建时，在refresh里会 ,容器刷新refresh->finishReresh->publishEvent()
 * 在publishEvent方法里，通过获取事件多波器进行事件发送
 * getApplicationEventMulticaster().multicastEvent(applicationEvent, eventType);

###  3、通过注解监听事件

#### 1、@EventListener

```java
 @Service
 public class UserService {
 @EventListener
 public void listen(ApplicationEvent event) {
 System.out.println("user service event:" + event);
 }
 }
```



####  2.原理

 利用处理器EventListenerMethodProcessor来解析这个注解，实现了SmartInitializingSingleton, ApplicationContextAware,BeanFactoryPostProcessor这些接口，主要看SmartInitializingSingleton

 refresh->finishBeanFactoryInitialization-> {
 // Instantiate all remaining (non-lazy-init) singletons.
 beanFactory.preInstantiateSingletons();
 } -> smartSingleton.afterSingletonsInstantiated()/EventListenerMethodProcessor里的方法


