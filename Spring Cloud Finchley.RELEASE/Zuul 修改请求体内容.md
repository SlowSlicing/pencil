* 样例代码：

```
/**
 * @Author：大漠知秋
 * @Description：修改请求体内容
 * @CreateDate：4:35 PM 2018/10/31
 */
@Component
@Slf4j
public class ModifyRequestEntityFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return PRE_DECORATION_FILTER_ORDER + 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    @Override
    public Object run() throws ZuulException {
        RequestContext ctx = RequestContext.getCurrentContext();
        try {
            // 编码格式
            String charset = ctx.getRequest().getCharacterEncoding();
            InputStream in = (InputStream) ctx.get("requestEntity");
            if (null == in) {
                in = ctx.getRequest().getInputStream();
            }
            String requestEntityStr = StreamUtils.copyToString(in, Charset.forName(charset));
            requestEntityStr = URLDecoder.decode(requestEntityStr, charset);
            JSONObject requestEntityJson = JSONObject.parseObject(requestEntityStr);
            // 新增参数
            requestEntityJson.put("newParam", "1111111");
            byte[] requestEntityBytes = requestEntityJson.toJSONString().getBytes(charset);
            ctx.setRequest(new HttpServletRequestWrapper(ctx.getRequest()) {
                @Override
                public ServletInputStream getInputStream() throws IOException {
                    return new ServletInputStreamWrapper(requestEntityBytes);
                }

                @Override
                public int getContentLength() {
                    return requestEntityBytes.length;
                }

                @Override
                public long getContentLengthLong() {
                    return requestEntityBytes.length;
                }
            });
        } catch (Exception e) {
            // 用来给后面的 Filter 标识，是否继续执行
            ctx.set(SessionContants.LOGIC_IS_SUCCESS, false);
            // 返回信息
            ctx.setResponseBody(String.format(SessionContants.ERROR_RESPONSE_BODY, "修改请求体出错"));
            // 对该请求禁止路由，禁止访问下游服务
            ctx.setSendZuulResponse(false);
            log.error("【修改请求体 Filter】-出错了：{}", ExceptionUtils.getStackFrames(e));
        }
        return null;
    }

}
```
