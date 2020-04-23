*数据库笔记
讲解：

1.在了解了redis所依赖的核心技术和卓越性能之后，我们来通过一个具体的金融行业场景
应用来看一下redis应用难题及其解决方案。

2.我们知道对于金融行业，更高的峰值处理能力，与更低的业务延迟成为核心需求之一。

3.首先为了提高处理能力速度，人们开始引入缓存技术部署在网络模型上。
这个广义的cache使得某个用户的访问数据被暂存从而在其他用户访问的时候可以快速响应。
这个缓存可以进行一个全链路的部署。

4.遇到的问题：本质原因是：在内存上的不断增长的持久化文件与redis所管理的内存文件造成了内存紧张，由此产生
了内核的波动

5.

这是一个金融行业常见的场景，oracle是国内银行较为青睐的数据库。

**缓存的产生与发展（几句话带过）

国外有很多很有名且很好用的缓存技术产品例如 Nginx，属于 HTTP 和方向代理服务器，
是由俄罗斯某站点开发，2004 年发布了第一版本，其特点在于稳定性高、系统资源消耗
低，俄罗斯国内各大型的入口网站都在使用该技术，在美国很多虚拟主机行业
也在使用该技术，其并发能力强的特点在同类技术

Cache 是一个智能化的产品，当 html 页面、JS 脚本、图片等资源被首次访
问时会已副本的形式保存在服务器与客户端之间，当用户访问请求同一个资源
信息时，Cache 会自动根据相应机制判定资源是否更新，若无更新，则直接读取
本地信息返回客户端，若服务端标明资源有更新时，先从服务器端更新资源后
返回客户端,通过这种减少客户端与服务器端访问次数的方法从而达到节约带宽
和提高响应速度的目的。Cache 的主要作用表现为减少网络资源消耗、减少网络
延迟、减小服务器的请求压力、增强系统可用性。

缓存位与每个客户端 Client 的本地硬盘内或本地浏览器的内存中，这样优势
很明显：
访问容易且高效，避免了其他资源消耗，但劣势也是显而易见的：数据资
源只能存在于一个 Client 的本地硬盘内，独立于浏览器而存在，因此不能被更多
客户端共享。

给客户端 A 同时保留一个该请求的备份，那么在客户端 B 或
者 C 请求同一个源服务器的资源时，首先发往中间服务器，中间服务器根据相
应规则与访问控制列表去匹配，若匹配成功，则无需访问源服务器而能直接并
快速响应客户端请求，我们称这种中间服务器模式为代理缓存

反向代理服务器会向其后面对应的一台
或多台负载应用服务器，根据各服务器负载情况，动态的将请求发给合适的负
载服务器，以减少源服务器负载、保障源服务器的安全，同时还能完成源服务
器的负载均衡，在整个请求传输过程中，反向代理服务器可以理解为一个配备
有大容量内存和高速磁盘的智能路由器，负责匹配请求信息，若请求信息为静
态数据时，则可直接返回，若请求信息含有动态信息，则需要保持客户请求会
话的同时，智能获取动态信息后返回客户请求
请求动态数据反向代理要维持两个方向的会话的连接，会造成一定的网络资源消耗

应用层：代码逻辑和缓存策略实现对数据的内存存储
系统层：cache库对对常会且耗费资源的数据缓存起来，不过主要更新资源
费点功夫；
数据库层：内存数据库


这样的场景需求对于 Redis 来讲很简单，即使你的用户量有上百万或者数据
变动频率为百万/分钟，这都不成问题：
//登记新综合实力评分记录
ZADD leaderboard <score> <userid>
//获取综合实力排名前 100 的排行榜
ZREVRANGE leaderboard 0 99。
//获取某用户的全服务器综合实力排名
ZRANK leaderboard <userid>


**redis存在的问题（一分钟）

作为第二数据库需要考虑与第一数据库的一致性问题
（1）持久化不成熟，快照文件必然面临数据丢失，AOF 很影响性能；
（2）存储成本高，纯内存操作，不适合海量数据；
（3）注重内存容量规划；
（4）架构扩展不是很简单。

先来看一个 Redis 的系统表现：Redis 在物理内存上使用比较多，但还没有
超过服务器的实际物理内存总容量时，服务器就已经出现不稳定甚至崩溃的现
象。曾经以为这种风险是由于同一个物理机上的多个 Redis 实例同时触发持久化
的 BGSAVE 而导致瞬间 Fork 出多个 redis 进程，造成内存占用加倍而导致的。
后来经过研究发现这种不稳地现象是由于 Redis 的持久化使用了 Buffer IO 造成
的，即 Redis 持久化文件操作都是基于物理内存的 Page Cache，而大多数数据库
系统往往为了维护一个数据的 Cache 会使用 Direct IO 来绕过这层 Page Cache，
伴随数据量的增大，Redis 持久化文件会越来越大，尤其是快照文件会比 AOF
文件大数倍，在对此文件进行读写操作时，磁盘文件中的数据都会被加载到物
理内存中作为操作系统对该文件的一层 Cache，而这层 Cache 的数据与 Redis 内
存中管理的数据实际是重复存储的，虽然内核在物理内存紧张时会做 Page Cache
的剔除工作，但内核很可能认为某块 Page Cache 更重要，而让你的进程开始
Swap ,这时你的系统就会开始出现不稳定或者崩溃了。




**Redis 复制的改进思路（一分钟）

扩容
Redis 给出了一种叫做 Presharding 的方案来解决动态扩容和数据分区的问
题，就是在同一台机器上部署多个 Redis 实例的方式，当出现容量不足的情况时，
就把多个实例拆分到不同的物理机器上，完成实际的扩容效果。
Redis 实例拆分处理步骤如下：
1、在新机器上启动好对应端口的 Redis 实例。
2、配置新端口为待迁移端口的从库。
3、待复制完成，与主库完成同步后，切换所有客户端配置到新的从库的端
口。
4、配置从库为新的主库。
5、移除老的端口实例。
6、重复上述过程迁移好所有的端口到指定服务器上。


上面分析 Redis 复制的主要缺陷在于没有增量复制，导致无法快速恢复解决
单点问题，那么对持久化方案做改进，是否能完成增量同步？
改进的持久化方案是首先写 Redis 的 AOF 文件，并对这个 AOF 文件按文件
大小进行自动分割滚动，同时关闭 Redis 的 Rewrite 命令，然后会在业务低峰时
间进行内存快照存储，并把当前的 AOF 文件位置一起写入到快照文件中，这样
我们可以使快照文件与 AOF 文件的位置保持一致性，这样我们得到了系统某一
时刻的内存快照，并且同时也能知道这一时刻对应的 AOF 文件的位置，那么当
从库发送同步命令时，我们首先会把快照文件发送给从库，然后从库会取出该
快照文件中存储的 AOF 文件位置，并将该位置发给主库，主库会随后发送该位
置之后的所有命令，以后的复制就都是这个位置之后的增量信息了，整体改进

**结合金融应用（2分钟）

因其自身具备实时性要求高、数据量大、并
发访问高和数据类型多而杂等特点，所以仅使用关系数据库是无法满足系统架
构要求的，那么结合内存数据库速度快、扩展性好的特点，设置内存数据库在
关系数据库前端来完成数据缓存处理是个不错的架构设计方案。内存数据库的
数据操作都是直接存放在内存里，相比磁盘操作的关系数据库来说，性能高出
很多，这一点正是互联网金融平台所急需的。











**与一般的集群产品一样，ORACLE RAC实现需要考虑以下几个问题：

　　1，并发管理，当集群中多个节点同时接受外部客户端访问，如何使每个客户端得到的数据一样，这就需要保证在集群中每个节点看到的数据都要一样的，这就是在节点中如何保持数据一致性的问题。ORACLE RAC主要是使用DLM(分布式锁管理器）来保证集群节点之间的数据一致性
　　2，健忘症，如我们需要修改集群配置时，只是在集中一台节点上修改，这一台节点会自动把配置信息分发到其他的节点。考虑一种这样的情形，在节点A上修改了集群配置，而这时候节点B处理关机状态。当节点B开始的时候，并不知道配置已经发生了改变，这种情形称为健忘症，在ORACLE RAC中，主要通过把SPFILE存放在共享存储上来解决配置丢失的问题。
　　3，脑裂症，在一个共享存储的集群中，当集群中hearbeat丢失时，如果各节点还是同时对共享存储去进行操作，那么在这种情况下所引发的情况是灾难的。ORACLE RAC采用投票算e799bee5baa6e997aee7ad94e58685e5aeb931333361326361法来解决这个问题，思想是这样的：每个节点都有一票，考虑有A，B，C三个节点的集群情形，当A节点由于各种原因不能与B，C节点通信时，那么这集群分成了两个DOMAIN,A节点成为一个DOMAIN，拥有一票；B,C节点成为一个DOMAIN拥有两票，那么这种情况B，C节点拥有对集群的控制权，从而把A节点踢出集群，对要是通IO FENCING来实现。如果是两节点集群，则引入了仲裁磁盘，当两个节点不能通信时，请求最先到达仲裁磁盘的节点拥用对集群的控制权。

主从模式：
主数据库可以进行读写操作，当读写操作导致数据变化时会自动将数据同步给从数据库

* 从数据库一般都是只读的，并且接收主数据库同步过来的数据

* 一个master可以拥有多个slave，但是一个slave只能对应一个master

* slave挂了不影响其他slave的读和master的读和写，重新启动后会将数据从master同步过来

* master挂了以后，不影响slave的读，但redis不再提供写服务，master重启后redis将重新对外提供写服务

* master挂了以后，不会在slave节点中重新选一个master



我们还可以设置同步队列长度。队列长度（backlog)是主redis中的一个缓冲区，在与从redis断开连接期间，主redis会用这个缓冲区来缓存应该发给从redis的数据。这样的话，当从redis重新连接上之后，就不必重新全量同步数据，只需要同步这部分增量数据即可。

复制代码代码如下:

repl-backlog-size 1mb


如果主redis等了一段时间之后，还是无法连接到从redis，那么缓冲队列中的数据将被清理掉。我们可以设置主redis要等待的时间长度。如果设置为0，则表示永远不清理。默认是1个小时。

复制代码代码如下:

repl-backlog-ttl 3600


我们可以给众多的从redis设置优先级，在主redis持续工作不正常的情况，优先级高的从redis将会升级为主redis。而编号越小，优先级越高。比如一个主redis有三个从redis，优先级编号分别为10、100、25，那么编号为10的从redis将会被首先选中升级为主redis。当优先级被设置为0时，这个从redis将永远也不会被选中。默认的优先级为100。

复制代码代码如下:

slave-priority 100












**相关概念
HA再备模式
Oracle 数据
AOF
RDB持久化设计
运算数据库
虚拟主机
虑负载均衡
网络的网关

客户端和服务端交互的内容是序列化后的数据，服务器为每个客户端建立与之对应的连接，
在应用层维护一系列状态保存在connection 中，connection 间相互无关联。在Redis中，
connection 通过redisClient 结构体实现。


***确定更新点
采用更新时间戳、有的采用checkpoint等来标识和记录更新点。
list 和 sorted set
BufferIO I/O 的数据缓存在文件系统的页缓存（ page cache ）中，也就是说，
数据会先被拷贝到操作系统内核的缓冲区中，然后才会从操作系统内核的缓冲区拷贝
到应用程序的地址空间。缓存 I/O 有以下这些优点：
DirectIO 
pagecache其内容对应磁盘上的block。page cache的大小是动态变化的，可以扩大，
也可以在内存不足时缩小。cache缓存的存储设备被称为后备存储（backing store），
注意我们在block I/O一文中提到的：一个page通常包含多个block，这些block不一定是连续的。

：Memcached 本身不具备监听记录是否已
过期的功能，而是通过获取数据记录的时间戳信息，比对当前时间戳后来判定
记录是否过期，这种被动的技术被称之为 lazyexpiration，

内存数据库技术的进一步发展和成熟是在 1984 年

***Redis 为我们提供了四种持久化的方式，分别是：定时快照方式（snapshot）、
基于语句追加文件的方式（aof）、虚拟内存（vm）和 Diskstore。这四种方式根
据设计思路的不同可以分为两类，前两种方式主要是基于小数据量的磁盘落地
功能，即全部数据内存化；后两种方式是基于大数据量的数据存储，不过目前
后两种模式仍在实验阶段，并且 vm 模式已经被作者放弃，所以实际的使用过程
中，目前最常用的仅前两种，下面我们对这两种模式做分别介绍：
定时快照（snapshot）又称之为 RDB，就是按照事先约定好的时间频度，将
增量的 redis 存储数据以快照文件的形式存储在磁盘介质上的方式；AOF 全称是
Appendonly file，直译就是附加文件，实现原理就是 redis 把数据变更指令都追加
在文件记录里，当重启 redis 时，读取文件内的所有操作指令并且按照记录顺序
执行一遍即可。


 
##redis集群选举机制
概要
当redis集群的主节点故障时，Sentinel集群将从剩余的从节点中选举一个新的主节点，有以下步骤：

