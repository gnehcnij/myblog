# 2. Service Discovery: Eureka Server

### [ 2.1. How to Include Eureka Server](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#netflix-eureka-server-starter)

**引入依赖：**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

**注意：** 如果您的项目已经使用Thymeleaf作为模板引擎，则可能无法正确加载Eureka Server 的Freemarker模板。在这种情况下，需要手动配置模板加载器：

**application.yml**

```yaml
spring:
  freemarker:
    template-loader-path: classpath:/templates/
    prefer-file-system-access: false
```

### [2.2. How to Run a Eureka Server](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-running-eureka-server)

以下示例显示了最小的Eureka server：

```java
@SpringBootApplication
@EnableEurekaServer
public class Application {

    public static void main(String[] args) {
        new SpringApplicationBuilder(Application.class).web(true).run(args);
    }

}
```

服务server 有一个带有UI和HTTP API endpoints 的主页，在`/Eureka/*`下可访问正常Eureka功能。

### [ 2.3. High Availability, Zones and Regions](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-eureka-server-zones-and-regions)

Eureka server没有后端存储，但是注册中心中的服务实例都必须发送心跳信号以保持其注册是最新的（所以这可以在内存中完成），clients 还有一个Eureka注册的内存缓存（因此他们不必为每个服务请求都去注册中心）。

默认情况下，每个Eureka server也是Eureka client，需要（至少一个）服务URL来定位对等方。如果您不提供它，服务将运行并正常工作，但它会让您的日志中充满许多无法向对等方注册的噪音。

### [2.4. Standalone Mode](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-eureka-server-standalone-mode)

可理解为单机版。

两个缓存（client和server）和心跳信号的组合使得一个独立的Eureka server 对故障相当有弹性，只要有某种监视器或弹性运行时（如Cloud Foundry）使其保持活动。在独立模式下，您可能更喜欢关闭client行为，这样它就不会一直尝试，也不会联系不到它的对等方。下面的示例演示如何关闭client行为：

```yaml
server:
  port: 8761

eureka:
  instance:
    hostname: localhost
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

**注意**，`serviceUrl`指向的主机与本地实例相同。

### [ 2.5. Peer Awareness](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-eureka-server-peer-awareness)

通过运行多个实例并要求它们彼此注册，Eureka可以变得更具弹性和可用性。实际上，这是默认行为，因此要使其正常工作，只需向对等方添加有效的`serviceUrl`，如下例所示：

**application.yml (Two Peer Aware Eureka Servers)**

```yaml
---
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1
  client:
    serviceUrl:
      defaultZone: https://peer2/eureka/

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2
  client:
    serviceUrl:
      defaultZone: https://peer1/eureka/
```

在前面的例子中，我们有一个YAML文件，可以通过在不同的Spring profiles中运行它来在两台主机（`peer1`和`peer2`）上运行同一台服务器。通过操纵`/etc/hosts`来解析主机名，可以使用此配置在单个主机上测试对等感知（在生产环境中这样做没有多大价值）。事实上，如果在知道自己主机名的计算机上运行，则不需要`eureka.instance.hostname`（默认情况下，使用`java.net.InetAddress`查找）。

您可以将多个对等点添加到系统中，并且，只要它们都通过至少一个边缘彼此连接，它们就可以在它们之间同步注册。如果对等点在物理上是分离的（在一个数据中心内或多个数据中心之间），那么系统原则上可以经受“split-brain”类型的故障。您可以将多个对等点添加到一个系统中，只要它们彼此直接连接，它们就会在它们之间同步注册。

**application.yml (Three Peer Aware Eureka Servers)**

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: https://peer1/eureka/,http://peer2/eureka/,http://peer3/eureka/

---
spring:
  profiles: peer1
eureka:
  instance:
    hostname: peer1

---
spring:
  profiles: peer2
eureka:
  instance:
    hostname: peer2

---
spring:
  profiles: peer3
eureka:
  instance:
    hostname: peer3
```

### [2.6. When to Prefer IP Address](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#spring-cloud-eureka-server-prefer-ip-address)

在某些情况下，Eureka最好公布服务的IP地址，而不是主机名。将`eureka.instance.preferIpAddress`设置为`true`，当应用程序向eureka注册时，它将使用其IP地址而不是主机名。

**注意：**设置主机名的唯一方法是设置`eureka.instance.hostname`属性。您可以在运行时使用环境变量来设置主机名，例如，`eureka.instance.hostname = $ {HOST_NAME}`。

### [2.7. Securing The Eureka Server](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#securing-the-eureka-server)

只需通过`spring-boot-starter-security`将Spring Security添加到服务器的类路径中，即可保护Eureka服务器。默认情况下，当Spring Security位于类路径上时，它将要求在每次向应用程序发送请求时都发送有效的CSRF token 。Eureka客户通常不会拥有有效的跨站点请求伪造（CSRF）token ，您需要为/ eureka / **端点禁用此要求。例如：

```java
@EnableWebSecurity
class WebSecurityConfig extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        http.csrf().ignoringAntMatchers("/eureka/**");
        super.configure(http);
    }
}
```

A demo Eureka Server can be found in the Spring Cloud Samples [repo](https://github.com/spring-cloud-samples/eureka/tree/Eureka-With-Security).

### [2.8. Disabling Ribbon with Eureka Server and Client starters](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#disabling-ribbon-with-eureka-server-and-client-starters)

`spring-cloud-starter-netflix-eureka-server`和`spring-cloud-starter-netflix-eureka-client`附带有`spring-cloud-starter-netflix-ribbon`。由于Ribbon load-balancer现在处于维护模式，我们建议改用Eureka starters，其也包括了Spring Cloud LoadBalancer。

为此，可以将`spring.cloud.loadbalancer.ribbon.enabled`属性的值设置为`false`。

然后，您还可以在构建文件中从Eureka starters中排除功能区相关的依赖项，如下所示：

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    <exclusions>
        <exclusion>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-ribbon</artifactId>
        </exclusion>
        <exclusion>
            <groupId>com.netflix.ribbon</groupId>
            <artifactId>ribbon-eureka</artifactId>
        </exclusion>
    </exclusions>
</dependency>
```

### [2.9. JDK 11 Support](https://cloud.spring.io/spring-cloud-static/spring-cloud-netflix/2.2.2.RELEASE/reference/html/#jdk-11-support)

在JDK 11中删除了Eureka服务器所依赖的JAXB模块。如果打算在运行Eureka server时使用JDK 11，则必须在POM或Gradle文件中包括这些依赖项。

```xml
<dependency>
    <groupId>org.glassfish.jaxb</groupId>
    <artifactId>jaxb-runtime</artifactId>
</dependency>
```

