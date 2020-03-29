# 7. Client Side Load Balancer: Ribbon

Ribbon是客户端load balancer，可让您对HTTP和TCP客户端的行为进行大量控制。Feign已经使用了Ribbon，因此，如果您使用`@FeignClient`，也同样适用。

Ribbon中的中心概念是指定客户端的概念。每个load balancer都是组件的一部分，这些组件可以一起工作以按需联系远程服务器，并且该组件具有您作为应用程序开发人员提供的名称（例如，使用`@FeignClient`注解）。根据需要，Spring Cloud通过使用`RibbonClientConfiguration`为每个命名客户端创建一个新的集合作为`ApplicationContext`。它包含（除其他事项外）一个`ILoadBalancer`，一个`RestClient`和一个`ServerListFilter`。

### [ 7.1. How to Include Ribbon](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#netflix-ribbon-starter)

**引入依赖：**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-ribbon</artifactId>
</dependency>
```

### [7.2. Customizing the Ribbon Client](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#customizing-the-ribbon-client)

您可以使用`<client> .ribbon`。*中的外部属性来配置Ribbon client的某些属性值，这与本地使用Netflix API相似，不同之处在于可以使用Spring Boot配置文件。可以将本地选项检查为`CommonClientConfigKey`（功能区核心的一部分）中的静态字段。

Spring Cloud还允许您通过使用`@RibbonClient`声明其他配置（在`RibbonClientConfiguration`之上）来完全控制client，如以下示例所示：

```java
@Configuration
@RibbonClient(name = "custom", configuration = CustomConfiguration.class)
public class TestConfiguration {
}
```

在这种情况下，client由`RibbonClientConfiguration`中已有的组件以及`CustomConfiguration`中的任何组件组成（后者通常会覆盖前者）。

**注意：** `CustomConfiguration`类必须是`@Configuration`类，但请注意，对于main application context，它不在`@ComponentScan`中。否则，它由所有`@RibbonClients`共享。如果使用`@ComponentScan`（或`@SpringBootApplication`），则需要采取措施避免将其包括在内（例如，可以将其放在单独的，不重叠的程序包中，或指定要在`@ComponentScan`中显式扫描的程序包）。

下表显示了Spring Cloud Netflix默认为Ribbon提供的bean：

|      Bean Type      |         Bean Name         |            Class Name            |
| :-----------------: | :-----------------------: | :------------------------------: |
|   `IClientConfig`   |   `ribbonClientConfig`    |    `DefaultClientConfigImpl`     |
|       `IRule`       |       `ribbonRule`        |       `ZoneAvoidanceRule`        |
|       `IPing`       |       `ribbonPing`        |           `DummyPing`            |
|    `ServerList`     |    `ribbonServerList`     |  `ConfigurationBasedServerList`  |
| `ServerListFilter`  | `ribbonServerListFilter`  | `ZonePreferenceServerListFilter` |
|   `ILoadBalancer`   |   `ribbonLoadBalancer`    |     `ZoneAwareLoadBalancer`      |
| `ServerListUpdater` | `ribbonServerListUpdater` |    `PollingServerListUpdater`    |

创建这些类型之一的bean并将其放置在`@RibbonClient`配置（例如上面的`FooConfiguration`）中，可以覆盖所描述的每个bean，如以下示例所示：

```java
@Configuration(proxyBeanMethods = false)
protected static class FooConfiguration {

    @Bean
    public ZonePreferenceServerListFilter serverListFilter() {
        ZonePreferenceServerListFilter filter = new ZonePreferenceServerListFilter();
        filter.setZone("myTestZone");
        return filter;
    }

    @Bean
    public IPing ribbonPing() {
        return new PingUrl();
    }

}
```

上一示例中\用PingUrl代替了NoOpPing，并提供了一个自定义serverListFilter。

### [7.3. Customizing the Default for All Ribbon Clients](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#customizing-the-default-for-all-ribbon-clients)

通过使用`@RibbonClients`注解并注册默认配置，可以为所有Ribbon client供默认配置，如以下示例所示：

```java
@RibbonClients(defaultConfiguration = DefaultRibbonConfig.class)
public class RibbonClientDefaultConfigurationTestsConfig {

    public static class BazServiceList extends ConfigurationBasedServerList {

        public BazServiceList(IClientConfig config) {
            super.initWithNiwsConfig(config);
        }

    }

}

@Configuration(proxyBeanMethods = false)
class DefaultRibbonConfig {

    @Bean
    public IRule ribbonRule() {
        return new BestAvailableRule();
    }

    @Bean
    public IPing ribbonPing() {
        return new PingUrl();
    }

    @Bean
    public ServerList<Server> ribbonServerList(IClientConfig config) {
        return new RibbonClientDefaultConfigurationTestsConfig.BazServiceList(config);
    }

