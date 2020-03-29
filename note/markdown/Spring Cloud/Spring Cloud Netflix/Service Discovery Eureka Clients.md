# 1. Service Discovery: Eureka Clients

Eureka是Netflix服务发现Server和Client，服务Server可以配置和部署为高度可用，每个Server将注册服务的状态复制到其他Server，即可集群配置。

### [1.1. How to Include Eureka Client](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#netflix-eureka-client-starter)

**引入依赖：**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

### [1.2. Registering with Eureka](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#registering-with-eureka)

当Client向Eureka注册时，它提供有关自身的元数据，如主机、端口、健康状况指示器URL、主页和其他详细信息。Eureka从属于服务的每个实例接收heartbeat消息。如果在可配置的时间表上没有收到heartbeat，则通常会从注册表中删除该实例。

以下示例显示了一个最小的Eureka client应用程序：

```java
@SpringBootApplication
@RestController
public class Application {

    @RequestMapping("/")
    public String home() {
        return "Hello world";
    }

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

**注意**，前面的示例显示了一个普通的Spring Boot应用程序，通过在类路径上使用`spring-cloud-starter-netflix-eureka-client`，应用程序将自动注册到Eureka Server。需要配置Eureka Server得位置，如：

**application.yml**

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://localhost:8761/eureka/
```

`defaultZone`属性是区分大小写得，并且要驼峰命名，因为`serviceUrl`属性是`Map<String，String>`：

```java
/**
 * Default availability zone if none is resolved based on region.
 */
public static final String DEFAULT_ZONE = "defaultZone";
private Map<String, String> serviceUrl = new HashMap<>();
{
    this.serviceUrl.put(DEFAULT_ZONE, DEFAULT_URL);
}
```

默认的application name（即service ID）、virtual host（虚拟主机）和 non-secure port（非安全端口，取自`Environment`）分别是`${spring.application.name}`、`${spring.application.name}`和`${server.port}`。

在类路径上有*spring-cloud-starter-netflix-eureka-client*依赖，使得应用程序同时成为一个eureka“实例”（也就是说，它注册了自己）和一个“Client”（它可以查询注册表来定位其他服务）。实例行为可由由eureka.instance.*配置，`${spring.application.name}`是eureka service ID或VIP的默认值。

