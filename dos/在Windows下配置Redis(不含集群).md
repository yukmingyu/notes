# 在Windows下配置Redis(不含集群)

### 简单介绍

Redis是一款开源的、高性能的键 - 值存储（Key - Value store）。和 Memcached 类似，Redis 常被称作是一款 Key - Value 内存存储系统（或内存数据库）。

同时，由于它支持丰富的数据结构，又被称为一种数据结构服务器（Data Structure Server）。

但是，**Redis 官方并不支持 Windows**，[微软开源团队](https://link.jianshu.com?t=https://github.com/MSOpenTech) 将其移植到了 Windows 中。

*在实际生产环境中，还是建议将 Redis 部署在 CentOS 中来进行，毕竟 Redis 全部是通过 API 进行操作，跨平台访问是天生的。*

> - 配置环境：Windows 7 Pro x64

- [Redis 官网](https://link.jianshu.com?t=https://redis.io/)
- [Redis Windows 版](https://link.jianshu.com?t=https://github.com/MSOpenTech/redis/releases)

### Redis 安装

安装很简单，直接在上面的 Windows 版中下载 msi，双击运行即可安装（这里的版本是 v3.2.100）。

![image-20200625230749463](在Windows下配置Redis(不含集群).assets/image-20200625230749463.png)

*稍微说明一下：在安装过程中，最好把上图的这个选项勾上，方便在命令行中随时执行相应的指令。*

安装好之后，会这系统服务里面安装一个 Redis 的节点服务，并且已经启动了。

接下来，就可以进行操作了。

### 常规操作

登录到 Redis 服务器，在命令行中执行以下命令*（也可以直接双击安装目录下的 redis-cli.exe 登录到默认端口的 Redis 服务器）*：

```
// -h：主机 IP
// -p：主机端口
// 如果要登录集群节点，还需添加一个参数：-c
redis-cli -h 127.0.0.1 -p 6379
```

*由于在安装的时候，已经将 Redis 的安装路径注册到系统 PATH 了，所以，在命令行的随便哪个位置，都可以执行 Redis 的指令。*

看到如下界面，就表示登录成功了：

![image-20200625230823474](在Windows下配置Redis(不含集群).assets/image-20200625230823474.png)

登录成功

Redis 安装好以后，可通过常用的几个指令进行查看和操作：

- `info`（查看 Redis 服务器的全部信息）
- `keys *`（查看当前存储的所有 key）
- `get <keyname>`（查看指定 key 的内容）
- `del <keyname>`（删除指定 key 的内容）
- `set <keyname> <value>`（设置指定 key 的内容）
- `flushdb`（删除 **当前数据库** 中的所有 key）
- `flushall`（删除 **所有数据库** 中的所有 key）

> `info` 指令：查看 Redis 服务器的全部信息

![img](在Windows下配置Redis(不含集群).assets/3116004-c01ae8d50953e7a7.png)

Redis 整体信息

`info` 后面还可以跟具体某个部分的信息，只看这一项的具体内容，就是上图中，井号 “#” 后面的内容，比如：

```
info server   // 查看服务器基本信息
info memory   // 查看内存使用情况
// ……
```

*关于 `info` 指令的详细说明，[参见这里（英文）](https://redis.io/commands/info)。*

> 关于 `info memory` 指令的结果分析

这个指令主要是查看服务器内存使用情况的，在命令行执行之后，如下所示：

![img](在Windows下配置Redis(不含集群).assets/3116004-a2e13d5610b843c3.png)

该指令主要显示了当前内存的占用情况、碎片情况，还有就是内存可占用的最大空间以及占用达到最大空间后的处理策略。

> 先把这些指标简单分为两大类：

1. 末尾没有带 “_human”：即该节点的实际数值
2. 末尾带有 “_human”：即便于人类理解的数值（自动把实际数值按 1024 转换而来）

内存相关的指标说明：

> - **used_memory：**由 Redis 分配器分配的内存总量，以字节（byte）为单位。

- **used_memory_rss：**从操作系统的角度，返回 Redis 已分配的内存总量（俗称：常驻集大小。rss 是 Resident Set Size 的意思）。这个值和 top、ps 等命令的输出一致。
- **used_memory_peak：**Redis 的内存消耗峰值，以字节（byte）为单位。
- **total_system_memory：**Redis 所在的服务器实际内存大小，以字节（byte）为单位。
- **used_memory_lua：**Lua 引擎所使用的内存大小，以字节（byte）为单位。
- **maxmemory：**Redis 最大可用的内存空间，以字节（byte）为单位。
- **maxmemory_policy：**当数据占满最大可用空间时，需要执行的过期策略（见后：过期策略种类解释）
- **mem_allocator：**在编译时指定的，Redis 所使用的内存分配器。可以是 libc、jemalloc 或 tcmalloc。
- **mem_fragmentation_ratio：**used_memory_rss 和 used_memory 之间的比率（详见：特别说明）。

![img](在Windows下配置Redis(不含集群).assets/gif.latex)

> **特别说明：**

- 在理想情况下，**used_memory_rss 的值应该只比 used_memory 稍微高一点（比率不超过 1.5）。**
- 当 rss > used 时，且两者的值相差较大时，表示存在（内部或外部的）内存碎片（内存碎片的比率可以通过 mem_fragmentation_ratio 的值看出）。
- 当 used > rss 时，表示 Redis 的部分内存被操作系统换出到交换空间内了。在这种情况下，操作可能会产生明显的延迟。

> *当 Redis 释放内存时，分配器会（也可能不会）将内存返还给操作系统。
>  如果 Redis 释放了内存，却没有将内存返还给操作系统，那么 used_memory 的值就可能和操作系统显示的 Redis 内存占用的值不一致。
>  查看 used_memory_peak 的值，即可以验证这种情况是否发生。*

> *内存碎片是怎么产生的呢？简单来说：Redis 需要一个连续的内存空间来存储 1G 的数据，但是当前内存已经没有 1G 连续的空间了，这样，操作系统不得不将多个不连续的空间组合起来提供 1G 的大小，这就导致了内存碎片的产生。*

> *mem_fragmentation_ratio 稍大于 1 是合理的。这个值表示内存碎片率比较低，也说明 Redis 没有发生内存交换。

- 如果该值超过 1.5，那就说明 Redis 消耗了实际需要物理内存的150%，其中 50% 是内存碎片。
- 如果该值低于 1 的话（但 0.95 以上都算正常），说明Redis内存分配超出了物理内存，操作系统正在进行内存交换。而内存交换会引起非常明显的响应延迟。*

> 一般来说，碰到这样的情况，建议从三个方面进行优化：

- **重启 Redis 服务器**：记得在重启之前，用 redis-cli 工具先执行 shutdown save 指令，将数据持久化到硬盘上，待重启之后，再加载回内存，避免数据丢失。
- 限制内存交换
- 修改内存分配器

> *更多优化方法，可[参考这里](http://www.cnblogs.com/mushroom/p/4738170.html)*

在运行过程中

- 获取配置的指令：`config get "<info 中的 key>"`
- 设置配置的指令：`config set "<info 中的 key>" "<值>"`

比如：在 Redis 运行中，设置最大可用内存空间，输入以下命令：

```
config set "maxmemory" "300mb"
```

一般来说，如果设置了具体的最大可用内存空间，一定要关注过期策略：`maxmemory-policy`。

#### 过期策略种类解释（后有名词解释）：

> - **volatile-lru：**对 “过期集合” 中的数据采取 LRU 算法。将（已经过期 ÷ LRU）的数据优先移除。如果 “过期集合” 中的数据全部移除了，仍不能满足内存需求，则将 OOM。

- **allkeys-lru：**对所有的数据，采用 LRU 算法。
- **volatile-random：**对 “过期集合” 中的数据采取 “随机选取” 算法，并移除选中的 K - V，直到 “内存足够” 为止。如果 “过期集合” 中的数据全部移除了，仍不能满足内存需求，将 OOM。
- **allkeys-random：**对所有的数据，采取 “随机选取” 算法，并移除选中的 K - V，直到 “内存足够” 为止。
- **volatile-ttl：**对 “过期集合” 中的数据采取 TTL 算法，移除即将过期的数据。
- **noeviction：**不做任何干扰操作，直接返回 OOM 异常。

> *说明：*
>  *1. 过期集合：如果对 key 使用 `expire` 指令指定了过期时间，那么，此 key 将会被添加到 “过期集合” 中。*
>  *2. LRU：Least Recently Used，近期最少使用，[详见百科](https://link.jianshu.com?t=http://baike.baidu.com/item/LRU)*
>  *3. TTL：Time To Live，最小存活时间，[详见百科](https://link.jianshu.com?t=http://baike.baidu.com/item/TTL/130248)*
>  *4. OOM：Out Of Memory，Killer 会干掉 Redis 进程，俗称：死机。*
>  *5. [以上全部内容的官方说明（英文）](https://link.jianshu.com?t=https://redis.io/topics/lru-cache)*

至此，在 Windows 下配置 Redis 就差不多结束了。

> - 如需配置 Redis 集群，[请看这里](https://yukmingyu.github.io/notes/#/dos/在Windows下配置Redis集群.md)

