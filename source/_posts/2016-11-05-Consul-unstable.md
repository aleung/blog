---
layout: blog
title: Consul 集群不稳定的分析
date: 2016-11-05 11:22:20
tags:
- SoftwareDev
---

一个 Consul 集群由3个 Consul server 和近百个 Consul client 组成。观察发现集群状态不稳定，频繁出现以下现象：

1. 节点退出集群，又重新加入集群；
2. 重新选举 leader，有时候原来的 leader 会重新当选。

<!-- more -->

对于第一个问题，分析发现原因是我们错误使用了 fail fast 策略：我们平台中会有个supervisor daemon监控着进程的状态，当进程 health check 失败了，就会重启进程并且发送告警。对于 Consul agent 进程，health check 调用 Consul API，API不能正常返回就认为进程状态不健康。这样做本来的用意是及时发现集群配置的错误，另外也是服务依赖管理的需要，依赖 Consul 的其它服务会等到 Consul agent 进入正常状态后才启动。但是，由于第二个问题的存在，在 leader 重新选举过程中 API 不能正常响应，Consul agent 就被当作不健康而被重启，出现了 Consul 节点退出重新加入集群的现象。本来，群集是能自动恢复到正常工作状态的，重启进程反而造成更大的负面影响。

对于第二个问题，直接原因是某个 Consul server 所在的VM上有个异常的进程将CPU几乎全耗尽了，Consul 不能及时的处理集群间的通信，响应超时，产生了临时性的脑裂。但开始我们一直不能理解，单个 server 的 partitioning 为什么会影响到整个集群呢？剩余的 2 个 server 应该还是能够组成集群并保持稳定的 leader 才对。后来才发现，比起全部丢失，消息的部分丢失所引起的现象会诡异得多。

这是当前 leader cm121 的日志：

```log
50:32 [WARN] raft: Rejecting vote from 172.17.52.84:8300 since we have a leader: 172.17.52.121:8300
50:32 [ERR] raft: peer 172.17.52.84:8300 has newer term, stopping replication
50:32 [INFO] raft: aborting pipeline replication to peer 172.17.52.84:8300
50:32 [INFO] raft: aborting pipeline replication to peer 172.17.52.85:8300
50:32 [INFO] consul: cluster leadership lost
50:32 [INFO] raft: Node at 172.17.52.121:8300 [Follower] entering Follower state
50:33 [WARN] raft: Heartbeat timeout reached, starting election
50:33 [INFO] raft: Node at 172.17.52.121:8300 [Candidate] entering Candidate state
50:34 [INFO] raft: Election won. Tally: 2
50:34 [INFO] raft: Node at 172.17.52.121:8300 [Leader] entering Leader state
50:34 [INFO] consul: cluster leadership acquired
50:34 [INFO] consul: New leader elected: cm121
```

在这个例子中，CPU耗尽的 server 是 cm84，当前 leader 为 cm121。cm121 会定期发出 AppendEntries RPC，但 cm84 因为没有CPU资源没有处理到，等到 heartbeat timeout，cm84 错误认为当前没有 leader 了，就向所有其它 server 发出 RequestVote RPC（带有更大的 term 值），自荐为候选人，发起一轮新的投票。但是这个投票请求被其它 server 都拒绝了。

{%img /attachments/2016/11/raft_state.png %}

根据状态图可知，当 leader 收到其它 server 发来的消息有更大的 team 值时，就需要交出 leader 位置，转为 follower。因此被 cm84 一捣乱，集群里面就没有 leader 了，导致接下来的超时重新选举。在新一轮选举中，cm121得到两票，重新当选为 leader。

将消息的时间线画出来，其中 S1 为当前 leader (cm121)，S3 为会间歇性丢掉消息的 server (cm84)，可见 S3 如何导致重新选举 leader。

{%img /attachments/2016/11/timeline.jpg %}

对于 Consul 中 raft 实现的行为：reject vote since we have a leader，似乎跟 raft 论文描述的有些出入。在 Raft [论文](https://raft.github.io/raft.pdf)中 RequestVote RPC receiver implementation 部分，没有提到已有 leader 时需要 reject 新的 vote。不过，即使没有这个检查，cm84 大部分情况下还是不能满足当选条件，因为它错失了 leader 发来的 AppendEntries RPC，它的 log 就不一定是 up-to-date 的（除非错失的 AppendEntries 没有 log entries，只是heartbeat）。cm84 不能当选，cm121被迫退位，群集还是会进入无 leader 状态。

> **RequestVote RPC**
>
> Invoked by candidates to gather votes (§5.2).
>
>
> Arguments:
>
> - term : candidate’s term
> - candidateId : candidate requesting vote
> - lastLogIndex : index of candidate’s last log entry (§5.4)
> - lastLogTerm : term of candidate’s last log entry (§5.4)
>
>
>
> Results:
>
> - term : currentTerm, for candidate to update itself
> - voteGranted : true means candidate received vote
>
>
>
> Receiver implementation:
>
> 1. Reply false if term < currentTerm (§5.1)
> 2. If votedFor is null or candidateId, and candidate’s log is at
>    least as up-to-date as receiver’s log, grant vote (§5.2, §5.4)

下一步要做的是避免 Consul 进程得不到足够的CPU资源，计划尝试修改 Consul 进程的 nice 来提高调度优先级，或者配置 CPU affinity 绑定一个CPU核给 Consul。

另外，在分析过程中也进一步加深了对 Consul 的理解。

Consul agent 分为两种：server 和 client。实际上，这里是两个不同层次的群集（当涉及到多个 datacenter 时，还多一个 WAN layer，这里就不讨论了）。

{%img /attachments/2016/11/consul_cluster.jpg %}

所有 server 通过 Raft 协议组成一个集群，是一个 CP 系统。Consul 的 catelog 和 KV store 服务都是在这个 raft 集群中提供的，虽然 API 在 client 上也提供，但 client 只是转发给 server 来处理。集群中只会选出最多一个 leader，能容忍 partition 的发生（脑裂）。当存在 pratitioning 时，leader 在有多数 server 的分区里，只有少数 server 的分区选不出 leader。数据写入都由 leader 处理，保证了强一致性。 数据读取根据一致性要求（consistency mode），可以由 leader 或者 follower 处理。在没有选出 leader 的时候，Consul 的大部分 API 会不可用（consistency mode 为 stale 的读请求应该可以处理，需要测试确认一下）。

所有 consul agent，包括 server 和 client，组成一个更大的集群，就是平常所说的 consul cluster。这个集群是通过 Serf 协议（一种 gossip 协议）组成的，是一个AP系统，用于 consul 节点的发现和管理。AP系统只能保证最终一致性，因此 consul 节点的加入或者离开，是不能立刻被整个群集所见到的。