故障节点主观下线
故障节点客观下线
Sentinel集群选举Leader
Sentinel Leader决定新主节点
选举过程
1、主观下线
Sentinel集群的每一个Sentinel节点会定时对redis集群的所有节点发心跳包检测节点是否正常。如果一个节点在down-after-milliseconds时间内没有回复Sentinel节点的心跳包，则该redis节点被该Sentinel节点主观下线。

2、客观下线
当节点被一个Sentinel节点记为主观下线时，并不意味着该节点肯定故障了，还需要Sentinel集群的其他Sentinel节点共同判断为主观下线才行。

该Sentinel节点会询问其他Sentinel节点，如果Sentinel集群中超过quorum数量的Sentinel节点认为该redis节点主观下线，则该redis客观下线。

如果客观下线的redis节点是从节点或者是Sentinel节点，则操作到此为止，没有后续的操作了；如果客观下线的redis节点为主节点，则开始故障转移，从从节点中选举一个节点升级为主节点。

3、Sentinel集群选举Leader
如果需要从redis集群选举一个节点为主节点，首先需要从Sentinel集群中选举一个Sentinel节点作为Leader。

每一个Sentinel节点都可以成为Leader，当一个Sentinel节点确认redis集群的主节点主观下线后，会请求其他Sentinel节点要求将自己选举为Leader。被请求的Sentinel节点如果没有同意过其他Sentinel节点的选举请求，则同意该请求(选举票数+1)，否则不同意。

如果一个Sentinel节点获得的选举票数达到Leader最低票数(quorum和Sentinel节点数/2+1的最大值)，则该Sentinel节点选举为Leader；否则重新进行选举。

在这里插入图片描述

4、Sentinel Leader决定新主节点
当Sentinel集群选举出Sentinel Leader后，由Sentinel Leader从redis从节点中选择一个redis节点作为主节点：

过滤故障的节点
选择优先级slave-priority最大的从节点作为主节点，如不存在则继续
选择复制偏移量（数据写入量的字节，记录写了多少数据。主服务器会把偏移量同步给从服务器，当主从的偏移量一致，则数据是完全同步）最大的从节点作为主节点，如不存在则继续
选择runid（redis每次启动的时候生成随机的runid作为redis的标识）最小的从节点作为主节点
存在
不存在
存在
不存在
从节点列表
过滤故障节点
slave-priority最大的从节点
升级主节点
复制偏移量最大的从节点
升级主节点
runid最小的从节点升级主节点
为什么Sentinel集群至少3节点
一个Sentinel节选举成为Leader的最低票数为quorum和Sentinel节点数/2+1的最大值，如果Sentinel集群只有2个Sentinel节点，则

Sentinel节点数/2 + 1
= 2/2 + 1
= 2
1
2
3
即Leader最低票数至少为2，当该Sentinel集群中由一个Sentinel节点故障后，仅剩的一个Sentinel节点是永远无法成为Leader。

也可以由此公式可以推导出，Sentinel集群允许1个Sentinel节点故障则需要3个节点的集群；允许2个节点故障则需要5个节点集群。

##返回主页
ericnie的技术博客
信是未曾看见 依然仰望十架 信是完全交托 深知主已掌权
博客园首页新随笔联系订阅管理随笔 - 201  文章 - 0  评论 - 57
理解高可用和灾备
1.高可用 （High Availability，简称 HA）
高可用性是指提供在本地系统单个组件故障情况下，能继续访问应用的能力，无论这个故障是业务流程、物理设施、IT软/硬件的故障。最好的可用性， 就是你的一台机器宕机了，但是使用你的服务的用户完全感觉不到。你的机器宕机了，在该机器上运行的服务肯定得做故障切换（failover），切换有两个维度的成本：RTO （Recovery Time Objective）和 RPO（Recovery Point Objective）。RTO 是服务恢复的时间，最佳的情况是 0，这意味着服务立即恢复；最坏是无穷大意味着服务永远恢复不了；RPO 是切换时向前恢复的数据的时间长度，0 意味着使用同步的数据，大于 0 意味着有数据丢失，比如 ” RPO = 1 天“ 意味着恢复时使用一天前的数据，那么一天之内的数据就丢失了。因此，恢复的最佳结果是 RTO = RPO = 0，但是这个太理想，或者要实现的话成本太高，全球估计 Visa 等少数几个公司能实现，或者几乎实现。

对 HA 来说，往往使用共享存储，这样的话，RPO =0 ；同时往往使用 Active/Active （双活集群） HA 模式来使得 RTO 几乎0，如果使用 Active/Passive 模式的 HA 的话，则需要将 RTO 减少到最小限度。HA 的计算公式是[ 1 - (宕机时间)/（宕机时间 + 运行时间）]，我们常常用几个 9 表示可用性：

2 个9：99% = 1% * 365 = 3.65 * 24 小时/年 = 87.6 小时/年的宕机时间
4 个9: 99.99% = 0.01% * 365 * 24 * 60 = 52.56 分钟/年
5 个9：99.999% = 0.001% * 365 = 5.265 分钟/年的宕机时间，也就意味着每次停机时间在一到两分钟。
11 个 9：几乎就是几年才宕机几分钟。 据说 AWS S3 的设计高可用性就是 11 个 9。
服务的分类

HA 将服务分为两类：

有状态服务：后续对服务的请求依赖于之前对服务的请求。
无状态服务：对服务的请求之间没有依赖关系，是完全独立的。
HA 的种类

HA 需要使用冗余的服务器组成集群来运行负载，包括应用和服务。这种冗余性也可以将 HA 分为两类：

Active/Passive HA：集群只包括两个节点简称主备。在这种配置下，系统采用主和备用机器来提供服务，系统只在主设备上提供服务。在主设备故障时，备设备上的服务被启动来替代主设备提供的服务。典型地，可以采用 CRM 软件比如 Pacemaker 来控制主备设备之间的切换，并提供一个虚机 IP 来提供服务。
Active/Active HA：集群只包括两个节点时简称双活，包括多节点时成为多主（Multi-master）。在这种配置下，系统在集群内所有服务器上运行同样的负载。以数据库为例，对一个实例的更新，会被同步到所有实例上。这种配置下往往采用负载均衡软件比如 HAProxy 来提供服务的虚拟 IP。
2.灾难恢复 （Disaster Recovery）
几个概念：

灾难（Disaster）是由于人为或自然的原因，造成一个数据中心内的信息系统运行严重故障或瘫痪，使信息系统支持的业务功能停顿或服务水平不可接受、达到特定的时间的突发性事件，通常导致信息系统需要切换到备用场地运行。
灾难恢复（Diaster Recovery）是指当灾难破坏生产中心时在不同地点的数据中心内恢复数据、应用或者业务的能力。
容灾是指，除了生产站点以外，用户另外建立的冗余站点，当灾难发生，生产站点受到破坏时，冗余站点可以接管用户正常的业务，达到业务不间断的目的。为了达到更高的可用性，许多用户甚至建立多个冗余站点。 
衡量容灾系统有两个主要指标：RPO（Recovery Point Objective）和 RTO（Recovery Time Object），其中 RPO代表 了当灾难发生时允许丢失的数据量，而 RTO 则代表了系统恢复的时间。RPO 与 RTO 越小，系统的可用性就越高，当然用户需要的投资也越大。
 

大体上讲，容灾可以分为3个级别：数据级别、应用级别以及业务级别。

 

级别    	定义	RTO	CTO
数据级                	
指通过建立异地容灾中心，做数据的远程备份，在灾难发生之后要确保原有的数据不会丢失或者遭到破坏。但在数据级容灾这个级别，发生灾难时应用是会中断的。

在数据级容灾方式下，所建立的异地容灾中心可以简单地把它理解成一个远程的数据备份中心。数据级容灾的恢复时间比较长，但是相比其他容灾级别来讲它的费用比较低，而且构建实施也相对简单。

但是，“数据源是一切关键性业务系统的生命源泉”，因此数据级容灾必不可少。

RTO 最长(若干天) ，因为灾难发生时，需要重新部署机器，利用备份数据恢复业务。                                   	最低
应用级               	在数据级容灾的基础之上，在备份站点同样构建一套相同的应用系统，通过同步或异步复制技术，这样可以保证关键应用在允许的时间范围内恢复运行，尽可能减少灾难带来的损失，让用户基本感受不到灾难的发生，这样就使系统所提供的服务是完整的、可靠的和安全的。	RTO 中等（若干小时）	中等。异地可以搭建一样的系统，或者小些的系统。
业务级	全业务的灾备，除了必要的 IT 相关技术，还要求具备全部的基础设施。其大部分内容是非IT系统（如电话、办公地点等），当大灾难发生后，原有的办公场所都会受到破坏，除了数据和应用的恢复，更需要一个备份的工作场所能够正常的开展业务。	 RTO 最小（若干分钟或者秒）	最高

 

3.HA 和 DR 的关系
两者相互关联，互相补充，互有交叉，同时又有显著的区别：

HA 往往指本地的高可用系统，表示在多个服务器运行一个或多种应用的情况下，应确保任意服务器出现任何故障时，其运行的应用不能中断，应用程序和系统应能迅速切换到其它服务器上运行，即本地系统集群和热备份。HA 往往是用共享存储，因此往往不会有数据丢失（RPO = 0），更多的是切换时间长度考虑即 RTO。
DR 是指异地（同城或者异地）的高可用系统，表示在灾害发生时，数据、应用以及业务的恢复能力。异地灾备的数据灾备部分是使用数据复制，根据使用的不同数据复制技术（同步、异步、Strectched Cluster 等），数据往往有损失导致 RPO >0；而异地的应用切换往往需要更长的时间，这样 RT0 >0。 因此，需要结合特定的业务需求，来定制所需要的 RTO 和 RPO，以实现最优的 CTO。
也可以从别的角度上看待两者的区别： 

 

从故障角度，HA 主要处理单组件的故障导致负载在集群内的服务器之间的切换，DR 则是应对大规模的故障导致负载在数据中心之间做切换。
从网络角度，LAN 尺度的任务是 HA 的范畴，WAN 尺度的任务是 DR 的范围。
从云的角度看，HA 是一个云环境内保障业务持续性的机制，DR 是多个云环境间保障业务持续性的机制。
从目标角度，HA 主要是保证业务高可用，DR 是保证数据可靠的基础上的业务可用。  
一个异地容灾系统，往往包括本地的 HA 集群和异地的 DR 数据中心。一个示例如下：


 

 

aster SQL Server 发生故障时，切换到 Standby SQL Server，继续提供数据库服务：

在主机房中心发生灾难时，切换到备份机房（总公司机房中心）上，恢复应用和服务：


4.现有系统的灾备架构
现有一个容器部署的应用，完整的应用灾备架构如图：

 



正常状况下，大家都联同一个区域的数据库



 

区域1应用当机情况：

 



 

应用及数据库当机情况：

 



 

现有架构问题：

应用需要在两个区域分别发版本
在Kubernetes架构下，容器无法停止，只能基于helm进行删除，这导致再次切回来时需要重新发版本，存在风险
持久化数据通过nas进行不同区域间的拷贝，在非计划当机中，需要规划备份的时间
数据库和Redis域名同样基于域名切换，而不是VIP切换，易于出错
 

优化方案：

尝试Kubernetes Federation,解决在部署多地时应用多次发版本问题
Zone1区域的应用无法暂停运行，这将造成大量的无用的链接数据库的日志产生，如果希望切换期间应用停止，建议如下操作：
基于helm del releasename删除，这样会删除实例，但是会保留helm的版本信息
通过helm rollback进行恢复到原来版本。
#helm del helm1-testhelm

#helm rollback helm1-testhelm 1
 




##返回主页
leffss
博客园首页新随笔联系管理订阅订阅随笔- 57  文章- 0  评论- 4 
Redis 单机模式，主从模式，哨兵模式(sentinel)，集群模式(cluster)，第三方模式优缺点分析
Redis 的几种常见使用方式包括：

单机模式
主从模式
哨兵模式(sentinel)
集群模式(cluster)
第三方模式
单机模式
Redis 单副本，采用单个 Redis 节点部署架构，没有备用节点实时同步数据，不提供数据持久化和备份策略，适用于数据可靠性要求不高的纯缓存业务场景。

优点：

架构简单，部署方便。
高性价比：缓存使用时无需备用节点(单实例可用性可以用 supervisor 或 crontab 保证)，当然为了满足业务的高可用性，也可以牺牲一个备用节点，但同时刻只有一个实例对外提供服务。
高性能。
缺点：

不保证数据的可靠性。
在缓存使用，进程重启后，数据丢失，即使有备用的节点解决高可用性，但是仍然不能解决缓存预热问题，因此不适用于数据可靠性要求高的业务。
高性能受限于单核 CPU 的处理能力(Redis 是单线程机制)，CPU 为主要瓶颈，所以适合操作命令简单，排序、计算较少的场景。也可以考虑用 Memcached 替代。
主从模式
Redis 采用主从(可以多从)部署结构，相较于单副本而言最大的特点就是主从实例间数据实时同步，并且提供数据持久化和备份策略。主从实例部署在不同的物理服务器上，根据公司的基础环境配置，可以实现同时对外提供服务和读写分离策略。

优点：

