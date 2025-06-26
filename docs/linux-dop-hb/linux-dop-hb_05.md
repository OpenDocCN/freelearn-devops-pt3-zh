# 5

# 管理 Linux 中的服务

在本章中，我们将深入解释服务（作为守护进程在后台运行的程序）。我们将解释 `init` 脚本和 `systemd` 单元。我们还将介绍管理服务的 Alpine Linux `rc` 命令。

本章内容包括以下主题：

+   详细了解 Linux 服务

+   关于 Upstart 的简要介绍，作为一种替代方案

# 技术要求

本章内容，你需要一台可以执行特权命令的 Linux 系统，可以使用 `sudo` 或直接跳到 root 账户（尽管我们特别推荐使用前者）。你还需要一款 Linux 文本编辑器，能够生成纯文本文件。如果你打算在 Windows 系统上编辑，请使用支持保存 Unix 文件的文本编辑器。我们推荐在命令行中使用你喜欢的命令行文本编辑器进行编辑：`vim`、`emacs`、`joe`、`nano`，或者任何你习惯使用的编辑器。

# 详细了解 Linux 服务

除非你在桌面上运行某种低级别的嵌入式设备——我们强烈怀疑这一点——否则你的操作系统会管理大量任务，为你创建一个舒适且高效的环境。无论是 Mac OS X、Linux、Windows 还是 FreeBSD，它们都运行许多后台程序，这些程序共同提供一个有用的系统。服务器版本的操作系统也是如此。后台程序或后台进程（在 Unix 和 Linux 中称为“守护进程”）是指不与任何输入（键盘、鼠标等）或输出（显示器、终端等）连接的程序。这样，它们即使在没有人登录系统时也能开始工作，并且在用户注销时继续工作。它们还可以在一个永远无法登录系统的用户的权限下运行，从而使它们的执行更加安全。

你的 Linux 系统上运行的服务数量，很大程度上取决于发行版，甚至更大程度上取决于系统的用途。

## Linux 服务管理的历史

正如你可以想象的那样，管理系统服务——即那些让你的计算机能够使用的程序——是一项复杂的任务。运行这些服务的软件必须稳定且强大。这一点，以及系统启动不频繁，尤其是在服务器上，促使了 Linux 采用的解决方案得以持续数十年。随着许多线程 CPU 的出现并广泛使用，启动服务的需求日益增加，且更需要智能化管理，促成了新 `init` 系统的多个实现，我们将在接下来的部分中介绍。

## systemd

`systemd`是一个用于 Linux 的服务管理器，能够管理与操作系统一起启动的服务。它取代了传统的`init`脚本。它负责启动和停止系统服务，管理系统状态，以及记录系统事件。它已经成为许多流行 Linux 发行版的默认`init`系统，包括 CentOS、Fedora Linux、**红帽企业 Linux**（**RHEL**）和 Ubuntu Linux。

这个服务管理器负责控制系统本身的初始化（Linux 操作系统所需的服务）、启动和停止系统服务，以及管理系统资源。它提供了另一种管理服务和其他系统组件的方式，并且允许系统管理员以比`init`更标准化的方式配置和自定义系统行为。

`systemd`的一个关键特性是能够并行启动服务，这可以显著减少系统的启动时间。它还包括一些用于管理和监控系统服务的工具。

另外，`systemd`受到赞扬的另一点是它最终为 Linux 世界带来的服务配置统一性。在每个 Linux 发行版中，`systemd`配置文件都被送到相同的路径，并且看起来相同。然而，根据二进制文件的安装路径，仍然存在一些差异。`systemd`还更擅长判断进程是否正在运行，这使得我们更不容易遇到因为过时的文件无法启动进程的情况。

`systemd`的一个主要优势是它对依赖关系的意识。服务（在`systemd`控制下的运行程序）配置包含了关于它依赖的所有其他服务的信息，还可以指向依赖于它的服务。而且，服务可以向`systemd`告知它需要运行的目标：如果您的服务需要网络运行，您可以将此信息写入配置中，`systemd`会确保只有在网络正确配置后才会启动您的守护进程。

