## 一、依赖配置
Maven依赖配置只是配置中心需要的配置，其他配置自加，本文仅以扩展为目标~
#### 客户端
Maven配置
```
    <dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-starter-config</artifactId>
	</dependency>
```
Main
```
@SpringBootApplication
@EnableConfigServer
public class ConfigApp {

	public static void main(String[] args) throws Exception {
		SpringApplication.run(ConfigApp.class, args);
	}
}
```
application.yml
```
server:
  port: 8087
spring:
  cloud:
    config:
      label: master
      server:
        git:
          uri: git配置仓库http地址
          username: xxx
          password: xxx
```
#### 配置服务端
Maven配置
```
    <dependency>
		<groupId>org.springframework.cloud</groupId>
		<artifactId>spring-cloud-config-server</artifactId>
	</dependency>
```
application.yml
```
spring:
  cloud:
    config:
      profile: dev
      label: master
      uri: http://localhost:8087/
```
## 二、映射规则
```
/{application}/{profile}[/{label}]
/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.yml
/{label}/{application}-{profile}.properties
```
application：服务名
profile：环境```dev/test/pro```
lable：分支

仓库根目录下可以用```application.yml```作为全局配置，所有服务共享