高可靠性：一方面，采用双机主备架构，能够在主库出现故障时自动进行主备切换，从库提升为主库提供服务，保证服务平稳运行;另一方面，开启数据持久化功能和配置合理的备份策略，能有效的解决数据误操作和数据异常丢失的问题。
读写分离策略：从节点可以扩展主库节点的读能力，有效应对大并发量的读操作。
缺点：

故障恢复复杂，如果没有 RedisHA 系统(需要开发)，当主库节点出现故障时，需要手动将一个从节点晋升为主节点，同时需要通知业务方变更配置，并且需要让其它从库节点去复制新主库节点，整个过程需要人为干预，比较繁琐。
主库的写能力受到单机的限制，可以考虑分片。
主库的存储能力受到单机的限制，可以考虑 Pika。
原生复制的弊端在早期的版本中也会比较突出，如：Redis 复制中断后，Slave 会发起 psync，此时如果同步不成功，则会进行全量同步，主库执行全量备份的同时可能会造成毫秒或秒级的卡顿;又由于 COW 机制，导致极端情况下的主库内存溢出，程序异常退出或宕机;主库节点生成备份文件导致服务器磁盘 IO 和 CPU(压缩)资源消耗;发送数 GB 大小的备份文件导致服务器出口带宽暴增，阻塞请求，建议升级到最新版本。
哨兵模式
Redis Sentinel 是 2.8 版本后推出的原生高可用解决方案，其部署架构主要包括两部分：Redis Sentinel 集群和 Redis 数据集群。其中 Redis Sentinel 集群是由若干 Sentinel 节点组成的分布式集群，可以实现故障发现、故障自动转移、配置中心和客户端通知。Redis Sentinel 的节点数量要满足 2n+1(n>=1)的奇数个。

优点：

Redis Sentinel 集群部署简单。
能够解决 Redis 主从模式下的高可用切换问题。
很方便实现 Redis 数据节点的线形扩展，轻松突破 Redis 自身单线程瓶颈，可极大满足 Redis 大容量或高性能的业务需求。
可以实现一套 Sentinel 监控一组 Redis 数据节点或多组数据节点。
缺点：

部署相对 Redis 主从模式要复杂一些，原理理解更繁琐。
资源浪费，Redis 数据节点中 slave 节点作为备份节点不提供服务。
Redis Sentinel 主要是针对 Redis 数据节点中的主节点的高可用切换，对 Redis 的数据节点做失败判定分为主观下线和客观下线两种，对于 Redis 的从节点有对节点做主观下线操作，并不执行故障转移。
不能解决读写分离问题，实现起来相对复杂。
集群模式
Redis Cluster 是 3.0 版后推出的 Redis 分布式集群解决方案，主要解决 Redis 分布式方面的需求，比如，当遇到单机内存，并发和流量等瓶颈的时候，Redis Cluster 能起到很好的负载均衡的目的。Redis Cluster 集群节点最小配置 6 个节点以上(3 主 3 从)，其中主节点提供读写操作，从节点作为备用节点，不提供请求，只作为故障转移使用。Redis Cluster 采用虚拟槽分区，所有的键根据哈希函数映射到 0～16383 个整数槽内，每个节点负责维护一部分槽以及槽所印映射的键值数据。

优点：

无中心架构。
数据按照 slot 存储分布在多个节点，节点间数据共享，可动态调整数据分布。
可扩展性：可线性扩展到 1000 多个节点，节点可动态添加或删除。
高可用性：部分节点不可用时，集群仍可用。通过增加 Slave 做 standby 数据副本，能够实现故障自动 failover，节点之间通过 gossip 协议交换状态信息，用投票机制完成 Slave 到 Master 的角色提升。
降低运维成本，提高系统的扩展性和可用性。
缺点：

Client 实现复杂，驱动要求实现 Smart Client，缓存 slots mapping 信息并及时更新，提高了开发难度，客户端的不成熟影响业务的稳定性。目前仅 JedisCluster 相对成熟，异常处理部分还不完善，比如常见的“max redirect exception”。
节点会因为某些原因发生阻塞(阻塞时间大于 clutser-node-timeout)，被判断下线，这种 failover 是没有必要的。
数据通过异步复制，不保证数据的强一致性。
多个业务使用同一套集群时，无法根据统计区分冷热数据，资源隔离性较差，容易出现相互影响的情况。
Slave 在集群中充当“冷备”，不能缓解读压力，当然可以通过 SDK 的合理设计来提高 Slave 资源的利用率。
Key 批量操作限制，如使用 mset、mget 目前只支持具有相同 slot 值的 Key 执行批量操作。对于映射为不同 slot 值的 Key 由于 Keys 不支持跨 slot 查询，所以执行 mset、mget、sunion 等操作支持不友好。
Key 事务操作支持有限，只支持多 key 在同一节点上的事务操作，当多个 Key 分布于不同的节点上时无法使用事务功能。
Key 作为数据分区的最小粒度，不能将一个很大的键值对象如 hash、list 等映射到不同的节点。
不支持多数据库空间，单机下的 redis 可以支持到 16 个数据库，集群模式下只能使用 1 个数据库空间，即 db 0。
复制结构只支持一层，从节点只能复制主节点，不支持嵌套树状复制结构。
避免产生 hot-key，导致主库节点成为系统的短板。
避免产生 big-key，导致网卡撑爆、慢查询等。
重试时间应该大于 cluster-node-time 时间。
Redis Cluster 不建议使用 pipeline 和 multi-keys 操作，减少 max redirect 产生的场景。
优点多，缺点也多啊，事物都有双面性。



##断点，拆分

来源于《深入分布式缓存：从原理到实践》第八章 分布式Redis的读书笔记。

数据存储系统的挑战和Redis的应对策略
Redis作为数据存储系统，无论数据存储在内存中还是持久化到本地，作为单实例节点，在实际应用中总会面临如下挑战：

数据量伸缩：单实例Redis存储的key-value对的数量受限于单机的内存和磁盘容量。长期运行的生产环境中，随着数据不断地加入，存储容量会达到瓶颈。虽然Redis提供了key的过期机制，在作为缓存使用时通过淘汰过期的数据可以达到控制容量的目的。但当Redis作为NoSQL数据库时，业务数据长期有效使得淘汰机制不再适用。
访问量伸缩：单实例Redis单线程地运行，吞吐量受限于单次请求处理的平均时耗。当业务数据集面临超过单实例处理能力的高吞吐量需求时，如何提升处理能力成为难点。
单点故障。Redis持久化机制一定程度上缓解了宕机/重启带来的业务数据丢失问题，但当单实例所在的物理节点发生不可恢复故障时，如何保证业务数据不丢以及如何在故障期间迅速地恢复对应业务数据的可用性也成为单点结构的挑战。
image.png
image.png
上述问题对于数据存储系统而言是通用的，基于分布式的解决方案如下：

水平拆分：分布式环境下，节点分为不同的分组，每个分组处理业务数据的一个子集，分组之间的数据无交集。数据无交集的特性使得水平拆分解决了数据量瓶颈，随着分组的增加，单个分组承载的数据子集更小，即通过增加分组来伸缩数据量。同时水平拆分也也解决了访问量瓶颈，业务数据全集的请求被分摊到了不同分组，随着分组数的增加，数据全集的总吞吐量也增加，访问量的伸缩性得以实现。
主备复制：同一份业务数据存在多个副本，对数据的每次访问根据一定规则分发到某一个或多个副本上执行。通过W+R>N的读写配置可以做到读取数据内容的实时性。随着N的增加，当读写访问量差不多时，业务的吞吐量相比单实例会提升到逼近2倍。但实际中，读的访问量常常远高于写的量，W=N, R=1，吞度量会随着读写比例的增加而提升。
故障转移：当业务数据所在的节点故障时，这部分业务数据转移到其他节点上进行，使得故障节点在恢复期间，对应的业务数据仍然可用。显然，为了支撑故障转移，业务数据需要保持多个副本，位于不同的节点上。
水平拆分（sharding）
水平拆分（sharding）为了解决数据量和访问量增加后对单节点造成的性能压力，通常引入水平拆分机制，将数据存储和对数据的访问分散到不同节点上分别处理。水平拆分后的每个节点存储和处理的数据原则上没有交集，使得节点间相互独立；但内部的拆分和多节点通常对外部服务透明，通过数据分布和路由请求的配合，可以做到数据存放和数据访问对水平拆分的适配。

数据分布分布式环境下，有多个Redis实例I[i] (i=0, …, N)，同时业务数据的key全集为{k[0], k[1], …, k[M]}。数据分布指的是一种映射关系f，每个业务数据key都能通过这种映射确定唯一的实例I，即f(k)=i，其中k对应的业务数据存放于I[i]。

如何确定这个映射关系的算法？这其实主要取决于Redis的客户端。常用的是映射有3种：

image.png
❑ hash映射。为了解决业务数据key的值域不确定这个问题，引入hash运算，将不可控的业务值域key映射到可控的有限值域（hash值）上，且映射做到均匀，再将有限的均匀分布的hash值枚举地映射到Redis实例上。例如，crc16(key)%16384这个hash算法，将业务key映射到了0～16383这一万多个确定的有限整数集合上，再依据一定规则将这个整数集合的不同子集不相交地划分到不同Redis实例上，以此实现数据分布。

❑ 范围映射。和hash映射不同，范围映射通常选择key本身而不是key的某个函数运算值（如hash运算）作为数据分布的条件，且每个数据节点存放的key的值域是连续的一段范围。例如，当0≤key＜100时，数据存放到实例1上；当100≤key＜200时，数据存放到实例2上；以此类推。key的值域是业务层决定的，业务层需要清楚每个区间的范围和Redis实例数量，才能完整地描述数据分布。这使得业务域的信息（key的值域）和系统域的信息（实例数量）耦合，数据分布无法在纯系统层面实现，从系统层面看来，业务域的key值域不确定、不可控。

❑ hash和范围结合。典型的方式就是一致性hash，首先对key进行hash运算，得到值域有限的hash值，再对hash值做范围映射，确定该key对应的业务数据存放的具体实例。这种方式的优势是节点新增或退出时，涉及的数据迁移量小——变更的节点上涉及的数据只需和相邻节点发生迁移关系；缺点是节点不是成倍变更（数量变成原有的N倍或1/N）时，会造成数据分布的不均匀。

主备复制（replication）
前述水平拆分讨论如何将数据划分到没有交集的各个数据节点上，即，不同节点间没有相同的数据。而在有的场景下，我们需要将相同的数据存放在多个不同的节点上。例如，当某个节点宕机时，其上的数据在其他节点上有副本，使得该数据对外服务可以继续进行，即，数据复制为后续所述的故障转移提供了基础。

再如，同一份数据在多个节点上存储后，写入节点可以和读取节点分离，提升性能。当一份数据落在了多个不同节点上时，如何保证节点间数据的一致性将是关键问题，在不同的存储系统架构下方案不同，有的采用客户端双写，有的采用存储层复制。Redis采用主备复制的方式保证一致性，即所有节点中，有一个节点为主节点（master）它对外提供写入服务，所有的数据变更由外界对master的写入触发，之后Redis内部异步地将数据从主节点复制到其他节点（slave）上。

主备复制流程
Redis包含master和slave两种节点：master节点对外提供读写服务；slave节点作为master的数据备份，拥有master的全量数据，对外不提供写服务。
主备复制由slave主动触发，主要流程如图所示。

image.png
1）首先slave向master发起SYNC命令。这一步在slave启动后触发，master被动地将新进slave节点加入自己的主备复制集群。
2）master收到SYNC后，开启BGSAVE操作。BGSAVE是Redis的一种全量模式的持久化机制。
3）BGSAVE完成后，master将快照信息发送给slave。
4）发送期间，master收到的来自用户客户端的新的写命令，除了正常地响应之外，都再存入一份到backlog队列。
5）快照信息发送完成后，master继续发送backlog队列信息。
6）backlog发送完成后，后续的写操作同时发给slave，保持实时地异步复制。

在上图的slave侧，处理逻辑如下：
❑ 发送SYNC；
❑ 开始接收master的快照信息，此时，将slave现有数据清空，并将master快照写入自身内存；
❑ 接收backlog内容并执行它，即回放；
❑ 继续接收后续来自master的命令副本并继续回放，以保持数据和master一致。

注意：在slave没有收到master快照文件之前，会根据配置决定使用现有的数据响应客户端还是直接拒绝。

如果有多个slave节点并发发送SYNC命令给master，企图建立主备关系，只要第二个slave的SYNC命令发生在master完成BGSAVE之前，第二个slave将收到和第一个slave相同的快照和后续backlog；否则，第二个slave的SYNC将触发master的第二次BGSAVE。

断点续传
每次当slave通过SYNC和master同步数据时，master都会dump全量数据并发发送。当一个已经和master完成了同步并持续保持了长时间的slave网络断开很短的时间再重新连上时，master不得不重新做一遍全量dump的传送。然而由于slave只断开了很短时间，重连之后master-slave的差异数据很少，全量dump的数据中绝大部分，slave都已经具有，再次发送这些数据会导致大量无效的开销。最好的方式是，master-slave只同步断开期间的少量数据。

