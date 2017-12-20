## System Complexity

系统会趋向于越来越复杂。如何控制复杂性是本门课的重点。

### 复杂性有四个特点

- **Emergent properties**：系统的有些属性是设计时没有考虑到的，等到实现完才发现；
- **Propagation of effects**：即蝴蝶效应，微小的变动在复杂系统中导致巨大的效果；
- **Incommensurate scaling**：通常来说，系统负载增加10倍就需要重新设计了；
- **Trade-offs**：很多场景下，鱼与熊掌不可得兼，要做出取舍。

### 控制复杂性的方法

“分而治之”是控制复杂性的重要原则，也就是模块化设计，需要抽象出接口，然后通过Layer、Hierarchy等方式对模块进行组织。

错误不跨过模块的边界，方便debug。