以下是作为`systemd`一部分提供的一些工具和实用程序的列表：

+   `systemd`：这是主要的系统和服务管理器。它是控制服务和其他系统组件初始化和管理的主要程序。

+   `systemctl`：这是一个命令行工具，用于管理系统服务和其他系统组件。它可以用于启动、停止、重启、启用和禁用服务，还可以查看服务和其他系统组件的状态。

+   `journalctl`：用于查看和操作系统日志，日志由`systemd`管理。它可以用来查看日志消息，根据各种标准过滤日志消息，并将日志数据导出到文件。

+   `coredumpctl`：这是一个实用工具，顾名思义，它有助于从`systemd`的日志中检索核心转储。

+   `systemd-analyze`：这个工具可以用来分析系统的启动性能。它衡量系统启动所需的时间，以及识别潜在的瓶颈和性能问题的时间。

+   `systemd-cgls`：这是一个用于查看系统上控制组层级的命令行工具。`systemd` 用于管理系统资源并将进程相互隔离。

+   `systemd-delta`：这是一个用于分析 `systemd` 提供的默认配置文件与对这些文件所做的本地修改之间差异的命令行工具。

+   `systemd-detect-virt`：这是一个用于检测系统运行的虚拟化环境的命令行工具。它可以用来判断系统是运行在**虚拟机**（**VM**）、容器中，还是裸机上。

+   `systemd-inhibit`：这是一个命令行工具，用于防止执行某些系统操作，例如暂停或关闭系统。

+   `systemd-nspawn`：这是一个用于在轻量级容器中运行进程的命令行工具。它可以用来创建和管理容器，以及在容器内执行进程。

这只是 `systemd` 提供的一些常见工具和实用程序的列表。还有很多其他工具，但我们在这里不再讨论它们。

### 目标

在 `systemd` 中，**目标** 是系统可以处于的特定状态，并通过一个符号名称表示。目标用于定义系统的高级行为，通常用于将一组相关的服务和其他系统组件组合在一起。

例如，`multi-user.target` 是一个表示已准备好提供多用户访问的系统，启用了网络和其他服务；`graphical.target` 是一个表示已准备好显示图形登录界面的系统，启用了图形桌面环境和相关服务。

目标通常在单元文件中定义，这些文件是描述系统组件属性和行为的配置文件。当目标被激活时，`systemd` 将启动所有与该目标相关的服务和其他系统组件。

`systemd` 包括许多预定义的目标，涵盖了广泛的常见系统状态，管理员还可以定义自定义目标以满足其系统的特定需求。目标可以通过 `systemctl` 命令激活，或者通过修改系统启动时设置的默认目标来激活。

下面是一些预定义的 `systemd` 目标的示例：

+   `poweroff.target`：表示一个正在关机或已关闭的系统。

+   `rescue.target`：表示一个处于救援模式的系统，启用了最小化的服务。

+   `multi-user.target`：表示一个已经准备好提供多用户访问、启用网络和其他服务的系统。

+   `graphical.target`：表示一个已经准备好显示图形登录屏幕的系统，启用了图形桌面环境和相关服务。

+   `reboot.target`：表示一个正在重启的系统。

+   `emergency.target`：表示一个运行在紧急模式下的系统，仅启用最基本的服务。

这是一个定义自定义目标的`systemd`单元文件示例：

```
[Unit]
Description=Unit File With Custom Target
[Install]
WantedBy=multi-user.target
```

这个单元文件定义了一个名为`Custom Target`的目标，旨在作为`multi-user.target`的一部分被激活。`WantedBy`指令指定，当`multi-user.target`被激活时，目标应该被激活。

这是另一个名为`custom.target`的`systemd`单元文件示例，它定义了一个自定义目标：

```
[Unit]
Description=My simple service
[Install]
WantedBy=multi-user.target
```

以下是一个使用我们`custom.target`目标的单元文件：

```
[Service]
ExecStart=/usr/local/bin/my-simple-service
Type=simple
[Install]
WantedBy=custom.target
```

