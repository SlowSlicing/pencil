Spring Cloud Finchley.RELEASE
Spring Cloud,Eureka,CAP,Peer To Peer,SELF PRESERVATION
@[toc]

# 概述

> &emsp;&emsp;作为一个服务注册以及发现中心，需要解决很多问题。

## 服务实例如何注册到服务中心

&emsp;&emsp;本质上就是在服务启动的时候，需要调用 Eureka Server 的 REST API 的 register 方法，去注册该应用实例的信息。对于使用 Java 的应用服务，可以使用 Netflix 的 Eureka Client 封装的 API 去调用；对于 Spring Cloud 的应用，可以使用 `spring-cloud-starter-netfix-eureka-client`，基于 Spring Boot 的自动配置，自动帮你实现服务信息的注册。

## 服务实例如何从服务中心剔除

&emsp;&emsp;正常情况下服务实例在关闭应用的时候，应该通过钩子方法或其他生命周期回调方法去调用 Eureka Server 的 REST API 的 de-register 方法，来删除自身服务实例的信息。另外为了解决服务实例挂掉或其他异常情况没有及时删除自身信息的问题，Eureka Server 要求 Client 端定时进行续约，也就是`发送心跳`，来证明该服务实例还是存活的，是健康的，是可以调用的。**如果租约超过一定时间没有进行续约操作，Eureka Server 端会主动剔除**。这一点 Eureka Server 采用的就是分布式应里头经典的心跳模式。

## 服务实例信息的一致性问题

&emsp;&emsp;由于服务注册及发现中心不可能是单点的，其自身势必有个集群，那么服务实例注册信息如何在这个集群里保持一致呢？这跟 Eureka Server 的架构有关，理解其设计理念有助于使用的实战及调优，下面主要分 `AP 优于 CP`、 `Peer to Peer 架构`、`SELF PRESERVATION` 设计四个方面来说明。

### AP 优于 CP

&emsp;&emsp;分布式系统领域有个重要的 `CAP理论`，该理论由加州大学伯克利分校的 Eric Brewer 教授提出，由麻省理工学院的 Seth Gilbert 和 Nancy Lynch 进行理论证明。该理论提到了分布式系统的 CAP 三个特性：

* **Consistency**：数据一致性，即数据在存在多副本的情况下，可能由于网络、机器故障、软件系统等问题导致数据写入部分副本成功，部分副本失败，进而造成副本之间数据不一致，存在冲突。满足一致性则要求对数据的更新操作成功之后。多副本的数据保持一致。
* **Availability**：在任何时候客户端对集群进行读写操作时，请求能够正常响应，即在一定的延时内完成。
* **Partition Tolerance**：分区容忍性，即发生通信故障的时候，整个集群被分割为多个无法相互通信的分区时，集群仍然可用。

&emsp;&emsp;对于分布式系统来说，一般网络条件相对不可控，出现网络分区是不可避免的，因此系统必须具备分区容忍性。在这个前提下分布式系统的设计则在 AP 及 CP 之间进行选择。不过不能理解为 CAP 三者之间必须三选二，它们三者之间不是对等和可以相互替换的。在分布式系统领域，P是一个客观存在的事实，不可绕过，所以 P 与 AC 之间不是对等关系。

&emsp;&emsp;对于 ZooKeeper，它是 "C"P 的，之所以 C 加引号是因为 ZooKeeper 默认并不是严格的强一致，比如客户端 A 提交一个写操作，ZooKeeper 在过半数节点操作成功之后就返回，此时假设客户端 B 的读操作请求到的是 A 写操作尚未同步到的节点，那么读取到的就不是客户端 A 写操作成功之后的数据。如果在使用的时候需要强一致则需要在读取数据的时候先执行一下 `sync` 操作，即与 leader 节点先同步下数据，这样才能保证强一致。在极端的情况下发生网络分区的时候，如果 leader 节点不在 non-quorum 分区，那么对这个分区上节点的读写请求将会报错，无法满足 Availability 特性。

&emsp;&emsp;Eureka 是在部署在 AWS 的背景下设计的，其设计者认为，在云端，特别是在大规模部署的情况下，失败是不可避免的，可能因为 Eureka 自身部署失败，注册的服务不可用，或者由于网络分区导致服务不可用，因此不能回避这个问题。要拥抱这个问题，就需要 Eureka 在网络分区的时候，还能够正常提供服务注册及发现功能，因此 Eureka 选择满足 Availability 这个特性。 在实际生产实践中，服务注册及发现中心保留可用及过期的数据总比丢失掉可用的数据好。这样的话，应用实例的注册信息在集群的所有节点间并不是强一致的，这就需要客户端能够支持负载均
衡及失败重试。在 Netflix 的生态中，由 Ribbon 提供这个功能。

### Peer to Peer 架构

> &emsp;&emsp;般而言，分布式系统的数据在多个副本之间的复制方式，可分为主从复制和对等复制。

#### 主从复制

&emsp;&emsp;主从复制也就是广为人知的 Master-Slave 模式，即有一个主副本，其他副本为从副本。所有对数据的写操作都提交到主副本，最后再由主副本更新到其他从副本具体更新的方式，还可以细分为同步更新、异步更新、同步及异步混合。

