高可用集群
可扩展集群
数据恢复
故障转移

# 主从模式
https://redis.io/topics/replication
支持异步复制和同步复制，默认用异步复制
主从机都要开启持久化

Replication ID, offset

Replication ID 表示同一个数据集
offset同一数据集中主从节点当前数据的标记。
为什么升级为主副本的副本在故障转移后需要更改其复制ID：由于某些网络分区，旧主副本可能仍然作为主副本工作：保留相同的复制ID将违反以下事实：任意两个随机实例的同一ID和同一偏移量意味着它们具有相同的数据集。
禁用从节点的只读功能。
从Redis 2.8开始，只有当至少有N个副本当前连接到主服务器时，才可以将Redis主服务器配置为接受写查询。


Redis expires允许密钥有有限的生存时间。这样的特性取决于实例计算时间的能力，但是Redis副本可以正确地复制带expires的密钥，即使使用Lua脚本更改了这些密钥。



为了实现这样一个功能，Redis不能依赖于主机和副本拥有同步时钟的能力，因为这是一个无法解决的问题，会导致竞争条件和数据集发散，因此Redis使用三种主要技术来使过期密钥的复制能够工作：



副本不会使密钥过期，而是等待主服务器使密钥过期。



当主密钥过期（或由于LRU而逐出它）时，它将合成一个DEL命令，该命令将被传输到所有副本。




但是，由于主驱动过期，有时副本的内存中的密钥可能仍然在逻辑上过期，因为主服务器无法及时提供DEL命令。


此外，当轻轻关闭电源并重新启动时，副本能够在RDB文件中存储所需的信息，以便与主副本重新同步。这在升级时很有用。需要时，最好使用SHUTDOWN命令对副本执行save&quit操作。

# 持久化策略、

https://redis.io/topics/persistence


Redis持久性

Redis提供了一系列不同的持久性选项：

RDB持久性以指定的间隔执行数据集的时间点快照。

AOF持久性记录服务器接收到的每个写操作，这些操作将在服务器启动时再次播放，从而重建原始数据集。

命令使用与Redis协议本身相同的格式，以仅附加的方式记录。Redis可以在日志太大时在后台重写日志。

可以在同一个实例中同时使用AOF和RDB。注意，在这种情况下，当Redis重新启动时，AOF文件将用于重建原始数据集，因为它保证是最完整的。

最重要的是理解RDB和AOF持久性之间的不同权衡。

RDB优势

RDB是Redis数据的一种非常紧凑的单文件时间点表示。RDB文件非常适合备份。例如，您可能希望在最近的24小时内每小时存档一次RDB文件，并在30天内每天保存一个RDB快照。这允许您在发生灾难时轻松恢复数据集的不同版本。

RDB对于灾难恢复非常有用，它是一个可以传输到远数据中心或Amazon S3（可能是加密的）上的压缩文件。

RDB最大化ReRIS性能，因为ReDIS父进程需要做的唯一工作就是forking一个进程用来保存快照。父实例永远不会执行磁盘I/O或类似的操作。

与AOF相比，RDB允许使用大数据集更快地重新启。

RDB缺点

如果您需要在Redis停止工作时（例如断电后）将数据丢失的可能性降到最低，那么RDB就不太好。您可以在生成RDB的地方配置不同的保存点（例如，至少在对数据集进行5分钟和100次写入之后，但是您可以有多个保存点）。但是，您通常会每五分钟或更长时间创建一个RDB快照，因此如果Redis由于任何原因在没有正确关机的情况下停止工作，您应该准备好丢失最近几分钟的数据。

RDB通常需要fork（）才能使用子进程在磁盘上持久化。如果数据集很大，Fork（）可能会很费时，如果数据集很大，CPU性能不好，可能会导致Redis停止为客户机服务一毫秒甚至一秒钟。AOF还需要fork（），但您可以调整重写日志的频率，而不必在持久性上做任何权衡。

AOF优势

使用AOF Redis要持久得多：您可以有不同的fsync策略：根本没有fsync，每秒fsync，每次查询fsync。使用fsync的默认策略，每秒的写入性能仍然很好（fsync是使用后台线程执行的，当没有fsync进行时，主线程将努力执行写入操作）。但是您只能丢失一秒钟的写入操作。

AOF日志是一个只附加的日志，因此在断电的情况下不会出现查找或损坏问题。即使日志由于某种原因（磁盘已满或其他原因）以半写命令结束，redis check aof工具也能够轻松地修复它。

Redis可以在后台自动重写AOF。重写是完全安全的，因为当Redis继续追加到旧文件时，只需创建当前数据集所需的最小操作集，就会生成一个完全新的文件，一旦第二个文件准备好，Redis就会切换这两个文件并开始追加到新文件。

AOF缺点

AOF文件通常比相同数据集的等效RDB文件大。

AOF可能比RDB慢，具体取决于fsync策略。一般来说，fsync设置为每秒一次时，性能仍然非常高，如果fsync被禁用，那么即使在高负载下，它也应该和RDB一样快。RDB仍然能够提供最大延迟的更多保证，即使在巨大的写入负载的情况下也是如此。


一般情况下，如果希望获得与PostgreSQL相当的数据安全性，应该同时使用这两种持久性方法。



如果您非常关心自己的数据，但在发生灾难时仍然可以忍受几分钟的数据丢失，那么您只需使用RDB即可。



