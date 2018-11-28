# spring赋值

## @Value赋值

```java
@Value("${name}")
private String name;
```

## @PropertySource引入配置文件

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Repeatable(PropertySources.class)
public @interface PropertySource
```

是一个可以重复的注解,就代表还有相对应的@PropertySources注解

```
@PropertySource(value = "classpath:application.properties")
@Configuration
public class MainConfigOfProperty {

    @Bean
    public Person person() {
        return new Person();
    }
}
```

## @Autowired&@Qualifier

```java
@Controller
public class BookController {

    @Autowired
    BookService bookService;
}
```

可以使用@Qualifier指定需要装配的组建的id，而不是使用属性名

```
@Controller
public class BookController {

    @Qualifier("bookService1")
    @Autowired
    BookService bookService;
}
```

也可以使用@Primary来指定首选的装配的bean，但它与@Qualifier是互斥的

```
@Primary
@Bean(name = "bookService1")
public BookService bookService(){
    return  new BookService();
}
```

## @Resource(JSR250)&@Inject(JSR330)

@Resource和@Inject,java规范里的，不支持spring自己的注解