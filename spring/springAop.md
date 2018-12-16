# spring AOP

1. 导入aop的依赖:spring-aspects

```java
//创建一个切面类
//所有的切入方法的JoinPoint joinPoint一定要是第一个参数
@Aspect
public class LogAspects {

    //切入点
    @Pointcut("execution(public * com.ljc.test.aop.Math.*(..))")
    public void pointCut() {

    }

    //切入方法
    @Before("pointCut()")
    public void logStart(JoinPoint joinPoint) {

        System.out.println(joinPoint.getSignature() + "方法开始运行.....args=" + Arrays.asList(joinPoint.getArgs()));
    }

    //切入方法
    //用全路径名可以使用外部定义的切面
    @After("com.ljc.test.aop.LogAspects.pointCut()")
    public void logEnd(JoinPoint joinPoint) {
        System.out.println("方法结束运行.....");
    }

    //切入方法
    @AfterReturning(value = "pointCut()", returning = "result")
    public void logReturn(JoinPoint joinPoint, Object result) {
        System.out.println(joinPoint.getSignature() + "方法正常返回........" + result);
    }

    //切入方法
    @AfterThrowing(value = "pointCut()", throwing = "exception")
    public void logException(JoinPoint joinPoint, Exception exception) {
        System.out.println(joinPoint.getSignature() + "方法发送异常........" + exception.getMessage());
    }
}
```

2. 业务逻辑类

   ```
   public class Math {
   
       public int div(int i, int j) {
           return i / j;
       }
   }
   ```

3. 容器配置类和开启aop

   ```
   //开启aop
   @EnableAspectJAutoProxy
   @Configuration
   public class MianConfigOfAOP {
   
       @Bean
       public Math math() {
           return new Math();
       }
   
       @Bean
       public LogAspects LogAspects() {
           return new LogAspects();
       }
   
   }
   ```