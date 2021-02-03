# (9条消息) 探秘 Spring AOP到底是JDK 还是 CGLib动态代理（源码分析）_Daryl的博客-CSDN博客
![](https://csdnimg.cn/release/blogv2/dist/pc/img/original.png)

置顶 [十年少 i](https://blog.csdn.net/fristjcjdncg) 2020-08-11 19:50:33 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/articleReadEyes.png)
 97 ![](https://csdnimg.cn/release/blogv2/dist/pc/img/tobarCollect.png)
 收藏 

最后发布: 2020-08-11 19:50:33 首次发布: 2020-08-11 19:50:33

版权声明：本文为博主原创文章，遵循 [CC 4.0 BY-SA](http://creativecommons.org/licenses/by-sa/4.0/) 版权协议，转载请附上原文出处链接和本声明。

## 一、什么是 AOP？

-   与 OOP 相比，面向切面，传统的 oop 开发中的代码逻辑是至上而下的，在这个过程中 会产生一些横切性的问题，会散落在代码的各个角落中，造成难以维护，耦合度高，aop 编程思想就是将这些散落的代码分离出来，独立的封装出来，达到解耦的目的，提高代码的重用性和效率。
-   在日常的软件开发中，拿日志来说，一个系统软件的开发都是必须进行日志记录的，不然万一系统出现什么 bug，你都不知道是哪里出了问题。举个小栗子，当你开发一个登陆功能，你可能需要在用户登陆前后进行权限校验并将校验信息（用户名, 密码, 请求登陆时间，ip 地址等）记录在日志文件中，当用户登录进来之后，当他访问某个其他功能时，也需要进行合法性校验。想想看，当系统非常地庞大，系统中专门进行权限验证的代码是非常多的，而且非常地散乱，我们就想能不能将这些权限校验、日志记录等非业务逻辑功能的部分独立拆分开，并且在系统运行时需要的地方（连接点）进行动态插入运行，不需要的时候就不理，因此 AOP 是能够解决这种状况的思想吧！  
    ![](https://img-blog.csdnimg.cn/20200811194112785.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L2ZyaXN0amNqZG5jZw==,size_16,color_FFFFFF,t_70)
    AOP 思想的实现一般都是基于 **代理模式** ，在 JAVA 中一般采用 JDK 动态代理模式，但是我们都知道，**JDK 动态代理模式只能代理接口而不能代理类**。因此，Spring AOP 会这样子来进行切换，因为 Spring AOP 同时支持 CGLIB、ASPECTJ、JDK 动态代理。
-   aop 的实现用到了动态代理 并且 使用了 jdk 动态代理和 cglib 动态代理
-   具体如下：

    -   **他有一个注解 @EnableAspectJAutoProxy（ProxyTargetClass = false）**

**\[Spring AOP 基于 AspectJ 注解如何实现 AOP]**： **AspectJ 是一个 AOP 框架，它能够对 java 代码进行 AOP 编译（一般在编译期进行），让 java 代码具有 AspectJ 的 AOP 功能（当然需要特殊的编译器）**，可以这样说 AspectJ 是目前实现 AOP 框架中最成熟，功能最丰富的语言，更幸运的是，AspectJ 与 java 程序完全兼容，几乎是无缝关联，因此对于有 java 编程基础的工程师，上手和使用都非常容易。Spring 注意到 AspectJ 在 AOP 的实现方式上依赖于特殊编译器 (ajc 编译器)，因此 Spring 很机智回避了这点，转向采用动态代理技术的实现原理来构建 Spring AOP 的内部机制（动态织入），这是与 AspectJ（静态织入）最根本的区别。**Spring 只是使用了与 AspectJ 5 一样的注解，但仍然没有使用 AspectJ 的编译器，底层依是动态代理技术的实现，因此并不依赖于 AspectJ 的编译器**。 Spring AOP 虽然是使用了那一套注解，其实实现 AOP 的底层是使用了动态代理 (JDK 或者 CGLib) 来动态植入。至于 AspectJ 的静态植入，不是本文重点，所以只提一提。 
 [https://blog.csdn.net/fristjcjdncg/article/details/107942983](https://blog.csdn.net/fristjcjdncg/article/details/107942983)
