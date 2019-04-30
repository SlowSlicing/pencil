Spring Boot
Starter,Spring Boot
[TOC]

> &emsp;&emsp;这几天在写一个团队使用的工具，有一些东西呢，需要从 Spring Cloud 配置中心中读取配置信息进行加载初始化，所以做了一个 Starter 。

### pom 依赖

```
<properties>
    <!-- Spring Boot -->
    <spring.boot>1.5.9.RELEASE</spring.boot>
</properties>

<dependencyManagement>
    <dependencies>
        <dependency>
            <!-- Import dependency management from Spring Boot -->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring.boot}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencys>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-autoconfigure</artifactId>
    </dependency>
</dependencys>
```

### 七牛云服务

&emsp;&emsp;这里提供了一个七牛云客户端初始化的服务，伪代码如下：

```
@Slf4j
public class QiniuService {

    private static String AK;
    private static String SK;
    private static Auth AUTH;
    private static Configuration CFG;
    private static BucketManager BUCKET_MANAGER;
    private static String BUCKET_NAME;

    public QiniuService(String accesskey, String secretkey, String bucketName) {
        init(accesskey, secretkey, bucketName);
    }

    /**
     * 初始化七牛云配置
     */
    public static void init(String accesskey, String secretkey, String bucketName) {

        if (StringUtils.isBlank(accesskey)
                || StringUtils.isBlank(secretkey)
                || StringUtils.isBlank(bucketName)) {
            log.warn("【七牛云服务】-初始化参数不足，不予初始化");
            return;
        }

        try {
            AK = accesskey;
            SK = secretkey;
            AUTH = Auth.create(AK, SK);
            CFG = new Configuration(Zone.zone0());
            BUCKET_MANAGER = new BucketManager(AUTH, CFG);
            BUCKET_NAME = bucketName;
        } catch (Exception e) {
            log.error("【七牛云服务】-初始化出错：{}", ExceptionUtils.getStackTrace(e));
        }

    }

	······
}
```

&emsp;&emsp;这里啰嗦一下，官方对 Starter 包定义的 `artifactId` 是有要求的，当然也可以不遵守（手动滑稽）。Spring 官方 Starter 通常命名为 `spring-boot-starter-{name}`如：`spring-boot-starter-web`，Spring 官方建议非官方的 Starter 命名应遵守 `{name}-spring-boot-starter` 的格式。

### 编写 properties 类

```
/**
 * @Author：大漠知秋
 * @Description：配置中心配置信息导入到此类
 * @CreateDate：9:21 2018/3/1
 */
@ConfigurationProperties()
public class QiNiuProperties {

    /** 七牛云相关 */
    private String qiniuAccesskey;
    private String qiniuSecretkey;
    private String qiniuBucketName;
    private String qiniuUrl;

    public String getQiniuAccesskey() {
        return qiniuAccesskey;
    }

    public void setQiniuAccesskey(String qiniuAccesskey) {
        this.qiniuAccesskey = qiniuAccesskey;
        QiniuService.init(qiniuAccesskey, qiniuSecretkey, qiniuBucketName);
    }

    public String getQiniuSecretkey() {
        return qiniuSecretkey;
    }

    public void setQiniuSecretkey(String qiniuSecretkey) {
        this.qiniuSecretkey = qiniuSecretkey;
        QiniuService.init(qiniuAccesskey, qiniuSecretkey, qiniuBucketName);
    }

    public String getQiniuBucketName() {
        return qiniuBucketName;
    }

    public void setQiniuBucketName(String qiniuBucketName) {
        this.qiniuBucketName = qiniuBucketName;
        QiniuService.init(qiniuAccesskey, qiniuSecretkey, qiniuBucketName);
    }

    public String getQiniuUrl() {
        return qiniuUrl;
    }

    public void setQiniuUrl(String qiniuUrl) {
        this.qiniuUrl = qiniuUrl;
    }
}
```

### 重点，编写 AutoConfigure 类

```
/**
 * @Author：大漠知秋
 * @Description：七牛云服务自动配置类
 * @CreateDate：10:42 2018/7/25
 */
@Configuration
@ConditionalOnClass(QiniuService.class)
@EnableConfigurationProperties(QiNiuProperties.class)
public class QiNiuAutoConfig {

    @Resource
    private QiNiuProperties qiNiuProperties;

    @Bean
    @ConditionalOnMissingBean
    @ConditionalOnProperty(prefix = "qiniu", value = "enabled", havingValue = "true")
    QiniuService qiniuService (){
        return new QiniuService(qiNiuProperties.getQiniuAccesskey(), qiNiuProperties.getQiniuSecretkey(), qiNiuProperties.getQiniuBucketName());
    }

}
```

&emsp;&emsp;**有几个注解这里记录一下：**

* `@ConditionalOnClass`：某个 Class 位于类路径上，才会实例化一个Bean。也就是说，当 classpath 下发现该类的情况下进行实例化。
* `@EnableConfigurationProperties`：为带有 `@ConfigurationProperties` 注解的 Bean 提供有效的支持。这个注解可以提供一种方便的方式来将带有 `@ConfigurationProperties` 注解的类注入为 Spring 容器的 Bean。
* `@ConditionalOnMissingBean`：当 Spring Context中不存在该Bean时。
* `@ConditionalOnProperty`：我此处的意思为：当配置文件中 `qiniu.enable` 的值为 `true` 时，实例化此类。

### 建立 spring.factories 文件

&emsp;&emsp;在 `resources/META-INF/` 下创建 `spring.factories` 文件

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.hoppingcity.config.qiniu.QiNiuAutoConfiguration, \
......
```

> &emsp;&emsp;可以配置多个自动配置类，以`,`分割。

### 打包

```
mvn clean install
```

### 引用

&emsp;&emsp;只需把自己的 Starter 引入之后，在配置文件中配置加入七牛云配置即可。

```
### 七牛云相关
## 基础配置
# AK
qiniu-accesskey: 你的 accesskey
# SK
qiniu-secretkey: 你的 secretkey
# 芯球泊车app使用
qiniu-bucket-name: 你的 backet name

```
