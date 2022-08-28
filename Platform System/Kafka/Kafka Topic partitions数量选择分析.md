# Kafka Topic partitions数量选择分析

## 一、partition 越多，吞吐量越大

首先我们需要明白以下事实：在kafka中，单个patition是kafka并行操作的最小单元。在producer和broker端，向每一个分区写入数据是可以完全并行化的，此时，可以通过加大硬件资源的利用率来提升系统的吞吐量，例如对数据进行压缩。在consumer段，kafka只允许单个partition的数据被一个consumer线程消费。因此，在consumer端，每一个Consumer Group内部的consumer并行度完全依赖于被消费的分区数量。综上所述，通常情况下，在一个Kafka集群中，partition的数量越多，意味着可以到达的吞吐量越大。

我们可以粗略地通过吞吐量来计算kafka集群的分区数量。假设对于单个partition，producer端的可达吞吐量为p，Consumer端的可达吞吐量为c，期望的目标吞吐量为t，那么集群所需要的partition数量至少为partition数=max(t/p,t/c)。在producer端，单个分区的吞吐量大小会受到批量大小、数据压缩方法、 确认类型（同步/异步）、复制因子等配置参数的影响。经过测试，在producer端，单个partition的吞吐量通常是在10MB/s左右。在consumer端，单个partition的吞吐量依赖于consumer端每个消息的应用逻辑处理速度。因此，我们需要对consumer端的吞吐量进行测量。

虽然随着时间的推移，我们能够对分区的数量进行添加，但是对于基于Key来生成的这一类消息需要我们重点关注。当producer向kafka写入基于key的消息时，kafka通过key的hash值来确定消息需要写入哪个具体的分区。通过这样的方案，kafka能够确保相同key值的数据可以写入同一个partition。kafka的这一能力对于一部分应用是极为重要的，例如对于同一个key的所有消息，consumer需要按消息的顺序进行有序消费。如果partition的数量发生改变，那么上面的有序性保证将不复存在。为了避免上述情况发生，通常的解决办法是多分配一些分区，以满足未来的需求。通常情况下，我们需要根据未来1到2年的目标吞吐量来设计kafka的分区数量。

一开始，我们可以基于当前的业务吞吐量为kafka集群分配较小的broker数量，随着时间的推移，我们可以向集群中增加更多的broker，然后在线方式将适当比例的partition转移到新增加的broker中去。通过这样的方法，我们可以在满足各种应用场景（包括基于key消息的场景）的情况下，保持业务吞吐量的扩展性。

在设计分区数时，除了吞吐量，还有一些其他因素值得考虑。正如我们后面即将看到的，对于一些应用场景，集群拥有过的分区将会带来负面的影响。

越多的分区需要打开更多地文件句柄
　　在kafka的broker中，每个分区都会对照着文件系统的一个目录。在kafka的数据日志文件目录中，每个日志数据段都会分配两个文件，一个索引文件和一个数据文件。当前版本的kafka，每个broker会为每个日志段文件打开一个index文件句柄和一个数据文件句柄。因此，随着partition的增多，需要底层操作系统配置更高的文件句柄数量限制。这更多的是一个配置问题。我们曾经见到过，在生产环境Kafka集群中，每个broker打开的文件句柄数量超过30,000。

## 二、更多的 partition 会导致更高的不可用性

Kafka通过多副本复制技术，实现kafka集群的高可用和稳定性。每个partition都会有多个数据副本，每个副本分别存在于不同的broker。所有的数据副本中，有一个数据副本为Leader，其他的数据副本为follower。在kafka集群内部，所有的数据副本皆采用自动化的方式进行管理，并且确保所有的数据副本的数据皆保持同步状态。不论是producer端还是consumer端发往partition的请求，皆通过leader数据副本所在的broker进行处理。当broker发生故障时，对于leader数据副本在该broker的所有partition将会变得暂时不可用。Kafka将会自动在其他数据副本中选择出一个leader，用于接收客户端的请求。这个过程由kafka controller节点broker自动完成，主要是从Zookeeper读取和修改受影响partition的一些元数据信息。在当前的kafka版本实现中，对于zookeeper的所有操作都是由kafka controller来完成的（serially的方式）。
　　在通常情况下，当一个broker有计划地停止服务时，那么controller会在服务停止之前，将该broker上的所有leader一个个地移走。由于单个leader的移动时间大约只需要花费几毫秒，因此从客户层面看，有计划的服务停机只会导致系统在很小时间窗口中不可用。（注：在有计划地停机时，系统每一个时间窗口只会转移一个leader，其他leader皆处于可用状态。）

