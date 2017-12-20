## Paxos

Paxos分成很多轮（Round），每一轮都对应一个ID，每一轮都有分成若干阶段（Phase）。

首先，某个Proposer提出了Proposal，会选择一个N，大于之前所有见过的最大的N；然后这个Proposer会群发Proposal+N给所有的Acceptor。

当Acceptor收到一个Proposal+N后，若N小于之前看过的某个N，则reject；否则就accept，并发送消息给对应的Proposer，消息内容是<promise-OK, Na, Va>，其中Na表示之前看到的最大的N（当然现在这个值已经改成刚刚收到的N了），Va表示之前已经promise的Value，如果之前没有promise，这个Va就是null。

Proposer收到来自大部分的Acceptor的promise-OK消息，则群发所有人<accept, N, V>，其中V或者是之前收到的V，或者是新建的V（如果收到的V是null）。如果收到的消息不够，则delay并restart。（如果一共5个Acceptor，收到3个null，2个Va，那么Proposer还是可以随意定义一个V的值。另外要注意的是，N和V并没有对应关系，round和V有对应关系；因此每一轮都会有多个N，但V只有一个；同时也可以解释为什么Acceptor收到一个N后，如果是最大的，会把之前的V发回去）

Acceptor此时收到<accept, N, V>的消息，若V和之前记下的V一样（若null则不需要检查），返回 <accept-OK>，记录下N-V对应关系，并更新自己的最大N；否则返回<accept-reject>。`什么时候`

Leader收到了大部分Acceptor的<accept, N, V>的消息，则群发<decide, V>；如果没有收到足够的消息，则delay并restart。

总结一下：某个proposer提出N，为了确定必须尽快发给所有acceptor，并期望尽可能多的acceptor返回promise-OK。当>50%的acceptor在log里写了promise-OK之后，实际上就已经commit了。

一个游戏：建一个微信群，选择若干人propose，向群里发消息：N。

**几种场景**

V = non-empty value (highest Na received)的解释：

问：对于Leader来说，收到的promise数量大于majority，就可以发起accept，而不用关心promise的V是否完全一样。
答：正确。如果Leader收到的promise中含有Na，那么一定是因为之前有某个Leader（X）发送了<accept, Na, Va>，而这么做表示X已经收到了Majority的promise(Na)。

问：对于acceptor来说，Promise某个N之后是否可以改？
答：可以。promise本身并没有accept V，所以当promise N=10，收到accept N=11，也会accept。

问：对于acceptor来说， Accept之后是否可以改？
答：可以。比如之前accept (10, A)，然后收到<accept, 11, B>的消息，就会变成accept(11, B)，这是因为如果收到<accept, 11, B>，意味着已经有majority的acceptor promise了11。

一种场景：3个节点XYZ，XY promise(10, A)，然后发送<accept, 10, A>，只被节点X accept；

X    P(10)   A(10,A)
Y    P(10)                    P(11) 
Z                                 P(11)    A(11,B)

此时X和Z都已经accept。Y可以propose 12，然后只发给A，A会返回<promise-OK, 10, A>，Y就收到了两个promise-OK，选择高的值（12,A）来完成。


**其他参考**

- [Wikipedia: Paxos算法](https://zh.wikipedia.org/wiki/Paxos%E7%AE%97%E6%B3%95)
- [Raft Consensus Algorithm](https://raft.github.io/)
- [知乎：如何浅显易懂地解说 Paxos 的算法？](https://www.zhihu.com/question/19787937/answer/82340987)
