# sping源码分析

开始:spring配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean class="com.ljc.tset.User" name="user"></bean>
</beans>
```

测试入口

```
@Test
public void test() {

    XmlBeanFactory beanFactory = new XmlBeanFactory(new ClassPathResource("spring.xml"));
    User user = (User) beanFactory.getBean("user");
}
```



- XmlBeanFactory需要传入一个Resource，spring这里给我们封装了一个冲classpath目录下加载文件的ClassPathResource类，ClassPathResource的实现方式其实就是通过classLoader的方式加载文件

  ```
  public InputStream getInputStream() throws IOException {
      InputStream is;
      if (this.clazz != null) {
          is = this.clazz.getResourceAsStream(this.path);
      } else if (this.classLoader != null) {
          is = this.classLoader.getResourceAsStream(this.path);
      } else {
          is = ClassLoader.getSystemResourceAsStream(this.path);
      }
  
      if (is == null) {
          throw new FileNotFoundException(this.getDescription() + " cannot be opened because it does not exist");
      } else {
          return is;
      }
  }
  ```

- 在XmlBeanFactory初始化的时候会设置一下儿忽略的接口

  ```
  ignoreDependencyInterface(BeanNameAware.class);
  ignoreDependencyInterface(BeanFactoryAware.class);
  ignoreDependencyInterface(BeanClassLoaderAware.class);
  ```

- XmlBeanFactory中使用XmlBeanDefinitionReader来加载BeanDefinitions，在XmlBeanDefinitionReader加载BeanDefinitions之前，需要把Resource封装成EncodedResource，EncodedResource可以设置编码格式

- spring在加载文件的时候用了一个Set<EncodedResource> currentResources来避免重复加载文件

- spring加载Document doc = doLoadDocument(inputSource, resource);

  ```
  protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
     return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
           getValidationModeForResource(resource), isNamespaceAware());
  }
  
  getEntityResolver()获取实体解析器
      实例化了两个解析器
      this.dtdResolver = new BeansDtdResolver();
      this.schemaResolver = new PluggableSchemaResolver(classLoader);
  		PluggableSchemaResolver有个字段schemaMappingsLocation初始化的时候被设置为了		META-INF/spring.schemas
  getValidationModeForResource(resource)获取spring配置文件的验证方式获取方式为读取配置文件查看是否有”DOCTYPE“，如果有返回dtd的验证模式VALIDATION_DTD，否则返回xsd的验证模式VALIDATION_XSD
  ```

- spring代码风格喜欢使用xxxx()做准备工作，然后再doxxxx()方法里面做实际的工作:比如LoadBeanDefinitions()和doLoadBeanDefinitions()，还有getBean()和doGetBean()

- spring使用DefaultBeanDefinitionDocumentReader从doc解析BeanDefinition,DefaultBeanDefinitionDocumentReader中有一个BeanDefinitionParserDelegate，创建这个delegate的时候是在了beans的一些默认值

  ```
  //default-lazy-init属性
  String lazyInit = root.getAttribute(DEFAULT_LAZY_INIT_ATTRIBUTE);
  if (DEFAULT_VALUE.equals(lazyInit)) {
     // Potentially inherited from outer <beans> sections, otherwise falling back to false.
     lazyInit = (parentDefaults != null ? parentDefaults.getLazyInit() : FALSE_VALUE);
  }
  defaults.setLazyInit(lazyInit);
  
  //default-merge属性
  String merge = root.getAttribute(DEFAULT_MERGE_ATTRIBUTE);
  if (DEFAULT_VALUE.equals(merge)) {
     // Potentially inherited from outer <beans> sections, otherwise falling back to false.
     merge = (parentDefaults != null ? parentDefaults.getMerge() : FALSE_VALUE);
  }
  defaults.setMerge(merge);
  
  //default-autowire属性
  String autowire = root.getAttribute(DEFAULT_AUTOWIRE_ATTRIBUTE);
  if (DEFAULT_VALUE.equals(autowire)) {
     // Potentially inherited from outer <beans> sections, otherwise falling back to 'no'.
     autowire = (parentDefaults != null ? parentDefaults.getAutowire() : AUTOWIRE_NO_VALUE);
  }
  defaults.setAutowire(autowire);
  
  //default-dependency-check属性
  // Don't fall back to parentDefaults for dependency-check as it's no longer supported in
  // <beans> as of 3.0. Therefore, no nested <beans> would ever need to fall back to it.
  defaults.setDependencyCheck(root.getAttribute(DEFAULT_DEPENDENCY_CHECK_ATTRIBUTE));
  
  //default-autowire-candidates属性
  if (root.hasAttribute(DEFAULT_AUTOWIRE_CANDIDATES_ATTRIBUTE)) {
     defaults.setAutowireCandidates(root.getAttribute(DEFAULT_AUTOWIRE_CANDIDATES_ATTRIBUTE));
  }
  else if (parentDefaults != null) {
     defaults.setAutowireCandidates(parentDefaults.getAutowireCandidates());
  }
  
  //default-init-method属性
  if (root.hasAttribute(DEFAULT_INIT_METHOD_ATTRIBUTE)) {
     defaults.setInitMethod(root.getAttribute(DEFAULT_INIT_METHOD_ATTRIBUTE));
  }
  else if (parentDefaults != null) {
     defaults.setInitMethod(parentDefaults.getInitMethod());
  }
  //default-destroy-method属性
  if (root.hasAttribute(DEFAULT_DESTROY_METHOD_ATTRIBUTE)) {
     defaults.setDestroyMethod(root.getAttribute(DEFAULT_DESTROY_METHOD_ATTRIBUTE));
  }
  else if (parentDefaults != null) {
     defaults.setDestroyMethod(parentDefaults.getDestroyMethod());
  }
  
  defaults.setSource(this.readerContext.extractSource(root));
  ```

- 在解析beans时会首先判断profile属性，然后和环境中的设置比对，如果对不上直接跳过

  ```
  if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
     if (logger.isInfoEnabled()) {
     return;
  }
  
  preProcessXml(root);
  parseBeanDefinitions(root, this.delegate);
  postProcessXml(root);
  ```

