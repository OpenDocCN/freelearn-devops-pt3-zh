

# Docker 基础

本章我们介绍 DevOps 工具包的一个基础构件——容器。我们将解释虚拟化和容器之间的区别，并展示这两种解决方案的优缺点。此外，我们还将展示如何根据工作负载选择适合的解决方案。

本章覆盖的主要主题如下：

+   虚拟化与容器化

+   Docker 架构

+   Docker 命令

+   Dockerfile

+   Docker 镜像注册表

+   Docker 网络

# 技术要求

本章你需要一台安装了 Docker 引擎的 Linux 系统。我们这里不会涉及安装步骤。不同的 Linux 发行版提供 Docker 的方式不同。我们将使用 Docker 引擎版本 20.10.23。由于本章的所有示例都非常基础，较旧版本的 Docker 很可能也能正常工作。不过，如果你在跟随我们的示例时遇到问题，更新 Docker 到我们这个版本应当是排查问题的第一步。

# 虚拟化与容器化

本节我们将解释虚拟化和容器化是什么，它们之间的主要区别是什么。

## 虚拟化

虚拟化是一种在另一台计算机内运行完整模拟计算机的技术。完整意味着它模拟了物理计算机所具有的所有硬件：主板、BIOS、处理器、硬盘、USB 端口等。模拟意味着它完全是软件的产物。这台计算机在物理上并不存在，因此被称为虚拟计算机。为了存在，虚拟机（**VM**）需要一台真实的物理计算机来模拟它。物理计算机被称为宿主机或虚拟化管理程序（Hypervisor）。

所以，我有一台物理计算机。它非常强大。我为什么要在它上运行虚拟机呢？显而易见，虚拟机的性能会比主机差：毕竟，主机需要为自己分配 RAM、CPU 和硬盘空间。与物理机相比，虚拟机的性能也会有一些小幅下降（因为我们实际上是在运行一个模拟完整硬件的程序）。

理由因使用场景而异，但有很多。

你可能想要运行一个与你自己的操作系统不同的完整操作系统，用来测试一些软件，运行你当前操作系统中没有的软件，或者是为了忠实地重建你的应用程序开发环境。你可能想尽可能精确地重建一个生产环境来测试你的应用程序。这些都是使用虚拟机的有效且非常流行的理由。

让我们来看看虚拟化的优势：

+   **隔离**：正如前面所提到的，虚拟机（VMs）表现为完全功能的计算机。对运行中的操作系统来说，它们创造了与物理机器分离的假象。我们在虚拟机中运行的任何程序都不应能访问主机计算机（除非明确允许），实际上，除了几个由于编程错误导致的事故，虚拟机一直提供安全的环境。这种隔离在恶意软件分析、运行需要独立服务器的工作负载等场景中是一个非常好的解决方案。举例来说，如果一个虚拟机运行一个单独的 WWW 服务器，那么服务器的安全漏洞可能会使攻击者获取操作系统的访问权限，从而让他们可以自由操作。但由于其他基础设施组件（例如数据库）是运行在独立的虚拟机中的，这个问题只能局限于 WWW 服务器。

+   **调优**：借助足够强大的主机，可以对其资源进行分区，以确保每个运行中的虚拟机都有保证的内存、硬盘空间和 CPU。

+   **操作系统简化**：当运行各种工作负载时，比如数据库、WWW 服务器和邮件服务器，维持单一服务器同时运行这些服务的复杂性会迅速增加。每安装一个软件就需要安装额外的软件（例如库文件和辅助程序）。不同程序所需的库可能会引发不兼容的问题（尤其是当我们安装的不是操作系统开发者发布的软件，即所谓的第三方程序时）。在少数情况下，甚至操作系统自带的软件之间也可能存在不兼容的问题，使得在一个操作系统上安装它们变得不可能或非常困难。维护这样的系统可能会变得麻烦，并需要大量的排查工作。现代的虚拟机管理软件通过克隆、快照、黄金镜像等方式缓解了许多系统管理的难题。

+   **自动化**：现代虚拟化软件提供了许多功能，促进了多层次的系统管理自动化。快照——即系统的某一时刻快照——允许在任何时刻回滚到之前的系统状态。这使得轻松回退到最后一个已知的良好状态成为可能，避免了不想要的变化。克隆可以让我们基于另一个已经运行并配置好的虚拟机来配置新的虚拟机。黄金镜像是虚拟机的存档镜像，我们可以轻松快速地导入并启动，完全省略了安装过程，并将配置限制到绝对最小化。这也使得环境的可靠重建成为可能。

+   **加速**：正确设置虚拟机工作流可以让我们在几分钟内启动一个新的操作系统，配备自己的服务器或桌面硬件，而不是几小时。这为测试环境、远程桌面解决方案等开辟了新的可能性。

上述列表并非详尽无遗，但应该能清楚地展示为什么虚拟化成为数据中心和托管公司宠爱的技术。我们可以廉价租用的各种服务器，正是虚拟化使公司能够对硬件进行分区的直接结果。

尽管这一解决方案非常出色，但它并不是万灵药，并不是所有东西都应该虚拟化。而且，为每个软件单独运行一个虚拟机，很容易导致资源利用率的开销。100 个虚拟服务器不仅会使用分配给操作系统的 CPU 和 RAM，还会有一部分被用来做主机机器的管理工作。每个虚拟服务器都会占用操作系统所需的磁盘空间，即使它可能与同一服务器上其他 99 个虚拟机是完全一样的——这是一种空间、RAM 和 CPU 的浪费。而且，启动一个新的虚拟机也需要一些时间。诚然，如果一切配置和自动化得当，启动时间会比设置一台新硬件机器要短，但仍然需要时间。

在虚拟化广泛普及之前，操作系统开发人员试图提供一些技术，使系统管理员能够隔离各种工作负载。主要目标是数据中心，在那里一个物理服务器对于一个工作负载（如数据库或 WWW 服务器）来说已经太多，但运行多个工作负载又存在风险（如安全性、稳定性等）。随着虚拟化的普及，人们很快意识到有时使用虚拟化就像用大炮打麻雀。当你只想运行一个小程序时，采购一个全新的服务器（即便是虚拟的）和整个操作系统是没有意义的。因此，通过不断创新出更好的进程隔离特性，容器应运而生。

## 容器化

容器是轻量级的虚拟环境，允许我们以隔离的方式运行单个进程。一个理想配置的容器只包含运行应用程序所需的软件和库。主机操作系统负责操作硬件、管理内存和其他外围任务。容器的主要假设是它并不模拟一个独立的操作系统或独立的服务器。容器中的进程或用户可以轻松发现自己被隔离在其中。缺点是容器不会模拟硬件。例如，你不能用它们来测试新驱动程序。优点是单个容器的硬盘空间占用可能仅为几个兆字节，只需要运行进程所需的内存。

