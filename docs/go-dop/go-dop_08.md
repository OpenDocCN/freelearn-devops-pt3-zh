# 第七章：编写命令行工具

访问任何 DevOps 工程师，你会发现他们的屏幕上充满了执行 **命令行界面** (**CLI**) 应用程序的终端。

作为 DevOps 工程师，我们不只是希望使用别人为我们制作的应用程序；我们希望能够编写自己的 CLI 应用程序。这些应用程序可以通过 REST 或 gRPC 与各种系统通信，正如我们在前一章中讨论的那样。或者你可能想要执行各种应用程序，并通过自定义处理运行它们的输出。一个应用程序甚至可以设置开发环境并启动新发布的测试周期。

无论你的用例是什么，你都需要使用一些常见的包来帮助你管理应用程序的输入和输出处理。

在本章中，你将学习如何使用 `flag` 和 `os` 包编写简单的 CLI 应用程序。对于更复杂的应用程序，你将学习如何使用 Cobra 包。这些技能，加上我们之前章节中学到的技能，将让你能够为你自己或你客户的需求构建各种应用程序。

本章将涵盖以下主要主题：

+   实现应用程序的 I/O

+   使用 Cobra 构建高级 CLI 应用程序

+   处理操作系统信号

在本节中，我们将深入探讨如何使用标准库的 `flag` 包来构建基本的命令行程序。让我们开始吧！

# 技术要求

本章的代码文件可从 [`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/7`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/7) 下载。

# 实现应用程序的 I/O

CLI 应用程序需要一种方式来理解你希望它们执行的方式。这可能包括要读取哪些文件，要联系哪些服务器以及要使用哪些凭据。

有几种方式可以启动一个应用程序，以满足它所需的参数：

+   使用 `flag` 包定义命令行标志

+   使用 `os.Args` 读取未定义的参数

当你有一个命令行参数具有严格定义时，`flag` 包将对你有所帮助。这可能是一个定义所需服务端点的参数。程序可能希望在生产环境中有一个默认值，但在测试时允许覆盖。这对于标志非常合适。

例如，一个程序可能会查询我们之前创建的 **每日引语** (**QOTD**) 服务器。我们可能希望它自动使用我们的生产端点，除非我们指定使用另一个地址。这可能看起来像这样：

```
qotd
```

这简单地联系到我们的生产服务器并获取我们的报价。`--endpoint` 标志，默认使用我们的生产地址，将使用下面的另一个地址：

```
qotd --endpoint="127.0.0.1:3850"
```

有时，应用程序的参数就足够了。以一个用于重新格式化 JSON 数据以供人类可读的应用程序为例。如果没有提供文件，我们可能只希望从 `STDIN` 读取数据。在这种情况下，直接从命令行读取值就足够了，可以使用 `os` 包。这将使我们的执行结果如下所示：

```
reformat file1.json file2.json
```

这里，我们正在读取 `file1.json` 和 `file2.json`，并输出重新格式化的文本。

这里，我们接收 `wget` 调用的输出，并通过 STDIN 将其读取到我们的 `reformat` 二进制文件中。这类似于 `cat` 和 `grep` 的工作方式。当我们的参数为空时，它们会直接从 `STDIN` 中读取：

```
wget "http://some.server.com" | reformat
```

有时候，我们可能希望将标志和参数混合使用。`flag` 包也可以帮助处理这种情况。

那么，让我们开始使用 `flag` 包吧。

## `flag` 包

为了处理命令行参数，Go 提供了标准库中的 `flag` 包。使用 `flag`，你可以为标志设置默认值、提供标志描述，并允许用户在命令行中覆盖默认值。

使用 `flag` 包的标志通常以 `--` 开头，类似于 `--endpoint`。值可以是紧跟在端点后的连续字符串或用引号括起来的字符串。虽然你可以使用单个 `-` 替代 `--`，但在处理布尔型标志时有一些特殊情况，我建议在所有情况下使用 `--`。

