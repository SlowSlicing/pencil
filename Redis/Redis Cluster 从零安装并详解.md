# Redis 基础安装

### 基础环境

#### 三台机器

| OS | IP |
| --- | --- |
| CentOS 7.4 | 192.168.117.135 |
| CentOS 7.4 | 192.168.117.136 |
| CentOS 7.4 | 192.168.117.137 |

### 下载 Redis 源码包

　　首先在一台机器上部署两个不同端口的 Redis：`192.168.117.135`

```
[root@localhost tmp]# wget http://download.redis.io/releases/redis-4.0.10.tar.gz
--2018-07-02 22:50:18--  http://download.redis.io/releases/redis-4.0.10.tar.gz
Resolving download.redis.io (download.redis.io)... 109.74.203.151
Connecting to download.redis.io (download.redis.io)|109.74.203.151|:80... connected.
HTTP request sent, awaiting response... 200 OK
Length: 1738465 (1.7M) [application/x-gzip]
Saving to: ‘redis-4.0.10.tar.gz’

100%[===========================================================================================================================================================>] 1,738,465    558KB/s   in 3.0s

2018-07-02 22:50:21 (558 KB/s) - ‘redis-4.0.10.tar.gz’ saved [1738465/1738465]

[root@localhost tmp]# tar -xzvf redis-4.0.10.tar.gz
```

### 安装Redis

```
[root@localhost redis-4.0.10]# make PREFIX=/usr/local/redis1 install
```

　　**出错了：**

```
...
MAKE hiredis
cd hiredis && make static
make[3]: Entering directory `/usr/local/tmp/redis-4.0.10/deps/hiredis'
gcc -std=c99 -pedantic -c -O3 -fPIC  -Wall -W -Wstrict-prototypes -Wwrite-strings -g -ggdb  net.c
make[3]: gcc: Command not found
make[3]: *** [net.o] Error 127
make[3]: Leaving directory `/usr/local/tmp/redis-4.0.10/deps/hiredis'
make[2]: *** [hiredis] Error 2
make[2]: Leaving directory `/usr/local/tmp/redis-4.0.10/deps'
make[1]: [persist-settings] Error 2 (ignored)
    CC adlist.o
/bin/sh: cc: command not found
make[1]: *** [adlist.o] Error 127
make[1]: Leaving directory `/usr/local/tmp/redis-4.0.10/src'
make: *** [install] Error 2
```

> 　　make 编译 Redis 源码包时，需要用到 gcc 插件，我们安装上 gcc 插件即可

```
[root@localhost redis-4.0.10]# yum -y install gcc gcc-c++
```

　　**再次安装，还是报错：**

```
[root@localhost redis-4.0.10]# make PREFIX=/usr/local/redis1 install
cd src && make install
make[1]: Entering directory `/usr/local/tmp/redis-4.0.10/src'
    CC Makefile.dep
make[1]: Leaving directory `/usr/local/tmp/redis-4.0.10/src'
make[1]: Entering directory `/usr/local/tmp/redis-4.0.10/src'
    CC adlist.o
In file included from adlist.c:34:0:
zmalloc.h:50:31: fatal error: jemalloc/jemalloc.h: No such file or directory
 #include <jemalloc/jemalloc.h>
                               ^
compilation terminated.
make[1]: *** [adlist.o] Error 1
make[1]: Leaving directory `/usr/local/tmp/redis-4.0.10/src'
make: *** [install] Error 2
```

> 　　原因是 `jemalloc` 重载了Linux下的 `ANSIC` 的 `malloc` 和 `free` 函数。解决办法：**make 时添加参数。**

```
[root@localhost redis-4.0.10]# make MALLOC=libc PREFIX=/usr/local/redis1 install
cd src && make install
make[1]: Entering directory `/usr/local/tmp/redis-4.0.10/src'
rm -rf redis-server redis-sentinel redis-cli redis-benchmark redis-check-rdb redis-check-aof *.o *.gcda *.gcno *.gcov redis.info lcov-html Makefile.dep dict-benchmark
(cd ../deps && make distclean)
make[2]: Entering directory `/usr/local/tmp/redis-4.0.10/deps'
(cd hiredis && make clean) > /dev/null || true
(cd linenoise && make clean) > /dev/null || true
(cd lua && make clean) > /dev/null || true
(cd jemalloc && [ -f Makefile ] && make distclean) > /dev/null || true
(rm -f .make-*)
make[2]: Leaving directory `/usr/local/tmp/redis-4.0.10/deps'
(rm -f .make-*)
echo STD=-std=c99 -pedantic -DREDIS_STATIC= >> .make-settings
echo WARN=-Wall -W -Wno-missing-field-initializers >> .make-settings
***********************此处省略***********************
    CC geohash.o
    CC geohash_helper.o
    CC childinfo.o
    CC defrag.o
    CC siphash.o
    CC rax.o
    LINK redis-server
    INSTALL redis-sentinel
    CC redis-cli.o
    LINK redis-cli
    CC redis-benchmark.o
    LINK redis-benchmark
    INSTALL redis-check-rdb
    INSTALL redis-check-aof

