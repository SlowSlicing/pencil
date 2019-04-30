> 　　官方说：For servlet stack applications, the spring-boot-starter-web includes Tomcat by including spring-boot-starter-tomcat, but you can use spring-boot-starter-jetty or spring-boot-starter-undertow instead
> 　　When switching to a different HTTP server, you need to exclude the default dependencies in addition to including the one you need. Spring Boot provides separate starters for HTTP servers to help make this process as easy as possible.
>> 　　对于 servlet 栈应用程序，`spring-boot-starter-web` 包括 Tomcat `spring-boot-starter-tomcat`，但您可以使用 `spring-boot-starter-jetty` 或 `spring-boot-starter-undertow` 替代
>> 　　切换到其他 HTTP 服务器时，除了包含所需的依赖项外，还需要排除默认依赖项。 Spring Boot 为 HTTP 服务器提供单独的启动程序，以帮助使此过程尽可能简单。

# Undertow

　　`Undertow` 是什么? Undertow 翻译为`暗流`，即平静的湖面下暗藏着波涛汹涌，所以 JBoss 公司取其意，为它的轻量级高性能容器命名。 Undertow 提供阻塞或基于 XNIO 的非阻塞机制，它的包大小不足 `1MB`，内嵌模式运行时的堆内存占用只有 `4MB` 左右。要使用 Undertow 只需要在配置文件中移除 Tomcat，添加 Undertow 的依赖即可。

```
<!-- Srping Boot Web 支持 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
    <exclusions>
        <!-- 排除掉 内置 Tomcat -->
        <exclusion>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-tomcat</artifactId>
        </exclusion>
    </exclusions>
</dependency>

<!-- 内置服务器使用 Undertow -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-undertow</artifactId>
</dependency>
```

# Undertow 参数

| 配置项 | 默认值 | 说明 |
| --- | --- | --- |
| server.undertow.io-threads | Math.max(Runtime.getRuntime().availableProcessors(), 2) | 设置 IO 线程数，它主要执行非阻塞的任务，它们会负责多个连接，默认设置每个 CPU 核心有一个线程。不要设置过大，如果过大，启动项日会报错：打开文件数过多 |
| server.undertow.worker-threads | ioThreads * 8 | 阳塞任务线程数，当执行类似 Servlet 请求阻塞 IO 操作，Undertow 会从这个线程池中取得线程。它的值设置取决于系统线程执行任务的阻塞系数，默认值：IO 线程数 * 8 |
| server.undertow.direct-buffers | 取决于JVM 最大可用内存大小：（long maxMemory = Runtime.getRuntime().maxMemory();），小于 64MB 默认为 false，其余为 true | 是否分配直接内存（NIO 直接分配的是堆外内存） |
| server.undertow.buffer-size | 最大可用内存 < 64MB：512 字节。6MB <= 最大可用内存 <128MB：1024 字节。128MB < 最大可用内存：1024 * 16 - 20 字节 | 每块 buffer 的空间大小，空间越小利用越充分，不要设置太大，以免影响其他应用，合适即可 |