你可以在这里找到 `flag` 包的文档：[`pkg.go.dev/flag`](https://pkg.go.dev/flag)。

让我们展示一个标志的实际应用：

```
var endpoint = flag.String(
     "endpoint", 
     "myserver.aws.com", 
     "The server this app will contact",
)
```

这段代码的作用是：

+   定义一个 `endpoint` 变量来存储标志

+   使用 `String` 标志

+   将标志定义为 `endpoint`

+   设置标志的默认值为 `myserver.aws.com`

+   设置标志的描述

如果我们不传递`--endpoint`，代码将使用默认值。为了让我们的程序读取该值，我们只需执行以下操作：

```
func main() {
     flag.Parse()
     fmt.Println("server endpoint is: ", *endpoint)
}
```

重要提示

`flag.String()` 返回 `*string`，因此上面的 `*endpoint`。

`flag.Parse()` 对于使标志在应用程序中可用至关重要。这个方法应该只在 `main()` 包内调用。

提示

Go 中的最佳实践是从不在 `main` 包外定义标志。只需将值作为函数参数或在对象构造函数中传递。

除了 `String()`，`flag` 还定义了其他一些标志函数：

+   `Bool()` 用于捕获 `bool`

+   `Int()` 用于捕获 `int`

+   `Int64()` 用于捕获 `int64`

+   `Uint()` 用于捕获 `uint`

+   `Uint64()` 用于捕获 `uint64`

+   `Float64()` 用于捕获 `float64`

+   `Duration()` 用于捕获 `time.Duration`，例如 `3m10s`

现在我们已经看过基本类型，接下来我们来谈谈自定义标志类型。

## 自定义标志

有时，我们希望将值放入 `flag` 包中未定义的类型中。

要使用自定义标志，必须定义一个实现了 `flag.Value` 接口的类型，接口定义如下：

```
type Value interface {
     String() string
     Set(string) error
}
```

接下来，我们将借用一个来自 Godoc 的示例，展示一个名为 `URLValue` 的自定义值，用于处理表示 URL 的标志，并将其存储在我们的标准 `*url.URL` 类型中：

```
type URLValue struct {
    URL *url.URL
}
func (v URLValue) String() string {
    if v.URL != nil {
        return v.URL.String()
    }
    return ""
}
func (v URLValue) Set(s string) error {
    if u, err := url.Parse(s); err != nil {
        return err
    } else {
        *v.URL = *u
    }
    return nil
}
var u = &url.URL{}
func init() {
    flag.Var(&URLValue{u}, "url", "URL to parse")
}
func main() {
    flag.Parse()
    if reflect.ValueOf(*u).IsZero() {
        panic("did not pass an URL")
    }
    fmt.Printf(`{scheme: %q, host: %q, path: %q}`, 
                 u.Scheme, u.Host, u.Path)
}
```

这段代码执行了以下操作：

+   定义了一个名为`URLValue`的`flag.Value`类型。

+   创建一个名为`-url`的标志，用于读取有效的 URL。

+   使用`URLValue`包装器将 URL 存储在`*url.URL`变量中。

+   使用`reflect`包来判断`struct`是否为空。

通过在类型上定义`Set()`方法，像我们之前所做的那样，你可以读取任何自定义值。

现在我们已经掌握了标志类型的使用，接下来看看一些基本的错误处理。

## 基本的标志错误处理

当我们输入不兼容的标志或具有错误值的标志时，通常我们希望程序输出错误的标志及其值。

这可以通过`PrintDefaults()`选项来实现。以下是一个示例：

```
var (
     useProd = flag.Bool("prod", true, "Use a production endpoint")
     useDev = flag.Bool("dev", false, "Use a development endpoint")
     help = flag.Bool("help", false, "Display help text")
)
func main() {
     flag.Parse()
     if *help {
          flag.PrintDefaults()
          return
     }
     switch {
     case *useProd && *useDev:
          log.Println("Error: --prod and --dev cannot both be set")
          flag.PrintDefaults()
          os.Exit(1)
     case !(*useProd || *useDev):
          log.Println("Error: either --prod or --dev must be set")
          flag.PrintDefaults()
          os.Exit(1)
     }
}
```

这段代码执行了以下操作：

+   定义一个`--help`标志，如果设置了该标志，则只打印默认值。

+   定义了另外两个标志，`--prod`和`--dev`。

+   如果设置了`--prod`和`--dev`，则输出错误信息和默认标志值。

+   如果两个标志都没有设置，则输出错误信息和默认值。

下面是输出的示例：

```
Error: --prod and --dev cannot both be set
  -dev
         Use a development endpoint (default false)
  -prod
         Use a production endpoint (default true)
```

这段代码演示了如何拥有有效默认值的标志，但如果这些值被更改导致错误，我们可以检测并处理这个错误。按照良好的命令行工具精神，我们提供了`--help`，允许用户查看可以使用的标志。

## 简写标志

在前面的示例中，我们有一个`--help`标志。但是通常，你可能希望提供一个简写标志，比如`-h`，供用户使用。它们需要具有相同的默认值，并且都需要设置相同的变量，因此它们不能有两个不同的值。

我们可以使用`flag.[Type]Var()`调用来帮助我们实现这一点：

```
var (
    useProd = flag.Bool("prod", true, 
                  "Use a production endpoint")
    useDev = flag.Bool("dev", false, 
                  "Use a development endpoint")
    help = new(bool)
)
func init() {
    flag.BoolVar(help, "help", false, "Display help text")
    flag.BoolVar(help, "h", false, 
                 "Display help text (shorthand)")
}     
```

在这里，我们将`--help`和`--h`的结果存储在`help`变量中。我们使用`init()`进行初始化，因为`BoolVar()`不会返回变量；因此，它不能在`var()`语句中使用。

现在我们已经了解了简写标志的工作原理，让我们来看一下非标志参数。

## 访问非标志参数

Go 中的参数可以通过几种方式读取。你可以使用`os.Args`读取原始参数，它也会包含所有的标志。若没有使用标志，这是非常方便的。

使用标志时，`flag.Args()`可以用来仅检索非标志参数。如果我们想将一份作者列表发送到开发服务器，并为每个作者获取 QOTD，命令可能是这样的：

```
qotd --dev "abraham lincoln" "martin king" "mark twain"
```

在这个列表中，我们使用`--dev`标志来指示我们希望使用开发服务器。在标志之后，我们列出了参数。让我们来获取这些参数：

```
func main() {
     flag.Parse()
     authors := flag.Args
     if len(authors) == 0 {
          log.Println("did not pass any authors")
          os.Exit(1)
     }
     ...
```

在这段代码中，我们执行了以下操作：

+   使用`flag.Args()`获取非标志参数。

+   测试我们是否收到了至少一个作者，否则就退出并显示错误。

我们已经看到如何获取作为参数或标志传入的输入。这可以用来定义如何联系服务器或打开哪些文件。现在让我们看一下如何从流中接收输入。

## 从 STDIN 获取输入

现在，DevOps 社区中编写的大多数应用程序倾向于围绕标志和参数展开，正如之前所见。DevOps 人员每天使用的一种不太常见的输入方法是将输入通过管道传递到程序中。

工具如`cat`、`xargs`、`sed`、`awk`和`grep`允许你将一个工具的输出传递给下一个工具的输入以完成任务。一个简单的例子可能是查找我们从网络获取的文件中包含`error`字样的行：

```
wget http://server/log | grep -i "error" > only_errors.txt 
```

像`cat`这样的程序在未指定文件时从`STDIN`读取输入。我们在这里复制了这个行为，编写一个程序来查找每一行中的`error`并打印出来：

```
var errRE = regexp.MustCompile(`(?i)error`)
func main() { 
    var s *bufio.Scanner 
    switch len(os.Args) { 
  case 1:
          log.Println("No file specified, using STDIN")
          s = bufio.NewScanner(os.Stdin)
  case 2:
          f, err := os.Open(os.Args[1])
          if err != nil {
                  log.Println(err)
                  os.Exit(1)
          }
          s = bufio.NewScanner(f)
  default:
          log.Println("too many arguments provided")
          os.Exit(1)
  }
  for s.Scan() {
          line := s.Bytes()
          if errRE.Match(line) {
                  fmt.Printf("%s\n", line)
          }
  }
  if err := s.Err(); err != nil {
          log.Println("Error: ", err)
          os.Exit(1)
  }
}
```

这段代码做了以下事情：

+   使用`regexp`包编译一个正则表达式来查找包含`error`的行——匹配是大小写不敏感的。

+   使用`os.Args()`读取我们的参数列表。我们使用这个而不是`flag.Args()`，因为我们没有定义任何标志。

+   如果我们只有一个参数（程序名），它会使用`os.Stdin`，这是一个`io.Reader`，我们将其包装在一个`bufio.Scanner`中。

+   如果我们有一个文件参数，它会打开文件，并将`io.Reader`包装在一个`bufio.Scanner`对象中。

+   如果我们有更多的参数，它将返回一个错误。

+   一行一行地读取输入，并将每一行包含`error`字样的行打印到`os.Stdout`。

+   检查我们是否有输入错误——`io.EOF`不被视为错误，因此不会触发`if`语句。

你可以在这个代码库中找到此代码：[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/7/filter_errors/main.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/7/filter_errors/main.go)。

使用编译后的`filter_errors`代码，我们可以用它来扫描`wget`输入（或任何传入的输入），查找包含`error`字样的行，然后使用`grep`过滤特定的错误代码，如`401`（未授权）：

```
wget http://server/log | filter_errors | grep 401
```

或者我们也可以以相同的方式搜索日志文件：

```
filter_errors log.txt | grep 401
```

这是一个简单的例子，借助现有工具很容易实现，但它展示了如何构建类似的工具。

在这一节中，我们已经学习了如何从命令行读取不同的输入形式，包括标志和参数。我们看到了共享状态的简短标志与长格式标志。你也看到了如何创建自定义类型来用作标志。最后，我们学习了如何成功使用 STDIN 来读取通过管道传送的输入。

接下来，我们将学习如何使用第三方包 Cobra 来创建更复杂的命令行应用程序。

# 使用 Cobra 进行高级 CLI 应用程序

Cobra 是一组软件包，允许开发人员创建更复杂的 CLI 应用程序。当应用程序的复杂性导致标志列表变得繁多时，它比标准的`flag`包更有用。

在这一节中，我们将讨论如何使用 Cobra 创建结构化的 CLI 应用程序，这些应用程序对开发人员友好，便于添加功能，并使用户能够了解应用程序中可用的功能。

Cobra 提供的几个功能如下：

+   嵌套子命令

+   命令建议

+   为命令创建别名，以便你在不破坏用户的情况下进行更改

+   从标志和命令生成帮助文本

+   为各种 shell 生成自动补全代码

+   手册页创建

本节将大量引用 Cobra 文档，你可以在这里找到：[`github.com/spf13/cobra/blob/master/user_guide.md`](https://github.com/spf13/cobra/blob/master/user_guide.md)。

## 代码组织

为了有效使用 Cobra，并使开发人员更容易理解在哪里添加和更改命令，Cobra 建议使用以下结构：

```
 appName/
     cmd/
          add.go
          your.go
          commands.go
          here.go
     main.go
```

这个结构将你的主要 `main.go` 可执行文件放在顶层目录下，所有命令都位于 `cmd/` 目录下。

Cobra 应用程序的主文件主要用于初始化 Cobra 并让其执行命令。该文件将如下所示：

```
package main
import (
     "{pathToYourApp}/cmd"
)
func main() {
     cmd.Execute()
}
```

接下来，我们将使用 Cobra 生成器应用程序来生成基础代码。

## 可选的 Cobra 生成器

Cobra 提供了一个可以为我们的应用程序生成基础代码的应用程序。要开始使用生成器，我们将在根目录创建一个名为 `~/.cobra.yaml` 的配置文件：

```
author: John Doak myemail@somedomain.com
year: 2021
license: MIT
```

这将处理打印我们的 MIT 许可证。你可以使用以下任何一个内置许可证值：

+   GPLv2

+   GPLv3

+   LGPL

+   AGPL

+   2-Clause BSD

+   3-Clause BSD

如果你需要这里没有的许可证，可以在这里找到如何提供自定义许可证的说明：[`github.com/spf13/cobra-cli/blob/main/README.md#configuring-the-cobra-generator`](https://github.com/spf13/cobra-cli/blob/main/README.md#configuring-the-cobra-generator)。

默认情况下，Cobra 会使用来自你 `home` 目录的配置文件。如果你需要不同的许可证，请将配置文件放入你的仓库，并使用 `cobra --config="config/location.yaml"` 来使用替代的配置文件。

要下载 Cobra 并使用 Cobra 生成器进行构建，请在命令行中键入以下内容：

```
go get github.com/spf13/cobra/cobra
go install github.com/spf13/cobra/cobra
```

现在，为了初始化应用程序，请确保你处于新应用程序的根目录，并执行以下操作：

```
cobra init --pkg-name [repo path]
```

重要提示

`[repo path]` 将是类似 `github.com/spf13/newApp` 的路径。

让我们为我们的应用程序创建几个命令：

```
cobra add serve
cobra add config
cobra add create -p 'configCmd'
```

这将为我们提供以下内容：

```
app/
      cmd/
         serve.go
         config.go
         create.go
       main.go
```

重要提示

你需要使用 camelCase 格式来命名命令。如果不这样做，将会遇到错误。

`create` 的 `-p` 选项用于将其作为 `config` 的子命令。后面的字符串是父命令的名称加上 `Cmd`。所有其他 `add` 调用都将 `-p` 设置为 `rootCmd`。

在你执行 `go build` 后，我们可以像这样运行它：

+   `app`

+   `app serve`

+   `app config`

+   `app config create`

+   `app help serve`

在基础代码框架完成后，我们只需要配置要执行的命令。

## 命令包

在生成的 `cmd` 包中，你会找到每个可执行命令的文件。我们需要修改每个文件，以便提供正确的帮助文本、使用标志并执行命令。

我们将查看一个通过以下命令创建的应用程序生成的 `cmd/get.go` 文件：

```
cobra init --pkg-name [repo path]
cobra add get
```

这个应用程序将与我们在 *第六章* 中创建的 QOTD 服务器进行交互，*与远程数据源交互*。

生成的 `cmd/get.go` 文件将类似于以下内容：

```
var getCmd = &cobra.Command{
        Use:   "get",
        Short: "A brief description of your command",
        Long: `A longer description that spans multiple lines and likely contains examples and usage of using your command.`,
        Run: func(cmd *cobra.Command, args []string) {
                fmt.Println("get called")
        },
}
func init() {
        rootCmd.AddCommand(getCmd)
}
```

这段代码执行以下操作：

+   创建一个名为 `serveCmd` 的变量：

    +   变量名是基于命令名称加上 `Cmd` 后缀。

    +   `Use` 是命令行中的参数名称。

    +   `Short` 是简要描述。

    +   `Long` 是一个更长的描述，包含示例。

    +   `Run` 是你要执行的代码的入口点。

+   定义 `init()`，它执行以下操作：

    +   将此命令添加到 `rootCmd` 对象中。

让我们用这个来编写我们的 QOTD CLI：

```
     ...
     Run: func(cmd *cobra.Command, args []string) {
        const devAddr = "127.0.0.1:3450"
        fs := cmd.Flags()
        addr := mustString(fs, "addr")
        if mustBool(fs, "dev") {
                addr = devAddr
        }
        c, err := client.New(addr)
        if err != nil {
                fmt.Println("error: ", err)
                os.Exit(1)
        }
        a, q, err := c.QOTD(cmd.Context(), mustString(fs, "author"))
        if err != nil {
                fmt.Println("error: ", err)
                os.Exit(1)
        }
        switch {
        case mustBool(fs, "json"):
                b, err := json.Marshal(
                        struct{
                                Author string
                                Quote string
                        }{a, q},
                )
                if err != nil {
                        panic(err)
                }
                fmt.Printf("%s\n", b)
        default:
                fmt.Println("Author: ", a)
                fmt.Println("Quote: ", q)
        }
     },
}
```

这段代码执行以下操作：

+   设置一个 `addr` 变量来保存我们的服务器地址：

    +   如果传递了 `--dev`，它会将 `addr` 设置为 `devAddr`。

    +   否则，它使用 `--addr` 标志的值。

    +   `--addr` 默认为 `127.0.0.1:80`。

+   创建一个新的客户端，用于连接我们的 QOTD 服务器

+   调用 QOTD 服务器：

    +   使用传递给 `*cobra.Command` 的 `Context`

    +   使用 `--author` 标志的值，默认为空字符串

+   使用 `--json` 标志来决定输出是否为 JSON 格式：

    +   如果是 JSON 格式，它会将内联定义的结构体输出为 JSON。

    +   否则，它只是将其漂亮地打印到屏幕上。

        重要提示

        你将看到 `mustBool()` 和 `mustString()` 函数。这些函数只是返回传递的标志名称对应的值。如果标志未定义，它会触发 panic。这样可以去掉许多冗长的代码，因为这部分内容必须在 CLI 应用程序中始终有效。这些函数位于代码库版本中。

        你看到的标志并非来自标准库的 `flag` 包。相反，这个包使用了来自 [`github.com/spf13/pflag`](https://github.com/spf13/pflag) 的标志类型。这个包比标准 `flag` 包有更多的内置类型和方法。

现在，我们需要在 `Run` 函数中定义我们正在使用的标志：

```
func init() {
        rootCmd.AddCommand(getCmd)
        getCmd.Flags().BoolP("dev", "d", false, 
            "Uses the dev server instead of prod")
        getCmd.Flags().String("addr", "127.0.0.1:80", 
            "Set the QOTD server to use, 
            defaults to production")
        getCmd.Flags().StringP("author", "a", "", 
            "Specify the author to 
            get a quote for")
        getCmd.Flags().Bool("json", false, 
            "Output is in JSON format")
}
```

这段代码执行以下操作：

+   添加一个名为 `--dev` 的标志，可以缩写为 `-d`，默认为 `false`

+   添加一个名为 `--addr` 的标志，默认为 `"127.0.0.1:80"`

+   添加一个名为 `--author` 的标志，可以缩写为 `-a`

+   添加一个名为 `--json` 的标志，默认为 `false`

    重要提示

    以 `P` 结尾的方法，例如 `BoolP()`，定义了缩写的标志以及长标志名称。

我们定义的标志仅在执行 `get` 命令时可用。如果我们在 `get` 命令下创建子命令，这些标志只会在没有定义子命令的 `get` 命令中可用。

要添加适用于所有子命令的标志，请使用 `.PersistentFlags()` 而不是 `.Flags()`。

这个客户端的代码可以在以下仓库中找到：[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/7/cobra/app/`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/7/cobra/app/)。

现在，我们可以运行我们的应用程序并调用这个命令。在这些示例中，你需要运行 gRPC 章节中的 QOTD 服务器，像这样：

```
$ go run qotd.go --addr=127.0.0.1:3560
$ go run main.go get --addr=127.0.0.1:3560 --author="Eleanor Roosevelt" –json 
```

这将使用位于`127.0.0.1:3560`地址的服务器运行我们的应用程序，并请求一个来自埃莉诺·罗斯福的引用，输出为 JSON 格式：

```
{"Author":"Eleanor Roosevelt","Quote":"The future belongs to
those who believe in the beauty of their dreams"}
```

这个示例从地址为`127.0.0.1:3560`的服务器获取一个随机的引用：

```
$ go run main.go get --addr=127.0.0.1:3560 
Author: Mark Twain 
Quote: Golf is a good walk spoiled
```

在这一部分中，我们学习了 Cobra 包是什么，如何使用 Cobra 生成器工具来引导 CLI 应用程序，最后，如何使用这个包为你的应用程序构建命令。

接下来，我们将讨论如何处理信号，在退出应用程序之前进行清理。

# 处理操作系统信号

在编写 CLI 应用程序时，开发者有时需要处理操作系统信号。最常见的例子是用户试图退出程序，通常通过快捷键来实现。

在这些情况下，你可能想在退出之前进行一些文件清理，或取消对远程系统的调用。

在这一部分中，我们将讨论如何捕获并响应这些事件，以使你的应用程序更加强健。

## 捕获操作系统信号

Go 处理两种类型的操作系统信号：

+   同步

+   异步

同步信号通常与程序错误相关。Go 将这些信号视为运行时的恐慌，因此可以使用`defer`语句来处理这些信号的拦截。

根据平台的不同，异步信号有所不同，但对于 Go 程序员来说，最相关的信号如下：

+   `SIGHUP`：连接的终端断开。

+   `SIGTERM`：请退出并进行清理（由程序生成）。

+   `SIGINT`：与`SIGTERM`相同（从终端发送）。

+   `SIGQUIT`：与`SIGTERM`相同，外加一个核心转储（从终端发送）。

+   `SIGKILL`：程序必须退出；此信号无法捕获。

在出现这些情况时，拦截信号可以非常有用，这样你就可以取消正在进行的操作并在退出前进行清理。需要注意的是，`SIGKILL`无法被拦截，而`SIGHUP`只是表明一个进程失去了其终端连接，并不一定意味着进程被取消。这可能是因为进程被移动到后台或发生了其他类似事件。

要捕获一个信号，我们可以使用`os/signal`包。这个包允许程序接收来自操作系统的信号通知并做出响应。下面是一个简单的示例：

```
signals := make(chan os.Signal, 1)
signal.Notify(
     signals,
     syscall.SIGINT,
     syscall.SIGTERM,
     syscall.SIGQUIT,
)
go func() {
     switch <-signals {
     case syscall.SIGINT, syscall.SIGTERM:
          cleanup()
          os.Exit(1)
     case syscall.SIGQUIT:     
          cleanup()
          panic("SIGQUIT called")
     }
}()
```

这段代码执行以下操作：

+   创建一个`signals`通道来接收信号

+   订阅`SIGINT`、`SIGTERM`和`SIGQUIT`类型的信号

+   使用一个 goroutine 来处理传入的信号，执行以下操作：

    +   调用`cleanup()`函数来处理程序的清理工作

    +   在收到`SIGINT`和`SIGTERM`时以`1`代码退出

    +   在`SIGQUIT`信号下发生 panic，产生基本的核心转储

信号处理代码应该写在`main`包中。`cleanup()`函数应包含处理未完成项目的函数调用，例如远程调用取消和文件清理。

重要提示

你可以通过环境变量`GOTRACEBACK`来控制核心转储的数据量和生成方式。你可以在这里阅读相关内容：[`pkg.go.dev/runtime#hdr-Environment_Variables`](https://pkg.go.dev/runtime#hdr-Environment_Variables)。

## 使用`Context`进行取消

在 Go 中，停止操作的关键方法是使用 Go 的`context.Context`对象的上下文取消功能。该对象在*第二章*，《Go 语言精要》中有讨论，如果你需要复习。

只需在`main()`中创建一个带取消功能的`Context`对象，并将其传递给所有函数调用，我们就可以有效地取消所有正在进行的工作。当用户按下*Ctrl* + *C*时，这对于停止处理并进行清理非常有用。

我们将展示一种高级信号处理方法，程序的功能如下：

+   每 1 秒创建一个新临时文件，持续 30 秒

+   如果程序被取消，则清理文件

让我们首先创建一个处理信号的函数：

```
func handleSignal(cancel context.CancelFunc) chan os.Signal {
        out := make(chan os.Signal, 1)
        notify := make(chan os.Signal, 10)
        signal.Notify(
                notify,
                syscall.SIGINT,
                syscall.SIGTERM,
                syscall.SIGQUIT,
        )
        go func() {
                defer close(out)
                for {
                        sig := <-notify
                        switch sig {
                        case syscall.SIGINT, syscall.SIGTERM, syscall.SIGQUIT:
                                cancel()
                                out <- sig
                                return
                        default:
                                log.Println("unhandled signal: ", sig)
                        }
                }
        }()
        return out
}
```

这段代码执行以下操作：

+   创建一个名为`handleSignal()`的新函数

+   有一个名为`cancel`的参数，用于信号函数链停止处理

+   创建一个`out`通道，用来返回接收到的信号

+   创建一个`notify`通道，用于接收信号通知

+   创建一个 goroutine 来接收信号：

    +   如果信号是退出信号，则调用`cancel()`。

    +   返回通知我们退出的信号。

    +   如果是其他信号，仅记录日志。

现在，让我们创建一个函数来创建文件：

```
func createFiles(ctx context.Context, tmpFiles string) error {
        for i := 0; i < 30; i++ {
                if err := ctx.Err(); err != nil {
                        return ctx.Err()
                }
                _, err := os.Create(filepath.Join(tmpFiles, strconv.Itoa(i)))
                if err != nil {
                        return err
                }
                fmt.Println("Created file: ", i)
                time.Sleep(1 * time.Second)
        }
        return nil
}
```

这段代码执行以下操作：

+   循环 30 次，执行以下操作：

    +   检查我们的`ctx`是否已取消

    +   如果是错误，则返回该错误

    +   否则，在`tmpFiles`中创建文件

    +   每隔 1 秒创建一个新文件，持续 30 秒

这段代码将在`tmpFiles`中创建从`0`到`29`的文件，除非在写入文件时发生问题或`Context`被取消。

现在，我们需要一些代码来清理文件，如果我们收到`quit`信号。如果没有收到，文件将被保留：

```
func cleanup(tmpFiles string) {
        if err := os.RemoveAll(tmpFiles); err != nil {
                fmt.Println("problem doing file cleanup: ", err)
                return
        }
        fmt.Println("cleanup done")
}
```

这段代码执行以下操作：

+   使用`os.RemoveAll()`删除文件：

    +   同时移除临时目录

+   通知用户清理已完成

让我们将这些代码整合到我们的`main()`中：

```
func main() {
        tmpFiles, err := os.MkdirTemp("", "myApp_*")
        if err != nil {
                log.Println("error creating temp file directory: ", err)
                os.Exit(1)
        }
        fmt.Println("temp files located at: ", tmpFiles)
        ctx, cancel := context.WithCancel(context.Background())
        recvSig := handleSignal(cancel)
        if err := createFiles(ctx, tmpFiles); err != nil {
                cleanup(tmpFiles)
                select {
                case sig := <-recvSig:
                        if sig == syscall.SIGQUIT {
                                panic("SIGQUIT called")
                        }
                default:
                // Prevents waiting on a
                // signal if none exists.
                }
                log.Println("error: ", err)
                os.Exit(1)
        }
        fmt.Println("Done")
}
```

这段代码执行以下操作：

+   创建临时文件目录

+   创建一个根`Context`对象，`ctx`：

    +   `ctx`可以通过`cancel()`取消。

+   调用我们的`handleSignal()`来处理任何退出信号

+   执行我们的`createFiles()`函数：

    +   如果我们遇到错误，调用`cleanup()`。

    +   清理后，我们查看是否收到了信号，而不仅仅是错误。

    +   如果是信号并且是`SIGQUIT`，我们调用`panic()`。这是因为根据定义，`SIGQUIT`应该产生核心转储。

    +   如果只是一个错误，打印错误并返回错误代码。

此代码的完整内容可以在此仓库找到：[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/7/signals`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/7/signals)。

重要提示

代码必须通过`go build`构建并作为二进制文件运行。不能使用`go run`，因为`go`二进制文件在分叉我们的程序时会先拦截信号，导致我们的程序无法接收到信号。

在 Go 中，可以创建多种类型的核心转储，这些类型由环境变量控制。这个变量由`GOTRACEBACK`控制。你可以在这里阅读相关内容：[`pkg.go.dev/runtime#hdr-Environment_Variables`](https://pkg.go.dev/runtime#hdr-Environment_Variables)。

### 与 Cobra 一起的取消处理

当 Cobra 最初创建时，`context`包还不存在。在 2020 年，该程序经过修补，允许将`Context`对象传递给`cobra.Command`。但不幸的是，Cobra 生成器没有更新以生成必要的模板代码。

要像之前那样添加信号处理，我们只需进行几个修改 —— 首先是修改`main.go`文件：

```
func main() {
	ctx, cancel := context.WithCancel(context.Background())
	var sigCh chan os.Signal
	go func() {
		handleSignal(ctx, cancel)
	}()
	cmd.Execute(ctx)
	cancel()
	if sig := <-sigCh; sig == syscall.SIGQUIT {
		panic("SIGQUIT")
	}
}
```

我们还需要修改`handleSignal()`。你可以在这里看到这些更改：[`go.dev/play/p/F4SdN-xC-V_L`](https://go.dev/play/p/F4SdN-xC-V_L)

最后，你必须像这样修改`cmd/root.go`文件：

```
func Execute(ctx context.Context) {
        cobra.CheckErr(rootCmd.ExecuteContext(ctx))
}
```

现在我们有了信号处理。当编写我们的`Run`函数时，我们可以使用`cmd.Context()`来获取`Context`对象并检查是否有取消操作。

### 案例研究 – 缺乏取消处理导致死循环

早期的 Google 系统之一，旨在帮助自动化网络管理的系统叫做 Chipmunk。Chipmunk 包含网络上的权威数据，并根据这些数据生成路由器配置。

像大多数软件一样，Chipmunk 最初运行迅速并节省大量时间。随着网络每年增长十倍，设计和语言选择的局限性开始显现。

Chipmunk 是基于 Django 和 Python 构建的，并没有为横向扩展设计。当系统变得繁忙时，配置请求开始需要 30 分钟或更长时间。对于这些请求的计时器设置的最大时限为 30 分钟。

当生成接近这些限制时，设计存在一个致命缺陷 —— 如果请求被取消，取消信号不会传递给正在运行的配置生成器。

这意味着，如果生成过程花了 25 分钟，但在 1 分钟时被取消，那么生成器将继续工作 24 分钟，却没有人来接收这些工作。

当调用达到时间限制时，调用者会超时并重试。但生成器仍在处理前一个调用。这会导致级联失败，因为正在运行多个计算密集型计算，其中一些不再有接收者。这将推动新调用超过时间限制，因为 Python 的**全局解释器锁**（**GIL** [`wiki.python.org/moin/GlobalInterpreterLock`](https://wiki.python.org/moin/GlobalInterpreterLock)）阻止了真正的多线程，并且每次调用都会使 CPU 使用率加倍。

处理此类失败场景的关键之一是能够取消不再需要的作业。这就是为什么在整个函数调用链中传递`context.Context`对象并在逻辑点查找取消的重要性所在。这可以极大地减少达到阈值的系统负载，并减少**分布式拒绝服务**（**DDoS**）攻击的损害。

本节讨论了程序如何拦截操作系统信号并对这些信号进行响应。它提供了使用`Context`处理取消执行的示例，可用于任何应用程序。我们讨论了如何将其集成到使用 Cobra 生成器生成的程序中。

# 摘要

本章为你提供了编写基本和高级命令行应用程序的技能。我们讨论了如何使用`flag`包和`os`包接收来自用户的标志和参数形式的信号。我们还讨论了如何从`os.Stdin`读取数据，这使你可以将多个可执行文件串联起来进行处理。

我们还讨论了更高级的应用程序，特别是 Cobra 包及其附带的生成器二进制文件，用于构建带有帮助文本、快捷方式和子命令的高级命令行工具。

最后，我们讨论了如何处理信号并在这些信号的取消时提供清理工作。这包括一个关于为何取消可能至关重要的案例研究。

你在这里学到的技能将在未来编写工具时至关重要，从与本地文件的交互到与服务的交互。

在下一章中，我们将讨论如何自动化与本地设备或远程设备上命令行的交互。
