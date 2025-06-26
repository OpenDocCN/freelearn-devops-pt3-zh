# 5

# Facts 和 Functions（函数）

本章将讲解 facts（事实）。我们将展示 Facter 工具如何收集它们以显示系统配置文件，如何与 Facter 交互，以及如何在 Puppet 代码中使用它们。我们还将介绍如何将自定义和外部的 facts 添加到提供的核心 facts 中，以便收集更多特定于用户的 facts。

接下来，我们将介绍函数。我们将解释函数的作用以及三种类型的函数——语句函数、前缀函数和链式函数。我们将考察一些核心函数，展示它们的功能。同时，也会展示来自 `stdlib` 模块的一些函数，并解释该模块的使用方法和方式。

延迟函数（Deferred functions）是在 Puppet 6 中引入的，本节将对此进行讲解。在这里，我们将向您展示延迟函数与普通函数的区别，如何使一个函数成为延迟函数，以及在使用延迟函数时应避免的陷阱。

简而言之，本章将涵盖以下主题：

+   Facts 和 Facter

+   自定义 facts 和外部 facts

+   函数

+   stdlib 模块函数

+   延迟函数

# 技术要求

本章中，您需要通过下载 [`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch05/params.json`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch05/params.json) 文件，并使用以下命令从 `pecdm` 目录中配置一个标准的 Puppet 服务器架构，其中包含一个 Windows 客户端和一个 Linux 客户端：

```
bolt --verbose plan run pecdm::provision –params @params.json
```

# Facts 和 Facter

Facter 是 Puppet 的系统分析工具，是一组跨平台的 Ruby 库，用于收集关于客户端的信息（称为 **facts**）。这些工具提供了评估客户端配置文件所需的信息，并允许根据主机在 Puppet 代码中的先前状态做出配置决策。

Puppet 5 和 6 使用 Facter 3，而 Puppet 7 使用 Facter 4。Facter 4 中仅提供了一小部分功能，本文将重点介绍这些功能，并且少量事实（facts）有所变化，但大多数用户将不会发现差异。您可以通过运行 `puppet facts diff` 命令查看这些差异。在*第八章*中，我们将重点介绍如何通过模块测试确保代码在不同版本间的兼容性。

可以通过在命令行或 VSCode 终端中运行 `facter -p` 或 `puppet facts` 命令查看 Facter 的输出。运行这些命令而不添加任何额外选项时，将返回所有核心 facts。`-p` 标志确保收集 Puppet 特定的 facts。由于 Facter 和 Puppet 之间创建了循环依赖，之前计划废弃 `-p` 标志并用 `puppet facts` 命令代替，但随着 Facter 4 的发布，这一做法被放弃了。本书的示例将使用 `facter` 命令，这与文档和社区的实践一致。

注意

默认情况下，`facter` 命令以 Puppet 哈希格式输出，而 `puppet facts` 以 JSON 格式输出。这两个命令都接受选择适当格式的选项。

现在我们来看一些 Facter 输出的示例。最简单的事实类型是简单的键值对，例如 `Kernel` 事实，在本例中，它告诉我们内核是基于 Windows 的：

```
"Kernel": "windows"
```

还有一些称为结构化事实的哈希值，它们可以分解成嵌套级别。`os` 事实是常用的。以下是 Windows 10 笔记本电脑的示例，展示了可用的各种级别：

```
os => {
  architecture => "x64",
  family => "windows",
  hardware => "x86_64",
  name => "windows",
  release => {
    full => "10",
    major => "10"
  },
  windows => {
    display_version => "21H2",
    edition_id => "Core",
    installation_type => "Client",
    product_name => "Windows 10 Home",
    release_id => "21H2",
    system32 => "C:\WINDOWS\system32"
  }
}
```