Redis的PSYNC（Partial Sync）可以用于替代SYNC，做到master-slave基于断点续传的主备同步协议。master-slave两端通过维护一个offset记录当前已经同步过的命令，slave断开期间，master的客户端命令会保持在缓存中，在slave重连之后，告知master断开时的最新offset, master则将缓存中大于offset的数据发送给slave，而断开前已经同步过的数据，则不再重新同步，这样减少了数据传输开销。

故障转移（failover）
当两台以上Redis实例形成了主备关系，它们组成的集群就具备了一定的高可用性：当master故障时，slave可以成为新的master，对外提供读写服务，这种运营机制称为failover。剩下的问题在于：谁去发现master的故障做failover的决策？

一种方式是，保持一个daemon进程，监控着所有的master-slave节点，如图所示：

image.png
一个Redis集群里有一个master和两个slave，这个daemon进程监视着这三个节点。这种方式的问题在于：daemon作为单点，它本身的可用性无法保证。因此需要引入多daemon，如图所示：

image.png
在图中，为了解决一个daemon的单点问题，我们引入了两个daemon进程，同时监视着三个Redis节点。但是，多个daemon的引入虽然解决了可用性问题，但带来了一致性问题：多个daemon之间，如何就某个master是否可用达成一致？比如，daemon1和master之间的网络不通，但master和其余节点均畅通，那么daemon1和daemon2观察到的master可用状态不同，那么如何决策此时master是否需要failover？

Redis的sentinel提供了一套多daemon间的交互机制，解决故障发现、failover决策协商机制等问题，如图所示。

image.png
多个daemon组成了一个集群，称为sentinel集群，其中的daemon也被称为sentinel节点。这些节点相互间通信、选举、协商，在master节点的故障发现、failover决策上表现出一致性。

sentinel间的相互感知
sentinel节点间因为共同监视了同一个master节点从而相互也关联了起来，一个新加入的sentinel节点需要和有相同监视的master的其他sentinel节点相互感知，方式如下：所有需要相互感知的sentinel都向他们共同的master节点上订阅相同的channel:sentinel:hello，新加入的sentinel节点向这个channel发布一条消息，包含了自己信息，该channel的订阅者们就可以发现这个新的sentinel。随后新sentinel和已有的其他sentinel节点建立长连接。sentinel集群中所有节点两两连接，如图所示：

image.png
新的sentinel节点加入后，它向master节点发布自己加入这个信息，此时现有的订阅sentinel节点将会发现这条消息从而感知到了新sentinel节点的存在。

master的故障发现
sentinel节点通过定期地向master发送心跳包判断其存活状态，称为PING。一旦发现master没有正确地响应，sentinel将此master置为“主观不可用态”，所谓主观，是因为“master不可用”这个判定尚未得到其他sentinel节点的确认。如图所示：

image.png
随后它将“主观不可用态”发送给其他所有的sentinel节点进行确认（即图"is-master-down-by-addr"这条交互），当确认的sentinel节点数>=quorum（可配置）时，则判定为该master为“客观不可用”，随后进入failover流程。

failover决策
当一台master真正宕机后，可能多个sentinel节点同时发现了此问题并通过交互确认了相互的“主观不可用态”猜想，同时达到“客观不可用态”，同时打算发起failover。但是最终只能有一个sentinel节点作为failover的发起者，此时需要开始一个leader选举的过程，选择谁来发起failover。

Redis的sentinel机制采用类似Raft协议实现这个选举算法：

sentinelState的epoch变量类似于raft协议中的term（选举回合）；
每一个确认了master“客观不可用态”的sentinel节点都会向周围广自己参选的请求；
每一个接收到参选请求的sentinel节点如果还没人向它发送过参选请求，它就将本选举回合的意向置为首个参选sentinel并回复它；如果已经在本回合表过意向了，则拒绝本回合内所有其他的参选请求，并将已有意向回复给参选sentinel；
每一个发送参选请求的sentinel节点如果收到了超过一半的意向同意某个参选sentinel（也可能是自己），则确定该sentinel为leader；如果本回合持续了足够长的时间还未选出leader，则开启下一个回合。leader sentinel确定之后，从master所有的slave中依据一定的规则选取一个作为新的master，告知其他slave连接这个新的master。
  


##Redis的ACID
事务是一个数据库必备的元素，对于redis也不例外，对于一个传统的关系型数据库来说，数据库事务满足ACID四个特性：

A代表原子性：一个事务（transaction）中的所有操作，要么全部完成，要么全部不完成，不会结束在中间某个环节。事务在执行过程中发生错误，会被回滚（Rollback）到事务开始前的状态，就像这个事务从来没有执行过一样。
C代表一致性：事务应确保数据库的状态从一个一致状态转变为另一个一致状态。一致状态的含义是数据库中的数据应满足完整性约束
I代表隔离性：多个事务并发执行时，一个事务的执行不应影响其他事务的执行
D代表持久性：已被提交的事务对数据库的修改应该永久保存在数据库中
然而，对于redis来说，只满足其中的：

一致性和隔离性两个特性，其他特性是不支持的。

关于redis对ACID四个特性暂时先说这么多，在本文后面会详细说明。在详述之前，我们先来了解redis事务具体的使用和实现，这样我们接下来讨论ACID时才能更好的理解。

redis事务的使用
redis事务主要包括MULTI、EXEC、DISCARD和WATCH命令

MULTI命令
MULTI命令用来开启一个事务，当MULTI执行之后，客户端可以继续向服务器发送多条命令，这些命令会缓存在队列里面，只有当执行EXEC命令时，这些命令才会执行。

而DISCARD命令可以清空事务队列，放弃执行事务。

一个使用MULTI和EXEC执行事务的例子：


redis> MULTI
OK
 
redis> SET book-name "Mastering C++ in 21 days"
QUEUED
 
redis> GET book-name
QUEUED
 
redis> SADD tag "C++" "Programming" "Mastering Series"
QUEUED
 
redis> SMEMBERS tag
QUEUED
 
redis> EXEC
1) OK
2) "Mastering C++ in 21 days"
3) (integer) 3
4) 1) "Mastering Series"
   2) "C++"
   3) "Programming"
一个事务从开始到执行会经历以下三个阶段：

开始事务。
命令入队。
执行事务。
redis事务产生错误
使用redis事务可能会产生错误，主要分为两大类：

事务在执行EXEC之前，入队的命令可能出错。
命令可能在 EXEC 调用之后失败
redis在2.6.5之后，如果发现事务在执行EXEC之前出现错误，那么会放弃这个事务。

在EXEC命令之后产生的错误，会被忽略，其他正确的命令会被继续执行。

redis事务不支持回滚
如果你有使用关系式数据库的经验， 那么 “Redis 在事务失败时不进行回滚，而是继续执行余下的命令”这种做法可能会让你觉得有点奇怪。

以下是这种做法的优点：

Redis 命令只会因为错误的语法而失败（并且这些问题不能在入队时发现），或是命令用在了错误类型的键上面：这也就是说，从实用性的角度来说，失败的命令是由编程错误造成的，而这些错误应该在开发的过程中被发现，而不应该出现在生产环境中。
因为不需要对回滚进行支持，所以 Redis 的内部可以保持简单且快速。 有种观点认为 Redis 处理事务的做法会产生 bug ， 然而需要注意的是， 在通常情况下， 回滚并不能解决编程错误带来的问题。 举个例子， 如果你本来想通过 INCR 命令将键的值加上 1 ， 却不小心加上了 2 ， 又或者对错误类型的键执行了 INCR ， 回滚是没有办法处理这些情况的。
鉴于没有任何机制能避免程序员自己造成的错误， 并且这类错误通常不会在生产环境中出现， 所以 Redis 选择了更简单、更快速的无回滚方式来处理事务。

关于WATCH命令
WATCH命令可以添加监控的键，如果这些监控的键没有被其他客户端修改，那么事务可以顺利执行，如果被修改了，那么事务就不能执行。

1
2
3
4
5
6
7
8
9
10
11
redis> WATCH name
OK
 
redis> MULTI
OK
 
redis> SET name peter
QUEUED
 
redis> EXEC
(nil)
以下执行序列展示了上面的例子是如何失败的：

时间	客户端 A	客户端 B
T1	WATCH name	 
T2	MULTI	 
T3	SET name peter	 
T4	 	SET name john
T5	EXEC	 
在时间 T4 ，客户端 B 修改了 name 键的值， 当客户端 A 在 T5 执行 EXEC 时，Redis 会发现 name 这个被监视的键已经被修改， 因此客户端 A 的事务不会被执行，而是直接返回失败

当客户端发送 EXEC 命令、触发事务执行时， 服务器会对客户端的状态进行检查：

如果客户端的 REDIS_DIRTY_CAS 选项已经被打开，那么说明被客户端监视的键至少有一个已经被修改了，事务的安全性已经被破坏。服务器会放弃执行这个事务，直接向客户端返回空回复，表示事务执行失败。
如果 REDIS_DIRTY_CAS 选项没有被打开，那么说明所有监视键都安全，服务器正式执行事务。
这里是伪代码


def check_safety_before_execute_trasaction():
 
    if client.state & REDIS_DIRTY_CAS:
        # 安全性已破坏，清除事务状态
        clear_transaction_state(client)
        # 清空事务队列
        clear_transaction_queue(client)
        # 返回空回复给客户端
        send_empty_reply(client)
    else:
        # 安全性完好，执行事务
        execute_transaction()
 

redis事务的实现
在了解了redis事务的使用之后，我们再来看看redis事务的实现，主要是对上面说的MULTI、EXEC、DISCARD和WATCH命令的源码的分析。

MULTI命令实现
1
2
3
4
5
6
7
8
void multiCommand(redisClient *c) {
    if (c->flags & REDIS_MULTI) {
        addReplyError(c,"MULTI calls can not be nested");
        return;
    }
    c->flags |= REDIS_MULTI;
    addReply(c,shared.ok);
}
从上面的源码可以看出，MULTI只能执行一次，而且就做一件事，把客户端的标志打上REDIS_MULTI。

EXEC命令实现

void execCommand(redisClient *c) {
    int j;
    robj **orig_argv;
    int orig_argc;
    struct redisCommand *orig_cmd;
 
    if (!(c->flags & REDIS_MULTI)) {
        addReplyError(c,"EXEC without MULTI");
        return;
    }
 
    /* Check if we need to abort the EXEC if some WATCHed key was touched.
    * A failed EXEC will return a multi bulk nil object. */
    if (c->flags & REDIS_DIRTY_CAS) {
        freeClientMultiState(c);
        initClientMultiState(c);
        c->flags &= ~(REDIS_MULTI|REDIS_DIRTY_CAS);
        unwatchAllKeys(c);
        addReply(c,shared.nullmultibulk);
        goto handle_monitor;
    }
 
    /* Replicate a MULTI request now that we are sure the block is executed.
    * This way we'll deliver the MULTI/..../EXEC block as a whole and
    * both the AOF and the replication link will have the same consistency
    * and atomicity guarantees. */
    execCommandReplicateMulti(c);
 
    /* Exec all the queued commands */
    unwatchAllKeys(c); /* Unwatch ASAP otherwise we'll waste CPU cycles */
    orig_argv = c->argv;
    orig_argc = c->argc;
    orig_cmd = c->cmd;
    addReplyMultiBulkLen(c,c->mstate.count);
    for (j = 0; j < c->mstate.count; j++) {
        c->argc = c->mstate.commands[j].argc;
        c->argv = c->mstate.commands[j].argv;
        c->cmd = c->mstate.commands[j].cmd;
        call(c,REDIS_CALL_FULL);
 
        /* Commands may alter argc/argv, restore mstate. */
        c->mstate.commands[j].argc = c->argc;
        c->mstate.commands[j].argv = c->argv;
        c->mstate.commands[j].cmd = c->cmd;
    }
    c->argv = orig_argv;
    c->argc = orig_argc;
    c->cmd = orig_cmd;
    freeClientMultiState(c);
    initClientMultiState(c);
    c->flags &= ~(REDIS_MULTI|REDIS_DIRTY_CAS);
    /* Make sure the EXEC command is always replicated / AOF, since we
    * always send the MULTI command (we can't know beforehand if the
    * next operations will contain at least a modification to the DB). */
    server.dirty++;
 
handle_monitor:
    /* Send EXEC to clients waiting data from MONITOR. We do it here
    * since the natural order of commands execution is actually:
    * MUTLI, EXEC, ... commands inside transaction ...
    * Instead EXEC is flagged as REDIS_CMD_SKIP_MONITOR in the command
    * table, and we do it here with correct ordering. */
    if (listLength(server.monitors) && !server.loading)
        replicationFeedMonitors(c,server.monitors,c->db->id,c->argv,c->argc);
}
其主要步骤是：

检查是否已经是处在MULTI状态下，如果不是，直接返回
检查WATCH的key是否已经被其他客户端修改，如果是，放弃事务
执行事务命令队列里面的所有命令
执行命令队列里面的所有命令的代码如下：


