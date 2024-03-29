主从复制，读写分离！ 80% 对情况下都是在读操作！ 减缓服务器的压力！ 架构中经常使用！

一主二从 ，哨兵模式

**数据的复制是单向的，只能有主节点到从节点。Master以写为主，Slave以读为主**

**主从复制到作用主要包括：**

1. **数据冗余：主从复制实现了数据的热备份，是持久化的一种数据冗余方式。**
2. **故障恢复：当主节点出现问题时，可以由从节点提供服务，实现快速的故障恢复；实际上是一种服务的冗余。**
3. **负载均衡：在当主从复制的基础上，配合读写分离，可以由主节点提供写服务，由从节点提供读服务（即写Redis数据时应用连接主节点，读Redis数据时应用连接从节点）分担服务器负载；尤其是在写少读多的场景下，通过多个从节点分担读负载，可以大大提高Redis服务器的并发量。**
4. **高可用(集群)基石：除了上述作用以外，主从复制还是哨兵和集群能够实施的基础，因此说主从复制时Redis高可用的基础。**

**环境配置**

**只配置从库。无需配置主库**

```shell
127.0.0.1:6379> info replication #查看当前库的信息
# Replication
role:master #角色 master
connected_slaves:0 #没有从机
master_replid:fe434df8e7e85587e4682aa397000200ec8e8cfe
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:0
second_repl_offset:-1
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0
```



复制 3个配置文件，然后修改对应的信息

1.  端口
2.  pid名字
3.  log文件名字
4.  dump.rdb 名字

```shell
#redis.conf 永久配置
replicaof <masterip> <masterport>

#命令临时配置
slaveof 127.0.0.1 6379 


#当主节点断开以后，从节点设置为主节点. 此时主节点回来，只能重新配置。
SLAVEOF no one
```





测试：主机断开连接，从机依旧连接到主机的，但是没有写操作了。这个时候，主机如果回来了，从机依旧可以直接获取主机写的信息！

如果是使用命令行，来配置的主从，这个时候如果重启了。就会变回主机！只要变为从机，立马就会从主机中获取值！

Slave 启动成功连接到 Master 后会发送一个 sync 同步命令

Master 接到命令，启动后台的存盘进程，同时收集所有接收到的用于修改数据集命令，在后台进程执行完毕之后，

**Master将传送整个数据文件到Slave,并完成一次完全同步**。

- **全量复制**：而Slave服务载接收到数据库文件数据库后，将其存盘并加载到内存中。
- **增量复制**：Master继续将新的所有收集到的修改命令依次传给Slave，完成同步。

但是只要是重新连接Master ，一次完全同步（全量复制）将被自动执行。 我们的数据一定可以在从机中看到！

**哨兵模式(自动选举老大) Sentinel**

**哨兵通过发送命令，等待Redis 响应，从而监控运行的多个Redis实例。**

```shell
# 1. 配置哨兵配置文件 sentinel.conf
#                被监控的名称 host  port  代表主机挂了。 从机投票看谁接替成为主机。票数最多的，就会成为主机！
sentinel monitor myredis 127.0.0.1 6379 1


# 2. 启动哨兵
redis-sentinel sentinel.conf

```

