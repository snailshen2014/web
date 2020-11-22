###

### 目录

### 1、 @import注解须知

### 2、@import的三种用法

* 直接填写classs数组
* importSelector方式
* importBeanDefinitionRegistrar方式



### 正文

1、@import注解须知

@Import只能用在类上，@Import通过快速导入的方式实现把实例加入到spring的IOC容器中。

2、三种要主要用法

* 直接填写class数组

  ```java
  public class MyService1 {
      public void service1() {
          System.out.println("Service1 ....");
      }
  }
  public class MyService2 {
      public void Service2() {
          System.out.println("Service2...");
      }
  }
  @Service
  @Import({MyService1.class , MyService2.class })
  public class MyService {
      @Autowired
      MyService1 myService1;
      @Autowired
      MyService2 myService2;
      public void service() {
          myService1.service1();
          myService2.Service2();
      }
  }
  
  ```

  通过import可以把MyService1,MyService2注入到IOC容器中。

  

  * #### ImportSelector方式

    这种方式的前提就是一个类要实现ImportSelector接口，假如我要用这种方法，目标对象是Myclass这个类，分析具体如下：创建Myclass类并实现ImportSelector接口

    ```java
    public class MyClass implements ImportSelector {
        @Override
        public String[] selectImports(AnnotationMetadata annotationMetadata) {
            return new String[0];
        }
    }
    ```

    分析实现接口的selectImports方法中的：

    1、返回值： 就是我们实际上要导入到容器中的组件全类名【**重点** 】

    2、参数： AnnotationMetadata表示当前被@Import注解给标注的所有注解信息【不是重点】

    > 需要注意的是selectImports方法可以返回空数组但是不能返回null，否则会报空指针异常！

    以上分析完毕之后，具体用法步骤如下：

    第一步：创建MyService3Selector类并实现ImportSelector接口，这里用于演示就添加一个全类名给其返回值

    ```javascript
    public class MyService3Selector implements ImportSelector {
        @Override
        public String[] selectImports(AnnotationMetadata annotationMetadata) {
            return new String[]{"com.syj.prepare.springboot.services.MyService3"};
        }
    }
    
    public class MyService3 {
        public String say() {
            System.out.println("MyService3...");
            return "MyService3";
        }
    }
    
    ```

    第二步：编写TestImportSelector 类，并标注上使用ImportSelector方式的Myclass类

    ```javascript
    @Compont
    @Import({MyService3Selector.class})
    public class TestImportSelector {
        @Autowired
        MyService3 myService3;
        public String say() {
            return myService3.say();
        }
    
    }
    ```

  ​      测试

     ```java
  @Autowired
  	TestImportSelector testImportSelector;
  	@Test
  	public void testSelector() {
  		testImportSelector.say();
  	}
     ```

* ImportBeanDefinitionRegister方式

  同样是一个接口，类似于第二种ImportSelector用法，相似度80%，只不过这种用法比较自定义化注册，具体如下：

  第一步：创建MyService4Registrar类并实现ImportBeanDefinitionRegistrar接口

  ```java
  public class MyService4Registrar implements ImportBeanDefinitionRegistrar {
      @Override
      public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
          
      }
  }
  ```

  参数分析：

  第一个参数：annotationMetadata 和之前的ImportSelector参数一样都是表示当前被@Import注解给标注的所有注解信息

  第二个参数表示用于注册定义一个bean

  第二步：编写代码，自定义注册bean,注册一个MyService4

  ```java
  public class MyService4Registrar implements ImportBeanDefinitionRegistrar {
      @Override
      public void registerBeanDefinitions(AnnotationMetadata annotationMetadata, BeanDefinitionRegistry beanDefinitionRegistry) {
          //指定bean定义信息（包括bean的类型、作用域...）
          RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(MyService4.class);
          //注册一个bean指定bean名字（id）
          beanDefinitionRegistry.registerBeanDefinition("myservice4",rootBeanDefinition);
      }
  }
  ```

  第三步：编写TestDemo 类，并标注上使用ImportBeanDefinitionRegistrar方式的MyService4Registrar类

  ```java
  @Component
  @Import({MyService4Registrar.class})
  public class RegistrarClass {
      @Autowired
      MyService4 myService4;
      public void say() {
          myService4.say();
      }
  
    
    //在springboot单元测试里，测试通过Registrar注册的MyService4
    @Autowired
  	RegistrarClass registrarClass;
  	@Test
  	public void testRegistrar() {
  		registrarClass.say();
  	}
  ```

  

### @Import注解的三种使用方式总结

> 第一种用法：`@Import`（{ 要导入的容器中的组件 } ）：容器会自动注册这个组件，**id默认是全类名**
>
>  
>
> 第二种用法：`ImportSelector`：返回需要导入的组件的全类名数组，springboot底层用的特别多【**重点** 】
>
>  
>
> 第三种用法：`ImportBeanDefinitionRegistrar`：手动注册bean到容器

**以上三种用法方式皆可混合在一个@Import中使用，特别注意第一种和第二种都是以全类名的方式注册，而第三中可自定义方式。**

@Import注解本身在springboot中用的很多，特别是其中的第二种用法ImportSelector方式在springboot中使用的特别多，尤其要掌握！