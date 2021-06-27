# Paxos Made Simple

**The Paxos algorithm, when presented in plain English, is very simple.**

## 1. Introduction

简单总结一下第一段: paxos算法很简单，看不懂是你们的问题

## 2. The Consensus Algorithm

### 2.1 The Problem

&emsp;&emsp;假设一些可以提议的进程，一个共识算法确保只有一个值最终被选定。如果没有提议，就不能选定任何值。如果一个值被选定了，所有进程都要能够学习到这个值。共识的安全要求如下

- 只有一个提议的值可能被选中
- 只有一个值能被选中
- 一个进程只有在一个值被选中后才能知道它被选中了

&emsp;&emsp;共识算法中由三个角色：proposer, acceptor, learner。在实现中，一个进程可能扮演多个角色。假设代理之间可以通过发送消息来互相通信

- 代理以任意速度操作，fail-stop，而且可能重启。由于代理可能在值被选中之后崩溃并重启，因此代理必须能够记住一些信息
- 信息传输可能要随机时间，可以有重复，可能会丢失，但是不会出错

### 2.2 Choosing a Value

&emsp;&emsp;最简单的选值的方式是只有一个acceptor，proposer将提议发送给acceptor，acceptor选中第一个收到的值。如果有多个acceptor，首先要保证每个acceptor最多接受一个值，并且需要多数接受该值

**P1: acceptor必须接受它收到的第一个值**

&emsp;&emsp;这带来了一个问题，多个proposer可能提出多个值，这导致最终没有一个达到多数要求。通过给每个提议分配一个数字来追踪acceptor接受的不同提议，因此每个提议都有一个proposal number和值。需要每个提议的number不同。可以允许多个提议被选择，但是必须保证选中的提议都有相同的值

**P2: 如果一个提议值v被选中，那么所有number更高的提议都有v**

**P2_a: 如果一个带有v的提议被选中，那么所有被接受的number更高的提议的值都是v**

**P2_b: 如果一个带有v的提议被选中，那么所有proposer提出的值都是v**

&emsp;&emsp;假设一些number为m值为v的提议被选中，那么需要证明所有number n>m的提议都提出值v。对于被接受的m，存在某个集合C包含多数的acceptor接受该提议，m被接受的假设意味着

- 每个在C中的acceptor都接受了m..(n-1)，而且每个在m..(n-1)之间的提议提出值都是v

由于每个acceptor的多数子集都和C有交集我们可以通过以下规则保证n的值也是v

**P2_c: 对于任何一个v和n，如果一个值为v序号为n的提议被发部，那么存在一个多数集合S保证S中的acceptor要不接受了小于n的提议，要不v就是被接受的小于n中最高序号的提议**

&emsp;&emsp;为了位置P2_c的不变，一个想要提议的proposer必须学习到被多数acceptor接受序号小于n的最大序号的提议。学习到当前被接收到的值比较容易，预测未来更难。proposer通过通过一个未来不会有任何接受的承诺进行控制。换句话说，proposer需要所有的acceptor不再接受序列号小于n的提议。算法过程如下

1. 一个proposer选择一个序列号n，并且把请求发送给一些acceptor，请求它返回

   (a) 承诺不再接受序列号更小的提议

   (b) 序列号大于n的提议已经被接受

   这个过程为prepare(n)

2. 如果proposer收到了多数acceptor的响应，它就发布一个序列号为n以及值为v的提议，其中v是所有响应中序列号最高的提议的值

   这个过程成为accept

&emsp;&emsp;对于acceptor，它能接受两种请求，prepare和accept。一个acceptor可以在不影响安全性的情况下忽略任何请求。因此我们必须规定acceptor响应的条件，对于accept，需要满足以下条件

**P1_a：只有在没有响应过更高序列号的情况下可以接受序列号n**

下面是一些优化

&emsp;&emsp;假设一个acceptor收到了序列号为n的prepare请求，但是它已经响应了大于n的请求，因此不会响应n。通过这个优化，一个acceptor只需要记住它收到的最高提议，因为P2_c即使崩溃了也要保证，一个acceptor必须记住这些信息，一个proposer可以丢弃任何提议。因此，算法被划分为两个阶段

- Phase 1
  - proposer选择一个序列号n并给所属acceptor发送序列号
  - 如果一个acceptor收到了prepare消息，序列号为n，而且n是最大序列号，那么他就保证不会接受任何小于n的提议
- Phase 2
  - 如果proposer收到了多数响应，那么他就会发送提议 序列号为n,值为v，
  - 只有在acceptor曾经响应过n时才会接受

优化点：如果一个acceptor忽略了prepare或者accept请求，那么它应该告知proposer

### 2.3 Learning a Chosen Value

&emsp;&emsp;为了学习一个已经被选择的值，learner必须找到被多数接受的值。acceptor需要告知一些特殊的learner它的接受提议。如果一个learner想知道被选择的值，需要一个proposer进行提议

### 2.4 Progress

distinguished proposer

### 2.5 The Implementation

## 3.  Implementing a State Machine

