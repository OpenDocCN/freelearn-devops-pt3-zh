# 第八章：自动化命令行任务

大多数工作最初都是由工程师执行的某种手动操作。随着时间的推移，这些操作应成为已记录的程序，并且具有最佳实践，最终，这项工作应该由软件来完成，软件可以根据最佳实践高效地执行，而这种效率只有机器才能提供。

**开发运维**（**DevOps**）工程师的核心任务之一是自动化这些任务。任务可能从简单的运行几条命令到在成千上万台机器上更改配置。

自动化系统通常需要通过命令行操作系统并调用其他本地工具（**操作系统**（**OS**））。这可能包括使用**RPM 包管理器**（**RPM**）/**Debian 包**（**dpkg**）安装软件包，使用常见工具获取系统统计信息，或配置网络路由器。

一名 DevOps 工程师可能希望在本地自动化通常手动执行的一系列步骤（例如，自动化 Kubernetes 的`kubectl`工具），或者远程在数百台机器上同时执行命令。本章将讨论如何使用 Go 完成这些任务。

在本章中，您将学习如何在本地机器上执行命令行工具以实现自动化目标。要访问远程机器，我们将学习如何使用**安全外壳**（**SSH**）和 Expect 包。但了解如何调用机器上的可执行文件只是技能的一部分。我们还将讨论更改的结构以及如何安全地进行并发更改。

本章将涵盖以下主题：

+   使用`os/exec`来自动化本地更改

+   在 Go 中使用 SSH 自动化远程更改

+   设计安全的并发更改自动化

+   编写系统代理

# 技术要求

本章要求您安装最新的 Go 工具，并可以访问 Linux 系统以运行我们创建的任何服务二进制文件。本章中的所有工具都将面向控制 Linux 系统，因为它是最受欢迎的云计算平台。

对于远程机器访问要求，远程 Linux 系统需要运行 SSH，以允许远程连接。

要在本章的最后部分使用系统代理，您还需要使用已安装`systemd`的 Linux 发行版。大多数现代发行版使用`systemd`。

本章中编写的代码可以在这里找到：

[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/8`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/8)

# 使用`os/exec`来自动化本地更改

自动化执行本地机器上的工具可以为最终用户提供一系列好处。其中第一个是它可以减少团队的繁琐工作。DevOps 和 **站点可靠性工程师** (**SRE**) 的主要目标之一是消除重复的手动过程。那段时间可以用来读一本好书（比如这本），整理袜子抽屉，或者处理下一个问题。第二个好处是可以消除过程中的人为错误。打错字或复制粘贴错误都很容易发生。最后，它是大规模运营的核心基础。通过本地自动化结合书中详细描述的其他技术，可以在大规模上进行变更。

自动化生命周期通常分为三个阶段，从手动工作到自动化，如下所示：

1.  第一阶段涉及由经验丰富的工程师手动执行命令。虽然这本身不是自动化，但它启动了一个以某种形式的自动化结束的循环。

1.  第二阶段通常涉及将这些步骤记录下来，以便文档化过程，允许多人分担工作。这可能是一个 `git` 仓库。

1.  第三阶段通常是编写脚本使任务可重复执行。

一旦公司变得更大，这些阶段通常会合并为开发一个服务，在识别到需要时以完全自动化的方式处理任务。

一个很好的例子可能是在 Kubernetes 集群上部署 pods 或向 Kubernetes 配置中添加新的 pod 配置。这些操作是通过调用命令行应用程序如 `kubectl` 和 `git` 来驱动的。这些类型的工作一开始是手动的；最终，它们会被文档化，并最终以某种方式实现自动化。某个时候，这可能会转移到 **持续集成/持续部署** (**CI/CD**) 系统中，由其为你处理这些任务。

在本地自动化工具的关键是 `os/exec` 包。该包允许执行其他工具并控制它们的 `STDIN` / `STDOUT` / `STDERR` 流。

让我们更仔细地看一下。

## 确定必需工具的可用性

在编写调用系统上其他应用程序的应用程序时，关键是要在开始执行命令之前，确定所需的工具是否在系统中可用。没有什么比在执行过程中发现缺少关键工具更糟糕的了。

`exec` 包提供了 `LookPath()` 函数来帮助确定一个二进制文件是否存在。如果只提供了二进制文件的名称，则会查阅 `PATH` 环境变量，并在这些路径中搜索该二进制文件。如果名称中包含 `/`，则仅查阅该路径。

假设我们正在编写一个工具，需要安装 `kubectl` 和 `git` 才能正常工作。我们可以通过执行以下代码来测试这些工具是否在我们的 `PATH` 变量中可用：

```
const (
    kubectl = "kubectl"
    git = "git"
)
_, err := exec.LookPath(kubectl)
if err != nil {
    return fmt.Errorf("cannot find kubectl in our PATH")
}
_, err := exec.LookPath(git)
if err != nil {
    return fmt.Errorf("cannot find git in our PATH")
}
```

这段代码执行以下操作：

+   为我们的二进制文件名称定义常量

+   使用 `LookPath()` 来判断这些二进制文件是否存在于我们的 PATH 变量中

在这段代码中，如果我们找不到工具，就直接返回一个错误。还有其他选择，比如尝试通过本地包管理器安装这些工具。根据我们的环境配置，我们可能希望测试部署的版本，并且仅在版本兼容时才继续。

我们来看一下如何使用 `exec.CommandContext` 类型来调用二进制文件。

### 使用 `exec` 包执行二进制文件

`exec` 包允许我们使用 `exec.Cmd` 类型执行二进制文件。要创建其中一个，我们可以使用 `exec.CommandContext()` 构造函数。它接收要执行的二进制文件名称和传递给二进制文件的参数，如下面的代码片段所示：

```
cmd := exec.CommandContext(ctx, kubectl, "apply", "-f", config)
```

这创建了一个命令，将运行 `kubectl` 工具的 `apply` 函数，并指示它应用存储在 `config` 变量中的路径上的配置。

这个命令的语法是不是很熟悉？它应该是！`kubectl` 是使用我们上一章介绍的 Cobra 编写的！

我们可以通过多种方法执行 `cmd` 上的这个命令，如下所示：

+   `.CombinedOutput()`：运行命令并返回 `STDOUT` 和 `STDERR` 的合并输出。

+   `.Output()`：运行命令并返回 `STDOUT` 的输出。

+   `.Run()`：运行程序并等待其退出。若有问题，则返回错误。

+   `.Start()`：运行命令但不阻塞。用于你希望在命令运行时与其交互的场景。

`.CombinedOuput()` 和 `.Output()` 是启动程序最常见的方法。用户在终端中看到的输出通常来自 `STDOUT` 和 `STDERR`。选择使用哪一个取决于你希望如何响应程序的输出。

`.Run()` 用于当你只需要知道退出状态而不需要任何输出时。

使用 `.Start()` 有两个主要原因，如下所述：

+   需要在 `STDIN` 上对 `STDOUT` 的输出做出响应。

+   程序执行需要一段时间，你希望将其内容输出到屏幕上，而不是等待程序完成。

如果你需要对程序的输出在 `STDIN` 上做出响应，使用 Google 的 `goexpect` 包（[`github.com/google/goexpect`](https://github.com/google/goexpect)）或 Netflix 的 `go-expect` 包（[`github.com/Netflix/go-expect`](https://github.com/Netflix/go-expect)）可能是更好的选择。这些包延续了**工具命令语言**（**TCL**）Expect 扩展的光荣传统（[`en.wikipedia.org/wiki/Expect`](https://en.wikipedia.org/wiki/Expect)），并将其移植到其他语言中。

让我们编写一个简单的程序，测试我们在子网内登录主机的能力。我们将使用`ping`工具和`ssh`客户端程序来测试连通性。我们将依赖你的主机识别你的 SSH 密钥（这里不使用密码认证，因为那更复杂）。最后，我们将在远程机器上使用`uname`来确定操作系统。代码在以下片段中有所展示：

```
func hostAlive(ctx context.Context, host net.IP) bool {
        cmd := exec.CommandContext(ctx, ping, "-c", "1", "-t", "2", host.String())
        if err := cmd.Run(); err != nil {
            return false
        }
        return true
}
```

注意

`uname`是一个在类 Unix 系统中可用的程序，用于显示当前操作系统及其运行硬件的信息。只有 Linux 和 Darwin 机器可能拥有`uname`。由于 SSH 只是一个连接协议，我们可能会得到一个错误。此外，某些 Linux 发行版可能没有安装`uname`。不同版本的常见工具在类似平台上可能有细微差别。Linux 的`ping`和 OS X 的`ping`工具共享一些标志，但也有不同的标志。Windows 通常有完全不同的工具来完成相同的任务。如果你想通过一个使用`exec`的工具支持所有平台，你需要使用构建约束（[`pkg.go.dev/cmd/go#hdr-Build_constraints`](https://pkg.go.dev/cmd/go#hdr-Build_constraints)）或使用`runtime`包来在不同的平台上运行不同的工具。

这段代码执行了以下操作：

+   创建一个`*Cmd`来对主机进行 ping 操作

    +   `-c 1`发送一个单独的`-t 2`会在 2 秒后超时。

+   运行命令

    +   如果出错，则说明 ping 操作失败。

    +   否则，主机响应了 ping 请求。

现在让我们使用`ssh`工具向远程机器发送一个命令，如下所示：

```
func runUname(ctx context.Context, host net.IP, user string) 
(string, error) {
        if _, ok := ctx.Deadline(); !ok {
                var cancel context.CancelFunc
                ctx, cancel = context.WithTimeout(ctx, 5*time.Second)
                defer cancel()
        }
        login := fmt.Sprintf("%s@%s", user, host)
        cmd := exec.CommandContext(
                ctx,
                ssh,
                "-o StrictHostKeyChecking=no",
                "-o BatchMode=yes",
                login,
                "uname -a",
        )
        out, err := cmd.CombinedOutput()
        if err != nil {
                return "", err
        }
        return string(out), nil
}
```

这段代码执行了以下操作：

+   如果`ctx`没有设置，设置 5 秒的超时

+   创建一个`user@host`的登录行

+   创建一个*`CMD`，发出命令：`ssh user@host` "`uname -a`"

    +   `StrictHostKeyChecking`选项会自动添加主机密钥。

    +   `BatchMode`选项防止提示输入密码。

+   运行命令并捕获来自`STDOUT`的输出

    +   如果成功，它会运行`uname -a`并返回输出。

    +   主机必须拥有用户的 SSH 密钥才能正常工作。

        +   密码认证需要`sshpass`工具或 Expect 包。

我们需要一个类型来存储我们收集的数据。让我们创建它，如下所示：

```
type record struct{
    Host net.IP
    Reachable bool
    LoginSSH bool
    Uname string
}
```

现在，我们需要一些代码来接收一个包含**互联网协议**（**IP**）地址的通道，这些地址需要扫描。我们希望并行执行，因此我们将使用 goroutines，如下片段所示：

```
func scanPrefixes(ipCh chan net.IP) chan record {
        ch := make(chan record, 1)
        go func() {
                defer close(ch)
                limit := make(chan struct{}, 100)
                wg := sync.WaitGroup{}
                for ip := range ipCh {
                        limit <- struct{}{}
                        wg.Add(1)
                        go func(ip net.IP) {
                                defer func() { <-limit }()
                                defer wg.Done()
                                ctx, cancel := context.WithTimeout(context.Background(), 3*time.Second))
                                defer cancel()
                                rec := record{Host: ip}
                                if hostAlive(ctx, ip) {
                                        rec.Reachable = true
                                }
                                ch <- rec
                        }(ip)
                }
                wg.Wait()
        }()
        return ch
}
```

这段代码执行了以下操作：

+   接收一个`net.IP`类型的通道

+   创建一个通道来放置记录

+   启动一个 goroutine 来执行所有扫描操作

    +   延迟关闭我们的输出通道

    +   遍历所有进入通道的 IP 地址

    +   使用`limit`通道限制最多并发 100 个 ping 操作

    +   为每个 ping 操作启动一个 goroutine

        +   完成后减少限流器的数量

        +   为我们的 ping 操作设置 2 秒的超时时间

        +   调用我们的`hostAlive()`函数

        +   将结果输出到我们的`ch`输出通道

    +   等待所有的 ping 命令完成，使用`WaitGroup`

+   返回通道

我们现在有了一个异步并行 ping 主机并将结果放入通道的函数。

我们的`ssh`函数的函数签名与`scanPrefixes`相似，如下所示：

```
func unamePrefixes(user string, recs chan record) chan record
```

为了简洁起见，我们不会在这里包含代码，但你可以在练习结束时提供的代码库中查看它。

这些是`scanPrefixes()`和`unamePrefixes()`之间的主要区别：

+   我们接收到一个`record`的通道，这是`scanPrefixes()`的输出。

+   如果`rec.Reachable`为`false`，我们会将`rec`直接放入输出通道，而不将操作系统信息添加到字段中。

+   否则，我们调用`runUname()`而不是`hostAlive()`。

现在，让我们设置`main()`函数，具体如下：

```
func main() {
    _, err := exec.LookPath(ping)
    if err != nil {
        log.Fatal("cannot find ping in our PATH")
    }
    _, err := exec.LookPath(ssh)
    if err != nil {
        log.Fatal("cannot find ssh in our PATH")
    }
    if len(os.Args) != 2 {
        log.Fatal("error: only one argument allowed, the network CIDR to scan")
    }
    ipCh, err := hosts(os.Args[1])
    if err != nil {
            log.Fatalf("error: CIDR address did not parse: %s", err)
    }
    u, err := user.Current()
    if err != nil {
        log.Fatal(err)
    }
```

这段代码做了以下几件事：

+   检查我们的二进制文件是否存在于路径中

+   检查我们是否有正确数量的参数，即`1`

    +   我们检查`len(os.Args) == 2`，因为第一个参数是二进制文件名。

+   检索传递给参数的网络中 IP 的通道

    +   `hosts()`函数的实现没有在这里详细说明，但你可以在代码库中找到它。

+   获取当前用户的登录名

现在，我们需要扫描我们的前缀并通过进行登录并获取`uname`输出来并行处理结果，具体如下：

```
    scanResults := scanPrefixes(ipCh)
    unameResults := unamePrefixes(u.Username, scanResults)
    for rec := range unameResults {
        b, _ := json.Marshal(rec)
        fmt.Printf("%s\n", b)
    }
}
```

这段代码做了以下几件事：

+   发送 IP 的通道到`scanPrefixes()`

+   在`scanResults`上接收结果

+   将结果通道发送到`unamePrefixes()`中

+   打印`STDOUT`

这段代码的关键在于在`scanPrefixes()`和`unamePrefixes()`中的`for range`循环中读取通道。当所有的 IP 都已发送，`ipCh`将被关闭。那将停止`scanPrefixes()`中的`for range`循环，进而关闭它的输出通道。这会导致`unamePrefixes`看到关闭并关闭它的输出通道。这又会关闭`for rec := range unameResults`循环并停止打印。

使用这种链式并发模型，我们将同时扫描最多 100 个 IP，通过 SSH 连接最多 100 个主机，并将结果同时打印到屏幕上。

我们已经将`uname -a`的输出存储在我们的`record`变量中，但它是未经解析的格式。我们可以使用词法分析器/解析器或`struct`。如果你需要使用执行的二进制文件的输出，建议寻找可以输出结构化格式（如 JSON）的工具，而不是自己进行解析。

你可以在以下链接中查看这段代码：

[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/8/scanner`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/8/scanner)

### 使用 exec 包的注意事项

使用`exec`时需要注意一些问题。一个主要的问题是，如果被调用的二进制文件控制了终端。例如，`ssh`会这么做，从用户那里获取密码。我们在示例中抑制了这一行为，但发生这种情况时，它会绕过你正在读取的正常 STDOUT。

这种情况发生在有人使用终端模式时。在这些情况下，如果必须处理这种情况，你将需要使用 `goexpect` 或 `go-expect`。通常来说，这是一个你希望找到替代方案的地方。然而，一些软件和各种路由设备会实现基于菜单的系统，并使用无法避免的终端模式。

在本节中，我们讨论了如何使用 `exec` 包自动化命令行。现在你已经掌握了检查系统中二进制文件并执行这些二进制文件的技能。你可以检查错误条件并获取输出。

在下一节中，我们将讨论 Go 中 SSH 的基础知识。虽然在本节中，我们展示了如何使用 `ssh` 二进制文件，接下来我们将讨论如何使用 `ssh` 包来使用 SSH 而不依赖 SSH 库。这种方法更快，并且相较于调用二进制文件，具有一定的优势。

注意

一般来说，始终使用包而不是二进制文件，特别是在有可用包的情况下。这可以保持系统依赖性较低，并使代码更具可移植性。

# 使用 Go 中的 SSH 自动化远程更改

SSH 只是一个网络协议，可用于保障两台主机之间的通信安全。

虽然大多数人认为 `ssh` 二进制文件允许你从一个主机的终端连接到另一个主机的终端，但这只是其中的一种用法。SSH 还可以用于保护如**Google 远程过程调用**（**gRPC**）这样的服务的连接，或者用于隧道化图形界面，如**X 窗口系统**（**X11**）。

在本节中，我们将讨论如何使用 SSH 包（[`pkg.go.dev/golang.org/x/crypto/ssh`](https://pkg.go.dev/golang.org/x/crypto/ssh)）来创建客户端和服务器。

## 连接到另一台系统

SSH 的最基本用法是连接到另一台系统，并发送一个命令或调用一个 shell 并执行命令。SSH 只是一个传输机制，因此 SSH 还有很多其他用途，如连接隧道或封装**远程过程调用**（**RPCs**）。我们不会在这里讨论这些内容，因为它们超出了常规 DevOps 工作的使用范围。

和大多数连接技术一样，使用 SSH 客户端连接系统时，最难的部分是解决身份验证问题。最常见的 SSH 身份验证方式在这里进行了概述：

+   **用户名/密码**：用户名/密码是最常见的实现方式。它是默认选项，因此人们往往会选择使用它。在网络设备中，有时这是唯一的方式。使用此方法时，密码数据库可能存储在本地系统中，或者系统会将密码哈希传递到另一个系统进行验证。

+   **公钥认证**：公钥认证是用户在自己的机器上创建一对公钥/私钥，并可以选择设置密码短语。服务器为用户安装了公钥，而你的 SSH 客户端则配置为使用私钥。

+   **挑战-响应认证**：SSH 有多种类型的挑战-响应认证。这通常用于通过设备（如 Yubikey）实现**二因素认证**（**2FA**）。

我们将专注于使用前两种方法，并假设远程端会使用 OpenSSH。虽然安装应该转向使用二次身份验证(2FA)，但这个设置超出了我们这里的讨论范围。

我们将使用 Go 的优秀 SSH 包：[`golang.org/x/crypto/ssh`](http://golang.org/x/crypto/ssh)。

首先需要做的是设置我们的认证方式。我这里将展示的初始方法是使用用户名/密码，如下所示：

```
auth := ssh.Password("password")
```

这已经足够简单了。

注意

如果你正在编写一个命令行应用程序，使用标志或参数来获取密码是不安全的。你也不希望将密码回显到屏幕上。密码应该来自一个只有当前用户可以访问的文件，或者通过控制终端。SSH 包有一个终端包([`golang.org/x/crypto/ssh/terminal`](http://golang.org/x/crypto/ssh/terminal))，它可以提供帮助：

`fmt.Printf("SSH 密码: ")`

`password, err := terminal.ReadPassword(int(os.Stdin.Fd()))`

对于公钥，稍微复杂一点，如下所示：

```
func publicKey(privateKeyFile string) (ssh.AuthMethod, error) {
    k, err := os.ReadFile(privateKeyFile)
    if err != nil {
            return nil, err
    }
    signer, err := ssh.ParsePrivateKey(k)
    if err != nil {
            return nil, err
    }
    return ssh.PublicKeys(signer), nil
}
```

这段代码执行以下操作：

+   读取我们的私钥文件

+   解析我们的私钥

+   返回一个公钥授权实现的`ssh.AuthMethod`

现在，我们只需将我们的私钥提供给程序即可进行授权。许多时候，你的密钥并不是本地存储的，而是存储在云服务中，如 Microsoft Azure 的 Key Vault。在这种情况下，你只需更改`os.ReadFile()`以使用云服务。

既然我们的授权已经解决了，接下来让我们创建一个 SSH 配置，如下所示：

```
config := &ssh.ClientConfig {
    User: user,
    Auth: []ssh.AuthMethod{auth},
    HostKeyCallback: ssh.InsecureIgnoreHostKey(),
    Timeout: 5 * time.Second,
}
```

这段代码执行以下操作：

+   创建一个新的`*ssh.ClientConfig`配置

    +   使用存储在`user`变量中的用户名

    +   提供一个`AuthMethod`，但你可以使用多个`AuthMethod`(s)

    +   忽略主机密钥

        +   设置 5 秒的拨号超时

        重要提示

        使用`ssh.InsecureIgnoreHostKey()`来忽略主机密钥是不安全的。这可能导致你错误地将信息发送到一个你无法控制的系统。这个系统可能伪装成你的一台设备，试图让你在终端中输入某些内容，比如密码。在生产环境中，至关重要的是不要忽略主机密钥，并存储一个有效的主机密钥列表以供验证。

让我们连接到主机，如下所示：

```
conn, err := ssh.Dial("tcp", host, config)
if err != nil {
    fmt.Println("Error: could not dial host: ", err)
    os.Exit(1)
}
defer conn.Close()
```

现在我们已经建立了一个 SSH 连接，接下来让我们创建一个函数来运行一个简单的命令，如下所示：

```
func combinedOutput(conn *ssh.Client, cmd string) (string, error) {
    sess, err := conn.NewSession()
    if err != nil {
            return "", err
    }
    defer sess.Close()
    b, err := sess.Output(cmd)
    if err != nil {
            return "", err
    }
    return string(b), nil
}
```

这段代码执行以下操作：

+   创建一个 SSH 会话

    +   每个命令需要一个会话

+   在会话中运行命令并返回输出

    +   这会将 STDOUT 和 STDERR 合并为一个输出

此代码将允许您针对使用 OpenSSH 或类似 SSH 实现的系统发出命令。最佳实践是在为设备发出所有命令之前保持`conn`对象的打开状态。

您可以在此处查看此代码：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/8/ssh/client/remotecmd/remotecmd.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/8/ssh/client/remotecmd/remotecmd.go)

在可以简单地向远端发出命令并让其运行的情况下非常有用。但是如果程序需要一定程度的交互怎么办？在通过 SSH 与路由平台交互时，通常需要更多的交互。

当这种需求出现时，Expect 库可以提供帮助。接下来，让我们看看其中一个比较流行的库。

### 用于复杂交互的 Expect

`expect`包提供处理命令输出的能力，例如以下内容：`would you like to continue[y/n]`。

使用`expect`的最流行的包来自 Google。您可以在这里找到：[`github.com/google/goexpect`](https://github.com/google/goexpect)。

这是一个`expect`脚本示例，用于在 Ubuntu 主机上使用**高级包装工具**（**APT**）包管理器安装原始的 TCL `expect`工具。请注意，这不是最佳实践，只是一个简单的示例。

让我们首先配置我们的`expect`客户端以使用 SSH 客户端，如下所示：

```
config := &ssh.ClientConfig {
    User:            user,
    Auth:            []ssh.AuthMethod{auth},
    HostKeyCallback: ssh.InsecureIgnoreHostKey(),
}
conn, err := ssh.Dial("tcp", host, config)
if err != nil {
    return err
}
e, _, err := expect.SpawnSSH(conn, 5 * time.Second)
if err != nil {
    return err
}
defer e.Close()
```

此代码执行以下操作：

+   设置一个`*ssh.ClientConfig`配置

+   使用它建立连接

+   将连接传递给`expect`客户端

现在我们已经通过 SSH 登录了一个`expect`客户端，请确保我们有一个提示符，如下所示：

```
var (
    promptRE = regexp.MustCompile(`\$ `)
    aptCont = regexp.MustCompile(`Do you want to continue\? \[Y/n\] `)
    aptAtNewest = regexp.MustCompile(`is already the newest`)
)
_, _, err = e.Expect(promptRE, 10*time.Second)
if err != nil {
        return fmt.Errorf("did not get shell prompt")
}
```

此代码执行以下操作：

+   编译`$`正则表达式以期望我们的提示符

+   调用`Expect()`等待最多 10 秒的提示符

现在，让我们发送我们的命令通过`apt-get`工具安装`expect`。我们将使用`sudo`以 root 权限执行此命令。代码如下所示：

```
if err := e.Send("sudo apt-get install expect\n"); err != nil {
        return fmt.Errorf("error on send command: %s", err)
}
```

`apt-get`将提示我们是否可以安装或告诉我们它已经安装。让我们处理这两种情况，如下所示：

```
f _, _, ecase, err := e.ExpectSwitchCase(
    []expect.Caser{
            &expect.Case{
                    R: aptCont,
                    T: expect.OK(),
            },
            &expect.Case{
                    R: aptAtNewest,
                    T: expect.OK(),
            },
    },
    10*time.Second,
)
if err != nil {
        return fmt.Errorf("apt-get install did not send what we expected")
}
```

此代码执行以下操作：

+   等待显示以下内容之一：

    +   `Do you want to continue\? [Y/n]`

    +   `is already the newest`

+   如果两者都没有发生，它将给出一个错误

+   `ecase`将包含详细说明发生的条件的`case`类型

如果我们得到继续提示，我们需要发送`Y`到终端，执行以下代码：

```
switch ecase{
case 0:
        if err := e.Send("Y\n"); err != nil {
                return err
        }
}
```

最后，我们只需确保通过执行以下代码再次收到提示：

```
_, _, err = e.Expect(promptRE, 10*time.Second)
if err != nil {
        return fmt.Errorf("did not get shell prompt")
}
return nil
```

您可以在调试模式下查看此代码：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/8/ssh/client/expect/expect.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/8/ssh/client/expect/expect.go)

本节展示了如何在纯 Go 中启动一个 SSH 会话，使用它发送命令，然后获取输出。最后，我们还探讨了如何使用 `goexpect` 与应用程序进行交互。

现在，我们将展示如何利用这些知识编写工具，以便在多个系统上运行命令。

# 设计安全的并发变更自动化

到目前为止，我们已经展示了如何在本地或远程执行命令。

在现代，我们经常需要在多个系统上运行一组命令，以实现某个最终状态。根据规模的不同，你可能希望运行诸如 Ansible 或 Jenkins 这样的系统来尝试自动化这些过程。

对于某些工作，直接使用 Go 在一组系统上执行更改会更简单。这使得 DevOps 团队只需理解 Go 语言和少量代码，而无需理解像 Ansible 这样的工作流系统的复杂性，后者需要自己的技能集、系统更新等。

在本节中，我们将讨论如何更改一组系统的组成部分，达成这一目标的框架，以及一个示例应用程序来应用一组更改。

## 更改的组成部分

在编写一个进行更改的系统时，必须处理几种类型的操作。广义上来说，我将它们定义为以下几种：

+   **全局前提条件**：全局前提条件是一组必须为真的条件才能继续前进。在进行网络自动化时，这可能是网络丢包率低于某个阈值。对于设备来说，这可能意味着在继续操作之前，服务处于绿色状态。没有人愿意在出现问题时推送更改。

+   **本地前提条件**：本地前提条件是指单个工作单元（例如服务器）必须处于某种状态才能继续。

+   **操作**：操作是将改变工作单元状态的操作。

+   **操作验证**：用于验证操作是否成功的检查。

+   **本地后置条件**：本地后置条件是检查工作单元是否处于所需的配置状态并满足某些条件。这可能是它仍然可达，可能正在处理流量或没有处理流量，无论最终状态应该是什么。

+   **全局后置条件**：全局后置条件是在执行后条件的状态，通常类似于全局前提条件。

并非每一组跨多个系统的更改都需要这些所有条件，但至少需要其中的一部分。

让我们来看一下如何在单一数据中心的一组 **虚拟机** (**VMs**) 上进行作业的部署。对于机器数量有限的小型公司来说，当你没有足够大到可以使用像 Kubernetes 这样的工具，但又无法满足像 Azure Functions 或亚马逊的 **弹性容器服务** (**ECS**) 的限制时，这样的设置可能就足够了。或者，也可能是你在自己的机器上运行，而不是使用云服务提供商。

## 编写一个并发任务

让我们来处理我们想要执行的操作。我们想要做以下操作：

+   从负载均衡器中移除我们的任务

+   杀死虚拟机或服务器上的任务

+   将新的软件复制到服务器

+   启动我们的服务

+   检查服务是否可达

+   将任务重新添加到负载均衡器

从本质上讲，这正是 Kubernetes 在大规模微服务安装中的作用。我们将在即将到来的章节中讨论这一点。但在小规模应用中，即使基础设施由云服务提供商管理，运行 Kubernetes 集群的复杂性通常也不是最好的选择。

让我们定义执行我们操作的代码的总体结构，如下所示：

```
type stateFn func(ctx context.Context) (stateFn, error)
type actions struct {
    ... // Some set of attributes
}
func (s *actions) run(ctx context.Context) (err error) {
    fn := s.rmBackend
    if s.failedState != nil {
        fn = s.failedState
    }
    s.started = true
    for {
        if ctx.Err() != nil {
            s.err = ctx.Err()
            return ctx.Err()
        }
        fn, err = fn(ctx)
        if err != nil {
            s.failedState = fn
            s.err = err
            return err
        }
        if fn == nil {
            return nil
        }
    }
}
func (a *actions) rmBackend(ctx context.Context) (stateFn, error) {...}
func (a *actions) jobKill(ctx context.Context) (stateFn, error) {...}
func (a *actions) cp(ctx context.Context) (stateFn, error) {...}
func (a *actions) jobStart(ctx context.Context) (stateFn, error) {...}
func (a *actions) reachable(ctx context.Context) (stateFn, error) {...}
func (a *actions) addBackend(ctx context.Context) (stateFn, error) {...}
```

注意

这一部分大多是骨架代码—我们稍后将实现这些方法。

这段代码执行以下操作：

+   定义一个`stateFn`类型

    +   如果返回错误，停止处理。

    +   如果没有并且返回一个非空的`stateFn`类型，执行它。

    +   如果返回一个空的`stateFn`类型并且没有错误，我们就完成了。

+   定义一个`actions`类型

    +   这是一个用于服务器操作的状态机

    +   调用`run()`会执行以下操作：

        +   一次执行一个`stateFn`类型，直到出现错误或`stateFn == nil`

    +   `rmBackend()`、`jobKill()`、`cp()`以及其他将定义的都是`stateFn`类型。

    +   `.failedState`用于允许在多次调用`.run()`时重试失败的状态。

我们有一个简单的状态机，将执行操作。这将使我们完成系统上执行此类操作所需的所有状态。

让我们看看在实现时，几个`stateFn`类型会是什么样子，如下所示：

```
func (a *actions) rmBackend(ctx context.Context) (stateFn, error) {
    err := a.lb.RemoveBackend(ctx, a.config.Pattern, a.backend)
    if err != nil {
        return nil, fmt.Errorf("problem removing backend from pool: %w", err)
    }
    return a.jobKill, nil
}
```

这段代码执行以下操作：

+   调用客户端的网络负载均衡器以移除我们的服务器端点

+   如果成功，返回`jobKill`作为下一个要执行的状态

+   如果不成功，返回我们的错误

`s.lb.RemoveBackend()`在云端可能会调用**REST**服务，通知它移除我们的服务端点。或者，在你自己的数据中心，它可能是一个网络负载均衡器，你通过 SSH 客户端登录并发出命令。

一旦完成，它会告诉`run()`执行`jobKill()`。让我们看看实现后的样子，如下所示：

```
func (a *actions) jobKill(ctx context.Context) (stateFn, error) {
    pids, err := a.findPIDs(ctx)
    if err != nil {
        return nil, fmt.Errorf("problem finding existing PIDs: %w", err)
    }
    if len(pids) == 0 {
        return a.cp, nil
    }
    if err := a.killPIDs(ctx, pids, 15); err != nil {
        return nil, fmt.Errorf("failed to kill existing PIDs: %w", err)
    }
    if err := a.waitForDeath(ctx, pids, 30*time.Second); err != nil {
        if err := a.killPIDs(ctx, pids, 9); err != nil {
            return nil, fmt.Errorf("failed to kill existing PIDs: %w", err)
        }
        if err := a.waitForDeath(ctx, pids, 10*time.Second); err != nil {
            return nil, fmt.Errorf("failed to kill existing PIDs after -9: %w", err)
        }
        return a.cp, nil
    }
    return a.cp, nil
}
```

这段代码执行以下操作：

+   执行`findPIDs()`函数

    +   这通过 SSH 登录到一台机器并运行`pidof`二进制文件

+   执行`killPIDs()`函数

    +   这使用 SSH 执行`kill`命令来终止我们的进程

    +   使用信号 15 或`TERM`作为软终止

+   执行`waitForDeath()`函数

    +   这使用 SSH 等待`cp`操作

    +   如果没有，执行带信号 9 或`KILL`的`killPIDs()`，并再次执行`waitForDeath()`函数

    +   如果失败，返回一个错误

    +   如果成功，我们返回下一个状态，`cp`

这段代码实际上是在我们复制新的二进制文件并启动它之前，先杀死服务器上的任务。

其余的代码将在我们的代码库中（稍后将在本节提供链接）。现在，假设我们已经为我们的状态机编写了其余的操作。

现在我们需要执行所有操作。我们将创建一个具有基本结构的`workflow`结构体：

```
type workflow struct {
    config *config
    lb     *client.Client
    failures int32
    endState endState
    actions []*actions
}
```

这段代码执行以下操作：

+   有`*config`，将详细描述我们的发布设置

+   创建与负载均衡器的连接

+   跟踪我们遇到的失败次数

+   输出最终的结束状态，这是文件中的一个枚举值

+   创建所有操作的列表

一个典型的发布过程有两个阶段，如下所示：

+   **金丝雀**：金丝雀阶段是测试少量样本，以确保发布过程正常工作。在此阶段，您需要一次测试一个样本，并在继续下一个金丝雀测试之前等待一段时间。这为管理员提供了时间，以防发布过程未能检测到潜在问题。

+   **一般发布**：一般发布发生在金丝雀阶段之后。通常会设置一定的并发数和最大失败次数。根据环境的大小，失败可能很常见，因为环境在不断变化。这可能意味着您会容忍一定数量的失败，并继续重试这些失败，直到成功，但如果失败次数达到某个最大值，则停止。

    注意

    根据环境的不同，您可以使用更复杂的部署方案，但对于较小的环境，这通常已经足够。在进行并发发布时，失败的次数可能会超过您的最大失败设置，这取决于设置的具体情况。如果我们设置了最大失败次数，并且并发数设置为 5，那么可能会发生 5 到 9 次的失败。在处理并发发布时，请记住这一点。

处理发布过程的工作流中的主要方法叫做`run()`。它的任务是运行我们的前置检查，然后运行我们的金丝雀测试，最后以某种并发级别运行主要任务。如果问题太多，我们应该退出。我们来看一下，具体如下：

```
func (w *workflow) run(ctx context.Context) error {
    preCtx, cancel := context.WithTimeout(ctx, 30*time.Second)
    if err := w.checkLBState(preCtx); err != nil {
        w.endState = esPreconditionFailure
        return fmt.Errorf("checkLBState precondition fail: %s", err)
    }
    cancel()
```

这部分代码执行以下操作：

+   运行我们的`checkLBState()`前置条件代码

+   如果失败，记录一个`esPreconditionFailure`结束状态

    注意

    您可能会注意到在创建带有超时的`Context`对象时，会创建一个`cancel()`函数。这个函数可以在任何时候取消我们的`Context`对象。最佳实践是在使用后立即取消带有超时的`Context`对象，以退出正在后台运行并倒计时到超时的 Go 例程。

这是在我们对系统进行任何更改之前运行的。我们不希望在系统已经不健康时进行更改。

接下来，我们需要运行我们的金丝雀测试，如下所示：

```
for i := 0; i < len(w.actions) && 
int32(i) < w.config.CanaryNum; i++ {
    color.Green("Running canary on: %s", w.actions[i].endpoint)
    ctx, cancel := context.WithTimeout(ctx, 10*time.Minute)
    err := w.actions[i].run(ctx)
    cancel()
    if err != nil {
        w.endState = esCanaryFailure
        return fmt.Errorf("canary failure on endpoint(%s): %w\n", w.actions[i].endpoint, err)
    }
    color.Yellow("Sleeping after canary for 1 minutes")
    time.Sleep(1 * time.Minute)
}
```

这段代码执行以下操作：

+   运行若干个金丝雀测试

+   一次执行一个操作

+   每次等待 1 分钟

这些设置将在定义的配置文件中进行配置。休眠时间可以根据服务的需求进行配置，以便在工作流未检测到问题时能作出响应。您甚至可以定义在所有金丝雀测试和一般发布之间的休眠时间。

现在，我们需要在一定的并发水平下进行发布，同时检查失败的最大数量。让我们按照以下方式查看这一点：

```
limit := make(chan struct{}, w.config.Concurrency)
wg := sync.WaitGroup{}
for i := w.config.CanaryNum; int(i) < len(w.actions); i++ {
    i := i
    limit <- struct{}{}
    if atomic.LoadInt32(&w.failures) > w.config.MaxFailures {
        break
    }
    wg.Add(1)
    go func() {
        defer func(){<-limit}()
        defer wg.Done()
        ctx, cancel := context.WithTimeout(ctx, 10*time.Minute)
        color.Green("Upgrading endpoint: %s", 
w.actions[i]. endpoint)
        err := w.actions[i].run(ctx)
        cancel()
        if err != nil {
            color.Red("Endpoint(%s) had upgrade error: %s", w.actions[i].endpoint, err)
            atomic.AddInt32(&w.failures, 1)
        }
    }()
}
wg.Wait()
```

这段代码完成了以下操作：

+   启动运行我们操作的 goroutine。

+   并发通过我们的 `limit` 通道进行限制。

+   失败情况由我们的 `.failures` 属性检查进行限制。

这是我们第一次展示 `atomic` 包。`atomic` 是 `sync` 的一个子包，它允许我们在不使用 `sync.Mutex` 的情况下进行线程安全的数字操作。这对于计数器非常有用，因为它为这种特定类型的操作提供了类似 `sync.Mutex` 的功能。

我们现在展示了 `workflow` 结构体的 `.run()` 基本用法。您可以在 [`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/8/rollout`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/8/rollout) 找到这个版本应用的完整代码。

这个应用的代码只需要您的 SSH 密钥、描述发布的文件和要发布到服务器的二进制文件。该文件看起来应该是这样的：

```
{
    "Concurrency": 2,
    "CanaryNum": 1,
    "MaxFailures": 2,
    "Src": "/home/[user]/rollout/webserver",
    "Dst": "/home/[user]/webserver",
    "LB": "10.0.0.4:8081",
    "Pattern": "/",
    "Backends": [
            "10.0.0.5",
            "10.0.0.6",
            "10.0.0.7",
            "10.0.0.8",
            "10.0.0.9"
    ],
    "BackendUser": "azureuser",
    "BinaryPort": 8082
}
```

这描述了应用程序进行简单发布所需做的一切。

当然，我们可以使这个应用更具通用性，让它记录运行状态和最终状态到存储中，添加标志来忽略初始状态，以便我们可以进行回滚，将其放到 gRPC 服务后面，等等……

在不到 1,000 行代码的情况下，我们提供了一个简单的替代方案，用于当 Kubernetes 等系统不可用或您的规模不足以支持它们时的选择。

注意

这并没有解决程序崩溃时需要重新启动二进制文件的问题，比如通过 `systemd` 等软件实现的重启。在这种情况下，最好是创建一个代理程序，运行在设备上并提供 RPC 来控制本地服务，比如 `systemd`。

## 案例研究——网络发布

这里阐述的原则已经成为谷歌 B2 骨干网络上网络设备配置发布的核心，已有十年之久。

在此之前，我们仅仅是用脚本处理手工配置或生成的配置，将它们应用到网络上，同时操作员观察进度并处理可能出现的问题。

在大规模应用中，这成为了一个问题。SRE 服务团队开始远离类似的模型，因为它们的复杂性往往比网络增长得更快。

网络工程逐渐转向一个更为正式化的系统，以集中执行骨干网中的工作，为我们提供一个监控的地方，并在紧急情况下有一个中央位置来停止发布操作。

此外，还需要对所有发布操作进行正式化，以确保它们总是以相同的方式执行，并且具有相同的自动化检查，而不是依赖人工来做正确的操作。

我主导设计和实现的编排系统，实际上是一个更复杂且可插拔的版本，类似于这里所展示的内容。各个团队将它们的操作集成到系统中，而该系统则根据传递的参数执行这些操作，完成一系列任务。

在我离开 Google 时，采用这种方法已经实现了自动化零故障（这与零发布失败不同）。据我了解，当我写这篇文章时，你的猫咪视频仍然在这个系统上得到安全保存。

在本节中，我们了解了变更的组件以及使用 Go 实现这些变更的方式，并编写了一个示例发布应用程序，应用了这些原则。

接下来，我们将讨论编写一个系统代理，该代理可以部署在系统上，从而允许进行系统监控到控制本地发布的所有操作。

# 编写系统代理

到目前为止，当我们在设备上进行自动化操作时，我们要么是在本地执行的应用程序中做，要么是通过 SSH 远程运行命令。

但如果我们考虑管理一小部分机器集群，编写一个在设备上运行的服务，通过 RPC 连接进行控制可能会更为实际。利用我们在前面章节中讨论的 gRPC 服务知识，我们可以将这些概念结合起来，以更统一的方式控制我们的机器。

以下是我们可以使用系统代理的一些用途：

+   安装和运行服务

+   收集机器运行状态

+   收集机器库存信息

其中一些是 Kubernetes 使用其系统代理所做的事情。其他的，比如库存信息，对于运行健康的机器集群至关重要，尤其是在较小的环境中经常被忽视。即使在 Kubernetes 环境中，为某些任务运行自己的代理也可能带来优势。

系统代理可以提供多个优势。如果我们使用 gRPC 定义一个 **应用程序编程接口** (**API**)，我们可以让多个操作系统和不同的代理实现相同的 RPC，从而以统一的方式控制我们的机器集群，而不管操作系统是什么。而且因为 Go 几乎可以在任何平台上运行，你可以使用相同的语言编写不同的代理。

## 设计系统代理

对于我们的示例系统代理，我们将特别针对 Linux，但我们会使我们的 API 通用，以便其他操作系统也能实现相同的 API。我们来谈谈一些可能感兴趣的内容。我们可以考虑以下内容：

+   使用 `systemd` 安装/移除二进制文件

+   导出系统和已安装二进制文件的性能数据

+   允许拉取应用程序日志

+   将我们的应用程序容器化

对于不熟悉 `systemd` 的朋友，它是一个在后台运行软件服务的 Linux 守护进程。利用 `systemd` 可以实现应用程序失败后的自动重启，并通过 `journald` 实现日志轮转。

容器化，简单来说，是在一个自包含的空间内执行应用程序，只访问你希望其访问的操作系统部分。这与所谓的沙盒化（sandboxing）概念相似。容器化已经被 Docker 等软件所流行，并且催生了类似虚拟机的容器格式，这些容器内包含了整个操作系统镜像。然而，要在 Linux 上容器化一个应用程序，并不需要这些容器格式和工具。

由于我们将使用`systemd`来控制进程执行，我们将使用`systemd`的`Service`指令来提供容器化。这些细节可以在我们的代码库中的文件[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/8/agent/internal/service/unit_file.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/8/agent/internal/service/unit_file.go)中查看。

为了导出统计数据，我们将使用`expvar` Go 标准库包。这个包允许我们发布统计数据，`expvar`的统计数据是一个 JSON 对象，具有映射到代表我们的统计信息或数据的值的字符串键。系统内置的统计数据将自动提供，同时我们也会定义一些新的统计数据。

这使得你可以通过收集器或简单地使用网页浏览器或命令行工具（如`wget`）快速收集统计数据。

输出的一个`expvar`页面可能返回以下内容：

```
{
    "cmdline": ["/tmp/go-build7781/c0021/exe/main"],
    "cpu": "8",
    "goroutines": "16",
}
```

在我们示例中的书籍部分，我们将重点介绍*安装和移除二进制文件*和*导出系统性能数据*，以展示我们如何使用 RPC 服务进行交互调用，以及使用 HTTP 获取只读信息。我们代码库中的版本将实现比书中所能涵盖的更多功能。

现在我们已经讨论了系统代理要做的事情，接下来让我们为我们的服务设计 proto，具体如下：

```
syntax = "proto3";
package system.agent;
option go_package = "github.com/[repo]/proto/agent";
message InstallReq {
    string name = 1;
    bytes package = 2;
    string binary = 3;
    repeated string args = 4;
}
message InstallResp {}
message CPUPerfs {
    int32 resolutionSecs = 1;
    int64 unix_time_nano = 2;
    repeated CPUPerf cpu = 3;
}
message CPUPerf {
    string id = 1;
    int32 user = 2;
    int32 system = 3;
    int32 idle = 4;
    int32 io_wait = 5;
    int32 irq = 6;
}
message MemPerf {
    int32 resolutionSecs = 1;
    int64 unix_time_nano = 2;
    int32 total = 3;
    int32 free = 4;
    int32 avail = 5;
}
service Agent {
   rpc Install(InstallReq) returns (InstallResp) {};
}
```

现在我们已经有了 RPC 的通用框架，接下来我们来看一下如何为我们的`Install` RPC 实现一个方法。

## 实现安装功能

在 Linux 上实现安装将需要一个多步骤的过程。首先，我们将在代理的用户主目录下的`sa/packages/[InstallReq.Name]`目录中安装该包。`InstallReq.Name`需要是一个包含字母和数字的单一名称。如果该名称已经存在，我们将关闭现有的工作并在其位置安装新的包。Linux 上的`InstallReq.Package`将是一个 ZIP 文件，该文件将在该目录中解压。

`InstallReq.Binary`是根目录中要执行的二进制文件的名称。`InstallReq.Args`是要传递给二进制文件的参数列表。

我们将使用一个第三方包来访问`systemd`。你可以在这里找到该包：[`github.com/coreos/go-systemd/tree/main/dbus`](https://github.com/coreos/go-systemd/tree/main/dbus)。

让我们来看一下这部分的实现：

```
func (a *Agent) Install(ctx context.Context, req 
*pb.InstallReq) (*pb.InstallResp, error) {
    if err := req.Validate(); err != nil {
        return nil, status.Error(codes.InvalidArgument, 
err.Error())
    }
    a.lock(req.Name)
    defer a.unlock(req.Name, false)
    loc, err := a.unpack(req.Name, req.Package)
    if err != nil {
        return nil, err
    }
    if err := a.migrate(req, loc); err != nil {
        return nil, err
    }
    if err := a.startProgram(ctx, req.Name); err != nil {
        return nil, err
    }
    return &pb.InstallResp{}, nil
}
```

这段代码执行以下操作：

+   验证我们传入的请求以确保其有效

    +   实现代码位于代码库中

+   为这个特定的安装名称加锁

    +   这可以防止多个相同名称的安装同时进行

    +   实现代码在仓库中

+   将我们的 ZIP 文件解压到临时目录

    +   返回临时目录的位置

    +   验证我们的`req.Binary`二进制文件是否存在

    +   实现代码在仓库中

+   将我们的临时目录迁移到`req.Name`位置

    +   如果`systemd`单元已存在，则将其关闭

    +   在`/home/[user]/.config/systemd/user/`下创建一个`systemd`单元文件

    +   如果最终路径已存在，则删除它

    +   将临时目录移动到最终位置

    +   实现代码在仓库中

+   启动我们的二进制文件

    +   确保它已启动并运行 30 秒

这是设置我们 gRPC 服务的一个简单示例，用于设置和运行一个`systemd`服务。我们跳过了各种实现细节，但你可以在本章末尾列出的仓库中找到它们。

现在我们完成了`Install`，接下来让我们实现`SystemPerf`。

## 实现 SystemPerf

为了收集我们的系统信息，我们将使用`goprocinfo`包，您可以在这里找到它：[`github.com/c9s/goprocinfo/tree/master/linux`](https://github.com/c9s/goprocinfo/tree/master/linux)。

我们希望每 10 秒更新一次，因此我们将在一个循环中实现数据收集，所有调用者都从相同的数据中读取。

让我们首先收集系统的**中央处理单元**（**CPU**）数据，如下所示：

```
func (a *Agent) collectCPU(resolution int) error {
    stat, err := linuxproc.ReadStat("/proc/stat")
    if err != nil {
        return err
    }
    v := &pb.CPUPerfs{
        ResolutionSecs: resolution,
        UnixTimeNano:   time.Now().UnixNano(),
    }
    for _, p := range stat.CPUStats {
        c := &pb.CPUPerf{
            Id:     p.Id,
            User:   int32(p.User),
            System: int32(p.System),
            Idle:   int32(p.Idle),
            IoWait: int32(p.IOWait),
            Irq:    int32(p.IRQ),
        }
        v.Cpu = append(v.Cpu, c)
    }
    a.cpuData.Store(v)
    return nil
}
```

这段代码执行以下操作：

+   读取我们的 CPU 状态数据

+   将其写入协议缓冲区

+   将数据存储在`.cpuData`中

`.cpuData`将是`atomic.Value`类型。当你希望同步整个值，而不是修改值时，这种类型非常有用。每次我们更新`a.cpuData`时，我们都会把一个新值放入其中。如果你在`atomic.Value`中存储`struct`、`map`或`slice`，你不能修改键/字段——你*必须*制作一个包含所有键/索引/字段的新副本并存储，而不是修改单个键/字段。

当值较小时，这比使用互斥锁更适合读取，当存储少量计数器时非常完美。

`collectMem`内存收集器类似于`collectCPU`，并在仓库代码中有详细说明。

让我们来看看在`New()`构造函数中启动的用于收集性能数据的循环，如下所示：

```
func (a *Agent) perfLoop() error {
    const resolutionSecs = 10
    if err := a.collectCPU(resolutionSecs); err != nil {
        return err
    }
    expvar.Publish(
        "system-cpu",
        expvar.Func(
            func() interface{} {
                return a.cpuData.Load().(*pb.CPUPerfs)
            },
        ),
    )
    go func() {
        for {
            time.Sleep(resolutionSecs * time.Second)
            if err := a.collectCPU(resolutionSecs); err != nil {
                log.Println(err)
            }
        }
    }()
        return nil
}
```

这段代码执行以下操作：

+   收集我们初始的 CPU 统计信息

+   发布`system-cpu`的`expvar.Var`类型

    +   我们的变量类型是`func() interface{}`，它实现了`expvar.Func`

    +   这只是读取由`collectCPU()`函数设置的`atomic.Value`

        +   当有人查询我们位于`/debug/vars`的网页时，会发生读取操作

+   每 10 秒刷新我们的数据收集

`expvar`定义了其他一些简单的类型，例如`String`、`Float`、`Map`等。然而，我更喜欢使用协议缓冲区（proto）而不是`Map`来将内容分组到一个单一的、可共享的消息类型中，这种消息类型可以在任何语言中使用。因为 proto 是 JSON 可序列化的，它可以在`expvar.Func`的返回值中使用，只需借助`protojson`包即可。在代码库中，那个辅助代码位于`agent/proto/extra.go`。

这段代码仅共享最新的数据收集。重要的是不要在每次调用时直接从统计文件中读取数据，因为这可能会轻易导致系统过载。

当你访问`/debug/vars`的 Web 端点时，现在可以看到以下内容：

```
"system-cpu": {"resolutionSecs":10,"unixTimeNano":"1635015190106788056","cpu":[{"id":"cpu0","user":13637,"system":10706,"idle":17557545,"ioWait":6663},{"id":"cpu1","user":12881,"system":22465,"idle":17539705,"ioWait":2997}]},
"system-mem": {"resolutionSecs":10,"unixTimeNano":"163501519010 6904757","total":8152984,"free":6594776,"avail":7576540}
```

还有一些其他的统计信息是针对系统代理本身的，这些在调试代理时可能会有用。这些是由`expvar`自动导出的。通过使用连接并读取这些统计信息的收集器，可以查看这些统计数据随时间的趋势。

我们现在有一个每 10 秒获取一次性能数据的代理，这为我们提供了一个有效的系统代理。值得注意的是，我们在讨论 RPC 系统时避免谈论**认证、授权和计账**（**AAA**）。gRPC 支持**传输层安全**（**TLS**），既可以保护传输过程，也可以实现互信 TLS。你还可以实现用户/密码、**开放授权**（**OAuth**）或任何你感兴趣的 AAA 系统。

Web 服务可以为类似`expvar`的内容实现自己的安全性。`expvar`会在`/debug/vars`上发布它的统计信息，因此最好不要将这些信息暴露给外部世界。可以通过防止所有负载均衡器导出，或者在端点上实现某种类型的安全措施来保护这些信息。

你可以在这里找到我们系统代理的完整代码：[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/8/agent`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/8/agent)。

在我们的完整代码中，我们决定通过 SSH 实现我们的系统代理。这使我们可以使用已经存在的授权系统，并提供强大的传输安全性。此外，gRPC 服务通过私有 Unix 域套接字导出服务，因此非`root`的本地服务无法访问该服务。

你还会发现代码会将我们通过`systemd`指令安装的应用容器化。这提供了本地隔离，有助于保护系统。

在本节中，我们学习了系统代理的可能用途，构建系统代理的基本设计指南，并最终介绍了如何在 Linux 上实现一个基本的代理。我们还讨论了我们的 gRPC 接口是如何被设计成通用的，以便可以实现其他操作系统的代理。

在构建代理的过程中，我们简要介绍了如何使用`expvar`导出变量。在下一章中，我们将讨论`expvar`的“大哥”——Prometheus 包。

# 总结

本章是对自动化命令行的介绍。我们已经展示了如何使用`exec`包在设备上本地执行命令。当需要将一组已有工具串联起来时，这非常有用。我们还展示了如何使用`ssh`包在远程系统上运行命令，或使用`ssh`和`goexpect`包与复杂的程序进行交互。我们将这部分与前几章的 Go 知识结合，实施了一个基本的工作流应用程序，该程序能够并行且安全地在多个系统上升级二进制文件。最后，在本章中，我们学习了如何创建一个在设备上运行的系统代理，使我们能够收集重要数据并将其导出。我们还通过使用该代理控制 Linux 设备上的`systemd`，进一步提高了安装程序的能力。

本章已经为你提供了新的技能，使你能够控制本地命令行应用程序，在任意数量的机器上执行远程应用程序，并处理交互式应用程序。你还获得了构建工作流应用程序的基本理解，学习了如何开发可以控制本地机器的 RPC 服务，以及如何使用 Go 的`expvar`包导出统计数据。

在下一章中，我们将讨论如何观察正在运行的软件，以便在问题变成故障之前及时检测，并在事件发生时进行故障诊断。
