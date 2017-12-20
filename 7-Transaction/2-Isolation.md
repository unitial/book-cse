## Isolation

Conflict serializable，指某一个schedule中所有conflict的operation出现的顺序，与某个sequential schedule中对应的conflict operation的顺序相同。也就是说，这个schedule本身的conflict operation并不会导致无法serialize——因为有另外一个schedule也有同样的conflict operation且order一样，而且是sequential schedule。所以虽然有conflict存在，但能够serializability。（类比：鱼骨头不变，鱼就等价；这里鱼骨头是指conflict operation，鱼指所有operation。）

Conflict serializable的充要条件，就是看是否是acyclic conflict graph。这个方法更直观简单，比上面的定义更容易实际操作。

Final-state serializability则只看结果不看过程，这种定义不容易实现，需要预先知道什么结果是正确的，即先要有一个正确结果的集合；无法通过保证过程的正确来推导出结果一定正确。从理论上，这种serializability的schedule集合是最大的。

View serializability，是指过程正确且结果正确，是conflict serializability的超集，但同样不方便操作，属于NP问题。

三者的关系：**Final-state > View > Conflict**

为了产生Conflict Serializable Schedule，一种方法就是用lock，将会发生conflict的地方都保护起来。

2PL可能产生死锁，因此必须按顺序获得锁。提问：若在获取了7号锁后，又需要4号锁，那么必须先放掉7号锁，再去获取4号锁和7号锁，这样岂不是违反了2PL的规则？是的，所以要全部放掉，再重新拿。

BCC比OCC多走了一步，但多走两步性能反而会变差。

Intel的HTM，其实现机制和OCC是类似的。

OCC本身在validation阶段也需要用锁。

2PL和OCC似乎在数据库课上已经讲过了，后续可结合数据库提问。

提问：View为什么比Conflict还要多？因为View的限制更少一些，PPT中有例子属于View但不属于Conflict。

