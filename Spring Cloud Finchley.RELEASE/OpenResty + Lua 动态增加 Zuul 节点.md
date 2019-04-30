Spring Cloud Finchley.RELEASE
Spring Cloud,OpenResty,Lua,动态
[toc]

# 隐忧

&emsp;&emsp;在 Spring Cloud 微服务架构体系中，所有请求的前门的网关 Zuul 承担着请求转发的主要功能，对后端服务起着举足轻重的作用。当业务体量猛增之后得益于 Spring Cloud 的横向扩展能力，往往加节点、加机器就可以使得系统支撑性获得大大提升，但是仅仅加服务而不加网关是会有性能瓶颈的，实践经验得出的结论是单一 Zuul 的处理能力十分有限，因此扩张节点往往是服务连带 Zuul 一起扩张，然后再在请求上层加一层软负载，通常是使用 Nginx（Nginx 均分请求到 Zuul 负载层，“完美”地解决了问题），如下图：

![Nginx 负载](http://img.lynchj.com/09c2d95c82a840bf9a12839c13c9ee7f.png)

&emsp;&emsp;上图的这种方式在实际运行中存在两个缺点：

1. 在后端 Zuul 运行一段时间之后，其中一台 Zuul 挂掉了，前端请求的服务会有一部分挂掉。原因很简单，Nginx 和后端 Zuul 没有关联性，Zuul 宕掉以后，Nginx 还是会把请求分发过来。
2. 新加机器、对 Zuul 做新的横向扩展，就需要去更改 Nginx 的配置，对新扩展的 Zuul 进行配置。

* 解决方案：

&emsp;&emsp;`OpenResty` 整合了 Nginx 与 Lua，实现了可伸缩的 Web 平台，内部集成了大量精良的 Luα 库、第三方模块以及多数的依赖项。能够非常快捷地搭建处理超高并发、扩展性极高的动态 Web 应用、Web 服务和动态网关。我们可以使用 Luα 脚本模块与注册中心构建一个服务动态增减的机制，通过 Lua 获取注册中心状态为 `UP` 的服务，动态地加入到 Nginx 的均衡列表中去。

![OpenResty + Lua](http://img.lynchj.com/42718069fc794a7d82e49936b7f1a6b1.png)

# 实践

&emsp;&emsp;Spring Cloud 中国社区针对上面说的这种场景开源了相关的 Lua 插件源码（[Github 地址](https://github.com/SpringCloud/nginx-zuul-dynamic-lb)）

## OpenResty 安装与配置

```
1、环境
yum -y install readline-devel pcre-devel openssl-devel gcc
2、下载解压OpenResty包
wget https://openresty.org/download/openresty-1.13.6.1.tar.gz
tar -zxvf openresty-1.13.6.1.tar.gz
3、下载ngx_cache_purge模块，该模块用于清理nginx缓存
cd openresty-1.13.6.1/bundle
wget http://labs.frickle.com/files/ngx_cache_purge-2.3.tar.gz
tar -zxvf ngx_cache_purge-2.3.tar.gz
4、下载nginx_upstream_check_module模块，该模块用于upstream健康检查
wget https://github.com/yaoweibin/nginx_upstream_check_module/archive/v0.3.0.tar.gz
tar -zxvf v0.3.0.tar.gz
5、OpenResty配置增加
cd openresty-1.13.6.1
./configure --prefix=/usr/local/openresty --with-http_realip_module  --with-pcre  --with-luajit --add-module=./bundle/ngx_cache_purge-2.3/ --add-module=./bundle/nginx_upstream_check_module-0.3.0/ -j2 
6、编译安装
make
make install

7、OpenResty没有http模块，需要单独安装
cd /usr/local/openresty/lualib/resty
wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http_headers.lua  
wget https://raw.githubusercontent.com/pintsized/lua-resty-http/master/lib/resty/http.lua

8、项目脚本拷贝到这里
copy dynamic_eureka_balancer.lua into this dir
9、Nginx配置文件
vim /usr/local/openresty/nginx/conf/nginx.conf
```

## Nginx 配置如下

```
http {
	#sharing cache area
	lua_shared_dict dynamic_eureka_balancer 128m;

	init_worker_by_lua_block {
		-- 引入 Lua 插件文件
		local file = require "resty.dynamic_eureka_balancer"
		local balancer = file:new({dict_name="dynamic_eureka_balancer"})
		
		-- Eureka server list
		balancer.set_eureka_service_url({"127.0.0.1:8888", "127.0.0.1:9999"})
		
		-- The service name that needs to be monitored
		balancer.watch_service({"zuul", "client"})
	}
	
	upstream springcloud_cn {
		server 127.0.0.1:666; # Required, because empty upstream block is rejected by nginx (nginx+ can use 'zone' instead)
		
		balancer_by_lua_block {    
		
			--The zuul name that needs to be monitored
			local service_name = "zuul"
			
			local file = require "resty.dynamic_eureka_balancer"
			local balancer = file:new({dict_name="dynamic_eureka_balancer"}) 
			
			--balancer.ip_hash(service_name) --IP Hash LB
			balancer.round_robin(service_name) --Round Robin LB
		}
	}

    server {
        listen       80;
        server_name  localhost;
		
		location / {
			proxy_pass  http://springcloud_cn/;
			proxy_set_header Host $http_host;
			proxy_set_header X-Real-IP $remote_addr;
			proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_set_header X-Forwarded-Proto  $scheme;
		}

        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }
	}
}
```

&emsp;&emsp;实现原理是使用 Lua 脚本定时根据配置的服务名与 Eureka 地址，去拉取该服务的信息，在 Eureka 里面提供 `/eureka/apps/{serviceId}` 端点，返回服务的注册信息，所以我们只需要取用状态为 `UP` 的服务，将它的地址加入 Nginx 负载列表即可。此项目使得 Nginx 与 Zuul 之间拥有一个动态感知能力，不用手动配置 Nginx 负载与 Zuul 负载，这样对于应用弹性扩展是极其友好的。


