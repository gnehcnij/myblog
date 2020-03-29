### Bean生命周期

**Bean的生命周期就是Bean创建 --> 初始化 --> 销毁的过程。**

**我们可以自定义初始化和销毁方法，ioc容器在bean进行到对应生命周期的时候，就会执行我们自定义的初始化和销毁方法。**

#### 1、@Bean注解指定初始化和销毁方法

**在<bean></bean>标签指定init-method或者destroy-method，如：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">

    <!-- 指定初始化和销毁方法   -->
    <bean id="phone" class="com.gnehcnij.beans.Phone" init-method="init" destroy-method="destroy">
        
    </bean>
    
</beans>
```

**Phone类：**

```java
public class Phone {

    public Phone() {
        System.out.println("Phone的构造函数。。。。。。");
    }

    public void init() {
        System.out.println("Phone中自定义的init方法。。。。。。");
    }

    public void destroy() {
        System.out.println("Phone中自定义的destroy方法。。。。。。");
    }

}
```

**测试类：**

```java
public class LifeOfCircleTest {

    @Test
    public void test01() {
        ClassPathXmlApplicationContext ioc = new ClassPathXmlApplicationContext("classpath:beans.xml");
        System.out.println("ioc容器创建完成");
		ioc.close();
        // 输出：
        // Phone的构造函数。。。。。。
		// Phone中自定义的init方法。。。。。。
		// ioc容器创建完成
		// Phone中自定义的destroy方法。。。。。。
    }

}
```

**上面基于xml的配置等同于下面代码（使用@Bean注解）：**

**配置类：**

```java
@Configuration
public class LifeOfCircleConfig {
	// 默认是单实例的
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Phone phone() {
        return new Phone();
    }

}
```

**测试类：**

```java
public class LifeOfCircleTest {

    @Test
    public void test02() {
        AnnotationConfigApplicationContext ioc = new AnnotationConfigApplicationContext(LifeOfCircleConfig.class);
        System.out.println("ioc容器创建完成");
        ioc.close();
        // 输出：
        // Phone的构造函数。。。。。。
		// Phone中自定义的init方法。。。。。。
		// ioc容器创建完成
		// Phone中自定义的destroy方法。。。。。。
    }

}
```

**把配置类的Phone改成多实例的：**

```java
@Configuration
public class LifeOfCircleConfig {

    @Scope(value = "prototype")
    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Phone phone() {
        return new Phone();
    }

}
```

**测试类：**

```java
public class LifeOfCircleTest {

    @Test
    public void test02() {
        AnnotationConfigApplicationContext ioc = new AnnotationConfigApplicationContext(LifeOfCircleConfig.class);
        System.out.println("ioc容器创建完成");
        ioc.close();
        // 输出：
		// ioc容器创建完成
    }
    
    @Test
    public void test03() {
        AnnotationConfigApplicationContext ioc = new AnnotationConfigApplicationContext(LifeOfCircleConfig.class);
        System.out.println("ioc容器创建完成");
        ioc.getBean("phone");	// 使用bean
        ioc.close();
        // 输出：
		// ioc容器创建完成
        // Phone的构造函数。。。。。。
		// Phone中自定义的init方法。。。。。。
    }

}
```

**结论：**

- 单实例：对象创建完成，并赋值好，调用初始化方法。
- 销毁：单实例，在容器关闭的时候调用销毁方法；多实例，容器不会管理这个bean

#### 2、通过让bean实现InitializingBean和DisposableBean接口

**实现InitializingBean接口定义初始化逻辑。**

```java
/**
 * Interface to be implemented by beans that need to react once all their properties
 * have been set by a {@link BeanFactory}: e.g. to perform custom initialization,
 * or merely to check that all mandatory properties have been set.
 *
 * <p>An alternative to implementing {@code InitializingBean} is specifying a custom
 * init method, for example in an XML bean definition. For a list of all bean
 * lifecycle methods, see the {@link BeanFactory BeanFactory javadocs}.
 */
public interface InitializingBean {

	/**
	 * Invoked by the containing {@code BeanFactory} after it has set all bean properties
	 * and satisfied {@link BeanFactoryAware}, {@code ApplicationContextAware} etc.
	 * <p>This method allows the bean instance to perform validation of its overall
	 * configuration and final initialization when all bean properties have been set.
	 * @throws Exception in the event of misconfiguration (such as failure to set an
	 * essential property) or if initialization fails for any other reason
	 */
	void afterPropertiesSet() throws Exception;

}
```

**实现DisposableBean接口定义销毁逻辑。**

```java
/**
 * Interface to be implemented by beans that want to release resources on destruction.
 * A {@link BeanFactory} will invoke the destroy method on individual destruction of a
 * scoped bean. An {@link org.springframework.context.ApplicationContext} is supposed
 * to dispose all of its singletons on shutdown, driven by the application lifecycle.
 *
 * <p>A Spring-managed bean may also implement Java's {@link AutoCloseable} interface
 * for the same purpose. An alternative to implementing an interface is specifying a
 * custom destroy method, for example in an XML bean definition. For a list of all
 * bean lifecycle methods, see the {@link BeanFactory BeanFactory javadocs}.
 */
