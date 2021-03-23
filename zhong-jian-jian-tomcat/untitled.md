# 深入剖析Tomcat-note

## 0 核心-Cataline

1、servlet 是如何工作的？

分为连接器（connector）和容器（container）。

![](../.gitbook/assets/image%20%28125%29.png)

连接器接收到请求，然后创建一个request对象和response对象，然后将处理过程交给容器。容器从连接器中接收到request对象和response对象，并负责调用相应的servlet 的service 方法。

## 1 HTTP服务器是如何工作的？

底层是socket ， 客户端利用socket对象，请求服务器端。服务器端解析请求，并组装成request 对象，然后找到具体的service处理，然后返回Response 对象给客户端。

## 2 一个简单的servlet

1、对一个servlet 的每个HTTP请求，一个功能齐全的servlet 容器需要做一下几件事情：

* 当第一次调用某个servlet 时，要载入该servlet 类 ,并调用其init 方法（仅此一次）
* 针对每个request请求，创建一个javax.servlet.ServletRequest 和 javax.servlet.ServletResponse 实例
* 调用该servlet 的service方法，将servletRequest 和 servletResponse 对象作为参数传入
* 当关闭该servlet 类时，调用其destroy 方法，并卸载该servlet 类。

2、在加载servlet 类时，需要使用类加载器。Tomcat 中可以指定类加载器，有些也破坏了类加载的双亲委派模式。在第8章中会有介绍。



## 3 连接器

（第三、四章）

### 3.1 连接器

1、httpConnector 连接器必须解析从HTTP请求中获取的所有信息。但是解析HTTP 请求涉及一些系统开销大的字符串操作以及一些其他操作。

2、httpProcessor 对象负责创建HttpRequest 的实例，并填充它的成员变量。HttpProcessor 类使用其parse（） 方法解析HTTP请求中的请求头和请求行，并将其填充到HttpRequest 对象的成员变量中。但是parse 方法并不会解析请求体，这个任务由各个httpRequest 对象自己完成。

3、Tomcat中的连接器 需要满足如下要求

* 实现org.apache.catalina.Connection 接口
* 负责创建实现了org.apache.catalina.Request 接口的request对象
* 负责创建实现了org.apache.catalina.Response 接口的response对象



4、todo springboot 中tomcat 连接器 除了基础功能，都做了哪些优化？？

（1）池化

（2）keep-alive 

5、http1.1 的新特性（理解新特性，对理解如何处理HTTP请求很重要）

（1）持久连接

在HTTP1.1 中，会默认使用持久连接。 connection: keep-alive 。

（2）块编码

通过这种方法告诉接收方在不知道发送长度内容的情况下，如果解析已收到的内容。

http1.1 中增加了名为“transfer-encoding” 的特殊请求头，来说明字节流将会分块发送。

（3）状态码100的使用

客户端可以在向服务器发送请求体之前发送如下的请求头，并等待服务器的确认：

Expect: 100-continue

当客户端准备发送一个较长的请求体，而不确定服务端是否会接收时，就可能会发送上面的头信息。若是客户端发送了较长的请求体，却发现服务器拒绝接收时，会是较大的浪费。

接收到“Expect: 100-continue” 请求头后，若服务器可以接收并处理该请求时，可以发送如下响应头：

HTTP/1.1 100 continue

然后，服务器继续读取输入流的内容。



## 

## 4 容器中的各个组件

（第五~20章）

### 4.1 Container 与管道

1、Tomcat 中 有4种类型的容器，都继承自container 接口，4种容器如下：

Engine: 表示整个container servlet 引擎。

Host：表示包含一个或者多个context容器的虚拟主机

Context：表示一个web 应用程序，一个context 包含多个wrapper。

 Wrapper：表示一个独立的servlet 实现。

2、 管道与阀：

管道包含servlet 要调用的任务。一个阀表示一个具体的执行任务。管道中有一个基础阀，然后可以添加其他任意阀。