这个单元文件定义了一个名为`Unit File With Custom Target`的目标和一个名为`My simple service`的服务。`ExecStart`指令指定了启动服务时应使用的命令，而`Type`指令指定了服务的类型。服务单元中`[Install]`部分的`WantedBy`指令指定，当`custom.target`被激活时，服务应该被激活。

现在，既然我们已经简单介绍了单元文件，让我们深入了解它们，看看能用它们做些什么。

### 单元文件

单元文件通常存储在 Linux 操作系统文件系统中的`/lib/systemd/`系统目录中。此目录中的文件不应以任何方式修改，因为当通过软件包管理器升级服务时，这些文件将会被包中的文件替换。

相反，要修改特定服务的单元文件，请在`/etc/systemd/system`目录中创建自定义单元文件。此`etc`目录中的文件将优先于默认位置的文件。

`systemd`能够使用以下方式提供单元激活：

+   `systemd`。最简单的情况是，如果您的服务使用了网络，您需要添加一个与`multi-user.target`或`network.target`的依赖关系。

+   `/etc/systemd/system`目录。

+   **模板**：您还可以定义模板单元文件。这些特殊单元可用于创建同一通用单元的多个实例。

+   `private/tmp`或网络访问，并限制内核功能。

+   **路径**：您可以基于文件系统中某个文件或目录的活动或可用性来启动一个单元。

+   **套接字**：这是 Linux 操作系统中的一种特殊类型的文件，它允许两个进程之间进行通信。通过使用此功能，您可以延迟启动服务，直到相关的套接字被访问。您还可以创建一个单元文件，在启动过程的早期仅创建一个套接字，并创建一个单独的单元文件来使用此套接字。

+   **总线**：您可以使用 D-Bus 提供的总线接口来激活单元。D-Bus 只是一个用于**进程间通信**（**IPC**）的消息总线，最常用于 GNOME 或 KDE 等图形用户界面中。

+   `dev` 文件，位于 `/dev` 目录下）。这将利用一种被称为 `udev` 的机制，`udev` 是一个 Linux 子系统，用于提供设备事件。

一旦你启动了一个服务，你可能希望通过查看日志文件来检查它是否正常运行。这项工作由 `journald` 完成。

### 日志记录

每个由 `systemd` 管理的服务都会将其日志发送到 `journald`——`systemd` 的一个特殊部分。管理这些日志有一个专用的命令行工具：`journalctl`。在最简单的形式下，运行 `journalctl` 命令会输出所有系统日志，最新的日志会显示在最上面。虽然日志的格式类似于 `syslog`——这是 Linux 上用于收集日志的传统工具——但 `journald` 捕获了更多数据。它收集了启动过程、内核日志等信息。

启动日志默认是临时的。这意味着它们不会在系统重启之间保存。然而，也可以将它们永久记录下来。下面是两种方法：

+   创建一个特殊的目录。当 `journald` 在系统启动时检测到该目录时，它会将日志保存在该目录中：`sudo mkdir -p /var/log/journal`。

+   编辑 `journald` 配置文件并启用持久化启动日志。使用你喜欢的编辑器打开 `/etc/systemd/journald.conf` 文件，在 `[Journal]` 部分，将 `Storage` 选项修改为如下所示：

    ```
    [Journal]
    ```

    ```
    Storage=persistent
    ```

`journald` 可以通过使用 `-u service.name` 选项按服务过滤日志——也就是说，`journalctl -u httpd.service` 只会打印来自 `httpd` 守护进程的日志。你也可以通过 `man journalctl` 命令来查看如何在指定的时间范围内打印日志，或从多个服务中打印日志。

在本节中，我们介绍了 Linux 世界中最常用的服务软件——`systemd`。在下一节中，我们将探讨 OpenRC——一种用于 Alpine Linux 的系统，Alpine Linux 是云端容器首选的 Linux 发行版。

## OpenRC