因此，启动一个容器只需要应用程序启动所需的时间。启动时间——BIOS、硬件测试和操作系统启动时间——都被缩短了。所有应用程序不需要的软件可以并且应该被省略。由于容器镜像的体积较小，它们的重新分发时间几乎可以忽略不计，启动时间几乎是瞬时的，构建时间和过程也大大简化。这使得环境的重建变得更加容易。反过来，这使得测试环境的设置变得更简单，而且通常会频繁地部署软件的新版本——有时一天可以进行几千次部署。应用程序的扩展变得更加容易和快速。

上述变化引入了另一种运行应用程序的方法。上述变化的逻辑结果是，容器的维护方式与操作系统不同。你不会在容器内升级软件——你会部署一个包含新版本软件的容器，替换掉过时的版本。这就导致了一个假设：你不应该在容器内保存数据，而是将数据保存在运行时挂载到容器的文件系统中。

Linux 容器化的代表性技术是**Docker**。Docker 所做的一件事可能帮助推动了这场革命，那就是创建了一个容器镜像共享的生态系统。

在 Docker 中，容器镜像是一个简单的存档，包含了应用程序所需的所有二进制文件和库以及一些配置文件。由于镜像体积通常较小，且镜像内部从不包含数据，因此允许人们共享他们构建的镜像是合乎逻辑的。Docker 有一个镜像中心（称为 Docker Hub），提供了一个漂亮的 WWW 界面，以及用于搜索、下载和上传镜像的命令行工具。这个中心允许对镜像进行评分，并给作者提供评论和反馈。

既然我们已经知道容器化是什么，我们可以更深入地了解 Docker 如何在内部工作以及它的运作原理。让我们来看看 Docker 的构成。

# Docker 的构成

Docker 包含多个组件：

+   命令行工具 – Docker

+   主机

+   对象

+   注册表

Docker CLI 工具——`docker`——是管理容器和镜像的主要工具。它用于构建镜像、从注册表中拉取镜像、上传镜像到注册表、运行容器、与容器互动、设置运行时选项，并最终销毁容器。它是一个命令行工具，通过 API 与 Docker 主机进行通信。默认情况下，假设在主机上调用 `docker` 命令，但这并非严格要求。一个 `docker` CLI 工具可以管理多个主机。

主机更有趣。主机运行`dockerd`——一个负责实际执行通过`docker`工具下达的操作的守护进程。正是在这里存储容器镜像。主机还提供诸如网络、存储以及容器本身等资源。

`dockerd`守护进程是容器的心脏。它是一个在主机上运行的后台进程，负责管理容器。`dockerd`管理容器的创建与管理，提供与守护进程交互的 API，管理卷、网络和镜像分发，提供管理镜像和容器的接口，并存储和管理容器与镜像的元数据。它还负责在 Docker Swarm 模式下管理其他进程之间的通信。

## OverlayFS

**OverlayFS**首次作为 Linux 内核版本 3.18 的一部分于 2014 年 8 月发布。最初它是为了提供一种比之前的存储驱动程序**Another UnionFS**（**AUFS**）更高效和灵活的方式来处理容器存储。OverlayFS 被认为是 UnionFS 的下一代，而 UnionFS 是当时 Docker 使用的存储驱动程序。

从 Docker 版本 1.9.0 开始，这个文件系统被作为内置存储驱动程序包含在 Docker 中。从那时起，OverlayFS 成为大多数 Linux 发行版中 Docker 的默认存储驱动程序，并且在各种容器编排平台（如 Kubernetes 和 OpenShift）中得到广泛使用。

OverlayFS 是 Linux 的一个文件系统，允许一个目录覆盖另一个目录。它允许创建一个由两个不同目录组成的*虚拟*文件系统：一个下层目录和一个上层目录。上层目录包含用户可见的文件，而下层目录包含隐藏的*基础*文件。

当访问上层目录中的文件或目录时，OverlayFS 首先在上层目录中查找，如果没有找到，再查找下层目录。如果在上层目录中找到该文件或目录，则使用该版本；如果在下层目录中找到，则使用该版本。

该机制允许创建*覆盖*文件系统，在这种文件系统中，上层目录可以用于添加、修改或删除下层目录中的文件和目录，而无需修改下层目录本身。这在容器化场景中非常有用，其中上层可以用于存储在容器中所做的更改，而下层则包含容器的基础镜像。

## 什么是镜像？

**Docker 镜像**是一个预构建的包，包含了在容器中运行软件所需的所有文件和设置。它包括你的应用程序代码或二进制文件、运行时、系统工具、库和所有需要的配置文件。一旦镜像构建完成，它可以用于启动一个或多个容器，这些容器是提供一致方式运行软件的隔离环境。

启动容器时，你必须选择一个程序作为容器的主进程来运行。如果这个进程退出，整个容器也会被终止。

构建一个 Docker 镜像通常涉及创建一个 Dockerfile，这是一个包含构建镜像指令的脚本。Dockerfile 指定了使用的基础镜像、需要安装的任何附加软件、需要添加到镜像中的文件以及需要应用的任何配置设置。

在构建镜像时，Docker 会读取 Dockerfile 中的指令并执行我们在 Dockerfile 中准备的步骤。

一旦镜像构建完成，它可以被保存并用于启动一个或多个容器。构建镜像的过程也可以通过使用 Jenkins、GitHub 或 GitLab 等工具来自动化，这些工具可以在每次对代码库进行更改时自动构建和测试新的镜像。

结果镜像由一个唯一的 ID（SHA-256 哈希）组成，这是镜像内容和元数据的哈希值，它还可以有一个标签，这是一个可读的字符串，可以用来引用镜像的特定版本。UnionFS 负责在运行容器时合并所有内容。

要检查镜像的元数据和内容部分，可以运行以下命令：

