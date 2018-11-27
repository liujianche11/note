# spring组件注册

## @Configuration&@Bean

@Configuration  //告诉Spring这是一个配置类  === 配置文件

```java
@Configuration
public class MainConfig {
    //默认是通过方法名作为id
    //可以通过@Bean的value设置id
    @Bean("person")
    public Person person(){
        return new Person();
    }
}


 @Test
public void confignationTest(){
    AnnotationConfigApplicationContext annotationConfigApplicationContext = new AnnotationConfigApplicationContext(MainConfig.class);
    Person person = (Person) annotationConfigApplicationContext.getBean("person");
    System.out.println(person);
}
```

## @ComponentScan自动扫描

//@ComponentScan  value:指定要扫描的包
//excludeFilters = Filter[] ：指定扫描的时候按照什么规则排除那些组件
//includeFilters = Filter[] ：指定扫描的时候只需要包含哪些组件
//FilterType.ANNOTATION：按照注解
//FilterType.ASSIGNABLE_TYPE：按照给定的类型；
//FilterType.ASPECTJ：使用ASPECTJ表达式
//FilterType.REGEX：使用正则指定
//FilterType.CUSTOM：使用自定义规则

```java
@Configuration
@ComponentScan(value = "com.ljc.test", excludeFilters = {
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Controller.class)
})
//@ComponentScan value:指定要扫描的包
//excludeFilters = Filter[],指定扫描的时候按照什么规则排除组件
//includeFilters = Filter[],指定扫描的时候只包含哪些组建
//在java8以上@ComponentScan是一个重复注解@Repeatable,可以同时写好几个
public class MainConfig {

    @Bean
    public Person person() {
        return new Person();
    }
}
```

## @ComponentScans

```java
@Configuration
//@ComponentScan value:指定要扫描的包
//excludeFilters = Filter[],指定扫描的时候按照什么规则排除组件
//includeFilters = Filter[],指定扫描的时候只包含哪些组建
//使用includeFilters，需要设置useDefaultFilters = false
//在java8以上@ComponentScan是一个重复注解@Repeatable,可以同时写好几个
//如果不是jdk8 可以使用@ComponentScans设置多个@ComponentScan
@ComponentScans(
        value = {
                @ComponentScan(value = "com.ljc.test", includeFilters = {
                        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = Controller.class)
                }, useDefaultFilters = false)
        }
)
public class MainConfig {

    @Bean
    public Person person() {
        return new Person();
    }
}
```

## 自定义TypeFilter过来扫描规则

```java
public class MyFilter implements TypeFilter {

    @Override
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {

        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        if (annotationMetadata.hasAnnotation("org.springframework.stereotype.Controller")) {
            System.out.println("------------------------");
            return true;
        }
        return false;
    }
}

@Configuration
//@ComponentScan value:指定要扫描的包
//excludeFilters = Filter[],指定扫描的时候按照什么规则排除组件
//includeFilters = Filter[],指定扫描的时候只包含哪些组建
//使用includeFilters，需要设置useDefaultFilters = false
//在java8以上@ComponentScan是一个重复注解@Repeatable,可以同时写好几个
//如果不是jdk8 可以使用@ComponentScans设置多个@ComponentScan
@ComponentScans(
        @ComponentScan(
                value = "com.ljc.test", includeFilters = {
                @ComponentScan.Filter(type = FilterType.CUSTOM, classes = MyFilter.class)
        }, useDefaultFilters = false
        )
)
public class MainConfig {

    @Bean
    public Person person() {
        return new Person();
    }
}
```

## @Scope设置作用域

```java
@ComponentScans(
        @ComponentScan(
                value = "com.ljc.test", includeFilters = {
                @ComponentScan.Filter(type = FilterType.CUSTOM, classes = MyFilter.class)
        }, useDefaultFilters = false
        )
)
public class MainConfig {

    //prototype 调用getbean的时候创建对象
    //singleton ioc启动的时候就会创建对象
    // request/
    // session
    @Scope("prototype")
    @Bean
    public Person person() {
        return new Person();
    }
}
```

## @Lazy懒加载

```java
//prototype 调用getbean的时候创建对象
//singleton ioc启动的时候就会创建对象
// request/
// session
//@Scope("prototype")
// @Lazy 容器启动不创建实例，第一次使用bean时才会创建并初始化
@Lazy
@Bean
public Person person() {
    return new Person();
}
```

