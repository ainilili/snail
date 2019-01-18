## 定义
责任链模式旨在对一些数据做流水线式处理，在整个过程能保证流水线中每个工作者都可以接触到目标，根据自我匹配度来决定当前工作者是否处理目标。

责任链模式在实际场景中运用颇多，例如日志级别就属于其中一个典型的例子，这种模式对外屏蔽了复杂的链路逻辑，提供一个简单的入口供外界调用，开发阶段会复杂一些，但是对于扩展则及其简便。

对于链式的实现多为一个元素内部指针指向下一个元素，不过对于链式这种抽象的概念，使用一个链表存储也未必不可。
## 实现
**场景**：模拟Http报文处理

报文载体：
```java
public class Request {

    private String url;

    private String type;

    private List<Map<String, String>> headers;
```

责任链抽象类，内部实现工作者的添加和串联：
```java
public abstract class ChainHandler {

    private final static LinkedList<ChainHandler> CHAIN_HANDLERS = new LinkedList<ChainHandler>();

    private ChainHandler next;

    public final static void add(ChainHandler handler) {
        if(! CHAIN_HANDLERS.isEmpty()) {
            CHAIN_HANDLERS.getLast().next = handler;
        }
        CHAIN_HANDLERS.add(handler);
    }

    public final static Request handler(String payload) {
        return CHAIN_HANDLERS.getFirst().handler(new Request(), payload);
    }

    static {
        add(new InfoChainHandler());
        add(new HeaderChainHandler());
    }

    public abstract Request handler(Request request, String payload);

    protected String[] lines(String payload) {
        return payload.split("\n");
    }

    protected Request next(Request request, String payload) {
        if(next != null) {
            return next.handler(request, payload);
        }else {
            return request;
        }
    }
}
```

报文第一行信息处理类：
```java
public class InfoChainHandler extends ChainHandler{

    @Override
    public Request handler(Request request, String payload) {
        String[] lines = lines(payload);

        String[] infos = lines[0].split(" ");
        request.setType(infos[0]);
        request.setUrl(infos[1]);

        return next(request, payload);
    }

}
```

报文头处理类：
```java
public class HeaderChainHandler extends ChainHandler{

    @Override
    public Request handler(Request request, String payload) {
        String[] lines = lines(payload);

        List<Map<String, String>> headers = new ArrayList<>();
        for(int index = 1; index < lines.length; index ++) {
            String line = lines[index];
            String[] datas = line.split("[:]", 2);

            Map<String, String> header = new TreeMap<String, String>();
            header.put(datas[0], datas[1]);
            headers.add(header);
        }
        request.setHeaders(headers);
        return next(request, payload);
    }

}
```

测试：
```java
public class ChainTest {

    @Test
    public void test() {

        String payload = "GET /search?hl=zh-CN&source=hp&q=domety&aq=f&oq= HTTP/1.1  \n" +
                "Accept: image/gif \n" +
                "Referer: <a href=\"http://www.google.cn/\">http://www.google.cn/</a>  \n" +
                "Accept-Language: zh-cn  \n" +
                "Accept-Encoding: gzip, deflate";

        Request request = ChainHandler.handler(payload);

        System.out.println(request.getUrl());
        System.out.println(request.getType());
        request.getHeaders().forEach(System.out::println);

    }
}
```

测试结果：
```powershell
/search?hl=zh-CN&source=hp&q=domety&aq=f&oq=
GET
{Accept= image/gif }
{Referer= <a href="http://www.google.cn/">http://www.google.cn/</a>  }
{Accept-Language= zh-cn  }
{Accept-Encoding= gzip, deflate}
```

## 总结
插拔方便！