```
admin@myhome:~$ docker pull ubuntu
admin@myhome:~$ docker inspect ubuntu
[
    {
        "Id": "sha256:6b7dfa7e8fdbe18ad425dd965a1049d984f31cf0ad57fa6d5377cca355e65f03",
        "RepoTags": [
            "ubuntu:latest"
        ],
        "RepoDigests": [
            "ubuntu@sha256:27cb6e6ccef575a4698b66f5de06c7ecd61589132d5a91d098f7f3f9285415a9"
        ],
        "Created": "2022-12-09T01:20:31.321639501Z",
        "Container": "8bf713004e88c9bc4d60fe0527a509636598e73e3ad1e71a9c9123c863c17c31",
            "Image": sha256:070606cf58d59117ddc1c48c0af233d6761addbcd4bf9e8e39fd10eef13c1bb7",
        "GraphDriver": {
            "Data": {
                "MergedDir": "/var/lib/docker/overlay2/f2c75e37be7af790f0823f6e576ec511396582ba71defe5a3ad0f661a632f11e/merged",
                "UpperDir": "/var/lib/docker/overlay2/f2c75e37be7af790f0823f6e576ec511396582ba71defe5a3ad0f661a632f11e/diff",
                "WorkDir": "/var/lib/docker/overlay2/f2c75e37be7af790f0823f6e576ec511396582ba71defe5a3ad0f661a632f11e/work"
            },
            "Name": "overlay2"
        },
        "RootFS": {
            "Type": "layers",
            "Layers": ["sha256:6515074984c6f8bb1b8a9962c8fb5f310fc85e70b04c88442a3939c026dbfad3"
            ]
        },
    }
]
```

这里有很多信息，所以我们已经去除了不必要的输出，只保留了我们要关注的信息。你可以看到一个镜像 ID，`GraphDriver`部分下合并的所有目录，以及`RootFS sha256`层。`RootFS`包含了当我们在容器内启动一个进程时，由 UnionFS 创建的整个文件系统。

## 什么是容器运行时？

**容器运行时**（或**容器引擎**）是一个在你的系统上运行容器的软件组件。容器运行时从 Docker 注册表加载容器镜像，监控系统资源，为容器分配系统资源，并管理其生命周期。

有许多正在使用的运行时容器。最著名并且在笔记本电脑上使用得最多的容器运行时是`containerd`——你可能已经在你的系统上安装了它。它是一个高性能的容器运行时，旨在嵌入到更大的系统中。许多云服务提供商使用它，它也是 Kubernetes 的默认运行时。

LXC 是一个使用 Linux 命名空间和 cgroups 提供容器隔离的运行时。它被认为比 Docker（`containerd`）更加轻量级和高效，但也更难使用。

另一个有趣的运行时是 **Open Container 接口容器运行时**（**CRI-O**）。CRI-O 完全符合 **Open Container Initiative** (**OCI**) 规范，这意味着它可以运行任何符合 OCI 规范的容器镜像。此外，它设计为与 Kubernetes Pods 原生兼容，这使得它在与 Kubernetes 的集成上优于其他运行时。

**Rocket** (**rkt**) 是一种替代的容器运行时，旨在比 Docker 更加安全和高效。它使用 **App Container** (**appc**) 镜像格式，并且架构比 Docker 更加简洁。它也不是很常用。

其他值得注意的容器引擎包括 **run Open Container** (**runC**)，这是一个低级别的容器引擎，提供了创建和管理容器的基本功能，以及由 AWS 开发的 Firecracker。

## cgroups

Linux **cgroups**（即 **控制组**）是 Linux 内核的一项功能，允许管理和隔离一组进程的系统资源。Cgroups 允许系统管理员为特定的进程组分配资源，如 CPU、内存和网络带宽，并监控和控制它们的使用。

这可以用于限制特定应用程序或用户使用的资源，或者在共享系统上隔离不同类型的工作负载。

默认情况下，Docker 不会限制容器内应用程序的 CPU 或内存使用。启用此功能非常简单，不需要直接与 cgroups 或内核设置交互——Docker 守护进程会为我们处理这些。

你可以通过在 `docker` `run` 命令中使用 `--memory` 或 `-m` 选项来限制 Docker 容器可以使用的内存量。

例如，使用以下命令以 500 MB 的内存限制运行 `alpine` 镜像：

```
admin@myhome:~$ docker run --memory 500m alpine /bin/sh
```

你可以通过使用适当的后缀（`b`、`k`、`m` 或 `g`）来指定内存限制，单位可以是字节、千字节、兆字节或吉字节。

当你限制容器的内存时，Docker 还会限制容器可以使用的交换内存量。默认情况下，交换内存的限制是内存限制的两倍。也可以通过使用 `--memory-swap` 或 `--memory-swappiness` 选项来限制交换内存。

通过使用 CPU 配额限制（`--cpus` 或 `-c` 选项），可以限制 Docker 容器内应用程序可以使用的 CPU 时间。CPU 配额是容器可以使用的 CPU 时间的相对度量。默认情况下，容器会被分配一定数量的 CPU 配额，容器可以根据其分配的份额来消耗 CPU 时间。例如，如果一个容器有 0.5 个 CPU 配额，那么如果没有其他容器消耗 CPU，它最多可以使用 50% 的 CPU 时间。

其他可用的选项如下：

+   `--cpuset-cpus`：此选项允许你指定容器可以使用的 CPU 核心范围，例如 `0-1` 表示使用前两个核心，或者 `0,2` 表示使用第一个和第三个核心。

+   `--cpu-shares`：允许你为 Docker 容器设置 CPU 时间限制。它指定了容器在给定时间内可以使用的 CPU 时间（以微秒为单位）。时间段由 `--cpu-period` 选项指定。

+   `--cpu-quota` 和 `--cpu-period`：`--cpu-quota` 是 CPU 时间限制（以微秒为单位），`--cpu-period` 是 CPU 时间周期的长度（以微秒为单位）。

`--cpu-quota` 和 `--cpu-period` 选项允许你为容器指定比 `--cpus` 和 `--cpuset-cpus` 选项更精确的 CPU 时间限制。如果你需要更精确地限制容器的 CPU 时间，以防止性能问题或确保应用程序可靠运行，这些选项非常有用。

在这一节中，我们讲解了容器运行时及其工作原理。接下来，我们将探讨 `containerd` 守护进程的命令行界面，以便更轻松、强大地与所有 Docker 组件交互。

# Docker 命令

`containerd` 守护进程使用套接字文件或网络。

你可以使用的最常见命令如下：

+   `build`：允许你使用 Dockerfile 构建新的 Docker 镜像

+   `run`：启动一个新的容器

+   `start`：重新启动一个或多个停止的容器

+   `stop`：停止一个或多个正在运行的容器

+   `login`：用于访问私有注册表

+   `pull`：从注册表下载镜像或仓库

+   `push`：将镜像或仓库上传到注册表

+   `build`：帮助从提供的 Dockerfile 创建镜像

+   `images`：列出你机器上的所有镜像

+   `ps`：列出所有正在运行的容器

+   `exec`：在正在运行的容器中执行命令

+   `logs`：显示容器的日志

+   `rm`：删除一个或多个容器

+   `rmi`：删除一个或多个镜像

+   `network`：用于管理 Docker 网络

+   `volume`：用于管理卷

## docker build

`docker build` 命令用于从 Dockerfile 构建 Docker 镜像。基本语法如下：

```
docker build [OPTIONS] PATH | URL | -
```

