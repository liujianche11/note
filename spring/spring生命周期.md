# spring生命周期

## @Bean指定初始化方法和销毁方法

在@Bean使用initMethod指定spring注入创建对象的初始化方法，@destroyMethod指定spring容器销毁时调用的bean的销毁方法

```
public class Car {

    public Car() {
        System.out.println("car constructor.......");
    }

    public void init() {
        System.out.println("car init........");
    }

    public void destory() {
        System.out.println("car destory........");
    }
}


@Configuration
public class MainCofigOfLifeCycle {

    @Bean(initMethod = "init",destroyMethod = "destory")
    public Car car() {
        return new Car();
    }
}

 @Test
    public void test(){
        Car car = applicationContext.getBean(Car.class);
        applicationContext.close();
    }
```

## 使用InitializingBean和DisposableBean接口

```java
//实现InitializingBean的afterPropertiesSet
//实现DisposableBean的destroy
@Component
public class Cat implements InitializingBean, DisposableBean {
    @Override
    public void destroy() throws Exception {
        System.out.println("------------destroy");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("------------afterPropertiesSet");
    }
}
```

## 使用JSR250定义的@PostConstruct、@PreDestroy

```java
@Component
public class Dog {
    public Dog() {
        System.out.println("Dog constructor...");
    }

    /**
     * 对象创建并赋值之后调用
     */
    @PostConstruct
    public void init() {
        System.out.println("Dog init......");
    }

    /**
     * 容器移除对象之前
     */
    @PreDestroy
    public void destory() {
        System.out.println("Dog destory.....");
    }
}
```

## 使用BeanPostProcessor

/**
 * bean的生命周期：
 * bean创建---初始化----销毁的过程
 * 容器管理bean的生命周期；
 * 我们可以自定义初始化和销毁方法；容器在bean进行到当前生命周期的时候来调用我们自定义的初始化和销毁方法
 * 
 * 构造（对象创建）
 * 单实例：在容器启动的时候创建对象
 * 多实例：在每次获取的时候创建对象\
 * 
 * BeanPostProcessor.postProcessBeforeInitialization
 * 初始化：
 * 对象创建完成，并赋值好，调用初始化方法。。。
 * BeanPostProcessor.postProcessAfterInitialization
 * 销毁：
 * 单实例：容器关闭的时候
 * 多实例：容器不会管理这个bean；容器不会调用销毁方法；
 * 
 * 
 * 遍历得到容器中所有的BeanPostProcessor；挨个执行beforeInitialization，
 * 一但返回null，跳出for循环，不会执行后面的BeanPostProcessor.postProcessorsBeforeInitialization
 * 
 * BeanPostProcessor原理
 * populateBean(beanName, mbd, instanceWrapper);给bean进行属性赋值
 * initializeBean
 * {
 * applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
 * invokeInitMethods(beanName, wrappedBean, mbd);执行自定义初始化
 * applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    *}
 * 
 * 
 * 
 * 1）、指定初始化和销毁方法；
 * 通过@Bean指定init-method和destroy-method；
 * 2）、通过让Bean实现InitializingBean（定义初始化逻辑），
 * DisposableBean（定义销毁逻辑）;
 * 3）、可以使用JSR250；
 * @PostConstruct：在bean创建完成并且属性赋值完成；来执行初始化方法
 * @PreDestroy：在容器销毁bean之前通知我们进行清理工作
 * 4）、BeanPostProcessor【interface】：bean的后置处理器；
 * 在bean初始化前后进行一些处理工作；
 * postProcessBeforeInitialization:在初始化之前工作
 * postProcessAfterInitialization:在初始化之后工作
 * 
 * Spring底层对 BeanPostProcessor 的使用；
 * bean赋值，注入其他组件，@Autowired，生命周期注解功能，@Async,xxx BeanPostProcessor;
 * 
 * @author lfy
    *
     */





BeanPostProcessor:bean的后置处理器，在bean初始化前后进行一些处理

```
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessBeforeInitialization......."+beanName);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("postProcessAfterInitialization......."+beanName);
        return bean;
    }
}
```

//使用了这个以后，会在每个加载的bean的前后执行postProcessBeforeInitialization和postProcessAfterInitialization