for (j = 0; j < c->mstate.count; j++) {
    c->argc = c->mstate.commands[j].argc;
    c->argv = c->mstate.commands[j].argv;
    c->cmd = c->mstate.commands[j].cmd;
    call(c,REDIS_CALL_FULL);
 
     /* Commands may alter argc/argv, restore mstate. */
    c->mstate.commands[j].argc = c->argc;
    c->mstate.commands[j].argv = c->argv;
    c->mstate.commands[j].cmd = c->cmd;
}
遍历queue队列的所有命令，一条条的执行

DISCARD命令实现
DICARD命令主要由下面两个函数完成：


void discardCommand(redisClient *c) {
    if (!(c->flags & REDIS_MULTI)) {
        addReplyError(c,"DISCARD without MULTI");
        return;
    }
    discardTransaction(c);
    addReply(c,shared.ok);
}
 
void discardTransaction(redisClient *c) {
    freeClientMultiState(c);
    initClientMultiState(c);
    c->flags &= ~(REDIS_MULTI|REDIS_DIRTY_CAS);;
    unwatchAllKeys(c);
}
 
void freeClientMultiState(redisClient *c) {
    int j;
 
    for (j = 0; j < c->mstate.count; j++) {
        int i;
        multiCmd *mc = c->mstate.commands+j;
 
        for (i = 0; i < mc->argc; i++)
            decrRefCount(mc->argv[i]);
        zfree(mc->argv);
    }
    zfree(c->mstate.commands);
}
函数调用的关系如下

  discardCommand
       |
       |
       |
discardTransaction
       |
       |
       |
freeClientMultiState
freeClientMultiState就是完成DISCARD命令主要功能的函数，把其命令队列里面的所有命令所在的存储空间都释放。

WATCH命令

void watchForKey(redisClient *c, robj *key) {
    list *clients = NULL;
    listIter li;
    listNode *ln;
    watchedKey *wk;
 
    /* Check if we are already watching for this key */
    listRewind(c->watched_keys,&li);
    while((ln = listNext(&li))) {
        wk = listNodeValue(ln);
        if (wk->db == c->db && equalStringObjects(key,wk->key))
            return; /* Key already watched */
    }
    /* This key is not already watched in this DB. Let's add it */
    clients = dictFetchValue(c->db->watched_keys,key);
    if (!clients) {
        clients = listCreate();
        dictAdd(c->db->watched_keys,key,clients);
        incrRefCount(key);
    }
    listAddNodeTail(clients,c);
    /* Add the new key to the lits of keys watched by this client */
    wk = zmalloc(sizeof(*wk));
    wk->key = key;
    wk->db = c->db;
    incrRefCount(key);
    listAddNodeTail(c->watched_keys,wk);
}
这个函数是在WATCH命令执行后，主要完成两件事情：

把key添加到当前客户端c->watched_keys的链表中，
并且把当前客户端添加到c->db->watched_keys的字典里面去。
当db中有能修改键状态的命令（比如INCR等）执行时，会自动把watch这个key的客户端的事务设置为放弃状态：

void touchWatchedKey(redisDb *db, robj *key) {
    list *clients;
    listIter li;
    listNode *ln;
 
    if (dictSize(db->watched_keys) == 0) return;
    clients = dictFetchValue(db->watched_keys, key);
    if (!clients) return;
 
    /* Mark all the clients watching this key as REDIS_DIRTY_CAS */
    /* Check if we are already watching for this key */
    listRewind(clients,&li);
    while((ln = listNext(&li))) {
        redisClient *c = listNodeValue(ln);
 
        c->flags |= REDIS_DIRTY_CAS;
    }
}
如上源码所示，当某个key被修改后，会遭到其字典中所在的hash链表，然后把链表中所有的客户端都设置成REDIS_DIRTY_CAS状态，即事务被DISCARD的状态。

redis事务的ACID性质讨论
A原子性
单个 Redis 命令的执行是原子性的，但 Redis 没有在事务上增加任何维持原子性的机制，所以 Redis 事务的执行并不是原子性的

如果一个事务队列中的所有命令都被成功地执行，那么称这个事务执行成功

另一方面，如果 Redis 服务器进程在执行事务的过程中被停止 —— 比如接到 KILL 信号、宿主机器停机，等等，那么事务执行失败

事务失败时，Redis 也不会进行任何的重试或者回滚动作，不满足要么全部全部执行，要么都不执行的条件

C一致性
一致性分下面几种情况来讨论：

首先，如果一个事务的指令全部被执行，那么数据库的状态是满足数据库完整性约束的

其次，如果一个事务中有的指令有错误，那么数据库的状态是满足数据完整性约束的

最后，如果事务运行到某条指令时，进程被kill掉了，那么要分下面几种情况讨论：

如果当前redis采用的是内存模式，那么重启之后redis数据库是空的，那么满足一致性条件
如果当前采用RDB模式存储的，在执行事务时，Redis 不会中断事务去执行保存 RDB 的工作，只有在事务执行之后，保存 RDB 的工作才有可能开始。所以当 RDB 模式下的 Redis 服务器进程在事 务中途被杀死时，事务内执行的命令，不管成功了多少，都不会被保存到 RDB 文件里。 恢复数据库需要使用现有的 RDB 文件，而这个 RDB 文件的数据保存的是最近一次的数 据库快照（snapshot），所以它的数据可能不是最新的，但只要 RDB 文件本身没有因为 其他问题而出错，那么还原后的数据库就是一致的

如果当前采用的是AOF存储的，那么可能事务的内容还未写入到AOF文件，那么此时肯定是满足一致性的，如果事务的内容有部分写入到AOF文件中，那么需要用工具把AOF中事务执行部分成功的指令移除，这时，移除之后的AOF文件也是满足一致性的

所以，redis事务满足一致性约束

I隔离性
Redis 是单进程程序，并且它保证在执行事务时，不会对事务进行中断，事务可以运行直到执行完所有事务队列中的命令为止。因此，Redis 的事务是总是带有隔离性的。

D持久性
因为事务不过是用队列包裹起了一组 Redis 命令，并没有提供任何额外的持久性功能，所以事务的持久性由 Redis 所使用的持久化模式决定

在单纯的内存模式下，事务肯定是不持久的
在 RDB 模式下，服务器可能在事务执行之后、RDB 文件更新之前的这段时间失败，所以 RDB 模式下的 Redis 事务也是不持久的
在 AOF 的“总是 SYNC ”模式下，事务的每条命令在执行成功之后，都会立即调用 fsync 或 fdatasync 将事务数据写入到 AOF 文件。但是，这种保存是由后台线程进行的，主线程不会阻塞直到保存成功，所以从命令执行成功到数据保存到硬盘之间，还是有一段非常小的间隔，所以这种模式下的事务也是不持久的。
其他 AOF 模式也和“总是 SYNC ”模式类似，所以它们都是不持久的。


##Redis之各版本特性
1.Redis2.6

Redis2.6在2012年正是发布，经历了17个版本，到2.6.17版本，相对于Redis2.4，主要特性如下：

1）服务端支持Lua脚本。

2）去掉虚拟内存相关功能。

3）放开对客户端连接数的硬编码限制。

4）键的过期时间支持毫秒。

5）从节点支持只读功能。

6）两个新的位图命令：bitcount和bitop。

7）增强了redis-benchmark的功能：支持定制化的压测，CSV输出等功能。

8）基于浮点数自增命令：incrbyfloat和hincrbyfloat。

9）redis-cli可以使用--eval参数实现Lua脚本执行。

10）shutdown命令增强。

11）重构了大量的核心代码，所有集群相关的代码都去掉了，cluster功能将会是3.0版本最大的亮点。

12）info可以按照section输出，并且添加了一些统计项

13）sort命令优化

 

 

2.Redis2.8

Redis2.8在2013年11月22日正式发布，经历了24个版本，到2.8.24版本，相比于Redis2.6，主要特性如下：

1）添加部分主从复制的功能，在一定程度上降低了由于网络问题，造成频繁全量复制生成RDB对系统造成的压力。

2）尝试性的支持IPv6.

3）可以通过config set命令设置maxclients。

4）可以用bind命令绑定多个IP地址。

5）Redis设置了明显的进程名，方便使用ps命令查看系统进程。

6）config rewrite命令可以将config set持久化到Redis配置文件中。

7）发布订阅添加了pubsub。

8）Redis Sentinel第二版，相比于Redis2.6的Redis Sentinel，此版本已经变成生产可用。

 

3.Redis3.0（里程碑）

Redis3.0在2015年4月1日正式发布，相比于Redis2.8主要特性如下：

Redis最大的改动就是添加Redis的分布式实现Redis Cluster。

1）Redis Cluster：Redis的官方分布式实现。

2）全新的embedded string对象编码结果，优化小对象内存访问，在特定的工作负载下载速度大幅提升。

3）Iru算法大幅提升。

4）migrate连接缓存，大幅提升键迁移的速度。

5）migrate命令两个新的参数copy和replace。

6）新的client pause命令，在指定时间内停止处理客户端请求。

7）bitcount命令性能提升。

8）cinfig set设置maxmemory时候可以设置不同的单位（之前只能是字节）。

9）Redis日志小做调整：日志中会反应当前实例的角色（master或者slave）。

10）incr命令性能提升。

 

4.Redis3.2

Redis3.2在2016年5月6日正式发布，相比于Redis3.0主要特征如下：

1）添加GEO相关功能。

2）SDS在速度和节省空间上都做了优化。

3）支持用upstart或者systemd管理Redis进程。

4）新的List编码类型：quicklist。

5）从节点读取过期数据保证一致性。

6）添加了hstrlen命令。

7）增强了debug命令，支持了更多的参数。

8）Lua脚本功能增强。

9）添加了Lua Debugger。

10）config set 支持更多的配置参数。

11）优化了Redis崩溃后的相关报告。

12）新的RDB格式，但是仍然兼容旧的RDB。

13）加速RDB的加载速度。

14）spop命令支持个数参数。

15）cluster nodes命令得到加速。

16）Jemalloc更新到4.0.3版本。

 

 

5.Redis4.0

可能出乎很多的意料，Redis3.2之后的版本是4.0，而不是3.4、3.6、3.8。

一般这种重大版本号的升级也意味着软件或者工具本身发生了重大改革。下面是Redis4.0的新特性：

1）提供了模块系统，方便第三方开发者拓展Redis的功能。

2）PSYNC2.0：优化了之前版本中，主从节点切换必然引起全量复制的问题。

3）提供了新的缓存剔除算法：LFU（Last Frequently Used），并对已有算法进行了优化。

4）提供了非阻塞del和flushall/flushdb功能，有效解决删除了bigkey可能造成的Redis阻塞。

5）提供了memory命令，实现对内存更为全面的监控统计。

6）提供了交互数据库功能，实现Redis内部数据库的数据置换。

7）提供了RDB-AOF混合持久化格式，充分利用了AOF和RDB各自优势。

8）Redis Cluster 兼容NAT和Docker。

 
6.Redis5.0
1.新的Stream数据类型。[1]5.0

2.新的Redis模块API：Timers and Cluster API。

3. RDB现在存储LFU和LRU信息。

4.集群管理器从Ruby（redis-trib.rb）移植到C代码。可以在redis-cli中。查看`redis-cli —cluster help`了解更多信息。

5.新sorted set命令：ZPOPMIN / MAX和阻塞变量。

6.主动碎片整理V2。

7.增强HyperLogLog实现。

8.更好的内存统计报告。

9.许多带有子命令的命令现在都有一个HELP子命令。

10.客户经常连接和断开连接时性能更好。

11.错误修复和改进。

12. Jemalloc升级到5.1版



【redis是什么】

redis是一个开源的、使用C语言编写的、支持网络交互的、可基于内存也可持久化的Key-Value数据库。

redis的官网地址，非常好记，是redis.io。（特意查了一下，域名后缀io属于国家域名，是british Indian Ocean territory，即英属印度洋领地）

目前，Vmware在资助着redis项目的开发和维护。

【redis的作者何许人也】

开门见山，先看照片：



是不是出乎了你的意料，嗯，高手总会有些地方与众不同的。

这位便是redis的作者，他叫Salvatore Sanfilippo，来自意大利的西西里岛，现在居住在卡塔尼亚。目前供职于Pivotal公司。

他使用的网名是antirez，如果你有兴趣，可以去他的博客逛逛，地址是antirez.com，当然也可以去follow他的github，地址是http://github.com/antirez。

【谁在使用redis】

Blizzard、digg、stackoverflow、github、flickr …

【学会安装redis】

从redis.io下载最新版redis-X.Y.Z.tar.gz后解压，然后进入redis-X.Y.Z文件夹后直接make即可，安装非常简单。

make成功后会在src文件夹下产生一些二进制可执行文件，包括redis-server、redis-cli等等：
复制代码代码如下:

$ find . -type f -executable
./redis-benchmark //用于进行redis性能测试的工具
./redis-check-dump //用于修复出问题的dump.rdb文件
./redis-cli //redis的客户端
./redis-server //redis的服务端
./redis-check-aof //用于修复出问题的AOF文件
./redis-sentinel //用于集群管理
【学会启动redis】

启动redis非常简单，直接./redis-server就可以启动服务端了，还可以用下面的方法指定要加载的配置文件：
复制代码代码如下:

./redis-server ../redis.conf


默认情况下，redis-server会以非daemon的方式来运行，且默认服务端口为6379。

