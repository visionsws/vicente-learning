# Spring Boot Actuator监控



### 文章来源

[Spring Boot Actuator 模块 详解：健康检查，度量，指标收集和监控](https://ricstudio.top/archives/spring_boot_actuator_learn)



## 一、什么是 Spring Boot Actuator

Spring Boot Actuator 模块提供了生产级别的功能，比如健康检查，审计，指标收集，HTTP 跟踪等，帮助我们监控和管理Spring Boot 应用。这个模块是一个采集应用内部信息暴露给外部的模块，上述的功能都可以通过HTTP 和 JMX 访问。

因为暴露内部信息的特性，Actuator 也可以和一些外部的应用监控系统整合（[Prometheus](https://prometheus.io/), [Graphite](https://graphiteapp.org/), [DataDog](https://www.datadoghq.com/), [Influx](https://www.influxdata.com/), [Wavefront](https://www.wavefront.com/), [New Relic](https://newrelic.com/)等）。这些监控系统提供了出色的仪表板，图形，分析和警报，可帮助你通过一个统一友好的界面，监视和管理你的应用程序。

Actuator使用[Micrometer](http://micrometer.io/)与这些外部应用程序监视系统集成。这样一来，只需很少的配置即可轻松集成外部的监控系统。

> Micrometer 为 Java 平台上的性能数据收集提供了一个通用的 API，应用程序只需要使用 Micrometer 的通用 API 来收集性能指标即可。Micrometer 会负责完成与不同监控系统的适配工作。这就使得切换监控系统变得很容易。
>
> 对比 Slf4j 之于 Java Logger 中的定位。

## 二、快速开始

- 对应的maven依赖：

```xml
<dependencies>
    ...
	<dependency>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-actuator</artifactId>
	</dependency>
    ...
</dependencies>
 
```

## 三、Endpoints 介绍

Spring Boot 提供了所谓的 endpoints （下文翻译为端点）给外部来与应用程序进行访问和交互。

打比方来说，`/health` 端点 提供了关于应用健康情况的一些基础信息。`metrics` 端点提供了一些有用的应用程序指标（JVM 内存使用、系统CPU使用等）。

这些 Actuator 模块本来就有的端点我们称之为原生端点。根据端点的作用的话，我们大概可以分为三大类：

- 应用配置类：获取应用程序中加载的应用配置、环境变量、自动化配置报告等与Spring Boot应用密切相关的配置类信息。
- 度量指标类：获取应用程序运行过程中用于监控的度量指标，比如：内存信息、线程池信息、HTTP请求统计等。
- 操作控制类：提供了对应用的关闭等操作类功能。

> 详细的原生端点介绍，请以[官网](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints)为准，这里就不赘述徒增篇幅。

需要注意的就是：

- 每一个端点都可以通过配置来单独禁用或者启动
- 不同于Actuator 1.x，**Actuator 2.x 的大多数端点默认被禁掉**。 Actuator 2.x 中的默认端点增加了`/actuator`前缀。默认暴露的两个端点为`/actuator/health`和 `/actuator/info`

## 四、端点暴露配置

我们可以通过以下配置，来配置通过JMX 和 HTTP 暴露的端点。

| Property                                    | Default       |
| ------------------------------------------- | ------------- |
| `management.endpoints.jmx.exposure.exclude` |               |
| `management.endpoints.jmx.exposure.include` | `*`           |
| `management.endpoints.web.exposure.exclude` |               |
| `management.endpoints.web.exposure.include` | `info, healt` |

可以打开所有的监控点

```properties
management.endpoints.web.exposure.include=*
```

也可以选择打开部分，"*" 代表暴露所有的端点，如果指定多个端点，用","分开

```properties
management.endpoints.web.exposure.include=beans,trace
```

Actuator 默认所有的监控点路径都在`/actuator/*`，当然如果有需要这个路径也支持定制。

```properties
management.endpoints.web.base-path=/minitor
```

设置完重启后，再次访问地址就会变成`/minitor/*`。

现在我们按照如下配置：

```properties
# "*" 代表暴露所有的端点 如果指定多个端点，用","分开
management:
  endpoints:
    web:
      exposure:
        include: '*'
```

启动程序，访问`http://localhost:8080/actuator`，查看暴露出来的端点：

## 五、重要端点解析

Actuator 提供了 13 个接口，具体如下表所示。

| HTTP 方法 | 路径            | 描述                                                         |
| :-------- | :-------------- | :----------------------------------------------------------- |
| GET       | /auditevents    | 显示应用暴露的审计事件 (比如认证进入、订单失败)              |
| GET       | /beans          | 描述应用程序上下文里全部的 Bean，以及它们的关系              |
| GET       | /conditions     | 就是 1.0 的 /autoconfig ，提供一份自动配置生效的条件情况，记录哪些自动配置条件通过了，哪些没通过 |
| GET       | /configprops    | 描述配置属性(包含默认值)如何注入Bean                         |
| GET       | /env            | 获取全部环境属性                                             |
| GET       | /env/{name}     | 根据名称获取特定的环境属性值                                 |
| GET       | /flyway         | 提供一份 Flyway 数据库迁移信息                               |
| GET       | /liquidbase     | 显示Liquibase 数据库迁移的纤细信息                           |
| GET       | /health         | 报告应用程序的健康指标，这些值由 HealthIndicator 的实现类提供 |
| GET       | /heapdump       | dump 一份应用的 JVM 堆信息                                   |
| GET       | /httptrace      | 显示HTTP足迹，最近100个HTTP request/repsponse                |
| GET       | /info           | 获取应用程序的定制信息，这些信息由info打头的属性提供         |
| GET       | /logfile        | 返回log file中的内容(如果 logging.file 或者 logging.path 被设置) |
| GET       | /loggers        | 显示和修改配置的loggers                                      |
| GET       | /metrics        | 报告各种应用程序度量信息，比如内存用量和HTTP请求计数         |
| GET       | /metrics/{name} | 报告指定名称的应用程序度量值                                 |
| GET       | /scheduledtasks | 展示应用中的定时任务信息                                     |
| GET       | /sessions       | 如果我们使用了 Spring Session 展示应用中的 HTTP sessions 信息 |
| POST      | /shutdown       | 关闭应用程序，要求endpoints.shutdown.enabled设置为true       |
| GET       | /mappings       | 描述全部的 URI路径，以及它们和控制器(包含Actuator端点)的映射关系 |
| GET       | /threaddump     | 获取线程活动的快照                                           |



### 5.1 `/health`端点

`/health`端点会聚合你程序的健康指标，来检查程序的健康情况。端点公开的应用健康信息取决于：

```properties
management.endpoint.health.show-details=always
```

该属性可以使用以下值之一进行配置：

| Name              | Description                                                  |
| ----------------- | ------------------------------------------------------------ |
| `never`           | 不展示详细信息，up或者down的状态，默认配置                   |
| `when-authorized` | 详细信息将会展示给通过认证的用户。授权的角色可以通过`management.endpoint.health.roles`配置 |
| `always`          | 对所有用户暴露详细信息                                       |

按照上述配置，配置成`always`之后，我们启动项目，访问`http://localhost:8080/actuator/health`端口，可以看到这样的信息：

![image.png](http://ww1.sinaimg.cn/large/005XyHIDly1gayelea4kxj31al0fy77a.jpg)

是不是感觉好像健康信息有点少？先别急，那是因为我们创建的是一个最基础的Demo项目，没有依赖很多的组件。

`/health`端点有很多自动配置的健康指示器：如redis、rabbitmq、db等组件。当你的项目有依赖对应组件的时候，这些健康指示器就会被自动装配，继而采集对应的信息。如上面的 diskSpace 节点信息就是`DiskSpaceHealthIndicator` 在起作用。



当组件有一个状态异常，应用服务的整体状态即为down。我们也可以通过配置禁用某个组件的健康监测。

```properties
management.health.mongo.enabled: false
```

或者禁用所有自动配置的健康指示器：

```properties
management.health.defaults.enabled: false
```

#### ⭐自定义 Health Indicator

当然你也可以自定义一个Health Indicator，只需要实现`HealthIndicator` 接口或者继承`AbstractHealthIndicator`类。

```java
/**
 * @author Richard_yyf
 * @version 1.0 2020/1/16
 */
@Component
public class CustomHealthIndicator extends AbstractHealthIndicator {
 
    @Override
    protected void doHealthCheck(Health.Builder builder) throws Exception {
        // 使用 builder 来创建健康状态信息
        // 如果你throw 了一个 exception，那么status 就会被置为DOWN，异常信息会被记录下来
        builder.up()
                .withDetail("app", "这个项目很健康")
                .withDetail("error", "Nothing, I'm very good");
    }
}
```

**最终效果：**

![image.png](http://ww1.sinaimg.cn/large/005XyHIDly1gayf7plncej31ai0k5wiu.jpg)

### 5.2 `/metrics`端点

`/metrics`端点用来返回当前应用的各类重要度量指标，比如：内存信息、线程信息、垃圾回收信息、tomcat、数据库连接池等。访问：`http://localhost:8080/actuator/metrics`

```json
{
    "names": [
        "tomcat.threads.busy",
        "jvm.threads.states",
        "jdbc.connections.active",
        "jvm.gc.memory.promoted",
        "http.server.requests",
        "hikaricp.connections.max",
        "hikaricp.connections.min",
        "jvm.memory.used",
        "jvm.gc.max.data.size",
        "jdbc.connections.max",
         ....
    ]
}
```

**不同于1.x，Actuator在这个界面看不到具体的指标信息，只是展示了一个指标列表。**为了获取到某个指标的详细信息，我们可以请求具体的指标信息，像这样：

```java
http://localhost:8080/actuator/metrics/{MetricName}
```

比如我访问`/actuator/metrics/jvm.memory.max`，返回信息如下：

![image.png](http://ww1.sinaimg.cn/large/005XyHIDly1gayfpdzytmj30tc0nc77r.jpg)

你也可以用query param的方式查看单独的一块区域。比如你可以访问`/actuator/metrics/jvm.memory.max?tag=id:Metaspace`。结果就是：

![image.png](http://ww1.sinaimg.cn/large/005XyHIDly1gayftsuxk1j31ad0ew0wd.jpg)

### 5.3`/loggers`端点

`/loggers` 端点暴露了我们程序内部配置的所有logger的信息。我们访问`/actuator/loggers`可以看到，

![image.png](http://ww1.sinaimg.cn/large/005XyHIDly1gayg0wp041j31ax0uigtt.jpg)

你也可以通过下述方式访问单独一个logger，

```java
http://localhost:8080/actuator/loggers/{name}
```

比如我现在访问 `root` logger，`http://localhost:8080/actuator/loggers/root`

```json
{
    "configuredLevel": "INFO",
    "effectiveLevel": "INFO"
}
```

#### ⭐改变运行时的日志等级

`/loggers`端点我最想提的就是这个功能，能够动态修改你的日志等级。

比如，我们可以通过下述方式来修改 `root` logger的日志等级。我们只需要发起一个URL 为`http://localhost:8080/actuator/loggers/root`的`POST`请求，POST报文如下：

```json
{
   "configuredLevel": "DEBUG"
}
```

![image.png](http://ww1.sinaimg.cn/large/005XyHIDly1gaygr1xntij30v6086js5.jpg)

仔细想想，这个功能是不是非常有用。**如果在生产环境中，你想要你的应用输出一些Debug信息以便于你诊断一些异常情况，你只需要按照上述方式就可以修改，而不需要重启应用。**

> 如果想重置成默认值，把value 改成 `null`

### 5.4 `/info`端点

`/info`端点可以用来展示你程序的信息。我理解过来就是一些程序的基础信息。并且你可以按照自己的需求在配置文件`application.properties`中个性化配置（默认情况下，该端点只会返回一个空的json内容。）：

```properties
info.app.name=actuator-test-demo
info.app.encoding=UTF-8
info.app.java.source=1.8
info.app.java.target=1.8
# 在 maven 项目中你可以直接用下列方式引用 maven properties的值
# info.app.encoding=@project.build.sourceEncoding@
# info.app.java.source=@java.version@
# info.app.java.target=@java.version@
```

启动项目，访问`http://localhost:8080/actuator/info`：

```json
{
    "app": {
        "encoding": "UTF-8",
        "java": {
            "source": "1.8.0_131",
            "target": "1.8.0_131"
        },
        "name": "actuator-test-demo"
    }
}
```

### 5.5 `/beans`端点

`/beans`端点会返回Spring 容器中所有bean的别名、类型、是否单例、依赖等信息。

访问`http://localhost:8080/actuator/beans`，返回如下：

![image.png](http://ww1.sinaimg.cn/large/005XyHIDly1gazbki0ot6j31ae0j0ah4.jpg)

### 5.6 `/heapdump` 端点

访问：`http://localhost:8080/actuator/heapdump`会自动生成一个 Jvm 的堆文件 heapdump。我们可以使用 JDK 自带的 Jvm 监控工具 VisualVM 打开此文件查看内存快照。

![image.png](http://ww1.sinaimg.cn/large/005XyHIDly1gazbo6t1qbj314f0lq0wb.jpg)

### 5.7 `/threaddump` 端点

这个端点我个人觉得特别有用，方便我们在日常定位问题的时候查看线程的情况。 主要展示了线程名、线程ID、线程的状态、是否等待锁资源、线程堆栈等信息。就是可能查看起来不太直观。访问`http://localhost:8080/actuator/threaddump`返回如下：

![image.png](http://ww1.sinaimg.cn/large/005XyHIDly1gazbt9bta5j31ag0skajd.jpg)

### 5.8 `/shutdown`端点

这个端点属于操作控制类端点，可以优雅关闭 Spring Boot 应用。要使用这个功能首先需要在配置文件中开启：

```properties
management.endpoint.shutdown.enabled=true
```

由于 **shutdown 接口默认只支持 POST 请求**，我们启动Demo项目，向`http://localhost:8080/actuator/shutdown`发起`POST`请求。返回信息：

```json
{
    "message": "Shutting down, bye..."
}
```

然后应用程序被关闭。

由于开放关闭应用的操作本身是一件**非常危险**的事，所以真正在线上使用的时候，我们需要对其加入一定的保护机制，比如：**定制Actuator的端点路径、整合Spring Security进行安全校验**等。（不是特别必要的话，这个端点不用开）

## 六、整合Spring Security 对端点进行安全校验

由于端点的信息和产生的交互都是非常敏感的，必须防止未经授权的外部访问。如果您的应用程序中存在**Spring Security**的依赖，则默认情况下使用**基于表单的HTTP身份验证**来保护端点。

如果没有，只需要增加对应的依赖即可：

```xml
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

添加之后，我们需要定义安全校验规则，来覆盖Spring Security 的默认配置。

这里我给出了两个版本的模板配置：

```java
import org.springframework.boot.actuate.autoconfigure.security.servlet.EndpointRequest;
import org.springframework.boot.actuate.context.ShutdownEndpoint;
import org.springframework.boot.autoconfigure.security.servlet.PathRequest;
import org.springframework.context.annotation.Configuration;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.WebSecurityConfigurerAdapter;
 
/**
 * @author Richard_yyf
 */
@Configuration
public class ActuatorSecurityConfig extends WebSecurityConfigurerAdapter {
 
    /*
     * version1:
     * 1. 限制 '/shutdown'端点的访问，只允许ACTUATOR_ADMIN访问
     * 2. 允许外部访问其他的端点
     * 3. 允许外部访问静态资源
     * 4. 允许外部访问 '/'
     * 5. 其他的访问需要被校验
     * version2:
     * 1. 限制所有端点的访问，只允许ACTUATOR_ADMIN访问
     * 2. 允许外部访问静态资源
     * 3. 允许外部访问 '/'
     * 4. 其他的访问需要被校验
     */
 
    @Override
    protected void configure(HttpSecurity http) throws Exception {
        // version1
//        http
//                .authorizeRequests()
//                    .requestMatchers(EndpointRequest.to(ShutdownEndpoint.class))
//                        .hasRole("ACTUATOR_ADMIN")
//                .requestMatchers(EndpointRequest.toAnyEndpoint())
//                    .permitAll()
//                .requestMatchers(PathRequest.toStaticResources().atCommonLocations())
//                    .permitAll()
//                .antMatchers("/")
//                    .permitAll()
//                .antMatchers("/**")
//                    .authenticated()
//                .and()
//                .httpBasic();
 
        // version2
        http
                .authorizeRequests()
                .requestMatchers(EndpointRequest.toAnyEndpoint())
                    .hasRole("ACTUATOR_ADMIN")
                .requestMatchers(PathRequest.toStaticResources().atCommonLocations())
                    .permitAll()
                .antMatchers("/")
                    .permitAll()
                .antMatchers("/**")
                    .authenticated()
                .and()
                .httpBasic();
    }
}
```

`application.properties`的相关配置如下：

```properties
# Spring Security Default user name and password
spring.security.user.name=actuator
spring.security.user.password=actuator
spring.security.user.roles=ACTUATOR_ADMIN
```



## 小结

到这里我们的Spring Boot 微服务监控告警模块也就算讲述完毕了。希望能给你带来一些收获。

对应的源码可以[Github](https://github.com/Richard-yyf/springboot-actuator-prometheus-test)上看到。