有许多用户单独使用aof，但我们不鼓励这样做，因为不时地使用RDB快照是进行数据库备份、更快地重新启动以及在AOF引擎出现错误时的一个好主意。



快照

默认情况下，Redis将数据集的快照保存在磁盘上名为dump.rdb的二进制文件中。如果数据集中至少有M个更改，可以将Redis配置为每隔N秒保存一次数据集，也可以手动调用save或BGSAVE命令。



例如，如果至少更改了1000个密钥，此配置将使Redis每隔60秒自动将数据集转储到磁盘：


工作原理

每当Redis需要将数据集转储到磁盘时，都会发生以下情况：


Redis forks。我们现在有一个子进程和一个父进程。



子进程开始将数据集写入临时RDB文件。



当子进程完成新的RDB文件的编写时，它将替换旧的RDB文件。



这个方法允许Redis从copy-on-write语义中获益。


仅追加文件

如果运行Redis的计算机停止运行，电源线出现故障，或者意外地kill -9实例，那么在Redis上写入的最新数据将丢失。


append only文件是Redis的一种替代的、完全持久的策略。


您可以在配置文件中打开AOF：

appendonly yes
从现在起，每当Redis收到一个更改数据集（例如SET）的命令时，它就会将其附加到AOF中。重新启动Redis时，它将重新播放AOF以重建.

日志重写

如您所料，AOF随着写操作的执行而变得越来越大。例如，如果您将一个计数器递增100次，那么您的数据集中只会有一个键包含最终值，而AOF中只有100个条目。其中99个条目不需要重建当前状态。



所以Redis支持一个有趣的特性：它能够在后台重建AOF，而不会中断对客户端的服务。每当您发出BGREWRITEAOF Redis时，它都会写入重建内存中当前数据集所需的最短命令序列。如果将AOF与Redis 2.2一起使用，则需要不时运行BGREWRITEAOF。Redis2.4能够自动触发日志重写（更多信息请参见2.4示例配置文件）。



仅追加文件的持久性如何？

您可以配置Redis在磁盘上同步数据的次数。有三种选择：



appendfsync always:fsync每次向AOF追加新命令时。非常慢，非常安全。

每秒钟一次。足够快（在2.4中可能和快照一样快），如果发生灾难，您可能会丢失1秒的数据。

appendfsync否：永远不要fsync，把你的数据放在操作系统的手里。更快更不安全的方法。通常Linux会用这种配置每隔30秒刷新一次数据，但这取决于内核的精确调整。

建议（和默认）策略是每秒fsync一次。它既快又安全。always策略在实践中非常慢，但它支持组提交，因此如果有多个并行写操作，Redis将尝试执行一个fsync操作。


如果我的AOF被截断，我该怎么办？

可能是服务器在写入AOF文件时崩溃，或者是存储AOF文件的卷在写入时已满。



当这种情况发生时，aof仍然包含表示给定时间点版本的数据集的一致数据（使用默认AOF fsync策略，该数据集可能最长为1秒），但AOF中的最后一个命令可能会被截断。


最新的主要版本的Redis无论如何都可以加载AOF，只需放弃文件中最后一个格式不正确的命令。


注意监控你的机器磁盘空间。




# 当RAM空间占满时，清理出可用空间的策略

下面引用Redis常见问题解答中的一句话：“如果Redis内存不足会发生什么？”章节：

... [您]可以使用配置文件中的“maxmemory”选项限制Redis可以使用的内存。如果达到此限制，Redis将开始回复写入命令的错误（但将继续接受只读命令），或者在使用Redis进行缓存的情况下，可以将其配置为在达到最大内存限制时收回密钥。

Redis Cloud的固定大小计划将其maxmemory设置为计划的大小。您可以从帐户的控制台轻松配置实例的逐出策略（称为maxmemory策略），并将其设置为任何标准的Redis行为，而不会中断服务。以下列表详细列出了Redis的逐出策略：

allkeys-lru: the service evicts the least recently used keys out of all keys
allkeys-lfu: the service evicts the least frequently used keys out of all keys
allkeys-random: the service randomly evicts keys out of all keys
volatile-lru: the service evicts the least recently used keys out of all keys with an "expire" field set
volatile-ttl: the service evicts the shortest time to live keys (out of all keys with an "expire" field set)
volatile-lfu: the service evicts the least frequently used keys out of all keys with an "expire" field set
volatile-random: the service randomly evicts keys with an "expire" field set
no-eviction: the service will not evict any keys and no writes will be possible until more memory is freed

all keys lru：服务从所有密钥中逐出最近使用最少的密钥

all keys lfu：服务从所有密钥中逐出最不常用的密钥

all keys random：服务从所有密钥中随机取出密钥

volatile lru：该服务从所有设置了“expire”字段的密钥中逐出最近使用最少的密钥
volatile lfu：服务从所有设置了“expire”字段的键中逐出最不常用的键

volatile random：服务随机收回设置了“expire”字段的密钥

volatile ttl：服务逐出生存时间最短的密钥（在所有设置了“expire”字段的密钥中）

no eviction：服务不会逐出任何密钥，并且在释放更多内存之前无法进行写入




# 高可用

# 可扩展


# 单机Redis运行模式，为什么单线程执行效率确很高









