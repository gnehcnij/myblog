### Spring组件注册之FactoryBean

**Spring版本为5.2.2.RELEASE**

**使用FactoryBean给ioc容器注册bean。**

#### 1、配置类

```java
@Configuration
public class MainConfig5 {

    @Bean
    public AppleFactoryBean appleFactoryBean() {
        return new AppleFactoryBean();
    }

}
```

#### 2、自定义FactoryBean
```java
public class AppleFactoryBean implements FactoryBean<Apple> {

    @Override
    public Apple getObject() throws Exception {
        return new Apple();
    }

    @Override
    public Class<?> getObjectType() {
        return Apple.class;
    }

    @Override
    public boolean isSingleton() {
        return true;
    }

}
```
#### 3、测试类

```java
public class MainTest5 {

    @Test
    public void test03() {
        ApplicationContext ioc = new AnnotationConfigApplicationContext(MainConfig5.class);
        System.out.println("ioc容器创建完成");
        Object bean01 = ioc.getBean("appleFactoryBean");
        Object bean02 = ioc.getBean("&appleFactoryBean");
        System.out.println(bean01.getClass());
        System.out.println(bean02.getClass());
        // 输出：
        // ioc容器创建完成
		// class com.gnehcnij.beans.Apple
		// class com.gnehcnij.beans.AppleFactoryBean
    }

}
```

#### 4、说明

- 默认获取到的是工厂bean调用getObject()创建的对象
- 要获取工厂bean本身，要在id前加上&