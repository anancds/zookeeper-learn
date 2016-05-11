>>>>>>>>>>>#Zookeeper

[Zookeeper Document](https://zookeeper.apache.org/doc/r3.5.1-alpha/zookeeperOver.html "Zookeeper Document")

##Zookeeper: 一个分布式应用的协调服务

>   Zookeeper 是一个分布式的，开源协调服务框架。它暴露了一组简单的原语(primitives)，使得分布式应用能够基于这些服务，可以构建更高级的服务，比如同步，配置管理，分组，和命名。
Zookeeper设计上易于编码，同时在我们熟悉的目录树结构的文件系统上构建了一个数据模型。Zookeeper运行在java中，同时支持java和c。
协调服务众所周知很难保持正确性，很容易因为资源竞争和死锁而发生错误。所以Zookeeper的动机就是要解放每次分布式应用从必须从头开始实现协调服务。

###Zookeeper的设计目标

####Zookeeper是简单的

>   Zookeeper允许分布式进程通过一个共享分层的命名空间来相互协调，这个命名空间和标准的文件系统很类似。
Zookeeper命名空间由数据寄存器组成，用Zookeeper的说法就是znodes，这跟文件和目录非常相似。不像一个典型的文件系统设计之初是为了数据存储，而Zookeeper的数据是保存在内存中，这意味着Zookeeper可以实现高吞吐和低延迟。
Zookeeper的实现对高性能，高可用性，严格的顺序访问格外关注。高性能意味着它可以使用在大型分布式系统中。高可用性意味着它远离了单点错误。严格的顺序访问意味着复杂的同步原语可以在客户端实现。

####Zookeeper是有副本的

>   就像Zookeeper协调的分布式进程一样，Zookeeper本身就打算在服务器集群中使用副本机制，我们称之为ensemble。

![Zookeeper server](https://zookeeper.apache.org/doc/r3.5.1-alpha/images/zkservice.jpg "Zookeeper server")

>   组成Zookeeper服务的所有服务器必须彼此感知。它们维护了当前机器状态的内存镜像，有事务日志和持久化存储的快照。只要大多数集群是可用的，那么Zookeeper服务就是可用的。
客户端连接到其中一台Zookeeper服务器，客户端和服务器之间维护了一个TCP连接，并且通过TCP连接发送请求，获取响应，获取事件监听，发送心跳等。如果和这台Zookeeper服务器的TCP连接中断了，那么客户端会连上另外一台Zookeeper服务器。

####Zookeeper是有序的

>   Zookeeper给每一次的更新操作都赋予一个编号，这个编号反应了所有Zookeeper事务的顺序。随后的操作可以使用这个顺序去实现更高级的抽象，比如同步原语。

####Zookeeper是非常快的

>   Zookeeper对以读操作为主导的的应用负载尤其快速。Zookeeper应用运行在成千上万台服务器中，当读写比是10:1时，性能最优。


###数据模型和命名空间体系结构(erarchical namespace)

>   Zookeeper提供的命名空间非常像我们标准的文件系统。命名就是用slash(/)分割的路径元素。每一个Zookeeper节点在命名空间中都是用路径标记。

![ZooKeeper's Hierarchical Namespace](https://zookeeper.apache.org/doc/r3.5.1-alpha/images/zknamespace.jpg "ZooKeeper's Hierarchical Namespace")

###节点和临时节点

>   不像标准的文件系统，Zookeeper命名空间中的每个节点能有数据关联，也有子节点。这就像文件系统中的一个文件对象可以是一个文件也可以是一个目录。(Zookeeper被设计用来存储协调数据：状态信息，配置文件，位置信息等等，因此数据存储在每个节点通常非常小，从几字节到几千字节之间)。
>   我们使用znode这个术语来使得我们讨论Zookeeper数据节点相关内容时语义更加清晰明确。
>   znode维护了一个状态结构(stat structure),包含了数据修改的版本号，ACL修改及时间戳，允许缓存校验和协调更新。
>   znode数据每修改一次，版本号加1。比如每次客户端查询(retrieves)数据时也会收到当前数据的版本号。
>   存储在znode命名空间中的数据读写都是原子的。读数据时获得与znode相关的所有数据，写数据时替换所有数据，每个节点都有一个ACL(Access Control List)访问控制列表，严格控制着谁能进行操作。
>   Zookeeper还有临时节点的概念。这些节点随着会话创建而激活，会话结束节点被删除。临时节点(或者说会话节点)在实现例子的时候非常有用。

###条件更新和监听

>   Zookeeper支持监听，客户端可以设置监听znode节点，当znode节点更新时可以触发监听或者移除监听。当监听触发，客户端会收到一个包告诉客户端znode已经修改了。当客户端与Zookeeper节点的连接断开时，客户端会收到一个本地通知，这些特性都可以用到具体例子中。

###Zookeeper的保证(Guarantees)

>   Zookepper简单并且性能优异，基于这些目标，它成为构建复杂服务的基础，比如同步服务，Zookeeper提供了一系列的保证：

 * 顺序一致性：来自客户端的更新将会按发送的顺序被作用。
 * 原子性：更新要么成功要么失败，没有部分成功的结果。
 * 统一的系统镜像：无论客户端连接的是哪台Zookeeper服务器，看到的都是统一的服务(也就是说Zookeeper是无状态的)。
 * 可靠性保证：一旦客户端的更新操作被写入，这个状态从那个时间点开始就会被持久化，直到客户端覆盖这个修改。
 * 及时性(timeliness):客户端角度看Zookeeper系统是保证在特定时间内是实时更新的。
 
###简单的api

 Zookeeper的设计原则就是提供简单的编程接口，因此它仅仅提供了一下几个操作：
 * create：在目录树的某个位置创建一个节点。
 * delete：删除某个节点。
 * exists：判断在目录树上是否有这个节点。
 * get data：从节点上获取数据。
 * set data：写入数据到某个节点。
 * get children：获取某个节点的所有子节点。
 * sync：等待数据传播到其他节点。

###Zookeeper实现

>   Zookeeper组件构成图中展示了Zookeeper服务中比较高级的组件。除了请求处理器的异常情况外，组成Zookeeper集群的每台服务器保存着每个组件的副本。

![ZooKeeper components](https://zookeeper.apache.org/doc/r3.5.1-alpha/images/zkcomponents.jpg "ZooKeeper components")

>   副本数据库是一个内存数据库，保存着整个目录结构树的数据。所有的更新操作都记录在磁盘，用于异常情况的恢复。在更新操作作用于内存数据库前，所有的写入操作都是串行的(这就保证了数据的强一致性)。
>   每个Zookeeper服务器节点服务若干个客户端。客户端连接到某个指定服务器节点来提交请求。读请求都是由每个服务器内存数据库的本地副本进行服务的(可以提高读的性能,
这也就是为什么zookeeper在读写比例为10:1的情况下,性能最佳的原因)。
>   涉及到修改服务器状态和数据写入的请求，需要一致性协议来处理。
>   一致性协议中有一部分是这样规定的：客户端的所有写请求都是被作用于唯一的机器，称之为leader。其他的Zookeeper服务器称之为followers,接收从leader发来的建议，并同意传输过来的消息(也就是说对消息只有服从和传输的特性)。
>   消息层关注的是当leader挂掉后怎么去替换它，并同步leader和followers的数据。
>   Zookeeper使用客户端原子消息协议。因为消息层是原子的，Zookeeper能保证本地副本和服务器版本同步(never diverge)。当leader节点接收一个写入请求时，leader节点会计算下当前系统状态是什么，什么时候写入请求会作用到服务器，并把这个写入操作转换成一个事务记录下新的状态。

###使用情况

>   zookeeper的编程接口 设计的非常简单,但是用这些能实现一些其他更高级别的操作,比如同步原语,对成员进行分组和选举等等。
一些分布式的应用用这些接口实现一些其他比较高级的功能。

###性能

>   Zookeeper设计成高性能框架，事实如此吗？
>   来自雅虎的zookeeper开发团队的研究证明了这点。
>   在读占主要比例的应用中,性能尤佳,因为写操作涉及到服务器之间状态的同步.尤其是在协调服务这个典型案例中表现的尤其突出。

![ZooKeeper Throughput as the Read-Write Ratio Varies](https://zookeeper.apache.org/doc/r3.5.1-alpha/images/zkperfRW-3.2.jpg "ZooKeeper Throughput as the Read-Write Ratio Varies")

>   zookeeper 吞吐量和读写比例的变化关系用的是zookeeper3.2版本,运行在 双核 2Ghz 及SATA 15k RPM 处理器配置的服务器中.
其中一个负责zookeeper 日志设备. 快照信息写入操作系统驱动中.
写入请求是1Kb写入, 读请求也是1kb的写入(读写单元都是1Kb).
servers 标示着 整个zookeeper的集群大小, 即组成zookeeper服务的服务器数.
整个zookeeper集群有个限定,客户端都不能直接连接zookeeper的leader 节点.

###Zookeeper可靠性

>   为了展示下,我们系统随着时间的推移及错误出现,我们运行了一个一个由7个机器组成的zookeeper服务中,我们使用和此前一样饱和度的基准测试,但是这次我们设定写入操作的比例为30%, 这个比例是对我们预期的负载的保守估计值.
这个图中有几个比较关键的观测点.首先,如果从节点失败并快速恢复了,即使从节点失败了,zookeeper仍然能够承受住一个比较高的吞吐量.
但是,更为重要的一点事,leader节点的选举算法 能让系统快速恢复,防止吞吐量在短时间内迅速降低.观察发现, zookeeper能在200毫秒内选举出新的leader节点,
第三,随着从节点的恢复,zookeeper的吞吐量能快速提升,一旦恢复的从节点开始处理请求。

![Reliability in the Presence of Errors](https://zookeeper.apache.org/doc/r3.5.1-alpha/images/zkperfreliability.jpg "
Reliability in the Presence of Errors")