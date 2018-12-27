## 何为权限管理
权限管理已经不知不觉深入到了我们生活的每一个角落，例如地铁进站的闸机，高速公路上的过路费，停车场的杠杆等等等等。

作为一名开发人员，权限二字对我们的映像更加深刻，无论任何系统，都多多少少与权限管理会沾上关系！什么？你的系统和权限不沾边......好吧，你的代码拉取权限总得有吧！如果还没有的话，你登上掘金看到这篇文章并点了一个赞这个过程就需要好多次权限校验。好了扯远了，我们回归正题，这里使用一张图来简单展示web系统的权限是什么样子：

![](https://github.com/ainilili/snail/blob/master/docs/images/defender-1-1.jpg?raw=true)

看完之后，是不是感觉很简单，不错，权限管理并不难，我们只需要将**校验**这一环节进行开发即可，实现方式也有很多种：
### 方案一：组件封装
我们可以将权限校验的整个过程组件化，例如组件名为``AuthComponent``，之后再所有需要权限校验的接口对应的方法的开头，调用``AuthComponent``的``verify``方法进行校验，根据结果做不同的业务处理！
 - 优点：貌似能达到主要目的，而且非常灵活~
 - 缺点：代码冗余，低内聚，高耦合，不易于维护。

### 方案二：通用处理
在方案一的基础上我们稍加改造，例如使用``AOP``对需要校验的接口做个切面，在方法执行前我们使用``AuthComponent``校验一下即可，这样我们的代码就更方便维护了！
 - 优点：弥补了方案一的缺点。
 - 缺点：太过通用化，很难兼容所有的情况，不灵活。

### 方案三：自定义注解
我们将方案一和方案二结合一下，取一灵活，取二通用，我们自定义一个名为``@Auth``的注解，并且它需要传一个参数，我们这里直径定义为枚举类``Level``，简单结构如下：
```java
public enum Level { LOGIN, ADMIN }
```
之后我们定义一个注解切面，切向携带``@Auth``的方法，在方法执行前根据value值的内容，也就是``Level``的值去做不同的权限管理即可。
 - 优点：灵活可控，通用性还行。
 - 缺点：缺乏组件化，结构零散，不易复用，对于多种场景下需要制定多个切面，不优雅！

### 方案四：使用框架
这是最简单的方法，例如优秀的开源``shiro``、``spring-Security``等都可以满足我们的需求，唯一的区别是框架的轻重及使用方式！

## 如何更优雅的管理权限
想必很多同学都在使用第四种方案，也有不少的小伙伴在使用方案三，对于缺点明显的方案一和二，使用的应该很少。

如果我们的服务并不需要那么重的权限管理框架去解决权限问题，又不想不优雅的自定义注解去实现时，我们该怎么办呢？

不妨试试 [Defender](https://github.com/ainilili/defender)
### Defender是什么
``defender``是一款全面拥抱``spring-boot``的轻量级，高灵活，高可用的权限框架。如果日常中我们需要更加优雅的对服务增加权限管理，那么``defender``正合适！

![](https://github.com/ainilili/snail/blob/master/docs/images/defender-1-2.jpg?raw=true)

它可以免除我们重复编写自定义注解和切面，只需要调用简单的API即可灵活的指定不同模式的防御网络。
### 为何优雅
``defender``提供小巧灵活的API去制定你想要的权限过滤网络，提供很多种防御模式，我们可以通过调用简单的api是使用构建不同模式的校验器，从而迅速完成权限的管理：
```java
@Configuration
@EnableDefender("* org.nico.trap.controller..*.*(..)")
public class DefenderTestConfig {
	@Bean
	public Defender init(){
		return Defender.getInstance()
				.registry(Guarder.builder(GuarderType.URI)
						.pattern("POST /user/*")
						.preventer(caller -> {
							return caller.getRequest().getHeader("token") == null
								? Result.pass() : Result.notpass("error");
						}))
				.ready();
	}
}
```
上述代码的作用是对请求符合``POST``类型且URI前缀为``/user/``的所有接口做了权限管理。

另外，我们可以使用``lambda``简单完成权限校验逻辑，又或者使用匿名类实现复杂校验逻辑：
```java
Guarder.builder(GuarderType.ANNOTATION)
		.pattern("org.nico.trap.controller")
		.preventer(new AbstractPreventer() {

			@Autowired
			private AuthComponent authComponent;

			@Override
			public Result detection(Caller caller) {
				String identity = caller.getAccess().value();
				if(! identity.equals(AuthConst.VISITOR)) {
					UserBo user = authComponent.getUser();
					if(user != null) {
						if(identity.equals(AuthConst.ADMIN)){
							if(user.getRuleType() == null) {
								return Result.notpass(new ResponseVo<>(ResponseCode.ERROR_ON_USER_IDENTITY_MISMATCH));
							}
						}
					}else {
						return Result.notpass(new ResponseVo<>(ResponseCode.ERROR_ON_LOGIN_INVALID));
					}
				}
				return Result.pass();
			}
		})
```
其中``GuarderType.URI``和``GuarderType.ANNOTATION``分别代表URI ANT匹配模式和注解模式，后者是方案三的实现，``defender``提供简单优雅的api将各种模式的权限校验方式集合在一起。

相比``shiro``、``spring-security``，``defender``显得更加轻便灵活，因为它并没有提供一系列权限更具体的管理实现，而是将校验的实现开放一个接口面向开发者，总体代码大小不超过``21k``，显然对于轻量级的权限管理，``defender``更加适合！

### 各种传送门
``defender``刚刚起步，如果大家有兴趣可以将之集成在您的开发环境尝一下鲜，项目地址如下

 > [Defender传送门](https://github.com/ainilili/defender)

官方也提供有简单的使用文档

 > [中文文档](https://github.com/ainilili/defender/blob/master/DOC_CN.md)

 > [English Document](https://github.com/ainilili/defender/blob/master/DOC_EN.md)

如果您感觉不错，也想参与贡献

 > [如何贡献](https://github.com/ainilili/defender/blob/master/CONTRIBUTING.md)