有关作者为什么选择6379作为默认端口，还有一段有趣的典故，英语好的同学可以看看作者这篇博文中的解释。

【使用redis客户端】

我们直接看一个例子：
复制代码代码如下:

//这样来启动redis客户端了
$ ./redis-cli
//用set指令来设置key、value
127.0.0.1:6379> set name “roc” 
OK
//来获取name的值
127.0.0.1:6379> get name 
“roc”
//通过客户端来关闭redis服务端
127.0.0.1:6379> shutdown 
127.0.0.1:6379>
【redis数据结构 – 简介】

redis是一种高级的key:value存储系统，其中value支持五种数据类型：

1.字符串（strings）
2.字符串列表（lists）
3.字符串集合（sets）
4.有序字符串集合（sorted sets）
5.哈希（hashes）

而关于key，有几个点要提醒大家：

1.key不要太长，尽量不要超过1024字节，这不仅消耗内存，而且会降低查找的效率；
2.key也不要太短，太短的话，key的可读性会降低；
3.在一个项目中，key最好使用统一的命名模式，例如user:10000:passwd。

【redis数据结构 – strings】

有人说，如果只使用redis中的字符串类型，且不使用redis的持久化功能，那么，redis就和memcache非常非常的像了。这说明strings类型是一个很基础的数据类型，也是任何存储系统都必备的数据类型。

我们来看一个最简单的例子：
复制代码代码如下:

set mystr “hello world!” //设置字符串类型
get mystr //读取字符串类型


字符串类型的用法就是这么简单，因为是二进制安全的，所以你完全可以把一个图片文件的内容作为字符串来存储。

另外，我们还可以通过字符串类型进行数值操作：
复制代码代码如下:

127.0.0.1:6379> set mynum “2”
OK
127.0.0.1:6379> get mynum
“2”
127.0.0.1:6379> incr mynum
(integer) 3
127.0.0.1:6379> get mynum
“3”
看，在遇到数值操作时，redis会将字符串类型转换成数值。

由于INCR等指令本身就具有原子操作的特性，所以我们完全可以利用redis的INCR、INCRBY、DECR、DECRBY等指令来实现原子计数的效果，假如，在某种场景下有3个客户端同时读取了mynum的值（值为2），然后对其同时进行了加1的操作，那么，最后mynum的值一定是5。不少网站都利用redis的这个特性来实现业务上的统计计数需求。

【redis数据结构 – lists】

redis的另一个重要的数据结构叫做lists，翻译成中文叫做“列表”。

首先要明确一点，redis中的lists在底层实现上并不是数组，而是链表，也就是说对于一个具有上百万个元素的lists来说，在头部和尾部插入一个新元素，其时间复杂度是常数级别的，比如用LPUSH在10个元素的lists头部插入新元素，和在上千万元素的lists头部插入新元素的速度应该是相同的。

虽然lists有这样的优势，但同样有其弊端，那就是，链表型lists的元素定位会比较慢，而数组型lists的元素定位就会快得多。

lists的常用操作包括LPUSH、RPUSH、LRANGE等。我们可以用LPUSH在lists的左侧插入一个新元素，用RPUSH在lists的右侧插入一个新元素，用LRANGE命令从lists中指定一个范围来提取元素。我们来看几个例子：
复制代码代码如下:

//新建一个list叫做mylist，并在列表头部插入元素”1”
127.0.0.1:6379> lpush mylist “1” 
//返回当前mylist中的元素个数
(integer) 1 
//在mylist右侧插入元素”2”
127.0.0.1:6379> rpush mylist “2” 
(integer) 2
//在mylist左侧插入元素”0”
127.0.0.1:6379> lpush mylist “0” 
(integer) 3
//列出mylist中从编号0到编号1的元素
127.0.0.1:6379> lrange mylist 0 1 
1) “0”
2) “1”
//列出mylist中从编号0到倒数第一个元素
127.0.0.1:6379> lrange mylist 0 -1 
1) “0”
2) “1”
3) “2”
lists的应用相当广泛，随便举几个例子：

1.我们可以利用lists来实现一个消息队列，而且可以确保先后顺序，不必像MySQL那样还需要通过ORDER BY来进行排序。
2.利用LRANGE还可以很方便的实现分页的功能。
3.在博客系统中，每片博文的评论也可以存入一个单独的list中。

【redis数据结构 – 集合】

redis的集合，是一种无序的集合，集合中的元素没有先后顺序。

集合相关的操作也很丰富，如添加新元素、删除已有元素、取交集、取并集、取差集等。我们来看例子：
复制代码代码如下:

//向集合myset中加入一个新元素”one”
127.0.0.1:6379> sadd myset “one” 
(integer) 1
127.0.0.1:6379> sadd myset “two”
(integer) 1
//列出集合myset中的所有元素
127.0.0.1:6379> smembers myset 
1) “one”
2) “two”
//判断元素1是否在集合myset中，返回1表示存在
127.0.0.1:6379> sismember myset “one” 
(integer) 1
//判断元素3是否在集合myset中，返回0表示不存在
127.0.0.1:6379> sismember myset “three” 
(integer) 0
//新建一个新的集合yourset
127.0.0.1:6379> sadd yourset “1” 
(integer) 1
127.0.0.1:6379> sadd yourset “2”
(integer) 1
127.0.0.1:6379> smembers yourset
1) “1”
2) “2”
//对两个集合求并集
127.0.0.1:6379> sunion myset yourset 
1) “1”
2) “one”
3) “2”
4) “two”
对于集合的使用，也有一些常见的方式，比如，QQ有一个社交功能叫做“好友标签”，大家可以给你的好友贴标签，比如“大美女”、“土豪”、“欧巴”等等，这时就可以使用redis的集合来实现，把每一个用户的标签都存储在一个集合之中。

【redis数据结构 – 有序集合】

redis不但提供了无需集合（sets），还很体贴的提供了有序集合（sorted sets）。有序集合中的每个元素都关联一个序号（score），这便是排序的依据。

很多时候，我们都将redis中的有序集合叫做zsets，这是因为在redis中，有序集合相关的操作指令都是以z开头的，比如zrange、zadd、zrevrange、zrangebyscore等等

老规矩，我们来看几个生动的例子：
//新增一个有序集合myzset，并加入一个元素baidu.com，给它赋予的序号是1：
复制代码代码如下:

127.0.0.1:6379> zadd myzset 1 baidu.com 
(integer) 1
//向myzset中新增一个元素360.com，赋予它的序号是3
127.0.0.1:6379> zadd myzset 3 360.com 
(integer) 1
//向myzset中新增一个元素google.com，赋予它的序号是2
127.0.0.1:6379> zadd myzset 2 google.com 
(integer) 1
//列出myzset的所有元素，同时列出其序号，可以看出myzset已经是有序的了。
127.0.0.1:6379> zrange myzset 0 -1 with scores 
1) “baidu.com”
2) “1”
3) “google.com”
4) “2”
5) “360.com”
6) “3”
//只列出myzset的元素
127.0.0.1:6379> zrange myzset 0 -1 
1) “baidu.com”
2) “google.com”
3) “360.com”
【redis数据结构 – 哈希】

最后要给大家介绍的是hashes，即哈希。哈希是从redis-2.0.0版本之后才有的数据结构。

hashes存的是字符串和字符串值之间的映射，比如一个用户要存储其全名、姓氏、年龄等等，就很适合使用哈希。

我们来看一个例子：
复制代码代码如下:

//建立哈希，并赋值
127.0.0.1:6379> HMSET user:001 username antirez password P1pp0 age 34 
OK
//列出哈希的内容
127.0.0.1:6379> HGETALL user:001 
1) “username”
2) “antirez”
3) “password”
4) “P1pp0”
5) “age”
6) “34”
//更改哈希中的某一个值
127.0.0.1:6379> HSET user:001 password 12345 
(integer) 0
//再次列出哈希的内容
127.0.0.1:6379> HGETALL user:001 
1) “username”
2) “antirez”
3) “password”
4) “12345”
5) “age”
6) “34”
有关hashes的操作，同样很丰富，需要时，大家可以从这里查询。

【聊聊redis持久化 – 两种方式】

redis提供了两种持久化的方式，分别是RDB（Redis DataBase）和AOF（Append Only File）。

RDB，简而言之，就是在不同的时间点，将redis存储的数据生成快照并存储到磁盘等介质上；

AOF，则是换了一个角度来实现持久化，那就是将redis执行过的所有写指令记录下来，在下次redis重新启动时，只要把这些写指令从前到后再重复执行一遍，就可以实现数据恢复了。

其实RDB和AOF两种方式也可以同时使用，在这种情况下，如果redis重启的话，则会优先采用AOF方式来进行数据恢复，这是因为AOF方式的数据恢复完整度更高。

如果你没有数据持久化的需求，也完全可以关闭RDB和AOF方式，这样的话，redis将变成一个纯内存数据库，就像memcache一样。

【聊聊redis持久化 – RDB】

RDB方式，是将redis某一时刻的数据持久化到磁盘中，是一种快照式的持久化方法。

redis在进行数据持久化的过程中，会先将数据写入到一个临时文件中，待持久化过程都结束了，才会用这个临时文件替换上次持久化好的文件。正是这种特性，让我们可以随时来进行备份，因为快照文件总是完整可用的。

对于RDB方式，redis会单独创建（fork）一个子进程来进行持久化，而主进程是不会进行任何IO操作的，这样就确保了redis极高的性能。

如果需要进行大规模数据的恢复，且对于数据恢复的完整性不是非常敏感，那RDB方式要比AOF方式更加的高效。

虽然RDB有不少优点，但它的缺点也是不容忽视的。如果你对数据的完整性非常敏感，那么RDB方式就不太适合你，因为即使你每5分钟都持久化一次，当redis故障时，仍然会有近5分钟的数据丢失。所以，redis还提供了另一种持久化方式，那就是AOF。

【聊聊redis持久化 – AOF】

AOF，英文是Append Only File，即只允许追加不允许改写的文件。

如前面介绍的，AOF方式是将执行过的写指令记录下来，在数据恢复时按照从前到后的顺序再将指令都执行一遍，就这么简单。

我们通过配置redis.conf中的appendonly yes就可以打开AOF功能。如果有写操作（如SET等），redis就会被追加到AOF文件的末尾。

默认的AOF持久化策略是每秒钟fsync一次（fsync是指把缓存中的写指令记录到磁盘中），因为在这种情况下，redis仍然可以保持很好的处理性能，即使redis故障，也只会丢失最近1秒钟的数据。

如果在追加日志时，恰好遇到磁盘空间满、inode满或断电等情况导致日志写入不完整，也没有关系，redis提供了redis-check-aof工具，可以用来进行日志修复。

因为采用了追加方式，如果不做任何处理的话，AOF文件会变得越来越大，为此，redis提供了AOF文件重写（rewrite）机制，即当AOF文件的大小超过所设定的阈值时，redis就会启动AOF文件的内容压缩，只保留可以恢复数据的最小指令集。举个例子或许更形象，假如我们调用了100次INCR指令，在AOF文件中就要存储100条指令，但这明显是很低效的，完全可以把这100条指令合并成一条SET指令，这就是重写机制的原理。

在进行AOF重写时，仍然是采用先写临时文件，全部完成后再替换的流程，所以断电、磁盘满等问题都不会影响AOF文件的可用性，这点大家可以放心。

AOF方式的另一个好处，我们通过一个“场景再现”来说明。某同学在操作redis时，不小心执行了FLUSHALL，导致redis内存中的数据全部被清空了，这是很悲剧的事情。不过这也不是世界末日，只要redis配置了AOF持久化方式，且AOF文件还没有被重写（rewrite），我们就可以用最快的速度暂停redis并编辑AOF文件，将最后一行的FLUSHALL命令删除，然后重启redis，就可以恢复redis的所有数据到FLUSHALL之前的状态了。是不是很神奇，这就是AOF持久化方式的好处之一。但是如果AOF文件已经被重写了，那就无法通过这种方法来恢复数据了。

虽然优点多多，但AOF方式也同样存在缺陷，比如在同样数据规模的情况下，AOF文件要比RDB文件的体积大。而且，AOF方式的恢复速度也要慢于RDB方式。

如果你直接执行BGREWRITEAOF命令，那么redis会生成一个全新的AOF文件，其中便包括了可以恢复现有数据的最少的命令集。

如果运气比较差，AOF文件出现了被写坏的情况，也不必过分担忧，redis并不会贸然加载这个有问题的AOF文件，而是报错退出。这时可以通过以下步骤来修复出错的文件：

1.备份被写坏的AOF文件
2.运行redis-check-aof –fix进行修复
3.用diff -u来看下两个文件的差异，确认问题点
4.重启redis，加载修复后的AOF文件

【聊聊redis持久化 – AOF重写】

AOF重写的内部运行原理，我们有必要了解一下。

在重写即将开始之际，redis会创建（fork）一个“重写子进程”，这个子进程会首先读取现有的AOF文件，并将其包含的指令进行分析压缩并写入到一个临时文件中。

与此同时，主工作进程会将新接收到的写指令一边累积到内存缓冲区中，一边继续写入到原有的AOF文件中，这样做是保证原有的AOF文件的可用性，避免在重写过程中出现意外。

