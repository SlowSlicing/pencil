# REST API 介绍

　　Eureka 在 GitHub 的 wiki 上专门写了一篇[《 Eureka REST operations》](https://github.com/Netflix/eureka/wiki/Eureka-REST-operations)来介绍 Eureka Server 的 REST API 接口，Spring Cloud Netfix Eureka 跟 Spring Boot 适配之后，提供的 REST API 与原始的 REST API 有一点点不同，其路径中的 `{version}` 值固定为 `eureka`，其他的变化不大，如下图所示：

![Eureka Server REST API](http://img.lynchj.com/0b82c6b1486d47d5a900c3fcc8f7a481.png)

![Eureka Server REST API](http://img.lynchj.com/7623c331fe2846f48a40211ec6a45ad4.png)

# REST API 实战

## 查询所有应用实例

* `http://localhost:8761/eureka/apps`

![查询所有应用实例](http://img.lynchj.com/8aba965611054f8898e3fed2babb688d.png)

## 根据 AppId 查询

* `http://localhost:8761/eureka/apps/demo-order`

![根据 AppId 查询](http://img.lynchj.com/6399fe54d3134b12a94f068dc5f07d65.png)

## 根据 AppId 及 instanceId 查询

* `http://localhost:8761/eureka/apps/demo-order/guoqingsongmbp:demo-order:11100`

![根据 AppId 及 instanceId 查询](http://img.lynchj.com/4fe0aba9f38640b08ff742a41cb52c42.png)

## 根据 instanceId 查询

* `http://localhost:8761/eureka/instances/guoqingsongmbp:demo-order:11100`

![根据 instanceId 查询](http://img.lynchj.com/255455b511ff446c907f0702eef4b5db.png)

## 注册新应用实例

* `http://localhost:8761/eureka/apps/demo-order2`

　　请求体参数（XML 格式）：

```
<instance>
	<instanceId>demo-order2:11101</instanceId>
    <hostName>127.0.0.1</hostName>
	<app>DEMO-ORDER2</app>
    <ipAddr>127.0.0.1</ipAddr>
	<status>UP</status>
	<overriddenstatus>UNKNOWN</overriddenstatus>
	<port enabled="true">11101</port>
	<securePort enabled="false">443</securePort>
	<countryId>1</countryId>
	<dataCenterInfo class="com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo">
	    <name>MyOwn</name>
	</dataCenterInfo>
	<metadata class="java.util.Collections$EmptyMap"/>
	<vipAddress>demo-order2</vipAddress>
	<secureVipAddress>demo-order2</secureVipAddress>
	<isCoordinatingDiscoveryServer>false</isCoordinatingDiscoveryServer>
	<lastUpdatedTimestamp>1540186708769</lastUpdatedTimestamp>
	<lastDirtyTimestamp>1540186708747</lastDirtyTimestamp>
</instance>
```

![注册新应用实例](http://img.lynchj.com/d8a86e8432df4d3599fa77bf5eb171dd.png)

　　请求体参数（JSON 格式）：

```
{
	"instance": {
		"instanceId": "demo-order2:11101",
		"app": "demo-order2",
		"appGroutName": null,
		"ipAddr": "127.0.0.1",
		"sid": "na",
		"homePageUrl": null,
		"statusPageUrl": null,
		"healthCheckUrl": null,
		"secureHealthCheckUrl": null,
		"vipAddress": "demo-order2",
		"secureVipAddress": "demo-order2",
		"countryId": 1,
		"dataCenterInfo": {
			"@class": "com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo",
			"name": "MyOwn"
		},
		"hostName": "127.0.0.1",
		"status": "UP",
		"leaseInfo": null,
		"isCoordinatingDiscoveryServer": false,
		"lastUpdatedTimestamp": 1529391461000,
		"lastDirtyTimestamp": 1529391461000,
		"actionType": null,
		"asgName": null,
		"overridden_status": "UNKNOWN",
		"port": {
			"$": 11102,
			"@enabled": "false"
		},
		"securePort": {
			"$": 7002,
			"@enabled": "false"
		},
		"metadata": {
			"@class": "java.util.Collections$EmptyMap"
		}
	}
}
```

![注册新应用实例](http://img.lynchj.com/e10a10ab18164d0e910bc73b4294d22b.png)

* 查看注册中心结果：

![Eureka Server](http://img.lynchj.com/88d90397b870455eb66c4a879d1c303f.png)

## 注销应用实例

* `http://localhost:8761/eureka/apps/demo-order2/demo-order2:11101`

![注销应用实例](http://img.lynchj.com/10f15069802a4edcbaca6d0fb7ea7f41.png)

## 暂停/下线应用实例

* `http://localhost:8761/eureka/apps/demo-order2/demo-order2:11101/status?value=OUT_OF_SERVICE`

![暂停/下线应用实例](http://img.lynchj.com/0d6671f30ac646bf9f3b67bde4e06988.png)

* 查看注册中心：

![Eureka Server](http://img.lynchj.com/a3c1f11016674e81b024f6d7f9d079dc.png)

## 恢复应用实例

* `http://localhost:8761/eureka/apps/demo-order2/demo-order2:11101/status?value=UP`

![恢复应用实例](http://img.lynchj.com/46ee6d65a2284b2990bae4a18cec18d6.png)

## 应用实例发送心跳

* `http://localhost:8761/eureka/apps/demo-order2/demo-order2:11101`

![应用实例发送心跳](http://img.lynchj.com/17a9158e8794463cb6df450a520ba135.png)

## 修改应用实例元数据

* `http://localhost:8761/eureka/apps/demo-order2/demo-order2:11101/metadata?profile=canary`

![修改应用实例元数据](http://img.lynchj.com/06d0da7a8f7a4feb846832cc9adc41e4.png)

* 原元数据：

![原元数据](http://img.lynchj.com/17a9116ac11d4406b0b293d25044b591.png)

* 修改后的元数据：

![修改后的元数据](http://img.lynchj.com/250635e4306e452582f6b24b5c0e244b.png)