public interface DisposableBean {

	/**
	 * Invoked by the containing {@code BeanFactory} on destruction of a bean.
	 * @throws Exception in case of shutdown errors. Exceptions will get logged
	 * but not rethrown to allow other beans to release their resources as well.
	 */
	void destroy() throws Exception;

}
```

**Cat类：**

```java
public class Cat implements InitializingBean, DisposableBean {

    public Cat() {
        System.out.println("Cat的构造函数。。。。。。");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("Cat的afterPropertiesSet()。。。。。。");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("Cat的destroy()。。。。。。");
    }

}
```

**配置类：**

```java
@Configuration
public class LifeOfCircleConfig {

    @Bean
    public Cat cat() {
        return new Cat();
    }

}
```

**测试类：**

```java
public class LifeOfCircleTest {

    @Test
    public void test03() {
        AnnotationConfigApplicationContext ioc = new AnnotationConfigApplicationContext(LifeOfCircleConfig.class);
        System.out.println("ioc容器创建完成");
        ioc.close();
        // 输出：
        // Cat的构造函数。。。。。。
		// Cat的afterPropertiesSet()。。。。。。
		// ioc容器创建完成
		// Cat的destroy()。。。。。。
    }

}
```

#### 3、使用JSR250的@PostConstruct和@PreDestroy

**@PostConstruct：在bean创建完成并且属性赋值完成后执行对应的方法。**

```java
/**
 * The PostConstruct annotation is used on a method that needs to be executed
 * after dependency injection is done to perform any initialization. This
 * method MUST be invoked before the class is put into service. This
 * annotation MUST be supported on all classes that support dependency
 * injection. The method annotated with PostConstruct MUST be invoked even
 * if the class does not request any resources to be injected. Only one
 * method can be annotated with this annotation. The method on which the
 * PostConstruct annotation is applied MUST fulfill all of the following
 * criteria:
 * <p>
 * <ul>
 * <li>The method MUST NOT have any parameters except in the case of
 * interceptors in which case it takes an InvocationContext object as
 * defined by the Interceptors specification.</li>
 * <li>The method defined on an interceptor class MUST HAVE one of the
 * following signatures:
 * <p>
 * void &#060;METHOD&#062;(InvocationContext)
 * <p>
 * Object &#060;METHOD&#062;(InvocationContext) throws Exception
 * <p>
 * <i>Note: A PostConstruct interceptor method must not throw application
 * exceptions, but it may be declared to throw checked exceptions including
 * the java.lang.Exception if the same interceptor method interposes on
 * business or timeout methods in addition to lifecycle events. If a
 * PostConstruct interceptor method returns a value, it is ignored by
 * the container.</i>
 * </li>
 * <li>The method defined on a non-interceptor class MUST HAVE the
 * following signature:
 * <p>
 * void &#060;METHOD&#062;()
 * </li>
 * <li>The method on which PostConstruct is applied MAY be public, protected,
 * package private or private.</li>
 * <li>The method MUST NOT be static except for the application client.</li>
 * <li>The method MAY be final.</li>
 * <li>If the method throws an unchecked exception the class MUST NOT be put into
 * service except in the case of EJBs where the EJB can handle exceptions and
 * even recover from them.</li></ul>
 */
@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PostConstruct {
}