当“重写子进程”完成重写工作后，它会给父进程发一个信号，父进程收到信号后就会将内存中缓存的写指令追加到新AOF文件中。

当追加结束后，redis就会用新AOF文件来代替旧AOF文件，之后再有新的写指令，就都会追加到新的AOF文件中了。

【聊聊redis持久化 – 如何选择RDB和AOF】

对于我们应该选择RDB还是AOF，官方的建议是两个同时使用。这样可以提供更可靠的持久化方案。

【聊聊主从 – 用法】

像MySQL一样，redis是支持主从同步的，而且也支持一主多从以及多级从结构。

主从结构，一是为了纯粹的冗余备份，二是为了提升读性能，比如很消耗性能的SORT就可以由从服务器来承担。

redis的主从同步是异步进行的，这意味着主从同步不会影响主逻辑，也不会降低redis的处理性能。

主从架构中，可以考虑关闭主服务器的数据持久化功能，只让从服务器进行持久化，这样可以提高主服务器的处理性能。

在主从架构中，从服务器通常被设置为只读模式，这样可以避免从服务器的数据被误修改。但是从服务器仍然可以接受CONFIG等指令，所以还是不应该将从服务器直接暴露到不安全的网络环境中。如果必须如此，那可以考虑给重要指令进行重命名，来避免命令被外人误执行。

【聊聊主从 – 同步原理】

从服务器会向主服务器发出SYNC指令，当主服务器接到此命令后，就会调用BGSAVE指令来创建一个子进程专门进行数据持久化工作，也就是将主服务器的数据写入RDB文件中。在数据持久化期间，主服务器将执行的写指令都缓存在内存中。

在BGSAVE指令执行完成后，主服务器会将持久化好的RDB文件发送给从服务器，从服务器接到此文件后会将其存储到磁盘上，然后再将其读取到内存中。这个动作完成后，主服务器会将这段时间缓存的写指令再以redis协议的格式发送给从服务器。

另外，要说的一点是，即使有多个从服务器同时发来SYNC指令，主服务器也只会执行一次BGSAVE，然后把持久化好的RDB文件发给多个下游。在redis2.8版本之前，如果从服务器与主服务器因某些原因断开连接的话，都会进行一次主从之间的全量的数据同步；而在2.8版本之后，redis支持了效率更高的增量同步策略，这大大降低了连接断开的恢复成本。

主服务器会在内存中维护一个缓冲区，缓冲区中存储着将要发给从服务器的内容。从服务器在与主服务器出现网络瞬断之后，从服务器会尝试再次与主服务器连接，一旦连接成功，从服务器就会把“希望同步的主服务器ID”和“希望请求的数据的偏移位置（replication offset）”发送出去。主服务器接收到这样的同步请求后，首先会验证主服务器ID是否和自己的ID匹配，其次会检查“请求的偏移位置”是否存在于自己的缓冲区中，如果两者都满足的话，主服务器就会向从服务器发送增量内容。

增量同步功能，需要服务器端支持全新的PSYNC指令。这个指令，只有在redis-2.8之后才具有。

【聊聊redis的事务处理】

众所周知，事务是指“一个完整的动作，要么全部执行，要么什么也没有做”。

在聊redis事务处理之前，要先和大家介绍四个redis指令，即MULTI、EXEC、DISCARD、WATCH。这四个指令构成了redis事务处理的基础。

1.MULTI用来组装一个事务；
2.EXEC用来执行一个事务；
3.DISCARD用来取消一个事务；
4.WATCH用来监视一些key，一旦这些key在事务执行之前被改变，则取消事务的执行。

纸上得来终觉浅，我们来看一个MULTI和EXEC的例子：
复制代码代码如下:

redis> MULTI //标记事务开始
OK
redis> INCR user_id //多条命令按顺序入队
QUEUED
redis> INCR user_id
QUEUED
redis> INCR user_id
QUEUED
redis> PING
QUEUED
redis> EXEC //执行
1) (integer) 1
2) (integer) 2
3) (integer) 3
4) PONG
在上面的例子中，我们看到了QUEUED的字样，这表示我们在用MULTI组装事务时，每一个命令都会进入到内存队列中缓存起来，如果出现QUEUED则表示我们这个命令成功插入了缓存队列，在将来执行EXEC时，这些被QUEUED的命令都会被组装成一个事务来执行。

对于事务的执行来说，如果redis开启了AOF持久化的话，那么一旦事务被成功执行，事务中的命令就会通过write命令一次性写到磁盘中去，如果在向磁盘中写的过程中恰好出现断电、硬件故障等问题，那么就可能出现只有部分命令进行了AOF持久化，这时AOF文件就会出现不完整的情况，这时，我们可以使用redis-check-aof工具来修复这一问题，这个工具会将AOF文件中不完整的信息移除，确保AOF文件完整可用。

有关事务，大家经常会遇到的是两类错误：

1.调用EXEC之前的错误
2.调用EXEC之后的错误

“调用EXEC之前的错误”，有可能是由于语法有误导致的，也可能时由于内存不足导致的。只要出现某个命令无法成功写入缓冲队列的情况，redis都会进行记录，在客户端调用EXEC时，redis会拒绝执行这一事务。（这时2.6.5版本之后的策略。在2.6.5之前的版本中，redis会忽略那些入队失败的命令，只执行那些入队成功的命令）。我们来看一个这样的例子：
复制代码代码如下:

127.0.0.1:6379> multi
OK
127.0.0.1:6379> haha //一个明显错误的指令
(error) ERR unknown command ‘haha’
127.0.0.1:6379> ping
QUEUED
127.0.0.1:6379> exec
//redis无情的拒绝了事务的执行，原因是“之前出现了错误”
(error) EXECABORT Transaction discarded because of previous errors.
而对于“调用EXEC之后的错误”，redis则采取了完全不同的策略，即redis不会理睬这些错误，而是继续向下执行事务中的其他命令。这是因为，对于应用层面的错误，并不是redis自身需要考虑和处理的问题，所以一个事务中如果某一条命令执行失败，并不会影响接下来的其他命令的执行。我们也来看一个例子：

复制代码代码如下:

127.0.0.1:6379> multi
OK
127.0.0.1:6379> set age 23
QUEUED
//age不是集合，所以如下是一条明显错误的指令
127.0.0.1:6379> sadd age 15 
QUEUED
127.0.0.1:6379> set age 29
QUEUED
127.0.0.1:6379> exec //执行事务时，redis不会理睬第2条指令执行错误
1) OK
2) (error) WRONGTYPE Operation against a key holding the wrong kind of value
3) OK
127.0.0.1:6379> get age
“29” //可以看出第3条指令被成功执行了
好了，我们来说说最后一个指令“WATCH”，这是一个很好用的指令，它可以帮我们实现类似于“乐观锁”的效果，即CAS（check and set）。

WATCH本身的作用是“监视key是否被改动过”，而且支持同时监视多个key，只要还没真正触发事务，WATCH都会尽职尽责的监视，一旦发现某个key被修改了，在执行EXEC时就会返回nil，表示事务无法触发。
复制代码代码如下:

127.0.0.1:6379> set age 23
OK
127.0.0.1:6379> watch age //开始监视age
OK
127.0.0.1:6379> set age 24 //在EXEC之前，age的值被修改了
OK
127.0.0.1:6379> multi
OK
127.0.0.1:6379> set age 25
QUEUED
127.0.0.1:6379> get age
QUEUED
127.0.0.1:6379> exec //触发EXEC
(nil) //事务无法被执行
【教你看懂redis配置 – 简介】

我们可以在启动redis-server时指定应该加载的配置文件，方法如下：
复制代码代码如下:

$ ./redis-server /path/to/redis.conf
接下来，我们就来讲解下redis配置文件的各个配置项的含义，注意，本文是基于redis-2.8.4版本进行讲解的。

redis官方提供的redis.conf文件，足有700+行，其中100多行为有效配置行，另外的600多行为注释说明。

在配置文件的开头部分，首先明确了一些度量单位：
复制代码代码如下:


# 1k => 1000 bytes
# 1kb => 1024 bytes
# 1m => 1000000 bytes
# 1mb => 1024*1024 bytes
# 1g => 1000000000 bytes
# 1gb => 1024*1024*1024 bytes
可以看出，redis配置中对单位的大小写不敏感，1GB、1Gb和1gB都是相同的。由此也说明，redis只支持bytes，不支持bit单位。

redis支持“主配置文件中引入外部配置文件”，很像C/C++中的include指令，比如：
复制代码代码如下:

include /path/to/other.conf
如果你看过redis的配置文件，会发现还是很有条理的。redis配置文件被分成了几大块区域，它们分别是：

1.通用（general）
2.快照（snapshotting）
3.复制（replication）
4.安全（security）
5.限制（limits)
6.追加模式（append only mode)
7.LUA脚本（lua scripting)
8.慢日志（slow log)
9.事件通知（event notification）

下面我们就来逐一讲解。

【教你看懂redis配置 -通用】

默认情况下，redis并不是以daemon形式来运行的。通过daemonize配置项可以控制redis的运行形式，如果改为yes，那么redis就会以daemon形式运行：
复制代码代码如下:

daemonize no


当以daemon形式运行时，redis会生成一个pid文件，默认会生成在/var/run/redis.pid。当然，你可以通过pidfile来指定pid文件生成的位置，比如：

复制代码代码如下:

pidfile /path/to/redis.pid


默认情况下，redis会响应本机所有可用网卡的连接请求。当然，redis允许你通过bind配置项来指定要绑定的IP，比如：

复制代码代码如下:

bind 192.168.1.2 10.8.4.2


redis的默认服务端口是6379，你可以通过port配置项来修改。如果端口设置为0的话，redis便不会监听端口了。

复制代码代码如下:

port 6379


有些同学会问“如果redis不监听端口，还怎么与外界通信呢”，其实redis还支持通过unix socket方式来接收请求。可以通过unixsocket配置项来指定unix socket文件的路径，并通过unixsocketperm来指定文件的权限。

复制代码代码如下:

unixsocket /tmp/redis.sock
unixsocketperm 755
当一个redis-client一直没有请求发向server端，那么server端有权主动关闭这个连接，可以通过timeout来设置“空闲超时时限”，0表示永不关闭。
复制代码代码如下:

timeout 0


TCP连接保活策略，可以通过tcp-keepalive配置项来进行设置，单位为秒，假如设置为60秒，则server端会每60秒向连接空闲的客户端发起一次ACK请求，以检查客户端是否已经挂掉，对于无响应的客户端则会关闭其连接。所以关闭一个连接最长需要120秒的时间。如果设置为0，则不会进行保活检测。

复制代码代码如下:

tcp-keepalive 0


redis支持通过loglevel配置项设置日志等级，共分四级，即debug、verbose、notice、warning。

复制代码代码如下:

loglevel notice


redis也支持通过logfile配置项来设置日志文件的生成位置。如果设置为空字符串，则redis会将日志输出到标准输出。假如你在daemon情况下将日志设置为输出到标准输出，则日志会被写到/dev/null中。

复制代码代码如下:

logfile “”


如果希望日志打印到syslog中，也很容易，通过syslog-enabled来控制。另外，syslog-ident还可以让你指定syslog里的日志标志，比如：

复制代码代码如下:

syslog-ident redis


而且还支持指定syslog设备，值可以是USER或LOCAL0-LOCAL7。具体可以参考syslog服务本身的用法。

复制代码代码如下:

syslog-facility local0


对于redis来说，可以设置其数据库的总数量，假如你希望一个redis包含16个数据库，那么设置如下：

复制代码代码如下:

databases 16


这16个数据库的编号将是0到15。默认的数据库是编号为0的数据库。用户可以使用select <DBid>来选择相应的数据库。

【教你看懂redis配置 – 快照】

快照，主要涉及的是redis的RDB持久化相关的配置，我们来一起看一看。

我们可以用如下的指令来让数据保存到磁盘上，即控制RDB快照功能：
复制代码代码如下:

save <seconds> <changes>


举例来说：

复制代码代码如下:

save 900 1 //表示每15分钟且至少有1个key改变，就触发一次持久化
save 300 10 //表示每5分钟且至少有10个key改变，就触发一次持久化

save 60 10000 //表示每60秒至少有10000个key改变，就触发一次持久化


如果你想禁用RDB持久化的策略，只要不设置任何save指令就可以，或者给save传入一个空字符串参数也可以达到相同效果，就像这样：

复制代码代码如下:

save “”


如果用户开启了RDB快照功能，那么在redis持久化数据到磁盘时如果出现失败，默认情况下，redis会停止接受所有的写请求。这样做的好处在于可以让用户很明确的知道内存中的数据和磁盘上的数据已经存在不一致了。如果redis不顾这种不一致，一意孤行的继续接收写请求，就可能会引起一些灾难性的后果。

如果下一次RDB持久化成功，redis会自动恢复接受写请求。

当然，如果你不在乎这种数据不一致或者有其他的手段发现和控制这种不一致的话，你完全可以关闭这个功能，以便在快照写入失败时，也能确保redis继续接受新的写请求。配置项如下：
复制代码代码如下:

stop-writes-on-bgsave-error yes


