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