```

**@PreDestroy：在容器销毁bean之前执行对应方法。**

```java
/**
 * The PreDestroy annotation is used on methods as a callback notification to
 * signal that the instance is in the process of being removed by the
 * container. The method annotated with PreDestroy is typically used to
 * release resources that it has been holding. This annotation MUST be
 * supported by all container managed objects that support PostConstruct
 * except the application client container in Java EE 5. The method on which
 * the PreDestroy annotation is applied MUST fulfill all of the following
 * criteria:
 * <p>
 * <ul>
 * <li>The method MUST NOT have any parameters except in the case of
 * interceptors in which case it takes an InvocationContext object as
 * defined by the Interceptors specification.</li>
 * <li>The method defined on an interceptor class MUST HAVE one of the
 * following signatures:
 * <p>
 * void &#060;METHOD&#062;(InvocationContext)
 * <p>
 * Object &#060;METHOD&#062;(InvocationContext) throws Exception
 * <p>
 * <i>Note: A PreDestroy interceptor method must not throw application
 * exceptions, but it may be declared to throw checked exceptions including
 * the java.lang.Exception if the same interceptor method interposes on
 * business or timeout methods in addition to lifecycle events. If a
 * PreDestroy interceptor method returns a value, it is ignored by
 * the container.</i>
 * </li>
 * <li>The method defined on a non-interceptor class MUST HAVE the
 * following signature:
 * <p>
 * void &#060;METHOD&#062;()
 * </li>
 * <li>The method on which PreDestroy is applied MAY be public, protected,
 * package private or private.</li>
 * <li>The method MUST NOT be static.</li>
 * <li>The method MAY be final.</li>
 * <li>If the method throws an unchecked exception it is ignored except in the
 * case of EJBs where the EJB can handle exceptions.</li>
 * </ul>
 */

@Documented
@Retention (RUNTIME)
@Target(METHOD)
public @interface PreDestroy {
}
```

**Dog类：**

```java
public class Dog {

    public Dog() {
        System.out.println("Dog的构造函数。。。。。。");
    }

    @PostConstruct
    public void init() {
        System.out.println("Dog的init()。。。。。。@PostConstruct。。。。。。");
    }

    @PreDestroy
    public void destroy() {
        System.out.println("Dog的destroy()。。。。。。@PreDestroy。。。。。。");
    }

}
```

**配置类：**

```java
@Configuration
public class LifeOfCircleConfig {
    @Bean
    public Dog dog() {
        return new Dog();
    }
}
```

**测试类：**
```java
public class LifeOfCircleTest {

    @Test
    public void test03() {
        AnnotationConfigApplicationContext ioc = new AnnotationConfigApplicationContext(LifeOfCircleConfig.class);
        System.out.println("ioc容器创建完成");
        ioc.close();
        // 输出：
        // Dog的构造函数。。。。。。
		// Dog的init()。。。。。。@PostConstruct。。。。。。
		// ioc容器创建完成
		// Dog的destroy()。。。。。。@PreDestroy。。。。。。
    }

}
```

#### 4、使用BeanPostProcessor接口

**BeanPostProcessor：bean的后置处理器，Spring提供的组件，在bean的初始化前后做一些处理工作。**

```java
public interface BeanPostProcessor {

	/**
	 * Apply this {@code BeanPostProcessor} to the given new bean instance <i>before</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>The default implementation returns the given {@code bean} as-is.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * 在初始化前工作
	 */
	@Nullable
	default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

	/**
	 * Apply this {@code BeanPostProcessor} to the given new bean instance <i>after</i> any bean
	 * initialization callbacks (like InitializingBean's {@code afterPropertiesSet}
	 * or a custom init-method). The bean will already be populated with property values.
	 * The returned bean instance may be a wrapper around the original.
	 * <p>In case of a FactoryBean, this callback will be invoked for both the FactoryBean
	 * instance and the objects created by the FactoryBean (as of Spring 2.0). The
	 * post-processor can decide whether to apply to either the FactoryBean or created
	 * objects or both through corresponding {@code bean instanceof FactoryBean} checks.
	 * <p>This callback will also be invoked after a short-circuiting triggered by a
	 * {@link InstantiationAwareBeanPostProcessor#postProcessBeforeInstantiation} method,
	 * in contrast to all other {@code BeanPostProcessor} callbacks.
	 * <p>The default implementation returns the given {@code bean} as-is.
	 * @param bean the new bean instance
	 * @param beanName the name of the bean
	 * @return the bean instance to use, either the original or a wrapped one;
	 * if {@code null}, no subsequent BeanPostProcessors will be invoked
	 * @throws org.springframework.beans.BeansException in case of errors
	 * @see org.springframework.beans.factory.InitializingBean#afterPropertiesSet
	 * @see org.springframework.beans.factory.FactoryBean
	 * 在初始化后工作
	 */
	@Nullable
	default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
		return bean;
	}

}
```

**自定义一个beanPostProcessor：**

```java
@Component
public class MyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "......postProcessBeforeInitialization......" + bean);
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println(beanName + "......postProcessAfterInitialization......" + bean);
        return bean;
    }

}
```

**配置类：**

```java
@Configuration
@ComponentScan(value = "com.gnehcnij.utils")
public class LifeOfCircleConfig {

    @Bean(initMethod = "init", destroyMethod = "destroy")
    public Phone phone() {
        return new Phone();
    }

    @Bean
    public Cat cat() {
        return new Cat();
    }

    @Bean
    public Dog dog() {
        return new Dog();
    }

}
```

**测试类：**

```java
public class LifeOfCircleTest {

