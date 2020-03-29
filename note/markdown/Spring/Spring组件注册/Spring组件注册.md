### Spring组件注册：

#### 有以下方式：

```
1、包扫描 + 组件注解（@Controller、@Service、@Component、@Repository）
2、@Bean（可以导入第三方的组件）
3、@Import（给ioc容器快速导入组件）
	1）、@Import(要导入的组件):容器就会自动注册这个组件，id默认为组件的全类名
	2）、ImportSelector：返回需要导入的组件全类名数组
4、FactoryBean
```

#### 推荐阅读顺序：

1. Spring组件注册制@Bean
2. Spring组件注册制@ComponentScan
3. Spring组件注册制@Scope
4. Spring组件注册制@Conditional
5. Spring组件注册制@Import
6. Spring组件注册制FactoryBean