`PATH` 是包含 Dockerfile 的目录路径。

`URL` 是指向包含 Dockerfile 的 Git 仓库的 URL。

`-`（破折号）用于从 `stdin` 的内容构建镜像，因此你可以将 Dockerfile 内容从先前命令的输出管道传输给它，例如，从模板生成它。

要从当前目录中的 Dockerfile 构建镜像，你需要运行以下命令：

```
admin@myhome:~$ docker build .
```

你也可以使用一个特定标签来构建镜像，如以下示例所示：

```
admin@myhome:~$ docker build -t my-image:1.0 .
```

你也可以将 `--build-arg` 参数传递给构建命令，以将构建时的变量传递给 Dockerfile：

```
admin@myhome:~$ docker build --build-arg VAR1=value1 -t my-image:1.0 .
```

## docker run

当你运行一个容器时，本质上是将一个 Docker 镜像拿来并在该环境中执行一个进程。镜像是一个容器的蓝图或快照；它是一个只读模板，包含创建容器的指令。正在运行的容器是该镜像的一个实例，但有自己的状态。

`docker run`命令用于从 Docker 镜像启动一个容器。例如，要从`myimage`镜像启动一个容器并运行`/bin/bash`，你可以运行以下命令：

```
admin@myhome:~$ docker run myimage /bin/bash
```

你也可以向`run`命令传递选项，如下例所示：

```
admin@myhome:~$ docker run -d -p 8080:80 --name containername myimage
```

此命令以分离模式启动容器（`-d`选项；它将容器置于后台），将容器中的端口`80`映射到主机的端口`8080`（`-p 8080:80`），并将容器命名为`containername`（`--``name containername`）。

你还可以向容器传递环境变量：

```
admin@myhome:~$ docker run -e VAR1=value1 -e VAR2=value2 myimage:latest
```

Docker 容器在被终止后不会存储数据。为了使数据持久化，你需要使用容器本身以外的存储。在最简单的设置中，这可以是容器外部文件系统中的一个目录或文件。

有两种方法可以实现：创建一个 Docker 卷：

```
admin@myhome:~$ docker volume create myvolume
```

要使用此卷进行数据持久化，你需要在启动容器时挂载它：

```
admin@myhome:~$ docker run –v myvolume:/mnt/volume myimage:latest
```

你还可以绑定挂载本地文件夹（`-v`选项。在这种情况下，你不需要运行 Docker 卷创建命令：

```
admin@myhome:~$ docker run -v /host/path:/mnt/volume myimage:latest
```

你还可以使用`-``w`选项来指定容器内的工作目录：

```
admin@myhome:~$ docker run -w /opt/srv my-image
```

其他有用的选项如下：

+   `--rm`：此选项将在容器停止后删除该容器

+   `-P`，`--publish-all`：此选项将所有暴露的端口（`Dockerfile`中的`EXPOSE`选项）发布到一个随机的本地端口

+   `--network`：此选项将容器连接到指定的 Docker 网络

你可以通过执行`docker run --``help`命令来查看更多可用选项。

## docker start

`docker start`命令用于启动一个或多个已停止的 Docker 容器。例如，要启动一个容器，你可以运行以下命令：

```
admin@myhome:~$ docker start mycontainer
```

`mycontainer`是你想要启动的容器的名称或 ID。你可以使用`docker ps`命令查看所有正在运行和已停止的容器；稍后我们会详细介绍。你也可以一次启动多个容器。为此，你可以列出它们的名称或 ID，用空格隔开。

要启动多个容器，你可以运行以下命令：

```
admin@myhome:~$ docker start mycontainer othercontainer lastcontainer
```

要附加到容器的进程，以便查看其输出，在启动容器时使用`-a`选项：

```
admin@myhome:~$ docker start -a mycontainer
```

## docker stop

此命令用于停止后台运行的容器。该命令的语法与启动容器时相同，区别在于你可以使用的可用选项。

要一次停止多个容器，你可以列出它们的名称或 ID，用空格隔开。

```
admin@myhome:~$ docker stop mycontainer
```

要停止多个容器，你可以运行以下命令：

```
admin@myhome:~$ docker stop mycontainer othercontainer lastcontainer
```

您还可以使用`-t`选项来指定等待容器停止的时间（以秒为单位），然后发送`SIGKILL`信号。例如，要在停止容器之前等待`10`秒，请运行以下命令：

```
admin@myhome:~$ docker stop -t 10 mycontainer
```

您也可以使用`--time`或`-t`来指定在发送`SIGKILL`信号之前等待容器停止的时间。

默认情况下，`docker stop`命令会向容器发送`SIGTERM`信号，这样容器中的进程就有机会进行正常关闭。如果容器在默认的 10 秒超时后仍未停止，将会发送`SIGKILL`信号强制停止。

## docker ps

该命令用于列出正在运行或已停止的容器。当您没有任何选项地运行`docker ps`命令时，它将显示正在运行的容器列表，并附带它们的容器 ID、名称、镜像、命令、创建时间和状态：

```
admin@myhome:~$ docker ps
```

可以使用`-a`选项查看所有容器：

```
admin@myhome:~$ docker ps -a
CONTAINER ID           IMAGE                          COMMAND         
STATUS                          PORTS                    NAMES
a1e83e89948e   ubuntu:latest                                "/bin/bash -c 'while…"   29 seconds ago       Up 29 seconds                                            pedantic_mayer
0e17e9729e9c   ubuntu:latest                                "bash"                   About a minute ago   Exited (0) About a minute ago                            angry_stonebraker
aa3665f022a5   ecr.aws.com/pgsql/server:latest   "/opt/pgsql/bin/nonr…"   5 weeks ago          Exited (255) 8 days ago         0.0.0.0:1433->1433/tcp   db-1
```

您可以使用`--quiet`或`-q`选项只显示容器 ID，这在脚本编写中可能会很有用：

```
admin@myhome:~$
docker ps --quiet
```

您可以使用`--filter`或`-f`选项根据某些标准过滤输出：

```
admin@myhome:~$
docker ps --filter "name=ubuntu"
```

要检查容器使用的磁盘空间，您需要使用`-s`选项：

```
admin@myhome:~$ docker ps -s
CONTAINER ID      IMAGE      COMMAND   CREATED          STATUS          PORTS     NAMES            SIZE
a1e83e89948e   ubuntu:latest   "/bin/bash -c 'while…"   14 seconds ago   Up 13 seconds             pedantic_mayer   0B (virtual 77.8MB)
```

这个容器不会占用额外的磁盘空间，因为它没有保存任何数据。

## docker login