然而，当broker非计划地停止服务时（例如，kill -9方式)，系统的不可用时间窗口将会与受影响的partition数量有关。假如，一个2节点的kafka集群中存在2000个partition，每个partition拥有2个数据副本。当其中一个broker非计划地宕机，所有1000个partition同时变得不可用。假设每一个partition恢复时间是5ms，那么1000个partition的恢复时间将会花费5秒钟。因此，在这种情况下，用户将会观察到系统存在5秒钟的不可用时间窗口。

更不幸的情况发生在宕机的broker恰好是controller节点时。在这种情况下，新leader节点的选举过程在controller节点恢复到新的broker之前不会启动。Controller节点的错误恢复将会自动地进行，但是新的controller节点需要从zookeeper中读取每一个partition的元数据信息用于初始化数据。例如，假设一个kafka集群存在10,000个partition，从zookeeper中恢复元数据时每个partition大约花费2ms，则controller的恢复将会增加约20秒的不可用时间窗口。

通常情况下，非计划的宕机事件发生的情况是很少的。如果系统可用性无法容忍这些少数情况的场景，我们最好是将每个broker的partition数量限制在2,000到4,000，每个kafka集群中partition的数量限制在10,000以内。

越多的分区可能增加端对端的延迟
　　Kafka端对端延迟定义为producer端发布消息到consumer端接收消息所需要的时间。即consumer接收消息的时间减去producer发布消息的时间。Kafka只有在消息提交之后，才会将消息暴露给消费者。例如，消息在所有in-sync副本列表同步复制完成之后才暴露。因此，in-sync副本复制所花时间将是kafka端对端延迟的最主要部分。在默认情况下，每个broker从其他broker节点进行数据副本复制时，该broker节点只会为此工作分配一个线程，该线程需要完成该broker所有partition数据的复制。经验显示，将1000个partition从一个broker到另一个broker所带来的时间延迟约为20ms，这意味着端对端的延迟至少是20ms。这样的延迟对于一些实时应用需求来说显得过长。

注意，上述问题可以通过增大kafka集群来进行缓解。例如，将1000个分区leader放到一个broker节点和放到10个broker节点，他们之间的延迟是存在差异的。在10个broker节点的集群中，每个broker节点平均需要处理100个分区的数据复制。此时，端对端的延迟将会从原来的数十毫秒变为仅仅需要几毫秒。

根据经验，如果你十分关心消息延迟问题，限制每个broker节点的partition数量是一个很好的主意：对于b各broker节点和复制因子为r的kafka集群，整个kafka集群的partition数量最好不超过100br个，即单个partition的leader数量不超过100.

越多的partition意味着需要客户端需要更多的内存
　　在最新发布的0.8.2版本的kafka中，我们开发了一个更加高效的Java producer。新版producer拥有一个比较好的特征，他允许用户为待接入消息存储空间设置内存大小上限。在内部实现层面，producer按照每一个partition来缓存消息。在数据积累到一定大小或者足够的时间时，积累的消息将会从缓存中移除并发往broker节点。

如果partition的数量增加，消息将会在producer端按更多的partition进行积累。众多的partition所消耗的内存汇集起来，有可能会超过设置的内容大小限制。当这种情况发生时，producer必须通过消息堵塞或者丢失一些新消息的方式解决上述问题，但是这两种做法都不理想。为了避免这种情况发生，我们必须重新将produder的内存设置得更大一些。

根据经验，为了达到较好的吞吐量，我们必须在producer端为每个分区分配至少几十KB的内存，并且在分区数量显著增加时调整可以使用的内存数量。