更过的配置可以参照 [EurekaInstanceConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaInstanceConfigBean.java) 和 [EurekaClientConfigBean](https://github.com/spring-cloud/spring-cloud-netflix/tree/master/spring-cloud-netflix-eureka-client/src/main/java/org/springframework/cloud/netflix/eureka/EurekaClientConfigBean.java) 。

通过指定以下配置：

```yaml
eureka:
  client:
    enabled: false
# 或者
spring:
  cloud:
    discovery:
      enabled: false
```

可以关闭Eureka Discovery Client。

### [1.3. Authenticating with the Eureka Server（使用Eureka Server进行身份验证）](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#authenticating-with-the-eureka-server)

如果配置了`eureka.client.serviceUrl.defaultZone`，HTTP基本身份认证会自动加入到Eureka Client。

### [1.4. Status Page and Health Indicator（状态页与健康指示器）](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#status-page-and-health-indicator)

Eureka实例的状态页和健康指示器分别默认为`/info`和`/health`，这是Spring Boot Actuator application中endpoints的默认位置。可以改变其值：

**application.yml**

```yaml
eureka:
  instance:
    statusPageUrlPath: ${server.servlet.servlet-path}/info
    healthCheckUrlPath: ${server.servlet.servlet-path}/health
```

### [1.5. Registering a Secure Application](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#registering-a-secure-application)

使用HTTPS。

### [1.6. Eureka’s Health Checks](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#eurekas-health-checks)

默认情况下，Eureka使用 client 心跳来确定 client 是否在线（UP状态）。除非有指定，否则Discovery Client不会根据Spring Boot Actuator传播应用程序当前运行状况检查状态，因此，在成功注册后，Eureka总是宣布处于“UP”状态。可以通过启用Eureka健康检查来更改此行为，从而将应用程序状态传播到Eureka，因此，其他所有应用程序都不会将流量发送到“UP”之后的其他状态的应用程序，以下示例显示如何为 client 启用运行状况检查：

**application.yml**

```yaml
eureka:
  client:
    healthcheck:
      enabled: true
```

**注意：**`eureka.client.healthcheck.enabled=true`只能在application.yml中设置，在bootstrap.yml中设置该值会导致不良的副作用，例如在Eureka中注册状态为“`UNKNOWN`”。

如果需要对运行状况检查进行更多控制，可以实现`com.netflix.appinfo.HealthCheckHandler`接口

### [1.7. Eureka Metadata for Instances and Clients](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#eureka-metadata-for-instances-and-clients)

#### [1.7.1. Using Eureka on Cloud Foundry](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-eureka-on-cloud-foundry)

#### [1.7.2. Using Eureka on AWS](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-eureka-on-aws)

#### [1.7.3. Changing the Eureka Instance ID](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#changing-the-eureka-instance-id)

Netflix Eureka实例使用与其主机名相等的ID注册（即，每个主机只有一个服务）。Spring Cloud Eureka提供了一个合理的默认值，定义如下：

`${spring.cloud.client.hostname}:${spring.application.name}:${spring.application.instance_id:${server.port}}}`

如：`myhost:myappname:8080`

通过使用Spring Cloud，可以通过在`eureka.instance.instanceId`中提供唯一标识符来覆盖此值，如下例所示：

**application.yml**

```yaml
eureka:
  instance:
    instanceId: ${spring.application.name}:${spring.application.instance_id:${random.value}}}
```

### [1.8. Using the EurekaClient](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-the-eurekaclient)

一旦应用程序是discovery client，就可以使用它从Eureka Server发现服务实例，一种方法是使用本地的`com.netflix.discovery.EurekaClient`（与Spring Cloud DiscoveryClient相反），如下例所示：

```java
@Autowired
private EurekaClient discoveryClient;

public String serviceUrl() {
    InstanceInfo instance = discoveryClient.getNextServerFromEureka("STORES", false);
    return instance.getHomePageUrl();
}
```

默认情况下，EurekaClient使用Jersey进行HTTP通信。如果您希望避免来自`Jersey`的依赖项，可以将其从依赖项中排除。SpringCloud会自动配置Spring RestTemplate。以下示例显示排除了`Jersey`：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <exclusions>
        <exclusion>
            <groupId>com.sun.jersey</groupId>
            <artifactId>jersey-client</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.sun.jersey</groupId>
            <artifactId>jersey-core</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.sun.jersey.contribs</groupId>
            <artifactId>jersey-apache-client4</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### [1.9. Alternatives to the Native Netflix EurekaClient](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#alternatives-to-the-native-netflix-eurekaclient)

您不需要使用原始Netflix EurekaClient，另外，在某种包装器后面使用它通常更方便。Spring Cloud通过逻辑Eureka服务标识符（VIPs）而不是物理URLs支持Feign（REST client 构建器）和Spring RestTemplate。要使用物理服务器的固定列表配置Ribbon，可以将`<client>.Ribbon.listOfServers`设置为物理地址（或主机名）的逗号分隔列表，其中`<client>` 是 client 的ID。

您还可以使用`org.springframework.cloud.client.discovery.DiscoveryClient`，它为discovery clients提供了一个简单的API（不是Netflix特有的），如下例所示：

```java
@Autowired
private DiscoveryClient discoveryClient;

public String serviceUrl() {
    List<ServiceInstance> list = discoveryClient.getInstances("STORES");
    if (list != null && list.size() > 0 ) {
        return list.get(0).getUri();
    }
    return null;
}
```

### [ 1.10. Why Is It so Slow to Register a Service?](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#why-is-it-so-slow-to-register-a-service)

一个实例还涉及到一个到registry（注册中心）的周期性心跳（通过 client 的`serviceUrl`），默认持续时间为30秒。在实例、server和client的本地缓存中都有相同的元数据之前，clients无法发现服务（因此可能需要3次心跳）。您可以通过设置`eureka.instance.leaseRenewalIntervalInSeconds`来更改周期，将其设置为小于30的值将加快clients连接到其他服务的过程。在生产中，可能最好坚持使用默认值，因为服务器中的内部计算会对租约续订期进行假设。

### [1.11. Zones](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#zones)

如果您已经将Eureka clients 部署到多个区域，那么在尝试在另一个区域中的服务之前，您可能希望这些客户机使用同一区域中的服务。要设置它，您需要正确配置Eureka clients 。

首先，您需要确保在每个区域部署了Eureka servers，并且它们彼此是对等的。

接下来，你需要告诉Eureka你的服务在哪个区域，可以使用`metadataMap`属性来执行此操作。例如，如果`service 1`同时部署到`zone 1`和`zone 2`，则需要在service 1中设置以下Eureka属性：

**Service 1 in Zone 1**

```properties
eureka.instance.metadataMap.zone = zone1
eureka.client.preferSameZoneEureka = true
```

**Service 1 in Zone 2**

```properties
eureka.instance.metadataMap.zone = zone2
eureka.client.preferSameZoneEureka = true
```

### [1.12. Refreshing Eureka Clients](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#refreshing-eureka-clients)

默认情况下，Eureka client bean是可刷新的，这意味着可以更改和刷新Eureka client属性。当刷新发生时，client将从Eureka server 注销，并且可能会有一段短暂的时间，给定服务的所有实例都不可用。消除这种情况的一种方法是禁用刷新Eureka客户端的功能。为此，请设置*eureka.client.refresh.enable=false*。

### [1.13. Using Eureka with Spring Cloud LoadBalancer](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#using-eureka-with-spring-cloud-loadbalancer)

我们提供对Spring Cloud LoadBalancer `ZonePreferenceServiceInstanceListSupplier` 的支持。来自Eureka实例元数据（`Eureka.instance.metadataMap.zone`）的 `zone` 值，用于设置用于按区域筛选服务实例的`spring-clod-loadbalancer-zone`属性的值。

如果缺少该标志，并且`spring.cloud.loadbalancer.eureka.approximateZoneFromHostname`标志设置为`true`，则它可以使用服务器主机名中的域名作为 `zone` 的代理。

如果没有其他 zone 数据源，则根据客户端配置（与实例配置相反）进行猜测。我们采用`eureka.client.availabilityZones`，它是从区域名称到区域列表的映射，并为实例自己的区域（即`eureka.client.region`，默认为“us-east-1”，用于与本地 Netflix 兼容）拉出第一个区域。