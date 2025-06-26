# 第七章：运行你的工作流

在本章中，我将向你展示运行你的**工作流**的不同选项。我们将研究托管和自托管的运行器，我将解释如何使用不同的托管选项解决混合云场景或硬件在环测试。我还会向你展示如何设置、管理和扩展自托管的运行器，并展示如何进行监控和故障排除。

以下是本章将要涵盖的核心主题：

+   托管运行器

+   自托管的运行器

+   通过运行器组管理访问权限

+   使用标签

+   扩展你的自托管运行器

+   监控和故障排除

# 托管的运行器

我们在前一章中已经使用过**托管运行器**。托管运行器是 GitHub 托管的虚拟机，可用于运行你的工作流。这些运行器支持**Linux**、**Windows**和**macOS**操作系统。

## 隔离与权限

工作流中的每个作业都在一个新的虚拟机实例中执行，并且是完全隔离的。你拥有*完全管理员访问权限*（在 Linux 上为**无密码 sudo**），并且在 Windows 机器上禁用了**用户账户控制**（**UAC**）。这意味着你可以在工作流中安装任何需要的工具（这只会增加构建时间的代价）。

运行器还可以访问**用户界面**（**UI**）元素。这使得你可以在运行器内部执行**UI 测试**，例如**Selenium**，无需通过其他虚拟机来执行这些测试。

## 硬件

GitHub 在**Microsoft Azure**的**Standard_DS2_v2**虚拟机上托管 Linux 和 Windows 运行器。Windows 和 Linux 虚拟机的硬件规格如下：

+   2 核 CPU

+   7 GB 的内存

+   14 GB 的 SSD 硬盘空间

MacOS 运行器托管在 GitHub 的 macOS 云上，具有以下硬件规格：

+   3 核 CPU

+   14 GB 的内存

+   14 GB 的 SSD 硬盘空间

## 软件

在*表 7.1*中，你可以看到当前可用镜像的列表：

![表 7.1 – 当前可用的托管运行器镜像](img/Table_012.jpg)

表 7.1 – 当前可用的托管运行器镜像

