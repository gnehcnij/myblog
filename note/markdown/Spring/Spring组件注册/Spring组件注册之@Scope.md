### Spring组件注册之@Scope

**Spring版本为5.2.2.RELEASE**

#### @Scope设置组件的作用域

```java
@Configuration
public class MainConfig2 {

    @Bean	// 作用域默认是单例的（singleton）
    public Person person() {
        return new Person("Scope", 1);
    }

}

@Test
public void test03() {
    ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig2.class);
    Object person01 = ioc.getBean("person");
    Object person02 = ioc.getBean("person");
    System.out.println(person01 == person02);	// 输出：true
}
```

#### 作用域类型：

```
1、singleton	// 单实例的（默认），ioc启动时会调用方法创建对象并放到ioc容器中，以后使用时直接从容器中拿

2、prototype // 多实例的，ioc启动时不会调用方法创建对象，每次使用时才会调用方法创建对象

3、request	// 同一次请求创建一个实例（少用）

4、session	// 同一个session创建一个实例（少用）
```

##### 1、singleton

```java
@Configuration
public class MainConfig2 {
	
    @Scope(value = "singleton")
    @Bean
    public Person person() {
        return new Person("Scope", 1);
    }

}

@Test
public void test03() {
    // ioc启动时会调用方法创建对象并放到ioc容器中，以后使用时直接从容器中拿
    ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig2.class);
    System.out.println("ioc容器创建完成");
    Object person01 = ioc.getBean("person");
    Object person02 = ioc.getBean("person");
    System.out.println(person01 == person02);
    // 输出：
    // Person的构造函数：Person(String name, int age)......
	// ioc容器创建完成
	// true
}
```

##### 2、prototype

```java
@Configuration
public class MainConfig2 {
	
    @Scope(value = "prototype")
    @Bean
    public Person person() {
        return new Person("Scope", 1);
    }

}

@Test
public void test03() {
    // ioc启动时不会调用方法创建对象，每次使用时才会调用方法创建对象
    ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig2.class);
    System.out.println("ioc容器创建完成");
    Object person01 = ioc.getBean("person");
    Object person02 = ioc.getBean("person");
    System.out.println(person01 == person02);
    // 输出：
	// ioc容器创建完成
    // Person的构造函数：Person(String name, int age)......
    // Person的构造函数：Person(String name, int age)......
	// false
}
```

##### 3、@Lazy实现singleton的懒加载

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

@Test
public void test03() {
    // ioc启动时会调用方法创建对象并放到ioc容器中，以后使用时直接从容器中拿
    ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig2.class);
    System.out.println("ioc容器创建完成");
    Object person01 = ioc.getBean("person");
    Object person02 = ioc.getBean("person");
    System.out.println(person01 == person02);
    // 输出：
	// ioc容器创建完成
    // Person的构造函数：Person(String name, int age)......
	// true
}
```