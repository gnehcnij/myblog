### Spring容器创建于初始化

**以AnnotationConfigApplicationContext为例：**

```java
@Configuration
public class MainConfig2 {

    @Scope(value = "singleton")
    @Lazy
    @Bean
    public Person person() {
        return new Person("Scope", 1);
    }
}

public class MainTest2 {

    @Test
    public void test03() {
        ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig2.class);
        System.out.println("ioc容器创建完成");
        Object person01 = ioc.getBean("person");
        Object person02 = ioc.getBean("person");
        System.out.println(person01 == person02);
    }
}
```

```java
/**
 * AnnotationConfigApplicationContext.java
 * Create a new AnnotationConfigApplicationContext, deriving bean definitions
 * from the given component classes and automatically refreshing the context.
 * @param componentClasses one or more component classes &mdash; for example,
 * {@link Configuration @Configuration} classes
 */
public AnnotationConfigApplicationContext(Class<?>... componentClasses) {
    this();							// 1
    register(componentClasses);		// 2. 往DefaultListableBeanFactory设置值
    refresh();						// 3
}
```

#### 1、this();

**创建AnnotatedBeanDefinitionReader、ClassPathBeanDefinitionScanner**

```java
/**
 * AnnotationConfigApplicationContext.java
 * Create a new AnnotationConfigApplicationContext that needs to be populated
 * through {@link #register} calls and then manually {@linkplain #refresh refreshed}.
 */
public AnnotationConfigApplicationContext() {
    this.reader = new AnnotatedBeanDefinitionReader(this);
    this.scanner = new ClassPathBeanDefinitionScanner(this);
}
```

#### 2、register(componentClasses);

```java
/**
 * AnnotationConfigApplicationContext.java
 * Register one or more component classes to be processed.
 * <p>Note that {@link #refresh()} must be called in order for the context
 * to fully process the new classes.
 * @param componentClasses one or more component classes &mdash; for example,
 * {@link Configuration @Configuration} classes
 * @see #scan(String...)
 * @see #refresh()
 */
@Override
public void register(Class<?>... componentClasses) {
    Assert.notEmpty(componentClasses, "At least one component class must be specified");
    this.reader.register(componentClasses);		// this.reader是上面创建的AnnotatedBeanDefinitionReader
}

/**
 * AnnotatedBeanDefinitionReader.java
 * Register one or more component classes to be processed.
 * <p>Calls to {@code register} are idempotent; adding the same
 * component class more than once has no additional effect.
 * @param componentClasses one or more component classes,
 * e.g. {@link Configuration @Configuration} classes
 */
public void register(Class<?>... componentClasses) {
    for (Class<?> componentClass : componentClasses) {
        registerBean(componentClass);
    }
}

/**
 * AnnotatedBeanDefinitionReader.java
 * Register a bean from the given bean class, deriving its metadata from
 * class-declared annotations.
 * @param beanClass the class of the bean
 */
public void registerBean(Class<?> beanClass) {
    doRegisterBean(beanClass, null, null, null, null);
}
```

```java
/**
 * AnnotatedBeanDefinitionReader.java
 * Register a bean from the given bean class, deriving its metadata from
 * class-declared annotations.
 * @param beanClass the class of the bean
 * @param name an explicit name for the bean
 * @param qualifiers specific qualifier annotations to consider, if any,
 * in addition to qualifiers at the bean class level
 * @param supplier a callback for creating an instance of the bean
 * (may be {@code null})
 * @param customizers one or more callbacks for customizing the factory's
 * {@link BeanDefinition}, e.g. setting a lazy-init or primary flag
 * @since 5.0
 */
private <T> void doRegisterBean(Class<T> beanClass, @Nullable String name,
                                @Nullable Class<? extends Annotation>[] qualifiers, @Nullable Supplier<T> supplier,
                                @Nullable BeanDefinitionCustomizer[] customizers) {

    AnnotatedGenericBeanDefinition abd = new AnnotatedGenericBeanDefinition(beanClass);
    if (this.conditionEvaluator.shouldSkip(abd.getMetadata())) {
        return;
    }

    abd.setInstanceSupplier(supplier);
    // 作用域元数据解析器解析出作用域元数据
    ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(abd);
    abd.setScope(scopeMetadata.getScopeName());
    String beanName = (name != null ? name : this.beanNameGenerator.generateBeanName(abd, this.registry));

    AnnotationConfigUtils.processCommonDefinitionAnnotations(abd);
    if (qualifiers != null) {
        for (Class<? extends Annotation> qualifier : qualifiers) {
            if (Primary.class == qualifier) {
                abd.setPrimary(true);
            }
            else if (Lazy.class == qualifier) {
                abd.setLazyInit(true);
            }
            else {
                abd.addQualifier(new AutowireCandidateQualifier(qualifier));
            }
        }
    }
    if (customizers != null) {
        for (BeanDefinitionCustomizer customizer : customizers) {
            customizer.customize(abd);
        }
    }

    BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(abd, beanName);
    definitionHolder = AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
    BeanDefinitionReaderUtils.registerBeanDefinition(definitionHolder, this.registry);
}
```