# 名字

名字在Golang中和其他语言中一样重要。甚至有语意上的影响：在一个包外的名字可见性取决于是否第一个字符是大写。

## 包名



## Getter

Go并不给getter和setter提供自动支持。你自己提供也没有什么问题，通常这么做也是合适的。但是，将Get放到getter的名字中既不是习惯用法，也不必要。假如你有一个叫做owner的字段（小写的，所以不是exported字段），getter方法应该叫做Owner(大写的，导出字段)，而非GetOwner。导出字段使用大写的名字提供了一种hook机制，这种hook机制可以将方法名和字段区分开。如果需要setter函数，名字为SetOwner。两个名字在实践中都读起来很好:

```golang
owner := obj.Owner()
if owner != user {
    obj.SetOwner(user)
}
```

## 接口名

按照惯例，单方法的接口用方法名称加上一个-er后缀或类似的修饰名来构造一个代理名词：Reader，Writer，Formatter，CloseNotifier等。

有很多这种名称，遵守这些名称以及他们中的函数名非常有生产力。Read, Write, Close, Flush, String等等都有规范的签名和含义。为了避免混淆，除非你的方法真的有相同的签名和含义，否则不要给你的方法名成那种名字。相反，如果您的类型实现的方法的含义与熟知类型上的方法的含义相同，则为其赋予相同的名称和签名；调用您的字符串转换器方法String而不是ToString。

## 混合大小写
在golang的习惯中，写多个单词的名字时，使用`MixedCaps`或者`mixedCaps`而不是用下划线。

# 控制结构

## switch

go的switch比C语言的更通用。表达式不要求是一个常量，也不要求是一个整数。如果switch一直都没有一个表达式能够为true，则case都是从上到下计算，直到有一个能匹配。因此，有可能且惯用地将if-else-if-else链写成switch语句。
```golang
func unhex(c byte) byte {
    // 对于多种情况就自然而然写成这种switch语句
    switch {
    case '0' <= c && c <= '9':
        return c - '0'
    case 'a' <= c && c <= 'f':
        return c - 'a' + 10
    case 'A' <= c && c <= 'F':
        return c - 'A' + 10
    }
    return 0
}
```

## type switch

swith可以用于挖掘一个接口变量的动态类型。这种type switch语句使用一种类型断言的语法(也就是关键字`type`是在括号中的,跟那种类型断言的语法一样)。如果switch在这个表达式中声明一个变量，则这个变量在每个子语句中都是相对应的类型。这也是在这种场景下复用名字的一个惯用法，实际上，就是在每个case下声明了一个名字相同但是类型不同的新变量。
```golang
var t interface{}
t = functionOfSomeType()
switch t := t.(type) {
default:
    fmt.Printf("unexpected type %T\n", t)     // %T prints whatever type t has
case bool:
    fmt.Printf("boolean %t\n", t)             // t has type bool
case int:
    fmt.Printf("integer %d\n", t)             // t has type int
case *bool:
    fmt.Printf("pointer to boolean %t\n", *t) // t has type *bool
case *int:
    fmt.Printf("pointer to integer %d\n", *t) // t has type *int
}
```


# 参考资料
1 https://golang.org/doc/effective_go.html