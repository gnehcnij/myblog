### Spring组件注册之@Bean

**Spring版本为5.2.2.RELEASE**

**@Bean**等同于在spring配置文件的<bean></bean>标签，如：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <!-- 给ioc容器注册一个bean，id为person -->
    <bean id="person" class="com.gnehcnij.beans.Person">
        <property name="name" value="zhangsan"/>
        <property name="age" value="21"/>
    </bean>
</beans>
```

**等同于以下代码：**

```java
// 配置类 == 配置文件
@Configuration	// 告诉spring这是一个配置类
public class MainConfig {
    @Bean						// 给ioc容器注册一个bean，id默认为方法名
    public Person person666() {
        return new Person("lisi01", 22);
    }
    
    @Bean(value = "person")		// 给ioc容器注册一个bean，id也可以显式指定
    public Person getPerson() {
        return new Person("lisi02", 22);
    }
}
```

**测试：**

```java
public class MainTest {
    @Test
    public void test01() {
        ApplicationContext ioc = new ClassPathXmlApplicationContext("classpath:beans.xml");
        Object person = ioc.getBean("person");
        System.out.println(person);					// 输出：Person{name='zhangsan', age=21}
    }

    @Test
    public void test02() {
        ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig.class);
        Object person666 = ioc.getBean("person666");
        System.out.println(person666);				// 输出：Person{name='lisi01', age=22}
        
        Object person = ioc.getBean("person");
        System.out.println(person);					// 输出：Person{name='lisi02', age=22}
    }
}
```

**Person类：**

```java
public class Person {

	private String name;
    private int age;

    public Person() {}
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person{" +
                "name='" + name + '\'' +
                ", age=" + age +
                '}';
    }
}
```
