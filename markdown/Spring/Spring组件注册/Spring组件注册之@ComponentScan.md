### Spring组件注册之@ComponentScan

**Spring版本为5.2.2.RELEASE**

**@ComponentScan**标在配置类上：

```java
@Configuration
@ComponentScan(value = "com.gnehcnij")		// 包扫描com.gnehcnij
public class MainConfig {

    @Bean
    public Person person666() {
        return new Person("lisi01", 22);
    }
    
    @Bean(value = "person")
    public Person getPerson() {
        return new Person("lisi02", 22);
    }

}
```
**等同于如下代码：**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd 			  	http://www.springframework.org/schema/context https://www.springframework.org/schema/context/spring-context.xsd">
    
	<!-- 包扫描，只要标注@Controller、@Service、@Repository、@Component的类都会加进ioc容器中   -->
    <context:component-scan base-package="com.gnehcnij"/>

    <bean id="person" class="com.gnehcnij.beans.Person">
        <property name="name" value="zhangsan"/>
        <property name="age" value="21"/>
    </bean>
</beans>
```

**定义几个类：**

```java
@Controller
public class HelloController {
}

@Service
public class HelloService {
}

@Repository
public class HelloDao {
}

@Test
public void test03() {
    ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig.class);
    for (String definitionName : ioc.getBeanDefinitionNames()) {
        System.out.println(definitionName);
    }
    // 输出：
    // org.springframework.context.annotation.internalConfigurationAnnotationProcessor
    // org.springframework.context.annotation.internalAutowiredAnnotationProcessor
    // org.springframework.context.annotation.internalCommonAnnotationProcessor
    // org.springframework.context.event.internalEventListenerProcessor
    // org.springframework.context.event.internalEventListenerFactory
    // 以上为spring本身定义的，以下为自己定义的，可见id默认为类名的首字母小写
    // mainConfig
    // helloController
    // helloDao
    // helloService
    // person666
    // person
}
```

### @ComponentScan指定过滤规则：

#### 1、指定includeFilters

```java
@Configuration
@ComponentScan(value = "com.gnehcnij", includeFilters = {
    	// 按照注解类型过滤，把标的@Controller添加到ioc容器，即只要Controller
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class})
}, useDefaultFilters = false)	// 设置useDefaultFilters为false，禁用默认规则，因为默认规则是扫描所有
public class MainConfig {

    @Bean
    public Person person666() {
        return new Person("lisi01", 22);
    }

    @Bean(value = "person")
    public Person getPerson() {
        return new Person("lisi02", 22);
    }
}

@Test
public void test03() {
    ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig.class);
    for (String definitionName : ioc.getBeanDefinitionNames()) {
        System.out.println(definitionName);
    }
    // 输出：
    // org.springframework.context.annotation.internalConfigurationAnnotationProcessor
    // org.springframework.context.annotation.internalAutowiredAnnotationProcessor
    // org.springframework.context.annotation.internalCommonAnnotationProcessor
    // org.springframework.context.event.internalEventListenerProcessor
    // org.springframework.context.event.internalEventListenerFactory
    // 可见没有helloController、helloService
    // mainConfig
    // helloDao
    // person666
    // person
}
```

#### 2、指定excludeFilters

```java
@Configuration
@ComponentScan(value = "com.gnehcnij", excludeFilters = {
    	// 按照注解类型过滤，把标@Controller、@Service的类排除
        @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class, Service.class})
})
public class MainConfig {

    @Bean
    public Person person666() {
        return new Person("lisi01", 22);
    }

    @Bean(value = "person")
    public Person getPerson() {
        return new Person("lisi02", 22);
    }
}

@Test
public void test03() {
    ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig.class);
    for (String definitionName : ioc.getBeanDefinitionNames()) {
        System.out.println(definitionName);
    }
    // 输出：
    // org.springframework.context.annotation.internalConfigurationAnnotationProcessor
    // org.springframework.context.annotation.internalAutowiredAnnotationProcessor
    // org.springframework.context.annotation.internalCommonAnnotationProcessor
    // org.springframework.context.event.internalEventListenerProcessor
    // org.springframework.context.event.internalEventListenerFactory
    // 可见没有helloDao
    // mainConfig
	// helloController
	// person666
	// person
}
```

#### 3、@ComponentScans：

```java
@Configuration
@ComponentScans(value = {		// 标注多个
        @ComponentScan(value = "com.gnehcnij", excludeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Controller.class})
        }),
        @ComponentScan(value = "com.gnehcnij", excludeFilters = {
                @ComponentScan.Filter(type = FilterType.ANNOTATION, classes = {Service.class})
        })
})
public class MainConfig {

    @Bean
    public Person person666() {
        return new Person("lisi01", 22);
    }

    @Bean(value = "person")
    public Person getPerson() {
        return new Person("lisi02", 22);
    }
}
```

#### 4、@ComponentScan.Filter的FilterType（过滤规则）

```java
public enum FilterType {

	/**
	 * Filter candidates marked with a given annotation.
	 * @see org.springframework.core.type.filter.AnnotationTypeFilter
	 * 按照注解
	 */
	ANNOTATION,

