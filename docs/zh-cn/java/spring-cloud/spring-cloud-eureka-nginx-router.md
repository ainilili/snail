## 一、起始
在分布式系统的体系中，注册中心的作用及其重要，每个服务可以将自己注册到Eureka中，然后通过心跳包去实时获取注册中心的服务列表，因此达到分布式环境下的Rpc调用及负载。

但是如果使用Eureka做负载均衡，那么将会面临着一个问题：
```
如果要调整负载均衡方案，例如复杂的加权，那么整个系统就要面临着停服的尴尬。
```
那么我们能不能将负载均衡交给系统之外的中间件处理？本文就拿Spring Cloud环境来举例如何将配置Eureka Client以至将负载的主动权交给Nginx！
## 二、可行性
各服务注册在Eureka上的可识别HOST默认是本机IP，也就是**ipAddress**这个参数，各个服务从Eureka获取的服务列表大多数情况下是**ipAddress:port**的搭配形式，而Nginx通常通过**server_name**来监听**80**端口去转发各个请求，**server_name**为域名的形式，那么我们只需要想办法将注册列表的搭配变成如下方式即可：
```
ipAddress = account.xt.org
port = 80
```
也就是说从Eureka上获得的服务列表将会是**account.application.api:80**，那么这个想法到底能不能做到呢？

我们都知道Spring Cloud提供了Eureka的配置方式，配置域有两个，他们分别是**Client**和**Instance**，而关于网络方面的配置大多在后者，其中有两个参数恰好符合我们上述要求：
 - IpAddress：实例IP
 - NonSecurePort：获取该实例应该接收通信的非安全端口，在Spring Cloud中默认为服务的IP

## 三、实现

接下来我们需要将IpAddress配置成我们需要的域名，NonSecurePort配置为80（不配置默认会赋值为当前服务的端口号）
```
eureka:
  client:
    serviceUrl:
      defaultZone: @serviceUrl@
    registryFetchIntervalSeconds: 5
  instance:
    preferIpAddress: true
    ipAddress: account.xt.org
    nonSecurePort: 80
    lease-renewal-interval-in-seconds: 10
    lease-expiration-duration-in-seconds: 90
    instance-id: ${spring.cloud.client.ipAddress}:${spring.application.name}:${server.port}
```
至此，我们的服务间便可以通过域名进行RPC调用，我们可以通过Eureka提供的Rest Operations查看方才修改的服务的注册信息：
```
curl GET http://localhost:8089/eureka/apps/SERVICE-XTOKEN-ACCOUNT
RESPONSE:
<application>
    <name>SERVICE-XTOKEN-ACCOUNT</name>
    <instance>
        <instanceId>192.168.50.200:service-xtoken-account:8094</instanceId>
        <hostName>account.xt.org</hostName>
        <app>SERVICE-XTOKEN-ACCOUNT</app>
        <ipAddr>account.xt.org</ipAddr>
        <status>UP</status>
        <overriddenstatus>UNKNOWN</overriddenstatus>
        <port enabled="true">80</port>
        <securePort enabled="false">443</securePort>
        <countryId>1</countryId>
        <dataCenterInfo class="com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo">
            <name>MyOwn</name>
        </dataCenterInfo>
        <leaseInfo>
            <renewalIntervalInSecs>10</renewalIntervalInSecs>
            <durationInSecs>90</durationInSecs>
            <registrationTimestamp>1537168990103</registrationTimestamp>
            <lastRenewalTimestamp>1537171761036</lastRenewalTimestamp>
            <evictionTimestamp>0</evictionTimestamp>
            <serviceUpTimestamp>1537168990103</serviceUpTimestamp>
        </leaseInfo>
        <metadata>
            <management.port>8094</management.port>
        </metadata>
        <homePageUrl>http://account.xt.org:80/</homePageUrl>
        <statusPageUrl>http://192.168.50.200:8094/info</statusPageUrl>
        <healthCheckUrl>http://192.168.50.200:8094/health</healthCheckUrl>
        <vipAddress>service-xtoken-account</vipAddress>
        <secureVipAddress>service-xtoken-account</secureVipAddress>
        <isCoordinatingDiscoveryServer>false</isCoordinatingDiscoveryServer>
        <lastUpdatedTimestamp>1537168990103</lastUpdatedTimestamp>
        <lastDirtyTimestamp>1537168990048</lastDirtyTimestamp>
        <actionType>ADDED</actionType>
    </instance>
</application>
```
我们发现ipAddr和port两个参数已经变成了我们想要的结果！

不过这时的域名我们的网关是识别不了的，需要我们去本机的Hosts文件中配置一下：
```
## 其他配置
..
..
127.0.0.1 account.xt.org
```
然后Nginx配置一个代理即可：
```
server {
        listen       80;
        server_name  account.xt.org;

        location / {
            proxy_pass  http://127.0.0.1:8094;
        }
    }
```
