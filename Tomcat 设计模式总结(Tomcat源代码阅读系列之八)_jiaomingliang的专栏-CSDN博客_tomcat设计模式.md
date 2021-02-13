# Tomcat 设计模式总结(Tomcat源代码阅读系列之八)_jiaomingliang的专栏-CSDN博客_tomcat设计模式
本篇我们将来分析一下 Tomcat 中所涉及到设计模式，本文我们将主要来分析 `外观模式` ， `观察者模式` ， `责任链模式` ， `模板方法模式` , `命令模式` 。 

在开始本文之前，笔者先说明一下对于设计模式的一点看法。笔者曾经经常看到网上有人讨论设计模式，也偶尔会遇到有人非要严格按照 GOF 设计模式的类图以及其中的角色去套用别人的设计，只要类图不一样，或者角色多了或者少了就会觉得怎么和官方定义的模式不一样，其实这都是对设计模式的误解。设计模式其实不仅仅存在软件行业，各行各业其实都有模式，它是所在行业对一些通用问题解决方案的总结和抽象，是一种对常见问题的抽象的解决方案，不是一种具体的实现，所以我们在讨论设计模式的时候，千万别一个劲的套用 GOF 设计模式中的类图以及其中所涉及到的角色，而是要理解设计模式的思维，理解设计模式的使用场景，只有理解了设计模式中所蕴含的思维以及具体的使用场景以后，你才算是真正的掌握了它。ok, 小扯了一下淡，接下来我们进入主题吧。

## 定义

外观模式封装了子系统的具体实现，提供统一的外观类给外部系统，这样当子系统内部实现发生变化的时候，不会影响到外部系统。

## 外观模式在 Tomcat 的应用

在 Tomcat 中对于 Request,Response,StandardSession,ApplicationContext,StandardWrapper 都采用了外观模式，它的类图如下：

