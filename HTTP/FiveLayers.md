## 5层网络模型

![网络请求流程](https://github.com/ltf9651/Blog/blob/master/HTTP/HTTP_Process.png)


![网络模型分层](https://github.com/ltf9651/Blog/blob/master/HTTP/Layers.png)

1. 物理层：定义物理设备如何传输数据
1. 数据链路层：在通信的实体间建立数据链路连接
1. 网络层：为数据在结点间传输创建逻辑链路
1. 传输层：提供端到端服务(TCP、UDP)
1. 应用层：为软件提供服务（TCP)

用户发送请求过程

1.客户机通过TCP/IP协议建立到服务器的TCP连接
1.客户机向服务端发送HTTP协议请求
1.服务器向客户机发送HTTP协议应答包
1.断开连接，客户端渲染HTMl文档