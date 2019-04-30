# 简介

　　Zuul 的核心逻辑是由一系列紧密配合工作的 Filter 来实现的，它们能够在进行 HTTP 请求或者响应的时候执行相关操作。可以说，没有 Filter 责任链，就没有如今的 Zuul，更不可能构成功能丰富的`网关`。基本上你想要在网关实现的功能都要与 Filter 有关。它是 Zuul 中最为开放与核心的功能。 Zuul Filter 的主要特性有以下几点：

* **Filter 的类型**：Filter 的类型决定了此 Filter 在 Filter 链中的执行顺序。可能是路由动作发生前，可能是路由动作发生时，可能是路由动作发生后，也可能是路由过程发生异常时。
* **Filter 的执行顺序**：同一种类型的 Filter 可以通过 `flterOrder()` 方法来设定执行顺序。一般会根据业务的执行顺序需求，来设定自定义 Filter 的执行顺序。
* **Filter 的执行条件**：Filter 运行所需要的标准或条件。
* **Filter 的执行效果**：符合某个 Filter 执行条件，产生的执行效果。

　　Zuul 内部提供了一个动态读取、编译和运行这些 Filter 的机制。Filter 之间不直接通信，在请求线程中会通过 `RequestContext` 来共享状态，它的内部是用 ThreadLocal 实现的，当然你也可以在 Filter之间使用 ThreadLocal 来收集自己需要的状态或数据。

# Filter 类型

　　Zuul 中不同类型 filter 的执行逻辑核心在 com.netflix.zuul.http.ZuulServlet 类中定义，该类相关代码如下：

```
@Override
public void service(javax.servlet.ServletRequest servletRequest, javax.servlet.ServletResponse servletResponse) throws ServletException, IOException {
    try {
        init((HttpServletRequest) servletRequest, (HttpServletResponse) servletResponse);

        // Marks this request as having passed through the "Zuul engine", as opposed to servlets
        // explicitly bound in web.xml, for which requests will not have the same data attached
        RequestContext context = RequestContext.getCurrentContext();
        context.setZuulEngineRan();

        try {
            preRoute();
        } catch (ZuulException e) {
            error(e);
            postRoute();
            return;
        }
        try {
            route();
        } catch (ZuulException e) {
            error(e);
            postRoute();
            return;
        }
        try {
            postRoute();
        } catch (ZuulException e) {
            error(e);
            return;
        }

    } catch (Throwable e) {
        error(new ZuulException(e, 500, "UNHANDLED_EXCEPTION_" + e.getClass().getName()));
    } finally {
        RequestContext.getCurrentContext().unset();
    }
}
```