## @Conditional根据条件注册bean


![](D:\note\spring\conditional.png)

@conditional既可以加在方法上，也可以加在类上

```java
public class WindowsConditional implements Condition {

    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        Environment environment = context.getEnvironment();
        String property = environment.getProperty("os.name");
        if (property.contains("Windows")){
            return true;
        }

        return false;
    }
}

//类中组件统一设置。满足当前条件，这个类中配置的所有bean注册才能生效；
   @Conditional(WindowsConditional.class)
    @Bean("windows")
    public Person person01(){
        Person person = new Person();
        person.setName("windows");
        return person;

    }
```

## @Import导入组建

/**
	 * 给容器中注册组件；
	 * 1）、包扫描+组件标注注解（@Controller/@Service/@Repository/@Component）[自己写的类]
	 * 2）、@Bean[导入的第三方包里面的组件]
	 * 3）、@Import[快速给容器中导入一个组件]
	 * 		1）、@Import(要导入到容器中的组件)；容器中就会自动注册这个组件，id默认是全类名
	 * 		2）、ImportSelector:返回需要导入的组件的全类名数组；
	 * 		3）、ImportBeanDefinitionRegistrar:手动注册bean到容器中
	 * 4）、使用Spring提供的 FactoryBean（工厂Bean）;
	 * 		1）、默认获取到的是工厂bean调用getObject创建的对象
	 * 		2）、要获取工厂Bean本身，我们需要给id前面加一个&
	 * 			&colorFactoryBean
	 */

```
@Configuration
//@ComponentScan value:指定要扫描的包
//excludeFilters = Filter[],指定扫描的时候按照什么规则排除组件
//includeFilters = Filter[],指定扫描的时候只包含哪些组建
//使用includeFilters，需要设置useDefaultFilters = false
//在java8以上@ComponentScan是一个重复注解@Repeatable,可以同时写好几个
//如果不是jdk8 可以使用@ComponentScans设置多个@ComponentScan
@ComponentScans(
        @ComponentScan(
                value = "com.ljc.test", includeFilters = {
                @ComponentScan.Filter(type = FilterType.CUSTOM, classes = MyFilter.class)
        }, useDefaultFilters = false
        )
)
  /**
     * 容器中注册主件:
     * 1)，包扫描+主件标注注解
     * 2),@Bean
     * 3),@Import
     */
@Import(Color.class)//使用import引入
public class MainConfig {

```

### @Import使用ImportSelector

```
//使用importSelector
@Import({Color.class, MyImportSelector.class})


//自定义的MyImportSelector
public class MyImportSelector implements ImportSelector {
    @Override
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        return new String[]{"com.ljc.test.entity.Blue", "com.ljc.test.entity.Yellow"};
    }
}
```

### @Import使用ImportBeanDefinitionRegistrar

```java
@Import({Color.class, MyImportSelector.class, MyImportBeanDefinitionRegistrar.class})



/**
 * 可以手动注册bean
 */
public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
    @Override
    public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
        boolean b = registry.containsBeanDefinition("com.ljc.test.entity.Yellow");
        boolean b1 = registry.containsBeanDefinition("com.ljc.test.entity.Blue");
        if(b&&b1){
            RootBeanDefinition rootBeanDefinition = new RootBeanDefinition(Green.class);
            registry.registerBeanDefinition("green",rootBeanDefinition);
        }
    }
}
```

## 使用FactoryBean注册bean

```
public class ColorFactoryBean implements FactoryBean<Color> {
    //创建对象，添加到容器中
    @Override
    public Color getObject() throws Exception {
        System.out.println("colorFactoryBean getObject()......");
        return new Color();
    }

    @Override
    public Class<?> getObjectType() {
        return Color.class;
    }

    //返回是否單例
    @Override
    public boolean isSingleton() {
        return true;
    }
}



@Configuration
public class MainConfig {

	//使用factorybean的时候注入的是colorFactoryBean
	//但在使用applicationContext.getBean是时返回的是ColorFactoryBean.getObject()中的值
    @Bean
    public ColorFactoryBean colorFactoryBean() {
        return new ColorFactoryBean();
    }
}



@Test
    public void testFactoryBean() {
    //这里返回的是color
        Color colorFactoryBean = (Color) annotationConfigApplicationContext.getBean("colorFactoryBean");
        System.out.println(colorFactoryBean.getClass().getName());
    }
```