核心事实的完整列表可以在 [`puppet.com/docs/puppet/latest/core_facts.html`](https://puppet.com/docs/puppet/latest/core_facts.html) 中找到；建议在客户端系统上运行 `facter -p` 并查看输出。可以通过运行 `facter` 命令并指定事实名称来访问单个事实，例如运行 `facter -p kernel` 来返回 `kernel` 事实。要访问结构化事实中的特定嵌套级别值，使用点符号表示法，点号（`.`）分隔每个键级名称。因此，要访问 `os` 结构化事实中的 `family` 事实，可以运行 `facter -p os.family` 命令。

由于 Facter 经历了多个版本迭代，并且早期版本没有结构化事实，Facter 3 隐藏了多个遗留事实，如架构，它被放入 `os` 结构化事实中作为 `os.structured`。`--show-legacy` 标志可以使这些事实在 Facter 输出中可见；它们在核心事实文档中有记录。

当 Puppet 运行时，无论是通过代理还是在命令行上运行 `puppet apply`，Facter 都会运行，使用遗留事实，并且输出将分配给全局变量。

然后，这些变量可以通过两种方式在 Puppet 清单中访问——要么直接通过事实的名称作为全局变量，要么通过 facts 数组。强烈建议仅通过 facts 数组访问事实，因为这样可以明确表示正在访问事实，而不是其他潜在的全局变量。

例如，在以下代码中，`notify` 资源将访问 `kernel` 和 `os family` 变量，并打印包含主机 `kernel` 和 `os` 系列的日志信息：

```
notify { "This clients kernel is ${facts[kernel]}": }
notify { "This client is a member of the os family ${facts[os][family]": }
```

请注意，并非所有事实都会出现在所有客户端上。事实通常会根据某些上下文进行过滤，例如正在运行的操作系统，或者是否使用了特定的底层硬件。

注意

正如你将在下一节中看到的那样，函数使用点号来表示链式函数，因此 `facter` 命令的点分隔访问语法不能用于直接调用 `facts` 变量。然而，可以使用 `getvar` 函数。

Facter 可以通过配置 `facter.conf` 文件，在每个主机上进行自定义和调整。默认情况下，此文件不会创建，应在 Nix 系统上的 `/etc/puppetlabs/facter/facter.conf` 和 Windows 上的 `C:\ProgramData\PuppetLabs\facter\etc\facter.conf` 创建。为了测试，可以使用 `-c` 标志运行 `facter` 命令，选择要运行的配置文件。

一个示例 `facter.conf` 组如下所示：

```
facts : {
    blocklist : [ "disks", "dmi.product.serial_number", "file system" ],
    ttls : [
        { "processor" : 30 days },
    ]
}
global : {
    external-dir     : [ "/home/david/external1", "/home/david/external2" ],
    custom-dir       : [ "/home/david/customtest" ],
    no-exernal-facts : false,
    no-custom-facts  : false,
    no-ruby          : false
}
cli : {
    debug     : false,
    trace     : true,
    verbose   : false,
    log-level : "warn"
}
fact-groups : {
 custom-exampleapp : ["exampleapp1", "exampleapp2"],
}
```

第一部分 `facts` 包括一个阻止列表，它允许我们列出将不会运行的事实和事实组。这在计算事实可能会非常耗费资源的情况下很有用。例如，在前面的示例中，我们阻止了 `disks` 和 `file system` 组，因为在一些传统的 UNIX 系统中，SAN 存储可能会配置有成千上万条路径。它还禁用了 `dmi.product.serial_number`，这可能被认为是某些安全信息，不应在 Puppet 中显示。要查看所有可阻止组的完整列表，可以运行 `facter --list-block-groups` 命令，它将列出组名以及其中包含的事实列表。例如，`disks` 组如下所示：

```
disks
- blockdevices
- disks
```

事实部分的另一部分是 `ttls`，它允许配置缓存。缓存的事实以 JSON 格式存储在 UNIX 系统上的 `/opt/puppetlabs/facter/cache/cached_facts` 和 Windows 上的 `C:\ProgramData\PuppetLabs\facter\cache\cached_facts` 中。在前面的示例中，`processor` 组将每 30 天刷新一次。要查看所有可缓存组的完整列表，可以运行 `facter --list-cache-groups` 命令，这将显示类似于块组的格式。

`global` 部分允许传递一个目录数组到 `external-dir`，以便定义 `facter` 应该在何处查找外部事实。类似地，可以传递一个目录数组到 `custom-dir`，定义 `facter` 应该在何处查找自定义事实。自定义和外部事实将在下一部分中讨论。

`global` 部分有三个布尔值：

+   `no-external-facts`：如果设置为 `true`，则禁用外部事实。

+   `no-custom-facts`：如果设置为 `true`，则禁用自定义事实。

+   `no-ruby`：防止通过 Ruby 加载 Facter。任何使用 Ruby 和自定义事实的事实如果设置为 `true`，则会被禁用。

所有这些设置更可能用于调试和开发目的。

`cli` 部分设置日志级别，值为（`none`、`trace`、`debug`、`info`、`warn`、`error`、`fatal`）的字符串，并有三个选项：`verbose`、`trace` 和 `debug`。这三个选项的启用或禁用通过布尔值 `true` 或 `false` 来设置。`trace` 选项将在自定义事实发生异常时显示回溯。这个选项不应与追踪日志级别混淆；这个选项的更合适名称可能是 stacktrace。`verbose` 选项启用 Facter 的详细信息输出，而 `debug` 选项启用 Facter 的调试级别输出。

`fact-group` 部分是 Facter 4 中 Puppet 新增的功能，允许你为缓存和阻止定义自定义组。可以指定核心事实和自定义事实，但不能指定外部事实。

注意

由于 `facter.conf` 文件使用 HOCON 格式，因此可以通过 Puppet Forge 的 HOCON 模块（[`forge.puppet.com/modules/puppetlabs/hocon`](https://forge.puppet.com/modules/puppetlabs/hocon)）更轻松地管理它，在此过程中可以根据需要按单个节点或节点组进行分类。

Puppet 7 中的 Facter 4 重新引入了事实基准测试功能，这一功能之前在 Facter 2 中就已存在。要对某个特定的事实进行基准测试，可以运行 `facter -t <fact name>` 命令。例如，运行 `facter -t os` 将生成类似于以下的输出：

```
fact 'os.name', took: (0.000007) seconds
fact 'os.family', took: (0.000006) seconds
fact 'os.hardware', took: (0.000007) seconds
```

如果选择了结构化事实，它将对事实的每个部分进行计时，并在执行完后将其返回到 facter 调用的正常输出中。

在了解了核心事实是什么以及如何运行和配置 Facter 来测试和管理它们之后，下一步是通过自定义和外部事实添加个性化配置。

# 自定义事实和外部事实

在本节中，你将学习如何通过自定义事实将其添加到核心事实提供的事实中。这些事实是用 Ruby 编写的，类似于核心事实或外部事实，它们可以是硬编码的值，也可以是客户端本地可执行的脚本。虽然收集所有数据可能很有诱惑力，但应考虑到外部事实对 Puppet 基础架构的额外负担，特别是在有大量代理的情况下，并需要平衡数据需求与系统性能。

## 外部事实

外部事实是可执行文件，它们可以根据脚本中的逻辑设置事实，或者根据文件的结构化数据静态设置事实。

外部事实可以存储在以下目录中，适用于基于 Unix 的操作系统：

+   `/``opt/puppetlabs/facter/facts.d/`

+   `/``etc/puppetlabs/facter/facts.d/`

+   `/``etc/facter/facts.d/`

对于 Windows 系统，外部事实可以存储在 `C:\ProgramData\PuppetLabs\facter\facts.d\` 中。

在 *第八章* 中，你将学习如何通过插件同步过程将外部事实从模块分发到客户端，在此过程中，模块中 `facts.d` 文件夹内的事实会被添加进来。

注意

Puppet 可以在基于 UNIX 的系统上作为非 root 用户运行，而外部事实可以存储在 `~/.facter/facts.d/` 中。然而，本书不会涉及作为非 root 用户运行的相关内容。

### 静态外部事实

静态外部事实必须采用 JSON、YAML 或 TXT 格式。例如，我们可以将 `Application` 事实设置为 `exampleapp`，将 `Use` 事实设置为 `production`，将 `Owner` 事实设置为 `exampleorg`。在 YAML 文件中，可以像这样创建：

```
---
Application :  exampleapp
Use : Production
Owner : exampleorg
```

在 JSON 文件中，可以像这样设置它们：

```
{ "Application": "exampleapp", "Use": "Production", "Owner": "exampleorg"}
```

在 TXT 文件中，同样的事实可以像这样设置：

```
Application=exampleapp
Use=Production
Owner=exampleorg
```

对于 Windows，这些文件中使用的行结束符必须是 LF（换行符，Unicode 字符 000A）或 CRLF（回车符和换行符，Unicode 字符 000D 和 000A），且文件的编码必须是 ANSI 或 UTF8 且不带 BOM。

到目前为止我们所看过的示例都被称为**平面事实**。然而，通过创建数组格式，可以返回结构化的事实。例如，在 YAML 中，我们可以通过添加数组和嵌套数组来允许两个所有者。在这个示例中，假设有多个应用程序，每个应用程序可以有联合所有权：

```
---
Application :
  Exampleapp
Use : production
Owner
- Exampleorg
- anotherorg
  Anotherapp
Use : Production
Owner : exampleorg
```

这将允许我们调用 `facter application.exampleapp.owner` 来检索所有者数组，或者调用 `facter application.anotherapp` 来接收使用者和所有者的键值对。

请注意，静态外部事实在输出中将始终返回字符串类型。

### 可执行外部事实

可执行外部事实在 Windows 和 UNIX 上有所不同，但它们都是可运行的脚本，输出键值对或数组以返回事实或结构化事实。

在 Windows 上，可以使用以下文件类型：

+   二进制可执行文件（`.com` 和 `.exe` 文件）

+   批处理脚本（`.bat` 和 `.cmd` 文件）

+   PowerShell 脚本（`.ps1` 文件）

在 UNIX 平台上，任何具有有效 shebang (`#!`) 声明的可执行文件都可以运行。如果缺少 shebang 声明，脚本执行将失败。

对于两个平台，这些脚本应该返回文本。文本将被读取为键值对，或作为 YAML 或 JSON，可以解析成结构化事实。

例如，一个返回 `exampleapp` 进程 PID 作为事实的 Unix bash 脚本，同时返回 `exampleapp_cpu_use` 和 `example_memory_use` 的事实，可能如下所示：

```
#!/bin/bash
echo "exampleapp_pid = ${pidof exampleapp}"
echo "exampleapp_cpu_use = ${ps -C exampleapp} %cpu"
echo "exampleapp_memory_use = ${ps -C exampleapp} %mem"
```

对于 Windows，一个 PowerShell 脚本返回相同的事实将如下所示：

```
Write-Output "exampleapp_pid=$((Get-Process explorer).id)"
Write-Output "exampleapp_cpu=$(Get-Process explorer).cpu)"
Write-Output "exampleapp_mem=$(Get-Process explorer).pm)"
```

注意

要查找外部事实的问题，可以运行 `facter --debug`。这将显示事实是否对 Facter 可见，以及是否有任何输出未被解析并被忽略。

## 自定义事实

自定义事实是可以用来设置事实并扩展核心 Facter 事实的 Ruby 代码段。使用自定义事实相较于外部事实的主要优势在于其内置的机制。在本节中，您将了解如何使用自定义事实访问其他事实的值，如何进行多个加权解析，以及如何使用 `confine` 确保只有特定的节点会尝试运行该事实。

使用自定义事实的主要缺点是它们需要用 Ruby 编写，而 Ruby 存在学习曲线。深入学习 Ruby 的细节超出了本书的范围，但本书将展示其基本结构以及一些在 Windows 和 UNIX 系统上运行良好的核心库，以便您能够为进一步研究打下基础。

与外部事实类似，自定义事实通常通过 Puppet 模块分发。然而，在进行本地测试时，有三种方法可以指示 Facter 查找我们存储本地事实的位置：

+   Ruby 库加载路径

+   在 Facter 命令中使用 `--custom-dir` 选项（请注意，此选项可以多次标记）

+   设置 `FACTERLIB` 环境变量

Ruby 库加载路径可以通过运行 `ruby -e 'puts $LOAD_PATH'` 来检查。记得确保所使用的 Ruby 二进制文件是 Puppet 提供的版本，在 Windows 上是 `C:\Program Files\Puppet Labs\Puppet\puppet\bin\ruby.exe`，在 UNIX 系统上是 `/opt/puppetlabs/puppet/bin`。

自定义事实使用 `Facter.add('<fact_name>')` 声明自己，并使用 `setcode` 语句运行代码块来解析事实。这就是事实值确定的方式。作为一个简单的例子，可以通过将命令括起来使用反引号（`` ` ``）直接运行 UNIX shell 或 Windows 终端命令：

```
Facter.add('exampleapp_version') do
  setcode do
    `exampleapp –version`
  end
end
```

由于只有一个命令，这也可以用单个 `setcode` 行来编写：

```
Facter.add('exampleapp_version') do
  setcode `exampleapp --version`
end
```

两者都会将 `exampleapp_version` 事实设置为 `exampleapp --version` 命令的输出结果。

如果你的事实更加复杂，需要运行多个命令或处理输出，可以通过 Ruby 类来运行命令。

在以下示例中，`Facter::Core::Execution.execute` Ruby 类将运行名为 `exampleapp` 的命令，并带有 `version` 标志，然后将命令的输出通过管道传递给 `awk`，以打印第二个返回值：

```
Facter::Core::Execution.execute('exampleapp –version' | awk '{print $2}' )
```

可以使用 `powershell` 命令执行 PowerShell 命令，示例如下：

```
Facter::Core::Execution.execute('powershell (Get-WindowsCapability -Online -Name "Microsoft.Windows.PowerShell.ISE~~~~0.0.1.0").state')
```

虽然出于熟悉感，可以将所有操作都当作终端命令来执行，但需要小心，并不是所有终端中可以使用的命令都能正常工作。例如，bash 风格的 `if` 语句无法使用，应当用 Ruby 代码来编写。

调用另一个事实的值到变量中可能会很有用。以下代码将 `os arch` 事实的值存入 `arch` 变量：

```
arch = Facter.value('os.arch')
```

### 限制自定义事实

自定义事实的主要优点之一是可以限制它们将运行的节点。可以通过 `confine` 语句实现，并选择事实和值来匹配运行的事实。`confine` 函数的语法如下所示：

```
confine <fact_name>: '<fact_value>'
```

在 `confine` 函数之后定义的事实，只有在条件满足时才会运行。例如，你可以将事实限制为仅在 Windows 内核节点上运行：

```
confine kernel: 'Windows'
```

也可以使用数组，匹配任何一个值都可以让事实运行。例如，我们可以检查内核是否来自 Linux 或 Solaris：

```
confine kernel: ['Linux', 'Solaris']
```

对于结构化事实，可以使用 `Facter.value` 方法来访问。例如，要测试 `os.release.major` 事实是否等于 `10`，可以使用以下代码，其中 `=>` 被用来代替冒号（`:`）来匹配事实的值：

```
confine Facter.value(:os)['release']['major'] => '10'
```

除了事实之外，Ruby 命令和库命令也可以用来限制事实。例如，`confine` 可以与 `Facter::Core::Execution.where` 或 `Facter::Core::Execution.which` 一起使用，分别用于确认 Windows 或 Linux 的路径中是否存在某个命令。此外，Ruby 库如 File 也可以用来检查这一点。

例如，要限制一个事实，仅在 Windows 路径中找到 `git` 命令时运行，可以运行以下代码：

```
confine { Facter::Core::Execution.where('git') }
```

以下代码会限制事实仅在 `/opt/app/exampleapp` 存在作为文件或目录时运行：

```
confine { File.exist? '/opt/app/exampleapp' }
```

要编写一个可以涵盖多种实现并且具有粒度限制的单一事实，我们可以同时使用多个解析（`Facter.add` 语句）和多个限制块。以下示例展示了一个简单的示例，设置 `whoami` 的 Facter 值为 `I am windows 10`，如果内核事实是 Windows 并且 `os.release.major` 为 `10`，或者设置为 `I am Sparc` 字符串，如果内核是 `sparc`：

```
Facter.add('whoami') do
  setcode do
    confine kernel: 'Windows'
    confine Facter.value(:os)['release']['major'] => '10'
    'I am windows 10'
  end
end
Facter.add('whoami') do
  setcode do
    confine kernel: 'Sparc'
    'I am Sparc'
  end
end
```

另一种限制事实的方法是使用特性。特性是 Ruby 代码的一部分，添加到模块的 `lib/puppet/feature` 目录下。例如，`exampleapp` 模块可以包含一个 `exampleapp.rb` 特性，用来检查 `exampleapp` 是否安装在 Windows 或 Linux 上：

```
require 'puppet/util/feature'
Puppet.features.add(:example_app)
do
windows= `powershell '(Get-Command exampleapp).source'`.strip
linux = `sh -c 'command -v exampleapp`.strip
windows.empty? && linux.empty? ? false : true end
```

然后，自定义事实可以使用 `confine` 语句，这样只有在 `exampleapp` 命令可用的节点上才会运行该事实：

```
Facter.add('exampleapp) do
setcode do
confine { Puppet.features.example_app? }
```

这消除了需要创建额外事实并收集和处理不必要的信息（除了评估限制之外）的需求。

注意

在 `setcode` 和 `confine` 块中执行所有逻辑代码非常重要；否则，在加载事实时，它会运行这些代码，而不是在查询事实进行解析时运行。这是因为事实加载的顺序是不可预测的，因此如果代码被事实要求但位于块外部，可能会导致顺序错误。

### 加权解析

写自定义事实的另一种方法是有多个解析，同时知道某些解析可能返回 null 值，但我们希望逐一尝试不同选项。审查解析时，Facter 会排除所有没有被限制的解析。然后，它会查看每个解析的权重。默认情况下，权重为 `0`，但可以通过 `has_weight` 函数进行设置。如果两个解析的权重相同，Facter 会使用代码中列出的第一个解析。

例如，要使用多个解析选项设置 `exampleapp_version` 事实，在第一个解析中，它将以 `100` 权重运行带有 `version` 标志的命令，然后尝试以 `50` 权重在配置文件中查找版本：

```
Facter.add('exampleapp_version') do
has_weight 100
setcode do
`exampleapp --version`
end
Facter.add('exampleapp_version') do
has_weight 50
setcode do
`grep version /etc/exampleapp/exampleapp.conf | awk '{print $2}'`
end
```

这允许命令失败，从而可以通过第二个来源进行备份。

注意

外部事实的权重为 `1000`。因此，为了防止外部事实覆盖自定义事实解析，可以将解析权重设置为高于 `1000` 的值。

### 异常处理块

默认情况下，如果任何解析失败并产生错误，Facter 将报告错误并无法返回任何值。使用`rescue`块可以在失败时返回默认值，并选择打印警告。这与加权解析配合使用，在加权解析中，通常会预期解析失败。

一个简单的`rescue`块，在运行`exampleapp –version`命令并记录失败后，返回`nil`，看起来像这样：

```
setcode do
`exampleapp --version`
rescue
  nil
  Facter.warn("exampleapp command failed")
end
```

使用`Facter.warn`可以确保当通过`Facter`命令使用时，这条消息会打印到 STDERR。当在 Puppet catalog 应用过程中使用时，它将确保该消息打印到 Puppet 的日志中。返回`nil`将确保其他解析可以在返回非 nil 值时被使用。

### 超时

作为 Puppet 7 中的 Facter 4 的一部分，现在可以为解析添加超时。可以通过在事实的名称后添加逗号，作为`Facter.add`解析语句的一部分，并使用`{timeout: <秒数值>}`语法来实现，其中秒数值可以是整数或浮动值。例如，要确保`exampleapp_version`事实的超时时间为 0.2 秒，代码可以像这样设置：

```
Facter.add('exampleapp_version', {timeout: 0.2}) do
```

尽管这是 Facter 4 和 Puppet 7 中的一个功能，但在 Facter 3 和 Puppet 5 及更高版本中，仍然可以通过直接在`execute`函数上设置`options`变量来对执行命令设置超时。例如，可以通过修改`execute`命令来将相同的 0.2 秒超时应用于`exampleapp –version`命令的执行，而不是整个解析：

```
Facter::Core::Execution.execute('exampleapp --version', options = {:timeout => 0.2})
```

### 聚合和结构化事实

聚合事实允许将事实的解析分成多个块。然后，这些块可以合并。合并数组或哈希会创建结构化的事实或执行其他功能，例如将事实的值相加。

聚合事实仍然有一个`Facter.add`声明，但在`Facter.add`内，它将类型变量设置为`aggregate`。然后，代替使用`setcode`部分，它使用`chunk`部分来进行解析。默认情况下，每个`chunk`都会被合并，除非声明了聚合块来执行其他功能。

例如，以下代码将创建一个名为`exampleapp`的结构化事实。它将包含`exampleapp.version`和`exampleapp.fullpath`，其中包含在块中运行的命令的输出：

```
Facter.add(:exampleapp, :type => :aggregate) do
  Chunk(:version) do
`exampleapp –version`
  end
  Chunk(:fullpath) do
`which exampleapp`
  end
end
```

要使用聚合块并将事实合并，可以使用以下代码，它创建了一个名为`exampleapp_memory_usage`的事实，该事实使用一个包含`exampleapp`使用的总内存的事实，并将其添加到`exampleapp2`使用的内存中，从而得出总内存使用情况：

```
Facter.add(: exampleapp_memory_usuage, :type => :aggregate) do
  chunk(:exampleapp1_usage) do
    Facter.value(:exampleapp1_usage)
  end
  chunk(:exampleapp2_usage) do
    Facter.value(:exampleapp2_usage))
  end
  aggregate do |chunks|
    total = 0
    chunks.each_count do |value|
      total += value
    end
    total
  end
end
```

Puppet 7 与 Facter 4 中提供了一种新的返回结构化事实的方法。这采用了事实名称中的点表示法，允许定义将不同层次的结构化事实进行赋值。例如，要设置带有嵌套层次的`exampleapp`事实，包括`exampleapp.version`和`exampleapp.pid`，可以使用以下代码：

```
Facter.add('exampleapp.version') do
setcode do
`exampleapp --version`
end
Facter.add('exampleapp.pid') do
setcode do
`pidof exampleapp`
end
```

这相比使用聚合有一个核心优势。与聚合不同，声明中某一部分的失败只会影响该声明，其他部分将继续赋值。

注意

本节试图为你提供足够的信息，以便你开始使用自定义事实。在 Puppet 的自定义事实和模块代码文档中，你会发现许多我们讨论过的功能的替代语法。由于它是 Ruby 代码，声明的方式有更多变化。本书选择了它认为最好的风格和实践，以保持简洁并避免列出过多选项。

一些可以帮助你进一步跟随示例的模块可以在 GitHub 上找到：

[`github.com/puppetlabs/puppetlabs-pe_status_check/blob/main/lib/facter/`](https://github.com/puppetlabs/puppetlabs-pe_status_check/blob/main/lib/facter/)

[`github.com/puppetlabs/puppetlabs-stdlib/tree/main/lib/facter`](https://github.com/puppetlabs/puppetlabs-stdlib/tree/main/lib/facter)

[`github.com/puppetlabs/puppetlabs-lvm/tree/master/lib/facter`](https://github.com/puppetlabs/puppetlabs-lvm/tree/master/lib/facter)

[`github.com/puppetlabs/puppetlabs-java/tree/main/lib/facter`](https://github.com/puppetlabs/puppetlabs-java/tree/main/lib/facter)

## 实验

对于这个实验，我们将创建一个静态外部事实和一个自定义事实，并使用`bolt upload`进行分发，然后运行这些事实并在控制台上查看它们是否已变得可见。

对于静态外部事实，创建一个结构，将`packtlab.use`设置为`lab`，并将`packlab.student`设置为你的名字。

对于自定义事实，将创建一个`tmp_count`事实，它将计算 Linux 中`/tmp`目录和 Windows 中`C:\Users\admin\AppData\Local\Temp`目录中的文件数量。对于 Linux，第一个权重较高的解析应为`'find /tmp -type f | wc -l'`，而第二个权重较低的解析应为`ls /tmp | wc -l`。对于 Windows，第一个权重较高的解析应为`(ls $env:Temp | Measure-Object -line).Lines` PowerShell 命令，权重较低的解析应为`(Get-ChildItem $env:Temp | Measure-Object).Count`。

所有解析应在错误结果中返回`undef`，并且在 10 秒后超时。

请注意，在 Web 控制台查看客户端当前的事实可能很有用，这样你就能知道如何限制它们。

对于每个事实，使用以下`bolt`命令将其上传到正确的位置：

```
bolt file upload path_of_your_fact /path/to/destination --targets windows_server_fqdn linux_sever_fqdn
bolt task run facts --targets windows_server_fqdn linux_sever_fqdn
```

前往 Web 控制台并查看节点中的事实，以确认它们是否已出现在客户端。

你可以在 [`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch05/tmp_count.rb`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch05/tmp_count.rb) 和 [`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch05/packlab.yaml`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch05/packlab.yaml) 找到示例解决方案。

注意

在测试自定义或外部事实时，可以通过设置环境变量手动设置它们，在 UNIX 环境中使用 `export FACTER_exampleapp ="test"`，或者在 Windows 环境中使用 `env FACTER_exampleapp="test"` —— 这样会强制设置 `exampleapp` 事实。此方法仅适用于自定义或外部事实，而不适用于核心事实。

# 函数

函数是可以在目录编译过程中运行的 Ruby 代码段，允许你修改目录或计算并返回值。Puppet 提供了许多内建函数，更多的函数可以通过 Puppet Forge 模块提供，如 [`forge.puppet.com/modules/puppetlabs/stdlib`](https://forge.puppet.com/modules/puppetlabs/stdlib)，或者通过自定义编写的函数添加到模块中。本书不会涵盖编写函数的内容，但可以在 [`puppet.com/docs/puppet/latest/writing_custom_functions.html`](https://puppet.com/docs/puppet/latest/writing_custom_functions.html) 找到完整的指南。

本节将介绍三种不同类型的函数：语句函数、前缀函数和链式函数。我们将展示一组核心的 Puppet 函数，并按目的分组，展示最常用和最有用的函数。

注意

许多函数已从如 `stdlib` 模块等源移入核心 Puppet 函数。完整列表可在 [`puppet.com/docs/puppet/6/release_notes_puppet.html#release_notes_puppet_x-0-0`](https://puppet.com/docs/puppet/6/release_notes_puppet.html#release_notes_puppet_x-0-0) 中查看。

## 语句函数

语句函数是 Puppet 语言提供的函数，仅用于它们的副作用，这些函数总是返回 `undef` 值。语句函数可以省略括号，不像我们将在本节中介绍的其他函数那样。你不能添加自定义的或由 Forge 提供的语句函数。

### 目录语句

目录语句会影响目录的内容，允许类被包含，依赖关系和包含关系影响目录的顺序，并且可以应用标签。以下是目录语句的示例语法：

```
Include <class name>
require <class name>
contain <class name>
tag <tag name> , *<tag name>
```

`Include` 和 `tag` 的使用已在 *第三章* 中讨论，但我们没有详细探讨 `tag` 函数。`tag` 函数用于在类中标记该类，并将标签或标签列表应用于所有包含的对象。

在 *第六章* 中，我们将详细介绍 `require` 和 `contain` 的完整使用。

### 日志语句

日志语句允许将字符串消息发送到 Puppet 服务器的日志输出。在 *第十章* 中，将全面回顾服务器和代理的日志记录，因为日志位置取决于配置以及使用的是 Puppet 企业版还是开源版。日志语句的语法就是 `<logging` `level>(<string >)`。

可以应用以下日志级别：

+   `debug`

+   `info`

+   `notice`

+   `warning`

+   `err`

+   `fail`

要记录 `'code unexpected'` 的警告信息，Puppet 代码如下：

```
warning('code unexpected')
```

字符串消息可以包含变量，如果它们被双引号包围以进行插值。因此，为了在 pa-risc 架构系统上产生 `'pa-risc is unsupported'` 错误消息，可以在错误函数的字符串中使用 Facter `os.arch` 的事实：

```
error("${facts['os']['arch']} is unsupported")
```

这与本书之前使用的示例不同，特别是前一章示例中使用的 `notify` 资源。`notify` 资源会返回到客户端的日志，而日志级别函数则会记录到 Puppet 服务器上。由于 `notify` 是资源而不是函数，每次调用 `notify` 资源时，报告都会显示该资源发生了变化。

`fail` 与其他级别不同，因为将其作为函数调用时会终止编译，并且不会将目录发送到代理。

## 前缀和链式函数

Puppet 函数可以通过两种方式调用，对于许多函数，两种方式都可以适用。

前缀函数通过编写函数名称然后提供一个括号中的参数列表来调用：

```
function_name(argument, *argument)
```

链式函数是通过一个参数、一个句点 (`.`)，然后是函数名称和括号，以及括号内的任何其他参数来创建的：

```
argument.function_name(argument, *argument)
```

## 一些内置函数的选择

核心 Puppet 中有许多可用的函数，本节将对不同函数进行分组，展示它们如何使用，或指明本书中的哪些地方会更详细地介绍它们。本章的目的是展示函数的多样性，而不是给出每个函数的完整语法。你可以在 [`puppet.com/docs/puppet/latest/function.html`](https://puppet.com/docs/puppet/latest/function.html) 查阅完整的函数列表。确保选择适合你正在使用的 Puppet 环境的文档版本。

### 比较和大小测量

以下函数允许你比较和测量变量的大小。它们提供了比数据类型直接操作更多的功能。

`length` 和 `size` 函数实际上是相同的，都可以作为前缀或链式函数应用于数组（元素数量）、哈希（键值对数量）、字符串（字符数）或二进制数据（字节数），以确认变量的相对大小/长度。例如，以下命令将返回 `4` 作为字符串 "four" 的长度，返回 `5` 作为数组的大小：

```
Stringwithfour = 'four'.length()
Array_of_five = Size([8,4,5,7,0])
```

`match` 用作字符串或字符串数组上的链式函数，结合正则表达式匹配模式。它返回一个数组，包含第一个匹配的字符串，后跟匹配的模式。如果没有匹配模式，则会返回一个示例，其中字符串必须以小写字母开头，长度为 `6` 到 `8` 的数字。变量匹配 `a123456` 并返回一个包含 `[ 'a123456', 'a' , '123456' ]` 的数组：

```
$matches = "a123456".match(/([a-z]{1})([1-9]{6,8})/)
```

如果我们在一个不匹配的字符串 `1a23456` 上尝试相同的正则表达式，则会返回 `undef`：

```
$nomatch = "1a23456".match(/([a-z]{1})([1-9]{6,8})/)# $matches contains [abc123]
```

使用字符串数组（`'a123456'`、`'b1254678'` 和 `'1a23456'`）和相同的正则表达式，会导致 `multi_match` 变量包含一个数组的数组。如果对每个字符串单独使用 `match`，则输出将是：

```
$multi_match = ['a123456','b1254678','1a23456'].match(/([a-z]{1})([1-9]{6,8})/)
```

这意味着 `multi_match` 将包含 `[['a123456','a','123456'],['b1254678','b','1254678'],undef]`。

`max` 和 `min` 用作前缀函数。它们接受一个字符串或数字的数组，并返回每种情况中的最大值和最小值。在 Puppet 6.0 之前，关于如何转换和处理这些函数中使用的混合类型有一些指导意见。然而，由于它已经被弃用，现在强烈建议确保比较的类型一致。在以下示例中，`highest number` 变量将包含 `88`，而 `lowest letter` 变量将包含 `'a'`：

```
$highest_number = max( [5,3,88,46] )
$lowest_letter = ['d','b','a'].min()
```

`empty` 用作前缀或链式函数，用来确认一个数组或哈希是否不包含元素，或者一个字符串或二进制是否为空。在以下示例中，`empty_array` 和 `empty` 字符串将包含 `true`，而 `non_empty_string` 变量将包含 `false`：

```
$empty_array = [].empty
$empty_string =empty('')
$nonempty_string='not_empty'.empty()
```

`compare` 用作前缀函数，用于比较两个值，并返回 `-1`、`0` 或 `1`，分别表示第一个值小于、等于或大于第二个值。两个值必须为相同类型，可以是数字、字符串、时间段、时间戳或语义版本。对于两个字符串，可以使用第三个参数（布尔值）来检查比较是否忽略大小写。

例如，`numeric_compare` 变量将包含 `-1`，而 `string_compare` 变量将包含 `1`，因为大写字母大于小写字母，`A` 会排在 `b` 前面。如果布尔值设置为 `true`，则返回 `1`：

```
$numeric_compare = compare(5 , 6)
$string_compare = compare('A', 'b', false)
```

### 改变大小写

以下函数用于改变字符串或字符串的数组/哈希的大小写。对于整数，它们保持不变，并可能包含其他无法计算的数据类型错误。

`capitalize`、`camelCase`、`downcase` 和 `upcase` 都用作前缀或链式函数来改变字符串或可迭代对象（如数组）中字符串的大小写。`downcase` 和 `upcase` 也可以在数组上使用。它们都可以用于数字类型，但会返回未受影响的数字。

CamelCase 会去除应用时使用的所有下划线 (`_`)。`camelCase` 和 `capitalize` 对数组不是递归的，而 `upcase` 和 `downcase` 是递归的。

如果 `downcase` 或 `upcase` 在递归使用时更改了数组中的键，并且因此产生了重复项，它将覆盖该键，使用最后更新的键值对替代。为了举例说明，`upper_case` 变量在将整个字符串转换为大写后将包含一个名为 `UPANDDOWN` 的字符串，而 `downcase` 变量在将键都转换为小写并覆盖第一个时，将包含一个哈希值 `{'lower' => 'case2'}`：

```
$upper_case = 'UpAnDdOwN'.upcase()
```

`capitals` 变量在将数组中的每个字符串首字母大写后，将包含一个名为 `['Up, Mix']` 的数组：

```
$capitals =capitalize(['down','miX'])
```

`downcase` 变量在将键值都转换为小写并覆盖第一个值后，将包含一个哈希值 `{'lower' => 'case2'}`：

```
$downcase = {'lower' = > 'case', 'Lower => 'Case2}.downcase()
```

`camel` 变量在去除下划线并将大写方式设置为 `camelCase` 后，将包含 `Word1Word2Word3`：

```
$camel = camelCase('word1_word2_word3')
```

如果你使用国际字符，你需要检查 Ruby 系统的区域设置是如何处理这些字符的，因为它用于处理大小写转换。

### 字符串操作

`lstrip`、`rstrip` 和 `strip` 函数可以移除字符串中的空格。它们都是前缀或链式函数，用于从字符串中移除空格。`lstrip` 移除前导空格，`rstrip` 移除尾随空格，`strip` 同时移除前导和尾随空格（如空格、制表符、换行符和回车符），但不包括硬空格。它们可以在字符串或可迭代对象上使用，但不能递归使用。如果用于数字类型，它们将返回未经调整的数字类型，但在任何其他不支持的类型上会产生错误。

以下示例使用了所有三个函数，最终将使 `left` 变量包含 `'first second'`，`right` 变量包含 `'first second'`，并且 `all` 变量包含 `'firstsecond'`：

```
$spaces = " first second "
$left = $spaces.lstrip()
$right = rstrip($spaces)
$all = $spaces.strip()
```

### 闭包

这些函数本身不是闭包，但在与闭包一起使用时最为有用，因为它们允许对数组或哈希变量进行迭代或转换，并传递给闭包，闭包是 Puppet 代码的一个部分。以下函数用于定义变量的行为：`all`、`any`、`break`、`each`、`filter`、`index`、`lest`、`map`、`next`、`return`、`reduce`、`reverse_each`、`step`、`then`、`tree_each`、`unique` 和 `with`。

这些函数的语法和行为将在*第六章*中详细讲解，但为了举个例子，这里我们使用了 `each` 函数和一个包含用户名键及其对应用户 ID 数字的哈希。`each` 函数可以将每对键值作为一个数组，并允许为用户资源创建已分配的 ID：

```
$usersids = {'admin' => 1, 'operator' => 2, 'viewer' => 3}
$userids.each |$users| {
  user { $users[0]:
    id  => $users[1]
  }
}
```

注意

许多函数可以使用 lambda 进行错误处理，这使得您可以循环处理错误部分、消息和问题代码，并允许采取更详细的消息或动作。这将在*第六章*中讨论。

### 模板

模板允许您通过简单的输入替换创建复杂的文本。在*第六章*中，我们将详细讨论模板，但`template`和`epp`函数允许通过`file`资源的`content`属性使用 ERB 和 EPP 格式的模板。使用 ERB 格式并通知`content`属性的示例可以在`exampleapp`模块中找到：

```
file { '/etc/exampleapp.conf':
  ensure  => file,
  content => template(exampleapp/exampleapp.conf.erb')
}
```

模块的结构以及如何存储模板文件将在*第八章*中介绍。

或者，为了使用包含模板格式的字符串并传递值，可以使用`inline_template`和`epp_inline`。例如，要使用 EPP 样式模板，假设`$exampleapp_conf_template`包含 EPP 模板格式的字符串，`inline_epp`将替换端口和调试变量值`exampleapp_port`和`exampleapp_debugging_enabled`：

```
file { '/etc/ntp.conf':
  ensure  => file,
  content => inline_epp($exampleapp_conf_template, {'port' => $exampleapp_port, 'debugging' => $exampleapp_debugging_enabled}),
}
```

### 哈希/数组

以下函数用于访问和操作哈希和数组数据，超出了*第四章*中讨论的常规操作符，或者用于将变量转换为哈希和数组。

`dig`函数用于通过提供各种键或索引在复杂数据结构中进行查找。当结构不明确时，它特别有用。例如，假设我们有一个名为`exampleapp_proc`的数据结构，并且我们想访问 ID 为`124`的进程状态。如果我们尝试使用哈希索引如`exampleapp_proc['exampleapp_pids']['124']['state']`来访问，但哈希中没有`124`这个键，我们将收到错误，目录运行将失败。然而，通过使用`dig`函数，通知将为未定义：

```
$exampleapp_proc = { exmpleapp_pids => { 123 => { state => running , user => root } }
notice exampleapp_proc.dig('exampleapp_pids','124','state')
```

`getvar`函数用于使用点符号返回结构化变量的部分。如果变量不存在，它将返回`undef`，而不是抛出错误，这与直接访问结构化变量不同。如果没有找到值，您还可以设置默认值；否则，它将返回`undef`。

第一个命令使用`getvar`访问`os.release.full`事实，而第二个命令在未找到结构化事实时将返回`'not_found'`：

```
getvar('facts.os.release.full')
getvar('facts.os.release.full','not_found')
```

`join`函数用于将一个数组转换为使用指定分隔符的元素字符串。例如，如果你有一个数据中心位置数组`dc_locations = ['london', 'falkirk', 'portland', 'belfast']`，你可以使用`join`函数打印一个以冒号分隔的字符串，表示这些位置；例如，`notice(join(${dc_locations}, ":"))`。这将在通知中生成字符串`"london:falkirk:portland:belfast"`：

```
dc_locations = [ 'london','falkirk','portland','belfast']
notice ( join(${dc_locations}, ":")
```

然而，如果你对包含嵌套数组的数组使用`join`，它将展平数组，但不会影响哈希或哈希中的数组。例如，`join([{London => ['bromley', 'brentford']}, 'Berlin', 'Falkirk', 'Grangemouth'], '@@')`将打印`[ { London => [ 'bromley', 'brentford' ] }@@Berlin@@Falkirk@@Grangemouth ]`，因为数组的第一个元素是哈希，它不会被展平，尽管它包含了哈希：

```
dr_locations = [ { London = > [ 'bromley','brentford']},Berlin,['Falkirk','Grangemouth']]
notice ( join(${dr_locations}, "@@")
```

`keys`和`values`函数接受一个哈希并返回哈希中键的数组，可以作为前缀或链式函数运行。例如，要打印`offices`变量的键列表，前两个`notice`函数将打印数组`['Germany','Holland']`，而接下来的两个将打印数组`['Berlin',Amsterdam']`：

```
$offices = {'Germany' => 'Berlin', 'Holland' => 'Amsterdam'}
notice(keys(${offices})
notice($offices.keys())
notice(values(${offices})
notice($offices.values())
```

这些键或值将与它们在哈希中声明时的顺序相同。如果哈希为空，则返回一个空数组。

`split`函数接受一个字符串，并使用一个模式来表示字段分隔符，可以将字符串拆分为一个数组元素。这个模式可以是字符串、正则表达式或正则表达式。以下示例展示了如何使用不同的模式方法进行拆分，并选择不同的分隔符或多个分隔符：

```
$exmple_split = north@south.east@west
$split_on_at = split($example_split, /@/)
$split_on_fullstop = split($example_split, '[.]'
$split_on_both = split($example_split, Regexp['[.@]')
```

`split_on_at`变量将包含数组`['north','south.east','west']`，`split_on_fullstop`将包含数组`['north@south ','east@west']`，而`split_on_both`将包含数组`['north','south','east','west']`。

`sort`函数接受一个数组，并按数字顺序或字典顺序对数组进行排序。无法混合这些排序方式，也无法同时进行数字和字典值的排序，否则会导致错误且没有转换。字符比较基于系统语言环境，并且区分大小写，除非使用`compare`和匿名函数。

在最简单的形式下，`sort`将按升序对数字和字符串进行排序——例如，我们可以拿一个无序的数字数组和一个无序的字符串数组，并使用`sort`作为前缀或链式函数。在这个示例中，代码将得到按升序排列的数字`[0,1,2,3,4,5,7,8,9]`和按升序排列的字符串`['a','b','c','d']`：

```
$unordered_numbers = [7,9,8,0,2,4,3,1,5]
$unordered_strings = ['d','c','b','a']
$ordered_numbers = $unordered_numbers.sort()
$ordered_strings = sort($unordered_strings)
```

为了明确指定顺序，你可以使用`compare`函数对变量进行排序，强调它们应该是升序还是降序。在以下示例中，整数将在升序变量中按`[1950,1980,1984,1985]`排序，而在降序变量中按`[1985,1984,1980,1950]`排序：

```
$ascending =(sort([1984,1950,1985,1980]) |$a,$b| { compare($a, $b) })
$descending = (sort([1984,1950,1985,1980]) |$a,$b| { compare($b, $a) })
```

正如我们在讨论 *比较和大小* 部分的 `compare` 时学到的，布尔值可以用于 `compare`，以确定是否按大写字母排序。

注意

可以使用 *比较和大小* 部分中的其他函数，如 `max` 或 `min`，来替代使用 `compare` 函数。

### 数据处理

针对 Hiera 和加密的 EYAML，提供了几个与数据相关的函数。它们将在*第九章*中详细讨论，但作为参考，它们是 `eyaml_look_up_key`，`lookup` 和 `yaml_data`。函数文档指出，几个 `hiera_<type>` 函数已被废弃，取而代之的是 `lookup` 函数。

`unwrap` 函数已在*第三章*中讨论过，该函数用于在必要时使敏感数据类型在 Puppet 代码中可见/可访问。

# stdlib 模块函数

模块将在*第八章*中详细讨论，但 `stdlib` 模块（[`forge.puppet.com/modules/puppetlabs/stdlib`](https://forge.puppet.com/modules/puppetlabs/stdlib)）被广泛使用，值得突出一些该模块提供的函数，因为几乎每个 Puppet 安装都会使其可用。

需要注意的是，`stdlib` 中的函数允许一些高级行为，这些行为并不总是 Puppet 代码的最佳实践方式，例如能够将 YAML 文件的内容读取为字符串，并使用 `ensure_package` 函数，后者用于允许对包资源进行多次声明。在复杂的情况下或代码在多个团队的政治环境中进行管理时，它们可以提供有用的变通方法。

注意

许多函数已被文件类型转换所替代，该功能自 Puppet 5 版本开始提供，此外还有其他新特性，但这些函数为了兼容性目的被保留。

### 数组和字符串

以下函数通过合并、操作和以多种方式生成新数组来与字符串和数组进行交互。

`intersection` 函数是一个链式函数，当提供两个数组时，会生成一个包含两个数组中都存在的值的单一数组。例如，以下代码将 `['both']` 数组放入 `chained_array` 变量中：

```
$chained_array = intersection(['first','both']['second','both])
```

`union` 函数是一个链式函数，当提供两个数组时，会生成一个包含唯一值的单一数组。在以下示例中，`union_array` 变量将包含 `['first','second']` 数组：

```
$union_array = union(['first','both'],['second','both']
```

`range` 函数是一个链式函数，提供起始、结束或步长区间（如果没有提供，默认步长为 `1`）。起始和结束可以是字符串或数字，而可选的步长应该是整数。

例如，`onetoten` 变量将包含一个数组 `[1,2,3,4,5,6,7,8,9,10]`，`etog` 变量将包含 `['E','F','G']`，而 `good_trek` 变量将包含 `['StarTrek2','startrek4','startrek6','starttrek8']`：

```
$onetoten = range(1,10)
$etog = range('E','G')
$good_trek = ('StarTrek2', 'StarTrek8', 2)
```

`start_with`和`end_with`函数是链式函数，允许你检查一个字符串是否以提供的字符串或字符串列表开始或结束，尝试匹配列表中的任意字符串。它将根据匹配情况返回`true`或`false`。在以下示例中，`truestart`将包含`true`，因为`server`匹配了`server1234`的开头，`falseend`将包含`false`，因为`wales`并没有以`land`结尾，而`trueoptions`将包含`true`，因为`aws104`以`aws`开头，并且可能匹配以`gcp`或`az`开头的其他字符串：

```
$truestart = 'server1234'.startswith('server')
$falseend = 'wales'.endswith('land')
$trueoptions = 'aws104'.startswith['gcp','az','aws']
```

### 文件信息

`basename`、`dirname`和`extname`函数可以作为单独的函数使用，也可以链式使用，从文件路径中提取文件名、目录或扩展名。以下是一个示例：

```
$full_path = 'C:\Users\david\fact.ps1'
$file_name = basename(${full_path})
$dir_name = dirname(${full_path})
$ext =  ${full_path}.extname
```

请注意，`extname`仅适用于格式为`filename.extension`的文件名。如果字符串不包含点（`.`），或者点出现在字符串的开头或结尾，它将仅返回空字符串。

# 实验

在涵盖了各种功能之后，让我们练习使用其中的一些。我们创建一个名为`example_functions`的类，它接受一个作为字符串的用户前缀和若干个作为整数的用户数量。此类应接受两个参数：一个用户前缀字符串和若干个用户整数。确保前缀是小写的。应从`0`开始创建一个用户名数组，直到指定的用户数量。然后将该数组传递给用户资源来创建用户。

使用`user`字符串和数字`5`来定义你的类。

代码还应记录一条警告消息，其中包含`os.windows.product_name`事实的内容，或者如果你不使用 Windows 机器，则为`linux`。

最后，代码应采用`fact`路径，并确保每个目录都经过审计。提示：你可能需要将此路径拆分为一个数组，并将其传递给文件资源。`windows`和`linux`使用不同的路径分隔符——即`；`和`:`。以下的`if`语句应该能帮助你：

```
if $facts['os.family'] == 'windows' {
}else{
}
```

你应使用`bolt`使`stdlib`在我们的客户端上可用：

```
bolt command run "puppet module install stdlib" -t windowsclient linuxclient
```

然后，通过以下命令使用`bolt`应用`puppet`类：

```
bolt puppet apply example_functions.pp -t windowsclient linuxclient
```

你可以在[`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch05/example_functions.pp`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch05/example_functions.pp)找到一些示例解决方案。

# 延迟函数

`Deferred`函数（也称为代理端函数）是应用了`Deferred`类型的函数。这会导致该函数在应用目录时在客户端本地运行，而不是在 Puppet 服务器编译时运行。一个延迟函数的目录包含要在客户端运行的内容，而不是函数的输出。`Deferred`类型是在 Puppet 6.0 中引入的，并且在所有后续版本中可用。

当编译服务器无法访问函数中所需的源时，通常会使用此方法——例如，当从 HashiCorp Vault 服务器检索秘密时，安全设置只允许客户端访问秘密。

应用`Deferred`的语法如下：

```
Deferred( name of function, [arguments])
```

以下是从`vault`检索秘密的示例。这可以在`exampleapp`的用户资源中使用，从 Vault 路径`exampleapp/password`设置密码：

```
user { 'exampleapp':
password => Deferred('vault_lookup::lookup', ["exampleapp/password"])
}
```

该函数来自`vault_lookup`模块（[`forge.puppet.com/modules/puppet/vault_lookup`](https://forge.puppet.com/modules/puppet/vault_lookup)），并需要根据模块中的说明以及 Hashicorp 的指南：[`www.hashicorp.com/resources/agent-side-lookups-with-hashicorp-vault-puppet-6`](https://www.hashicorp.com/resources/agent-side-lookups-with-hashicorp-vault-puppet-6)）设置底层的 Vault 客户端。

理解使用带有`Deferred`的函数之间的差异很重要。你不能使用`Deferred`函数将变量传递给字符串。这样会导致目录创建该对象的字符串化版本。在以下示例中，涉及从`vault`查找名为`exampleapp/message`的键值，第一个`notify`将返回一个字符串，其中包含目录中函数名称的字符串化翻译，而第二个`notify`将返回`vault lookup`本身的值：

```
$deferred =>  Deferred('vault_lookup::lookup', ["exampleapp/message"])
notify {'this will return the object name':
  message => "Secret message is ${deferred"
}
notify {'this will return the message':
message => $deferred
}
```

这反映了在编译时计算字符串值的目录编译过程。这种不匹配可能会出现在其他地方，例如模板中，但可以通过确保任何延迟的值仅在单独使用或在其他延迟函数中使用来克服。在*第七章*中，你将学习如何使用延迟模板。

函数只能在使用核心数据类型时延迟，因为客户端在运行时通过插件同步仅提供核心数据类型。在*第十章*中，你将学习插件同步如何与客户端协作。

同时需要注意，是否返回敏感值以及如何失败，取决于函数本身的实现。对于`vault_lookup`函数，无法优雅地失败；它将返回一个错误，导致目录运行出错。

注意

从 Puppet 7.17.0 开始，延迟函数现在可以按需调用，而不是预处理。使用此方法，目录可以为延迟函数提供输入。如果延迟函数失败，则只有受影响的资源会失败，而所有其他资源仍然会被应用。要启用此行为，设置`Puppet[:preprocess_deferred] = false`或使用`--no-preprocess_deferred`。

所有这些行为适用于本地的`puppet apply run`，因为`puppet apply run`将生成目录并在本地应用。

# 摘要

在本章中，你学习了 Facter 工具如何通过其事实提供系统配置文件信息，以及如何使用外部事实和自定义事实来扩展这些信息。我们提醒你，收集事实会产生基础设施成本，且应平衡其适用的规模。我们指出，外部事实可以是操作系统允许的简单静态数据的平面文件或可执行脚本。尽管自定义事实是用 Ruby 编写的，但它们相比外部事实具有若干优势。能够将自定义事实仅限于在某些系统上运行，可以让你选择不同的分辨率，并根据权重选择应被选中的分辨率，以及在 Puppet 7 中的分辨率级别或 Puppet 6 及以下版本中的执行级别设置超时。

接下来，我们回顾了函数，并强调了函数在操控目录或返回计算值方面可以完成的广泛任务。在这里，我们讨论了目录语句，它们用于在目录中包含类，以及日志记录语句，它们用于设置日志信息。我们还突出了另外两种类型的函数，即前缀函数和链式函数，并展示了它们的语法。然后，我们展示了一个核心函数的选择，并介绍了暴露可用函数的各种类别。

然后，我们讨论了`stdlib`模块中的一小部分函数，重点介绍了可以提供的内容。请注意，一些`stdlib`函数已经被弃用，仅用于向后兼容或处理极限情况，这并不是最佳实践。

最后，我们讨论了延迟函数，这些函数允许在客户端应用目录时运行。我们强调了这对某些只对客户端可用的服务（例如对安全服务进行 API 调用）或可能不希望在与其他服务共享的 Puppet 基础设施上运行的服务的优势。

在下一章中，你将学习资源和类之间的关系和依赖关系如何工作。我们将查看作用域和包含性如何影响资源、变量和类，以及如何构建代码和必要的依赖关系，而不陷入常见的陷阱和依赖地狱。

# 第二部分——在 Puppet 语言中结构化、排序和管理数据

本部分将介绍更高级的 Puppet 语言特性。我们将展示如何使用迭代和条件来管理代码中的依赖关系和流程。接着，我们将看到如何使用最佳实践将 Puppet 结构化为模块，采用角色和配置模式。Puppet Forge 将展示为一个有用的预构建模块源，我们将查看如何理解和审查这些模块的源代码和内容。接着，我们将查看如何使用 Hiera 管理 Puppet 中的数据，并了解何时使用独立的数据源和变量的最佳实践。

本部分包括以下章节：

+   *第六章*，*关系、排序与作用域*

+   *第七章*，*模板化、迭代**和条件语句*

+   *第八章*，*开发与管理模块*

+   *第九章*，*使用 Puppet 处理数据*