Alpine Linux 使用另一种用于管理系统服务的系统——`init` 系统，最初为 Gentoo Linux 开发。它旨在轻量、简单且易于维护。OpenRC 使用纯文本配置文件，使其易于定制和配置。它也很容易通过自定义脚本和程序进行扩展。OpenRC 灵活，可以在各种系统上使用，从嵌入式设备到服务器。

以下是 OpenRC 在 Alpine Linux 中的使用示例：

+   `ssh` 和 `cron`。你可以使用 `rc-service` 命令来启动、停止或检查服务的状态。例如，要启动 `ssh` 服务，可以运行 `rc-service` `ssh start`。

+   **自定义系统初始化和关机**：OpenRC 允许你编写自定义脚本来定制系统在启动或关机过程中的行为。这些脚本会在启动过程中的特定点执行，用于设置自定义配置或执行其他任务。

+   `rc-update`命令用于将服务添加或移除到不同的运行级别。例如，要使服务在启动时启动，可以运行`rc-update add <service> boot`。

要启动服务，请使用以下`rc-service`命令：

```
admin@myhome:~$ rc-service <service> start
```

要停止服务，请使用以下`rc-service`命令：

```
admin@myhome:~$ rc-service <service> stop
```

要检查服务的状态，请使用以下`rc-service`命令：

```
admin@myhome:~$ rc-service <service> status
```

要启用服务在启动时启动，请使用以下`rc-update`命令：

```
admin@myhome:~$ rc-update add <service> default
```

要禁止服务在启动时启动，请使用以下`rc-update`命令：

```
admin@myhome:~$ rc-update del <service> default
```

在此上下文中，默认指的是 Alpine Linux 系统的默认运行级别。运行级别通常用于定义系统的行为。大多数 Linux 发行版都有几个预定义的运行级别，每个运行级别对应一组特定的服务，这些服务会被启动或停止。

在 Alpine Linux 中，以下是默认的运行级别：

+   `default`：这是默认的运行级别，用于系统正常启动时。此运行级别中启动的服务包括网络、SSH 和系统日志。

+   `boot`：此运行级别在系统启动时使用。此运行级别中启动的服务包括系统控制台、系统时钟和内核。

+   `single`：此运行级别在系统启动到单用户模式时使用。此运行级别中启动的服务仅包含一组最小服务，包括系统控制台和系统时钟。

+   `shutdown`：此运行级别在系统关闭时使用。此运行级别中停止的服务包括网络、SSH 和系统日志。

OpenRC 使用与我们在本章前面提到的 SysV `init`非常相似的方式来定义服务操作。像`start`、`stop`、`restart`和`status`这样的命令是在 Bash 脚本中定义的。以下是一个基本的服务示例：

```
# Name of the service
name="exampleservice"
# Description of the service
description="This is my example service"
# Start command
start() {
  # Add your start commands
}
# Stop command
stop() {
  # Add your stop command here
}
# Restart command
restart() {
  stop
  start
}
```

要创建一个新的服务，可以将此文件复制到新文件并根据需要修改`name`、`description`、`start`、`stop`和`restart`函数。`start`函数应包含启动服务的命令，`stop`函数应包含停止服务的命令，`restart`函数应停止并重新启动服务。它们可以与 SysV `init`中的相同。

在 OpenRC 中，`init`脚本通常存储在`/etc/init.d`目录中。这些脚本用于启动和停止服务以及管理系统的运行级别。

要为 OpenRC 创建一个新的`init`脚本，可以在`/etc/init.d`目录中创建一个新文件并使其可执行。

创建`init`脚本后，可以使用`rc-update`命令将其添加到默认运行级别，这将导致服务在启动时启动。例如，要将`exampleservice`服务添加到默认运行级别，可以运行以下命令：

```
admin@myhome:~$ rc-update add exampleservice default
```

在大多数情况下，我们将在 Docker 环境中使用 Alpine Linux，在这种环境下 OpenRC 的使用不多，但它仍然对某些边缘案例的使用有所帮助。我们将在*第八章*和*第九章*中更详细地讨论 Docker。