`docker login`命令用于登录 Docker 注册表。注册表是您存储和分发 Docker 镜像的地方。最常用的注册表是 Docker Hub，但您也可以使用其他注册表，如 AWS **Elastic Container Registry**（**ECR**）、Project Quay 或 Google Container Registry。

默认情况下，`docker login`会连接到 Docker Hub 注册表。如果您想登录到其他注册表，您可以将服务器 URL 作为参数指定：

```
admin@myhome:~$ docker login quay.io
```

当您运行`docker login`命令时，它会提示您输入用户名和密码。如果您在注册表中没有帐户，可以通过访问注册表的网站来创建一个帐户。

登录后，您将能够从注册表推送和拉取镜像。

您还可以使用`--username`或`-u`选项来指定您的用户名，使用`--password`或`-p`来指定密码，但由于安全原因，不推荐在命令行中这样做。

您也可以使用`--password-stdin`或`-P`选项通过`stdin`传递密码：

```
admin@myhome:~$ echo "mypassword" | docker login --username myusername --password-stdin
```

它可以是任何命令的输出。例如，要登录到 AWS ECR，您可以使用以下命令：

```
admin@myhome:~$ aws ecr get-login-password | docker login --username AWS --password-stdin 1234567890.dkr.ecr.region.amazonaws.com
```

您还可以使用`--token`或`-t`选项来指定您的令牌：

```
admin@myhome:~$ docker login --token usertokenwithrandomcharacters
```

登录后，您将能够从注册表推送和拉取镜像。

## docker pull

要拉取 Docker 镜像，您可以使用`docker pull`命令，后跟镜像名称和标签。默认情况下，`pull`会拉取标签`latest`（latest 是标签的名称，或镜像的版本）。

例如，使用以下命令从 Docker Hub 拉取`alpine`镜像的最新版本：

```
admin@myhome:~$ docker pull alpine
```

使用以下命令拉取特定版本的`alpine`镜像，例如版本 3.12：

```
admin@myhome:~$ docker pull alpine:3.12
```

你还可以通过在镜像名称中指定注册表 URL 来从不同的注册表拉取镜像。

例如，使用以下命令从私有注册表拉取镜像：

```
admin@myhome:~$ docker pull myregistry.com/myimage:latest
```

## docker push

在构建新镜像后，你可以将该镜像推送到镜像注册表。默认情况下，`push`将尝试将其上传到 Docker Hub 注册表：

```
admin@myhome:~$ docker push myimage
```

要推送特定版本的镜像，例如版本 1.0，你需要在本地将镜像标记为版本 1.0，然后推送到注册表：

```
admin@myhome:~$ docker tag myimage:latest myimage:1.0
admin@myhome:~$ docker push myimage:1.0
```

你还可以通过在镜像名称中指定注册表 URL，将镜像推送到不同的注册表：

```
admin@myhome:~$ docker push myregistry.com/myimage:latest
```

在推送镜像之前，你需要使用`docker login`命令登录到你要推送镜像的注册表。

## docker image

这是一个用于管理镜像的命令。`docker image`的常见用例如下所示：

要列出你机器上可用的镜像，可以使用`docker image` `ls`命令：

```
admin@myhome:~$ docker image ls
```

REPOSITORY          TAG              IMAGE ID       CREATED         SIZE

```
ubuntu           latest        6b7dfa7e8fdb   7 weeks ago    77.8MB
mcr.microsoft.com/mssql/server     2017-latest               a03c94c3147d   4 months ago   1.33GB
mcr.microsoft.com/azure-functions/python   3.0.15066-python3.9-buildenv   b4f18abb38f7   2 years ago    940MB
```

你也可以使用`docker images`命令执行相同的操作：

```
admin@myhome:~$ docker images
REPOSITORY     TAG           IMAGE ID       CREATED        SIZE
ubuntu        latest        6b7dfa7e8fdb   7 weeks ago    77.8MB
mcr.microsoft.com/mssql/server             2017-latest                    a03c94c3147d   4 months ago   1.33GB
mcr.microsoft.com/azure-functions/python   3.0.15066-python3.9-buildenv   b4f18abb38f7   2 years ago    940MB
```

要从 Docker 注册表拉取镜像，请使用以下命令：

```
admin@myhome:~$ docker image pull ubuntu
```

当你指定镜像名称时，可以使用完整的仓库名称（例如，`docker.io/library/alpine`）或仅使用镜像名称（例如，`alpine`），如果镜像位于默认仓库（Docker Hub）中。另请参阅之前部分中讨论的`docker pull`命令。

也可以构建镜像：

```
admin@myhome:~$ docker image build -t <image_name> .
```

另请参阅有关`docker build`命令的部分，了解更多关于构建镜像的详细信息。

要为镜像创建标签，应运行以下命令：

```
admin@myhome:~$ docker image tag <image> <new_image_name>
```

最后，你可以删除一个镜像：

```
admin@myhome:~$ docker image rm <image>
```

请参阅有关`docker rmi`命令的部分，它是此命令的别名。

另一种删除镜像的选项是`docker image prune –`命令。此命令将删除所有未使用的镜像（悬空镜像）。

## docker exec

`docker exec`允许你在运行中的 Docker 容器中执行命令。基本语法如下：

```
docker exec CONTAINER COMMAND ARGUMENTS
```

在前面的示例中，术语的含义如下：

+   `CONTAINER`是要在其中运行命令的容器的名称或 ID

+   `COMMAND`是要在容器中运行的命令

+   `ARGUMENTS`表示命令的任何附加参数（这是可选的）

例如，要在名为`my_container`的容器中运行`ls`命令，可以使用以下命令：

```
admin@myhome:~$ docker exec mycontainer ls
```

## docker logs

`docker logs`用于获取 Docker 容器生成的日志：

```
docker logs CONTAINER_NAME_OR_ID
```

你可以传递给命令的其他选项如下：

+   `--details, -a`：显示日志提供的额外详细信息

+   `--follow, -f`：跟踪日志输出

+   `--since, -t`：仅显示自某个日期以来的日志（例如，2013-01-02T13:23:37）

+   `--tail, -t`：从日志的末尾显示的行数（默认`all`）

它的使用示例如下：

```
admin@myhome:~$ docker logs CONTAINER_ID
```

## docker rm

`docker rm`用于删除一个或多个 Docker 容器：

```
docker rm CONTAINER_NAME_OR_ID
```

它的使用示例如下：

```
admin@myhome:~$ docker rm CONTAINER_ID
```

要列出所有容器，请使用`docker ps -a`命令。

## docker rmi

从本地拉取或构建的镜像可能会占用大量磁盘空间，因此检查和删除未使用的镜像是很有用的。`docker rmi`用于删除一个或多个 Docker 镜像。

以下是它的用法：

```
docker rmi IMAGE
```

它的使用示例如下：

