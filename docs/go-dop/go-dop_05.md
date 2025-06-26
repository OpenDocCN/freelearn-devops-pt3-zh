# 第四章：文件系统交互

每个开发者生活中的一个基本部分就是与**文件**的交互。文件代表了必须处理和配置的系统数据，缓存项可以被提供，还有许多其他用途。

Go 最强大的特点之一就是它对**文件接口**的抽象，使得一套通用工具能够与来自磁盘和网络的数据流交互。这些接口设定了一个共同标准，所有主要包都使用它们来导出数据流。不同接口间的转换变成了一个简单的任务，只需用必要的凭证访问**文件系统**即可。

与特定数据格式相关的包，如 CSV、JSON、YAML、TOML 和 XML，基于这些通用的文件接口构建。这些包使用标准库定义的接口从磁盘或 HTTP 流中读取这些类型的文件。

由于 Go 是跨平台的，你可能希望编写能够在不同操作系统上运行的软件。Go 提供了可以检测操作系统并处理操作系统路径差异的包。

本章将覆盖以下主题：

+   Go 中的所有 I/O 都是文件

+   读取和写入文件

+   流式文件内容

+   跨操作系统路径

+   跨操作系统文件系统

完成本章内容后，你应当掌握一套与各种介质中存储的数据交互的技能，这对你作为 DevOps 工程师的日常工作非常有帮助。

# Go 中的所有 I/O 都是文件

Go 提供了一个基于文件的**输入输出**（**I/O**）系统。这一点并不令人惊讶，因为 Go 是两位杰出工程师 Rob Pike 和 Ken Thompson 的心血结晶，他们在 *贝尔实验室* 时设计了 UNIX 和 Plan 9 操作系统——这两个系统几乎把所有事物都视作文件。

Go 提供了 `io` 包，其中包含与 I/O 基本操作进行交互的接口，如磁盘文件、远程文件和网络服务。

## I/O 接口

I/O 的基本单元是 `byte`，一个 8 位值。I/O 使用字节流来实现读写操作。对于某些 I/O，你只能按顺序从头到尾读取流（如网络 I/O）。一些 I/O，如磁盘，允许你在文件中进行*定位*。

我们在与字节流交互时进行的一些常见操作包括读取、写入、在字节流中定位某个位置，以及在完成工作后关闭流。

Go 为这些基本操作提供了以下接口：

```
// Read from an I/O stream.
type Reader interface {
     Read(p []byte) (n int, err error)
}
// Write to an I/O stream.
type Writer interface {
     Write(p []byte) (n int, err error)
}
// Seek to a certain spot in the I/O stream.
type Seeker interface {
     Seek(offset int64, whence int) (int64, error)
}
// Close the I/O stream.
type Closer interface {
     Close() error
}
```

`io` 包还包含复合接口，如 `ReadWriter` 和 `ReadWriteCloser`。这些接口在允许与文件或网络交互的包中很常见。通过这些接口，你可以使用常见的工具，无论底层是什么（例如本地文件系统、远程文件系统或 HTTP 连接）。

在本节中，我们已经了解到 Go 文件交互是基于`[]byte`类型的，并介绍了基本的 I/O 接口。接下来，我们将学习如何利用这些接口的方法来读取和写入文件。

# 读取和写入文件

在 DevOps 工具中最常见的场景是需要操作文件：*读取*、*写入*、*重新格式化*或*分析*这些文件中的数据。这些文件可以有多种格式——JSON、YAML、XML、CSV 等格式，应该都对你来说并不陌生。它们用于配置本地服务以及与云网络提供商进行交互。

在本节中，我们将介绍读取和写入整个文件的基础知识。

## 读取本地文件

让我们开始使用`os.Readfile()`函数读取本地磁盘上的配置文件：

```
data, err := os.ReadFile("path/to/file")
```

`ReadFile()`方法从其函数参数中读取文件位置并返回该文件的内容。返回值会存储在 data 变量中。如果文件无法读取，则会返回一个错误。有关错误处理的更多信息，请参阅*第二章*中关于*Go 中的错误处理*部分，*Go 语言基础*。