在本节中，我们已经介绍了 OpenRC，它是管理 Alpine Linux 中系统服务的软件。在下一节中，我们将简要介绍一种过时的 SysV `init`形式，这种形式可能出现在较老或最小化的 Linux 发行版中。

## SysV init

如前所述，`init`进程是系统中最重要的、持续运行的进程。它负责在系统启动时或管理员请求时启动系统服务，在系统关机时或请求时停止系统服务，且按正确的顺序执行。它还负责在请求时重新启动服务。由于`init`将代表 root 用户执行代码，因此必须确保其经过充分的稳定性和安全性测试。

旧的`init`系统的一个迷人特点是它的简单性。服务的启动、停止和重启是通过脚本来管理的——这些脚本必须由应用程序作者、该应用程序的发行版包所有者或系统管理员编写——如果该服务不是通过包安装的。脚本简单且易于理解、编写和调试。

然而，随着软件复杂性的增加，旧的`init`系统的局限性变得越来越明显。启动一个简单的服务还可以，但启动由多个程序组成的应用变得更加困难，尤其是当它们之间的依赖关系变得更加重要时。`init`缺乏对它所管理的服务启动依赖关系的观察。

旧的`init`系统越来越不适合现代系统的另一个原因是串行启动：它无法并行启动服务，从而抵消了现代多核 CPU 的优势。是时候寻找一个更适合新时代的系统了。

一个典型的`init`系统由以下几个组件组成：

+   一个包含启动/停止脚本的`/etc/init.d`或`/etc/rc.d/init.d`目录。

+   一个`/etc/inittab`文件，用于定义运行级别并设置默认的运行级别。

+   一个`/etc/rcX.d`目录，包含所有应在运行级别`X`中启动或停止的服务的脚本，其中`X`是从`0`（零）到`6`的数字。我们将在下一段中详细介绍。

`/etc/init.d/` 目录包含用于启动、停止和重启服务的 Shell 脚本。该脚本接受一个参数，可以是 `start`、`stop` 或 `restart`。传递给脚本的每个参数都会执行相应的函数（通常函数名称与参数相同：`start`、`stop` 或 `restart`），该函数会执行一系列步骤来正确启动、停止或重启指定的服务。系统最终进入的启动过程的最终状态被称为**运行级别**。运行级别决定了服务是否正在启动或停止，并且如果是启动服务，哪些服务会被启动。

为了确定要调用的操作类型，会在与相关运行级别相对应的目录中创建一个指向脚本的链接。

假设我们希望系统最终进入运行级别 `3`。如果我们希望在该运行级别启动我们的服务，我们会创建一个指向 `/etc/rc.d/my_service` 脚本的链接，该链接指向 `/etc/rc3.d/` 目录。链接的名称决定了操作的类型和顺序。因此，如果我们希望服务在 `01` 到 `49` 之间的数字之后启动，我们会将其命名为 `/etc/rc.3/S50my_service`。字母 `S` 告诉 `init` 系统启动该服务，数字 `50` 告诉它在所有较低数字的服务启动之后再启动该服务。请注意，编号更多的是一种框架，并不能保证在 `50` 之前有所有其他数字的脚本。停止服务也是如此。在确定停止系统的默认运行级别后（通常是 `0`），会为服务脚本创建一个适当的 `symlink`。

前述框架的主要问题在于它完全没有考虑依赖关系。确保守护进程所依赖的服务正在运行的唯一方法是将其脚本化，放入 `start` 函数中。在包含多个服务的复杂应用程序中，这可能会导致更大的启动/停止脚本。

管理员和开发人员提出的另一个与 `init` 脚本相关的问题是，关于如何编写这些脚本有多个标准，并且有多种不同的工具与 `init` 脚本相关。基本上，每个主要的 Linux 发行版都有自己编写这些脚本的方式和自己的辅助函数库。我们来看看在 Slackware Linux 和 Debian/GNU Linux 上启动相同的 `My Service` 服务的 `init` 脚本。这也是编写 Shell 脚本的入门章节派上用场的地方。