类似的事情对于consumer端依然有效。Consumer端每次从kafka按每个分区取出一批消息进行消费。消费的分区数越多，需要的内存数量越大。尽管如此，上述方式主要运用于非实时的应用场景。

总结
　　通常情况下，kafka集群中越多的partition会带来越高的吞吐量。但是，我们必须意识到集群的partition总量过大或者单个broker节点partition过多，都会对系统的可用性和消息延迟带来潜在的影响。未来，我们计划对这些限制进行一些改进，让kafka在分区数量方面变得更加可扩展。

## 三、如何确定分区数

“我应该选择几个分区？”——如果你在Kafka中国社区的群里，这样的问题你会经常碰到的。不过有些遗憾的是，我们似乎并没有很权威的答案能够解答这样的问题。其实这也不奇怪，毕竟这样的问题通常都是没有固定答案的。Kafka官网上标榜自己是"high-throughput distributed messaging system"，即一个高吞吐量的分布式消息引擎。那么怎么达到高吞吐量呢？Kafka在底层摒弃了Java堆缓存机制，采用了操作系统级别的页缓存，同时将随机写操作改为顺序写，再结合Zero-Copy的特性极大地改善了IO性能。但是，这只是一个方面，毕竟单机优化的能力是有上限的。如何通过水平扩展甚至是线性扩展来进一步提升吞吐量呢？ Kafka就是使用了分区(partition)，通过将topic的消息打散到多个分区并分布保存在不同的broker上实现了消息处理(不管是producer还是consumer)的高吞吐量。
Kafka的生产者和消费者都可以多线程地并行操作，而每个线程处理的是一个分区的数据。因此分区实际上是调优Kafka并行度的最小单元。对于producer而言，它实际上是用多个线程并发地向不同分区所在的broker发起Socket连接同时给这些分区发送消息；而consumer呢，同一个消费组内的所有consumer线程都被指定topic的某一个分区进行消费(具体如何确定consumer线程数目我们后面会详细说明)。所以说，如果一个topic分区越多，理论上整个集群所能达到的吞吐量就越大。
但分区是否越多越好呢？显然也不是，因为每个分区都有自己的开销：

- 客户端/服务端需要使用的内存就越多

  先说说客户端的情况。Kafka 0.8.2之后推出了Java版的全新的producer，这个producer有个参数batch.size，默认是16KB。它会为每个分区缓存消息，一旦满了就打包将消息批量发出。看上去这是个能够提升性能的设计。不过很显然，因为这个参数是分区级别的，如果分区数越多，这部分缓存所需的内存占用也会更多。假设你有10000个分区，按照默认设置，这部分缓存需要占用约157MB的内存。而consumer端呢？我们抛开获取数据所需的内存不说，只说线程的开销。如果还是假设有10000个分区，同时consumer线程数要匹配分区数(大部分情况下是最佳的消费吞吐量配置)的话，那么在consumer client就要创建10000个线程，也需要创建大约10000个Socket去获取分区数据。这里面的线程切换的开销本身已经不容小觑了。
  服务器端的开销也不小，如果阅读Kafka源码的话可以发现，服务器端的很多组件都在内存中维护了分区级别的缓存，比如controller，FetcherManager等，因此分区数越多，这种缓存的成本越久越大。

- 文件句柄开销

  每个分区在底层文件系统都有属于自己的一个目录。该目录下通常会有两个文件： base_offset.log和base_offset.index。Kafak的controller和ReplicaManager会为每个broker都保存这两个文件句柄(file handler)。很明显，如果分区数越多，所需要保持打开状态的文件句柄数也就越多，最终可能会突破你的ulimit -n的限制。

- 降低高可用性

  Kafka通过副本(replica)机制来保证高可用。具体做法就是为每个分区保存若干个副本(replica_factor指定副本数)。每个副本保存在不同的broker上。期中的一个副本充当leader 副本，负责处理producer和consumer请求。其他副本充当follower角色，由Kafka controller负责保证与leader的同步。如果leader所在的broker挂掉了，contorller会检测到然后在zookeeper的帮助下重选出新的leader——这中间会有短暂的不可用时间窗口，虽然大部分情况下可能只是几毫秒级别。但如果你有10000个分区，10个broker，也就是说平均每个broker上有1000个分区。此时这个broker挂掉了，那么zookeeper和controller需要立即对这1000个分区进行leader选举。比起很少的分区leader选举而言，这必然要花更长的时间，并且通常不是线性累加的。如果这个broker还同时是controller情况就更糟了。