```
admin@myhome:~$ docker rmi IMAGE_ID
```

## docker network

`docker network`命令用于管理 Docker 网络。除了常见的操作（创建、删除和列出），还可以将运行中的 Docker 容器连接（和断开连接）到不同的网络。

也可以通过选择专用的网络插件来扩展 Docker。这里有多个选择，我们只列出一些网络插件及其简短描述。插件也可以用于更高级的设置，如 Kubernetes 集群：

+   **Contiv-VPP** ([`contivpp.io/`](https://contivpp.io/)) 使用**向量数据包处理**（**VPP**）技术，提供一个高效、可扩展和可编程的容器网络解决方案，适用于企业和服务提供商环境，在这些环境中，高性能和可扩展的网络是必需的。

+   **Weave Net** ([`www.weave.works/docs/net/latest/overview/`](https://www.weave.works/docs/net/latest/overview/)) 允许容器之间进行通信，无论它们在哪个主机上运行。Weave Net 创建一个跨多个主机的虚拟网络，使得容器能够以高可用性、冗余和负载均衡的方式进行部署。

+   **Calico** ([`www.tigera.io/tigera-products/calico/`](https://www.tigera.io/tigera-products/calico/)) 是最知名的插件之一。它采用纯 IP 驱动的网络方式，提供了简洁性和可扩展性。Calico 允许管理员定义和执行网络策略，例如根据源、目的地和端口允许或拒绝特定的流量。Calico 设计用于大规模部署，并支持虚拟和物理网络。

使用`docker network`命令时的常见操作如下：

+   创建新网络：

```
    admin@myhome:~$ docker network create mynetwork
```

+   检查网络：

```
    admin@myhome:~$ docker network inspect mynetwork
```

+   删除网络：

```
    admin@myhome:~$ docker network rm mynetwork
```

+   列出网络：

```
    admin@myhome:~$ docker network ls
```

+   连接容器到网络：

```
    admin@myhome:~$ docker network connect mynetwork        CONTAINER_NAME_OR_ID
```

+   从网络中断开容器：

```
    admin@myhome:~$ docker network disconnect mynetwork         CONTAINER_NAME_OR_ID
```

## docker volume

`docker volume`命令用于管理 Docker 中的卷。通过这个命令，你可以列出可用的卷，清理未使用的卷，或者创建一个以便稍后使用。

Docker 支持多种卷驱动程序，包括以下几种：

+   `local`：默认驱动程序；使用 UnionFS 将数据存储在本地文件系统上

+   `awslogs`：使应用程序生成的日志能够存储在 Amazon CloudWatch Logs 中

+   `cifs`：允许你将 SMB/CIFS（Windows）共享挂载为 Docker 卷

+   `GlusterFS`：将 GlusterFS 分布式文件系统挂载为 Docker 卷

+   `NFS`：将**网络文件系统**（**NFS**）挂载为 Docker 卷

还有许多其他驱动程序可供选择。可用驱动程序的列表可以在官方 Docker 文档中找到：[`docs.docker.com/engine/extend/legacy_plugins/`](https://docs.docker.com/engine/extend/legacy_plugins/)。

以下是其使用示例：

+   `docker` `volume ls`

+   `docker volume` `create <volume-name>`

+   `docker volume` `inspect <volume-name>`

+   `docker volume` `rm <volume-name>`

+   `docker volume` `create myvolume`

    `docker run -v` `myvolume:/opt/data alpine`

在本节中，我们已经学习了如何使用命令行界面与所有 Docker 组件进行交互。到目前为止，我们一直在使用公开可用的 Docker 镜像，但现在是时候学习如何构建自己的镜像了。

# Dockerfile

Dockerfile 本质上是一个具有预定结构的文本文件，包含一组构建 Docker 镜像的指令。Dockerfile 中的指令指定了从哪个基础镜像开始（例如，Ubuntu 20.04），安装哪些软件以及如何配置镜像。Dockerfile 的目的是自动化构建 Docker 镜像的过程，以便镜像能够轻松重现和分发。

Dockerfile 的结构是一个命令列表（每行一个命令），Docker（准确来说是`containerd`）使用这些命令来构建镜像。每个命令在镜像的 UnionFS 中创建一个新层，最终生成的镜像是所有层的联合体。我们管理的层越少，最终生成的镜像就越小。

Dockerfile 中最常用的命令如下：

+   `FROM`

+   `COPY`

+   `ADD`

+   `EXPOSE`

+   `CMD`

+   `ENTRYPOINT`

+   `RUN`

+   `LABEL`

+   `ENV`

+   `ARG`

+   `VOLUME`

+   `USER`

+   `WORKDIR`

你可以在官方 Docker 文档网站上找到完整的命令列表：[`docs.docker.com/engine/reference/builder/`](https://docs.docker.com/engine/reference/builder/)。

让我们逐一了解上述列表，理解每个命令的作用以及何时使用它。

## FROM

Dockerfile 从`FROM`命令开始，该命令指定要从哪个基础镜像开始：

```
FROM ubuntu:20.04
```

你还可以使用`as`关键字为此构建命名，并跟上自定义名称：

```
FROM ubuntu:20.04 as builder1
```

`docker build` 将尝试从公共 Docker Hub 注册表下载 Docker 镜像，但也可以使用其他注册表，或使用私有注册表。

## COPY 和 ADD

`COPY`命令用于将文件或目录从主机复制到容器文件系统。以下是一个示例：

```
COPY . /var/www/html
```

你还可以使用`ADD`命令将文件或目录添加到 Docker 镜像中。`ADD`具有比`COPY`更多的功能。它可以自动解压 TAR 压缩文件，并检查源字段中是否有 URL，如果找到 URL，它将从该 URL 下载文件。最后，`ADD`命令还具有`--chown`选项，用于设置目标中文件的所有权。通常情况下，建议在大多数情况下使用`COPY`，只有在需要`ADD`提供的附加功能时才使用它。

## EXPOSE

Dockerfile 中的 `EXPOSE` 命令告知 Docker 容器在运行时监听指定的网络端口。它并不会实际发布端口，而是用来向用户提供容器计划发布哪些端口的信息。

例如，如果一个容器在端口 `80` 上运行 Web 服务器，你应该在 Dockerfile 中包含以下行：

```
EXPOSE 80
```

你可以指定端口是监听 TCP 还是 UDP – 在指定端口号后，添加一个斜杠和 TCP 或 UDP 关键字（例如，`EXPOSE 80/udp`）。如果只指定端口号，则默认使用 TCP。

`EXPOSE` 命令并不实际发布端口。要使端口可用，你需要在运行 `docker` `run` 命令时使用 `-p` 或 `--publish` 选项来发布端口：

```
admin@myhome:~$ docker run -p 8080:80 thedockerimagename:tag
```

这将把主机上的端口 `8080` 映射到容器中的端口 `80`，这样所有到达端口 `8080` 的流量都会被转发到容器中运行的 Web 服务器的端口 `80`。

不管 `EXPOSE` 命令如何，你都可以在运行容器时发布不同的端口。`EXPOSE` 用于告知用户容器计划发布哪些端口。

## ENTRYPOINT 和 CMD

接下来是 `ENTRYPOINT` 命令，它在 Dockerfile 中指定容器启动时应始终执行的命令。它不能通过传递给 `docker` `run` 命令的任何命令行选项来覆盖。

`ENTRYPOINT` 命令用于将容器配置为可执行文件。它类似于 `CMD` 命令，但用于配置容器作为可执行文件运行。通常用于指定容器启动时应运行的命令，例如命令行工具或脚本。

例如，如果你有一个运行 Web 服务器的容器，你可能会使用 `ENTRYPOINT` 命令来指定启动 Web 服务器的命令：

```
ENTRYPOINT ["nginx", "-g", "daemon off;"]
```

如果你想使用不同的参数运行容器，可以使用 `CMD` 命令设置默认参数，这些参数可以在容器启动时被覆盖：

```
ENTRYPOINT ["/usr/bin/python"]
CMD ["main.py","arg1","arg2"]
```

`CMD` 用于指定从镜像启动容器时应执行的命令。以下是一个例子：

```
CMD ["nginx", "-g", "daemon off;"]
```

这里的经验法则是，如果你希望应用程序接受自定义参数，你可以使用 `ENTRYPOINT` 启动一个进程，使用 `CMD` 向其传递参数。这样，你可以通过命令行传递不同的选项，使进程更灵活地执行不同的操作。

## RUN

Dockerfile 中的 `RUN` 命令用于在容器内执行命令。每次执行时，它都会在镜像中创建一个新层。

`RUN` 命令用于安装软件包、创建目录、设置环境变量，以及执行设置容器内环境所需的任何其他操作。

例如，你可以使用 `RUN` 命令来安装一个软件包：

```
RUN apt-get update && apt-get install -y python3 python3-dev
```

你可以使用 `RUN` 命令来创建一个目录：

```
RUN mkdir /var/www
```

你可以使用 `RUN` 命令设置环境变量：

```
RUN echo "export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64" >> ~/.bashrc
```

值得注意的是，Dockerfile 中 `RUN` 命令的顺序很重要，因为每个命令都会在镜像中创建一个新层，最终的镜像是所有层的联合体。因此，如果你期望某些软件包在后续过程中安装，必须在使用它们之前进行安装。

## LABEL

`LABEL` 命令用于向镜像添加元数据。它基本上是将数据的键值对添加到镜像中。这些数据可以用于存储信息，如镜像的版本、维护者以及你在组织中可能需要的其他相关信息。以下是该命令的示例：

```
LABEL maintainer="Chris Carter <chcarter@your.comain.tld>"
```

你也可以在一行中添加多个标签：

```
LABEL maintainer="Chris Carter <chcarter@your.comain.tld>" version="0.2"
```

使用 `docker` `inspect` 命令可以查看添加到镜像上的标签：

```
admin@myhome:~$ docker inspect --format='{{json .Config.Labels}}' <image_name_or_ID>
```

使用 `LABEL` 命令向镜像添加元数据可以帮助用户理解镜像的目的或他们应该向谁询问细节，并有助于管理镜像。

## ENV 和 ARG

`ENV` 命令用于以以下格式设置环境变量：

```
ENV <key>=<value>
```

另一方面，`ARG` 命令用于定义构建时变量。这些变量可以通过 `--build-arg` 标志传递给 `docker build` 命令，并且它们的值可以在 Dockerfile 中使用。

`ARG` 命令用于定义类似于 `ENV` 格式的构建时变量：

```
ARG <name>[=<default value>]
```

`ARG` 命令创建的变量仅在构建过程中可访问，而 `ENV` 命令创建的环境变量则可以被容器内运行的所有进程访问。

在下一章节，我们将详细介绍 Docker 镜像的构建过程，其中 `ARG` 和 `ENV` 被一起使用，以便在构建阶段之间持久化 `ENV` 变量。

## VOLUME

另一个命令是 `VOLUME`。通过它，你可以配置容器在特定位置为卷创建挂载点。卷是一种将数据存储在容器文件系统外部的方式，这意味着即使容器被删除或重新创建，数据仍然可以保持。以下是该命令：

```
VOLUME /opt/postgresql_data
```

使用以下方式指定多个目录：

```
VOLUME /opt/postgresql_data /opt/postgresql_xferlog
```

或者，以下方式也有效：

```
VOLUME ["/opt/postgresql_data", "/opt/postgresql_xferlog"]
```

如果目录中有标记为 `volume` 的数据，当使用 `docker run` 命令运行 Docker 时，将使用该目录的内容创建一个新的卷。这样，即使容器被终止或以其他方式停止，确保在该 Docker 容器运行期间创建的数据不会丢失。这对于数据库尤为重要，正如我们在前面的示例中所建议的那样。

## USER

Dockerfile 中的 `USER` 命令用于设置容器运行时的默认用户。默认情况下，容器以 root 用户身份运行；建议以没有 root 权限的自定义用户身份运行容器。

`USER` 命令用于设置容器运行时的用户，且可以选择设置用户所属的组。例如，你可以使用 `USER` 命令以 `webserver` 用户身份运行容器：

```
USER webserver
```

你还可以指定用户和组：

```
USER webserver:webserver
```

也可以设置用户 ID 和组 ID，而不是使用名称：

```
USER 1001:1001
```

`USER` 命令仅设置容器的默认用户，但你在运行容器时可以覆盖该设置：

```
admin@myhome:~$ docker run --user=root webserver-image
```

出于安全原因，以非 root 用户身份运行应用程序是最佳实践。如果攻击者获得容器访问权限，这样可以限制潜在的损害，因为运行具有完全权限的进程在宿主机上也会以相同的 UID（此处为 `root`）运行。

## WORKDIR

Dockerfile 中的 `WORKDIR` 命令用于设置容器的当前工作目录。工作目录是容器文件系统中所有后续 `RUN`、`CMD`、`ENTRYPOINT`、`COPY` 和 `ADD` 命令执行的位置。

你可以使用 `WORKDIR` 命令将工作目录设置为 `/usr/local/app`：

```
WORKDIR /usr/local/app
```

使用 `WORKDIR` 时，你在使用其他命令时无需设置文件的完整路径，还可以将应用程序位置参数化（使用 `ARG` 或 `ENV`）。

现在我们已经熟悉了 Dockerfile 以及如何构建 Docker 镜像，接下来了解如何以某种方式存储这个新镜像是很有用的。Docker 镜像注册表正是用于这个目的。我们将在下一节讨论注册表。

# Docker 镜像注册表

**Docker 镜像注册表**用于托管 Docker 镜像。Docker 镜像通过标签组织，用户可以访问并下载这些标签。使用这些镜像可以在宿主机上创建并运行容器。镜像仓库可以托管在本地或远程服务器上，例如 Docker Hub，这是 Docker 提供的公共仓库。你还可以创建自己的私有镜像仓库，在组织内共享和分发镜像。

当你从 Docker 镜像仓库拉取镜像时，镜像由多个层组成。每一层代表 Dockerfile 中用于构建镜像的一条指令。这些层按顺序叠加在一起，创建最终的镜像。每一层是只读的，并具有唯一的 ID。

得益于 UnionFS，Docker 注册表在多个镜像和容器之间共享公共层，从而减少所需的磁盘空间。当容器修改文件时，它会在基础镜像上方创建一个新层，而不是修改基础镜像中的文件。这使得回滚到容器的先前状态变得容易，并且使镜像具有很高的可移植性。

根据你使用的云解决方案（例如 AWS 的 ECR 或 GCP 的 Google Container Registry）或 SaaS 解决方案（Docker Hub 是最受欢迎的 - [`hub.docker.com`](https://hub.docker.com)），你可以使用多个镜像仓库。也有一些开放源代码的授权解决方案可用：

+   **Harbor**：可以在[`goharbor.io/`](https://goharbor.io/)访问。

+   **Portus**：可以在[`port.us.org/`](http://port.us.org/)访问。

+   **Docker Registry**：可以在[`docs.docker.com/registry/`](https://docs.docker.com/registry/)访问。

+   **Project Quay**：可以在[`quay.io`](https://quay.io)访问，也可以在 GitHub 上访问：[`github.com/quay/quay`](https://github.com/quay/quay%20)

在本节中，我们已经了解了 Docker 镜像注册表，便于将 Docker 镜像存储在远程位置。在下一节中，我们将深入探讨 Docker 网络及其扩展。

# Docker 网络

Docker 网络有四种类型：none、bridge、host 和 overlay。

*Bridge*是 Docker 中的默认网络模式。处于同一桥接网络中的容器可以相互通信。简而言之，它创建了一个虚拟网络，在该网络中，容器被分配了 IP 地址，并可以通过这些地址进行通信，而网络之外的任何事物都无法访问这些地址。在*Host*网络中，容器使用主机的网络栈。这意味着容器共享主机的 IP 地址和网络接口。

*Overlay*模式允许你创建一个跨越多个 Docker 主机的虚拟网络。不同主机上的容器可以相互通信，就像它们在同一主机上一样。它在运行 Docker Swarm 时非常有用。

使用 Docker 命令行，你可以创建任何这些类型的自定义网络。

## None 网络

**None 网络**是 Docker 中的一种特殊网络模式，它禁用容器的所有网络功能。当容器在*none*网络模式下运行时，它无法访问任何网络资源，也不能与其他容器或主机进行通信。

要在*none*网络模式下运行容器，你可以在运行`docker run`命令时使用`--network none`选项。

例如，要在*none*网络模式下启动一个运行`nginx`镜像的容器，你可以运行以下命令：

```
admin@myhome:~$ docker run --network none -d ubuntu:20
```

*None*网络适用于运行不需要任何网络连接的工作负载，例如在连接的卷中处理数据。

## 桥接模式

在创建容器时使用**bridge 模式**时，还会创建一个虚拟接口，并将其附加到虚拟网络。每个容器将被分配一个唯一的 IP 地址，允许它与其他容器和主机进行通信。

主机机器充当容器的网关，在容器与外部网络之间路由流量。当一个容器想要与另一个容器或主机机器通信时，它将数据包发送到虚拟网络接口。虚拟网络接口然后将数据包路由到正确的目的地。

默认情况下，它是一个`172.17.0.0/16`网络，并且与您机器中的桥接设备`docker0`连接。在这个网络中，容器与主机机器之间的所有流量都是允许的。

如果在执行 `docker` `run` 命令时未使用 `--network` 选项选择网络，则所有容器都将附加到默认的桥接网络。

您可以使用以下命令列出所有可用的网络：

```
admin@myhome:~$ docker network ls
NETWORK ID     NAME                   DRIVER    SCOPE
6c898bde2c0c   bridge                 bridge    local
926b731b94c9   host                   host      local
b9f266305e10   none                   null      local
```

要获取更多关于网络的信息，您可以使用以下命令：

```
admin@myhome:~$  docker inspect bridge
[
    {
        "Name": "bridge",
        "Id": "6c898bde2c0c660cd96c3017286635c943adcb152c415543373469afa0aff13a",
        "Created": "2023-01-26T16:51:30.720499274Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.17.0.0/16",
                    "Gateway": "172.17.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
```

让我们进入 HOST 模式。

## 主机模式

在 **主机网络模式** 下，容器共享主机的网络栈和网络接口。这意味着容器使用您的机器的 IP 地址和网络设置，并且可以直接访问与其运行的机器相同的网络资源，包括其他容器。

运行在主机网络模式下的容器也可以直接监听主机机器的端口（绑定到它）。

主机网络模式的主要优点之一是提供更好的性能，因为容器不需要经过额外的网络栈。

与其他网络模式相比，这种模式的安全性较低，因为容器可以直接访问主机的网络资源，并能够监听主机接口上的连接。

## 覆盖网络

**覆盖网络**是由管理节点创建的，管理节点负责维护网络配置并管理工作节点的成员资格。管理节点创建一个虚拟网络交换机，并为网络中的每个容器分配 IP 地址。

每个工作节点运行 Docker 引擎和容器网络驱动程序，负责将该主机上的容器连接到虚拟网络交换机。容器网络驱动程序还确保数据包被正确封装并路由到正确的目的地。

当一个主机上的容器想要与另一个主机上的容器通信时，它将数据包发送到虚拟网络交换机。交换机然后将数据包路由到正确的主机，容器网络驱动程序在目标主机上解封装数据包并将其传送到目标容器。

覆盖网络使用 **虚拟扩展局域网**（**VXLAN**）协议来封装 IP 数据包，并使得在多个主机之间创建二层网络成为可能。

# 总结

在本章中，我们介绍了现代 DevOps 引领的基础设施的主要组成部分之一，那就是容器。我们描述了最突出的容器技术——Docker。我们还介绍了运行 Docker 容器和构建自定义容器的基础知识。在下一章中，我们将基于这些知识，介绍更高级的 Docker 主题。
