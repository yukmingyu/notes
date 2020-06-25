# 在Windows下配置多个Redis(简化配置)

如果在单机上安装了多个 Redis，修改配置的时候，需要一个一个的配置文件去修改，很麻烦。

肯定有简单的办法，本文就是咯~~~

### 实现步骤

在 Redis 的安装根目录，新建一个统一的配置文件，例如：`redis.cluster.conf`。

在各个 Redis 的配置文件中，提取共同的配置项，写入上面建立的统一配置文件。

在各个 Redis 的配置文件中，只保留该 Redis 独有的配置项，**同时，添加 `include` 配置项**，引用统一的配置内容即可。



### 详细说明

- 比如，在上一篇中，Redis 安装在 `C:\Program Files\\` 下面，所以，就在 `C:\Program Files\Redis\\` 路径下，建立 `redis.cluster.conf` 文件。

- 将各个 Redis 的配置文件中的相同项全部提取出来，写入上面的文件中，如下所示：

```
# // redis.cluster.conf 全部内容
appendonly yes

maxmemory 200mb
maxmemory-policy allkeys-lru

cluster-enabled yes
cluster-node-timeout 15000
cluster-slave-validity-factor 10
cluster-migration-barrier 1
cluster-require-full-coverage yes
```

- 将单个 Redis 的配置文件按照如下方式修改（以 7100 为例）：

```
# // redis.7100.conf 全部内容
port 7100

appendfilename "appendonly.7100.aof"

cluster-config-file nodes-7100.conf

include redis.cluster.conf
```

*其它 Redis 的配置文件，如上所示。*

至此，统一的配置文件就建立完毕了，逐个重启 Redis 服务，使其生效即可。

- 如果需要修改全部 Redis 的配置，只需要修改 `redis.cluster.conf` 这个文件即可。
- 如果需要修改单个 Redis 的配置，则在修改单个 `conf` 文件即可。

*修改完毕，记得逐个重启 Redis 服务，使其生效哟。*

**特别说明**：关于统一配置文件的位置，如果没有将其放在 Redis 的安装根目录的话，记得在 `include` 后面添加