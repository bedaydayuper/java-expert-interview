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



##   

