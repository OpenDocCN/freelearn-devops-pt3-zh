

# 深入了解 Docker

Docker 的出现彻底改变了我们运行、部署和维护应用程序的方式。随着容器化的兴起，我们能够抽象出应用程序所依赖的许多底层基础设施和依赖项，使得跨不同环境部署和管理应用程序变得比以往更容易。然而，强大的能力也伴随着巨大的责任，我们必须理解 Docker 的内部机制，并建立良好的实践，以确保我们的应用程序是安全、可靠和高效的。

在本章中，我们将深入探讨 Docker 的细节，探索它的架构、组件和关键特性。我们还将研究一些在 Docker 之上出现的辅助项目，如 Docker Compose 和 Kubernetes，并学习如何使用它们来构建更复杂和可扩展的应用程序。在整个过程中，我们将强调与 Docker 一起工作的最佳实践，如创建高效的 Docker 镜像、管理容器和优化性能。到本章结束时，你将能够自信地在 Docker 中运行应用程序，并利用其全部功能构建强大和可扩展的系统。

本章包含以下主题：

+   Docker 的高级用例

+   Docker Compose

+   高级 Dockerfile 技术

+   Docker 编排

# Docker 的高级用例

在使用 Docker 及其 CLI 时，我们需要处理许多关于容器生命周期、构建过程、数据卷和网络的事项。你可以通过使用其他工具来自动化其中的一些任务，但了解底层的工作原理仍然是很有用的。

## 运行公共镜像