&emsp;&emsp;对于主从复制模式来讲，写操作的压力都在主副本上，它是整个系统的瓶颈，但是从副本可以帮主副本分担读请求。

#### 对等复制

&emsp;&emsp;即 Peer to Peer 的模式，副本之间不分主从，任何副本都可以接收写操作，然后每个副本之间相互进行数据更新

&emsp;&emsp;对于对等复制模式来讲，由于任何副本都可以接收写操作请求，不存在写操作压力瓶颈。但是由于每个副本都可以进行写操作处理，各个副本之间的数据同步及冲突处理是一个比较棘手的问题。

&emsp;&emsp;Eureka Server 采用的就是 Peer to Peer 的复制模式。这里来分为客户端及服务端两个角度来阐述。

##### 客户端

&emsp;&emsp;Client 端一般通过如下配置 Eureka Server 的 Peer 节点：

```
eureka
	client: 
		serviceUrl: 
			defaultzone: http://127.0.0.1:8761/eureka/,http://127.0.0.1:8762/eureka/
```

&emsp;&emsp;实际代码里支持 `preferSameZoneEureka`，即有多个分区的话，优先选择与应用实例所在分区一样的其他服务的实例，如果没找到则默认使用 `defaultZone`。客户端使用 `quarantineSet` 维护了一个不可用的 Eureka Server 列表，进行请求的时候，优先从可用的列表中进行选择，如果请求失败则切换到下一个 Eureka Server进行重试，重试次数默认为3。

&emsp;&emsp;另外为了防止每个 Client 端都按配置文件指定的顺序进行请求造成 Eureka Server 节点请求分布不均衡的情况，Client 端有个定时任务（默认5分钟执行一次）来刷新并随机化 Eureka Server 的列表。

##### 服务端

&emsp;&emsp;Eureka Server 本身依赖了 Eureka Client，也就是每个 Eureka Server 是作为其他 Eureka Server 的 Client。在单个 Eureka Server 启动的时候，会有一个 syncUp 的操作，通过 Eureka Client 请求其他 Eureka Server 节点中的一个节点获取注册的应用实例信息，然后复制到其他 peer 节点 Eureka Server 在执行复制操作的时候，使用 HEADER_REPLICATION 的 http header 来将这个请求操作与普通应用实例的正常请求操作区分开来。通过 HEADER_ REPLICATION 来标识是复制请求，这样其他 peer 节点接收到请求的时候，就不会再对它的 peer 节点进行复制操作，从而避免死循环。

&emsp;&emsp;Eureka Server 由于采用了 Peer to Peer 的复制模式，其重点要解决的另外一个问题就是数据复制的冲突问题。针对这个问题,，Eureka采用如下两个方式来解决：

* lastDirtyTimestamp 标识
* heartbeat

&emsp;&emsp;针对数据的不一致，一般是通过版本号机制来解决，最后在不同副本之间只需要判断请求复制数据的版本号与本地数据的版本号高低就可以了。 Eureka 没有直接使用版本号的属性，而是采用一个叫作 `lastDirtyTimestamp` 的字段来对比。

&emsp;&emsp;如果开启 SyncWhenTimestampDiffers 配置（默认开启），当 lastDirtyTimestamp 不为空的时候，就会进行相应的处理：

* 如果请求参数的 lastDirtyTimestamp 值大于 Server 本地该实例的 lastDirtyTimestamp 值，则表示 Eureka Server 之间的数据出现冲突，这个时候就返回404，要求应用实例重新进行 register 操作。
* 如果请求参数的 lastDirtyTimestamp 值小于 Server 本地该实例的 lastDirtyTimestamp 值，如果是 peer 节点的复制请求，则表示数据出现冲突，返回409给 peer 节点，要求其同步自己最新的数据信息。

&emsp;&emsp;peer 节点之间的相互复制并不能保证所有操作都能够成功，因此 Eureka 还通过应用实例与 Server 之间的 heartbeat 也就是 renewLease 操作来进行数据的最终修复，即如果发现应用实例数据与某个 Server 的数据出现不一致，则 Server返回404，应用实例重新进行 register 操作。

### SELF PRESERVATION 设计

&emsp;&emsp;在分布式系统设计里头，通常需要对应用实例的存活进行健康检查，这里比较关键的问题就是要处理好网络偶尔抖动或短暂不可用时造成的误判。另外 Eureke Server 端与 Client 端之间如果出现网络分区问题，在极端情况下可能会使得 Eureka Server 清空部分服务的实例列表，这个将严重影响到 Eureka Aerver 的 Availability 属性。因此 Eureka Server 引入了 SELF RESERVATION 机制。

&emsp;&emsp;Eureka Client 端与 Server 端之间有个租约，Client 要定时发送心跳来维持这个租约，表示自己还存活着。 Eureka 通过当前注册的实例数，去计算每分钟应该从应用实例接收到的心跳数，如果最近一分钟接收到的续约的次数小于等于指定阈值的话，则关闭租约失效剔除，禁止定时任务剔除失效的实例，从而保护注册信息。
