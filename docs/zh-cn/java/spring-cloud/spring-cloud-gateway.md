## Spring Cloud Gateway介绍
废话不多说，看官方文档的介绍
> This project provides an API Gateway built on top of the Spring Ecosystem, including: Spring 5, Spring Boot 2 and Project Reactor. Spring Cloud Gateway aims to provide a simple, yet effective way to route to APIs and provide cross cutting concerns to them such as: security, monitoring/metrics, and resiliency.

有道翻译一下:

这个项目提供了一个建在Spring生态系统之上的API网关，包括:Spring 5, Spring Boot 2和project Reactor。Spring Cloud Gateway旨在提供一种简单而有效的方式来路由到api，并为它们提供交叉关注，例如:安全性、监视/度量和弹性。

工作原理如下：

![这里写图片描述](https://github.com/ainilili/snail/blob/master/images/spring_cloud_gateway-1-1.jpg?raw=true)

Gateway实际上提供了一个在路由上的控制功能，大体包含两个大功能：
 - Route
 - Filter
 - Forward

我们可以通过Route去匹配请求的uri，而每个Router下可以配置一个Filter Chain，我们可以通过Filter去修饰请求和响应及一些类似鉴权等中间动作，通过Forward可以控制重定向和请求转发（实际上Filter也可以做到，只不过这里分开说清晰一点）。在路由控制上，Gateway表现的非常灵活，它有两种配置方式：
 - Yml or Properties File
 - Code

主要名词如下：
 - **Route**: Route the basic building block of the gateway. It is defined by an ID, a destination URI, a collection of predicates and a collection of filters. A route is matched if aggregate predicate is true.
 - **Predicate**: This is a Java 8 Function Predicate. The input type is a Spring Framework ServerWebExchange. This allows developers to match on anything from the HTTP request, such as headers or parameters.
 - **Filter**: These are instances Spring Framework GatewayFilter constructed in with a specific factory. Here, requests and responses can be modified before or after sending the downstream request.

Route 作为Gateway中的基本元素，它有自己的ID、URI和一个Predicate集合、Filter集合。Predicate的作用是判断请求的Uri是否匹配当前的Route，Filter则是匹配通过之后对请求和响应的处理及修饰，那么在Gateway中Route基本结构如下
```
Gateway{
    Route1 {
        String id;
        String path;
        List<Predicate> predicates;
        List<Filter> filters;
    };
    Route2 {
        String id;
        String path;
        List<Predicate> predicates;
        List<Filter> filters;
    };
    ...
    ...
}
```
Route中的ID作为它的唯一标识，path的作用是正则匹配请求路径，Predicate则是在path匹配的情况下进一步去更加细致的匹配请求路径，如一下例子：
```
spring:
  cloud:
    gateway:
      routes:
      - id: before_route
        uri: http://example.org
        predicates:
        - Before=2017-01-20T17:42:47.789-07:00[America/Denver]
```
只匹配在```Jan 20, 2017 17:42```发起的请求
```
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: http://example.org
        predicates:
        - Cookie=chocolate, “”
```
只匹配请求中携带```chocolate```且值为```ch.p```的请求

更多例子这里就不一一展开，有兴趣的可以去看下官方文档: [传送门](http://cloud.spring.io/spring-cloud-gateway/single/spring-cloud-gateway.html#gateway-request-predicates-factories)

## Spring Cloud Gateway 配置
#### Maven
```
<dependencyManagement>
		<dependencies>
			<dependency>
				<groupId>org.springframework.cloud</groupId>
				<artifactId>spring-cloud-gateway</artifactId>
				<version>2.0.2.BUILD-SNAPSHOT</version>
				<type>pom</type>
				<scope>import</scope>
			</dependency>
		</dependencies>
	</dependencyManagement>
	<dependencies>
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-gateway</artifactId>
		</dependency>
		<dependency>
		    <groupId>org.yaml</groupId>
		    <artifactId>snakeyaml</artifactId>
		</dependency>
	</dependencies>
	<repositories>
		<repository>
			<id>spring-snapshots</id>
			<name>Spring Snapshots</name>
			<url>https://repo.spring.io/libs-snapshot</url>
			<snapshots>
				<enabled>true</enabled>
			</snapshots>
		</repository>
	</repositories>
```
#### Yml
```
spring:
  cloud:
    gateway:
      routes:
      - id: cookie_route
        uri: http://example.org
        predicates:
        - Cookie=chocolate, ch.p
```
#### Java Config
```
@Configuration
@RestController
@SpringBootApplication
public class Application {

	@Bean
	public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
		return builder.routes()
				.route(r -> r.path("/request/**")
						.and()
						.predicate(new Predicate<ServerWebExchange>() {
							@Override
							public boolean test(ServerWebExchange t) {
								boolean access = t.getRequest().getCookies().get("_9755xjdesxxd_").get(0).getValue().equals("32");
								return access;
							}
						})
						.filters(f -> f.stripPrefix(2)
							.filter(new GatewayFilter() {
								@Override
								public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
									return chain.filter(exchange);
								}
							}, 2)
							.filter(new GatewayFilter() {
								@Override
								public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
									return chain.filter(exchange);
								}
							}, 1))
						.uri("http://localhost:8080/hello")
						).build();
	}

	@GetMapping("/hello")
	public String hello() {
		return "hello";
	}

	public static void main(String[] args) throws ClassNotFoundException {
		SpringApplication.run(Application.class, args);
	}

}
```
Yml配置和Java代码配置可以共存，Yml配置的好处是可以直接用自带的一些谓词和Filter，而Java代码配置更加灵活！
## Spring Cloud Gateway使用
上文已经说过，Gateway支持两种配置，本文主要以Java Config的方式着重讲解，因为官方文档中对于Yml配置的讲解已经足够深入，如有兴趣可以进入[传送门](http://cloud.spring.io/spring-cloud-gateway/single/spring-cloud-gateway.html#gateway-how-it-works)

#### Route
一个Route的配置可以足够简单
```
@Configuration
@RestController
@SpringBootApplication
public class Application1 {

	@Bean
	public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
		return builder.routes()
				.route(r -> r.path("/user/**")
						.uri("http://localhost:8080/hello")
						).build();
	}

	@GetMapping("/hello")
	public String hello() {
		return "hello";
	}

	public static void main(String[] args) throws ClassNotFoundException {
		SpringApplication.run(Application1.class, args);
	}

}
```
上述Demo定义了一个Route，并且path值为```/**```，意味着匹配多层uri，如果将path改为```/*```则意味着只能匹配一层。所以运行上面的程序，那么所有的请求都将被转发到```http://localhost:8080/hello```

如果uri的配置并没有一个确定的资源，例如```http://ip:port```，那么```/**```所匹配的路径将会自动拼装在uri之后：
```
request http://当前服务/user/1
forward http://ip:port/user/1
```
这种方式更适合服务之间的转发，我们可以将uri设置为```ip:port```也可以设置为```xxx.com```域名，但是不能自己转发自己的服务，例如
```
request http://当前服务/user/1
forward http://当前服务/user/1
```
这就导致了HTTP 413的错误，无限转发至自己，也就意味着请求死锁，非常致命！最好的方式如下：

我们拟定开两个服务占用的端口分别是```8080```和```8081```，我们假如要从```8080```服务通过```/user/**```路由匹配转发至```8081```服务，可以这样做：
```
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
	return builder.routes()
			.route("hello", r -> r
					.path("/user/**")
					.and()
					.uri("http://localhost:8081")
					).build();
}
```
工作跟踪：
```
request http://localhost:8080/user/hello
forward http://localhost:8081/user/hello
```
8081服务接口定义：
```
@GetMapping("/user/hello")
public String hello() {
	return "User Say Hello";
}
```
Reponse Body：
```
User Say Hello
```

当Gateway代替Zuul时，也就是说在服务间的通讯由Zuul转换成Gateway之后，uri的写法将会变成这个样子：
```
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
	return builder.routes()
			.route(r -> r.path("user/**")
					.uri("lb://USER_SERVER_NAME")
					).build();
}
```
上述代码将```user/**```所匹配的请求全部转发到```USER_SERVER_NAME```服务下：
```
request /user/1
forward http://USER_SERVER_HOST:USER_SERVER_PORT/user/1
```
其中**lb**的含义其实是```load balance```的意思，我想开发者以lb来区分路由模式可能是负载均衡意味着多服务的环境，因此lb可以表示转发对象从指定的uri转变成了服务！
#### Predicate
Predicate是Java 8+新出的一个库，本身作用是进行逻辑运算，支持种类如下：
- isEqual
- and
- negate
- or
另外还有一个方法```test(T)```用于触发逻辑计算返回一个Boolean类型值。

Gateway使用Predicate来做除path pattern match之外的匹配判断，使用及其简单：
```
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
	return builder.routes()
			.route("hello", r -> r
					.path("/user/**")
					.and()
					.predicate(e -> e.getClass() != null)
					.uri("http://localhost:8081")
					).build();
}
```
输入的**e**代表着```ServerWebExchange```对象，我们可以通过```ServerWebExchange```获取所有请求相关的信息，例如Cookies和Headers。通过Lambda语法去编写判断逻辑，如果一个Route中所有的Predicate返回的结果都是TRUE则匹配成功，否则匹配失败。

> Tp：path和predicate需要使用and链接，也可以使用or链接，分别代表不同的逻辑运算！

#### Filter
Filter的作用类似于Predicate，区别在于，Predicate可以做请求中断，Filter也可以做，Filter可以做Reponse的修饰，Predicate并做不到，也就是说Filter最为最后一道拦截，可以做的事情有很多，例如修改响应报文，增加个Header或者Cookie，甚至修改响应Body，相比之下，Filter更加全能！
```
@Bean
public RouteLocator customRouteLocator(RouteLocatorBuilder builder) {
	return builder.routes()
			.route("hello", r -> r
					.path("/user/**")
					.and()
					.predicate(e -> e.getClass() != null)
					.filters(fn -> fn.addResponseHeader("developer", "Nico"))
					.uri("http://localhost:8081")
					).build();
}
```
## Spring Cloud Gateway 工作原理
Gateway是基于Spring MVC之上的网关路由控制器，我们可以直接定位到Spring MVC的```org.springframework.web.reactive.DispatcherHandler```类，它的```handle```方法将会处理解析Request之后的```ServerWebExchange```对象。

进入```handle```方法，将会使用Flux遍历```org.springframework.web.reactive.DispatcherHandler.handlerMappings```对```ServerWebExchange```进行处理，```handlerMappings```中包含着一下处理器：
```
org.springframework.web.reactive.function.server.support.RouterFunctionMapping@247a29b6, org.springframework.web.reactive.result.method.annotation.RequestMappingHandlerMapping@f6449f4, org.springframework.cloud.gateway.handler.RoutePredicateHandlerMapping@535c6b8b,
org.springframework.web.reactive.handler.SimpleUrlHandlerMapping@5633e9e
```
以上只是一个简单请求的处理器，Flux的```concatMap```方法会将每个处理器的Mono合并成一个Flux，然后调用```org.springframework.web.reactive.DispatcherHandler```类中的```invokeHandler```方法开始处理```ServerWebExchange```，处理完毕之后将紧接着处理返回值，这时使用```handleResult```方法，具体实现如下：
```
@Override
public Mono<Void> handle(ServerWebExchange exchange) {
	if (logger.isDebugEnabled()) {
		ServerHttpRequest request = exchange.getRequest();
		logger.debug("Processing " + request.getMethodValue() + " request for [" + request.getURI() + "]");
	}
	if (this.handlerMappings == null) {
		return Mono.error(HANDLER_NOT_FOUND_EXCEPTION);
	}
	return Flux.fromIterable(this.handlerMappings)
			.concatMap(mapping -> mapping.getHandler(exchange))
			.next()
			.switchIfEmpty(Mono.error(HANDLER_NOT_FOUND_EXCEPTION))
			.flatMap(handler -> invokeHandler(exchange, handler))
			.flatMap(result -> handleResult(exchange, result));
}
```

Gateway起作用的关键在于```invokeHandler```方法中：
```
private Mono<HandlerResult> invokeHandler(ServerWebExchange exchange, Object handler) {
	if (this.handlerAdapters != null) {
		for (HandlerAdapter handlerAdapter : this.handlerAdapters) {
			if (handlerAdapter.supports(handler)) {
				return handlerAdapter.handle(exchange, handler);
			}
		}
	}
	return Mono.error(new IllegalStateException("No HandlerAdapter: " + handler));
}
```




对应的处理器的适配器匹配到本身之后，将会触发适配器的处理方法，每个处理器都会实现一个对应的接口，他们大致都有一个共同的特点：
```
public interface Handler {

	Mono<Void> handle(ServerWebExchange exchange);

}
```
每个适配器的处理方法中都会在穿插入适配逻辑代码之后调用处理器的```handle```方法，Spring Cloud Gateway的所有Handler都在```org.springframework.cloud.gateway.handler```包下：
```
org.springframework.cloud.gateway.handler.AsyncPredicate<T>
org.springframework.cloud.gateway.handler.FilteringWebHandler
org.springframework.cloud.gateway.handler.RoutePredicateHandlerMapping
```
可以看到，上述三个处理器分别处理了Filter和Predicate，有兴趣的朋友可以去看一下这些类内部的具体实现，这里就不一一细说。

总的来讲，Spring Cloud Gateway的原理是在服务加载期间读取自己的配置将信息存放在一个容器中，在Spring Mvc 处理请求的时候取出这些信息进行**逻辑判断**及**过滤**，根据不同的处理结果触发不同的事件！

而请求转发这一块有兴趣的同学可以去研究一下！

> TP: path的匹配其实也是一个Predicate逻辑判断

## Spring Cloud Gateway 总结
笔者偶然间看到spring-cloud-netflix的issue下的一个回答[传送门](https://github.com/spring-cloud/spring-cloud-netflix/issues/2951)：
```
Lovnx：Zuul 2.0 has opened sourcing,Do you intend to integrate it in some next version?
spencergibb：No. We created spring cloud gateway instead.
```
这才感觉到Spring Cloud果然霸气，也因此接触到了Spring Cloud Gateway，觉得很有必要学习一下，总体来讲，Spring Cloud Gateway简化了之前过滤器配置的复杂度，也在新的配置方式上增加了微服务的网关配置，可以直接代替掉Zuul，期待着Spring会整出自己的注册中心来。

笔者学艺不精，以上阐述有误的地方，希望批评指出，联系方式：ainililia@163.com

## 相关文档
[官方文档](http://cloud.spring.io/spring-cloud-gateway/single/spring-cloud-gateway.html#_glossary)
[Spring Cloud Gateway 入门](http://ju.outofmemory.cn/entry/346688)
[响应式编程之Reactor的关于Flux和Mono概念](https://blog.csdn.net/qq_28089993/article/details/79443447)
