> 　　`Apache Maven` 是当 Java 技术栈前最流行的项目管理工具，它提供了一系列方便快捷的命令帮助程序员们进行Java工程的开发工作。Maven 服务器位于美国，由于出国带宽和众多因素，在国内直接使用 Maven 的体验并不好。同时，国内还有很多公司的程序员开发机器无法直接连接互联网，鉴于这种情况，在公司区域网架设一部Maven私服能大大提高开发效率。Apache Maven的私服有很多开源提供商，目前用的最多的就是 `Sonatype Nexus Repository`。

## 安装环境

* ContOS 7.4
* Docker 环境
　　可参考：[Docker CE 安装](https://blog.csdn.net/wo18237095579/article/details/80481030)

## 安装

* 创建 Nexus 数据卷目录

```
[root@localhost ~]# mkdir -p /data/docker/volumes/nexus3
[root@localhost ~]# chmod 777 /data/docker/volumes/nexus3
[root@localhost ~]# cd /data/docker/volumes/nexus3
```

* 运行 Docker 镜像

```
[root@localhost /]# docker run -d -p 8081:8081 --name nexus -v /data/docker/volumes/nexus3/data:/nexus-data sonatype/nexus3
```

* 访问 8081 端口

![访问](http://img.lynchj.com/6ab303a9e2fb4c4e9c931294f1a19ade.png)

>　　默认用户名：`admin`，默认密码：`admin123`

## HTTPS 配置

* 新增文件夹待 https 配置使用

```
makir /data/docker/volumes/nexus3/etc
```

* 启动

```
[root@localhost /]# docker run -d -p 8081:8081 -p 4445:8443 --name nexus -v /data/docker/volumes/nexus3/data:/nexus-data -v /data/docker/volumes/nexus3/etc:/opt/sonatype/nexus/etc sonatype/nexus3
```

* 把 自己的 ssl 证书放在下方图中位置：

![证书存放地](http://img.lynchj.com/d31b868afbda4ba69feaa40865b26592.png)

* 修改 jetty-https.xml 配置

![jetty-https.xml 位置所在](http://img.lynchj.com/c26e8cdb0b854f99992a772042968d7b.png)

![修改信息](http://img.lynchj.com/77b72eb2110a40f2bf347591cef63b9b.png)

* 最后配置 Nginx 重定向即可

```
server {
        listen       80;
        server_name  你的域名;
        index index.html index.htm index.php;

        server_name_in_redirect off;

        rewrite ^/(.*)$ https://你的域名/$1 permanent;

        access_log  /usr/local/nginx/access/nexus3.log;
}

### HTTPS server
server {
    listen 443;
    server_name 你的域名;
    ssl on;
    index index.html index.htm;
    ssl_certificate   你的证书所路径 .pem;
    ssl_certificate_key  你的证书路径 .key;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

    location / {

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto https;
        proxy_pass    https://127.0.0.1:4445;

    }

    access_log  /usr/local/nginx/access/nexus3_ssl.log;

}

```

## 配置

* 更改中央远程仓库地址为阿里云的仓库地址

![修改远程中央仓库地址](http://img.lynchj.com/37704785ddbb4e138cb9b1b741253d29.png)

* 添加角色

![添加角色和用户](http://img.lynchj.com/7a614ef9d21148a89d2edd053e0c10f6.png)

* 仓库简单说明

![仓库列表](http://img.lynchj.com/ff6802193a624e58ab15a58d3ec5bb97.png)

　　打开 Repositories 这个页面可能有点懵，怎么这么多？其实常用的只有四个：`maven-central`、`maven-public`、`maven-releases`、`maven-snapshots`。这里说明如下：

　　**maven-central：**主要负责拉取公共仓库镜像，供平常使用，如：Apache Commons 系列、Spring 系列。
　　**maven-public：**我们项目中配置的仓库地址，它其中包含了`maven-central`、`maven-releases`、`maven-snapshots`中的内容（可配置），其实它本身是没有内容的。
　　**maven-releases：**我们使用私服就是因为团队内部有自己的工具包、私密包等之类的原因，而这些包如果来回 Copy 这用太不方便，所以最后 `releases` 版的包部署之后就在此仓库下。
　　**maven-snapshots：**同上，只是版本为 `snapshots` 版

## Maven 项目配置

* setting.xml

```
<servers>
	<server>
	  <id>nexus-xinqiu-public</id>
	  <username>私服用户名</username>
	  <password>私服密码</password>
	</server>
	<server>
	  <id>nexus-xinqiu-releases</id>
	  <username>私服用户名</username>
	  <password>私服密码</password>
	</server>
	<server>
	  <id>nexus-xinqiu-snapshots</id>
	  <username>私服用户名</username>
	  <password>私服密码</password>
	</server>
</servers>
<mirrors>
	<mirror>
		<!--此镜像一般用来作为公司第三方引用基础类库镜像，是所有仓库的镜像地址 -->
		<id>nexus-xinqiu-public</id>
		<!-- 为*表示为所有的仓库做镜像，有了这个配置，所有的构建都会包含public组，如果你想包含public-snapshots组，
		你必须添加public-snapshots这个Profile，通过在命令行使用如下的 -P 标志:$ mvn -P public-snapshots clean install -->
		<mirrorOf>*</mirrorOf>
		<url>http://host:port/repository/maven-public/</url>
	</mirror>
</mirrors>
<!-- 四个环境 -->
<profiles>
	<!-- 本地 -->
    <profile>
      <id>localhost</id>
      <properties>
        <maven.profiles.activation>localhost</maven.profiles.activation>
      </properties>
      <build>
        <finalName>${project.name}-${project.version}</finalName>
        <resources>
          <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
          </resource>
        </resources>
      </build>
      <repositories>
        <repository>
          <id>nexus-xinqiu-public</id>
          <url>http://central</url>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>nexus-xinqiu-public</id>
          <url>http://central</url>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>

    <!-- 开发 -->
    <profile>
      <id>dev</id>
      <properties>
        <maven.profiles.activation>dev</maven.profiles.activation>
      </properties>
      <activation>
        <activeByDefault>true</activeByDefault>
      </activation>
      <build>
        <finalName>${project.name}-${project.version}</finalName>
        <resources>
          <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
          </resource>
        </resources>
      </build>
      <repositories>
        <repository>
          <id>nexus-xinqiu-public</id>
          <url>http://central</url>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>nexus-xinqiu-public</id>
          <url>http://central</url>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>

    <!--测试-->
    <profile>
      <id>test</id>
      <properties>
        <maven.profiles.activation>test</maven.profiles.activation>
      </properties>
      <build>
        <finalName>${project.name}-${project.version}</finalName>
        <resources>
          <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
          </resource>
        </resources>
      </build>
      <repositories>
        <repository>
          <id>nexus-xinqiu-public</id>
          <url>http://central</url>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>nexus-xinqiu-public</id>
          <url>http://central</url>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>

    <!--生产-->
    <profile>
      <id>pro</id>
      <properties>
        <maven.profiles.activation>pro</maven.profiles.activation>
      </properties>
      <build>
        <finalName>${project.name}-${project.version}</finalName>
        <resources>
          <resource>
            <directory>src/main/resources</directory>
            <filtering>true</filtering>
          </resource>
        </resources>
      </build>
      <repositories>
        <repository>
          <id>nexus-xinqiu-public</id>
          <url>http://central</url>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </repository>
      </repositories>
      <pluginRepositories>
        <pluginRepository>
          <id>nexus-xinqiu-public</id>
          <url>http://central</url>
          <releases>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </releases>
          <snapshots>
            <enabled>true</enabled>
            <updatePolicy>always</updatePolicy>
          </snapshots>
        </pluginRepository>
      </pluginRepositories>
    </profile>
</profiles>
```

## 部署自己的项目到仓库

* 在 pom 中配置

```
<build>
	<plugins>
		<plugin>
			<groupId>org.apache.maven.plugins</groupId>
			<artifactId>maven-compiler-plugin</artifactId>
			<version>3.3</version>
			<configuration>
				<source>1.8</source>
				<target>1.8</target>
				<encoding>UTF-8</encoding>
			</configuration>
		</plugin>
		<!-- 要将源码放上去，需要加入这个插件 -->
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-source-plugin</artifactId>
            <version>2.1</version>
            <configuration>
                <attach>true</attach>
            </configuration>
            <executions>
                <execution>
                    <phase>compile</phase>
                    <goals>
                        <goal>jar</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
	</plugins>
</build>

<distributionManagement>
	<repository>
		<!-- 这里的ID要和setting的id一致 -->
		<id>nexus-xinqiu-releases</id>
		<url>http://host:port/repository/maven-releases/</url>
	</repository>
	<!--这是打成快照版本的配置，如果不用这个snapshotRepository标签，打包失败，会报权限问题 -->
	<snapshotRepository>
		<id>nexus-xinqiu-snapshots</id>
		<url>http://host:port/repository/maven-snapshots/</url>
	</snapshotRepository>
</distributionManagement>
```

* 部署

```
clean deploy
```