- 更多的分区会导致端对端的延迟

  kafka端对端的延迟为producer端发布消息到consumer端消费消息所需的时间，即consumer接收消息的时间减去produce发布消息的时间。kafka在消息正确接收后才会暴露给消费者，即在保证in-sync副本复制成功之后才会暴露，瓶颈则来自于此。在一个broker上的副本从其他broker的leader上复制数据的时候只会开启一个线程，假设partition数量为n，每个副本同步的时间为1ms，那in-sync操作完成所需的时间即n*1ms，若n为10000，则需要10秒才能返回同步状态，数据才能暴露给消费者，这就导致了较大的端对端的延迟。

- 越多的partition会导致更长时间的恢复期

  kafka通过多副本复制技术，实现kafka的高可用性和稳定性。每个partition都会有多个副本存在于多个broker中，其中一个副本为leader，其余的为follower。当kafka集群其中一个broker出现故障时，在这个broker上的leader会需要在其他broker上重新选择一个副本启动为leader，这个过程由kafka controller来完成，主要是从Zookeeper读取和修改受影响partition的一些元数据信息。

  通常情况下，当一个broker有计划的停机上，该broker上的partition leader会在broker停机前有次序的一一移走，假设移走一个需要1ms，10个partition leader则需要10ms，这影响很小，并且在移动其中一个leader的时候，其他九个leader是可用的，因此实际上每个partition leader的不可用时间为1ms。但是在宕机情况下，所有的10个partition

  leader同时无法使用，需要依次移走，最长的leader则需要10ms的不可用时间窗口，平均不可用时间窗口为5.5ms，假设有10000个leader在此宕机的broker上，平均的不可用时间窗口则为5.5s。

  更极端的情况是，当时的broker是kafka controller所在的节点，那需要等待新的kafka leader节点在投票中产生并启用，之后新启动的kafka leader还需要从zookeeper中读取每一个partition的元数据信息用于初始化数据。在这之前partition leader的迁移一直处于等待状态。


说了这么多“废话”，很多人肯定已经不耐烦了。那你说到底要怎么确定分区数呢？答案就是：视情况而定。基本上你还是需要通过一系列实验和测试来确定。当然测试的依据应该是吞吐量。虽然LinkedIn这篇文章做了Kafka的基准测试，但它的结果其实对你意义不大，因为不同的硬件、软件、负载情况测试出来的结果必然不一样。我经常碰到的问题类似于，官网说每秒能到10MB，为什么我的producer每秒才1MB？ —— 且不说硬件条件，最后发现他使用的消息体有1KB，而官网的基准测试是用100B测出来的，因此根本没有可比性。不过你依然可以遵循一定的步骤来尝试确定分区数：创建一个只有1个分区的topic，然后测试这个topic的producer吞吐量和consumer吞吐量。假设它们的值分别是Tp和Tc，单位可以是MB/s。然后假设总的目标吞吐量是Tt，那么分区数 = Tt / max(Tp, Tc)
Tp表示producer的吞吐量。测试producer通常是很容易的，因为它的逻辑非常简单，就是直接发送消息到Kafka就好了。Tc表示consumer的吞吐量。测试Tc通常与应用的关系更大， 因为Tc的值取决于你拿到消息之后执行什么操作，因此Tc的测试通常也要麻烦一些。
另外，Kafka并不能真正地做到线性扩展(其实任何系统都不能)，所以你在规划你的分区数的时候最好多规划一下，这样未来扩展时候也更加方便。

## 消息-分区的分配

默认情况下，Kafka根据传递消息的key来进行分区的分配，即hash(key) % numPartitions，如下图所示:

```java
def partition(key: Any, numPartitions: Int): Int = {
    Utils.abs(key.hashCode) % numPartitions
}
```