在 Slackware 和 Debian 两种情况下，为了简洁起见，我们将删去一些原始内容，只保留最重要的部分。请不用担心，因为这两种发行版都提供了完全注释的示例脚本。

以下的 `init` 脚本将在 Slackware Linux 环境中工作。脚本以头部开始，我们在头部声明服务名称和一些重要的路径：

```
#!/bin/bash
#
# my_service       Startup script for the My Service daemon
#
# chkconfig: 2345 99 01
# description: My Service is a custom daemon that performs some important functions.
# Source function library.
. /etc/rc.d/init.d/functions
# Set the service name.
SERVICE_NAME=my_service
# Set the path to the service binary.
SERVICE_BINARY=/usr/local/bin/my_service
# Set the path to the service configuration file.
CONFIG_FILE=/etc/my_service/my_service.conf
# Set the user that the service should run as.
SERVICE_USER=my_service
# Set the process ID file.
PIDFILE=/var/run/my_service.pid
# Set the log file.
LOGFILE=/var/log/my_service.log
```

好的一点是，Slackware 提供的示例脚本有很好的注释。我们需要声明守护进程的二进制文件路径。我们还需要声明服务将以哪个用户和组身份运行，这实际上决定了它的文件系统权限。

接下来的部分定义了服务的所有重要操作：`start`、`stop` 和 `restart`。通常会有其他操作，但为了简洁我们将其省略：

```
start() {
  echo -n "Starting $SERVICE_NAME: "
  # Check if the service is already running.
  if [ -f $PIDFILE ]; then
    echo "already running"
    return 1
  fi
  # Start the service as the specified user and group.
  daemon --user $SERVICE_USER --group $SERVICE_GROUP $SERVICE_BINARY -c $CONFIG_FILE -l $LOGFILE -p $PIDFILE
  # Write a lock file to indicate that the service is running.
  touch $LOCKFILE
  echo "done"
}
```

`start` 函数利用 `pid` 文件检查服务是否正在运行。`pid` 文件是一个包含服务 PID 的文本文件。它提供关于服务主进程及其状态的信息。然而有一个问题，服务可能已经停止运行，但 `pid` 文件仍然存在。这会导致 `start` 函数无法实际启动服务。

在检查到 `pid` 文件不存在后，这意味着服务没有运行，一个名为 `daemon` 的特殊工具被用来以用户和组权限启动进程，并指向配置文件、日志文件和 `pid` 文件的位置。

该函数通过 `bash echo` 命令来传达它的操作，`echo` 命令会打印出给定的文本。如果是自动执行，`echo` 命令的输出将根据日志 `daemon` 配置记录到系统日志中：

```
stop() {
  echo -n "Stopping $SERVICE_NAME: "
  # Check if the service is running.
  if [ ! -f $PIDFILE ]; then
    echo "not running"
    return 1
  fi
  # Stop the service.
  killproc -p $PIDFILE $SERVICE_BINARY
  # Remove the lock file.
  rm -f $LOCKFILE
  # Remove the PID file.
  rm -f $PIDFILE
  echo "done"
}
```

同样，`stop` 函数使用 `pid` 文件检查服务是否正在运行。没有 `pid` 文件而服务仍在运行的可能性几乎为零。检查完成后，一个名为 `killproc` 的特殊命令被用来终止该进程。在函数的最后部分，脚本执行一些清理任务，清除 `pid` 文件和 `lock` 文件：

```
restart() {
  stop
  start
}
```

`restart` 函数非常简单。它重用已经定义的 `start` 和 `stop` 函数，准确执行它所说的：重启服务。如果服务配置需要重新加载二进制文件，这通常很有用：

```
case "$1" in
  start)
    start
    ;;
  stop)
    stop
    ;;
  restart)
    restart
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|uninstall}"
esac
```

脚本的最后部分评估我们想要执行的操作——启动、停止或重启服务——并调用相应的函数。如果我们要求它执行一个它不识别的操作，脚本会打印出使用说明。

然而，以下脚本是为 Debian Linux 环境设计的：

