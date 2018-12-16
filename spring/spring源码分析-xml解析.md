# sping源码分析-xml解析

spring代码风格喜欢使用xxxx()做准备工作，然后再doxxxx()方法里面做实际的工作:比如LoadBeanDefinitions()和doLoadBeanDefinitions()，还有getBean()和doGetBean()

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



- XmlBeanFactory需要传入一个Resource，spring这里给我们封装了一个从classpath目录下加载文件的ClassPathResource类，ClassPathResource的实现方式其实就是通过classLoader的方式加载文件

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

  在XmlBeanFactory初始化的时候会往beanFactory的Set<Class<?>> ignoredDependencyInterfaces设置忽略依赖的接口

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

- spring通过sax的方式解析xml文档所以可以通过自己注册EntityResolver的方式解析自己定义的xml文件

- spring把xml文档解析成document以后，使用使用DefaultBeanDefinitionDocumentReader从doc解析

- DefaultBeanDefinitionDocumentReader中有一个BeanDefinitionParserDelegate，创建这个delegate的时候设置了beans的一些默认值

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

- BeanDefinitionParserDelegate在parseBeanDefinitions的时候会根据xml的每个Element的namespace是否是spring的来选择默认解析还是根据客户自定义来解析
- parseDefaultElement()方法会根据xml配置的规则分为4种不同的解析:IMPORT_ELEMENT、ALIAS_ELEMENT、BEAN_ELEMENT、NESTED_BEANS_ELEMENT解析

- spring对应bean标签的解析processBeanDefinition()

  先委托给BeanDefinitionParserDelegate.parseBeanDefinitionElement()解析spring的基本属性:**class、parent、scope、abstract、lazy-init、autowire、dependency-check、depends-on、autowire-candidate、primary、init-method、destroy-method、factory-method、factory-bean、description、meta、lookup-method、replaced-method、constructor-arg、property、qualifier**封装成GenericBeanDefinition对象,然后把GenericBeanDefinition和beanName以及别名数组一起封装成BeanDefinitionHolder，如果bean存在没有beanName的情况spring会自动给bean生成一个beanName

  - [ ] class、parent、scope、abstract、lazy-init、autowire、dependency-check、depends-on、autowire-candidate、primary、init-method、destroy-method、factory-method、factory-bean、description这些基础信息会直接被封装在GenericBeanDefinition的对应属性中
  - [ ] meta数据spring包装成一个BeanMetadataAttribute对象存放在GenericBeanDefinition属性attributes中，attributes是一个map
  - [ ] lookup-method数据spring包装成一个LookupOverride对象存放在GenericBeanDefinition的methodOverrides属性中，methodOverrides是一个set
  - [ ] replaced-method数据spring包装成一个ReplaceOverride对象封装在GenericBeanDefinition的methodOverrides属性中
  - [ ] constructor_arg的解析比较复杂，constructor_arg可以设置index、type、name三个属性，如果xml文件中构造参数设置了index属性，获取ref或者value属性的值，ref封装成RuntimeBeanReference，value封装成TypedStringValue，然后把RuntimeBeanReference或者TypedStringValue封装进ConstructorArgumentValues.ValueHolder中，然后把ConstructorArgumentValues.ValueHolder加入到GenericBeanDefinition的indexedArgumentValues中，indexedArgumentValues是个map一index为key。如果没有设置index属性，获取ref或者value属性的值，ref封装成RuntimeBeanReference，value封装成TypedStringValue，然后把RuntimeBeanReference或者TypedStringValue封装进ConstructorArgumentValues.ValueHolder中然后把ConstructorArgumentValues.ValueHolder加入GenericBeanDefinition的genericArgumentValues列表中
  - [ ] 解析bean中包含的property元素，获取ref或者value属性，然后和name属性封装成PropertyValue对象加入到GenericBeanDefinition的propertyValueList列表中
  - [ ] 解析bean中的qualifier元素，先获取type元素值，封装成AutowireCandidateQualifier对象然后加入GenericBeanDefinition的qualifiers中，qualifiers是map，以qualifier设置的type为key

  - 处理完基础属性以后，spring还会处理一些可能用户自定义的xml属性如果有的话。
  - 把bean转换为GenericBeanDefinition后，需要把GenericBeanDefinition注册到容器中DefaultListableBeanFactory.beanDefinitionMap



  - spring对应的import标签的解析importBeanDefinitionResource()

    获取resource属性，resource可以用占位符比如"${user.dir}"所以要先处理占位符

    ```
    getReaderContext().getEnvironment().resolveRequiredPlaceholders(location);
    ```

    获取路径后调用loadBeanDefinitions()

  - spring对应的alias标签的解析processAliasRegistration()

    解析name和alias标签，注册alias和name之间的关系

  - spring对应beans标签的解析

    对应内置beans标签的解析，就是递归调用doRegisterBeanDefinitions()



  - 用户自定义标签的解析parseCustomElement()----这个单独开一篇讲吧
