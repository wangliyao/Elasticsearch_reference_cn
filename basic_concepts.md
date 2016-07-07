# 基本概念

Elasticsearch 有一些核心概念，了解这些概念对于学习 Elasticsearch 有极大的帮助。

### 接近实时（NRT）

Elasticsearch 是一个接近实时的搜索平台，这意味着文档从索引到可被搜索会有轻微的延迟（通常是1秒）。

### 集群（Cluster）

一个集群包含了单个或多个节点（server），这些节点保存了所有的数据，并且提供了跨节点的联合索引和搜索功能。一个集群有一个唯一的标示名称，默认是 `elasticsearch`。这个名称非常重要，节点是根据集群的名称加入集群的，且单个节点只能是一个集群的一部分。

要确保在不同的开发环境下不重用相同的集群名称，否则你的节点会加入到错误的集群。例如，你可以使用 `logging-dev`, `logging-stage`, `logging-prod` 分别针对开发，测试和生产环境。

请注意：集群只有一个节点是完全合法的，此外，你也可以有多个独立的集群，每个集群的名称唯一。

### 节点（Node）

节点就是一台服务器，它是集群的一部分，保存了数据并且参与了集群的索引和搜索。与集群一样，节点也有唯一的标示符，它是一个随机的传奇人物名称，在集群启动的时候被分配给节点。如果你不想使用默认的名称，你可以使用任何你想要的名称。名称的重要性是出于管理目的，因为你需要知道网络中的服务器对应集群中的哪一个节点。

节点可以通过配置集群名称加入指定的集群。默认地，每个节点会被加入到名为 `elasticsearch` 的集群，这就意味着如果你在网络中启动一系列的节点，假设这些节点可以彼此发现，它们会自动的联结并加入到名为 `elasticsearch` 的集群。

一个单独的集群可以有任意数量的节点。此外，如果没有其他的 Elasticsearch 节点运行在你的网络中，启动一个单独的节点会默认自动加入到名为 `elasticsearch` 的集群。

### 索引（Index）

索引是一系列有相似特征文档（document）的集合。例如，你有一个顾客数据的索引，一个产品类别的索引，还有一个订单数据索引。索引被一个名称（必须都为小写）所标示，这个名称在索引，搜索，更新和删除文档的时候被使用。

在一个单独的集群中，你可以定义任意数量的索引。

### 文档（Document）

文档是可以被索引的基本数据单元。例如，你可以针对一个顾客建立一个文档，针对一个产品建立一个文档，也可以根据一个订单建立一个文档。文档使用 JSON(JavaScript Object Notation)数据格式，它是被广泛使用的互联网数据交换格式。

在索引当中，你可以存储任意数量的文档。注意，虽然文档在物理上是驻留在索引当中，但是文档实际上必须被索引或分配到索引内部的一个类型（type）当中。

### 分片和副本（Shards & Replicas)

索引可以存储大量的数据，并且可以突破单个节点的硬件限制。例如，一个包涵10亿文档的索引会占据1TB的磁盘，单个节点也许无法容纳这么多的数据，也可能无法快速响应搜索请求。

为了解决这个问题，Elasticsearch 提供了将索引切分成多个分片的能力。当你创建索引的时候，你可以指定分片的数量。每一个分片本身是拥有完整功能和独立的索引，可以放置在集群中的任意节点。

分片的重要性主要体现在两个方面：
* 可以水平的切分／扩展数据集。
* 允许跨分片的分配和并行化操作以提高性能和吞吐量。

分片的分配机制和文档聚合回搜索请求的机制是完全由 Elasticsearch 管理的，对用户是透明的。

在网络／云环境下，失败是无法避免的，一旦分片／节点因为某些原因下线或消失，强烈推荐使用故障转移机制，为此，Elasticsearch 允许你为分片设置一份或多份拷贝，这些拷贝被称作副本分片，简称副本。

副本的重要性体现在两个方面：
* 一旦分片／节点出现故障，它提供了高可用性。基于这个原因，副本分片和原始分片不在同一个节点是非常有必要的。
* 因为搜索可以并行的在所有的副本上执行，因此可以扩大搜索数据集和吞吐量。

总的来说，每个索引都可以被切割成多个分片，每个索引也可以被复制零次（意味着没有副本）或多次。一旦索引被复制，每个分片就有主分片（被复制前的原始分片）和副本分片（主分片的副本）之分。分片和副本的数量可以在索引被创建的时候指定。索引创建之后，你可以动态的改变副本的数量，但是你无法改变分片的数量。

默认情况下，Elasticsearch 中的索引会产生5个主分片和一个副本。如果你的集群有两个节点，每个索引将会有5个主分片和5个副本分片，总共10个分片。

> 每个 Elasticsearch 分片都是一个 `Lucene index`，单个 `Lucene index` 有最大文档数量限制。根据[LUCENE-5843](https://issues.apache.org/jira/browse/LUCENE-5843)，限制是 2,147,483,519（= Integer.MAX_VALUE - 128）。你可以使用 [_cat/shards](https://www.elastic.co/guide/en/elasticsearch/reference/current/cat-shards.html) API 监控分片数量。

现在，把上述事情放在一边，让我们开始有趣的部分...