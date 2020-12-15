#                    AOP

# 一、基本用法

AOP:程序在运行期间能够将某段代码切入到指定方法指定位置进行运行
 * 导入AOP依赖，spring-aspects

 * 定义一个业务逻辑类(MathCalculator),在方法运行前、运行后、运行后、运行异常打印日志

 * 定义一个日志切面类(LogAspects),切面类里的方法需要动态感知MathCalculator.div的运行

    ```pr
    通知方法：前置通知(@Before) ：logStart在目标方法(div)运行之前运行
    
    后置通知(@After)：logEnd在目标方法运行之后运行
    
    返回通知(@AfterReturning)：logReturn在目标方法正常返回之后运行
    
    异常通知(@AfterThrowing)：logException,在目标方法运行异常以后运行
    
    环绕通知(@Around)：动态代理，手动推进目标方法运行(joinPoint.proceed())
    ```

    

 * 给切面类的目标方法标注何时何地运行（通知注解）

 * 将切面类和业务逻辑类（目标方法所在类）都加入到容器中

 * 告诉Spring哪个类是切面类，给切面类添加注解@Aspect

 * 给配置勒种添加@EnableAspectJAutoProxy，开启aspect自动代理

# 二、例子

```java
//切面类
@Aspect
public class LogAspects {

    //抽取公共的切入点表达式
    @Pointcut("execution(public int com.snail.aop.MathCalculator.*(..))")
    public void pointCut(){}
    //在方法执行之前执行
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint) {
        Object[] args = joinPoint.getArgs();
        System.out.println(joinPoint.getSignature().getName()+"除法运行，，参数列表{}"+ args);
    }
    @After("pointCut()")
    public void logEnd(JoinPoint joinPoint) {
        System.out.println(joinPoint.getSignature().getName()+" 除法结束");
    }
    @AfterReturning(value = "pointCut()",returning = "result")
    public void logReturn(JoinPoint joinPoint,Object result) {
        System.out.println(joinPoint.getSignature().getName() + "除法正常返回，运行结果{}"+ result);
    }
    @AfterThrowing(value = "pointCut()",throwing = "exception")
    public void logException(JoinPoint joinPoint,Exception exception) {
        System.out.println(joinPoint.getSignature().getName() + "除法异常！"+ exception);
    }
}


//业务类
public class MathCalculator {
    public int div(int i,int j) {
        System.out.println("div 运行！！！");
        return i/j;
    }
}

//AOP配置类
@EnableAspectJAutoProxy
@Configuration
public class MainConfigOfAop {
    @Bean
    public MathCalculator mathCalculator() {
        return new MathCalculator();
    }
    @Bean
    public LogAspects logAspects() {
        return new LogAspects();
    }

}

```



## 三、基本原理

1.利用@EnableAspectJAutoProxy开启AOP功能，他会给容器注册一个组件AnnotationAwareAspectJAutoProxyCreator,它是一个后置处理器
2.容器在创建时
refresh()->registerBeanPostProcessors()注册后置处理器，创建AnnotationAwareAspectJAutoProxyCreator
finishBeanFractoryInitialization()创建剩下的单实例bean
 *      创建业务逻辑组件和切面组件
 *      AnnotationAwareAspectJAutoProxyCreator拦截组件的创建过程
 *      组件创建完成后，判断组件是否需要增强，
 *      是：切面的通知方法，包装成(Adivsor),给业务逻辑组件创建一个代理对象(cglib)
 执行目标方法，代理对象执行目标方法，通过CglibAopProxy.intercept()进行方法拦截
 *  得到目标方法的拦截器链（增强器包装成拦截器MethodInterceptor)
 *  利用拦截器的链式机制，依次进去每一个拦截器进行执行
 *  效果：前置通知->目标方法执行->后续通知-> 返回通知/异常通知