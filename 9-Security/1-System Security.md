## System Security

安全很难。因为安全是一个“negative goal”。

设计一个安全的系统，要考虑两个方面：目标（goal）和威胁模型（thread model）。目标通常分三类：隐私性、完整性、可用性。威胁模型则需要考虑对所有的系统组件安全性与对攻击者的假设。

人，是系统安全中非常薄弱的一环，要考虑在thread model中。

最小特权原则（Least Privilege Principle），在能完成功能的前提下，只给每个模块最少的权限。