你可以在 Docker Hub 上找到的许多公共镜像（[`hub.docker.com`](https://hub.docker.com)）都提供了初始化脚本，这些脚本从环境变量或挂载的文件中获取配置，写入到预定义的目录中。

最常用的镜像是使用了两种技术的数据库镜像。让我们来看看官方的**Docker PostgreSQL**镜像。我们将使用的镜像可以在这里找到：[`hub.docker.com/_/postgres`](https://hub.docker.com/_/postgres)。

要运行官方的 PostgreSQL Docker 镜像，你可以使用以下命令：

```
admin@myhome:~$ docker run --name some-postgres -e POSTGRES_PASSWORD=mysecretpassword -d postgres
```

在这个命令中，我们有以下内容：

+   `--name some-postgres`给容器指定了一个名称*some-postgres*

+   `-e POSTGRES_PASSWORD=mysecretpassword`为默认的 PostgreSQL 用户(*postgres*)设置密码

+   `-d`在后台运行容器；`postgres`指定使用的镜像

还可以通过添加`POSTGRES_USER`环境变量来覆盖默认用户(*postgres*)。其他配置环境变量可以在文档中找到。

当你使用官方 PostgreSQL 镜像时，一个非常有用的功能是通过 SQL 脚本进行数据库预填充。为此，你需要将一个本地目录与脚本绑定挂载到容器内的 `/docker-entrypoint-initdb.d`。有两件事需要注意：空数据目录和确保所有脚本成功完成。空数据目录是必要的，因为它将充当你可以加载 SQL 或 shell 脚本的入口点；它还可以防止数据丢失。如果任何脚本出现错误，数据库将无法启动。

对于在 Docker Hub 中运行的其他数据库，类似的功能也可以使用。

你可以使用的另一个有用的官方镜像是**nginx**：它可能更简单，因为你已经有一个配置好的 web 服务器，并且你需要提供内容（HTML 文件、JavaScript 或 CSS）供其提供，或者覆盖默认配置。

下面是将一个静态 HTML 网站挂载到容器的示例：

```
admin@myhome:~$ docker run -p 8080:80 -v /your/webpage:/usr/share/nginx/html:ro -d nginx
```

在这个命令中，我们有以下内容：

+   `-p 8080:80`：这个选项将主机上的 `8080` 端口映射到容器内的 `80` 端口。这意味着当有人访问主机上的 `8080` 端口时，它将被重定向到容器内的 `80` 端口。

+   `-v /your/webpage:/usr/share/nginx/html:ro`：这个选项将主机上的 `/your/webpage` 目录挂载到容器内的 `/usr/share/nginx/html` 目录。`ro` 选项意味着挂载是只读的，这意味着容器不能修改 `/your/webpage` 目录中的文件。

+   `-d`：这个选项告诉 Docker 以分离模式运行容器，这意味着它将在后台运行。

+   `nginx`：这是将用于运行容器的 Docker 镜像的名称。在本例中，它是来自 Docker Hub 的官方 nginx 镜像。

我们可以像这样覆盖默认的 nginx 配置：

```
admin@myhome:~$ docker run -p 8080:80 -v ./config/nginx.conf:/etc/nginx/nginx.conf:ro -d nginx
```

在这个命令中，大部分之前的选项重复出现，只有一个例外：`-v ./config/nginx.conf:/etc/nginx/nginx.conf:ro`。这个选项将主机上的 `./config/nginx.conf` 文件挂载到容器内的 `/etc/nginx/nginx.conf` 文件。`ro` 选项表示挂载是只读的，意味着容器不能修改 `nginx.conf` 文件。

## 运行调试容器

在生产环境中运行的容器通常只有很少的工具对故障排除有帮助。更重要的是，这些容器不是以 root 用户身份运行，并且有多个安全机制来防止篡改。考虑到这一点，如果出现问题，我们该如何进入 Docker 网络进行调试呢？

这个问题的答案就是运行另一个我们可以进入的容器。这个容器会预安装一些工具，或者允许我们在运行时安装所需的工具。我们可以使用多种技术来实现这一点。

首先，我们需要一个会无限运行的进程，直到我们手动停止它。这个进程运行时，我们可以进入并使用 `docker exec` 命令进入正在运行的 Docker 容器中。

了解 Bash 脚本后，执行这个过程最简单的方式是创建一个 `while` 循环：

```
admin@myhome:~$ docker run -d ubuntu while true; do sleep 1; done
```

另一种方法是使用 `sleep` 程序：

```
admin@myhome:~$ docker run -d ubuntu sleep infinity
```

或者，你可以尝试*读取*一个特殊设备，`/dev/null`，它什么都不输出，以及使用`tail`命令：

```
admin@myhome:~$ docker run -d ubuntu tail -f /dev/null
```

最后，当这些命令之一在你正在尝试排查故障的网络中运行时，你可以在其中运行一个命令，并有效地能够从你需要调查的环境中运行命令：

```
admin@myhome:~$ docker exec -it container_name /bin/bash
```

现在让我们来看看如何清理未使用的容器。

## 清理未使用的容器

随着时间的推移，Docker 镜像可能会积累，尤其是当你频繁构建和实验容器时。这些镜像中的一些可能不再需要，并且它们可能占用宝贵的磁盘空间。要清理这些未使用的镜像，你可以使用 `docker image prune` 命令。该命令会删除所有未与容器关联的镜像，也就是所谓的**悬挂镜像**。

除了未使用的镜像外，可能还有一些未正确移除的容器。这些容器可以通过 `docker ps -a` 命令来识别。要删除特定的容器，你可以使用 `docker rm <container_id>` 命令，其中 `<container_id>` 是你想要删除的容器的标识符。如果你想删除所有已停止的容器，你可以使用 `docker container prune` 命令。

定期执行镜像和容器清理是一个良好的实践，有助于保持健康的 Docker 环境。这不仅可以节省磁盘空间，还可以防止与未使用资源相关的潜在安全漏洞。最好还要从容器和镜像中移除所有敏感信息，如密码和密钥。

这是一个使用 `docker image prune` 命令来删除悬挂镜像的示例：

```
admin@myhome:~$ docker image prune
Deleted Images:
deleted:
sha256:5f70bf18a086007016e948b04aed3b82103a36bea41755b6cddfaf10ace3c6ef
deleted:sha256:c937c4dd0c2eaf57972d4f80f55058b3685f87420a9a9fb9ef0dfe3c7c3e60bc
Total reclaimed space: 65.03MB
```

这是一个使用 `docker container prune` 命令删除所有已停止容器的示例：

```
admin@myhome:~$ docker container prune
WARNING! This will remove all stopped containers.
Are you sure you want to continue? [y/N] y
Deleted Containers:
8c922b2d9708fcee6959af04c8f29d2d6850b3a3b3f3b966b0c360f6f30ed6c8
6d90b6cbc47dd99b2f78a086e959a1d18b7e0cf4778b823d6b0c6b0f6b64b6c64
Total reclaimed space: 0B
```

为了自动化这些任务，你可以使用 `crontab` 来安排定期清理。要编辑你的 `crontab` 文件，你可以使用 `crontab -e` 命令。以下是一个例子，展示如何在每天凌晨 3 点安排定期清理悬挂镜像：

```
0 3 * * * docker image prune -f
```

这一行由五个字段组成，字段之间由空格分隔。这些字段表示命令执行时的分钟、小时、月日、月份和星期几。在这个例子中，命令将在每天的凌晨 3 点执行。让我们详细看看每个元素：

+   第一个字段，`0`，表示分钟。在这种情况下，我们希望命令在整点的 0 分钟执行。

+   第二个字段，`3`，表示小时。在这个例子中，我们希望命令在凌晨 3 点执行。

+   第三个字段，`*`，表示每月的天数。星号表示“任何”一天。

+   第四个字段，`*`，表示月份。星号意味着“任何”一个月份。

+   第五个字段，`*`，表示星期几。星号意味着“任何”一天。`1`表示星期一，`2`表示星期二，以此类推，直到`7`表示星期天。

下面是一个示例，演示如何安排在每周日凌晨 4 点清理停止的容器：

```
0 4 * * 7 docker container prune -f
```

`-f`标志用于强制删除镜像或容器，而不需要用户确认。

要列出当前用户的所有 cron 作业，你可以使用`crontab -l`命令。关于`crontab`的更多信息可以在线查找或使用`man crontab`命令。关于如何使用它的一篇优秀教程文章可以在 Ubuntu Linux 知识库中找到：[`help.ubuntu.com/community/CronHowto`](https://help.ubuntu.com/community/CronHowto)。

## Docker 卷和绑定挂载

如前一章所述，Docker 卷和绑定挂载是两种不同的持久化 Docker 容器生成的数据的方法。卷由 Docker 管理，并存在于容器的文件系统之外。它们可以在容器之间共享和重用，即使原始容器被删除，它们也会持久存在。而绑定挂载则将主机系统上的文件或目录链接到容器中的文件或目录。绑定挂载中的数据可以从主机和容器直接访问，并且只要主机文件或目录存在，它就会持续存在。

要使用 Docker 卷，你可以在运行`docker run`命令时使用`-v`或`--mount`标志，并指定主机源和容器目标。例如，要创建一个卷并将其挂载到容器的`/app/data`，你可以运行以下命令：

```
admin@myhome:~$ docker run -v my_data:/app/data <image_name>
```

要使用绑定挂载，你可以使用相同的标志并指定主机源和容器目标，就像使用卷一样。然而，与你使用卷名称不同的是，你需要使用主机文件或目录的路径。例如，要将主机目录`/host/data`绑定挂载到容器的`/app/data`，你可以运行以下命令：

```
admin@myhome:~$ docker run -v /host/data:/app/data <image_name>
```

在使用 Docker 的绑定挂载时，你可能会遇到与挂载中文件和目录的权限问题。这是因为主机和容器的**用户 ID**（**UIDs**）和**组 ID**（**GIDs**）可能不匹配，从而导致无法访问或修改绑定挂载中的数据。

例如，如果主机文件或目录的所有者是 UID 为 1000 的用户，而容器中的 UID 不同，那么容器可能无法访问或修改绑定挂载中的数据。同样，如果组 ID 不匹配，由于组权限，容器也可能无法访问或修改数据。

为了避免这些权限问题，你可以在运行`docker run`命令时指定主机文件或目录的 UID 和 GID。例如，要使用 UID 和 GID 为 1000 的绑定挂载运行容器，你可以运行以下命令：

```
admin@myhome:~$ docker run -v /local/data:/app/data:ro,Z --user 1000:1000 <image_name>
```

在此示例中，`:ro` 标志指定绑定挂载应为只读，`,Z` 标志告诉 Docker 使用私有标签标记绑定挂载，以防它与其他容器交互。`--user` 标志将容器内运行的进程的 UID 和 GID 设置为 `1000`。

通过在容器中指定主机文件或目录的 UID 和 GID，可以避免 Docker 中绑定挂载的权限问题，并确保容器可以按预期访问和修改数据。

## Docker 网络高级用法

Docker 提供了一种便捷的方式来管理用户定义网络中容器的网络。通过使用 Docker 网络，您可以轻松控制容器之间的通信，并将它们与主机网络隔离开来。

Docker 桥接网络是一种默认的网络配置，可以使运行在同一主机上的容器之间进行通信。它通过在主机系统上创建一个虚拟网络接口来实现容器与主机网络之间的桥接。桥接网络上的每个容器都被分配一个唯一的 IP 地址，允许它与其他容器和主机进行通信。

桥接网络彼此隔离，这意味着连接到不同桥接网络的容器无法直接通信。要在不同网络的容器之间实现通信，可以使用 Docker 服务发现机制，例如连接到特定容器 IP 地址或使用负载均衡器。

在实际中使用桥接网络，您可以使用 Docker CLI 创建一个新的桥接网络。例如，您可以使用命令 `docker network create --driver bridge production-network` 来创建一个名为 *production-network* 的新桥接网络。网络创建后，您可以使用 `docker run` 命令中的 `--network` 选项将容器连接到网络。您可以使用命令 `docker run --network production-network my-image` 在 *production-network* 网络上运行来自 *my-image* 镜像的容器。

除了创建新的桥接网络外，还可以将容器连接到安装 Docker 时自动创建的默认 *bridge* 网络。要将容器连接到默认网络，您无需在 `docker run` 命令中指定 `--network` 选项。容器将自动连接到默认 *bridge* 网络，并从桥接网络子网中分配 IP 地址。

现在，如果创建多个网络，默认情况下它们将被隔离，不允许它们之间进行通信。要允许两个桥接网络之间的通信，例如 *production-network* 和 *shared-network*，您需要通过连接您选择的容器到这两个网络或允许这两个网络之间的所有通信来创建网络连接。如果可能的话，后者选项不受支持。

最后一个选项是使用**Docker Swarm**模式和覆盖网络模式，我们将在本章的*Docker* *编排*部分稍后讨论。

以下是将容器同时连接到两个网络的示例。首先，让我们创建一个生产网络和一个共享网络：

```
admin@myhome:~$ docker network create production-network
fd1144b9a9fb8cc2d9f65c913cef343ebd20a6b30c4b3ba94fdb1fb50aa1333c
admin@myhome:~$ docker network create shared-network
a544f0d39196b95d772e591b9071be38bafbfe49c0fdf54282c55e0a6ebe05ad
```

现在，我们可以启动一个连接到`production-network`的容器：

```
admin@myhome:~$ docker run -itd --name prod-container --network production-network alpine sh
Unable to find image 'alpine:latest' locally
latest: Pulling from library/alpine
8921db27df28: Pull complete
Digest:sha256:f271e74b17ced29b915d351685fd4644785c6d1559dd1f2d4189a5e851ef753a
Status: Downloaded newer image for alpine:latest
287be9c1c76bd8aa058aded284124f666d7ee76c73204f9c73136aa0329d6bb8
```

我们也对`shared-network`做同样的操作：

```
admin@myhome:~$ docker run -itd --name shared-container --network shared-network alpine sh
38974225686ebe9c0049147801e5bc777e552541a9f7b2eb2e681d5da9b8060b
```

让我们检查一下两个容器是否都在运行：

```
admin@myhome:~$ docker ps
CONTAINER ID    IMAGE    COMMAND   CREATED     STATUS    PORTS   NAMES
38974225686e   alpine    "sh"      4 seconds ago    Up 3 seconds              shared-container
287be9c1c76b   alpine    "sh"      16 seconds ago   Up 14 seconds             prod-container
```

最后，我们还需要将`prod-container`连接到共享网络：

```
admin@myhome:~$ docker network connect shared-network prod-container
```

然后，我们可以进入`prod-container`并 ping `shared-container`：

```
admin@myhome:~$ docker exec -it prod-container /bin/sh
/ # ping shared-container
PING shared-container (172.24.0.2): 56 data bytes
64 bytes from 172.24.0.2: seq=0 ttl=64 time=0.267 ms
64 bytes from 172.24.0.2: seq=1 ttl=64 time=0.171 ms
^C
--- shared-container ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.171/0.219/0.267 ms
/ # ping prod-container
PING prod-container (172.23.0.2): 56 data bytes
64 bytes from 172.23.0.2: seq=0 ttl=64 time=0.167 ms
64 bytes from 172.23.0.2: seq=1 ttl=64 time=0.410 ms
64 bytes from 172.23.0.2: seq=2 ttl=64 time=0.108 ms
^C
5 packets transmitted, 5 packets received, 0% packet loss
round-trip min/avg/max = 0.108/0.188/0.410 ms
```

你可以在 Docker 官方网页上了解更多关于网络的内容：[`docs.docker.com/network/`](https://docs.docker.com/network/)。

## Docker 的安全特性

Docker 的核心并非旨在作为安全工具。这是在后期通过 Linux 内核特性的支持逐步构建的，且仍在开发中，更多的安全特性也在不断加入。

这个话题涉及很多内容，但我们将重点介绍四个最常用的安全特性：

+   命名空间

+   **安全计算** **模式**（**seccomp**）

+   无根模式

+   Docker 签名镜像

### Linux 内核命名空间

**内核命名空间**是 Docker 安全的重要组成部分，因为它们为容器和主机系统之间提供隔离。它们允许每个容器查看系统资源，如文件系统、网络和进程表，而不会影响主机或其他容器。这意味着运行在容器内的进程不能访问或修改主机文件系统、网络配置或进程，从而帮助保护主机系统免受恶意或不受控制的容器的威胁。

Docker 使用多个 Linux 内核命名空间为容器提供隔离环境。这些命名空间用于为进程、网络、挂载点等创建隔离的环境。

Docker 守护进程的`USER`命名空间将确保容器内的 root 用户与主机环境中的 root 用户处于不同的上下文。这是为了确保容器内的 root 用户与主机上的 root 用户不相同。

`PID`命名空间隔离容器之间的进程 ID。每个容器只能看到自己的进程集，且与其他容器和主机隔离。

`NET`命名空间的功能是隔离每个容器的网络堆栈，使得每个容器都有自己的虚拟网络堆栈，包括网络设备、IP 地址和路由。

`IPC`命名空间处理容器之间的**进程间通信**（**IPC**）资源。每个容器都有自己独立的 IPC 资源，如 System V IPC 对象、信号量和消息队列。

`UTS`命名空间用于容器的主机名和域名隔离。在这里，每个容器都有自己的主机名和域名，这些不会影响其他容器或主机。

最后，`MNT`命名空间隔离了每个容器的挂载点。这意味着每个容器都有一个私有的文件系统层级，拥有自己的根文件系统、挂载的文件系统和绑定挂载。

通过使用这些命名空间，Docker 容器彼此之间以及与主机之间都被隔离，这有助于确保容器和主机系统的安全性。

`USER`命名空间是最难使用的，因为它需要特别的 UID 和 GID 映射配置。默认情况下并未启用它，因为与主机共享`PID`或`NET`命名空间（`–pid=host`或`–network=host`）是不可能的。此外，使用`docker run`时如果没有指定`–userns=host`（从而禁用`USER`命名空间的隔离），将无法使用`–privileged mode`标志。之前列出的其他命名空间大多无需任何特别配置即可生效。

### Seccomp

**Seccomp**，即**安全计算模式**，是一个 Linux 内核功能，允许进程指定其允许进行的系统调用。这使得可以限制容器能够执行的系统调用类型，从而通过减少容器逃逸或特权提升的风险来提高主机系统的安全性。

当进程指定其 seccomp 配置文件时，Linux 内核会过滤传入的系统调用，并仅允许在配置文件中指定的那些调用。这意味着，即使攻击者获得了容器的访问权限，他们也会受到能执行的操作类型的限制，从而减少攻击的影响。

要为容器创建 seccomp 配置文件，可以在`docker run`命令中使用`seccomp 配置`选项。这允许你在启动容器时指定要使用的 seccomp 配置文件。

创建 seccomp 配置文件有两种主要方式：使用预定义的配置文件或创建自定义配置文件。预定义配置文件适用于常见的使用场景，可以在`docker run`命令中轻松指定。例如，默认配置文件允许所有系统调用，而限制配置文件仅允许一组被认为对大多数使用场景安全的系统调用。

要创建自定义的 seccomp 配置文件，可以使用**Podman**（[`podman.io/blogs/2019/10/15/generate-seccomp-profiles.xhtml`](https://podman.io/blogs/2019/10/15/generate-seccomp-profiles.xhtml)）或**seccomp-gen**（[`github.com/blacktop/seccomp-gen`](https://github.com/blacktop/seccomp-gen)）工具。这两个工具自动识别容器在生产环境中使用时会进行的系统调用，并生成一个 JSON 文件，可用作 seccomp 配置文件。

Seccomp 并不能保证安全。了解你的应用所需的系统调用并确保它们在 seccomp 配置文件中被允许，是非常重要的。

以下是一个 seccomp 配置文件示例，它允许运行 Web 服务器应用的容器使用有限的系统调用：

```
{
    "defaultAction": "SCMP_ACT_ALLOW",
    "syscalls": [
        {
            "name": "accept",
            "action": "SCMP_ACT_ALLOW"
        },
        {
            "name": "bind",
            "action": "SCMP_ACT_ALLOW"
        },
        {
            "name": "connect",
            "action": "SCMP_ACT_ALLOW"
        },
        {
            "name": "listen",
            "action": "SCMP_ACT_ALLOW"
        },
        {
            "name": "sendto",
            "action": "SCMP_ACT_ALLOW"
        },
        {
            "name": "recvfrom",
            "action": "SCMP_ACT_ALLOW"
        },
        {
            "name": "read",
            "action": "SCMP_ACT_ALLOW"
        },
        {
            "name": "write",
            "action": "SCMP_ACT_ALLOW"
        }
    ]
}
```

在这个示例中，`defaultAction`被设置为`SCMP_ACT_ALLOW`，意味着所有未在`syscalls`数组中特别列出的系统调用都会被允许。为了阻止所有未定义的调用，你可以将默认操作设置为`SCMP_ACT_ERRNO`。所有可用的操作都在`seccomp_rule_add`过滤器规格的在线手册中有所描述：[`man7.org/linux/man-pages/man3/seccomp_rule_add.3.xhtml`](https://man7.org/linux/man-pages/man3/seccomp_rule_add.3.xhtml)。

`syscalls`数组列出了应该允许容器使用的系统调用，并为每个调用指定了要采取的行动（在这种情况下，所有调用都被允许）。该配置文件仅允许 Web 服务器运行所需的系统调用，阻止所有其他系统调用，从而提高容器的安全性。

有关系统调用的更多信息，请参阅：[`docs.docker.com/engine/security/seccomp/`](https://docs.docker.com/engine/security/seccomp/)。

### 无根模式

**Docker 无根**模式是一个功能，允许用户在不以 root 用户身份运行 Docker 守护进程的情况下运行 Docker 容器。该模式通过减少主机系统的攻击面并最小化特权升级的风险，提供了额外的安全层。

让我们在 Ubuntu Linux 或 Debian Linux 上设置一个无根 Docker 守护进程。首先，确保你已从官方 Docker 软件包仓库安装了 Docker，而不是从 Ubuntu/Debian 软件包中安装：

```
admin@myhome:~$ sudo apt-get install -y -qq apt-transport-https ca-certificates curl
admin@myhome:~$ sudo mkdir -p /etc/apt/keyrings && sudo chmod -R 0755 /etc/apt/keyrings
admin@myhome:~$ curl -fsSL "https://download.docker.com/linux/ubuntu/gpg" | sudo gpg --dearmor --yes -o /etc/apt/keyrings/docker.gpg
admin@myhome:~$ sudo chmod a+r /etc/apt/keyrings/docker.gpg
admin@myhome:~$ echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu jammy stable" | sudo tee /etc/apt/sources.list.d/docker.list
admin@myhome:~$ sudo apt-get update
admin@myhome:~$ sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-scan-plugin docker-compose-plugin docker-ce-rootless-extras docker-buildx-plugin
```

`docker-ce-rootless-extras`将在你的`/usr/bin`目录中安装一个名为`dockerd-rootless-setuptool.sh`的脚本，它将自动化整个过程：

```
admin@myhome~$ dockerd-rootless-setuptool.sh  --help
Usage: /usr/bin/dockerd-rootless-setuptool.sh [OPTIONS] COMMAND
A setup tool for Rootless Docker (dockerd-rootless.sh).
Documentation: https://docs.docker.com/go/rootless/
Options:
  -f, --force                Ignore rootful Docker (/var/run/docker.sock)
      --skip-iptables        Ignore missing iptables
Commands:
  check        Check prerequisites
  install      Install systemd unit (if systemd is available) and show how to manage the service
  uninstall    Uninstall systemd unit
```

要运行此脚本，我们需要一个具有配置环境的非 root 用户，以便能够运行 Docker 守护进程。让我们首先创建一个`dockeruser`用户：

```
admin@myhome~$ sudo adduser dockeruser
Adding user `dockeruser' ...
Adding new group `dockeruser' (1001) ...
Adding new user `dockeruser' (1001) with group `dockeruser' ...
Creating home directory `/home/dockeruser' ...
Copying files from `/etc/skel' ...
New password:
Retype new password:
passwd: password updated successfully
Changing the user information for dockeruser
Enter the new value, or press ENTER for the default
     Full Name []:
     Room Number []:
     Work Phone []:
     Home Phone []:
     Other []:
Is the information correct? [Y/n] y
```

在继续之前，我们还需要创建一个 UID 映射配置。为此，我们需要安装`uidmap`软件包，并创建`/etc/subuid`和`/etc/subgid`配置文件：

```
admin@myhome~$ sudo apt install -y uidmap
admin@myhome~$ echo "dockeruser:100000:65536" | sudo tee /etc/subuid
admin@myhome~$ echo "dockeruser:100000:65536" | sudo tee /etc/subgid
```

以`dockeruser`身份登录并运行`dockerd-rootless-setuptool.sh`脚本：

```
admin@myhome~$ sudo -i -u dockeruser
```

确保设置了`environment XDG_RUNTIME_DIR`，并且 systemd 可以从`dockeruser`读取环境变量：

```
$ export XDG_RUNTIME_DIR=/run/user/$UID
$ echo 'export XDG_RUNTIME_DIR=/run/user/$UID' >> ~/.bashrc
$ systemctl --user show-environment
HOME=/home/dockeruser
LANG=en_US.UTF-8
LOGNAME=dockeruser
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin:/snap/bin
SHELL=/bin/bash
SYSTEMD_EXEC_PID=720
USER=dockeruser
XDG_RUNTIME_DIR=/run/user/1001
XDG_DATA_DIRS=/usr/local/share/:/usr/share/:/var/lib/snapd/desktop
DBUS_SESSION_BUS_ADDRESS=unix:path=/run/user/1001/bus
```

现在，你可以使用`dockerd-rootless-setuptool.sh`脚本安装无根（rootless）Docker（部分输出已被截断以提高可读性）：

```
$ dockerd-rootless-setuptool.sh install
[INFO] Creating  [condensed for brevity]
     Active: active (running) since Fri 2023-02-17 14:19:04 UTC; 3s ago
+ DOCKER_HOST=unix:///run/user/1001/docker.sock /usr/bin/docker version
Client: Docker Engine - Community
 Version:           23.0.1
[condensed for brevity]
Server: Docker Engine - Community
 Engine:
  Version:          23.0.1
 [condensed for brevity]
 rootlesskit:
  Version:          1.1.0
 [condensed for brevity]
+ systemctl --user enable docker.service
Created symlink /home/dockeruser/.config/systemd/user/default.target.wants/docker.service → /home/dockeruser/.config/systemd/user/docker.service.
[INFO] Installed docker.service successfully.
```

现在，让我们验证一下能否使用 Docker 无根守护进程：

```
dockeruser@vagrant:~$ export DOCKER_HOST=unix:///run/user/1001/docker.sock
dockeruser@vagrant:~$ docker ps
CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

此时，我们已经有一个以*dockeruser*系统用户身份运行的 Docker 守护进程，而不是 root 用户。我们将能够像在标准配置中一样运行所有需要的服务。某些情况，如 Docker 中的 Docker 配置，需要进一步的配置。

有关无根模式的更多详细信息，请访问[`docs.docker.com/engine/security/rootless/`](https://docs.docker.com/engine/security/rootless/)。

### Docker 签名镜像

Docker 签名镜像是一种安全措施，它确保用户知道 Docker 镜像来自受信任的源，并且没有被篡改。Docker 使用数字签名对镜像进行签名，Docker 引擎可以验证签名，以确保镜像与发布者签名时完全一致。

Docker 签名镜像可以通过检查签名者的公钥来验证，公钥可以从受信任的注册中心（如 Docker Hub）获取。如果镜像有效，Docker 会允许你在本地拉取并运行该镜像。

签名 Docker 镜像的第一步是生成一个签名密钥。可以使用 `docker trust key` `generate` 命令：

```
admin@myhome:~/$ docker trust key generate devops
Generating key for devops...
Enter passphrase for new devops key with ID 6b6b768:
Repeat passphrase for new devops key with ID 6b6b768:
Successfully generated and loaded private key. Corresponding public key available: /home/admin/devops.pub
```

记得使用强密码来保护密钥免受访问。私钥将保存在你的主目录中——例如，`~/.docker/trust/private`。文件的名称会被哈希化。

一旦你生成了签名密钥，下一步就是初始化镜像的信任元数据。信任元数据包含关于镜像的信息，包括授权签名镜像的密钥列表。要初始化信任元数据，可以使用 `docker trust signer add` 命令。请注意，你需要登录到所使用的 Docker 注册中心（通过 `docker` `login` 命令）：

```
admin@myhome:~/$ docker trust signer add --key devops.pub private-registry.mycompany.tld/registries/pythonapps
Adding signer "devops" to private-registry.mycompany.tld/registries/pythonapps/my-python-app...
Initializing signed repository for private-registry.mycompany.tld/registries/pythonapps/my-python-app...
You are about to create a new root signing key passphrase. This passphrase
will be used to protect the most sensitive key in your signing system. Please
choose a long, complex passphrase and be careful to keep the password and the
key file itself secure and backed up. It is highly recommended that you use a
password manager to generate the passphrase and keep it safe. There will be no
way to recover this key. You can find the key in your config directory.
Enter passphrase for new root key with ID a23d653:
Repeat passphrase for new root key with ID a23d653:
Enter passphrase for new repository key with ID de78215:
Repeat passphrase for new repository key with ID de78215:
Successfully initialized "private-registry.mycompany.tld/registries/pythonapps/my-python-app"
Successfully added signer: devops to private-registry.mycompany.tld/registries/pythonapps/my-python-app
```

你可以在成功构建 Docker 镜像并用注册中心名称进行标签标记后，使用 `docker trust sign` 命令对镜像进行签名。此命令会使用信任元数据中的授权密钥对镜像进行签名，并将该信息连同你的 Docker 镜像一起推送到注册中心：

```
admin@myhome:~/$ docker trust sign private-registry.mycompany.tld/registries/pythonapps/my-python-app:2.9BETA
Signing and pushing trust data for local image private-registry.mycompany.tld/registries/pythonapps/my-python-app:2.9BETA, may overwrite remote trust data
The push refers to repository [private-registry.mycompany.tld/registries/pythonapps]
c5ff2d88f679: Mounted from library/ubuntu
latest: digest:sha256:41c1003bfccce22a81a49062ddb088ea6478eabea1457430e6235828298593e6 size: 529
Signing and pushing trust metadata
Enter passphrase for devops key with ID 6b6b768:
Successfully signed private-registry.mycompany.tld/registries/pythonapps/my-python-app:2.9BETA
```

要验证你的 Docker 镜像是否已经签名，并且使用了哪个密钥，你可以使用 `docker trust` `inspect` 命令：

```
admin@myhome:~/$ docker trust inspect --pretty private-registry.mycompany.tld/registries/pythonapps/my-python-app:2.9BETA
Signatures for private-registry.mycompany.tld/registries/pythonapps/my-python-app:2.9BETA
SIGNED TAG    DIGEST                  SIGNERS
latest       41c1003bfccce22a81a49062ddb088ea6478eabea1457430e6235828298593e6   devops
List of signers and their keys for private-registry.mycompany.tld/registries/pythonapps/my-python-app:2.9BETA
SIGNER    KEYS
devops    6b6b7688a444
Administrative keys for private-registry.mycompany.tld/registries/pythonapps/my-python-app:2.9BETA
  Repository Key: de782153295086a2f13e432db342c879d8d8d9fdd55a77f685b79075a44a5c37
  Root Key: c6d5d339c75b77121439a97e893bc68a804368a48d4fd167d4d9ba0114a7336b
```

将 `DOCKER_CONTENT_TRUST` 环境变量设置为 `1`。这样将防止 Docker 下载未签名和未验证的镜像到本地存储。

关于 DCT 的更详细信息可以在官方站点找到：[`docs.docker.com/engine/security/trust/`](https://docs.docker.com/engine/security/trust/)。

## Docker 用于 CI/CD 管道集成

**持续集成**（**CI**）和 **持续部署**（**CD**）是流行的软件开发实践，旨在确保软件开发过程高效流畅，并保持代码质量。

CI 指的是在共享代码库中自动构建和测试代码更改的实践。CD 是 CI 之后的下一步，其中代码更改会自动部署到生产或暂存环境中。

Docker 是在 CI/CD 管道中广泛使用的工具，它提供了一种高效的方式来打包和分发应用程序。在这一小节中，我们将展示如何使用 GitHub Actions 构建并将 Docker 镜像推送到 AWS **Elastic Container Registry**（**ECR**）。

让我们来看一个示例，展示如何设置 GitHub Action 来构建并将 Docker 镜像推送到 AWS ECR。

通过在你的仓库的 `.github/workflows` 目录中创建一个名为 `main.yml` 的新文件来创建一个新的 GitHub Actions 工作流。添加并推送到主分支后，它将在每次该分支有新的推送时可用并被触发。

在`main.yml`文件中，定义工作流的步骤，如下所示：

```
name: Build and Push Docker Image
on:
  push:
    branches:
      - main
env:
  AWS_REGION: eu-central-1
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout code
      uses: actions/checkout@v2
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v1
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}
    - name: Build Docker image
      uses: docker/build-push-action@v2
      with:
        push: true
        tags: ${{ env.AWS_REGION }}/my-image:${{ env.GITHUB_SHA }}
    - name: Push Docker image to AWS ECR
      uses: aws-actions/amazon-ecr-push-action@v1
      with:
        region: ${{ env.AWS_REGION }}
        registry-url: ${{ env.AWS_REGISTRY_URL }}
        tags: ${{ env.AWS_REGION }}/my-image:${{ env.GITHUB_SHA }}
```

用你的特定值替换 `AWS_REGION` 和 `AWS_REGISTRY_URL` 环境变量。你还应该将 `my-image` 替换为你 Docker 镜像的名称。

在你的 GitHub 仓库设置中，创建两个密钥，命名为 `AWS_ACCESS_KEY_ID` 和 `AWS_SECRET_ACCESS_KEY`，并使用具有推送权限的 AWS 凭证。或者，你可以使用你自己的运行器和附加到运行器的 AWS IAM 角色，或 GitHub OIDC，这将通过 AWS 账户进行身份验证。你可以在这里找到相关文档：[`docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services`](https://docs.github.com/en/actions/deployment/security-hardening-your-deployments/configuring-openid-connect-in-amazon-web-services)。

完成这些步骤后，每当你将代码更改推送到主分支时，你的 GitHub Action 将自动构建并推送 Docker 镜像到 AWS ECR。推送之后，你可以触发服务器端的另一个过程，评估并将新 Docker 镜像部署到你的某个环境中，而无需进一步的手动操作。这有助于简化你的 CI/CD 管道，并确保你的代码更改可以放心地部署到生产环境。

也可以以类似的方式将相同的管道集成到 GitLab 或其他 CI/CD 工具中。

在本节中，我们学习了一些容器的非常见使用场景，如无根模式、受保护计算模式、网络高级用例以及如何启动调试容器。在下一节中，我们将进一步集中讨论如何自动化设置 Docker 容器的过程，并探讨如何比手动逐个启动容器更好地进行编排。

# Docker Compose

**Docker Compose** 是一个控制台工具，用于通过一个命令运行多个容器。它提供了一种简便的方式来管理和协调多个容器，使构建、测试和部署复杂应用变得更加容易。使用 Docker Compose，你可以在一个 YAML 文件中定义应用程序的服务、网络和卷，然后通过命令行启动和停止所有服务。

要使用 Docker Compose，首先需要在 `docker-compose.yml` 文件中定义应用程序的服务。该文件应包括有关你希望运行的服务、它们的配置以及它们如何连接的信息。文件还应指定每个服务使用的 Docker 镜像。

`docker-compose.yaml` 文件是一个核心配置文件，用于 Docker Compose 管理应用程序的部署和运行。它采用 YAML 语法编写。

`docker-compose.yaml` 文件的结构分为几个部分，每个部分定义部署的不同方面。第一部分 `version` 指定了使用的 Docker Compose 文件格式的版本。第二部分 `services` 定义了组成应用程序的服务，包括它们的镜像名称、环境变量、端口和其他配置选项。

`services` 部分是 `docker-compose.yaml` 文件中最重要的部分，因为它定义了应用程序的构建、运行和连接方式。每个服务通过其自己的键值对集合来定义其配置选项。例如，`image` 键用于指定要用于服务的 Docker 镜像的名称，而 `ports` 键用于指定服务的端口映射。

`docker-compose.yaml` 文件还可以包含其他部分，比如 `volumes` 和 `networks`，这些部分允许您为应用程序定义共享数据存储和网络配置。总体来说，`docker-compose.yaml` 文件提供了一种集中式、声明式的方式来定义、配置和运行使用 Docker Compose 的多容器应用程序。借助其简单的语法和强大的功能，它是简化复杂应用程序开发和部署的关键工具。

环境变量是键值对，允许您在运行服务时传递配置信息。在 `docker-compose.yaml` 文件中，可以使用服务定义中的环境键来指定环境变量。

在 `docker-compose.yaml` 文件中指定环境变量的一种方法是在环境部分中简单地列出它们作为键值对。以下是一个例子：

```
version: '3'
services:
  db:
    image: mariadb
    environment:
      MYSQL_ROOT_PASSWORD: example
      MYSQL_DATABASE: example_db
```

除了直接在 `docker-compose.yaml` 文件中指定环境变量之外，还可以将它们存储在外部文件中，并在 `docker-compose.yaml` 文件中使用 `env_file` 键引用该文件。以下是一个例子：

```
version: '3'
services:
  db:
    image: mariadb
    env_file:
      - db.env
```

`db.env` 文件的内容可能如下所示：

```
MYSQL_ROOT_PASSWORD=example
MYSQL_DATABASE=example_db
```

通过使用外部的 `env_file` 键，您可以将敏感信息与 `docker-compose.yaml` 文件分开，并在不同的环境中轻松管理环境变量。

举例来说，考虑一个 MariaDB Docker 镜像。MariaDB 镜像需要设置几个环境变量来配置数据库，例如 `MYSQL_ROOT_PASSWORD` 用于 root 密码，`MYSQL_DATABASE` 用于默认数据库的名称，以及其他变量。这些环境变量可以在 `docker-compose.yaml` 文件中定义以配置 MariaDB 服务。

让我们看一个使用 Docker Compose 设置 nginx 容器、PHP-FPM 容器、WordPress 容器和 MySQL 容器的例子。我们将在 `docker-compose.yml` 文件中定义我们的服务，并通过注释将其分解成较小的块：

```
version: '3'
```

前一行定义了 Docker Compose 文件语法的版本。接下来，我们将定义所有要运行并与彼此交互的 Docker 镜像：

```
services:
  web:
    image: nginx:latest
    depends_on:
      - wordpress
    ports:
      - 80:80
    volumes:
      - ./nginx.conf:/etc/nginx/conf.d/default.conf
      - wordpress:/var/www/html
    networks:
      - wordpress
```

这定义了我们应用程序栈中名为`web`的组件。它将使用来自 Docker Hub 的`nginx`镜像，标签为`latest`。以下是一些其他重要设置：

+   `depends_on`: 这告诉 Docker Compose 在`wordpress`服务之后启动此组件。

+   `ports`: 这将您的主机端口转发到 Docker 端口；在这种情况下，它将在计算机上打开端口`80`并将所有传入流量转发到 Docker 镜像中的相同端口，就像在命令行中启动单个 Docker 容器时所做的`-p`设置一样。

+   `volumes`: 这个设置等同于 Docker 命令行工具中的`-v`选项，因此它将从本地目录将一个`nginx.conf`文件挂载到 Docker 镜像内的`/etc/nginx/conf.d/default.conf`文件。

+   `wordpress:/var/www/html`: 这一行将一个名为`wordpress`的 Docker 卷挂载到 Docker 镜像内部的目录。卷将在前面定义。

+   `networks`: 在这里，我们将此服务连接到名为`wordpress`的 Docker 网络，其定义如下：

```
  wordpress:
    image: wordpress:php8.2-fpm-alpine
    depends_on:
      - db
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER: example_user
      WORDPRESS_DB_NAME: example_database
      WORDPRESS_DB_PASSWORD: example_password
    restart: unless-stopped
    volumes:
      - wordpress:/var/www/html
    networks:
      - wordpress
```

前面的服务与*web*服务非常相似，具有以下附加项：

+   `environment`: 这定义了 Docker 镜像内部存在的环境变量。

+   `restart`: 这配置了服务，使其在某些原因导致进程停止工作时自动重启。如果我们手动停止了该服务，Docker Compose 将不会尝试重新启动它。

+   `depends_on`: 该服务器只会在`db`服务启动后才会启动。

让我们看看`db`服务：

```
  db:
    image: mariadb:10.4
    environment:
      MYSQL_ROOT_PASSWORD: example_password
      MYSQL_DATABASE: example_database
      MYSQL_USER: example_user
      MYSQL_PASSWORD: example_password
    restart: unless-stopped
    volumes:
      - dbdata:/var/lib/mysql
    networks:
      - wordpress
```

此服务正在设置 MariaDB 数据库，以便存储 WordPress 数据。请注意，我们可以为 MariaDB 或 WordPress 镜像使用的所有环境变量都记录在它们各自的 Docker Hub 页面上：

```
volumes:
  wordpress:
  dbdata:
```

在这里，我们正在定义用于 WordPress 和 MariaDB 的 Docker 卷。这些是存储在本地的常规 Docker 卷，但通过安装 Docker 引擎插件，可以将它们作为分布式文件系统（如 GlusterFS 或 MooseFS）分发：

```
networks:
  wordpress:
    name: wordpress
    driver: bridge
```

最后，我们定义了一个`wordpress`网络，使用`bridge`驱动程序，允许所有前述服务之间的通信，并与在其上运行的 Docker 镜像隔离开来。

在前面的示例中，除了本节已经涵盖的选项之外，还有一个服务依赖项（`depends_on`），这将允许我们强制容器启动的顺序。

我们定义的两个卷（`wordpress`和`dbdata`）用于数据持久化。`wordpress`卷用于托管所有 WordPress 文件，并且它也被挂载到运行 nginx Web 服务器的 web 容器上。这样，Web 服务器将能够提供静态文件，如 CSS、图像和 JavaScript，同时将请求转发到 PHP-FPM 服务器。

这是 nginx 配置，它使用 `fastcgi` 连接到运行 PHP-FPM 守护进程的 WordPress 容器：

```
server {
    listen 80;
    listen [::]:80;
    index index.php index.htm index.xhtml;
    server_name _;
    error_log  /dev/stderr;
    access_log /dev/stdout;
    root /var/www/html;
    location / {
        try_files $uri $uri/ /index.php;
    }
    location ~ \.php$ {
      include fastcgi_params;
      fastcgi_intercept_errors on;
      fastcgi_pass wordpress:9000;
      fastcgi_param  SCRIPT_FILENAME $document_root$fastcgi_script_name;
    }
    location ~* \.(js|css|png|jpg|jpeg|gif|ico)$ {
      expires max;
      log_not_found off;
    }
}
```

使用这个 `docker-compose.yml` 文件，你可以通过 `docker-compose up` 和 `docker-compose down` 命令分别启动和停止文件中定义的所有服务。当你运行 `docker-compose up` 时，Docker 将下载必要的镜像并启动容器，你就可以在 [`localhost`](http://localhost) 访问你的 WordPress 网站。

`docker-compose` 是一个非常有用的工具，用于以简单且可重复的方式运行需要多个服务的应用程序。它通常在本地开发时运行应用程序时使用，但一些组织决定在生产系统中使用 `docker-compose`，在这种情况下它也能发挥作用。

如果你能使用现成的 Docker 镜像进行本地开发或生产环境，实际上是极为罕见的。使用公共镜像作为自定义的基础是所有使用 Docker 的组织普遍采用的做法。考虑到这一点，在接下来的章节中，我们将学习如何使用多阶段构建构建 Docker 镜像，以及如何正确使用每个 Dockerfile 命令。

# 高级 Dockerfile 技巧

Dockerfile 用于定义应用程序在 Docker 容器内应如何构建。我们在 *第八章* 中介绍了大部分可用命令。在这里，我们将介绍一些更高级的技术，例如多阶段构建或不太常见的 `ADD` 命令用法。

## 多阶段构建

多阶段构建是 Docker 的一个功能，允许你使用多个 Docker 镜像创建一个最终的单一镜像。通过创建多个阶段，你可以将构建过程分为不同的步骤，从而减少最终镜像的大小。多阶段构建在构建需要多个依赖的复杂应用程序时特别有用，因为它允许开发人员将必要的依赖项放在一个阶段，而将应用程序本身放在另一个阶段。

使用多阶段构建来构建 Golang 应用程序的一个例子涉及创建两个阶段：一个用于构建应用程序，另一个用于运行它。在第一个阶段，Dockerfile 拉取必要的依赖项并编译应用程序代码。在第二个阶段，仅从第一个阶段复制已编译的二进制文件，从而减少最终镜像的大小。以下是一个 Golang 应用程序的 Dockerfile 示例：

```
# Build stage
FROM golang:alpine AS build
RUN apk add --no-cache git
WORKDIR /app
COPY . .
RUN go mod download
RUN go build -o /go/bin/app
# Final stage
FROM alpine
COPY --from=build /go/bin/app /go/bin/app
EXPOSE 8080
ENTRYPOINT ["/go/bin/app"]
```

在上述示例中，Dockerfile 创建了两个阶段。第一个阶段使用 `golang:alpine` 镜像并安装必要的依赖项。然后，它编译应用程序并将二进制文件放置在 `/go/bin/app` 目录下。第二个阶段使用更小的 Alpine 镜像，并将第一个阶段中的二进制文件复制到 `/go/bin/app` 目录下。最后，它将入口点设置为 `/go/bin/app`。

## `ADD` 命令的用法示例

Dockerfile 中的 `ADD` 命令用于将文件或目录添加到 Docker 镜像中。它的工作方式与 `COPY` 相同，但具有一些额外的功能。我们之前已经讨论过基本的用法，但还有其他用例。

第二种用法允许你即时解压 ZIP 或 TAR 和 gzip 工具压缩的文件。在将压缩文件添加到镜像时，文件将被解压，所有文件将被提取到目标文件夹中。以下是一个示例：

```
ADD my-tar-file.tar.gz /app
```

使用 `ADD` 命令的第三种方式是从 URL 将远程文件复制到 Docker 镜像中。例如，要从 URL [`yourdomain.tld/configurations/nginx.conf`](https://yourdomain.tld/configurations/nginx.conf) 下载名为 `file.txt` 的文件并将其复制到 Docker 镜像中的 nginx 配置目录 `/etc/nginx`，可以使用以下 `ADD` 命令：

```
ADD https://yourdomain.tld/configurations/nginx.conf /etc/nginx/nginx.conf
```

你也可以使用 Git 仓库来添加代码：

```
ADD --keep-git-dir=true https://github.com/your-user-or-organization/some-repo.git#main /app/code
```

要通过 SSH 克隆 Git 仓库，你需要允许 Docker 内的 `ssh` 命令访问拥有访问你要克隆仓库的私钥。你可以通过在多阶段构建中添加私钥，并在克隆仓库的阶段结束时将其移除来实现这一点。如果可能的话，一般不推荐这样做。你可以通过使用 Docker secrets 并在构建时挂载该 secret 来更安全地实现。

下面是使用 `ARG` 配合私钥的示例：

```
ARG SSH_PRIVATE_KEY
RUN mkdir /root/.ssh/
RUN echo "${SSH_PRIVATE_KEY}" > /root/.ssh/id_rsa
# make sure your domain is accepted
RUN touch /root/.ssh/known_hosts
RUN ssh-keyscan gitlab.com >> /root/.ssh/known_hosts
RUN git clone git@gitlab.com:your-user/your-repo.git
```

下面是使用 Docker secret 和挂载的示例：

```
FROM python:3.9-alpine
WORKDIR /app
RUN --mount=type=secret,id=ssh_id_rsa,dst=~/id_rsa chmod 400 ~/id_rsa \
  && ssh-agent bash -c 'ssh-add ~/id_rsa; git clone git@gitlab.com:your-user/your-repo.git' && rm -f ~/id_rsa
  # Rest of the build process follows…
```

在前面的示例中，我们假设你的私钥没有密码保护，并且你的密钥保存在 `ssh_id_rsa` 文件中。

使用 `ADD` 命令的最后一种方式是从主机机器中提取 TAR 压缩包并将其内容复制到 Docker 镜像中。例如，要从主机机器中提取一个名为 `data.tar.gz` 的 TAR 压缩包并将其内容复制到 Docker 镜像中的 `/data` 目录，可以使用以下 `ADD` 命令：

```
ADD data.tar.gz /data/
```

## Secrets 管理

Docker secrets 管理是构建安全可靠的容器化应用程序的一个重要方面。

Secrets 是应用程序需要运行的敏感信息，但不应暴露给未授权的用户或进程。Secrets 的例子包括密码、API 密钥、SSL 证书以及其他身份验证或授权令牌。这些 secrets 通常在应用程序运行时需要，但如果将其以明文形式存储在代码或配置文件中，则存在安全风险。

保护 secrets 对于确保应用程序的安全性和可靠性至关重要。泄露 secrets 可能导致数据泄露、服务中断和其他安全事件。

在基本的 Docker 设置中，您只能通过环境变量将密钥提供给 Docker 镜像，正如我们在*第八章*中介绍的那样。Docker 还提供了一个内置的密钥管理机制，可以安全地存储和管理密钥。但是，它仅在启用 Swarm 模式时可用（我们将在本章的*Docker* *编排*部分进一步探讨 Swarm）。

要使密钥可用于运行在 Docker 中的应用程序，您可以使用`docker secret create`命令。例如，要为 MySQL 数据库密码创建一个密钥，您可以使用以下命令：

```
admin@myhome:~/$ echo "mysecretpassword" | docker secret create mysql_password -
```

此命令创建了一个名为`mysql_password`的密钥，值为`mysecretpassword`。

要在服务中使用密钥，您需要在服务配置文件中定义该密钥。例如，要在服务中使用`mysql_password`密钥，您可以在`docker-compose.yml`文件中定义它，如下所示：

```
version: '3'
services:
  db:
    image: mysql
    environment:
      MYSQL_ROOT_PASSWORD_FILE: /run/secrets/mysql_password
    secrets:
      - mysql_password
    volumes:
      - db_data:/var/lib/mysql
secrets:
  mysql_password:
    external: true
volumes:
  db_data:
```

在此配置文件中，`mysql_password`密钥定义在`secrets`部分，且`MYSQL_ROOT_PASSWORD_FILE`环境变量设置为密钥文件的路径，即`/run/secrets/mysql_password`。

要部署服务，您可以使用`docker stack deploy`命令。例如，要部署在`docker-compose.yml`文件中定义的服务，您可以使用以下命令：

```
admin@myhome:~/$ docker stack deploy -c docker-compose.yml myapp
```

从安全角度来看，小心处理密钥至关重要。最常见的错误是将密钥直接放入 Docker 镜像、环境文件或提交到 Git 仓库的应用程序配置文件中。虽然已有现成的应急措施防止用户这么做（如 GitHub 中的 Dependabot），但如果它们失败，之后从 Git 历史中删除这些密钥将非常困难。

在这一部分中，我们介绍了如何处理构建容器的不同方面以及高级构建技巧。掌握这些知识并使用 Docker Compose，您将能够以相当程度的自动化构建和运行您的应用程序。如果您有 10 个这样的应用程序呢？100 个？甚至更多呢？

在接下来的部分，我们将深入探讨集群，它将进一步自动化操作，并将您的应用程序同时部署到多个主机。

# Docker 编排

在容器化的世界中，**编排**是自动化部署、管理和扩展应用程序到多个主机的过程。编排解决方案通过提供一个抽象层，帮助简化容器化应用程序的管理，提升可用性，并改善可扩展性，使您可以在更高层次上管理容器，而不是手动管理每个容器。

**Docker Swarm** 是一个原生的 Docker 集群和编排工具，允许你创建并管理 Docker 节点的集群，使用户能够在多个主机上部署和管理 Docker 容器。Docker Swarm 是一个易于使用的解决方案，并且与 Docker 一起内置，成为了那些已经熟悉 Docker 的用户的热门选择。

**Kubernetes** 是一个开源容器编排平台，最初由 Google 开发。Kubernetes 允许你在多个主机上部署、扩展和管理容器化应用，同时提供了自愈、自动发布和回滚等高级功能。Kubernetes 是目前最流行的编排解决方案之一，并广泛应用于生产环境。

**OpenShift** 是一个建立在 Kubernetes 基础上的容器应用平台，由 Red Hat 开发。这个平台提供了一个完整的解决方案，用于部署、管理和扩展容器化应用，附加功能包括内建的 CI/CD 流水线、集成监控和自动扩展。OpenShift 设计为企业级平台，具备多租户支持和**基于角色的访问控制**（**RBAC**）等功能，是需要管理复杂容器化环境的大型组织的热门选择。

市场上有各种各样的编排解决方案，每种方案都有其优缺点。选择使用哪种方案最终取决于你的具体需求，但 Docker Swarm、Kubernetes 和 OpenShift 都是提供强大且可靠的容器化应用编排功能的热门选择。

## Docker Swarm

Docker Swarm 是一个为 Docker 容器提供的原生集群和编排解决方案。它提供了一种简单而强大的方式来管理和扩展跨主机的 Docker 化应用。通过 Docker Swarm，用户可以创建并管理一个由 Docker 节点组成的集群，这些节点作为一个虚拟系统共同工作。

Docker Swarm 的基本组件如下：

+   **节点**：这些是构成 Swarm 的 Docker 主机。节点可以是运行 Docker 守护进程的物理或虚拟机，并可以根据需要加入或退出 Swarm。

+   **服务**：这些是在 Swarm 上运行的应用程序。服务是一个可扩展的工作单元，定义了应用程序应该运行多少个副本，以及如何在 Swarm 上部署和管理它们。

+   **管理节点**：这些节点负责管理 Swarm 状态并协调服务的部署。管理节点负责维持服务的期望状态，确保它们按预期运行。

+   **工作节点**：这些节点运行实际的容器。工作节点从管理节点接收指令，并运行所需的服务副本。

+   **覆盖网络**：这些是允许服务相互通信的网络，无论它们运行在哪个节点上。覆盖网络提供了一个跨越整个 Swarm 的透明网络。

Docker Swarm 提供了一种简单且易于使用的方式来管理容器化应用程序。它与 Docker 生态系统紧密集成，为 Docker 用户提供了熟悉的界面。借助其内置的服务发现、负载均衡、滚动更新和扩展功能，Docker Swarm 是刚开始使用容器编排的组织的热门选择。

要初始化 Docker Swarm 模式并将两个工作节点添加到集群中，你需要初始化 Swarm 模式：

```
admin@myhome:~/$ docker swarm init
Swarm initialized: current node (i050z7b0tjoew7hxlz419cd8l) is now a manager.
To add a worker to this swarm, run the following command:
    docker swarm join --token SWMTKN-1-0hu2dmht259tb4skyetrpzl2qhxgeddij3bc1wof3jxh7febmd-6pzkhrh4ak345m8022hauviil 10.0.2.15:2377
To add a manager to this swarm, run 'docker swarm join-token manager' and follow the instructions.
```

这将创建一个新的 Swarm，并将当前节点设置为 Swarm 管理器。

一旦 Swarm 初始化完成，你可以将工作节点添加到集群中。为此，你需要在每个工作节点上运行以下命令：

```
admin@myhome:~/$ docker swarm join --token <token> <manager-ip>
```

在这里，`<token>` 是 `docker swarm init` 命令输出中生成的令牌，你可以在前面的代码块中找到它，而 `<manager-ip>` 是 Swarm 管理器的 IP 地址。

例如，如果令牌是 `SWMTKN-1-0hu2dmht259tb4skyetrpzl2qhxgeddij3bc1wof3jxh7febmd-6pzkhrh4ak345m8022hauviil`，而管理节点 IP 地址是 `10.0.2.15`，命令将如下所示：

```
admin@myhome:~/$ docker swarm join --token SWMTKN-1-0hu2dmht259tb4skyetrpzl2qhxgeddij3bc1wof3jxh7febmd-6pzkhrh4ak345m8022hauviil 10.0.2.15
```

在每个工作节点上运行 `docker swarm join` 命令后，你可以通过在 Swarm 管理节点上运行以下命令来验证它们是否已加入 Swarm：

```
admin@myhome:~/$ docker node ls
ID                      HOSTNAME   STATUS    AVAILABILITY   MANAGER STATUS   ENGINE VERSION
i050z7b0tjoew7hxlz419cd8l *   myhome    Ready     Active         Leader           23.0.1
```

这将显示 Swarm 中所有节点的列表，包括管理节点和你已添加的任何工作节点。

在那之后，你可以添加更多节点并开始将应用程序部署到 Docker Swarm。你可以重用任何正在使用的 Docker Compose 文件或 Kubernetes 清单。

要部署一个示例应用程序，我们可以通过部署 `wordpress` 服务来重用 Docker Compose 模板，但我们需要稍微更新它，通过在环境变量中使用 MySQL 用户和密码文件：

```
  wordpress:
    image: wordpress:php8.2-fpm-alpine
    depends_on:
      - db
    environment:
      WORDPRESS_DB_HOST: db:3306
      WORDPRESS_DB_USER_FILE: /run/secrets/mysql_user
      WORDPRESS_DB_NAME: example_database
      WORDPRESS_DB_PASSWORD_FILE: /run/secrets/mysql_password
```

以下是向 `wordpress` 和 `db` 服务添加机密的示例：

```
    secrets:
      - mysql_user
      - mysql_password
  db:
    image: mariadb:10.4
    environment:
      MYSQL_ROOT_PASSWORD: example_password
      MYSQL_DATABASE: example_database
      MYSQL_USER_FILE: /run/secrets/mysql_user
      MYSQL_PASSWORD_FILE: /run/secrets/mysql_password
```

以下是一个在 `docker-compose.yml:secrets` 文件底部添加机密定义的示例：

```
  mysql_user:
    external: true
  mysql_password:
    external: true
```

`external: true` 设置告诉 `docker-compose` 机密已经存在，它不应该尝试更新或重新创建这些机密。

在这个版本的 Compose 文件中，我们使用机密来存储 `wordpress` 和 `db` 服务的 MySQL 用户和密码。

要将此文件部署到 Docker Swarm，我们可以使用以下命令：

```
admin@myhome:~/$ echo "root" | docker secret create mysql_user -
vhjhswo2qg3bug9w7id08y34f
echo "mysqlpwd" | docker secret create mysql_password -
oy9hsbzmzrh0jrgjo6bgsydol
```

然后，我们可以部署堆栈：

```
admin@myhome:~/$ docker stack deploy -c docker-compose.yml wordpress
Ignoring unsupported options: restart
Creating network wordpress_wordpress
Creating service wordpress_web
Creating service wordpress_wordpress
Creating service wordpress_db
```

这里，`docker-compose.yaml` 是 Compose 文件的名称，`my-stack-name` 是 Docker 堆栈的名称。

一旦堆栈部署完成，`wordpress`、`web` 和 `db` 服务将使用在机密中指定的 MySQL 用户和密码运行。你可以通过列出堆栈并检查容器是否正在运行来验证这一点：

```
admin@myhome:~/$ root@vagrant:~# docker stack ls
NAME        SERVICES
wordpress   3
root@vagrant:~# docker ps
CONTAINER ID     IMAGE     CREATED      STATUS       PORTS      NAMES
7ea803c289b0   mariadb:10.4                  "docker-entrypoint.s…"   28 seconds ago   Up 27 seconds   3306/tcp   wordpress_db.1.dogyh5rf52zzsiq0t95nrhuhc
ed25de3273a2   wordpress:php8.2-fpm-alpine   "docker-entrypoint.s…"   33 seconds ago   Up 31 seconds   9000/tcp   wordpress_wordpress.1.xmmljnd640ff9xs1249jpym45
```

Docker Swarm 是一个很好的项目，可以让你开始学习 Docker 编排方法。通过使用各种插件扩展其默认功能，它也可以用于生产级系统。

## Kubernetes 和 OpenShift

两个最受欢迎的 Docker 容器编排工具是 Kubernetes 和 OpenShift。尽管它们有一些相似之处，但也存在一些显著的区别。以下是 Kubernetes 和 OpenShift 之间的主要区别：

+   **架构**：Kubernetes 是一个独立的编排平台，旨在与多个容器运行时（包括 Docker）一起工作。而 OpenShift 是建立在 Kubernetes 之上的平台。它提供了额外的功能和工具，如源代码管理、持续集成和部署。这些附加功能使 OpenShift 成为一个更全面的解决方案，适合需要端到端 DevOps 功能的企业。

+   **易用性**：Kubernetes 是一个强大的编排工具，需要较高的技术水平才能设置和操作。而 OpenShift 则旨在更加用户友好，适合具有不同技术背景的开发人员。OpenShift 提供了一个基于 Web 的界面来管理应用，并且可以与各种开发工具集成，方便开发人员使用。

+   **成本**：Kubernetes 是一个开源项目，可以免费使用，但企业可能需要投入额外的工具和资源来搭建和操作它。OpenShift 是一个企业级平台，完全访问其功能和支持需要订阅。OpenShift 的成本可能高于 Kubernetes，但它提供了额外的功能和支持，对于需要高级 DevOps 功能的企业来说，可能值得投资。

这两种解决方案都是强大的 Docker 编排工具，提供不同的优势和权衡。Kubernetes 高度可定制，适合更具技术背景的用户。另一方面，OpenShift 提供了更全面的解决方案，具有额外的功能和更易于使用的界面，但成本较高。在选择这两种工具时，你应该考虑你组织的具体需求，同时记住 Docker Swarm 也是一个可选项。云服务提供商也开发了自己的解决方案，其中 Elastic Container Service 就是其中之一。

# 总结

本章中，我们讨论了关于 Docker 的更高级话题，仅涉及了容器编排相关的内容。Kubernetes、OpenShift 和云服务提供商提供的 SaaS 解决方案正在推动新工具的创造，这些工具将进一步简化 Docker 在现代应用中的使用。

Docker 对软件开发和部署的世界产生了深远影响，使我们能够比以往更加高效和可靠地构建、发布和运行应用程序。通过了解 Docker 的内部机制并遵循最佳实践，我们可以确保我们的应用程序在各种环境中都是安全、高效和可扩展的。

在下一章，我们将探讨在基于 Docker 容器的分布式环境中，如何监控和收集日志的挑战。

# 第三部分：DevOps 云工具包

本书的最后部分将更多地关注通过**配置即代码**（**CaC**）和**基础设施即代码**（**IaC**）实现自动化。我们还将讨论监控和追踪作为现代应用程序开发和维护的关键部分。在最后一章，我们将讨论我们在许多项目中经历过的 DevOps 陷阱。

本部分包含以下章节：

+   *第十章*，*监控、追踪与分布式日志记录*

+   *第十一章*，*使用 Ansible 进行配置即代码*

+   *第十二章*，*利用基础设施即代码*

+   *第十三章*，*使用 Terraform、GitHub 和 Atlantis 进行 CI/CD*

+   *第十四章*，*避免 DevOps 中的陷阱*