```
#!/bin/bash
#
# my_service       Startup script for the My Service daemon
#
# chkconfig: 2345 99 01
# description: My Service is a custom daemon that performs some important functions.
# Source function library.
. /lib/lsb/init-functions
# Set the service name.
SERVICE_NAME=my_service
# Set the path to the service binary.
SERVICE_BINARY=/usr/local/bin/my_service
# Set the path to the service configuration file.
CONFIG_FILE=/etc/my_service/my_service.conf
# Set the user that the service should run as.
SERVICE_USER=my_service
# Set the group that the service should run as.
SERVICE_GROUP=my_service
```

同样，脚本以一个头部部分开始，定义了稍后将在 `start` 和 `stop` 部分使用的路径。通常会有更多的行，但为了简洁，我们将它们省略了：

```
start() {
  log_daemon_msg "Starting $SERVICE_NAME"
  # Check if the service is already running.
  if [ -f $PIDFILE ]; then
    log_failure_msg "$SERVICE_NAME is already running"
    log_end_msg 1
    return 1
  fi
  # Start the service as the specified user and group.
  start-stop-daemon --start --background --user $SERVICE_USER --group $SERVICE_GROUP --make-pidfile --pidfile $PIDFILE --startas $SERVICE_BINARY -- -c $CONFIG_FILE -l $LOGFILE
  # Write a lock file to indicate that the service is running.
  touch $LOCKFILE
  log_end_msg 0
}
```

`start` 函数与 Slackware 版本中的类似。但你会注意到一些微妙的差异，如下所示：

+   这里使用了`start-stop-daemon`辅助函数来管理运行中的服务，而不是使用`daemon`和`killproc`。

+   这里使用了专门的日志记录函数，而不是简单的 echo：`log_daemon_msg`、`log_failure_msg` 和 `log_end_msg`。

+   `start-stop-daemon`函数接受特殊标志来确定操作（`start`和`stop`），并将程序从终端中分离，使其有效地成为系统服务（`--background`），如下面所示：

```
stop() {
  log_daemon_msg "Stopping $SERVICE_NAME"
  # Check if the service is running.
  if [ ! -f $PIDFILE ]; then
    log_failure_msg "$SERVICE_NAME is not running"
    log_end_msg 1
    return 1
  fi
  # Stop the service.
  start-stop-daemon --stop --pidfile $PIDFILE
  # Remove the lock file.
  rm -f $LOCKFILE
  # Remove the PID file.
  rm -f $PIDFILE
  log_end_msg 0
}
```

`stop`函数与 Slackware 的`stop`函数非常相似，具有与`start`函数相似的差异。

剩下的脚本包含`restart`函数和任务评估部分并不太有趣，因此我们已将其省略。

正如你可能还记得在关于`systemd`的章节中提到的，它解决了其中的一些问题。SysV 在现代系统中不常见，因此你通常需要处理的是`systemd`。

然而，还有另一个替代 SysV `init`的工具，叫做 Upstart。

# 关于 Upstart，一种替代方案，简要说明

Upstart 是一个基于事件的替代方案，替代传统的 SysV `init`系统，用于管理和控制系统上的服务和守护进程。Upstart 在 Ubuntu `6.10`及其后续版本中被引入，旨在提高启动时间，简化系统配置，并提供更灵活的系统服务管理方式。现在，它已在大多数 Linux 发行版中被`systemd`取代。

Upstart 用于管理系统的初始化过程，并启动、停止和监督任务与服务。它设计上比传统的`init`守护进程更灵活高效，并提供关于任务和服务状态的更多信息。

所有可以由`systemd`和/或`cron`管理的内容已成为行业标准，所以如果没有充分的理由使用它们，或者你已经有一个使用 Upstart 的系统，我们不建议你将其作为默认选择。

# 概述

在本章中，我们讨论了系统服务——即 Unix 和 Linux 世界中的守护进程——以及最常用的管理它们的软件。我们解释了这些是什么，如何检查它们的状态，以及如何控制它们。在下一章中，我们将深入探讨 Linux 网络。