对于存储到磁盘中的快照，可以设置是否进行压缩存储。如果是的话，redis会采用LZF算法进行压缩。如果你不想消耗CPU来进行压缩的话，可以设置为关闭此功能，但是存储在磁盘上的快照会比较大。

复制代码代码如下:

rdbcompression yes


在存储快照后，我们还可以让redis使用CRC64算法来进行数据校验，但是这样做会增加大约10%的性能消耗，如果你希望获取到最大的性能提升，可以关闭此功能。

复制代码代码如下:

rdbchecksum yes


我们还可以设置快照文件的名称，默认是这样配置的：

复制代码代码如下:

dbfilename dump.rdb


最后，你还可以设置这个快照文件存放的路径。比如默认设置就是当前文件夹：

复制代码代码如下:

dir ./
【教你看懂redis配置 – 复制】

redis提供了主从同步功能。

通过slaveof配置项可以控制某一个redis作为另一个redis的从服务器，通过指定IP和端口来定位到主redis的位置。一般情况下，我们会建议用户为从redis设置一个不同频率的快照持久化的周期，或者为从redis配置一个不同的服务端口等等。
复制代码代码如下:

slaveof <masterip> <masterport>


如果主redis设置了验证密码的话（使用requirepass来设置），则在从redis的配置中要使用masterauth来设置校验密码，否则的话，主redis会拒绝从redis的访问请求。

复制代码代码如下:

masterauth <master-password>


当从redis失去了与主redis的连接，或者主从同步正在进行中时，redis该如何处理外部发来的访问请求呢？这里，从redis可以有两种选择：

第一种选择：如果slave-serve-stale-data设置为yes（默认），则从redis仍会继续响应客户端的读写请求。

第二种选择：如果slave-serve-stale-data设置为no，则从redis会对客户端的请求返回“SYNC with master in progress”，当然也有例外，当客户端发来INFO请求和SLAVEOF请求，从redis还是会进行处理。

你可以控制一个从redis是否可以接受写请求。将数据直接写入从redis，一般只适用于那些生命周期非常短的数据，因为在主从同步时，这些临时数据就会被清理掉。自从redis2.6版本之后，默认从redis为只读。
复制代码代码如下:

slave-read-only yes


只读的从redis并不适合直接暴露给不可信的客户端。为了尽量降低风险，可以使用rename-command指令来将一些可能有破坏力的命令重命名，避免外部直接调用。比如：

复制代码代码如下:

rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c52


从redis会周期性的向主redis发出PING包。你可以通过repl_ping_slave_period指令来控制其周期。默认是10秒。

复制代码代码如下:

repl-ping-slave-period 10


在主从同步时，可能在这些情况下会有超时发生：

1.以从redis的角度来看，当有大规模IO传输时。
2.以从redis的角度来看，当数据传输或PING时，主redis超时
3.以主redis的角度来看，在回复从redis的PING时，从redis超时

用户可以设置上述超时的时限，不过要确保这个时限比repl-ping-slave-period的值要大，否则每次主redis都会认为从redis超时。
复制代码代码如下:

repl-timeout 60


我们可以控制在主从同步时是否禁用TCP_NODELAY。如果开启TCP_NODELAY，那么主redis会使用更少的TCP包和更少的带宽来向从redis传输数据。但是这可能会增加一些同步的延迟，大概会达到40毫秒左右。如果你关闭了TCP_NODELAY，那么数据同步的延迟时间会降低，但是会消耗更多的带宽。（如果你不了解TCP_NODELAY，可以到这里来科普一下）。

复制代码代码如下:

repl-disable-tcp-nodelay no


我们还可以设置同步队列长度。队列长度（backlog)是主redis中的一个缓冲区，在与从redis断开连接期间，主redis会用这个缓冲区来缓存应该发给从redis的数据。这样的话，当从redis重新连接上之后，就不必重新全量同步数据，只需要同步这部分增量数据即可。

复制代码代码如下:

repl-backlog-size 1mb


如果主redis等了一段时间之后，还是无法连接到从redis，那么缓冲队列中的数据将被清理掉。我们可以设置主redis要等待的时间长度。如果设置为0，则表示永远不清理。默认是1个小时。

复制代码代码如下:

repl-backlog-ttl 3600


我们可以给众多的从redis设置优先级，在主redis持续工作不正常的情况，优先级高的从redis将会升级为主redis。而编号越小，优先级越高。比如一个主redis有三个从redis，优先级编号分别为10、100、25，那么编号为10的从redis将会被首先选中升级为主redis。当优先级被设置为0时，这个从redis将永远也不会被选中。默认的优先级为100。

复制代码代码如下:

slave-priority 100


假如主redis发现有超过M个从redis的连接延时大于N秒，那么主redis就停止接受外来的写请求。这是因为从redis一般会每秒钟都向主redis发出PING，而主redis会记录每一个从redis最近一次发来PING的时间点，所以主redis能够了解每一个从redis的运行情况。

复制代码代码如下:

min-slaves-to-write 3
min-slaves-max-lag 10


上面这个例子表示，假如有大于等于3个从redis的连接延迟大于10秒，那么主redis就不再接受外部的写请求。上述两个配置中有一个被置为0，则这个特性将被关闭。默认情况下min-slaves-to-write为0，而min-slaves-max-lag为10。

【教你看懂redis配置 – 安全】

我们可以要求redis客户端在向redis-server发送请求之前，先进行密码验证。当你的redis-server处于一个不太可信的网络环境中时，相信你会用上这个功能。由于redis性能非常高，所以每秒钟可以完成多达15万次的密码尝试，所以你最好设置一个足够复杂的密码，否则很容易被黑客破解。
复制代码代码如下:

requirepass zhimakaimen


这里我们通过requirepass将密码设置成“芝麻开门”。

redis允许我们对redis指令进行更名，比如将一些比较危险的命令改个名字，避免被误执行。比如可以把CONFIG命令改成一个很复杂的名字，这样可以避免外部的调用，同时还可以满足内部调用的需要：
复制代码代码如下:

rename-command CONFIG b840fc02d524045429941cc15f59e41cb7be6c89


我们甚至可以禁用掉CONFIG命令，那就是把CONFIG的名字改成一个空字符串：

复制代码代码如下:

rename-command CONFIG “”


但需要注意的是，如果你使用AOF方式进行数据持久化，或者需要与从redis进行通信，那么更改指令的名字可能会引起一些问题。

【教你看懂redis配置 -限制】

我们可以设置redis同时可以与多少个客户端进行连接。默认情况下为10000个客户端。当你无法设置进程文件句柄限制时，redis会设置为当前的文件句柄限制值减去32，因为redis会为自身内部处理逻辑留一些句柄出来。

如果达到了此限制，redis则会拒绝新的连接请求，并且向这些连接请求方发出“max number of clients reached”以作回应。
复制代码代码如下:

maxclients 10000


我们甚至可以设置redis可以使用的内存量。一旦到达内存使用上限，redis将会试图移除内部数据，移除规则可以通过maxmemory-policy来指定。

如果redis无法根据移除规则来移除内存中的数据，或者我们设置了“不允许移除”，那么redis则会针对那些需要申请内存的指令返回错误信息，比如SET、LPUSH等。但是对于无内存申请的指令，仍然会正常响应，比如GET等。
复制代码代码如下:

maxmemory <bytes>


需要注意的一点是，如果你的redis是主redis（说明你的redis有从redis），那么在设置内存使用上限时，需要在系统中留出一些内存空间给同步队列缓存，只有在你设置的是“不移除”的情况下，才不用考虑这个因素。

对于内存移除规则来说，redis提供了多达6种的移除规则。他们是：

1.volatile-lru：使用LRU算法移除过期集合中的key
2.allkeys-lru：使用LRU算法移除key
3.volatile-random：在过期集合中移除随机的key
4.allkeys-random：移除随机的key
5.volatile-ttl：移除那些TTL值最小的key，即那些最近才过期的key。
6.noeviction：不进行移除。针对写操作，只是返回错误信息。

无论使用上述哪一种移除规则，如果没有合适的key可以移除的话，redis都会针对写请求返回错误信息。
复制代码代码如下:

maxmemory-policy volatile-lru


LRU算法和最小TTL算法都并非是精确的算法，而是估算值。所以你可以设置样本的大小。假如redis默认会检查三个key并选择其中LRU的那个，那么你可以改变这个key样本的数量。

复制代码代码如下:

maxmemory-samples 3


最后，我们补充一个信息，那就是到目前版本（2.8.4）为止，redis支持的写指令包括了如下这些：

复制代码代码如下:

set setnx setex append
incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
getset mset msetnx exec sort
【教你看懂redis配置 – 追加模式】

默认情况下，redis会异步的将数据持久化到磁盘。这种模式在大部分应用程序中已被验证是很有效的，但是在一些问题发生时，比如断电，则这种机制可能会导致数分钟的写请求丢失。

如博文上半部分中介绍的，追加文件（Append Only File）是一种更好的保持数据一致性的方式。即使当服务器断电时，也仅会有1秒钟的写请求丢失，当redis进程出现问题且操作系统运行正常时，甚至只会丢失一条写请求。

我们建议大家，AOF机制和RDB机制可以同时使用，不会有任何冲突。对于如何保持数据一致性的讨论，请参见本文。

复制代码代码如下:

appendonly no


我们还可以设置aof文件的名称：

复制代码代码如下:

appendfilename “appendonly.aof”
fsync()调用，用来告诉操作系统立即将缓存的指令写入磁盘。一些操作系统会“立即”进行，而另外一些操作系统则会“尽快”进行。

redis支持三种不同的模式：

1.no：不调用fsync()。而是让操作系统自行决定sync的时间。这种模式下，redis的性能会最快。
2.always：在每次写请求后都调用fsync()。这种模式下，redis会相对较慢，但数据最安全。
3.everysec：每秒钟调用一次fsync()。这是性能和安全的折衷。

默认情况下为everysec。有关数据一致性的揭秘，可以参考本文。

复制代码代码如下:

appendfsync everysec


当fsync方式设置为always或everysec时，如果后台持久化进程需要执行一个很大的磁盘IO操作，那么redis可能会在fsync()调用时卡住。目前尚未修复这个问题，这是因为即使我们在另一个新的线程中去执行fsync()，也会阻塞住同步写调用。

为了缓解这个问题，我们可以使用下面的配置项，这样的话，当BGSAVE或BGWRITEAOF运行时，fsync()在主进程中的调用会被阻止。这意味着当另一路进程正在对AOF文件进行重构时，redis的持久化功能就失效了，就好像我们设置了“appendsync none”一样。如果你的redis有时延问题，那么请将下面的选项设置为yes。否则请保持no，因为这是保证数据完整性的最安全的选择。
复制代码代码如下:

no-appendfsync-on-rewrite no


我们允许redis自动重写aof。当aof增长到一定规模时，redis会隐式调用BGREWRITEAOF来重写log文件，以缩减文件体积。

redis是这样工作的：redis会记录上次重写时的aof大小。假如redis自启动至今还没有进行过重写，那么启动时aof文件的大小会被作为基准值。这个基准值会和当前的aof大小进行比较。如果当前aof大小超出所设置的增长比例，则会触发重写。另外，你还需要设置一个最小大小，是为了防止在aof很小时就触发重写。
复制代码代码如下:

auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb


如果设置auto-aof-rewrite-percentage为0，则会关闭此重写功能。

【教你看懂redis配置 – LUA脚本】

lua脚本的最大运行时间是需要被严格限制的，要注意单位是毫秒：
复制代码代码如下:

lua-time-limit 5000


如果此值设置为0或负数，则既不会有报错也不会有时间限制。

【教你看懂redis配置 – 慢日志】

redis慢日志是指一个系统进行日志查询超过了指定的时长。这个时长不包括IO操作，比如与客户端的交互、发送响应内容等，而仅包括实际执行查询命令的时间。

针对慢日志，你可以设置两个参数，一个是执行时长，单位是微秒，另一个是慢日志的长度。当一个新的命令被写入日志时，最老的一条会从命令日志队列中被移除。

单位是微秒，即1000000表示一秒。负数则会禁用慢日志功能，而0则表示强制记录每一个命令。
复制代码代码如下:

slowlog-log-slower-than 10000


慢日志最大长度，可以随便填写数值，没有上限，但要注意它会消耗内存。你可以使用SLOWLOG RESET来重设这个值。

复制代码代码如下:

slowlog-max-len 128
【教你看懂redis配置 – 事件通知】

redis可以向客户端通知某些事件的发生。这个特性的具体解释可以参见本文。

【教你看懂redis配置 – 高级配置】

有关哈希数据结构的一些配置项：
复制代码代码如下:

hash-max-ziplist-entries 512
hash-max-ziplist-value 64


有关列表数据结构的一些配置项：

复制代码代码如下:

list-max-ziplist-entries 512
list-max-ziplist-value 64


有关集合数据结构的配置项：

复制代码代码如下:

set-max-intset-entries 512


有关有序集合数据结构的配置项：

复制代码代码如下:

zset-max-ziplist-entries 128
zset-max-ziplist-value 64


关于是否需要再哈希的配置项：

复制代码代码如下:

activerehashing yes



