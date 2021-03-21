# 深入剖析Tomcat-note

## 0 核心-Cataline

1、servlet 是如何工作的？

分为连接器（connector）和容器（container）。

![](../.gitbook/assets/image%20%28125%29.png)

连接器接收到请求，然后创建一个request对象和response对象，然后将处理过程交给容器。容器从连接器中接收到request对象和response对象，并负责调用相应的servlet 的service 方法。

## 1 HTTP服务器是如何工作的？

（第一章）

## 

## 2 一个简单的servlet

（第二章）

## 

## 3 连接器

（第三、四章）

## 

## 4 容器中的各个组件

（第五~20章）

##   

