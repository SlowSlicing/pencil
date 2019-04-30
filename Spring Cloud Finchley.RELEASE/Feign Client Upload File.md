Spring Cloud Finchley.RELEASE
Spring Cloud,Feign,Upload File
> &emsp;&emsp;早期的 Feign 是不支持文件上传的，后来支持了，但是有部分缺陷，需要一次性读取到内存中再编码发送。

1. 引入 Feign Client 文件上传依赖

```
<!-- Feign Client 上传文件支持 -->
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form</artifactId>
    <version>${feign-form.version}</version>
</dependency>
<dependency>
    <groupId>io.github.openfeign.form</groupId>
    <artifactId>feign-form-spring</artifactId>
    <version>${feign-form-spring.version}</version>
</dependency>
```

2. 配置类

```
/**
 * @Author：大漠知秋
 * @Description：Feign Client 进行上传文件配置类
 * @CreateDate：3:48 PM 2018/10/25
 */
@Configuration
public class FeignUploadFileSupportConfiguration {

    @Bean
    @Primary
    @Scope("prototype")
    public Encoder multipartFormEncoder() {
        return new SpringFormEncoder();
    }

}
```

3. 消费者 Controller

```
/**
 * @Author：大漠知秋
 * @Description：Feign 上传文件 Controller
 * @CreateDate：3:27 PM 2018/10/25
 */
@RestController
@RequestMapping("/upload")
public class FeignUploadFileController {

    @Resource
    private FeignUploadFileFeign feignUploadFileFeign;

    @RequestMapping(
            value = "/file",
            method = RequestMethod.POST,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE,
            consumes = MediaType.MULTIPART_FORM_DATA_VALUE
    )
    public ResponseEntity<byte[]> file(@RequestPart("file") MultipartFile file) {
        return feignUploadFileFeign.file(file);
    }

}
```

4. 消费者 Feign

```
/**
 * @Author：大漠知秋
 * @Description：Feign 上传文件
 * @CreateDate：3:27 PM 2018/10/25
 */
@FeignClient(name = "demo-goods", configuration = FeignUploadFileSupportConfiguration.class)
public interface FeignUploadFileFeign {

    @RequestMapping(
            value = "/upload/file",
            method = RequestMethod.POST,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE,
            consumes = MediaType.MULTIPART_FORM_DATA_VALUE
    )
    ResponseEntity<byte[]> file(@RequestPart("file") MultipartFile file);

}
```

5. 提供者 Controller

```
/**
 * @Author：大漠知秋
 * @Description：Feign 上传文件 Controller
 * @CreateDate：3:27 PM 2018/10/25
 */
@RestController
@RequestMapping("/upload")
public class FeignUploadFileController {

    @RequestMapping(
            value = "/file",
            method = RequestMethod.POST,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE,
            consumes = MediaType.MULTIPART_FORM_DATA_VALUE
    )
    public String file(MultipartFile file) {
        System.err.println("原始文件名：" + file.getOriginalFilename());
        System.out.println("文件大小：" + file.getSize());
        return "上传成功";
    }

}
```

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/FeignUploadFile