	/**
	 * Filter candidates assignable to a given type.
	 * @see org.springframework.core.type.filter.AssignableTypeFilter
	 * 按照给定的类型
	 */
	ASSIGNABLE_TYPE,

	/**
	 * Filter candidates matching a given AspectJ type pattern expression.
	 * @see org.springframework.core.type.filter.AspectJTypeFilter
	 * 按照AspectJ表达式
	 */
	ASPECTJ,

	/**
	 * Filter candidates matching a given regex pattern.
	 * @see org.springframework.core.type.filter.RegexPatternTypeFilter
	 * 按照正则表达式
	 */
	REGEX,

	/** Filter candidates using a given custom
	 * {@link org.springframework.core.type.filter.TypeFilter} implementation.
	 * 自定义规则
	 */
	CUSTOM

}
```

#### 5、自定义规则举例

**返回false，不会添加到容器：**

```java
public class MyTypeFilter implements TypeFilter {
    /**
     * @param metadataReader            读取到的当前类正在扫描的类的信息
     * @param metadataReaderFactory     可以获取的其他任何类的信息
     * @return
     * @throws IOException
     */
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        // 获取当前扫描的类的注解信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        // 获取当前扫描的类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        // 获取当前扫描的类的资源（类的路径）
        Resource resource = metadataReader.getResource();
        String className = classMetadata.getClassName();
        System.out.println("==========================>>> " + className);
        return false;
    }
}

@Configuration
@ComponentScan(value = "com.gnehcnij", includeFilters = {
        @ComponentScan.Filter(type = FilterType.CUSTOM, classes = {MyTypeFilter.class})
}, useDefaultFilters = false)
public class MainConfig {

    @Bean
    public Person person666() {
        return new Person("lisi01", 22);
    }

    @Bean(value = "person")
    public Person getPerson() {
        return new Person("lisi02", 22);
    }
}

@Test
public void test03() {
    ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig.class);
    for (String definitionName : ioc.getBeanDefinitionNames()) {
        System.out.println(definitionName);
    }
    // 输出：
    // ==========================>>> com.gnehcnij.test.MainTest
    // ==========================>>> com.gnehcnij.beans.Person
    // ==========================>>> com.gnehcnij.controller.HelloController
    // ==========================>>> com.gnehcnij.dao.HelloDao
    // ==========================>>> com.gnehcnij.filter.MyTypeFilter
    // ==========================>>> com.gnehcnij.service.HelloService
    // org.springframework.context.annotation.internalConfigurationAnnotationProcessor
    // org.springframework.context.annotation.internalAutowiredAnnotationProcessor
    // org.springframework.context.annotation.internalCommonAnnotationProcessor
    // org.springframework.context.event.internalEventListenerProcessor
    // org.springframework.context.event.internalEventListenerFactory
    // mainConfig
    // person666
    // person
}
```

**自定义全类名包含"er"的添加到spring容器中：**

```java
public class MyTypeFilter implements TypeFilter {
    /**
     * @param metadataReader            读取到的当前类正在扫描的类的信息
     * @param metadataReaderFactory     可以获取的其他任何类的信息
     * @return
     * @throws IOException
     */
    public boolean match(MetadataReader metadataReader, MetadataReaderFactory metadataReaderFactory) throws IOException {
        // 获取当前扫描的类的注解信息
        AnnotationMetadata annotationMetadata = metadataReader.getAnnotationMetadata();
        // 获取当前扫描的类的类信息
        ClassMetadata classMetadata = metadataReader.getClassMetadata();
        // 获取当前扫描的类的资源（类的路径）
        Resource resource = metadataReader.getResource();
        String className = classMetadata.getClassName();
        System.out.println("==========================>>> " + className);
        return className.contains("er");
    }
}

@Configuration
@ComponentScan(value = "com.gnehcnij", includeFilters = {
        @ComponentScan.Filter(type = FilterType.CUSTOM, classes = {MyTypeFilter.class})
}, useDefaultFilters = false)
public class MainConfig {

    @Bean
    public Person person666() {
        return new Person("lisi01", 22);
    }

    @Bean(value = "person")
    public Person getPerson() {
        return new Person("lisi02", 22);
    }
}

@Test
public void test03() {
    ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig.class);
    for (String definitionName : ioc.getBeanDefinitionNames()) {
        System.out.println(definitionName);
    }
    // 输出：
    // ==========================>>> com.gnehcnij.test.MainTest
    // ==========================>>> com.gnehcnij.beans.Person
    // ==========================>>> com.gnehcnij.controller.HelloController
    // ==========================>>> com.gnehcnij.dao.HelloDao
    // ==========================>>> com.gnehcnij.filter.MyTypeFilter
    // ==========================>>> com.gnehcnij.service.HelloService
    // org.springframework.context.annotation.internalConfigurationAnnotationProcessor
    // org.springframework.context.annotation.internalAutowiredAnnotationProcessor
    // org.springframework.context.annotation.internalCommonAnnotationProcessor
    // org.springframework.context.event.internalEventListenerProcessor
    // org.springframework.context.event.internalEventListenerFactory
    // mainConfig
    // person
    // helloController
    // myTypeFilter
    // helloService
    // person666
}
```