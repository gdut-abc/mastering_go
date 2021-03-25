
* Span是用来表示一个时间跨度的，规范中只定义了Span这个接口，要求有访问其内部的一些属性的接口。但没有定义任何数据细节。 
    * Span中的属性主要有
        * log  span中的日志，应该是给用户使用的一些信息而已
        * tag  span中的标签，部分标签的key名字有实际意义
        * baggageItem  span中传递给下游的一些字段属性
* SpanContext也是一个接口而已，但是这个接口本身的抽象意义是想让span的上下文在下游中传递
    * SpanContext在规范中只是一个非常简单的接口，所以里面承载的核心传递数据是由厂商来确定的
* Tracer是一个接口，提供如下的功能
    * StartSpan
    * Inject
    * Extract
span是Tracer实现生成的，所以span的生成细节是完全由厂商来确定的。
* 目前内建支持的spancontext的载体的格式只有如下三种：
    * HTTPHeaders  字符串形式的kv对，但是因为会放到HTTP头部，所以必然value会需要遵守http的规范，如不能有非法字符等等
    * TextMap 字符串形式的kv对
    * Binary  二进制
* 对于提取的规范是有定义特定的注入失败和提取失败的错误类型的，Tracer实现必须遵循这种出错约定