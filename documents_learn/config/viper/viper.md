# viper

## 安装

使用如下方式安装：
`go get github.com/spf13/viper`

## viper是什么

viper是一个针对12-factor的golang程序的一个完整的配置解决方案。设计用来在一个应用程序内工作，并且可以处理各种类型的配置需要和格式。其支持：

* 设置默认值
* 从JSON，TOML，YAML，HCL，env文件以及Java属性配置文件中读取
* 实时监控和重新读取配置文件（可选）
* 从环境变量中读取
* 从远程的配置系统（etcd或者Consul）中读取并监控变化
* 从命令行flags中读取
* 从buffer中读取
* 设置显式的值
Viper可以理解为是一个可以满足你应用程序的所有配置需求的一个注册机制。

## 为什么用viper

当构建一个现代的应用程序的时候，你不用担心配置文件的格式问题。你想要将精力都集中在构建软件上。viper就是解决这个问题的。

viper给你做了如下的事情：

1. 找到，加载并unmarshal配置文件，配置文件可以是JSON，TOML，YAML，HCL，INI，envfile或者Java属性配置文件格式的。

2. 给你的不同的配置选项提供一个机制用于设置默认值

3. 给通过命令行flags指定的选项，提供设置覆盖值的机制

4. 给重新命名参数提供了一个更简单的别名系统，而且不用破坏已经存在的代码

5. 易于区分用户何时提供了与默认值相同的命令行或配置文件。

Viper使用下面的优先级顺序，按照优先级递减的顺序如下：

* explicit call to Set
* flag
* env
* config
* key/value store
* default

注意：
Viper配置的key是不区分大小写。

## 设置值到viper中

### 建立默认值

一个好的配置系统支持默认值。一个默认值对于一个key并不是必须的，但是当一个key没有通过config文件，环境变量或者远程的配置或者命令行flag设置的时候，默认值是非常有用的。实例如下：

```go
viper.SetDefault("ContentDir", "content")
viper.SetDefault("LayoutDir", "layouts")
viper.SetDefault("Taxonomies", map[string]string{"tag": "tags", "category": "categories"})
```

### 读取配置文件

Viper需要最小的配置，主要是为了能知道到哪里去找到配置文件。Viper支持JSON, TOML, YAML, HCL, INI, envfile and Java Properties文件。Viper可以搜索多个路径，但是当前单个Viper实例仅仅单个配置文件。Viper并不默认设置任何默认搜索路径，将默认的决策留给应用程序来设置。

下面是一个如何使用Viper来搜索和读取一个配置文件的实例。不需要任何特定的路径，但是至少要提供一个存放配置文件的路径。

```golang
viper.SetConfigName("config") // config文件的名字，没有扩展名
viper.SetConfigName("yaml") // 必须的 如果config文件在名字中没有扩展名的话
viper.AddConfigPath("/etc/appname/") // 寻找config文件所在的路径
viper.AddConfigPath("$HOME/.appname") // 调用多次增加多个搜索路径
viper.AddConfigPath(".") // 在当前工作目录搜索配置，可选滴
err := viper.ReadInConfig() // 找并读取config文件
if err != nil { // 处理读取配置文件的错误
    panic(fmt.Errorf("Fatal error config file:%s \n", err))
}
```

你可以像下面这样处理没有找到任何配置文件的情况：

```golang
if err := viper.ReadIntConfig(); err != nil {
    if _, ok := err.(viper.ConfigFileNotFoundError); ok {
        // Config file not found, ignore error if desired
    } else {
        // Config file was found but another errror was produced
    }
}
// Config file found and successfully parsed
```

### 写配置文件

从配置文件读取是有用的，但是有时候你可能想存储所有运行时的变化。对于那种情况，有很多命令可用，每一个都有自己的目的：

* WriteConfig - 写当前的viper配置到先前定义好的路径（如果存在的话）。如果没有先前定义的路径则报错。如果配置文件已经存在的话，则覆盖当前的config文件。（注意区分路径和配置文件本身）
* SafeWriteConfig - 写当前的viper配置到先前定义好的路径（如果存在的话）。如果没有先前定义的路径则报错。如果配置文件已经存在的话，则不会覆盖当前的config文件。
* WriteConfigAs - 写当前的viper配置到指定的filepath。如果已经存在的话，则覆盖指定的文件。
* SafeWriteConfigAs - 写当前的viper配置到指定的filepath。如果已经存在的话，则不覆盖指定的文件。

根据经验，标记为安全的所有内容都不会覆盖任何文件，而是仅在不存在时创建，而默认行为是创建或截断。

一个小的实例：

```golang
viper.WriteConfig() // writes current config to predefined path set by 'viper.AddConfigPath()' and 'viper.SetConfigName'
viper.SafeWriteConfig()
viper.WriteConfigAs("/path/to/my/.config")
viper.SafeWriteConfigAs("/path/to/my/.config") // will error since it has already been written
viper.SafeWriteConfigAs("/path/to/my/.config")
```

