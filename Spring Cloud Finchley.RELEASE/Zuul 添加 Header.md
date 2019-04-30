Spring Cloud Finchley.RELEASE
Spring Cloud,Zuul,Header
> &emsp;&emsp;在网关层添加新的 Header 信息是很常见的，比如天剑唯一id、时间戳等。

&emsp;&emsp;如下伪代码添加 Header 头：

```
@Component
@Slf4j
public class ModifyHeaderFilter extends ZuulFilter {
......
	@Override
	public Object run() throws ZuulException {
	    RequestContext ctx = RequestContext.getCurrentContext();
	    ctx.addZuulRequestHeader("newHeader", "111111");
	    return null;
	}
.......
}
```