Hint: It's a good idea to run 'make test' ;)

    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
    INSTALL install
make[1]: Leaving directory `/usr/local/tmp/redis-4.0.10/src'
```

　　安装完成，我们再在此台机器上部署第二个 Redis，直接安装即可

```
[root@localhost redis-4.0.10]# make MALLOC=libc PREFIX=/usr/local/redis2 install
```

　　**至此，一台机器（192.168.117.135）上的两个 Redis 安装完成，另外两台机器相同操作即可**

# Redis Cluster 概念

> 这节较为无聊，想快速使用的，可以直接看下一节：[Redis Cluster 部署](https://blog.csdn.net/wo18237095579/article/details/80895413#redis-cluster-%E9%83%A8%E7%BD%B2)

### 结构设计

　　Redis集群搭建的方式有多种，例如使用 `Zookeeper`、`Proxy` 等，但从`Redis 3.0` 之后版本支持 `Redis Cluster` 集群，Redis Cluster采用无中心结构，每个节点保存数据和整个集群状态，每个节点都和其他所有节点连接。其 Redis Cluster 架构图如下：

![Redis Cluster架构图](http://img.lynchj.com/2bd98ffb1a644583b174f24523569ed7.png)

#### 高性能

* 采用了异步复制机制，向某个节点写入数据时，无需等待其它节点的写数据响应。
* 无中心代理节点，而是将客户端直接重定向到拥有数据的节点。
* 对于N个 Master 节点的 Cluster ，整体性能理论上相当于单个 Redis 的性能的N倍。

#### 高可用

* 采用了主从复制的机制，Master 节点失效时 Slave 节点自动提升为 Master 节点。如果 Cluster 中有N个 Master 节点，每个 Master 拥有1个 Slave 节点，那么这个 Cluster 的失效概率为 `1/(2*N-1)`，可用概率为 `1-1/(2*N-1)`。

#### 高可扩展

* 可支持多达`1000`个服务节点。随时可以向 Cluster 中添加新节点，或者删除现有节点。Cluster 中每个节点都与其它节点建立了相互连接。

### 结构特点

* 所有的 `Redis 节点`彼此互联(PING-PONG机制)，内部使用二进制协议优化传输速度和带宽。
* 节点的`fail`是通过集群中`超过半数`的节点检测失效时才生效。
* 客户端与 Redis 节点直连，不需要中间Proxy层，客户端不需要连接集群所有节点，连接集群中任何一个可用节点即可。
* Redis Cluster 把所有的物理节点映射到`[0-16383] slot`（哈希槽） 上（不一定是平均分配），Cluster 负责维护`node <-> slot <-> value`。
* Redis 集群预分好 `16384` 个哈希槽，**当需要在 Redis 集群中放置一个 `key-value` 时，根据 `CRC16`(key) mod 16384 的值，决定将一个 key 放到哪个桶中。**

### 主要组件

#### 键分布模型

　　键(keys)空间有 `16384` 个 `slots`。理论上 Redis Cluster 可以支持 16384 个 Master 节点，但是推荐的节点数量最多为 1000 个 Master 。每个 Master 节点处理这些 slots 中的一部分。如果没有 Cluster 的重新配置，那么 Cluster 中的 slots 的分配是稳定的。在稳定状态下，每个 slot 被分配到唯一一个 Master 节点上，这个 Master 节点可以有一个或多个 Slave 节点，从而同样保证了高可用。

　　键按照下面的公式分配到某个slot中：

> HASH_SLOT = CRC16(key) mod 16384

　　据此，对于任意一个指定的键，可以知道这个键存储在哪个 slot 中，从而进一步可以知道在哪个 Master 中。

#### 键哈希标记(key hash tags)

　　对于形如 `{XXXXXX}YYYYY` 这样的键，Redis 只对花括号中的部分`XXXXXX`进行哈希运算。

　　`{redis.coe2coe.me}red`，`{redis.coe2coe.me}green`，这两个键将分配到同一个 slot 中。

#### Cluster 结点属性

　　结点ID：Cluster 中每个节点都有一个唯一的名称。这个名称由一个 `160bits` 的随机数的十六进制形式来表示，这个随机数在结点首次启动时产生。修改结点的IP地址无需修改结点的ID，Redis Cluster 使用 `gossip` 协议自动检测结点的IP和端口的变化。

#### Cluster 总线(Bus)

　　Redis Cluster 的节点之间通过 Cluster 总线进行相互通信。如果节点通过端口 6379 与客户端通信，那么节点同时还需要通过 `16379（10000+6379）`与其它节点通信。节点之间的通信使用 `Cluster总线协议(gossip)` 进行通信，这个协议是一种二进制通信协议。

#### Cluster拓扑

　　N个节点的 Redis Cluster 中，每个节点都与 Cluster 中的每个其它节点之间建立了一个连接。N个节点的Cluster中，每个节点有 `N-1`个连接。整个 Cluster 为 `PING/PONG` 机制建立了 `N*(N-1)/2` 个连接。

#### 节点握手

　　Redis Cluster 中的节点A发送 `MEET` 消息通知另外一个节点B，让节点B把节点A当做 Cluster 中的一员。如果节点A认识节点B，节点B认识节点C，那么节点B可以发送包含节点C信息的 `MEET` 消息给节点A，从而节点A也认识节点C了，即节点A将与节点C建立连接。

　　这个机制确保了以下两个结论的成立：

* Redis Cluster 能够在人工组建了一个初始网络后，最终能够自动完成全网连接拓扑的建立。
* Redis Cluster 能够阻止一个 Cluster 的节点在改变IP和端口后错误的混入其它 Cluster。

### 重定向

#### MOVED重定向

　　Redis 客户端可以向 Cluster 中的任何一个节点，包括任何 Master 和任何 Slave 节点发起查询。如果这个查询只包括一个键，或者所有的键都在同一个 slot 中，那么这个节点将继续判断这个 slot 是否归自己负责。如果是归自己负责，就直接向客户端返回查询结果；如果不归自己负责，就根据本节点保存的 slot 到节点的映射关系，找到负责这个 slot 的节点，取得其IP和端口，然后向客户端发送一个重定向消息`MOVED`，这个消息包含了负责这个 slot 的节点的 `IP` 和 `端口` 以及这个键的 slot，此时客户端需要重新向负责该 slot，即负责该键的新节点发起查询。

　　基于 Redis Cluster 的重定向机制，客户端存在以下的优化途径来降低重定向的成本，主要思路是客户端尽可能提高节点的命中率，即尽可能确保发起查询的键所对应的 slot 在查询的节点中。

　　客户端可以自行维护一个slot到节点(IP+端口)的映射。每当收到重定向消息时，将消息中包含的 slot 和 IP 和端口加入到这个映射中。

　　另外一种方法是在客户端刚启动时，以及在收到重定向消息时，发出命令 `cluster nodes` 或者 `cluster slots` 命令，查询 Cluster 中的 slot 和 节点 的映射关系，并保存到客户端本地。

#### Cluster 在线重新配置(live reconfiguration)

　　Redis Cluster 支持在 Cluster 运行过程中动态的增加或删除节点。增加或删除节点的本质是 slot 在节点之间的重新分配，即将 slot 从由一个节点负责，修改为由另外一个节点负责。

　　新增节点：**将现有节点负责的一部分 slots 分配给新增的节点，现有节点不再负责这部分 slots。**

　　删除节点：**将待删除节点负责的全部 slots 分配到其它存活的节点，待删除节点不再负责这部分 slots。**

#### ASK 重定向

　　假设的场景：`slot8` 当前由节点A负责，但是正在迁移到节点B，即由节点B负责。

　　客户端连接到节点A。客户端查询某个位于 slot8 中的键，该键已经迁移节点B。此时节点A可以向客户端发送 `ASKING` 重定向消息，表示客户端本次需要重定向到节点B去查询，但是所有后续查询还是应该查询节点A，因为 `slot8` 中的其它键还在节点A中，所以此时不能直接发送 `MOVED` 重定向消息。客户端收到 `ASKING` 重定向消息时显然不应该立即据此更新其 slot 到节点的映射关系。

　　当迁移完成后，节点A应该向客户端发送 `MOVED` 重定向消息，表示 slot8 已经永久重定向到了节点B。

#### 应用程序的 mset/mget 优化

　　由于 `mset/mget` 命令中含有多个不同的键，如果这些键分不到很多不同的 slots 中，就有可能会分布到不同的节点中，这会带来很大的问题。此时应该使用`键哈希标记`(key hash tags)方法来解决这个问题，即通过让这写键有相同的标记的方式使得这写键全部位于相同的 slots 中，从而位于相同的节点中。

### 故障容忍度

#### 心跳和 gossip 消息

　　Redis Cluster 持续的交换 `PING` 和 `PONG` 数据包。这两种数据包的数据结构相同，都包含重要的配置信息，唯一的不同是消息类型字段。PING 和 PONG 数据包统称为心跳数据包。

　　每个节点在每一秒钟都向一定数量的其它节点发送 PING 消息，这些节点应该向发送 PING 的节点回复一个 PONG 消息。节点会尽可能确保拥有每个其它节点在 `NOTE_TIMEOUT / 2` 秒时间内的最新信息，否则会发送一个 PING 消息，以确定与该节点的连接是否正常。

　　假定一个 Cluster 有 301 个节点，NOTE_TIMEOUT 为60秒，那么每30秒每个节点至少发送300个PING，即每秒10个 PING， 整个 Cluster 每秒发送 `10 x 301 = 3010` 个 PING。这个数量级的流量不应该会造成网络负担。

#### 故障检测

　　Redis Cluster 的故障检测用于检测一个 Master 节点何时变得不再有效，即不能提供服务，从而应该让 Slave 节点提升为 Master 节点。如果提升失败，则整个 Cluster 失效，不再接受客户端的服务请求。

　　当一个节点A向另外一个节点B发送了 PING 消息之后，经过 NODE_TIMEOUT 秒时间之后仍然没有收到 PONG 应答，则节点A认为节点B失效，节点A将为该节点B设置 `PFAIL` 标志。

　　在 `NODE_TIMEOUT * FAIL_REPORT_VALIDITY_MULT` 时间内，当 Cluster 中大多数节点认为节点B失效，即设置 `PFAIL` 标志时，这个 Cluster 认为节点B真的失效了，此时节点A将为节点B设置 `FAIL`标志，并向所有节点发送 `FAIL` 消息。

　　在一些特定情况下，拥有 `FAIL` 标志的节点，也可以清除掉 `FAIL` 标志。

　　Redis Cluster 故障检测机制最终应该让所有节点都一致同意某个节点处于某个确定的状态。如果发生这样的情况少数节点确信某个节点为 `FAIL`，同时有少数节点确认某个节点为非 `FAIL`，则 Redis Cluster 最终会处于一个确定的状态：

* 情况1：**最终大多数节点认为该节点FAIL，该节点最终实际为FAIL。**
* 情况2：**最终在 `N x NODE_TIMEOUT` 时间内，仍然只有少数节点将给节点标记为FAIL，此时最终会清除这个节点的FAIL标志。**

### 故障切换

#### 当前世代(current epoch)

　　当多个节点提供了有矛盾的信息时，需要一种机制来决定哪个信息是最新的信息。在 Redis Cluster 中使用`世代`(epoch)这个概念来解决这个问题，它是一个64位无符号整数，相当于 Redis Cluster 中的逻辑时钟。

　　每个节点在创建时的世代为 `0`。当一个节点收到一条消息时，会比较发送者的世代与自己的世代。如果发送者的世代较大，则使用发送者的世代作为自己的世代。最终，Cluster 中所有节点的世代都是相同的，即那个最大的世代。

　　这个世代就是当前世代。当前世代如何增加的问题，在后面会介绍。

#### 配置世代(config epoch)

　　在 Slave 节点提升为 Master 节点的过程中，Slave 节点会创建一个配置世代，等于 Master 节点的当前世代，同时还要增大这个值。当所有节点同意了这个提升，即赢得 Slave 选举时，Slave 节点将配置世代作为自己的当前世代。最终会成为 Cluster 中所有节点的当前世代。

#### Slave 选举和提升

　　当一个 Master 失效时，在符合以下情况时，将在该 Master 的所有 Slave 的范围内进行选举和提升：

　　1、该 Master 为 FAIL。
　　2、该 Master 失效前负责至少一个 slot。
　　3、Master 和它的 Slave 之间的复制连接已经断开。

　　**被选举者的范围：该 Master 的所有 Slaves。**

　　**选民的范围：所有的 Master。**

　　**投票规则：**

　　1、在 `NOTE_TIMEOUT * 2` 时间之内，一个 Master 只能选中唯一一个 Slave，超出此时间可以再选其它Slave。目的是为了确保选举结果唯一。
　　2、世代小于当前世代的投票将被忽略。目的的是为了确保该投票不是过期的之前的选举的投票。
　　3、在符合`1`和`2`的前提下，投票赞成该 Slave 当选。

　　**获胜规则：**

* 某个 Slave 获得大多数的 Master 的投票。

　　**投票时间：**

* 在选举开始之后的 `2 * NODE_TIMEOUT` 时间内投票有效。超时则选举自动取消。下一次选举至少在 `4 * NOTE_TIMEOUT` 时间之后才能举行。

　　**选举开始：**

* Slave 发出 `FAILOVER_AUTH_REQUEST` 消息。

　　**投票：**

* Master 发出 `FAILOVER_AUTH_ACK` 消息。

　　在 Master 失效之后，Slave 按照以下公式计算结果决定何时开始发起选举请求：

```
DELAY = 500 milliseconds + random delay between 0 and 500 milliseconds + SLAVE_RANK * 1000 milliseconds.
```

　　其中的`固定值`部分，确保 Master 的 FAIL 状态已经传播到每个节点；`随机值`部分则避免了多个 Slaves 在同一时刻发起选举请求。

　　`SLAVE_RANK` 由 Slave 从 Master 那里复制了多少数据决定，复制最多的 Slave 的 RANK为 0，第 2 多的 RANK 为 1，依次类推。

　　基于投票规则，最早发起投票的 Slave，最有可能获胜。因此复制最多的 Slave 可能最先进行选举。

### slots 配置传播

　　Redis Cluster 采用两种方式进行各个 Master 节点的 Slots 配置信息的传播。所谓 Slots 配置信息，即 Master 负责存储哪几个 Slots。

#### 心跳消息

　　在 `PING/PONG` 消息中包含了所负责的 Slots 配置信息。

#### UPDATE 消息

　　当一个节点收到 `PING/PONG` 消息后，如果发现发送者的世代小于自己的世代，则向其发送 `UPDATE` 消息，来更新其有可能已经过时的 Slots 配置信息。如果发现发送者的世代大于自己的世代，则用消息中的 Slots 配置信息更新自己的 Slots 配置信息。 

### Resharding

　　Redis Cluster 的 `Resharding` 是指在 Cluster 内的节点之间转移 Slots 中的键数据，一个 Slot 之前由某个节点负责，在 Resharding 之后，可能由另外一个节点负责。

### 复制迁移

　　Redis Cluster 在节点失效时，可能进行自动的 Slave 节点重新配置，修改了 Cluster 中部分节点的 Master-Slave 复制关系，即`复制迁移`。

　　**假定场景：**

　　Cluster中有三个 Master 节点：A、B、C。A有1个 Slave 节点 A1，B有1个 Slave 节点B1，C有2个 Slave 节点C1和C2。A节点失效了，将A1节点提升为 Master 节点。

　　考虑不进行自动的 Slave 节点的复制迁移：

　　如果A失效了，则会将唯一的 Slave 节点A1提升为 Master 节点，但是它没有 Slave 节点。这时如果A1节点又失效了，则原来A节点负责的 Slots 将失效，导致整个 cluster 不能正常工作。

　　考虑进行自动的slave节点的复制迁移：

　　如果A节点失效了，将唯一的 Slave 节点A1提升为 Master 节点，由于它没有 Slave 节点，此时发现C节点有2个 Slave 节点，将其中的C2节点重新配置为A1节点的子节点。这时，Cluster 中每个 Master 节点至少有1个 Slave 节点。如果A1节点失效，可将C2节点提升为 Master。这样的结果是提高了整个 Cluster 的可用性。

# Redis Cluster 部署

### 部署步骤

　　先前已经安装完毕的三台机器，每台机器中有两个 Redis。如下：

| IP | 端口号 |
| --- | --- |
| 192.168.1.135 | 6379 |
| 192.168.1.135 | 6380 |
| 192.168.1.136 | 7379 |
| 192.168.1.136 | 7380 |
| 192.168.1.137 | 8379 |
| 192.168.1.137 | 8380 |

　　1、修改配置文件信息，以下需修改处：

```
bind 0.0.0.0						// 测试环节，任何地址都可连接
port 6379							// 修改成对应的端口号
daemonize yes						// 后台运行
pidfile /var/run/redis_6379.pid		// pid文件
logfile "./redis.log"				// 日志
appendonly yes						// 开启 aop 备份
appendfsync always					// 每写一条 备份 一次
cluster-enabled yes					// 开启 Redis Cluster 
cluster-config-file nodes-6379.conf	// 记录集群信息，不用手动维护，Redis Cluster 会自动维护
cluster-node-timeout 15000			// Cluster 超时时间
cluster-require-full-coverage no	// 只要有结点宕机导致16384个槽没全被覆盖，整个集群就全部停止服务，所以一定要改为no
```

　　2、依次启动6个 Redis 实例：

```
[root@localhost redis1]# ./bin/redis-server redis.conf
```

　　3、启动完毕之后，使用 Redis 源码包中附带的 Ruby 工具创建集群。

　　因为要使用 Ruby 工具，所以要先保证有 Ruby 环境，并且安装 Redis 插件。

```
[root@localhost redis-4.0.10]# yum -y install ruby ruby-devel rubygems
[root@localhost redis-4.0.10]# gem install redis
```

　　由于网络原因，上步的 `gem install redis` 可能下载失败，所以可以使用本地安装，[点击下载](http://img.lynchj.com/CSDN/redis-3.2.1.gem)本地安装文件进行安装。

```
[root@localhost redis-4.0.10]# gem install redis-3.2.1.gem
Successfully installed redis-3.2.1
Parsing documentation for redis-3.2.1
Installing ri documentation for redis-3.2.1
1 gem installed
[root@localhost redis-4.0.10]# gem list --local

*** LOCAL GEMS ***

bigdecimal (1.2.0)
io-console (0.4.2)
json (1.7.7)
psych (2.0.0)
rdoc (4.0.0)
redis (3.2.1)
```

　　 **开始使用 Redis 自带集群管理工具创建集群**

```
[root@localhost redis-4.0.10]# ./src/redis-trib.rb create --replicas 1 192.168.117.135:6379 192.168.117.135:6380 192.168.117.136:7379 192.168.117.136:7380 192.168.117.137:8379 192.168.117.137:8380
>>> Creating cluster
>>> Performing hash slots allocation on 6 nodes...
Using 3 masters:
192.168.117.135:6379
192.168.117.136:7379
192.168.117.137:8379
Adding replica 192.168.117.136:7380 to 192.168.117.135:6379
Adding replica 192.168.117.137:8380 to 192.168.117.136:7379
Adding replica 192.168.117.135:6380 to 192.168.117.137:8379
M: 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a 192.168.117.135:6379
   slots:0-5460 (5461 slots) master
S: 5f6541f54bf8a7057bde3a3187701cdb617d35a6 192.168.117.135:6380
   replicates 60fd13209f9c0ef15cf7c196f1402c2643dc1b41
M: 81c6232822552e7fb9ec95b32ffad5d58f308cb4 192.168.117.136:7379
   slots:5461-10922 (5462 slots) master
S: 9925473a2e7473e2fc2fe6788f798ef6643cadec 192.168.117.136:7380
   replicates 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
M: 60fd13209f9c0ef15cf7c196f1402c2643dc1b41 192.168.117.137:8379
   slots:10923-16383 (5461 slots) master
S: cddb39f323cec35d5ef013a26219e83994113ea4 192.168.117.137:8380
   replicates 81c6232822552e7fb9ec95b32ffad5d58f308cb4
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join......
>>> Performing Cluster Check (using node 192.168.117.135:6379)
M: 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a 192.168.117.135:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
M: 60fd13209f9c0ef15cf7c196f1402c2643dc1b41 192.168.117.137:8379
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
S: 9925473a2e7473e2fc2fe6788f798ef6643cadec 192.168.117.136:7380
   slots: (0 slots) slave
   replicates 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
S: cddb39f323cec35d5ef013a26219e83994113ea4 192.168.117.137:8380
   slots: (0 slots) slave
   replicates 81c6232822552e7fb9ec95b32ffad5d58f308cb4
M: 81c6232822552e7fb9ec95b32ffad5d58f308cb4 192.168.117.136:7379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
S: 5f6541f54bf8a7057bde3a3187701cdb617d35a6 192.168.117.135:6380
   slots: (0 slots) slave
   replicates 60fd13209f9c0ef15cf7c196f1402c2643dc1b41
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
```

> 　　看到如 `[OK] All 16384 slots covered.` 的提示说明集群创建成功。

　　仔细查看上面的日志信息，M：主节点。S：从节点。可以看到，自动把6个 Redis 分成了 3 个主节点和 3 个从节点，并且 3 个主节点所在服务器是不同的，而从节点的分布也是在三台服务器上各有一个，并且还不和其自己的主节点在同一个服务器，这样做的好处是保证了集群的高可用性。

　　4、查看集群

　　随便连接一个 Redis 进行查看集群操作，连接 Redis 记得使用 `-c` 参数，表示启用集群模式，可以使用重定向等功能。

```
[root@localhost redis1]# ./bin/redis-cli -c -p 6379
127.0.0.1:6379> CLUSTER NODES
60fd13209f9c0ef15cf7c196f1402c2643dc1b41 192.168.117.137:8379@18379 master - 0 1530604479000 5 connected 10923-16383
670bd5ae86dae00c8cd8c3d31cd84e9faea4234a 192.168.117.135:6379@16379 myself,master - 0 1530604474000 1 connected 0-5460
9925473a2e7473e2fc2fe6788f798ef6643cadec 192.168.117.136:7380@17380 slave 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a 0 1530604480306 4 connected
cddb39f323cec35d5ef013a26219e83994113ea4 192.168.117.137:8380@18380 slave 81c6232822552e7fb9ec95b32ffad5d58f308cb4 0 1530604477276 6 connected
81c6232822552e7fb9ec95b32ffad5d58f308cb4 192.168.117.136:7379@17379 master - 0 1530604479000 3 connected 5461-10922
5f6541f54bf8a7057bde3a3187701cdb617d35a6 192.168.117.135:6380@16380 slave 60fd13209f9c0ef15cf7c196f1402c2643dc1b41 0 1530604480000 5 connected
```

　　也可以这样单独查看主节点或者从节点

```
[root@localhost redis1]# ./bin/redis-cli -c -p 6379 cluster nodes | grep master
60fd13209f9c0ef15cf7c196f1402c2643dc1b41 192.168.117.137:8379@18379 master - 0 1530604808096 5 connected 10923-16383
670bd5ae86dae00c8cd8c3d31cd84e9faea4234a 192.168.117.135:6379@16379 myself,master - 0 1530604809000 1 connected 0-5460
81c6232822552e7fb9ec95b32ffad5d58f308cb4 192.168.117.136:7379@17379 master - 0 1530604810104 3 connected 5461-10922
[root@localhost redis1]# ./bin/redis-cli -c -p 6379 cluster nodes | grep slave
9925473a2e7473e2fc2fe6788f798ef6643cadec 192.168.117.136:7380@17380 slave 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a 0 1530604813120 4 connected
cddb39f323cec35d5ef013a26219e83994113ea4 192.168.117.137:8380@18380 slave 81c6232822552e7fb9ec95b32ffad5d58f308cb4 0 1530604814126 6 connected
5f6541f54bf8a7057bde3a3187701cdb617d35a6 192.168.117.135:6380@16380 slave 60fd13209f9c0ef15cf7c196f1402c2643dc1b41 0 1530604816135 5 connected
```

　　**上方信息展示说明如下：**

* 节点ID
* IP:端口
* 标志：master，slave，myself,，fail，…
* 如果是个从节点,，这里是它的主节点的 `NODE ID`
* 集群最近一次向节点发送 `PING` 命令之后， 过去了多长时间还没接到回复。.
* 节点最近一次返回 `PONG` 回复的时间。
* 节点的配置纪元（`configuration epoch`）：详细信息请参考 [Redis 集群规范](http://www.redis.cn/topics/cluster-spec.html) 。
* 本节点的网络连接情况：例如：`connected` 。
* 节点目前包含的槽：例如 192.168.117.135:6379 目前包含号码为 0 至 5460 的哈希槽。

### Redis Cluster 常用命令

| 格式 | 说明 |
| --- | --- |
| CLUSTER NODES | 查看集群节点信息 |
| CLUSTER REPLICATE `<master-id>` | 进入一个从节点 Redis，切换其主节点 |
| CLUSTER MEET `<ip:port>` | 发现一个新节点，新增主节点 |
| CLUSTER FORGET `<id>` | 忽略一个节点，前提是他不能是有 Solts 的主节点 |
| CLUSTER FAILOVER | 手动故障转移 |

　　**CLUSTER FAILOVER 命令：**

　　有的时候在主节点没有任何问题的情况下强制手动故障转移也是很有必要的，比如想要升级主节点的 Redis 进程，我们可以通过故障转移将其转为 Slave 再进行升级操作来避免对集群的可用性造成很大的影响。

　　Redis 集群使用 `CLUSTER FAILOVER` 命令来进行故障转移，不过要在被转移的主节点的从节点上执行该命令，手动故障转移比主节点失败自动故障转移更加安全，因为手动故障转移时客户端的切换是在确保新的主节点完全复制了失败的旧的主节点数据的前提下下发生的，所以避免了数据的丢失。

　　其基本过程如下：**客户端不再链接我们淘汰的主节点，同时主节点向从节点发送复制偏移量，从节点得到复制偏移量后故障转移开始，接着通知主节点进行配置切换，当客户端在旧的 Master 上解锁后重新连接到新的主节点上。**

### redis-trib.rb 常用命令

#### redis-trib.rb add-node 

* **添加主节点：redis-trib.rb add-node `<new_ip:new_port>` `<exist_ip:exist_port>`**

　　使用 `add-node` 命令来添加节点，第一个参数是新节点的地址，第二个参数是任意一个已经存在的节点的IP和端口，这样，新节点就会添加到集群中。

* **添加从节点：redis-trib.rb add-node --slave `<new_ip:new_port>` `<exist_ip:exist_port>`**

　　此处的命令和添加一个主节点命令类似，此处并没有指定添加的这个从节点的主节点，这种情况下系统会在其他的集群中的主节点中随机选取一个作为这个从节点的主节点。

* **添加指定主节点 id 的从节点：redis-trib.rb add-node --slave --master-id `<主节点id>` `<new_ip:new_port>` `<exist_ip:exist_port>`**

　　也可以使用 `CLUSTER REPLICATE` 命令，这个命令也可以改变一个从节点的主节点，要在从节点中使用哦。

#### redis-trib del-node

* **删除节点：redis-trib del-node `<exist_ip:exist_port>` `<node-id>`**

　　第一个参数是任意一个节点的地址（确定集群所在），第二个节点是你想要移除的节点地址。

　　使用同样的方法移除主节点，不过在移除主节点前，需要确保这个主节点是空的（无 Solts）。如果不是空的，需要将这个节点的数据重新分片到其他主节点上。

　　替代移除主节点的方法是手动执行故障恢复，被移除的主节点会作为一个从节点存在，不过这种情况下不会减少集群节点的数量，也需要重新分片数据。

#### redis-trib.rb reshard

* **从新分片：redis-trib.rb reshard `<exist_ip:exist_port>`**

　　你只需要指定集群中其中一个节点的地址，redis-trib 就会自动找到集群中的其他节点。

　　目前 redis-trib 只能在管理员的协助下完成重新分片的工作，要让 redis-trib 自动将哈希槽从一个节点移动到另一个节点，目前来说还做不到。

　　**下列尝试从 `192.168.117.136:7379` 主节点移动 1000 个 Solt 到主节点 `192.168.117.136:7379`：

```
[root@localhost redis-4.0.10]# ./src/redis-trib.rb reshard 192.168.117.135:6379
>>> Performing Cluster Check (using node 192.168.117.135:6379)
M: 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a 192.168.117.135:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
S: 60fd13209f9c0ef15cf7c196f1402c2643dc1b41 192.168.117.137:8379
   slots: (0 slots) slave
   replicates 5f6541f54bf8a7057bde3a3187701cdb617d35a6
S: 9925473a2e7473e2fc2fe6788f798ef6643cadec 192.168.117.136:7380
   slots: (0 slots) slave
   replicates 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
S: cddb39f323cec35d5ef013a26219e83994113ea4 192.168.117.137:8380
   slots: (0 slots) slave
   replicates 81c6232822552e7fb9ec95b32ffad5d58f308cb4
M: 81c6232822552e7fb9ec95b32ffad5d58f308cb4 192.168.117.136:7379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
M: 5f6541f54bf8a7057bde3a3187701cdb617d35a6 192.168.117.135:6380
   slots:10923-16383 (5461 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 1000					# 移动 1000 个 Slot
What is the receiving node ID? 81c6232822552e7fb9ec95b32ffad5d58f308cb4		# 移动到哪个主节点
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1:670bd5ae86dae00c8cd8c3d31cd84e9faea4234a						# 从哪个主节点移动，`all`代表各个主节点平均移一点
Source node #2:done															# 完成

Ready to move 1000 slots.
  Source nodes:
    M: 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a 192.168.117.135:6379
   slots:0-5460 (5461 slots) master
   1 additional replica(s)
  Destination node:
    M: 81c6232822552e7fb9ec95b32ffad5d58f308cb4 192.168.117.136:7379
   slots:5461-10922 (5462 slots) master
   1 additional replica(s)
  Resharding plan:
    Moving slot 0 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 1 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 2 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 3 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 4 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 5 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 6 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 7 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
*****************************此处省略*****************************
    Moving slot 995 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 996 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 997 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 998 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
    Moving slot 999 from 670bd5ae86dae00c8cd8c3d31cd84e9faea4234a
Do you want to proceed with the proposed reshard plan (yes/no)? yes					# 继续执行重新分片吗？
Moving slot 0 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 1 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 2 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 3 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 4 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 5 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 6 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 7 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 8 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 9 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 10 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 11 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 12 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 13 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 14 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 15 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 16 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 17 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 18 from 192.168.117.135:6379 to 192.168.117.136:7379:
Moving slot 19 from 192.168.117.135:6379 to 192.168.117.136:7379:
*****************************此处省略*****************************
```

> 　　已上分片结束

　　查看最新的结果：

![这里写图片描述](http://img.lynchj.com/fb52b8eb2e0d4a8abc012652700c2b59.png)

# 遇到的问题以及解决方案

### 新增节点直接变成从节点

> 这个是因为要新增的那个 Redis 节点中有原来残留的数据信息，新增前请把 AOF 和 RDB 备份的文件都删除掉，并且进入准备新增的 Redis 中执行一下下面两条命令

```
127.0.0.1:6379> FLUSHALL 
127.0.0.1:6379> CLUSTER RESET
```
