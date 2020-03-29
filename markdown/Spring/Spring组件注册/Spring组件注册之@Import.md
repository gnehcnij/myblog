### Spring组件注册之@Import

**Spring版本为5.2.2.RELEASE**

**@Import可以快速给ioc容器导入组件，id默认为组件的全类名**

#### 1、配置类

```java
@Configuration
public class MainConfig4 {
}
```

#### 2、测试类
```java
public class MainTest4 {

    @Test
    public void test03() {
        ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig4.class);
        System.out.println("ioc容器创建完成");
        for (String name : ioc.getBeanDefinitionNames()) {
            System.out.println(name);
        }
        // 输出：
        // ioc容器创建完成
        // org.springframework.context.annotation.internalConfigurationAnnotationProcessor
        // org.springframework.context.annotation.internalAutowiredAnnotationProcessor
        // org.springframework.context.annotation.internalCommonAnnotationProcessor
        // org.springframework.context.event.internalEventListenerProcessor
        // org.springframework.context.event.internalEventListenerFactory
        // mainConfig4
    }
}
```

#### 3、方式一：@Import(要导入的组件)

**给ioc容器导入Phone组件**

```
@Configuration
@Import(value = {Phone.class})
public class MainConfig4 {
}

// 测试类输出：
// ioc容器创建完成
// org.springframework.context.annotation.internalConfigurationAnnotationProcessor
// org.springframework.context.annotation.internalAutowiredAnnotationProcessor
// org.springframework.context.annotation.internalCommonAnnotationProcessor
// org.springframework.context.event.internalEventListenerProcessor
// org.springframework.context.event.internalEventListenerFactory
// mainConfig4
// com.gnehcnij.beans.Phone
```

#### 4、方式二：使用ImportSelector

**ImportSelector：返回需要导入的组件全类名数组，不会导入Selector本身**

##### ImportSelector定义：

```java
public interface ImportSelector {

	/**
	 * Select and return the names of which class(es) should be imported based on
	 * the {@link AnnotationMetadata} of the importing @{@link Configuration} class.
	 * AnnotationMetadata	当前标注@Impo注解的类的所有注解信息
	 * return 				导入容器的组件的全类名
	 */
	String[] selectImports(AnnotationMetadata importingClassMetadata);
}
```

##### 给ioc容器导入Phone、Apple组件。

```java
@Configuration
@Import(value = {MyImportSelector.class})
public class MainConfig4 {
}

// 测试类输出：
// ioc容器创建完成
// org.springframework.context.annotation.internalConfigurationAnnotationProcessor
// org.springframework.context.annotation.internalAutowiredAnnotationProcessor
// org.springframework.context.annotation.internalCommonAnnotationProcessor
// org.springframework.context.event.internalEventListenerProcessor
// org.springframework.context.event.internalEventListenerFactory
// mainConfig4
// com.gnehcnij.beans.Phone
// com.gnehcnij.beans.Apple
```

#### 5、方式三：使用ImportBeanDefinitionRegistrar

##### ImportBeanDefinitionRegistrar定义：

```java
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * <p>The default implementation delegates to
	 * {@link #registerBeanDefinitions(AnnotationMetadata, BeanDefinitionRegistry)}.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 * @param importBeanNameGenerator the bean name generator strategy for imported beans:
	 * {@link ConfigurationClassPostProcessor#IMPORT_BEAN_NAME_GENERATOR} by default, or a
	 * user-provided one if {@link ConfigurationClassPostProcessor#setBeanNameGenerator}
	 * has been set. In the latter case, the passed-in strategy will be the same used for
	 * component scanning in the containing application context (otherwise, the default
	 * component-scan naming strategy is {@link AnnotationBeanNameGenerator#INSTANCE}).
	 * @since 5.2
	 * @see ConfigurationClassPostProcessor#IMPORT_BEAN_NAME_GENERATOR
	 * @see ConfigurationClassPostProcessor#setBeanNameGenerator
	 */
	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry,
			BeanNameGenerator importBeanNameGenerator) {

		registerBeanDefinitions(importingClassMetadata, registry);
	}

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * <p>The default implementation is empty.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	default void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
	}

}
```

##### 给ioc容器导入Person组件

```java
@Configuration
@Import(value = {MyImportBeanDefinitionRegistrar.class})
public class MainConfig4 {
}

// 测试类输出：
// ioc容器创建完成
// org.springframework.context.annotation.internalConfigurationAnnotationProcessor
// org.springframework.context.annotation.internalAutowiredAnnotationProcessor
// org.springframework.context.annotation.internalCommonAnnotationProcessor
// org.springframework.context.event.internalEventListenerProcessor
// org.springframework.context.event.internalEventListenerFactory
// mainConfig4
// Bill Gates
```

#### 6、@Import注解在Spring的应用举例：

**基于注解的AOP。**

**1、配置类**

```java
@Configuration
@EnableAspectJAutoProxy
public class AopConfig {
}

```

**2、@EnableAspectJAutoProxy**

```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(AspectJAutoProxyRegistrar.class)	// 这里使用了@Import注解
public @interface EnableAspectJAutoProxy {

	/**
	 * Indicate whether subclass-based (CGLIB) proxies are to be created as opposed
	 * to standard Java interface-based proxies. The default is {@code false}.
	 */
	boolean proxyTargetClass() default false;

	/**
	 * Indicate that the proxy should be exposed by the AOP framework as a {@code ThreadLocal}
	 * for retrieval via the {@link org.springframework.aop.framework.AopContext} class.
	 * Off by default, i.e. no guarantees that {@code AopContext} access will work.
	 * @since 4.3.1
	 */
	boolean exposeProxy() default false;

}
```

**3、AspectJAutoProxyRegistrar**

**使用了上面说的第三种方式：**

```java
class AspectJAutoProxyRegistrar implements ImportBeanDefinitionRegistrar {

	/**
	 * Register, escalate, and configure the AspectJ auto proxy creator based on the value
	 * of the @{@link EnableAspectJAutoProxy#proxyTargetClass()} attribute on the importing
	 * {@code @Configuration} class.
	 */
	@Override
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {

		AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);

		AnnotationAttributes enableAspectJAutoProxy =
				AnnotationConfigUtils.attributesFor(importingClassMetadata, EnableAspectJAutoProxy.class);
		if (enableAspectJAutoProxy != null) {
			if (enableAspectJAutoProxy.getBoolean("proxyTargetClass")) {
				AopConfigUtils.forceAutoProxyCreatorToUseClassProxying(registry);
			}
			if (enableAspectJAutoProxy.getBoolean("exposeProxy")) {
				AopConfigUtils.forceAutoProxyCreatorToExposeProxy(registry);
			}
		}
	}

}
```