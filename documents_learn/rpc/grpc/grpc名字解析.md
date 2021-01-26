# 概览

gRPC支持DNS作为默认的名字自动。在一些部署中使用了一些可选的名字系统。我们支持一个足够通用的API，以支持一系列的名字系统以及对应的名字语法。用各种语言实现的gRPC客户端库将提供一个插件机制，以便可以插入用于不同名称系统的解析器。

# 详细设计

## 名字语法

用于gRPC通道构造的、完全限定的、自包含名称，使用RFC 3986中定义的URI语法。
URI的scheme指定使用何种解析器插件。如果没有指定shceme前缀，或者scheme未知，则默认使用dns作为scheme。
URI的路径指定要被解析的名字。
大多数gRPC的实现支持下面的URI scheme：
* dns:[//authority/]host[:port]  --DNS(默认的)
* unix:path, unix://absolute_path --Unix域套接字(仅支持Unix系统)
* unix-abstract:abstract_path --在抽象名字空间中的Unix域套接字(仅支持Unix系统)
下面的schemes通过gRPC的C-核心实现支持，但是可能在其他语言中不支持：
* ipv4:address[:port][,address[:port],...] --IPv4 addresses
* ipv6:address[:port][,address[:port],...] --IPv6 addresses
未来，可能会添加上对etcd的scheme支持。

## 解析器插件

gRPC 客户端库使用指定的scheme来选择正确的解析器插件，并传递给完全限定的名字字符串。

解析程序应该能够与授权机构联系并获得解决方案，以便他们返回gRPC客户端库。 返回的内容包含如下：

* 解析出来的地址列表（包括IP地址和端口）
  * 每个地址可能具有与之关联的一组任意属性（键/值对），可用于将信息从解析器传递到负载平衡策略。
* 一个service config

插件API允许解析器持续监视一个端点并当需要的时候返回已经更新的解决方案。