## 监控和重新读取配置文件

Viper支持当应用运行的时候读取config文件。需要重新启动服务器以使配置生效的日子已经一去不复返了，使用Viper的应用程序可以在运行时读取配置文件的更新，而不会错过任何情况。

简单地告诉viper实例一个watchConfig。可选地，你可以通过提供一个给Viper在每次变化出现的时候运行。

确保你在调用WatchConfig()之前就已经添加了所有的configPaths。

```golang
viper.WatchConfig()
viper.OnConfigChange(func(e fsnotify.Event) {
    fmt.Println("Config file changed:", e.Name)
})
```

## 从io.Reader中读取配置

viper预先定义了很多的配置源，例如文件，环境变量，flags以及远程的KV存储，但是你不必绑定于这些。你可以实现你自己必须的配置源并将其用于viper。

```golang
viper.SetConfigType("yaml")

// any approach to require this configuration into your program.
var yamlExample = []byte(`
Hacker: true
name: steve
hobbies:
- skateboarding
- snowboarding
- go
clothing:
  jacket: leather
  trousers: denim
age: 35
eyes : brown
beard: true
`)

// 就是从这个byte数组中读取到所有的这些内容作为配置
viper.ReadConfig(bytes.NewBuffer(yamlExample))

viper.Get("name") // this would be "steve"
```

### 设置覆盖

这些可能是来自一个命令行flag或者是你自己的应用程序逻辑代码中。

```golang
viper.Set("Verbose", true)
viper.Set("LogFile", LogFile)
```

### 注册和使用别名

别名允许单个值被多个key引用。

```golang
viper.RegisterAlias("loud", "Verbose")

// 以下两行的效果是相同的
viper.Set("verbose", true) // same result as next line
viper.Set("loud", true)   // same result as prior line

// 以下两行其实是读取的同一个值
viper.GetBool("loud") // true
viper.GetBool("verbose") // true
```

### 使用环境变量

Viper对环境变量有完整的支持。这可以让12-factor应用程序从box中出来。总共有5个方法可以帮助与ENV一块工作：

* AutomaticEnv()
* BindEnv(string...) : error
* SetEnvPrefix(string)
* SetEnvKeyReplacer(string...) *strings.Replacer
* AllowEmptyEnv(bool)

使用ENV变量时，务必要意识到Viper将ENV变量视为区分大小写。  
Viper提供了一种机制来尝试确保ENV变量是唯一的。通过使用SetEnvPrefix，可以告诉Viper在读取环境变量时使用前缀。 BindEnv和AutomaticEnv都将使用此前缀。

BindEnv使用一个或两个参数。第一个参数是键名称，第二个参数是环境变量的名称。环境变量的名称区分大小写。如果未提供ENV变量名称，则Viper将自动假定ENV变量与以下格式匹配：前缀+"_"+key名的全大写。当您明确提供ENV变量名称（第二个参数）时，它不会自动添加前缀。例如，如果第二个参数是“ id”，Viper将查找ENV变量“ ID”。

使用ENV变量时要认识的重要一件事是，每次访问该值时都会读取该值。调用BindEnv时，Viper不会固定该值。

AutomaticEnv是强大的帮助程序，尤其是与SetEnvPrefix结合使用时。调用时，Viper会在发出viper.Get请求时随时检查环境变量。它将应用以下规则。用一个跟key的名字的全大写相匹配（如果设置了EnvPrefix的话则增加上前缀）的名字来检查环境变量。

SetEnvKeyReplacer 允许使用strings.Replacer对象来改写ENV keys作为

### ENV实例

```golang
SetEnvPrefix("spf") // will be uppercased automatically
BindEnv("id")

os.Setenv("SPF_ID", "13") // typically done outside of the app

id := Get("id") // 13
```

## 使用Flags

## 远程KV存储支持

## 远程KV存储实例


## 从Viper中获取值

在Viper中，有很多方法根据value的类型来获取一个值。下面的函数函数：

* Get(key string) : interface{}
* GetBool(key string) : bool
* GetFloat64(key string) : float64
* GetInt(key string) : int
* GetIntSlice(key string) : []int
* GetString(key string) : string
* GetStringMap(key string) : map[string]interface{}
* GetStringMapString(key string) : map[string]string
* GetStringSlice(key string) : []string
* GetTime(key string) : time.Time
* GetDuration(key string) : time.Duration
* IsSet(key string) : bool
* AllSettings() : map[string]interface{}

一个最重要的事是每个Get函数都会返回一个零值，如果没有找到的话。为了检查是否指定的key存在，提供了IsSet()函数。
实例：

```golang
viper.GetString("logfile") // case-insensitive Setting & Getting
if viper.GetBool("verbose") {
    fmt.Println("verbose enabled")
}
```

## 访问嵌套key

访问方法也接收
