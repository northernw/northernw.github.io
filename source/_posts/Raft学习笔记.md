---
title: Raft学习笔记
tags:
  - null
categories:
  - null
date: 2020-09-23 15:32:53
typora-copy-images-to: ../../image
---

https://ongardie.net/static/raft/userstudy/



保证：

1. 一个任期内只有一个leader



三个状态：

leader

follower

candidate

<img src="/github/northernw.github.io/image/image-20200923153700539.png" alt="image-20200923153700539" style="zoom:50%;" />



选举：

1. 任期加一
2. 切换为candidate状态
3. 为自己投票
4. 向其他所有节点发出投票请求（RequestVote RPCs），重试
   1. 从大多数节点收到投票
      1. 成为leader
      2. 向其他所有节点发送AppendEntries心跳
   2. 收到合法leader的RPC（=AppendEntries）
      1. 回退到follower状态
   3. 没有节点赢得选举（选举超时结束后）：【比如有多个节点进入candidate状态，选票分裂了，没有哪个节点收到大多数majority投票】
      1. 任期加一，开始新一轮选举



cont'd: continued的缩写，连接上文，续上

![image-20200924111446553](/github/northernw.github.io/image/image-20200924111446553.png)

性质

安全性：每个任期允许至多一个leader

1. 每个节点每个任期内只会发出一个投票（磁盘持久化）
2. 同一任期内，不同candidate不会累加选票（可以理解为，要么不存在大多数，要么至多有一个大多数，不会有两个节点都获得大多数选票）

活性：终有一个candidate会胜出

1. 随机选择选举超时`[T,2T]`
2. 一个节点通常在其他节点醒来前超时并赢得选举
3. 在`T >> broadcast time`时非常有效





normal operation 常规操作，正常运行

1. 客户端向leader发送指令command
2. leader将指令追加到日志log
3. leader向followers发送AppendEntries RPCs
4. 一旦新entry提交了committed：
   1. leader将指令发给状态机，向客户端返回结果
   2. leader在随后的AppendEntries RPCs中通知followers提交entry
   3. followers将指令发给各自的状态机
5. 对于宕机或者运行慢的followers：
   1. leader一直重试RPCs直到成功
   2. 并不影响leader对客户端的响应，不是必须等待
6. 通常情况下，表现理想
   1. 一次RPC能触及大多数节点





Log Consistency 日志一致性

日志间的高度一致性：

1. 如果不同节点的日志entry有相同的index和term，那么：
   1. 它们存储相同的指令
   2. 在这之前的日志项entry也都相同
2. 如果一个entry是提交的，那么在它之前的entry也是提交的



AppendEntries一致性检查

每个AppendEntries RPC包含前一个entry的index和term

follower包含前一个entry（index和term相同），才接受请求，否则拒绝这个请求



Safety Requirement 安全需求

已提交->保证majority大多数的节点上有最大的index，即使重新选举，那些缺失最大index的节点会选举失败->保证log出现在后续的leader节点上



Election Rules

如果投票节点的日志比candidate的"完整"，则拒绝candidate的选票

主要看term和index 任期和索引

> V：投票节点
>
> C：candidate 候选节点
>
> (lastTerm V > lastTerm C) ||
>  (lastTerm V == lastTerm C) && (lastIndex V > lastIndex C)



Commitment Rules

![image-20200924174604257](/github/northernw.github.io/image/image-20200924174604257.png)

leader判断一个entry是否已提交，需要考虑

1. entry必须存在大多数节点上
2. 至少有一个当前leader任期的新entry也存在大多数节点上

<u>第二点不是特别理解..</u>





Client Protocol

1. 将指令发送给leader
2. 直到指令追加进日志、已提交、并且被leader的状态机执行，leader才会响应客户端
3. 如果请求超时（比如leader宕机）：
   1. Client（随机）重发指令给其他节点
   2. 被定向到一个新leader
   3. 向新leader重试请求
4. 如果leader在执行完指令、响应之前宕机了，为了不执行同一条指令两次，解决方案：
   1. Client的指令请求里包含unique id
   2. 节点的log里存储每个请求的id
   3. leader在接收指令前，检查日志中是否已存在指令中的ID
   4. 如果存在，忽略新指令，直接返回旧指令对应的响应
   5. 达到exactly-once semantics 仅执行一次语义，线性一致性