    @Test
    public void test03() {
        AnnotationConfigApplicationContext ioc = new AnnotationConfigApplicationContext(LifeOfCircleConfig.class);
        System.out.println("ioc容器创建完成");
        ioc.close();
        // 输出：
        // lifeOfCircleConfig......postProcessBeforeInitialization......com.gnehcnij.config.LifeOfCircleConfig$$EnhancerBySpringCGLIB$$e9c528ee@63a65a25
        // lifeOfCircleConfig......postProcessAfterInitialization......com.gnehcnij.config.LifeOfCircleConfig$$EnhancerBySpringCGLIB$$e9c528ee@63a65a25
        // Phone的构造函数。。。。。。
        // phone......postProcessBeforeInitialization......com.gnehcnij.beans.Phone@75329a49
        // Phone中自定义的init方法。。。。。。
        // phone......postProcessAfterInitialization......com.gnehcnij.beans.Phone@75329a49
        // Cat的构造函数。。。。。。
        // cat......postProcessBeforeInitialization......com.gnehcnij.beans.Cat@7f010382
        // Cat的afterPropertiesSet()。。。。。。
        // cat......postProcessAfterInitialization......com.gnehcnij.beans.Cat@7f010382
        // Dog的构造函数。。。。。。
        // dog......postProcessBeforeInitialization......com.gnehcnij.beans.Dog@1f0f1111
        // Dog的init()。。。。。。@PostConstruct。。。。。。
        // dog......postProcessAfterInitialization......com.gnehcnij.beans.Dog@1f0f1111
        // ioc容器创建完成
        // Dog的destroy()。。。。。。@PreDestroy。。。。。。
        // Cat的destroy()。。。。。。
        // Phone中自定义的destroy方法。。。。。。
    }

}
```

**容易看出：postProcessBeforeInitialization是在bean初始化前执行的，postProcessAfterInitialization是在bean初始化后执行的。**

**BeanPostProcessor原理：**

```java
try {
    // 属性赋值
    populateBean(beanName, mbd, instanceWrapper);
	exposedObject = initializeBean(beanName, exposedObject, mbd) {
        wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
        try {
            // 执行初始化方法
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
        wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
    }
}
```

**完整过程：**

```java
new AnnotationConfigApplicationContext(LifeOfCircleConfig.class).init() {
    AbstractApplicationContext.refresh()
    {
        AbstractApplicationContext.finishBeanFactoryInitialization(beanFactory)
        {
            DefaultListableBeanFactory[beanFactory].preInstantiateSingletons()
            {
                AbstractBeanFactory.getBean(beanName)
                {
                    AbstractBeanFactory.doGetBean(name, null, null, false)
                    {
                        DefaultSingletonBeanRegistry.getSingleton(beanName, () -> {
                            try {
                                AbstractAutowireCapableBeanFactory.createBean(beanName, mbd, args) 
                                {
                                    AbstractAutowireCapableBeanFactory.doCreateBean()
                                    {
                                        try {
                                            // 属性赋值
                                            AbstractAutowireCapableBeanFactory.populateBean(beanName, mbd, instanceWrapper);
                                            exposedObject = AbstractAutowireCapableBeanFactory.initializeBean(beanName, exposedObject, mbd) {
                                                wrappedBean = AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
                                                try {
                                                    // 执行初始化方法
                                                    AbstractAutowireCapableBeanFactory.invokeInitMethods(beanName, wrappedBean, mbd);
                                                }
                                                wrappedBean = AbstractAutowireCapableBeanFactory.applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
                                            }
                                        }
                                    }
                                }
                            }
                        });
                    }
                }
            }
        }
    }
}
```

**化简：**

```java
new AnnotationConfigApplicationContext(LifeOfCircleConfig.class).init() {
	AbstractApplicationContext {
        => refresh()
		=> finishBeanFactoryInitialization(beanFactory)
		=> DefaultListableBeanFactory[beanFactory].preInstantiateSingletons()
		AbstractBeanFactory {
            => getBean(beanName)
			=> doGetBean(name, null, null, false)
			DefaultSingletonBeanRegistry {
                => getSingleton(beanName, () -> {
                    AbstractAutowireCapableBeanFactory {
                        => createBean(beanName, mbd, args)
                        => doCreateBean()
                        // 属性赋值
                        => populateBean(beanName, mbd, instanceWrapper);
                        => initializeBean(beanName, exposedObject, mbd)
                        => {
                            applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
                            // 执行初始化方法
                            invokeInitMethods(beanName, wrappedBean, mbd);
                            applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
                        }
                    }
                }
            }
        }
    }
}
```