    @Bean
    public ServerListSubsetFilter serverListFilter() {
        ServerListSubsetFilter filter = new ServerListSubsetFilter();
        return filter;
    }

}
```

### [7.4. Customizing the Ribbon Client by Setting Properties](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#customizing-the-ribbon-client-by-setting-properties)

从1.2.0版开始，Spring Cloud Netflix现在通过将属性设置为与 [Ribbon documentation](https://github.com/Netflix/ribbon/wiki/Working-with-load-balancers#components-of-load-balancer) 兼容来支持自定义Ribbon client。

这使您可以在启动时在不同环境中更改行为。

以下列表显示了受支持的属性>：

- `<clientName>.ribbon.NFLoadBalancerClassName`: 应该实现 `ILoadBalancer`
- `<clientName>.ribbon.NFLoadBalancerRuleClassName`: 应该实现 `IRule`
- `<clientName>.ribbon.NFLoadBalancerPingClassName`: 应该实现 `IPing`
- `<clientName>.ribbon.NIWSServerListClassName`: 应该实现 `ServerList`
- `<clientName>.ribbon.NIWSServerListFilterClassName`: 应该实现 `ServerListFilter`

**注意**： 这些属性中定义的类优先于使用`@RibbonClient（configuration = MyRibbonConfig.class）`定义的bean和Spring Cloud Netflix提供的默认值。

要为名为`users`的服务名称设置`IRule`，可以设置以下属性：

**application.yml**

```yaml
users:
  ribbon:
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.WeightedResponseTimeRule
```

### [7.5. Using Ribbon with Eureka](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-ribbon-with-eureka)

当Eureka与Ribbon结合使用时（也就是说，两者都在类路径上），`ribbonServerList`将被`DiscoveryDiscoveredNIWSServerList`的扩展名覆盖，该扩展名将填充Eureka中的服务器列表。它还用`NIWSDiscoveryPing`替换了`IPing`接口，该接口委托Eureka确定服务器是否启动。默认情况下安装的`ServerList`是`DomainExtractingServerList`。其目的是不使用AWS AMI元数据（这就是Netflix所依赖的）使元数据可用于load balancer。默认情况下，服务器列表是使用实例元数据中提供的“zone”信息构建的（因此，在远程客户端上，设置`eureka.instance.metadataMap.zone`）。如果缺少该字段，并且如果设置了`approximateZoneFromHostname`标志，则它可以使用服务器主机名中的域名作为该zone的代理。一旦zone信息可用，就可以在`ServerListFilter`中使用它。默认情况下，它用于在与客户端相同的zone中定位服务器，因为默认值为`ZonePreferenceServerListFilter`。默认情况下，以与远程实例相同的方式（即通过`eureka.instance.metadataMap.zone`）确定客户端的zone。

**注意：**

- 设置客户端zone的传统“ archaius”方法是通过名为“ @zone”的配置属性。如果可用，Spring Cloud会优先使用所有其他设置（请注意，key必须在YAML配置中用引号引起来）。
- 如果没有其他zone数据源，则根据client配置（而不是实例配置）进行猜测。我们使用`eureka.client.availabilityZones`（这是从region name到zone列表的映射），并为实例自己的region （即`eureka.client.region`）拉出第一个zone，默认为“ us-east-1“，以与本机Netflix兼容。

### [7.6. Example: How to Use Ribbon Without Eureka](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-ribbon-without-eureka)

Eureka是一种抽象发现远程服务器的便捷方法，因此您不必在client中对它们的URL进行硬编码。但是，如果您不想使用Eureka，Ribbon和Feign也可以使用。假设您已经为“stores”声明了`@RibbonClient`，并且Eureka未被使用（甚至不在类路径上）。Ribbon client默认为已配置的服务器列表。您可以提供以下配置：

**application.yml**

```yaml
stores:
  ribbon:
    listOfServers: example.com,google.com
```

### [ 7.7. Example: Disable Eureka Use in Ribbon](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#example-disable-eureka-use-in-ribbon)

将`ribbon.eureka.enabled`属性设置为`false`会显式禁用Ribbon中对Eureka的使用，如以下示例所示：

**application.yml**

```yaml
ribbon:
  eureka:
   enabled: false
```

### [ 7.8. Using the Ribbon API Directly](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-the-ribbon-api-directly)

您还可以直接使用`LoadBalancerClient`，如以下示例所示：

```java
public class MyClass {
    @Autowired
    private LoadBalancerClient loadBalancer;

    public void doStuff() {
        ServiceInstance instance = loadBalancer.choose("stores");
        URI storesUri = URI.create(String.format("https://%s:%s", instance.getHost(), instance.getPort()));
        // ... do something with the URI
    }
}
```

### [7.9. Caching of Ribbon Configuration](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#ribbon-child-context-eager-load)

每个命名为Ribbon的client都有一个Spring Cloud维护的相应子application context。该application context在对命名client的第一个请求上延迟加载。通过指定Ribbon clients的名称，可以更改此延迟加载行为，以代替在启动时eagerly加载这些子application context，如以下示例所示：

**application.yml**

```yaml
ribbon:
  eager-load:
    enabled: true
    clients: client1, client2, client3
```

### [ 7.10. How to Configure Hystrix Thread Pools](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#how-to-configure-hystrix-thread-pools)

如果将`zuul.ribbonIsolationStrategy`更改为`THREAD`，则Hystrix的线程隔离策略将用于所有路由。在这种情况下，`HystrixThreadPoolKey`会默认设置为`RibbonCommand`。这意味着所有路由的`HystrixCommands`在同一个Hystrix线程池中执行。可以使用以下配置更改此行为：

**application.yml**

```yaml
zuul:
  threadPool:
    useSeparateThreadPools: true	
```

前面的示例导致在Hystrix线程池中为每个路由执行`HystrixCommands`。

在这种情况下，默认的`HystrixThreadPoolKey`与每个路由的服务ID相同。要将前缀添加到`HystrixThreadPoolKey`，请将`zuul.threadPool.threadPoolKeyPrefix`设置为要添加的值，如以下示例所示：

**application.yml**

```yaml
zuul:
  threadPool:
    useSeparateThreadPools: true
    threadPoolKeyPrefix: zuulgw
```