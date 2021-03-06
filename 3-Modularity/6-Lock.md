## Lock

前面我们提到用锁来保护BB，我们再来看一个用锁的例子，这次我们用锁来保护文件系统的操作：MOVE（即RENAME）。第一个方法是粗粒度锁，整个文件系统只用一把锁，显然是正确的，但性能却很差，因为很多可以并行做的事情都不得不变成串行。第二个方法是细粒度锁，即每个目录一把锁，似乎也很直观，但却导致了问题：在放掉dir1.lock之后，在拿dir2.lock之前，其实有一个状态：该inode没有在任何dir中有任何filename的记录。这个状态其实暴露了RENAME的中间状态，破坏了原子性。

解决的上面的问题的方法，在于必须同时拿dir1.lock和dir2.lock，最后同时release两把锁。但这样会导致一个新的问题：Deadlock。死锁的原因在于多个thread要拿多把锁，然后不同的thread拿了不同的锁，彼此等待对方放锁。

解决死锁的问题，一个方法是对锁进行排序，然后按照一定的顺序取锁。这样可以避免不同thread拿到不同锁的问题。但又引入了新的问题：需要每个thread在一开始就知道自己要拿什么锁，否则一开始拿了10号锁，然后运行时发现又需要3号锁，这是就不得不把10号锁先放掉，然后拿3号锁再拿10号锁——注意，放掉10号锁意味着要把之前和10号锁相关的操作都回复回去（rollback，回滚），第二次拿到10号锁再重做，而这时运行状态变了可能又不需要3号锁了，或者又需要2号锁，那就只能再回滚再重做。怎么避免这种回滚重做？就是一开始就知道需要几把锁，然后按顺序拿锁。所以这种方法比较适合简单的场景。

**锁的实现**

在前面我们都是假设已经实现了锁，以及锁的接口和行为都符合我们的预期。具体包括：

- ACQUIRE()拿锁，RELEASE()放锁
- 同一时间只有一个thread能够拿到锁
- 如果某个thread拿不到锁，则会在ACQUIRE()处block住，直到锁被释放
- 如果没有thread拿着锁，那么ACQUIRE()一定能拿到锁
- …

那么，锁本身是如何实现的呢？

先来看第一种直观但错误的实现。这段代码的问题在于，如果有A和B两个CPU同时运行ACQUIRE，有可能同时拿到锁，这显然是错误的。错误的原因在于，这里锁的实现再次出现了race的情况。如何解决race呢？用锁。这里就有了一个矛盾——我们必须用锁去保护锁本身的race，出现了递归。

于是体系结构的人提出用硬件提供一些原子指令来实现锁。这里介绍了4种指令以及对应的锁的实现。