![](https://img-blog.csdn.net/20150811204654543)
![](https://img-blog.csdn.net/20150811204701382)

![](https://img-blog.csdn.net/20150811204624549?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

通过上图，我们可以看到 RequestFacade 包装了 Request，它们都实现了 HttpServletRequest，当传递 Request 对象给应用的时候，其实是返回了 RequestFacade 对象，而 RequestFacade 内部可以根据是否自定义了安全管理器来进行相应的操作。

对于 Response,StandardSession 等处理是类似的，这里就不赘述了。

## 定义

观察者模式是一种非常常用的模式，比如大家熟悉的发布 - 订阅模式，客户端应用中的事件监听器，以及通知等其实都属于观察者模式。观察者模式主要是在当系统中发生某个状态变更或者事件的时候，有另外一些组件或者对象对此次变化有兴趣，这个时候那些对变化感兴趣的对象就可以做为观察者对象来监听变化，而被观察对象要负责发生变化的时候触发通知操作。

## 观察者模式在 Tomcat 的应用

Tomcat 中需要对很多组件进行生命周期管理，为此 Tomcat 抽象了统一的生命周期管理骨架，通过这个骨架将所有需要进行生命周期管理的类都纳入进来管理，而这里的骨架的类图如下：

![](https://img-blog.csdn.net/20150811204816381)

通过上图我们可以看出 Tomcat 抽象了一个 LifecycleSupport 的类，而所有需要生命周期管理的组件通过 LifecycleSupport 类通知对某个生命周期事件感兴趣的观察者，而所有的观察者都需要实现 LifecycleListener。  
另外我们需要关注一下 EventObject 对象，它里面定义了一个事件源对象，所谓事件源就是事件发生的地方，而在 Tomcat 的设计中，事件源就是实现了 LifeCycle 接口的各个需要管理生命周期的组件，这里 LifecycleSupport 和 LifeCycleBase 之间是双向的关联，LifeCycleSupport 关联 LifeCycle 对象就是为了实现事件源的传递，这样在 LifeCycleSupport 触发事件的时候，可以通过事件源构建 EventObject. 这样以来 LifecycleListener 就可以通过事件对象获取到事件源，从而做一些与事件源相关的操作。  

## 定义

通过名称我们应该就能知道责任链模式是解决啥问题的？当我们系统在处理某个请求的时候，请求需要经过很多个节点进行处理，每个节点只关注自己的应该做的工作，做完自己的工作以后，将工作转给下一个节点进行处理，直到所有节点都处理完毕。责任链模式在日常生活中例子挺多，比如快递，当你发一个从深圳到北京的快递的时候，你的包裹会从一个分拨中心传递到下一个分拨中心，直到目的地，这里面每个分拨中心都是链路上的一个节点，它做完自己的工作，然后将工作传递到下一个节点，还比如路由器中传递某个数据包其实也是同样的思路。

## 责任链模式在 Tomcat 的应用

Tomcat 中请求的处理流程其实就是采用了责任链模式，关于 Tomcat 请求处理，大家可以参考下[Tomcat 请求处理流程（Tomcat 源代码阅读系列之五）](http://blog.csdn.net/jiaomingliang/article/details/47414657),Tomcat 中责任链模式的实现的类图如下图所示：

![](https://img-blog.csdn.net/20150811205130094)

从上图中，我们可以看到每一个容器都会有一个 Pipeline，而一个 Pipeline 又会具有多个 Valve 阀门，其中 StandardEngine 对应的阀门是 StandardEngineValve，StandardHost 对应的阀门是 StandardHostValve，StandardContext 对应的阀门是 StandardContextValve，StandardWrapper 对应的阀门是 StandardWrapperValve。这里每一 Pipeline 就好比一个管道，而每一 Valve 就相当于一个阀门，一个管道可以有多个阀门，而对于阀门来说有两种，一种阀门在处理完自己的事情以后，只需要将工作委托给下一个和自己在同一管道的阀门即可，第二种阀门是负责衔接各个管道的，它负责将请求传递给下个管道的第一个阀门处理，而这种阀门叫 Basic 阀门，它是每个管道中最后一个阀门，上面的 Standard\*Valve 都属于第二种阀门。我们可以形象的通过下图来描述上面的过程:  

![](https://img-blog.csdn.net/20150811205210727)

通过上图，我们可以很清楚的了解到 Tomcat 的请求处理流程。当用户请求服务器的时候，Connector 会接受请求，从 Socket 连接中根据 http 协议解析出对应的数据，构造 Request 和 Response 对象，然后传递给后面的容器处理，顶层容器是 StandardEngine，StandardEngine 处理请求其实是通过容器的 Pipeline 进行的，而 Pipeline 其实最终是通过管道上的各个阀门进行的，当请求到达 StandardEngineValve 的时候，此阀门会将请求转发给对应 StandardHost 的 Pipeline 的第一个阀门处理，然后以此最终到达 StandardHostValve 阀门，它又会将请求转发给 StandardContext 的 Pipeline 的第一个阀门，这样以此类推，最后到达 StandardWrapperValve，此阀门会根据 Request 来构建对应的 Servelt，并将请求转发给对应的 HttpServlet 处理。从这里我们可以看出其实 Tomcat 核心处理流程就是通过责任链一步步的组装起来的。

## 定义

模板方法模式抽象出某个业务操作公共的流程，将流程分为几个步骤，其中有一些步骤是固定不变的，有一些步骤是变化的，固定不变的步骤通过一个基类来实现，而变化的部分通过钩子方法让子类去实现，这样就实现了对系统中流程的统一化规范化管理。在日常生活中其实也有类似的例子，比如我们知道的连锁加盟店，他们都是有固定的加盟流程，只不过每一家店开的时候，店的选址，装修等不同的而已，但是总体的加盟流程已经是确定的。

## 模板方法模式在 Tomcat 的应用

Tomcat 中关于生命周期管理的地方很好应用了模板方法模式，在一个组件的生命周期中都会涉及到 init(初始化)，start（启动），stop(停止)，destory（销毁），而对于每一个生命周期阶段其实都有固定一些事情要做，比如判断前置状态，设置后置状态，以及通知状态变更事件的监听者等，而这些工作其实是可以固化的，所以 Tomcat 中就将每个生命周期阶段公共的部分固化，然后通过 initInternal,startInternal,stopInternal,destoryInternal 这几个钩子方法开放给子类去实现具体的逻辑。Tomcat 中关于模板方法模式的实现如下图所示：

![](https://img-blog.csdn.net/20150811205305085)

## 定义

命令模式将请求封装为一个命令，将命令发送者和命令接受者解耦，并且所有命令对客户端来说都有统一的调用接口，使用命令模式还可以支持命令的撤销操作，在很多 GUI 程序中大量使用了此模式。  
接下来我们来说一个场景大家感受下，我们有时候可能会遇到接口方法参数过多的问题，这样的接口不仅看起来丑陋而且不方便阅读，对客户端不友好。遇到这种情况我们可能选择将各种参数打包为一个参数对象，接口只需要一个参数对象即可，但是在具体的接口实现中，我们又要做条件判断根据参数值的不同做出不同的响应操作，这个时候其实就可以考虑将不同的逻辑实现和各种参数通过命令打包，然后提供一个命令工厂，客户端通过工厂生产出命令，然后直接调用即可。  
其实在日常生活中，命令模式也很常见，比如公司老大给你分配了个任务，让你去做，他可能不关心你具体怎么做的，你做完了以后告诉他结果即可。

## 命令模式在 Tomcat 的应用

命令模式在 Tomcat 中主要是应用在对请求的处理过程中，Tomcat 的实现中，根据它支持两种协议 AJP 和 Http, 而在具体的 IO 实现中，又分为 Java 同步阻赛 IO,Java 同步非祖塞 IO，以及采用 APR[Apache Portable Runtime ](http://tomcat.apache.org/tomcat-7.0-doc/apr.html)支持库, 因此 Tomcat 统一了`org.apache.coyote.Processor`接口，根据协议和 IO 实现的不同通过不同的 Process 子类去实现，Connector 作为客户端每次只需要根据具体的协议和 IO 实现创建对应的 Process 执行即可。下面我们来看一下命令模式在 Tomcat 中实现的相关类图:

![](https://img-blog.csdn.net/20150811205349833)

通过上图我们可以清楚的看到，Tomcat 首先根据协议的不同将 Processor 分为了 Ajp 和 Http 两组，然后又根据具体的 IO 实现方式的不同，将每一组都会实现同步祖塞 IO, 同步非祖塞 IO，以及 APR 的 Processor。 接下来我们再来看一个类图，我们就可以更加清楚的看到 Tomcat 中是如何利用命令模式来根据不同的协议以及 IO 实现方式来处理请求的。我们来看一下 Tomcat 中关于 ProtocolHandler 的类图。  

![](https://img-blog.csdn.net/20150811205435336)

通过上图我们可以看到针对每一种协议和 IO 实现方式的组合，都会有相应的协议处理类，而每个协议处理类都会有一个 Handler，而每一个 Handler 在运行的时候就会创建出对应的 Processor，比如 AjpProtocol.AjpConnectionHandler 创建 AjpProcessor 处理器，其它的类似。

通过上面的描述，我们可以看出 Tomcat 接受请求的处理流程如下：  
Connector 通过对应的 Endpint 监听 Socket 连接，当对应的端口有连接进来的时候，对应的 Endpoint 就会通过对应的 Handler 类处理，而 Handler 处理的时候，又会创建对应的 Processor 处理, 而对应的 Processor 命令对象会解析 Socket 流的数据，然后生成 Request 和 Response 对象，最终通过上面说的责任链模式一步步的通过各个容器。 
 [https://blog.csdn.net/jiaomingliang/article/details/47426683](https://blog.csdn.net/jiaomingliang/article/details/47426683)
