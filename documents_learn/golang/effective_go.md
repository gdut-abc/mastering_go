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