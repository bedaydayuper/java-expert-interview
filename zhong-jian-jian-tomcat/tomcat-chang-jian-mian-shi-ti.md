# Tomcat interview

## 1 Tomcat 有几种connector 运行模式？

BIO: 同步阻塞，一个线程处理一个请求。

NIO: 同步非阻塞。利用Java的异步IO处理，可以通过少量的线程处理大量的请求，可以复用同一个线程处理多个connection\(多路复用\)。

Tomcat8在Linux系统中默认使用这种方式

AIO: **异步非阻塞IO**\(Java NIO2又叫AIO\) 主要与NIO的区别主要是操作系统的底层区别.可以做个比喻:比作快递，NIO就是网购后要自己到官网查下快递是否已经到了\(可能是多次\)，然后自己去取快递；AIO就是快递员送货上门了\(不用关注快递进度\)。

## 2  Tomcat 创建servlet 实例的原理？

当容器启动时，会读取在webapps目录下所有的web应用中的web.xml文件，然后对 **xml文件进行解析，并读取servlet注册信息**。然后，将每个应用中注册的servlet类都进行加载，并通过 **反射的方式实例化**。

## 3 Tomcat顶层设计

![](../.gitbook/assets/image%20%28130%29.png)

> Tomcat中只有一个Server，一个Server可以有多个Service，一个Service可以有多个Connector和一个Container；
>
> Server掌管着整个Tomcat的生死大权；
>
> Service 是对外提供服务的；
>
> Connector用于接受请求并将请求封装成Request和Response来具体处理；
>
> Container用于封装和管理Servlet，以及具体处理request请求；

## 4 connector 

![](../.gitbook/assets/image%20%28129%29.png)

功能：

> 1. Connector如何接受请求的？
> 2. 如何将请求封装成Request和Response的？
> 3. 封装完之后的Request和Response如何交给Container进行处理的？
> 4. Container处理完之后如何交给Connector并返回给客户端的？

其中ProtocolHandler由包含了三个部件：Endpoint、Processor、Adapter。

Endpoint用来处理底层Socket的网络连接，Processor用于将Endpoint接收到的Socket封装成Request，Adapter用于将Request交给Container进行具体的处理。

Endpoint由于是处理底层的Socket网络连接，因此Endpoint是用来实现TCP/IP协议的，而Processor用来实现HTTP协议的，Adapter将请求适配到Servlet容器进行具体的处理。

Endpoint的抽象实现AbstractEndpoint里面定义的Acceptor和AsyncTimeout两个内部类和一个Handler接口。Acceptor用于监听请求，AsyncTimeout用于检查异步Request的超时，Handler用于处理接收到的Socket，在内部调用Processor进行处理。



## 5 container 架构分析

![](../.gitbook/assets/image%20%28131%29.png)

> Engine：引擎，用来管理多个站点，一个Service最多只能有一个Engine；
>
> Host：代表一个站点，也可以叫虚拟主机，通过配置Host就可以添加站点；
>
> Context：代表一个应用程序，对应着平时开发的一套程序，或者一个WEB-INF目录以及下面的web.xml文件；
>
> Wrapper：每一Wrapper封装着一个Servlet；

## 6 Container如何处理请求的

![](../.gitbook/assets/image%20%28128%29.png)

> Connector在接收到请求后会首先调用最顶层容器的Pipeline来处理，这里的最顶层容器的Pipeline就是EnginePipeline（Engine的管道）；
>
> 在Engine的管道中依次会执行EngineValve1、EngineValve2等等，最后会执行StandardEngineValve，在StandardEngineValve中会调用Host管道，然后再依次执行Host的HostValve1、HostValve2等，最后在执行StandardHostValve，然后再依次调用Context的管道和Wrapper的管道，最后执行到StandardWrapperValve。
>
> 当执行到StandardWrapperValve的时候，会在StandardWrapperValve中创建FilterChain，并调用其doFilter方法来处理请求，这个FilterChain包含着我们配置的与请求相匹配的Filter和Servlet，其doFilter方法会依次调用所有的Filter的doFilter方法和Servlet的service方法，这样请求就得到了处理！
>
> 当所有的Pipeline-Valve都执行完之后，并且处理完了具体的请求，这个时候就可以将返回的结果交给Connector了，Connector在通过Socket的方式将结果返回给客户端。

## 7 Servlet生命周期可分为5个步骤

> * 简单总结：**只要访问Servlet，service\(\)就会被调用。init\(\)只有第一次访问Servlet的时候才会被调用。destroy\(\)只有在Tomcat关闭的时候才会被调用。**
>
> 1. **加载Servlet**。当Tomcat第一次访问Servlet的时候，**Tomcat会负责创建Servlet的实例**
> 2. **初始化**。当Servlet被实例化后，Tomcat会**调用init\(\)方法初始化这个对象**
> 3. **处理服务**。当浏览器**访问Servlet**的时候，Servlet **会调用service\(\)方法处理请求**
> 4. **销毁**。当Tomcat关闭时或者检测到Servlet要从Tomcat删除的时候会自动调用destroy\(\)方法，**让该实例释放掉所占的资源**。一个Servlet如果长时间不被使用的话，也会被Tomcat自动销毁
> 5. **卸载**。当Servlet调用完destroy\(\)方法后，等待垃圾回收。如果**有需要再次使用这个Servlet，会重新调用init\(\)方法进行初始化操作**。

## 参考文献

1、[https://blog.csdn.net/ThinkWon/article/details/104397665](https://blog.csdn.net/ThinkWon/article/details/104397665)

2、[https://xie.infoq.cn/article/34904a222562319881838118d](https://xie.infoq.cn/article/34904a222562319881838118d)