这就保证了相同key的消息一定会被路由到相同的分区。如果你没有指定key，那么Kafka是如何确定这条消息去往哪个分区的呢？

```java
if(key == null) {  // 如果没有指定key
        val id = sendPartitionPerTopicCache.get(topic)  // 先看看Kafka有没有缓存的现成的分区Id
        id match {
          case Some(partitionId) =>  
            partitionId  // 如果有的话直接使用这个分区Id就好了
          case None => // 如果没有的话，
            val availablePartitions = topicPartitionList.filter(_.leaderBrokerIdOpt.isDefined)  //找出所有可用分区的leader所在的broker
            if (availablePartitions.isEmpty)
              throw new LeaderNotAvailableException("No leader for any partition in topic " + topic)
            val index = Utils.abs(Random.nextInt) % availablePartitions.size  // 从中随机挑一个
            val partitionId = availablePartitions(index).partitionId
            sendPartitionPerTopicCache.put(topic, partitionId) // 更新缓存以备下一次直接使用
            partitionId
        }
      }
```

可以看出，Kafka几乎就是随机找一个分区发送无key的消息，然后把这个分区号加入到缓存中以备后面直接使用——当然了，Kafka本身也会清空该缓存（默认每10分钟或每次请求topic元数据时）

## 如何设定 consumer 线程数

我个人的观点，如果你的分区数是N，那么最好线程数也保持为N，这样通常能够达到最大的吞吐量。超过N的配置只是浪费系统资源，因为多出的线程不会被分配到任何分区。让我们来看看具体Kafka是如何分配的。
topic下的一个分区只能被同一个consumer group下的一个consumer线程来消费，但反之并不成立，即一个consumer线程可以消费多个分区的数据，比如Kafka提供的ConsoleConsumer，默认就只是一个线程来消费所有分区的数据。——其实ConsoleConsumer可以使用通配符的功能实现同时消费多个topic数据，但这和本文无关。
再讨论分配策略之前，先说说KafkaStream——它是consumer的关键类，提供了遍历方法用于consumer程序调用实现数据的消费。其底层维护了一个阻塞队列，所以在没有新消息到来时，consumer是处于阻塞状态的，表现出来的状态就是consumer程序一直在等待新消息的到来。——你当然可以配置成带超时的consumer，具体参看参数consumer.timeout.ms的用法。
下面说说Kafka提供的两种分配策略： range和roundrobin，由参数partition.assignment.strategy指定，默认是range策略。本文只讨论range策略。所谓的range其实就是按照阶段平均分配。举个例子就明白了，假设你有10个分区，P0 ~ P9，consumer线程数是3， C0 ~ C2，那么每个线程都分配哪些分区呢？

C0 消费分区 0, 1, 2, 3

C1 消费分区 4, 5, 6

C2 消费分区 7, 8, 9

```java
val nPartsPerConsumer = curPartitions.size / curConsumers.size // 每个consumer至少保证消费的分区数
val nConsumersWithExtraPart = curPartitions.size % curConsumers.size // 还剩下多少个分区需要单独分配给开头的线程们
...
for (consumerThreadId <- consumerThreadIdSet) {   // 对于每一个consumer线程
        val myConsumerPosition = curConsumers.indexOf(consumerThreadId)  //算出该线程在所有线程中的位置，介于[0, n-1]
        assert(myConsumerPosition >= 0)
// startPart 就是这个线程要消费的起始分区数
        val startPart = nPartsPerConsumer * myConsumerPosition + myConsumerPosition.min(nConsumersWithExtraPart)
// nParts 就是这个线程总共要消费多少个分区
        val nParts = nPartsPerConsumer + (if (myConsumerPosition + 1 > nConsumersWithExtraPart) 0 else 1)
...
}
```

针对于这个例子，nPartsPerConsumer就是10/3=3，nConsumersWithExtraPart为10%3=1，说明每个线程至少保证3个分区，还剩下1个分区需要单独分配给开头的若干个线程。这就是为什么C0消费4个分区，后面的2个线程每个消费3个分区。

