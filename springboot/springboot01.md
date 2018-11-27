# springboot快速开始

## maven配置

```xml
<!--需要继承spring-boot-starter-parent-->
<!-- Inherit defaults from Spring Boot -->
<parent>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-parent</artifactId>
  <version>2.1.0.RELEASE</version>
</parent>

<!--引入web模块-->
<!-- Add typical dependencies for a web application -->
<dependencies>
  <dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
  </dependency>
</dependencies>

<!--打包成一个可运行的jar-->
<!-- Package as an executable jar -->
<build>
  <plugins>
    <plugin>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-maven-plugin</artifactId>
    </plugin>
  </plugins>
</build>
```

spring-boot-starter-parent继承了spring-boot-dependencies
spring-boot-dependencies里面控制了各个引入jar包的版本

spring-boot-starter-web引入了springmvc的web部分，各个starter(启动器都是引入了各自相关的jar)

`@SpringBootApplication`标注springboot的主程序





```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Inherited
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(
    excludeFilters = {@Filter(
    type = FilterType.CUSTOM,
    classes = {TypeExcludeFilter.class}
), @Filter(
    type = FilterType.CUSTOM,
    classes = {AutoConfigurationExcludeFilter.class}
)}
)
public @interface SpringBootApplication {
```

@**SpringBootConfiguration**:spring boot的配置类;

​		标注在某个类上，就表示是一个spring boot的配置类

​		@**Configuration**:配置类上来标注这个注解

​				配置类------配置文件；配置类也是一个容器中的组建;@Component

@**EnableAutoConfiguration**:开启自动配置功能;

```java
@AutoConfigurationPackage//自动配置包
@Import({AutoConfigurationImportSelector.class})
```