你可以在 [`github.com/actions/virtual-environments`](https://github.com/actions/virtual-environments) 找到当前的列表和所有包含的软件。

这是你可以提交问题的代码库，如果你希望请求将某个工具作为默认工具安装。这个代码库还包含有关运行器所有重大软件更新的公告，你可以使用 GitHub 仓库的*watch*功能来接收新版本发布的通知。

## 网络

托管的运行器使用的 IP 地址会不定期变更。你可以通过 GitHub API 获取当前的列表：

```
curl \
```

```
  -H "Accept: application/vnd.github.v3+json" \
```

```
  https://api.github.com/meta
```

你可以在 [`docs.github.com/en/rest/reference/meta#get-github-meta-information`](https://docs.github.com/en/rest/reference/meta#get-github-meta-information) 查找更多信息。

如果您需要一个允许列表来防止互联网访问您的内部资源，您可以使用这些信息。但请记住，每个人都可以使用托管运行器并执行代码！阻止其他 IP 地址并不能使您的资源更安全。不要将内部系统与这些您无法信任其安全性的 IP 地址对立！这意味着系统必须及时修补并具备安全的身份验证。如果做不到这一点，您必须使用自托管运行器。

注意

如果您为您的 GitHub 组织或企业账户使用了 *IP 地址允许列表*，则不能使用 GitHub 托管的运行器，必须使用自托管的运行器。

## 定价

对于公共仓库，使用托管运行器是免费的。根据您的 GitHub 版本，您将获得一定的存储量和每月免费的构建分钟（见 *表 7.2*）：

![表 7.2 – 不同 GitHub 版本中包含的存储和构建分钟](img/Table_021.jpg)

表 7.2 – 不同 GitHub 版本中包含的存储和构建分钟

如果您通过 **Microsoft 企业协议** 购买了 **GitHub Enterprise**，您可以将您的 **Azure** **订阅 ID** 连接到 GitHub Enterprise 账户。这使得您可以为额外的 **GitHub Actions** 使用付费，除此之外还包括您 GitHub 版本中提供的内容。

在 Windows 和 macOS 运行器上运行的作业消耗的构建分钟比 Linux 更多！Windows 消耗的分钟是 *因素 2*，而 macOS 是 *因素 10*。这意味着使用 1,000 个 Windows 分钟将消耗您账户中 2,000 个分钟，而使用 1,000 个 macOS 分钟将消耗 10,000 个分钟。

这是因为构建分钟的费用更高。您可以支付超出 GitHub 版本中包含的分钟的额外费用。以下是每个操作系统的构建分钟费用：

+   在 Linux 上：$0.008

+   在 macOS 上：$0.08

+   在 Windows 上：$0.016

    提示

    您应该尽可能多使用 Linux 进行工作流，并将 macOS 和 Windows 降到最低，以减少构建成本。Linux 还具有最佳的启动性能。

额外存储的费用在所有运行器中都是相同的，收费为每 GB $0.25。

如果您是按月计费的客户，您的账户将默认设置为 $0（美元）的支出限额。这可以防止使用额外的构建分钟或存储空间。如果您按发票付款，您的账户将默认没有支出限额。

如果您配置的支出限额高于 $0，您将在超过账户中包含的分钟或存储量后被计费，直到达到支出限额为止。

# 自托管运行器

如果你需要比 GitHub 托管的 runner 提供更多的控制（例如硬件、操作系统、软件和网络访问），可以自行托管 runner。**自托管的 runner** 可以安装在物理机器、虚拟机或容器中。它们可以在本地或任何公共云环境中运行。

自托管的 runner 允许从其他构建环境轻松迁移。如果你已经有自动化构建，只需在机器上安装 runner，代码就应该能够构建。但如果你的构建机器仍然是手动维护的“脚踏式”机器——有时还放置在开发者桌面之外——那么这并不是一个永久解决方案。请记住，构建和托管一个动态扩展的环境需要专业知识并且耗费资金，无论它是托管在云端还是本地。因此，如果可以使用托管的 runner，通常这是更简单的选择。然而，如果你需要一个自托管的解决方案，请确保将其设计为具有弹性可扩展的解决方案。

注意

托管你自己的 runner 使你能够在 **GitHub Enterprise Cloud** 内的本地环境中安全地构建和部署。这使你能够以 *混合模式* 运行 GitHub——也就是说，你可以将 GitHub Enterprise 与托管的 runner 一起使用进行基本的自动化和云环境的部署，但使用自托管的 runner 来构建或部署托管在本地的应用程序。这可能比运行 **GitHub Enterprise Server** 和为所有构建和部署创建自己托管的构建环境更便宜、更简单。

如果你依赖硬件来测试软件（例如，使用硬件在回路测试时），就必须使用自托管的 runner。这是因为无法将硬件连接到 GitHub 托管的 runner 上。

## runner 软件

该 runner 是开源的，可以在 [`github.com/actions/runner`](https://github.com/actions/runner) 上找到。它支持 Linux、macOS 和 Windows 上的 x64 处理器架构。它还支持 ARM64 和 ARM32 架构，但仅限于 Linux。该 runner 支持多种操作系统，包括 **Ubuntu**、**Red Hat Enterprise Linux** **7** 或更高版本、**Debian** **9** 或更高版本、**Windows** **7**/**8**/**10** 和 **Windows Server**、**macOS** **10.13** 或更高版本等。有关完整列表，请参见 [`docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners#supported-architectures-and-operating-systems-for-self-hosted-runners`](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners#supported-architectures-and-operating-systems-for-self-hosted-runners) 中的文档。

runner 会自动更新，因此你无需自己管理此事。

## runner 与 GitHub 之间的通信

runner 软件使用 `443` 端口通过出站连接轮询 GitHub。它会保持连接 50 秒，如果没有响应则超时。

您必须确保机器可以访问以下网址：

```
github.com
```

```
api.github.com
```

```
*.actions.githubusercontent.com
```

```
github-releases.githubusercontent.com
```

```
github-registry-files.githubusercontent.com
```

```
codeload.github.com
```

```
*.pkg.github.com
```

```
pkg-cache.githubusercontent.com
```

```
pkg-containers.githubusercontent.com
```

```
pkg-containers-az.githubusercontent.com
```

```
*.blob.core.windows.net
```

您无需在防火墙上打开任何入站端口。所有通信都通过客户端进行。如果您为 GitHub 组织或企业使用 IP 地址允许列表，您必须将自托管运行器的 IP 地址范围添加到该允许列表中。

## 在代理服务器后使用自托管运行器

如果您需要在代理服务器后运行自托管运行器，可以这样做。但请注意，这可能会导致很多问题。运行器本身可以正常通信——然而，软件包管理、容器注册表以及所有由运行器执行且需要访问资源的操作都会增加额外开销。如果您能避免这种情况，我建议您这么做。但如果您必须在代理服务器后运行工作流，您可以通过以下环境变量配置运行器：

+   `https_proxy`：此项包括 HTTPS（端口`443`）流量的代理 URL。您还可以包含基本身份验证（例如 `https://user:password@proxy.local`）。

+   `http_proxy`：此项包括 HTTP（端口`80`）流量的代理 URL。您还可以包含基本身份验证（例如 `http://user:password@proxy.local`）。

+   `no_proxy`：此项包括应绕过代理服务器的主机的以逗号分隔的列表。

如果更改环境变量，您必须重新启动运行器才能使更改生效。

使用环境变量的替代方法是使用`.env`文件。在运行器的应用程序文件夹中保存一个名为`.env`的文件。之后，语法与环境变量相同：

```
https_proxy=http://proxy.local:8081
```

```
no_proxy=example.com,myserver.local:443
```

接下来，我们来看看如何将自托管运行器添加到 GitHub。

## 将自托管运行器添加到 GitHub

您可以在 GitHub 上以不同级别添加运行器：仓库、组织和企业。如果在仓库级别添加运行器，它们将专用于该单个仓库。组织级别的运行器可以处理一个组织中多个仓库的工作，而企业级别的运行器可以分配给您企业中的多个组织。

安装运行器并将其注册到 GitHub 实例上非常简单。只需进入**设置** | **操作** | **运行器**，在您想要添加它们的级别。然后，选择操作系统和处理器架构（参见*图 7.1*）：

![图 7.1 – 安装自托管运行器](img/B17827_07_01.jpg)

图 7.1 – 安装自托管运行器

这将为您生成一个脚本，执行以下操作：

1.  下载并解压运行器

1.  配置运行器的相应值

1.  启动运行器

脚本的第一部分始终创建一个名为`actions-runner`的文件夹，然后将工作目录更改为该文件夹：

```
$ mkdir actions-runner && cd actions-runner
```

在 Linux 和 macOS 上，使用`curl`命令下载最新的运行器包，在 Windows 上使用`Invoke-WebRequest`：

```
# Linux and macOS:
```

```
$ curl -o actions-runner-<ver>.tar.gz -L https://github.com/actions/runner/releases/download/<ver>/actions-runner-<ver>.tar.gz
```

```
# Windows:
```

```
$ Invoke-WebRequest -Uri https://github.com/actions/runner/releases/download/<ver>/actions-runner-<ver>.zip -OutFile actions-runner-<ver>.zip
```

出于安全原因，会验证下载包的哈希值，以确保包没有被篡改：

```
# Linux and macOS:
```

```
$ echo "<hash> actions-runner-<ver>.tar.gz" | shasum -a 256 -c
```

```
# Windows:
```

```
$ if((Get-FileHash -Path actions-runner-<ver>.zip -Algorithm SHA256).Hash.ToUpper() -ne '<hash>'.ToUpper()){ throw 'Computed checksum did not match' }
```

然后，运行器会从 ZIP/TAR 文件中提取：

```
# Linux and macOS:
```

```
$ tar xzf ./actions-runner-<ver>.tar.gz
```

```
# Windows:
```

```
$ Add-Type -AssemblyName System.IO.Compression.FileSystem ; [System.IO.Compression.ZipFile]::ExtractToDirectory("$PWD/actions-runner-<ver>.zip", "$PWD")
```

配置通过使用 `config.sh` / `config.cmd` 脚本完成，网址和令牌由 GitHub 自动为您创建：

```
# Linux and macOS:
```

```
$ ./config.sh --url https://github.com/org --token token
```

```
# Widows:
```

```
$ ./config.cmd --url https://github.com/org --token token
```

配置会要求输入运行器组（默认是 `Default` 组）、运行器的名称（默认是机器名称）以及额外的标签。默认标签用于描述自托管状态、操作系统和处理器架构（例如，`self-hosted`、`Linux` 和 `X64`）。默认工作文件夹是 `_work`，不应更改。在 Windows 上，您还可以选择将操作运行器作为服务运行。在 Linux 和 macOS 上，您必须在配置后使用另一个脚本安装服务：

```
$ sudo ./svc.sh install
```

```
$ sudo ./svc.sh start
```

如果您不想将运行器作为服务运行，您可以使用 `run` 脚本以交互方式运行它：

```
$ ./run.sh
```

```
$ ./run.cmd
```

如果运行器正在运行，您可以在**设置** | **操作** | **运行器**下看到它，并查看其状态和标签（见*图 7.2*）：

![图 7.2 – 自托管运行器及其标签和状态](img/B17827_07_02.jpg)

图 7.2 – 自托管运行器及其标签和状态

现在让我们学习如何从 GitHub 中移除这些自托管的运行器。

## 移除自托管运行器

如果您想重新配置或移除 GitHub 上的运行器，您必须使用带有 `remove` 选项的 `config` 脚本。如果您通过点击运行器名称查看其详细信息，您将看到一个**移除**按钮（见*图 7.2*）。点击此按钮后，它会为您生成脚本和令牌。

![图 7.3 – 运行器详细信息](img/B17827_07_03.jpg)

图 7.3 – 运行器详细信息

对于不同的操作系统，脚本看起来如下所示：

```
# Linux and macOS
```

```
./config.sh remove --token <token>
```

```
# Windows
```

```
./config.cmd remove --token <token>
```

在销毁机器之前，务必先移除运行器！如果您忘记这样做，您仍然可以在**移除**对话框中使用**强制移除此运行器**按钮。但这应仅作为最后手段。

# 使用运行器组管理访问权限

如果您在组织或企业级别注册运行器，`Default` 是无法删除的。

注意

一个运行器只能属于一个运行器组。

要管理访问权限，请在企业级别打开**策略**，或在组织级别打开**设置**，并在菜单中找到**操作** | **运行器组**。在这里，您可以创建一个新的运行器组，或者点击现有的运行器组以调整其访问设置。根据您的级别是企业级别还是组织级别，您可以选择允许对特定的组织或仓库访问（见*图 7.3*）：

![图 7.4 – 运行器组的选项](img/B17827_07_04.jpg)

图 7.4 – 运行器组的选项

警告

默认情况下，公共仓库的访问权限是禁用的。请保持这个设置！你不应该在公共仓库中使用自托管运行器！Fork 的仓库可能会在你的运行器上执行恶意代码，因此这是一个风险。如果你需要为公共仓库使用自托管运行器，请确保使用*临时*和*强化*的运行器，它们无法访问你的内部资源。如果你需要为一个开源项目使用特定的工具，而这些工具在托管运行器上安装太慢，这种情况可能适用。但这种情况较为少见，你应尽量避免。

当你注册一个新的运行器时，会要求你输入运行器组的名称。你也可以将此作为参数传递给`config`脚本：

```
$ ./config.sh --runnergroup <group>
```

现在我们已经学习了如何通过运行器组来管理访问权限，接下来我们将学习如何使用标签。

# 使用标签

GitHub Actions 通过搜索正确的标签将你的工作流与运行器匹配。在注册运行器时会应用这些标签。你也可以将它们作为参数传递给`config`脚本：

```
$ ./config.sh --labels self-hosted,x64,linux
```

你可以稍后通过点击**标签**旁边的齿轮图标来修改标签，并在运行器的详细信息中创建新标签（见*图 7.4*）：

![图 7.5 – 为运行器创建新标签](img/B17827_07_05.jpg)

图 7.5 – 为运行器创建新标签

如果你的工作流有特定的需求，你可以为其创建自定义标签。自定义标签的一个例子可以是为工具添加标签，比如`matLab`或者必要的`gpu`访问权限。

所有自托管运行器默认都有`self-hosted`标签。

要在工作流中使用运行器，你需要通过标签的形式指定需求：

```
runs-on: [self-hosted, linux, X64, matlab, gpu]
```

这样，你的工作流就能找到满足必要需求的相应运行器。

# 扩展你的自托管运行器

在现有构建机器上安装操作运行器可以轻松迁移到 GitHub。但这不是一个长期解决方案！如果你不能使用托管运行器，你应该自行构建一个弹性扩展的构建环境。

## 临时运行器

如果你为构建机器或容器构建了一个弹性扩展解决方案，你应该使用临时运行器。这意味着你使用一个虚拟机或**Docker**镜像从空白镜像开始，并安装一个临时运行器。然后，运行结束后所有内容都会被删除。我们不推荐使用持久运行器的弹性扩展解决方案！

要将你的运行器配置为*临时*，你需要将以下参数传递给`config`脚本：

```
$ ./config.sh --ephemeral
```

## 使用 GitHub webhooks 进行扩展

为了上下扩展你的虚拟环境，你可以使用`workflow_job` webhook，如果新的工作流被排队，`queued`操作键会被调用。你可以使用这个事件启动一个新的构建机器并将其添加到机器池中。如果工作流运行完成，`workflow_job` webhook 会调用`completed`操作。你可以利用这个事件清理并销毁机器。

更多信息，请参见文档：[`docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#workflow_job`](https://docs.github.com/en/developers/webhooks-and-events/webhooks/webhook-events-and-payloads#workflow_job)。

## 现有解决方案

在`@jonico`构建的弹性虚拟构建环境中，已整理了一个包含所有现有解决方案的矩阵。你可以在[`github.com/jonico/awesome-runners`](https://github.com/jonico/awesome-runners)找到这个仓库。矩阵以 GitHub 页面的形式呈现，通常更易于阅读，因此你可能更愿意访问[`jonico.github.io/awesome-runners`](https://jonico.github.io/awesome-runners)。该矩阵根据目标平台、是否支持 GitHub Enterprise、自动扩展能力、清理因素及其他标准来比较这些解决方案。

提示

请记住，使用自定义镜像构建和运行可扩展的构建环境需要大量时间和精力，这些时间和精力本可以用于其他事务。使用托管运行器是更便宜、更可持续的解决方案。在进行投资前，确保你确实需要为自己的平台做出这样的投入。通常，托管自己的运行器还有其他选择——例如将自己的 Docker 镜像带入 GitHub Actions，或者使用机器人自动化部署到本地资源。

# 监控与故障排除

如果你在使用自托管运行器时遇到问题，有几个方面可以帮助你进行故障排除。

## 检查运行器的状态

你可以在`Idle`、`Active`或`Offline`状态下检查运行器的状态。如果运行器状态是`Offline`，可能是机器关机、未连接网络，或自托管运行器应用未在机器上运行。

## 审查应用程序日志文件

日志文件保存在运行器的根目录中的`_diag`文件夹下。你可以查看运行器的`_diag`。应用程序日志文件名以`Runner_`开头，并附加有 UTC 时间戳：

```
Runner_20210927-065249-utc.log
```

## 审查工作日志文件

`_diag`。每个工作都有自己的日志。应用程序日志文件名以`Worker_`开头，并附加有 UTC 时间戳：

```
Worker_20210927-101349-utc.log
```

## 检查服务状态

如果你的运行器作为服务运行，你可以根据操作系统检查服务状态。

### Linux

在 Linux 上，你可以从运行器文件夹中的`.service`文件中获取服务名称。使用`journalctl`工具来监控运行器服务的实时活动：

```
$ sudo journalctl -u $(cat ~/actions-runner/.service) -f 
```

你的服务配置可以在`/etc/systemd/systemd/`下检查和定制：

```
$ cat /etc/systemd/system/$(cat ~/actions-runner/.service)
```

### macOS

在 macOS 上，你可以使用`svc.sh`脚本检查服务的状态：

```
$ ./svc.sh status
```

上述脚本的输出包含*服务名称*和*进程 ID*。

要检查服务配置，请在以下位置找到文件：

```
$ cat /Users/<user_name>/Library/LaunchAgents/<service_name>
```

### Windows

在 Windows 上，你可以使用**PowerShell**来获取关于服务的信息：

```
$ Get-Service "action*"
```

使用`EventLog`监控你服务的最近活动：

```
Get-EventLog -LogName Application -Source ActionsRunnerService
```

## 监控跑步器更新过程

跑步器应自动更新自身。如果更新失败，跑步器将无法运行工作流。你可以在`Runner_*`日志文件中检查其更新活动，日志文件位于`_diag`目录下。

# 案例研究

*Tailwind Gears*的两个试点团队在新平台上开始了他们的第一个冲刺。他们自动化的第一件事是构建过程，以便所有的拉取请求在合并前都能被构建。Tailwind Gears 尽可能多地使用 GitHub 托管的跑步器。大多数软件构建都非常顺利。然而，一些用**C**编写的代码使用了旧版编译器，并且在当前的构建机器上安装了其他依赖项。这些代码目前在两台由开发人员自己维护的本地**Jenkins**服务器上构建。这些服务器还连接着用于硬件在环测试的硬件设备。为了顺利过渡，自托管的跑步器被安装在这些机器上，并且构建过程正常。IT 部门反正打算淘汰本地服务器，因此他们与 GitHub 合作，构建了一个弹性、可扩展、基于容器的解决方案，可以运行具有访问附加硬件的自定义镜像。

# 摘要

在本章节中，你了解了两种运行工作流的托管选项：

+   GitHub 托管的跑步器

+   自托管跑步器

我们解释了自托管跑步器如何允许你在混合云场景中运行 GitHub。你学习了如何设置自托管跑步器，并了解了在哪里可以找到帮助你构建弹性可扩展构建环境的信息。

在下一章中，你将学习如何使用**GitHub Packages**管理你的代码依赖。

# 深入阅读

有关本章节内容的更多信息，你可以参考以下资源：

+   *使用 GitHub 托管的跑步器*: [`docs.github.com/en/actions/using-github-hosted-runners`](https://docs.github.com/en/actions/using-github-hosted-runners)

+   *托管你自己的跑步器*: [`docs.github.com/en/actions/hosting-your-own-runners`](https://docs.github.com/en/actions/hosting-your-own-runners)

+   *awesome-runners – 一份精心整理的、包含大量对比矩阵的自托管 GitHub Action 跑步器解决方案清单*: [`jonico.github.io/awesome-runners`](https://jonico.github.io/awesome-runners)
