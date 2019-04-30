[toc]
　　上一次纪录了一篇《[MySQL 主主复制 高可用负载均衡集群](https://blog.csdn.net/wo18237095579/article/details/80944800)》，这个看似不会有问题的主主复制其实存在了两个大问题

　　1、事务不统一，造成数据的不统一。
　　多线程环境下，如果线程之间分负载到不同的后端数据库中，那么事务就是不统一的，都能获取锁，都能`FOR UPDATE`，都可以提交。然后再通过主从复制交换数据，其实有可能已经是垃圾数据。

　　2、主从复制延迟问题。
　　由于主从复制是一个异步操作，也就是说当一条线程在A数据库中操作完数据库并提交事务之后，这个时候数据可能还没有同步到B数据库，但是第二条线程就已经连接上B数据库进行操作了。这样就尝试了脏数据。

　　这次记录下使用 `Galera Cluster` 的一些笔记，`Galera Cluster` 在使用过程中就可以很好的避免上述问题。

# 简介

## 何谓Galera Cluster

　　就是集成了 `Galera插件` 的MySQL集群，是一种新型的，数据不共享的，高度冗余的高可用方案，目前 `Galera Cluster`有两个版本，分别是 `Percona Xtradb Cluster`及 `MariaDB Cluster`，都是基于 Galera 的，所以这里都统称为 Galera Cluster 了，因为 Galera 本身是具有多主特性的，所以 Galera Cluster 也就是 `multi-master` 的集群架构，如图1所示：

![架构图](http://img.lynchj.com/b497802cb5f542f0b6b384e2ad5fa583.png)

![架构图](http://img.lynchj.com/e0f17d9813f8413ebe346b6352e33a25.png)

　　上图中有三个实例，组成了一个集群，而这三个节点与普通的主从架构不同，它们都可以作为主节点，三个节点是对等的，这种一般称为 `multi-master` 架构，当有客户端要写入或者读取数据时，随便连接哪个实例都是一样的，读到的数据是相同的，写入某一个节点之后，集群自己会将新数据同步到其它节点上面，这种架构不共享任何数据，是一种`高冗余架构`。

　　一般的使用方法是：在这个集群上面，再搭建一个中间层，这个中间层的功能包括建立连接、管理连接池，负责使三个实例的负载基本平衡，负责在客户端与实例的连接断开之后重连，也可以负责读写分离（在机器性能不同的情况下可以做这样的优化）等等，使用这个中间层之后，由于这三个实例的架构在客户端方面是透明的，客户端只需要指定这个集群的数据源地址，连接到中间层即可，中间层会负责客户端与服务器实例连接的传递工作，由于这个架构支持多点写入，所以完全避免了主从复制经常出现的数据不一致的问题，从而可以做到主从读写切换的高度优雅，在不影响用户的情况下，离线维护等工作，MySQL 的高可用，从此开始，非常完美。

## 为什么需要 Galera Cluster

　　MySQL 在互联网时代，可谓是深受世人瞩目的。给社会创造了无限价值，随之而来的是，在 MySQL 基础之上，产生了形形色色的使用方法、架构及周边产品。在这方面，已经有很多成熟的被人熟知的产品，比如 MHA、MMM 等传统组织架构，而这些架构是每个需要数据库高可用服务方案的入门必备选型。

　　不幸的是，传统架构的使用，一直被人们所诟病，因为 MySQL 的主从模式，天生的不能完全保证数据一致，很多大公司会花很大人力物力去解决这个问题，而效果却一般，可以说，只能是通过牺牲性能，来获得数据一致性，但也只是在降低数据不一致性的可能性而已。所以现在就急需一种新型架构，从根本上解决这样的问题，天生的摆脱掉主从复制模式这样的`美中不足`之处了。

　　幸运的是，MySQL 的福音来了，Galera Cluster 就是我们需要的——从此变得完美的架构。

　　相比传统的主从复制架构，Galera Cluster 解决的最核心问题是，在三个实例（节点）之间，它们的关系是对等的，`multi-master` 架构的，在多节点同时写入的时候，能够保证整个集群数据的一致性，完整性与正确性。

　　在传统 MySQL 的使用过程中，也不难实现一种 `multi-master` 架构，但是一般需要上层应用来配合，比如先要约定每个表必须要有自增列，并且如果是2个节点的情况下，一个节点只能写偶数的值，而另一个节点只能写奇数的值，同时2个节点之间互相做复制，因为2个节点写入的东西不同，所以复制不会冲突，在这种约定之下，可以基本实现多master的架构，也可以保证数据的完整性与一致性。但这种方式使用起来还是有限制，同时还会出现复制延迟，并且不具有扩展性，不是真正意义上的集群。

## Galera Cluster 如何解决上述问题

　　现在已经知道，Galera Cluster 是 MySQL 封装了具有高一致性，支持多点写入的同步通信模块 Galera 而做的，它是建立在 MySQL 同步基础之上的，使用 Galera Cluster 时，应用程序可以直接读、写某个节点的最新数据，并且可以在不影响应用程序读写的情况下，下线某个节点，因为支持多点写入，使得 `Failover` 变得非常简单。

　　所有的 Galera Cluster，都是对 Galera 所提供的接口 API 做了封装，这些 API 为上层提供了丰富的状态信息及回调函数，通过这些回调函数，做到了真正的多主集群，多点写入及同步复制，这些 API 被称作是 Write-Set Replication API，简称为 `wsrep API`。

　　通过这些 API，Galera Cluster 提供了基于验证的复制，是一种乐观的同步复制机制，一个将要被复制的事务（称为写集），不仅包括被修改的数据库行，还包括了这个事务产生的所有`Binlog`，每一个节点在复制事务时，都会拿这些写集与正在 APPLY 队列的写集做比对，如果没有冲突的话，这个事务就可以继续提交，或者是 APPLY，这个时候，这个事务就被认为是提交了，然后在数据库层面，还需要继续做事务上的提交操作。

　　这种方式的复制，也被称为是`虚拟同步复制`，实际上是一种逻辑上的同步，因为每个节点的写入和提交操作还是独立的，更准确的说是`异步`的，Galera Cluster 是建立在一种乐观复制的基础上的，假设集群中的每个节点都是同步的，那么加上在写入时，都会做验证，那么理论上是不会出现不一致的，当然也不能这么乐观，如果出现不一致了，比如主库（相对）插入成功，而从库则出现主键冲突，那说明此时数据库已经不一致，这种时候 Galera Cluster 采取的方式是将出现不一致数据的节点踢出集群，其实是自己 shutdown 了。

　　而通过使用 Galera，它在里面通过判断键值的冲突方式实现了真正意义上的 multi-master，Galera Cluster 在 MySQL 生态中，在高可用方面实现了非常重要的提升，目前 Galera Cluster 具备的功能包括如下几个方面：

* **多主架构：**真正的多点读写的集群，在任何时候读写数据，都是最新的。
* **同步复制：**集群不同节点之间数据同步，没有延迟，在数据库挂掉之后，数据不会丢失。
* **并发复制：**从节点在 APPLY 数据时，支持并行执行，有更好的性能表现。
* **故障切换：**在出现数据库故障时，因为支持多点写入，切的非常容易。
* **热插拔：**在服务期间，如果数据库挂了，只要监控程序发现的够快，不可服务时间就会非常少。在节点故障期间，节点本身对集群的影响非常小。
* **自动节点克隆：**在新增节点，或者停机维护时，增量数据或者基础数据不需要人工手动备份提供，Galera Cluster 会自动拉取在线节点数据，最终集群会变为一致。
* **对应用透明：**集群的维护，对应用程序是透明的，几乎感觉不到。 以上几点，足以说明 Galera Cluster 是一个既稳健，又在数据一致性、完整性及高性能方面有出色表现的高可用解决方案。

## 注意点

　　在运维过程中，有些技术特点还是需要注意的，这样才能做到知此知彼，百战百胜，因为现在 MySQL 主从结构的集群已经都是被大家所熟知的了，而 Galera Cluster 是一个新的技术，是一个在不断成熟的技术，所以很多想了解这个技术的同学，能够得到的资料很少，除了官方的手册之外，基本没有一些讲得深入的，用来传道授业解惑的运维资料，这无疑为很多同学设置了不低的门槛，最终有很多人因为一些特性，导致最终放弃了 Galera Cluster 的选择。

　　目前熟知的一些特性，或者在运维中需要注意的一些特性，有以下几个方面：

* **Galera Cluster 写集内容：** Galera Cluster 复制的方式，还是基于 Binlog 的，这个问题，也是一直被人纠结的，因为目前 Percona Xtradb Cluster 所实现的版本中，在将 Binlog 关掉之后，还是可以使用的，这误导了很多人，其实关掉之后，只是不落地了，表象上看上去是没有使用 Binlog 了，实际上在内部还是悄悄的打开了的。除此之外，写集中还包括了事务影响的所有行的主键，所有主键组成了写集的 KEY，而 Binlog 组成了写集的 DATA，这样一个 KEY-DATA 就是写集。KEY 和 DATA 分别具有不同的作用的，KEY 是用来验证的，验证与其它事务没有冲突，而 DATA 是用来在验证通过之后，做 APPLY 的。

* **Galera Cluster 的并发控制：**现在都已经知道，Galera Cluster 可以实现集群中，数据的高度一致性，并且在每个节点上，生成的 Binlog 顺序都是一样的，这与 Galera 内部，实现的并发控制机制是分不开的。所有的上层到下层的同步、复制、执行、提交都是通过并发控制机制来管理的。这样才能保证上层的逻辑性，下层数据的完整性等。

![Galera 原理图](http://img.lynchj.com/09e880ad0b004d8eade2a239a741163f.png)

* 上图是从官方手册中截取的，从图中可以大概看出，从事务执行开始，到本地执行，再到写集发送，再到写集验证，再到写集提交的整个过程，以及从节点（相对）收到写集之后，所做的写集验证、写集 APPLY 及写集提交操作，通过对比这个图，可以很好的理解每一个阶段的意义及性能等，下面就每一个阶段以及其并发控制行为做一个简单的介绍：
　　a、**本地执行：**这个阶段，是事务执行的最初阶段，可以说，这个阶段的执行过程，与单点 MySQL 执行没什么区别，并发控制当然就是数据库的并发控制了，而不是 Galera Cluster 的并发控制了。
　　b、**写集发送：**在执行完之后，就到了提交阶段，提交之前首先将产生的写集广播出去，而为了保证全局数据的一致性，在写集发送时，需要串行，这个就属于 Galera Cluster 并发控制的一部分了。
　　c、**写集验证：**这个阶段，就是我们通常说的 Galera Cluster 的验证了，验证是将当前的事务，与本地写集验证缓存集来做验证，通过比对写集中被影响的数据库 KEYS，来发现有没有相同的，来确定是不是可以验证通过，那么这个过程，也是串行的。
　　d、**写集提交：**这个阶段，是一个事务执行时的最后一个阶段了，验证完成之后，就可以进入提交阶段了，因为些时已经执行完了的，而提交操作的并发控制，是可以通过参数来控制其行为的，即参数 `repl.commit_order`，如果设置为3，表示提交就是串行的了，而这也是推荐的（默认值）的一种设置，因为这样的结果是，集群中不同节点产生的 Binlog 是完全一样的，运维中带来了不少好处和方便。其它值的解释，以后有机会再做讲解。
　　e、**写集APPLY：**这个阶段，与上面的几个在流程上不太一样，这个阶段是从节点做的事情，从节点只包括两个阶段，即写集验证和写集 APPLY，写集 APPLY 的并发控制，是与参数 `wsrep_slave_threads` 有关系的，本身在验证之后，确定了相互的依赖关系之后，如果确定没有关系的，就可以并行了，而并行度，就是参数 `wsrep_slave_threads` 的事情了。`wsrep_slave_threads` 可以参照参数 `wsrep_cert_deps_distance` 来设置。

## 有很多坑?

　　有很多同学，在使用过 Galera Cluster 之后，发现很多问题，最大的比如 DDL 的执行，大事务等，从而导致服务的不友好，这也是导致很多人放弃的原因。

* **DDL执行卡死传说：**使用过的同学可能知道，在 Galera Cluster 中执行一个大的改表操作，会导致整个集群在一段时间内，是完全写入不了任何事务的，都卡死在那里，这个情况确实很严重，导致线上完全不可服务了，原因还是并发控制，因为提交操作设置为串行的，DDL 执行是一个提交的过程，那么串行执行改表，当然执行多久，就卡多久，直到改表执行完，其它事务也就可以继续操作了，这个问题现在没办法解决，但我们长期使用下来发现，小表可以这样直接操作，大一点或者更大的，都是通过 `osc（pt-online-schema-change）` 来做，这样就很好的避免了这个问题。

* **挡我者死：**由于 Galera Cluster 在执行 DDL 时，是 `Total Ordered Isolation（wsrep_OSU_method=TOI）` 的，所以必须要保证每个节点都是同时执行的，当然对于不是 DDL 的，也是 `Total Order` 的，因为每一个事务都具有同一个 `GTID` 值，DDL 也不例外，而 DDL 涉及到的是表锁，`MDL` 锁（Meta Data Lock），只要在执行过程中，遇到了 MDL 锁的冲突，所有情况下，都是 DDL 优先，将所有的使用到这个对象的事务，统统杀死，不管是读事务，还是写事务，被杀的事务都会报出死锁的异常，所以这也是一个 Galera Cluster 中，关于 DDL 的闻名遐迩的坑。不过这个现在确实没有办法解决，也没办法避免，不过这个的影响还算可以接受，先可以忍忍。

* **不死之身：**继上面的`挡我者死`，如果集群真的被一个 DDL 卡死了，导致整个集群都动不了了，所有的写请求都 Hang 住了，那么可能会有人想一个妙招，说赶紧杀死，直接在每个节点上面输入 `kill connection_id`，等等类似的操作，那么此时，很不愿意看到的信息报了出来：`You are not owner of thread connection_id`。此时可能有些同学要哭了，不过这种情况下，确实没有什么好的解决方法（其实这个时候，一个故障已经发生了，一年的 KPI 也许已经没有了，就看敢不敢下狠手了），要不就等 DDL 执行完成（所有这个数据库上面的业务都处于不可服务状态），要不就将数据库直接 Kill 掉，快速重启，赶紧恢复一个节点提交线上服务，然后再考虑集群其它节点的数据增量的同步等，这个坑非常大，也是在 Galera Cluster 中，最大的一个坑，需要非常小心，避免出现这样的问题。

## 适用场景

　　现在对 Galera Cluster 已经有了足够了解，但这样的“完美”架构，在什么场景下才可以使用呢？或者说，哪种场景又不适合使用这样的架构呢？针对它的缺点，及优点，我们可以扬其长，避其短。可以通过下面几个方面，来了解其适用场景。

* **数据强一致性：**因为 Galera Cluster，可以保证数据强一致性的，所以它更适合应用于对数据一致性和完整性要求特别高的场景，比如交易，正是因为这个特性，去哪儿网才会成为使用 Galera Cluster 的第一大户。
* **多点写入：**这里要强调多点写入的意思，不是要支持以多点写入的方式提供服务，更重要的是，因为有了多点写入，才会使得在 DBA 正常维护数据库集群的时候，才会不影响到业务，做到真正的无感知，因为只要是主从复制，就不能出现多点写入，从而导致了在切换时，必然要将老节点的连接断掉，然后齐刷刷的切到新节点，这是没办法避免的，而支持了多点写入，在切换时刻允许有短暂的多点写入，从而不会影响老的连接，只需要将新连接都路由到新节点即可。这个特性，对于交易型的业务而言，也是非常渴求的。
* **性能：**Galera Cluster，能支持到强一致性，毫无疑问，也是以牺牲性能为代价，争取了数据一致性，但要问：”性能牺牲了，会不会导致性能太差，这样的架构根本不能满足需求呢？”这里只想说的是，这是一个权衡过程，有多少业务，QPS 大到 Galera Cluster 不能满足的？我想是不多的（当然也是有的，可以自行做一些测试），在追求非常高的极致性能情况下，也许单个的 Galera Cluster 集群是不能满足需求的，但毕竟是少数了，所以够用就好，Galera Cluster 必然是 MySQL 方案中的佼佼者。

# 集群搭建

## 搭建环境

* CentOS 7.4
　　192.168.117.142
　　192.168.117.145
　　192.168.117.146

## 安装依赖包

　　建议使用[阿里云](https://blog.csdn.net/csdn_kou/article/details/81096744) yum 源

```
yum -y install gcc gcc-c++ openssl openssl-devel lsof socat perl boost-devel rsync jemalloc libaio libaio-devel
```

> 　　`jemalloc` 找不到的话安装一下数据源：rpm -ivh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm

## 安装 Galera Cluster For MySQL

### 先去官网下载

　　[下载地址](http://galeracluster.com/downloads/)，全部下载下来。

![下载步骤](http://img.lynchj.com/6b6b30ca1385416babfd4267336a7951.png)

　　还有 Galera 3 主从复制支持库，[下载地址](http://galeracluster.com/downloads/)

![下载步骤](http://img.lynchj.com/018f86885e1d49b2aa355f1c92b25bde.png)

### 下载到服务器中

```
wget http://releases.galeracluster.com/galera-3/centos/7/x86_64/galera-3-25.3.23-2.el7.x86_64.rpm
wget http://releases.galeracluster.com/mysql-wsrep-5.7/centos/7/x86_64/mysql-wsrep-5.7-5.7.21-25.14.el7.x86_64.rpm
wget http://releases.galeracluster.com/mysql-wsrep-5.7/centos/7/x86_64/mysql-wsrep-client-5.7-5.7.21-25.14.el7.x86_64.rpm
wget http://releases.galeracluster.com/mysql-wsrep-5.7/centos/7/x86_64/mysql-wsrep-common-5.7-5.7.21-25.14.el7.x86_64.rpm
wget http://releases.galeracluster.com/mysql-wsrep-5.7/centos/7/x86_64/mysql-wsrep-devel-5.7-5.7.21-25.14.el7.x86_64.rpm
wget http://releases.galeracluster.com/mysql-wsrep-5.7/centos/7/x86_64/mysql-wsrep-libs-5.7-5.7.21-25.14.el7.x86_64.rpm
wget http://releases.galeracluster.com/mysql-wsrep-5.7/centos/7/x86_64/mysql-wsrep-libs-compat-5.7-5.7.21-25.14.el7.x86_64.rpm
wget http://releases.galeracluster.com/mysql-wsrep-5.7/centos/7/x86_64/mysql-wsrep-server-5.7-5.7.21-25.14.el7.x86_64.rpm
wget http://releases.galeracluster.com/mysql-wsrep-5.7/centos/7/x86_64/mysql-wsrep-test-5.7-5.7.21-25.14.el7.x86_64.rpm
```

### 安装

　　先关闭 SELinux 和 防火墙：`setenforce 0 && systemctl stop firewalld`

```
rpm -ivh mysql-wsrep-common-5.7-5.7.21-25.14.el7.x86_64.rpm
rpm -ivh mysql-wsrep-libs-5.7-5.7.21-25.14.el7.x86_64.rpm
rpm -ivh mysql-wsrep-client-5.7-5.7.21-25.14.el7.x86_64.rpm
rpm -ivh mysql-wsrep-server-5.7-5.7.21-25.14.el7.x86_64.rpm
rpm -ivh mysql-wsrep-libs-compat-5.7-5.7.21-25.14.el7.x86_64.rpm
rpm -ivh galera-3-25.3.23-2.el7.x86_64.rpm
```

### 配置

　　1、编辑  `vim /etc/my.cnf`，其中内容全部注释掉，增加：`!includedir /etc/my.cnf.d/`，然后创建 wsrep.cnf 文件。

```
mkdir /etc/my.cnf.d
vim /etc/my.cnf.d/wsrep.cnf

[mysqld]
user=mysql
log_timestamps=SYSTEM                                  # 这个是我们自己加的，防止日志时间和系统时间不一样
port=3306
server_id=11                                           # MySQL服务器的ID，必须是唯一的，集群各个节点也不同
explicit_defaults_for_timestamp=true
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data                     
socket=/usr/local/mysql/data/mysql.sock
pid_file=/run/mysqld/mysqld.pid
log_error=/var/log/mysql.error
wsrep_cluster_name='cs_cluster'                       # galera集群的名字，必须是统一的
wsrep-provider=/usr/lib64/galera-3/libgalera_smm.so   # wsrep提供者,必须配置(.so文件的路径在哪，就配置成哪,一般安装好后都是在这个目录下)
wsrep_node_name = node1                               # wsrep节点的ID，必须是唯一的，集群各个节点也不同
wsrep_cluster_address=gcomm://192.168.117.142:4567,192.168.117.145:4567,192.168.117.146:4567   # 集群中的其他节点地址，可以使用主机名或IP
wsrep_node_address='192.168.117.142'                 # 本机节点地址，可以使用主机名或IP
wsrep_provider_options ="gmcast.listen_addr=tcp://192.168.117.142:4567"   # 指定wsrep启动端口号,4567为默认值
wsrep_sst_donor='node1,node2,node3'                    # 一个逗号分割的节点串作为状态转移源，比如 wsrep_sst_donor=node1,node2,node3 如果node1可用，用node2,如果node2不可用，用node3，最后的逗号表明让提供商自己选择一个最优的。
wsrep_sst_method=rsync                              # 集群同步方式，我的系统没有可以用yum安装一下这个远程连接 yum -y install  rsync
wsrep_sst_auth=test:123456                         # 集群同步的用户名密码
slow_query_log=on
[client]
default-character-set=utf8
socket=/usr/local/mysql/data/mysql.sock
[mysql]
default-character-set=utf8
socket=/usr/local/mysql/data/mysql.sock
[mysqldump]
max_allowed_packet = 512M
[mysqld_safe]
malloc-lib=/usr/lib64/libjemalloc.so.1             # 这个我的系统里也没有可以用yum安装一下  yum -y install jemalloc  如果获取不到的话，下载一个数据源 rpm -ivh http://download.fedoraproject.org/pub/epel/6/x86_64/epel-release-6-8.noarch.rpm
```

### 初始化数据库

```
[root@node1 tmp]# mysqld --initialize
mysqld: Can't create directory '/usr/local/mysql/data/' (Errcode: 2 - No such file or directory)
2018-07-29T01:30:51.187574-05:00 0 [ERROR] Can't find error-message file '/usr/local/mysql/share/mysql/errmsg.sys'. Check error-message file location and 'lc-messages-dir' configuration directive.
2018-07-29T01:30:51.188583-05:00 0 [ERROR] Aborting
```

　　出错了，可以看到是没有文件夹，创建并授权

```
[root@node1 tmp]# mkdir -p /usr/local/mysql/data/
[root@node1 tmp]# chmod -R 777 /usr/local/mysql
[root@node1 tmp]# mysqld --initialize
```

　　查看数据库初始化的密码

```
[root@node1 tmp]# grep 'temporary password' /var/log/mysql.error
2018-07-29T01:31:52.768493-05:00 1 [Note] A temporary password is generated for root@localhost: yxhGizyhW1,u
```

### 初次启动数据库

　　初次启动集群的第一个数据库时需要注意

* 1、wsrep.cnf 配置中的 `wsrep_cluster_address` 需要如此配置 `wsrep_cluster_address=gcomm://` ，以后再启动的话需要配置完整。

```
[root@node1 tmp]# vim /etc/my.cnf.d/wsrep.cnf
wsrep_cluster_address=gcomm://
```

* 2、要使用 `/usr/bin/mysqld_bootstrap` 进行启动，如果出线权限问题，自行更改权限

```
[root@node1 tmp]# bash /usr/bin/mysqld_bootstrap
```

### 修改密码

```
[root@node1 /]# mysql -uroot -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 4
Server version: 5.7.21-log

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> set password=PASSWORD('123456');	# 重置 root 密码
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> GRANT ALL PRIVILEGES ON *.* TO 'test'@'%' IDENTIFIED BY '123456' WITH GRANT OPTION; # 设置 复制 账号
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> FLUSH PRIVILEGES;	# 授予权限
Query OK, 0 rows affected (0.00 sec)
```

### 查看集群状态

```
mysql> show status like "wsrep%";
+------------------------------+--------------------------------------+
| Variable_name                | Value                                |
+------------------------------+--------------------------------------+
| wsrep_local_state_uuid       | 5f2c5bbf-92f8-11e8-ac89-73a38f0c69ed |
| wsrep_protocol_version       | 8                                    |
| wsrep_last_committed         | 3                                    |
| wsrep_replicated             | 3                                    |
| wsrep_replicated_bytes       | 712                                  |
| wsrep_repl_keys              | 4                                    |
| wsrep_repl_keys_bytes        | 104                                  |
| wsrep_repl_data_bytes        | 407                                  |
| wsrep_repl_other_bytes       | 0                                    |
| wsrep_received               | 2                                    |
| wsrep_received_bytes         | 144                                  |
| wsrep_local_commits          | 0                                    |
| wsrep_local_cert_failures    | 0                                    |
| wsrep_local_replays          | 0                                    |
| wsrep_local_send_queue       | 0                                    |
| wsrep_local_send_queue_max   | 1                                    |
| wsrep_local_send_queue_min   | 0                                    |
| wsrep_local_send_queue_avg   | 0.000000                             |
| wsrep_local_recv_queue       | 0                                    |
| wsrep_local_recv_queue_max   | 1                                    |
| wsrep_local_recv_queue_min   | 0                                    |
| wsrep_local_recv_queue_avg   | 0.000000                             |
| wsrep_local_cached_downto    | 1                                    |
| wsrep_flow_control_paused_ns | 0                                    |
| wsrep_flow_control_paused    | 0.000000                             |
| wsrep_flow_control_sent      | 0                                    |
| wsrep_flow_control_recv      | 0                                    |
| wsrep_cert_deps_distance     | 1.000000                             |
| wsrep_apply_oooe             | 0.000000                             |
| wsrep_apply_oool             | 0.000000                             |
| wsrep_apply_window           | 1.000000                             |
| wsrep_commit_oooe            | 0.000000                             |
| wsrep_commit_oool            | 0.000000                             |
| wsrep_commit_window          | 1.000000                             |
| wsrep_local_state            | 4                                    |
| wsrep_local_state_comment    | Synced                               |
| wsrep_cert_index_size        | 2                                    |
| wsrep_causal_reads           | 0                                    |
| wsrep_cert_interval          | 0.000000                             |
| wsrep_incoming_addresses     | 192.168.117.142:3306                 |
| wsrep_desync_count           | 0                                    |
| wsrep_evs_delayed            |                                      |
| wsrep_evs_evict_list         |                                      |
| wsrep_evs_repl_latency       | 0/0/0/0/0                            |
| wsrep_evs_state              | OPERATIONAL                          |
| wsrep_gcomm_uuid             | 5f2be7cd-92f8-11e8-8d80-8bdecaa9b672 |
| wsrep_cluster_conf_id        | 1                                    |
| wsrep_cluster_size           | 1                                    |
| wsrep_cluster_state_uuid     | 5f2c5bbf-92f8-11e8-ac89-73a38f0c69ed |
| wsrep_cluster_status         | Primary                              |
| wsrep_connected              | ON                                   |
| wsrep_local_bf_aborts        | 0                                    |
| wsrep_local_index            | 0                                    |
| wsrep_provider_name          | Galera                               |
| wsrep_provider_vendor        | Codership Oy <info@codership.com>    |
| wsrep_provider_version       | 3.23(rac090bc)                       |
| wsrep_ready                  | ON                                   |
+------------------------------+--------------------------------------+
57 rows in set (0.01 sec)
```

　　这时候可以把配置文件 `wsrep.cnf` 中的参数 `wsrep_cluster_address` 修改回来。

```
mysql> quit
Bye
[root@node1 /]# vim /etc/my.cnf.d/wsrep.cnf
wsrep_cluster_address=gcomm://192.168.117.142:4567,192.168.117.145:4567,192.168.117.146:4567
```

　　可以使用 systemctl 查看 mysql 状态

```
[root@node1 /]# systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since Sun 2018-07-29 02:26:48 EDT; 6min ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 53014 ExecStartPost=/usr/bin/mysqld_pre_systemd --post (code=exited, status=0/SUCCESS)
  Process: 52978 ExecStart=/usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid $MYSQLD_OPTS $MYSQLD_RECOVER_START (code=exited, status=0/SUCCESS)
  Process: 52922 ExecStartPre=/usr/bin/mysqld_pre_systemd --pre (code=exited, status=0/SUCCESS)
 Main PID: 52981 (mysqld)
   CGroup: /system.slice/mysqld.service
           └─52981 /usr/sbin/mysqld --daemonize --pid-file=/var/run/mysqld/mysqld.pid --wsrep_start_position=00000000-0000-0000-0000-000000000000:-1

Jul 29 02:26:45 node1 systemd[1]: Starting MySQL Server...
Jul 29 02:26:48 node1 systemd[1]: Started MySQL Server.
```

　　剩下两台机群如此配置即可，注意几点：

* 1、初次启动不需要使用启动脚本：`/usr/bin/mysqld_bootstrap`，因为这个脚本中带有一个参数：`--wsrep-new-cluster`，代表新集群。
* 2、配置文件 `wsrep.cnf` 中的参数 `wsrep_cluster_address`，一开始就要配置好了，不能为空。
* 3、类似配置文件 `wsrep.cnf` 中的 `wsrep_node_address` 这种参数要配置成自己当前及其所在的信息。
* 4、正式环境情况下，我们不要把防火墙关闭掉，把需要使用的端口开放给指定 IP 即可，更加安全。

> 参考：
>> https://www.sohu.com/a/147032902_505779
