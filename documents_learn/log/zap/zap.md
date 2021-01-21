快速、结构化、层次化日志库。
# 1 源码README
## 安装
`go get -u go.uber.org/zap`

# 快速启动
在那些性能不错，但不是很关键的场景下，使用`SugaredLogger`。比其他的日志包大概是快4-10倍，并且包含了很多结构化的和printf风格的API。
```golang
logger, _ := zap.NewProduction()
defer logger.Sync() // flushes buffer, if any
sugar := logger.Sugar()
sugar.InFow("failed to fetch URL",
  // 以松散的kv对形式格式化的上下文，这个跟下面的形成对比
  "url", url,
  "attempt", 3,
  "backoff", time.Second,
)
sugar.Infof("Failed to fetch URL:%s", url)
```
当性能和类型安全都很关键的时候，使用Logger。这个是比SugaredLogger更快，并且分配得更少的，但是仅仅支持结构化日志。
```golang
logger, _ := zap.NewProduction()
defer logger.Sync()
logger.Info("failed to fetch URL",
  // 这种就是必须是那种带类型的字段值的结构化上下文
  zap.String("url", url),
  zap.Int("attempt", 3),
  zap.Duration("backoff", time.Second),
)
```
## 性能
## 开发状态：Stable
所有的API都已经结束了，并且在1.x系列中不会再有breaking changes。
支持semver的依赖管理系统的用户应将zap固定为^1。
# 文档
## 概览
zap包提供了一个快速、结构化、层次的logging。
对于应用程序来说，在很hot的执行路径上的log，基于反射的序列化以及字符串格式化的开销都非常大——因为都是CPU密集型的，并且会做大量的小分配。换句话来讲，使用json.Marshal()以及fmt.Fprintf()来记录成千上万的interface{}，肯定会让你的程序变慢。
Zap采用了一个不同的方法。它包含一个无反射，0-分配的JSON编码器，并且基本的Logger尽力避免序列化开销以及分配。通过在此基础上构建高级别的SugaredLogger，zap让用户可以选择何时需要计算每个分配以及何时需要更熟悉的，松散类型的API。
### 选择一个Logger

### 配置zap
### 扩展Zap
### FAQ
# 源码分析
## 源码目录结构|
|路径|用法|
|:-:|:-:|
|buffer|buffer包就是在一个byte slice上提供了一层很薄的封装|
|internal/bufferpool|bufferpool包括zap的共享的内部buffer池|
|internal/color|给TTY输出添加彩色能力|
|internal/exit|就是提供一个打桩，用来给单元测试调用os.Exit(1)用的|
|internal/ztest|提供底层的helper用来测试log输出|
|zapcore|定义和实现了底层的接口，zap也就是在这个基础上构建的|
|zapgrpc|提供了一个与grpclog兼容的logger|
|zaptest|提供了一系列的helpers用来测试log的输出|
|zaptest/observer|observer提供了一个zapcore.Core，用于保留日志条目在内存中的，与编码无关的表示形式。|

# 参考资料
https://pkg.go.dev/go.uber.org/zap