![流程图](http://img.lynchj.com/4b8817851e2c42b3aeba2d0d59dbe6f6.png)

　　这张经典的官方流程图有些问题，其中 post Filter 抛错之后进入 error Filter，然后再进入 post Filter 是有失偏颇的。实际上 post Filter 抛错分两种情况：

1. 在 post Filter 抛错之前，pre、route Filter 没有抛错，此时会进入 ZuulException 的逻辑，打印堆栈信息，然后再返回 status = 500 的 ERROR 信息。
2. 在 post Filter 抛错之前，pre、route Filter 已有抛错，此时不会打印堆栈信息，直接返回status = 500 的 ERROR 信息。

　　也就是说，整个责任链流程终点不只是 post Filter，还可能是 error Filter，这里重新整理了一下，如图：

![流程图](http://img.lynchj.com/97ce23c9a3904d7dabd00f1ca172d862.png)

　　这样就比较直观地描述了 Zuul 关于 Filter 的请求生命周期。Zuul 中一共有四种不同生命周期的 Filter，分别是：

* **pre**：在 Zuul 按照规则路由到下级服务之前执行。如果需要对请求进行预处理，比如鉴权、限流等，都应考虑在此类 Filter 实现。
* **route**：这类 Filter 是 Zuul 路由动作的执行者，是 Apache Http Client 或 Netflix Ribbon 构建和发送原始 HTTP 请求的地方，目前已支持 Okhttp。
* **post**：这类 Filter 是在源服务返回结果或者异常信息发生后执行的，如果需要对返回信息做一些处理，则在此类 Filter 进行处理。
* **error**：在整个生命周期内如果发生异常，则会进入 error Filter，可做全局异常处理。

　　在实际项目中，往往需要自实现以上类型的 Filter 来对请求链路进行处理，根据业务的需求，选取相应生命周期的 Filter 来达成目的。在 Filter 之间，通过 `com.netflix.zuul.context.RequestContext` 类来进行通信，内部采用 `ThreadLocal` 保存每个请求的一些信息，包括请求路由、错误信息、HttpServletRequest、HttpServletResponse，这使得一些操作是十分可靠的，它还扩展了 ConcurrentHashMap，目的是为了在处理过程中保存任何形式的信息。

# Zuul 原生 Filter

　　官方文档提到, Zuul Server 如果使用 `@EnableZuulProxy` 注解搭配 Spring Boot Actuator，会多两个管控端点（**注意要开启对应的端点**）。

1. `/routes`

* **http://localhost:18000/actuator/routes**

![routes](http://img.lynchj.com/6843f3d58e5b48b089269f9ec0924a9d.png)

* **http://localhost:18000/actuator/routes/details**

![routes details](http://img.lynchj.com/f06b3864443e48bb8e0ffa547f454f54.png)

2. `/filters`

![filters](http://img.lynchj.com/94faa74bcd054556b219a54aa4c22f61.png)

　　在 `/filters` 接口中会返回很多的 Filter 信息，包括：类路径、执行顺序、是否被禁用、是否静态。可以组合成如下图：

![Filters 链](http://img.lynchj.com/22953b16a3fd4487a4ddd24bc3c69b12.png)

　　说明如下：

![Filter 功能](http://img.lynchj.com/dcb6b6fce22b42369aa86fa30f1d60a1.png)

　　以上是使用 `@EnableZuulProxy` 注解后安装的 Filter，如果使用 `@EnableZuulServer` 将缺少 `PreDecorationFilter`、`RibbonRoutingfilter`、`SimpleHostRoutingFilter`。

　　如果你不想使用原生的这些功能，可以采取替代实现的方式，覆盖掉其原生代码，也可以采取禁用策略，语法如下：

```
zuul.<SimpleClassName>.<flterType>.disable=true
```

　　比如要禁用 SendErrorFilter，在配置文件中添加 `zuul.SendErrorFilter.error.disable=true`即可。

# 自定义 Filter

```
import com.netflix.zuul.ZuulFilter;
import com.netflix.zuul.exception.ZuulException;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Component;

import static org.springframework.cloud.netflix.zuul.filters.support.FilterConstants.PRE_TYPE;

/**
 * @Author：大漠知秋
 * @Description：测试使用第一个 pre Filter
 * @CreateDate：4:34 PM 2018/10/30
 */
@Component
@Slf4j
public class FirstPreFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {

        log.info("经过第一个 pre 过滤器");

        return null;

    }

}
```

![日志](http://img.lynchj.com/f5795085f1574794bf81886569a30c22.png)

# Filter 小例

* 常量类

```
/**
 * @Author：大漠知秋
 * @Description：会话相关常量
 * @CreateDate：5:10 PM 2018/10/30
 */
public interface SessionContants {

    String LOGIC_IS_SUCCESS = "LOGIC_IS_SUCCESS";

    String ERROR_RESPONSE_BODY = "{\"status\": 10600, \"msg\":\"%s\"}";

}
```

* PreFirstFilter

```
/**
 * @Author：大漠知秋
 * @Description：测试使用第一个 pre Filter
 * @CreateDate：4:34 PM 2018/10/30
 */
@Component
@Slf4j
public class PreFirstFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {

        log.info("经过第一个 pre 过滤器");

        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        if (StringUtils.isBlank(request.getHeader("a"))) {
            // 未经过逻辑
            // 用来给后面的 Filter 标识，是否继续执行
            ctx.set(SessionContants.LOGIC_IS_SUCCESS, false);
            // 返回信息
            ctx.setResponseBody(String.format(SessionContants.ERROR_RESPONSE_BODY, "a Header头不足"));
            // 对该请求禁止路由，禁止访问下游服务
            ctx.setSendZuulResponse(false);
            return null;
        }

        // 用来给后面的 Filter 标识，是否继续执行
        ctx.set(SessionContants.LOGIC_IS_SUCCESS, true);
        return null;

    }

}
```

* PreSecondFilter

```
/**
 * @Author：大漠知秋
 * @Description：测试使用第二个 pre Filter
 * @CreateDate：4:34 PM 2018/10/30
 */
@Component
@Slf4j
public class PreSecondFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return 2;
    }

    @Override
    public boolean shouldFilter() {
        RequestContext ctx = RequestContext.getCurrentContext();
        return (boolean) ctx.get(SessionContants.LOGIC_IS_SUCCESS);
    }

    @Override
    public Object run() throws ZuulException {

        log.info("经过第二个 pre 过滤器");

        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        if (StringUtils.isBlank(request.getParameter("b"))) {
            // 未经过逻辑
            // 用来给后面的 Filter 标识，是否继续执行
            ctx.set(SessionContants.LOGIC_IS_SUCCESS, false);
            // 返回信息
            ctx.setResponseBody(String.format(SessionContants.ERROR_RESPONSE_BODY, "b 参数头不足"));
            // 对该请求禁止路由，禁止访问下游服务
            ctx.setSendZuulResponse(false);
            return null;
        }

        // 用来给后面的 Filter 标识，是否继续执行
        ctx.set(SessionContants.LOGIC_IS_SUCCESS, true);
        return null;

    }

}
```

* PostFirstFilter

```
/**
 * @Author：大漠知秋
 * @Description：测试使用第一个 post Filter
 * @CreateDate：4:34 PM 2018/10/30
 */
@Component
@Slf4j
public class PostFirstFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return POST_TYPE;
    }

    @Override
    public int filterOrder() {
        return 0;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {

        log.info("经过第一个 post 过滤器");

        RequestContext ctx = RequestContext.getCurrentContext();
        String responseBody = ctx.getResponseBody();
        if (StringUtils.isNotBlank(responseBody)) {
            // 说明逻辑没有通过
            // 设置编码
            ctx.getResponse().setContentType(MediaType.APPLICATION_JSON_UTF8_VALUE);
            ctx.getResponse().setCharacterEncoding("UTF-8");
            // 更改响应状态
            ctx.setResponseStatusCode(HttpStatus.INTERNAL_SERVER_ERROR.value());
            // 替换掉响应报文
            ctx.setResponseBody(responseBody);
        }

        return null;

    }

}
```

![结构图](http://img.lynchj.com/cfeed3efb3aa457d824cd60aca3c8210.png)

　　测试：

![测试](http://img.lynchj.com/6f49e80527c94d79b4547ef35e4d960b.gif)

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/ZuulFilter
