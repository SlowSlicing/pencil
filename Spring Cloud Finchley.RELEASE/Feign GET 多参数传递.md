Spring Cloud Finchley.RELEASE
Spring Cloud,Feign,GET Parameter
&emsp;&emsp;Spring MVC 是支持 GET/POST 多参数传递的，但是 Feign 并没有实现所有的 Spring MVC 的功能，暂时是不支持 GET  的 POJO 传递方法。一般的解决办法有一下三种：

* 把 POJO 拆散成一个一个单独的属性放在方法参数里。
* 把方法参数变成 Map 传递。
* 使用 GET 传递 `@Requestbody`，但此方式违反 RESTFul 规范。

&emsp;&emsp;以上三种方法都是可以的，这里要使用的方法是 Feign 的 RequestInterceptor 进行统一的处理，当检测到请求方法为 GET 时，把请求体里面的内容重新拼装成请求参数的形式。

1. 消费者的 Controller

```
/**
 * @Author：大漠知秋
 * @Description：测试 FeignGetInterceptor 的 Controller
 * @CreateDate：2:38 PM 2018/10/25
 */
@RestController
@RequestMapping("/feignGet")
public class FeignGetInterceptorController {

    @Resource
    private FeignGetInterceptorFeign feignGetInterceptorFeign;

    @RequestMapping(
            value = "/addUser",
            method = RequestMethod.GET,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE
    )
    ResponseEntity<byte[]> addUser(User user) {
        return feignGetInterceptorFeign.addUser(user);
    }

    @RequestMapping(
            value = "/updateUser",
            method = RequestMethod.POST,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE
    )
    ResponseEntity<byte[]> updateUser(@RequestBody User user) {
        return feignGetInterceptorFeign.updateUser(user);
    }

}
```

2. 消费者的 Feign

```
@FeignClient(name = "demo-goods")
public interface FeignGetInterceptorFeign {

    @RequestMapping(
            value = "/feignGet/addUser",
            method = RequestMethod.GET,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE
    )
    ResponseEntity<byte[]> addUser(User user);

    @RequestMapping(
            value = "/feignGet/updateUser",
            method = RequestMethod.POST,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE
    )
    ResponseEntity<byte[]> updateUser(@RequestBody User user);

}
```

3. 提供者的 Controller

```
/**
 * @Author：大漠知秋
 * @Description：测试 FeignGetInterceptor 的 Controller
 * @CreateDate：2:38 PM 2018/10/25
 */
@RestController
@RequestMapping("/feignGet")
public class FeignGetInterceptorController {

    @RequestMapping(
            value = "/addUser",
            method = RequestMethod.GET,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE
    )
    User addUser(User user) {
        user.setAge(user.getAge() + 1);
        user.setUsername(user.getUsername() + "1");
        user.setBirthday(new Date());
        return user;
    }

    @RequestMapping(
            value = "/updateUser",
            method = RequestMethod.POST,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE
    )
    User updateUser(@RequestBody User user) {
        user.setAge(user.getAge() + 1);
        user.setUsername(user.getUsername() + "1");
        user.setBirthday(new Date());
        return user;
    }

}
```

&emsp;&emsp;没有加拦截器之前，请求 GET 方式，会报如下错误：

![报错](http://img.lynchj.com/1fa82699472c435984e46cbf6a166ffb.png)

4. 增加 Feign 拦截器

```
/**
 * @Author：大漠知秋
 * @Description：拦截使用 Get 请求，并且使用的时 POJO 请求方式的 Feign 请求
 * @CreateDate：2:24 PM 2018/10/25
 */
@Component
@Slf4j
public class FeignRequestInterceptor implements RequestInterceptor {

    @Resource
    private ObjectMapper objectMapper;

    @Override
    public void apply(RequestTemplate template) {
        if (HttpMethod.GET.name().equals(template.method())
                && null != template.body()) {
            try {
                JsonNode jsonNode = objectMapper.readTree(template.body());
                template.body(null);

                Map<String, Collection<String>> queries = new HashMap<>();
                buildQuery(jsonNode, "", queries);
                template.queries(queries);
            } catch (IOException e) {
                log.error("【拦截GET请求POJO方式】-出错了：{}", ExceptionUtils.getStackFrames(e));
                throw new RuntimeException();
            }
        }
    }

    private void buildQuery(JsonNode jsonNode, String path, Map<String, Collection<String>> queries) {
        // 叶子节点
        if (!jsonNode.isContainerNode()) {
            if (jsonNode.isNull()) {
                return;
            }
            Collection<String> values = queries.get(path);
            if (null == values) {
                values = new ArrayList<>();
                queries.put(path, values);
            }
            values.add(jsonNode.asText());
            return;
        }
        // 数组节点
        if (jsonNode.isArray()) {
            Iterator<JsonNode> it = jsonNode.elements();
            while (it.hasNext()) {
                buildQuery(it.next(), path, queries);
            }
        } else {
            Iterator<Map.Entry<String, JsonNode>> it = jsonNode.fields();
            while (it.hasNext()) {
                Map.Entry<String, JsonNode> entry = it.next();
                if (StringUtils.hasText(path)) {
                    buildQuery(entry.getValue(), path + "." + entry.getKey(), queries);
                } else {  // 根节点
                    buildQuery(entry.getValue(), entry.getKey(), queries);
                }
            }
        }
    }

}
```

5. Model

```
@Data
public class User {

    private String username;

    private Integer age;

    @DateTimeFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    @JsonFormat(pattern = "yyyy-MM-dd HH:mm:ss")
    private Date birthday;

}
```

6. 结果：

![GET 请求](http://img.lynchj.com/cb2f0fc873a4462398c063dad2212696.png)

![POST 请求](http://img.lynchj.com/a12a667bb1c548b281fe6601eba29f53.png)

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/FeignGETRequestMoreParameter