当一个阀执行完成后，会调用下一个阀继续执行。基础阀总是最后一个执行的。多个阀就是责任链模式。

3、wrapper： 要负责管理其基础的servlet 类的servlet 生命周期。

4、组件关系

[https://v2as.com/article/b14d163f-4688-4a35-b1ba-194a76eb80af](https://v2as.com/article/b14d163f-4688-4a35-b1ba-194a76eb80af)

> Container是容器的父接口，该容器的设计用的是典型的责任链的设计模式，它由四个自容器组件构成，分别是Engine、Host、Context、Wrapper。这四个组件是负责关系，存在包含关系。通常一个Servlet class对应一个Wrapper，如果有多个Servlet定义多个Wrapper，如果有多个Wrapper就要定义一个更高的Container，如Context。   
> Context 还可以定义在父容器 Host 中，Host 不是必须的，但是要运行 war 程序，就必须要 Host，因为 war 中必有 web.xml 文件，这个文件的解析就需要 Host 了，如果要有多个 Host 就要定义一个 top 容器 Engine 了。而 Engine 没有父容器了，一个 Engine 代表一个完整的 Servlet 引擎。
>
> * Engine 容器  Engine 容器比较简单，它只定义了一些基本的关联关系
> * Host 容器  Host 是 Engine 的字容器，一个 Host 在 Engine 中代表一个虚拟主机，这个虚拟主机的作用就是运行多个应用，它负责安装和展开这些应用，并且标识这个应用以便能够区分它们。它的子容器通常是 Context，它除了关联子容器外，还有就是保存一个主机应该有的信息。
> * Context 容器  Context 代表 Servlet 的 Context，它具备了 Servlet 运行的基本环境，理论上只要有 Context 就能运行 Servlet 了。简单的 Tomcat 可以没有 Engine 和 Host。Context 最重要的功能就是管理它里面的 Servlet 实例，Servlet 实例在 Context 中是以 Wrapper 出现的，还有一点就是 Context 如何才能找到正确的 Servlet 来执行它呢？ Tomcat5 以前是通过一个 Mapper 类来管理的，Tomcat5 以后这个功能被移到了 request 中，在前面的时序图中就可以发现获取子容器都是通过 request 来分配的。
> * Wrapper 容器  Wrapper 代表一个 Servlet，它负责管理一个 Servlet，包括的 Servlet 的装载、初始化、执行以及资源回收。Wrapper 是最底层的容器，它没有子容器了，所以调用它的 addChild 将会报错。  Wrapper 的实现类是 StandardWrapper，StandardWrapper 还实现了拥有一个 Servlet 初始化信息的 ServletConfig，由此看出 StandardWrapper 将直接和 Servlet 的各种信息打交道。

### 4.2 生命周期

catalina 在设计上允许一个组件包含其他组件，父组件可以启动/ 关闭它的子组件。Catalina 的这种设计使所有的组件都置于其父组件的监护之下，这样，Catalina 的启动类只需要启动一个组件就可以将全部应用的组件都启动起来。这种单一启动、关闭机制是通过lifecycle 接口实现的。

### 4.3 类加载

1、为什么servlet 需要自己实现一个类加载器？

servlet 容器需要实现一个自定义的载入器，而不能使用简单的使用系统的类加载器， 因为servlet 不应该完全信任它正在运行的servlet 类。  使用系统类的类加载器加载某个servlet 类所使用的全部类，那么servlet 就能访问所有类，包括当前运行的Java虚拟机中环境变量classpath 指明的路径下的所有类和库。这是非常危险的。 servlet 只允许载入web-inf/classes 目录以及子目录下的类，和从部署的库到WEN-INF/lib 目录载入类。

另一个原因是提供自动重载的功能，即当web-inf/classes 目录或 web-inf/lib 目录下的类发生变化时，web 应用程序会重新载入这些类。在Tomcat 类加载器的实现中，类加载器使用一个额外的线程来不断检查sevlet 类和其他类文件的时间戳。

2、双亲委派模式的优点

* 安全
* 复用



### 4.4 

