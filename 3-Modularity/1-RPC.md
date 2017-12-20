## RPC

### Why RPC？

从Enforce Modularity讲起，如果把所有模块放在同一个机器上，会导致fate sharing，所以希望把不同的模块放在不同的机器上运行，因此不同模块间的通信需要通过网络完成。这种分离的思想，也叫做C/S（Client/Service）模型。

**为什么不直接用socket？**网络通信已经学过socket，但socket的问题在于过于底层，每次都需要新建socket，connect，accept等，过于复杂，需要提供一种对程序员友好的网络通信的接口——RPC就是基于程序员熟悉的函数调用（PC），不用操作socket，只需要使用简单的接口即可。

**为什么不适用HTTP协议？**同样可以跨网络，很成熟，而且可以在Internet的范围进行通信。一个原因在于必须用HTTP协议，很多时候为了传输小数据却不得不带一个很大的HTTP Header，因此性能会差很多，对于微博、淘宝等系统并不合适。RPC目前一般用在内部，而不是对外提供服务（当然也可以，不过并不多）。从另一个角度来看，RPC也可以用HTTP来实现，也可以直接用socket实现，所以RPC更多的是一个框架，是对程序员提供的简单易用的接口。

**普通程序员该如何使用RPC？**在server端：实现foo()代码逻辑；在client端，直接调用foo()即可。由RPC框架来完成中间所有的网络传输。

### RPC是如何实现的？

这是重点介绍的内容，包括以下几个子问题。

**如何实现网络的操作？**stub。通过在client和server端自动生成stub来实现。生成stub只需要了解函数的定义即可（即函数名、参数和返回值）。Client端的stub会生成request，发到网络；server端的stub则接收request，调用实际的函数foo()，并将返回值发回到client的stub；client端stub将返回值向上返回给调用函数。所有socket都封装在stub中，程序员不用关心，调用起来很方便。

**通过网络传递的数据有哪些？**Request中包含：transaction ID、function ID、version、parameter等；reply中包含类似内容。

**如何实现参数的传递？**Marshall。指针已经不能使用了，所以需要

**如何实现不同语言间的RPC？**IDL。与程序语言无关的一种表述，类似函数的原型定义。

### RPC存在哪些挑战？

**网络出问题怎么办？**At lease once和at most once。

**延迟很高怎么办？**如何划分哪些用RPC哪些用PC。

**网络安全问题？**server和cleint之间的双向认证。更多内容在11章。

