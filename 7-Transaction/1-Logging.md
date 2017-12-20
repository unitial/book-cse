## Logging

Logging是为了保证操作的原子性。一个场景是：假设一个操作可以分为若干个步骤，若在执行到某个步骤时发生crash，能够保证恢复到该操作执行之前或者之后。

### 三种存储介质：Log、Cell、Cache

可以使用Log-based storage，即磁盘仅仅存储Log，那么read/write也都基于Log。优点在于简单，且write和recover很快；缺点在于read很慢，因为要读取大量Log。

为了提高read的性能，引入Cell storage。Write时先写Log，再写Cell；Read时则只读Cell；Recover时则需要通过undo来取消掉所有未COMMIT的操作。特点在于Cell总是最新的（往往过于新了，所以需要undo）。优点是read变快了；缺点是write要写两遍磁盘。（注意这里有一个隐患：即Transaction在COMMIT之前，其write就已经对外可见了，这是shadow copy所没有的问题；最后会讨论）

为了提高write性能，引入基于内存的cache。Write时先写Log，再写cache；Read时则只读cache，miss则读Cell；Recover时需要读两遍Log，一遍从后往前做undo，一遍从前往后做redo。这是因为Cell的写入有可能被延后了，所以需要做redo。

### Checkpoint

为了提高recover的性能，引入checkpoint。Checkpoint是指将某个Transaction之前的所有操作都写入cell，这样之前的Log就可以忽略（truncate）了。为此需要在Log中写入一个checkpoint记录，当第一遍从后往前扫描Log的时候，找到checkpoint就可以停止了，再从checkpoint开始从前往后扫描Log。优点是log变小了，所以recover变快了；缺点在于checkpoint时系统会暂停接收新的transaction，会导致短暂的停顿。

为了避免checkpoint导致的停顿，引入Non-quiescent checkpoint，将checkpoint作为一个过程（类似一个Transaction），把开始和结束都记下来，从而在recover的时候可作为参考，即如果某个checkpoint之后开始没有结束，那么忽略这个checkpoint。具体来说，开始checkpoint时会在log里记录<1-k>，表示这个checkpoint只保证<1-k>的数据会从内存flush到disk cell；然后等待<1-k>确实都COMMIT或ABORT，并flush；最后在log里写一个END标记。

在上面这个步骤里，由于checkpoint和其他transaction是同时进行的，所以很可能flush到disk cell里的数据包含了T_k+1_的数据，这会不会导致checkpoint数据不一致？可能，但没有关系，因为这种情况等价于<1-k>已经刷到disk cell，然后被k+1覆盖。因为checkpoint的目的是为了删掉<1-k>的log record，无论何种情况，k写入的值一定已经在k+1的log record里，可以恢复。

### 其他

提一下rethink the sync，用external sync的思维来思考对外可见性对性能和容错的影响。

Logging并不能解决before-or-after的问题，在引入cell后，破坏了transaction的对外不可见性，这个问题正是isolation要解决的。

----
后续需要增加的内容：

- 与文件系统Log的结合。例如，[nilfs]的随机写性能的提高，以及一些利用多个磁盘做随机写性能加速的研究工作。
- 进一步区分redo log和undo log，两者各自的优点和缺点，以及不同的适用场景。
- 问题：PPT里在rename的时候，为什么第一种实现是不行的（即两个name指向一个file，refnum却为1）？为什么不直接把inode的refnum改为2，然后继续往下，最后删掉所有#开头的文件？

参考资料：

- [知乎：OCC和MVCC的区别是什么？](https://www.zhihu.com/question/60278698)


