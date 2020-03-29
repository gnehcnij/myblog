### Spring组件注册之@Conditional

**Spring版本为5.2.2.RELEASE**

**@Conditional设置条件，满足条件的组件才会注册到ioc容器中**。

```java
@Target({ElementType.TYPE, ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Conditional {

	/**
	 * All {@link Condition Conditions} that must {@linkplain Condition#matches match}
	 * in order for the component to be registered.
	 */
	Class<? extends Condition>[] value();	// Condition是一个接口

}
```

**场景：如果是Windows系统，则向ioc容器中注册一个Bill Gates，如果是Linux系统则注册一个Linus Benedict Torvalds。**

#### 1、定义两个Condition

```java
public class WindowsCondition implements Condition {

    /**
     * @param context       判断条件能使用的上下文
     * @param metadata      注释信息
     * @return
     */
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {
        // ioc当前使用的beanFactory
        ConfigurableListableBeanFactory beanFactory = context.getBeanFactory();
        // 获取类加载器
        ClassLoader classLoader = context.getClassLoader();
        // 获取当前的环境信息
        Environment environment = context.getEnvironment();
        // 获取到bean定义的注册类，可以判断容器中bean的注册情况，也可以注册bean
        BeanDefinitionRegistry registry = context.getRegistry();

        String osName = environment.getProperty("os.name");
        return osName.contains("Windows");
    }
}
```

```java
public class LinuxCondition implements Condition {
    @Override
    public boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata) {

        String osName = context.getEnvironment().getProperty("os.name");
        return osName.contains("Linux");

    }
}
```

#### 2、配置类

```java
@Configuration
public class MainConfig3 {

    @Bean
    @Conditional(value = {WindowsCondition.class} )
    public Person person01() {
        return new Person("Bill Gates", 64);
    }

    @Bean
    @Conditional(value = {LinuxCondition.class})
    public Person person02() {
        return new Person("Linus Benedict Torvalds", 50);
    }

}
```

#### 3、测试

```java
public class MainTest3 {

    @Test
    public void test03() {
        ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig3.class);
        System.out.println("ioc容器创建完成");
        Map<String, Person> beans = ioc.getBeansOfType(Person.class);
        beans.forEach((k, v) -> System.out.println(k + " : " + v));
        // 输出：
        // Person的构造函数：Person(String name, int age)......
		// ioc容器创建完成
		// person01 : Person{name='Bill Gates', age=64}
    }

}
```