`ReadFile()`是一个辅助函数，它调用`os.Open()`并获取一个`io.Reader`。`io.ReadAll()`函数用于读取`io.Reader`的全部内容。

`data`是`[]byte`类型，因此如果你想将其作为`string`使用，可以通过` s := string(data)`简单地将其转换成字符串。这被称为*类型转换*，即我们将一种类型转换为另一种类型。在 Go 中，只有某些类型可以进行转换。关于转换规则的完整列表，可以在[`golang.org/ref/spec#Conversions.strings`](https://golang.org/ref/spec#Conversions.strings)中找到。可以使用`b := []byte(s)`将其转换回字节数组。其他大多数类型则需要使用一个叫做`strconv`的包来进行字符串转换（[`pkg.go.dev/strconv`](https://pkg.go.dev/strconv)）。

如果文件中表示的数据是 JSON、YAML 等常见格式，那么我们可以高效地检索和写入这些数据。

## 写入本地文件

写入本地磁盘最常见的方式是使用`os.Writefile()`。这个方法会将整个文件写入磁盘。如果必要，`WriteFile`会创建文件，并且如果文件已存在，会将其截断：

```
if err := os.WriteFile(“path/to/fi”, data, 0644); err != nil {
     return err
}
```

上述代码以 Unix 风格的权限`0644`将数据写入`path/to/fi`。如果你之前没接触过 Unix 风格的权限，快速查阅一下网络资料就能帮助你理解。

如果你的数据存储在`string`中，可以通过`[]byte(data)`轻松地将其转换为`[]byte`。`WriteFile()`是一个封装了`os.OpenFile()`的函数，它处理文件标志和模式，并在写入完成后关闭文件。

## 读取远程文件

远程文件的读取方式将取决于具体的实现。不过，这些概念仍然会基于我们之前讨论的`io`接口。

例如，假设我们想连接到一个存储在 HTTP 服务器上的文本文件，以收集常见的文本格式信息，比如应用程序的指标数据。我们可以连接到该服务器，并以一种与前面示例非常相似的方式检索该文件：

```
client := &http.Client{}
req, err := http.NewRequest("GET", "http://myserver.mydomain/myfile", nil)
if err != nil {
        return err
}
req = req.WithContext(ctx) 
resp, err := client.Do(req) 
cancel()
if err != nil {
        return err
}
// resp contains an io.ReadCloser that we can read as a file. 
// Let's use io.ReadAll() to read the entire content to data. 
data, err := io.ReadAll(resp.Body) 
```

正如你所看到的，获取`io.ReadCloser`的设置依赖于我们的 I/O 目标，但它返回的只是`io`包中的接口，我们可以在任何支持这些接口的函数中使用。

因为它使用了`io`接口，我们可以做一些非常巧妙的操作，比如将内容直接流式传输到本地文件，而不是将整个文件复制到内存中再写入磁盘。这种方式更快且更加节省内存，因为每次读取的内容都会立即写入磁盘。

让我们使用`os.OpenFile()`打开一个文件进行写入，并将内容从网页服务器流式传输到文件中：

```
flags := os.O_CREATE|os.O_WRONLY|os.O_TRUNC
f, err := os.OpenFile("path/to/file", flags, 0644)
if err != nil {
     return err
}
defer f.Close() 
if err := io.Copy(f, resp.Body); err != nil { 
    return err 
} 
```

`OpenFile()`是一个更复杂的文件打开方法，适用于你需要写入文件或更精确地控制文件操作的情况。如果你只需要从本地文件读取数据，应该使用`os.Open()`。这里的标志是标准的类 Unix 位掩码，其作用如下：

+   如果文件不存在，则创建该文件：`os.O_CREATE`。

+   向文件写入数据：`os.O_WRONLY`。

+   如果文件已存在，则截断文件而不是追加：`os.O_TRUNC`。

标志列表可以在这里找到：[`pkg.go.dev/os#pkg-constants`](https://pkg.go.dev/os#pkg-constants)。

`io.Copy()`从`io.Reader`读取并写入`io.Writer`，直到`Reader`为空。这将文件从 HTTP 服务器复制到本地磁盘。

在本节中，你学习了如何使用`os.ReadFile()`读取整个文件，如何将`[]byte`类型转换为`string`，以及如何使用`os.WriteFile()`将整个文件写入磁盘。我们还了解了`os.Open()`与`os.OpenFile()`之间的区别，并展示了如何使用像`io.Copy()`和`io.ReadAll()`这样的实用函数。最后，我们学习了 HTTP 客户端如何将数据流暴露为`io`接口，并通过这些相同的工具读取数据。

接下来，我们将查看如何将这些文件接口作为流来操作，而不是一次性读取和写入整个文件。

# 流式传输文件内容

在前面的章节中，我们学习了如何使用`os.ReadFile()`和`os.WriteFile()`以大块数据进行读取和写入。

当文件较小时，这种方式效果很好，通常在进行 DevOps 自动化时会遇到这种情况。然而，有时我们想读取的文件非常大——在大多数情况下，你不会希望将一个 2 GiB 的文件全部读入内存。在这种情况下，我们希望以可管理的块流式传输文件内容，这样我们可以在保持较低内存使用的同时进行操作。

这种最基本的版本在上一节中已经展示过。在那里，我们使用了两个流来复制文件：`io.ReadCloser`来自 HTTP 客户端，`io.WriteCloser`用于写入本地磁盘。我们使用了`io.Copy()`函数来将网络文件复制到磁盘文件。

Go 的`io`接口还允许我们流式处理文件，复制内容，搜索内容，操作输入输出等。

## Stdin/Stdout/Stderr 只是文件

在本书中，你会看到我们使用`fmt.Println()`或`fmt.Printf()`这两个来自`fmt`包的函数来向控制台写入数据。这些函数实际上是向表示终端的文件读写数据。

这些函数使用一个名为`os.Stdout`的`io.Writer`。当我们在`log`包中使用相同的函数时，通常是向`os.Stderr`写入数据。

你可以使用我们一直在使用的相同接口来读写其他文件，也来读写这些文件。当我们想要复制一个文件并将其内容输出到终端时，我们可以这样做：

```
f, err := os.Open("path/to/file")
if err != nil {
     return err
}
if err := io.Copy(os.Stdout, f); err != nil {
     return err
}
```

虽然我们不会详细探讨，`os.Stdin`只是一个`io.Reader`。你可以使用`io`和`bufio`包从中读取数据。

## 从流中读取数据

如果我们想读取一个表示用户记录的流，并通过通道返回它们，该怎么办呢？

假设记录是简单的`<user>:<id>`文本，每条记录由换行符（`\n`）分隔。这些记录可能存储在 HTTP 服务器或本地磁盘上。这对我们来说并不重要，因为它只是一个接口背后的流。假设我们接收到这个流作为一个`io.Reader`。

首先，我们将定义一个`User`结构体：

```
type User struct{
  Name string
  ID int
}
```

接下来，让我们定义一个函数，来拆分我们接收到的每一行：

```
func getUser(s string) (User, error) {
     sp := strings.Split(s, ":")
     if len(sp) != 2 {
          return User{}, fmt.Errorf("record(%s) was not in the correct format", s)
    } 
    id, err := strconv.Atoi(sp[1]) 
     if err != nil {
          return User{}, fmt.Errorf("record(%s) had non-numeric ID", s)
     }
     return User{Name: strings.TrimSpace(sp[0]), ID: id}, nil
}
```

`getUser()`接收一个字符串并返回一个`User`。我们使用`strings`包的`Split()`函数将字符串按`:`作为分隔符拆分成`[]string`。

`Split()`应该返回两个值；如果不是，我们将返回一个错误。

由于我们在拆分字符串，用户 ID 被存储为`string`类型。但我们希望在`User`记录中使用整数值。在这里，我们可以使用`strconv`包的`Atoi()`方法，将字符串形式的数字转换为整数。如果它不是整数，则说明输入无效，我们将返回一个错误。

现在，让我们创建一个函数，读取流并将`User`记录写入通道：

```
func decodeUsers(ctx context.Context, r io.Reader) chan User {
     ch := make(chan User, 1)
     go func() {
          defer close(ch)
          scanner := bufio.NewScanner(r)
          for scanner.Scan() {
               if ctx.Err() != nil {
                    ch <- User{err: ctx.Err()}
                    return
               }
               u, err := getUser(scanner.Text())
               if err != nil {
                    u.err = err
                    ch <- u
                    return
               }
               ch <- u
          }
     }()
     return ch
}
```

这里，我们使用的是`bufio`包的`Scanner`类型。`Scanner`允许我们获取一个`io.Reader`并扫描它，直到找到分隔符。默认情况下，分隔符是`\n`，但你可以使用`.Split()`方法来更改它。`Scan()`将在读取器输出结束前一直返回`true`。请注意，当`io.Reader`到达流的末尾时，会返回一个错误`io.EOF`。

每次调用`Scan()`后，扫描器会存储读取的字节，你可以通过`.Text()`方法将其作为`string`提取出来。`.Text()`中的内容会在每次调用`.Scan()`时发生变化。同时，请注意，我们会检查`Context`对象，如果它被取消，则停止执行。

我们将该`string`的内容传递给我们之前定义的`getUser()`。如果我们收到一个`error`，我们将把它返回给`User`记录，以通知调用者错误。否则，我们返回包含所有信息的`User`记录。

现在，让我们对一个文件进行调用：

```
f, err := os.Open("path/to/file/with/users")
if err != nil {
     return err
}
defer f.Close()  
for user := range decodeUsers(ctx, f) {
     if user.err != nil {
          fmt.Println("Error: ", user.err)
          return err
     }
     fmt.Println(user)
}
```

在这里，我们打开磁盘上的文件并将其传递给`decodeUsers()`。我们从*输出通道*接收一个`User`记录，并在读取文件流的同时并发地将用户打印到屏幕上。

我们本可以通过`http.Client`打开文件并将其传递给`decodeUsers()`，而不是使用`os.Open()`。完整的代码可以在这里找到：[`play.golang.org/p/OxehTsHT6Qj`](https://play.golang.org/p/OxehTsHT6Qj)。

## 向流写入数据

向流写入数据更加简单——我们只需将`User`转换为`string`并将其写入`io.Writer`。如下所示：

```
func writeUser(ctx context.Context, w io.Writer, u User) error {
     if ctx.Err() != nil {
          return ctx.Err()
     }
     if _, err := w.Write([]byte(user.String())); err != nil {
          return err
     }
     return nil
}
```

在这里，我们接受一个`io.Writer`，它代表了写入的目标位置，以及一个我们想写入该输出的`User`记录。我们可以使用它将数据写入磁盘上的文件：

```
f, err := os.OpenFile("file", flags, 0644); err != nil{
     return err
}
defer f.Close() 
for i, u := range users {
     // Write a carriage return before the next entry, except
     // the first entry.
     if i != 0 {
          if err := w.Write([]byte("\n")); err != nil {
               return err
          }
     }
     if err := writeUser(ctx, w, u); err != nil {
          return err
     }
}
```

在这里，我们打开了本地磁盘上的一个文件。当我们包含的函数（未显示）返回时，文件将被关闭。然后，我们将存储在变量 users（`[]Users`）中的`User`记录逐个写入文件。最后，我们在每条记录之前（除了第一条记录）写入了一个回车符`("\n")`。

您可以在这里查看实际演示：[`play.golang.org/p/bxuFyPT5nSk`](https://play.golang.org/p/bxuFyPT5nSk)。我们提供了一个使用通道的流式版本，您可以在这里找到：[`play.golang.org/p/njuE1n7dyOM`](https://play.golang.org/p/njuE1n7dyOM)。

在下一部分，我们将学习如何使用`path/filepath`包编写适用于多个操作系统的软件，这些操作系统使用不同的路径分隔符。

# 操作系统无关的路径处理

Go 语言的一个最大优势是其多平台支持。开发人员可以在 Linux 工作站上开发，并将相同的 Go 程序重新编译为本地代码后，在 Windows 服务器上运行。

开发跨多个操作系统运行的软件时，*访问文件*是一个难点。每个操作系统的路径格式稍有不同。最明显的例子是不同操作系统的文件分隔符：Windows 上是`\`，而类 Unix 系统上是`/`。更不明显的是如何在特定操作系统上转义特殊字符，甚至在 Unix 类操作系统之间也可能有所不同。

`path/filepath`包提供了访问函数的功能，允许您处理本地操作系统的路径。这不应与根`path`包混淆，后者看起来类似，但处理的是更通用的 URL 样式路径。

## 我正在运行哪个操作系统/平台？

虽然我们将讨论如何使用无关操作系统的函数获取文件访问权限并执行路径处理，但了解您运行的操作系统仍然非常重要。您可能会根据运行的操作系统使用不同的文件位置。

使用`runtime`包，您可以检测到您运行的操作系统和平台：

```
fmt.Println(runtime.GOOS) // linux, darwin, ...
fmt.Println(runtime.GOARCH) // amd64, arm64, ...
```

这将为您提供正在运行的操作系统。我们可以使用**go tool dist list**命令打印出 Go 支持的当前操作系统类型和硬件架构列表。

## 使用 filepath

使用**filepath**，在处理路径时可以忽略所在操作系统的路径规则。路径被分为以下几个部分：

+   路径中的**目录**

+   路径中的**文件**

文件路径的最终目录或文件称为**基础路径**。你的二进制文件运行所在的路径称为**工作目录**。

### 连接文件路径

假设我们想访问一个名为`config.json`的配置文件，它存储在与我们的二进制文件相同目录下的`config/`目录中。让我们使用`os`和`path/filepath`以一种适用于所有操作系统的方式来读取该文件： 

```
wd, err := os.Getwd() 
if err != nil { 
    return err 
} 
content, err := os.ReadFile(filepath.Join(wd, "config", "config.json"))
```

在这个例子中，我们首先获取工作目录。这允许我们相对于我们的二进制文件所在位置进行调用。

`filepath.Join()` 允许我们将路径的各个组成部分连接成一个单一的路径。它会为你填充操作系统特定的目录分隔符，并使用本地的路径规则。在类 Unix 系统上，可能是 `/home/jdoak/bin/config/config.json`，而在 Windows 上，则可能是 `C:\Documents and Settings\jdoak\go\bin\config\config.json`。

### 拆分文件路径

在某些情况下，根据路径分隔符将文件路径拆分开来是很重要的。`filepath` 提供了以下功能：

+   `Base()`: 返回路径的最后一个元素

+   `Ext()`: 返回文件扩展名（如果有）

+   `Split()`: 返回分割后的目录和文件

我们可以使用这些来获取路径的各个部分。当我们希望将文件复制到另一个目录并保留文件名时，这可能会很有用。

让我们将一个文件从其位置复制到我们操作系统的**TMPDIR**：

```
fileName := filepath.Base(fp) 
if fileName == "." { 
    // Path is empty 
    return nil 
}
newPath := filepath.Join(os.TempDir(), fileName) 

r, err := os.Open(fp) 
if err != nil { 
    return err 
}
defer r.Close() 

w, err := os.OpenFile(newPath, O_WRONLY | O_CREATE, 0644) 
if err != nil { 
    return err 
}
defer w.Close()
// Copies the file to the temporary file.
_, err := io.Copy(w, r)
return err
```

现在，是时候看一下可以用来引用文件的不同路径选项，以及`filepath`包如何帮助你了。

## 相对和绝对路径处理

访问文件系统时有两种类型的路径处理方式：

+   **绝对路径**: 从根目录到文件的路径处理

+   **相对路径**: 在文件系统中从当前位置进行路径处理

在开发过程中，将相对路径转换为绝对路径及其反向转换通常很方便。

`filepath` 提供了几个函数来帮助处理这些问题：

+   `Abs()`: 返回绝对路径。如果它不是绝对路径，则返回工作目录以及路径。

+   `Rel()`: 返回路径相对于基本路径的相对路径。

我们将让你自己尝试使用这些功能。

在本节中，我们学习了如何使用`path/filepath`和`runtime`包处理不同操作系统的文件路径。我们介绍了`runtime.GOOS`来帮助您检测用户正在使用的操作系统，`os.Getwd()`来确定程序在文件系统中的位置。我们还介绍了`os.TempDir()`来定位您的操作系统中用于临时文件的位置。最后，我们学习了`path/filepath`中的函数，这些函数允许您在不考虑操作系统的情况下组合和拆分文件路径，并输出特定于操作系统的结果。

接下来，我们将看看 Go 的新 `io/fs` 包，它是在版本 1.16 中引入的。它通过类似于 `io` 对文件的处理方式，引入了新的接口来抽象文件系统。

# 操作系统无关的文件系统

在最新的 Go 版本中，最令人兴奋的两项新功能是新的 `io/fs` 和 `embed` 包，它们是在 **Go 1.16** 中引入的。

尽管我们已经展示了通过 `os` 包访问本地文件系统的通用方式，并通过 `filepath` 进行通用的文件路径操作，但我们还没有看到访问整个文件系统的通用方法。

在云计算时代，文件也很可能存储在远程数据中心的文件系统中，比如 Microsoft Azure 的 Blob 存储、Google Cloud 的 Filestore 或 Amazon AWS 的 EFS，就像它们存储在本地磁盘上一样。

这些文件系统每个都有一个用于在 Go 中访问文件的客户端，但它们是特定于该网络服务的。我们不能像处理本地文件系统一样处理这些文件。`io/fs` 旨在提供一个基础，帮助解决这个问题。

另一个问题是许多文件必须与二进制文件一起打包，通常是在容器定义中。这些文件在程序的生命周期内不会改变。将它们包含在二进制文件中并通过文件系统接口访问会更方便。一个需要图像、HTML 和 CSS 文件的简单 Web 应用程序就是这种用例的一个典型示例。新的 `embed` 包旨在解决这个问题。

## io.fs 文件系统

我们新的 `io/fs` 文件系统导出了可以由文件系统提供者实现的接口。根接口 `FS` 的定义最简单：

```
type FS interface {
     Open(name string) (File, error)
}
```

这让你可以打开任何文件，其中 `File` 被定义如下：

```
type File interface {
     Stat() (FileInfo, error)
     Read([]byte) (int, error)
     Close() error
}
```

这提供了最简单的文件系统。你可以打开路径中的文件，并获得文件的信息或读取文件。由于文件系统在功能上有差异，这是所有给定文件系统之间唯一共享的功能。

一个 `FS`（如 `ReadDirFS` 和 `StatFS`），它允许进行文件遍历并提供目录信息。注意，FS 对象缺少可写性。你必须自己提供一个，因为 Go 作者没有将其定义为标准库的一部分。

## embed

`embed` 包允许你使用 `//go:embed` 指令将文件直接嵌入到二进制文件中。

`embed` 可以通过三种方式嵌入文件，如下所示：

+   作为字节

+   作为一个字符串

+   转换为 `embed.FS`（它实现了 `fs.FS`）

前两个功能通过将指令放置在特定的变量类型上完成：

```
import _ "embed" 
//go:embed hello.txt
var s string
//go:embed world.txt
var b []byte
```

`//go:embed hello.txt` 表示 Go 指令，指示编译器获取名为 `hello.txt` 的文件并将其存储在变量中。

在`import`行上的`_`指示编译器忽略我们没有直接使用`embed`的事实。这称为*匿名导入*，即我们需要加载一个包但不直接使用其功能。没有`_`时，如果未使用导入的包，我们将收到编译错误。

使用 `embed.FS` 的最终方法在你希望将多个文件嵌入文件系统时非常有用：

```
// The lines beginning with //go: are not comments, but compiler directives
//go:embed image/*
//go:embed index.html
var content embed.FS
```

现在我们有一个 `fs.FS`，它存储了我们 `image` 目录中的所有文件和 `index.html` 文件。这些文件在我们发布二进制文件时不再需要包含在容器文件系统中。

## 遍历我们的文件系统

`io/fs` 包提供了一种与文件系统无关的遍历方法，*前提是文件系统支持该功能*。在前面的示例中，我们有一个嵌入式文件系统中的目录，里面存放着图像文件。我们可以使用目录遍历器打印出所有 `.jpg` 文件：

```
err := fs.WalkDir(
     content,
     ".",
     func(path string, d fs.DirEntry, err error) error {
          if err != nil {
               return err
          }
          if !d.IsDir() && filepath.Ext(path) == ".jpg" {
               fmt.Println("jpeg file: ", path)
          }
          return nil
     },
)
```

上述函数遍历了我们嵌入式文件系统（content）的目录结构（从根目录 `"."` 开始），并调用了已定义的函数，将文件路径、目录条目和错误（如果有）传递给它。

在我们的函数中，如果文件不是目录且具有 `.jpg` 扩展名，我们只是打印文件的路径。

那么，使用 `io/fs` 访问其他类型文件系统的包怎么办呢？

## io/fs 的未来

在写作时，`io/fs` 的主要用户是 `embed`。然而，我们开始看到第三方包实现了这个接口。

`absfs` 为他们的 `boltfs/memfs/os` 文件系统包提供了一个 `io.FS` 钩子（[`github.com/absfs`](https://github.com/absfs)）。这些包中的几个封装了流行的 `afero` 文件系统包（[`github.com/spf13/afero`](https://github.com/spf13/afero)）。Azure 有一个非官方的包，支持 Blob 存储（[`github.com/element-of-surprise/azfs`](https://github.com/element-of-surprise/azfs)）。

还有一些包可以访问**Redis**、**GroupCache**、**memfs**、本地文件系统以及 [`github.com/gopherfs/fs`](https://github.com/gopherfs/fs) 上的工具支持。

注意

`github.com/element-of-surprise` 和 [`github.com/gopherfs`](https://github.com/gopherfs) 由作者拥有。

在这一节中，你了解了 Go 的 `io/fs` 包，以及它如何成为与文件系统交互的标准。你还学会了如何使用 `embed` 包将文件嵌入到二进制文件中，并通过 `io/fs` 接口访问它们。

### 我们只是触及了表面

我强烈建议你阅读标准库的 **GoDoc** 页面，以便熟悉其功能。以下是本章涉及的 GoDocs。在这里，你可以找到许多处理文件的有用工具：

+   文件接口和基本 I/O 函数：[`pkg.go.dev/io`](https://pkg.go.dev/io)

+   缓冲 I/O 包：[`pkg.go.dev/bufio`](https://pkg.go.dev/bufio)

+   将字符串转换为/从其他类型转换： [`pkg.go.dev/strconv`](https://pkg.go.dev/strconv)

+   操作字符串的包：[`pkg.go.dev/strings`](https://pkg.go.dev/strings)

+   操作字节的包： [`pkg.go.dev/bytes`](https://pkg.go.dev/bytes)

+   操作操作系统交互的包：[`pkg.go.dev/os`](https://pkg.go.dev/os)

+   用于正斜杠路径（如 URL）的包： [`pkg.go.dev/path`](https://pkg.go.dev/path)

+   文件路径包：[`pkg.go.dev/path/filepath`](https://pkg.go.dev/path/filepath)

+   文件系统接口：https://pkg.go.dev/io/fs

+   嵌入式文件系统：https://pkg.go.dev/embed

在这一节中，我们学习了如何使用`io`接口将数据流进流出文件，以及`os`包的`Stdin/Stdout/Stderr`实现，用于读取和写入程序的输入/输出。我们还学习了如何使用`bufio`包按分隔符读取数据，以及如何使用`strings`包拆分字符串内容。

# 概述

本章为你提供了 Go 语言中处理文件 I/O 的基础。你学习了`io`包及其文件抽象，并了解了如何将文件读写到磁盘。接着，你学习了如何流式传输文件内容，以便与网络合作并提高内存效率。然后，你了解了`path/filepath`包，它可以帮助你处理多种操作系统。最后，你了解了 Go 的文件系统无关接口，用于与任何文件系统进行交互，从新的`embed`文件系统开始。

在下一章，你将学习如何使用流行的 Go 包与常见数据类型和存储交互。在那里，你将需要依赖本章中的文件和文件系统包来与数据类型进行交互。

与数据和存储系统的交互对于 DevOps 工作至关重要。它使我们能够读取和更改软件配置，存储数据并使其可搜索，要求系统代我们执行工作，并生成报告。

那么，让我们开始吧！
