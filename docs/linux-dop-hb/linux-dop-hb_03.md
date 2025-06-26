# 3

# 中级 Linux

在本章中，我们将继续介绍 Linux shell。本话题非常广泛，足以写成一本独立的书。我们将回顾上一章的内容并介绍新的主题。

在本章中，我们将涵盖以下内容：

+   通配符

+   自动化重复任务

+   软件安装

+   用户管理

+   **安全外壳**（**SSH**）协议

# 技术要求

强烈建议你安装并准备好使用 Linux 系统。我们建议使用虚拟机或笔记本电脑，这样如果出了什么大问题，可以安全地重新安装。这将让你能够跟随本书的示例并完成我们给出的任何练习。

我们不会讲解安装过程。每个发行版可能会使用不同的安装程序，无论是图形界面还是文本界面（具体取决于发行版及所选变种）。你需要记下或记住你的用户名称（方便地称为**用户名**或**登录名**）和密码。如果你有物理访问权限，但不知道登录名或密码，甚至两者都不知道，还是有方法可以进入系统，但这些方法远超本书的范围。

本书的主要发行版是**Debian**。不过，你应该可以使用上一章中介绍的任何主要发行版，只要它不是 Alpine。

# 通配符

Shell 能为你做很多事情，帮助你简化工作。其中之一就是允许在输入命令参数时存在一定的*不确定性*。为此，shell 定义了几个特殊字符，将它们作为符号使用，而不是字面输入。这些符号称为**全局模式**，或称为**通配符**。在通配符中使用的字符有时被称为**通配符**（wildcards）。

不要将通配符与**正则表达式**（**regexps**）混淆。虽然通配符本身非常强大，但它们无法与正则表达式相比。另一方面，bash 在执行模式匹配时并不会对正则表达式进行求值。

下表描述了 Shell 通配符及其含义。我们将通过几个示例来解释它们的确切含义：

| **通配符** | **含义** |
| --- | --- |
| `*` | 匹配任意数量的字符（包括零个字符） |
| `?` | 精确匹配一个字符 |
| `[...]` | 匹配括号内集合中的任意一个字符 |

表 3.1 – Shell 通配符

上表可能对你来说不够清晰，下面我们将通过一些示例来进一步说明：

| **示例** | **含义** |
| --- | --- |
| `*` | 这将匹配任意长度的任何字符串。 |
| `*``test*` | 这将匹配任何包含*test*的字符串：`test.txt`、`good_test.txt`、`test_run`，甚至是简单的`test`（记住，它也可以匹配空字符串）。 |
| `test*txt` | 这将匹配任何名称以*test*开头，*txt*结尾的文件，例如 `test.txt`、`testtxt`、`test_file.txt` 等。 |
| `test?` | 这将匹配任何一个包含*test*并加上一个字符的情况：`test1`、`test2`、`testa`、`test`，等等。 |
| `test.[ch]` | 这将匹配两种情况中的一种：`test.c` 或 `test.h`，其他情况不匹配。 |
| `*.[``ab]` | 这将匹配任何以点号结束并且后面跟着 *a* 或 *b* 的字符串。 |
| `?[``tf]` | 这将匹配任何一种字符，后跟 *t* 或 *f*。 |

表 3.2 – Shell 通配符 – 示例

通配符的真正威力在于你开始编写一些更复杂的命令字符串（所谓的单行命令）或脚本时才能显现。

一些简单命令在与通配符结合使用时会达到全新的层次，像是 `find`、`grep` 和 `rm`。在以下示例中，我使用通配符删除所有以任意字符开始、接着是 test 和一个点，然后是 log 和任意字符结尾的文件。因此，`weirdtest.log`、`anothertest.log1` 和 `test.log.3` 文件会被匹配，但 `testlog` 和 `important_test.out` 则不会。首先，让我们列出所有包含 *test* 字样的文件：

```
admin@myhome:~$ ls -ahl *test*
-rw-r--r-- 1 admin admin 0 Sep 17 20:36 importat_test.out
-rw-r--r-- 1 admin admin 0 Sep 17 20:36 runner_test.lo
-rw-r--r-- 1 admin admin 0 Sep 17 20:36 runner_test.log
-rw-r--r-- 1 admin admin 0 Sep 17 20:36 test.log
-rw-r--r-- 1 admin admin 0 Sep 17 20:35 test.log.1
-rw-r--r-- 1 admin admin 0 Sep 17 20:35 test.log.2
-rw-r--r-- 1 admin admin 0 Sep 17 20:35 test.log.3
-rw-r--r-- 1 admin admin 0 Sep 17 20:35 test.log.4
-rw-r--r-- 1 admin admin 0 Sep 17 20:35 test.log.5
-rw-r--r-- 1 admin admin 0 Sep 17 20:35 test.log.6
-rw-r--r-- 1 admin admin 0 Sep 17 20:35 test.log.7
```

你会注意到，我使用了通配符（`*`）来实现我的目标。现在，是时候进行实际的删除操作了：

```
admin@myhome:~$ rm *test.log*
admin@myhome:~$ ls -ahl *test*
-rw-r--r-- 1 admin admin 0 Sep 17 20:36 importat_test.out
-rw-r--r-- 1 admin admin 0 Sep 17 20:36 runner_test.lo
```

正如这里演示的那样，它已经成功执行了。你还会注意到，一个正确执行的命令不会打印任何消息。

在本节中，我们解释了如何使用通配符（globs）——这些特殊字符允许我们以一定的不确定性匹配系统中的名称。在下一节中，我们将介绍自动化重复任务的机制。

# 自动化重复任务

有时你可能希望使某些任务变得重复。你可能会编写一个脚本来备份数据库、检查用户的主目录权限，或者将当前操作系统的性能指标转储到文件中。现代的 Linux 发行版提供了两种设置这些任务的方法。还有第三种方法允许你在延迟的时间一次性运行任务（`at` 命令），但在这里我们关注的是重复性任务。

## Cron 作业

**Cron** 是一种传统的方式，用来定期执行需要在特定时间间隔内运行的任务。通常，它们应该被 **systemd 定时器** 取代，但许多软件仍通过 cron 作业提供可重复性，而 Alpine Linux 在最小化发行版的情况下不会包含这一特性。

Cron 作业本质上是定期执行的命令。这些命令及其触发定时器被定义在位于 `/etc/` 目录中的配置文件中。不同的发行版会有不同数量的文件和目录。所有的发行版都会包含一个 `/etc/crontab` 文件。这个文件通常包含对其中字段的解释以及几个实际命令，可以作为模板使用。在以下的代码块中，我粘贴了来自默认 `/etc/crontab` 文件的解释：

```
# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
```

通常，有两种方法来设置 cron 任务。其一是将一个运行命令的脚本放置在四个目录中的一个：`/etc/cron.hourly`、`/etc/cron.daily`、`/etc/cron.weekly` 或 `/etc/cron.monthly`。这些目录应该足以满足日常操作。`/etc/crontab` 文件指定了这些任务何时运行。

第二个选项是使用 `crontab` 命令。`crontab` 命令允许给定的用户在其 crontab 文件中创建一个条目。然而，系统范围的 crontab 文件（位于 `/etc/` 目录中）和每个用户的 crontab 文件之间是有区别的。用户的 cron 文件不指定用户字段。所有条目都以用户的权限运行。我们来看看这些区别：

+   `crontab -l` 列出用户在其 crontab 文件中定义的所有 cron 任务。

+   `crontab -e` 让用户编辑 crontab 文件，添加、删除或修改任务。

## Systemd 定时器

我们在这里不会详细介绍 **systemd 定时器**，只会简要提到它们为何可能是更好的选择。

**Systemd 定时器** 单元是 **cron 守护进程** 任务的更新版本。它们可以完成 cron 的所有功能。然而，它们提供了一些额外的能力：

+   您可以指定任务在系统启动后某个时间运行。

+   您可以指定任务必须在其他任务运行后的某个间隔内执行。

+   对定时器单元的依赖甚至可以是服务单元——即普通的系统服务任务。

+   精度更高。Cron 任务的精度通常只能到每分钟一次，而 Systemd 定时器的触发精度可以达到秒级。

在本节中，我们介绍了 cron 任务，并提到 systemd 定时器是自动化定期任务的两个最重要的方式。下一节中，我们将介绍软件管理。

# 软件安装

根据您选择的发行版和安装方式，您的系统可能缺少一些日常工作所需的软件。也可能有一天，您需要安装一款默认未安装的软件。

Linux 发行版在一些方面领先于其他操作系统，后来其他操作系统才开始效仿。Linux 操作系统上安装软件的常见方式是运行一个合适的命令，这个命令会下载一个二进制文件，正确地将其放置到系统中，必要时添加一些配置，并使其对用户可用。今天，这可能听起来并不算革命性。毕竟，我们生活在一个拥有 Apple App Store、Google Play 和 Microsoft Apps 的世界中。但是，在过去，当 Windows 和 macOS 用户需要在互联网上找到合适的软件安装程序时，Linux 用户只需一个命令就能安装大多数软件。

这一点在 DevOps 力图建立的自动化环境中至关重要，原因有多个：

+   可安装的软件（以软件包形式分发）存储在发行版团队维护的仓库中。这意味着你不需要知道软件在互联网上的位置；你只需要知道它的包名，并确保它在仓库中。

+   我们将在这里介绍的包管理标准（`rpm` 和 `deb`）*了解*依赖关系。这意味着，如果你尝试安装的软件依赖于另一个尚未安装的软件，它会自动拉取并安装。

+   我们将在这里介绍的发行版都有安全团队。它们与包维护者合作，确保修复已知的漏洞。然而，这并不意味着他们会主动研究这些包中的漏洞。

+   仓库会在互联网上进行镜像。这意味着，即使其中一个仓库出现故障（下线、被 DDoS 攻击或其他原因），你仍然可以从全球各地访问它的镜像副本。这对于商业仓库未必适用。

+   如果你愿意，可以在本地局域网中创建一个本地仓库镜像。这将为你提供最快的下载速度，但代价是大量的硬盘空间。软件包仓库可能非常庞大。

软件的数量和版本在许多方面取决于发行版：

+   **政策分发必须分发具有不同许可证类型的软件**：有些发行版会禁止任何不完全开放和免费的软件（根据开源倡议定义）。其他发行版则会让用户选择是否添加包含更严格许可的软件的仓库。

+   **维护者数量和维护模式**：显而易见，发行版能做的工作量取决于他们拥有多少人力资源。团队越小，能够打包和维护的软件就越少。部分工作是自动化的，但很多工作始终需要人工完成。由于 Debian 是一个非商业发行版，它完全依赖志愿者的工作。而 Ubuntu 和 Fedora 则有商业支持，部分团队成员甚至是由其中的公司雇佣的：Canonical 和 Red Hat。**红帽企业 Linux**（**RHEL**）完全由红帽员工构建和维护。

+   **你决定使用的仓库类型**：一些软件厂商将它们的软件包分发在独立的仓库中，你可以将其添加到你的配置中，像使用普通的发行版仓库一样使用它们。不过，如果你这样做了，有些事情需要注意：

    +   **第三方仓库中的软件不属于发行版的质量管理工作**：这完全取决于仓库维护者——在这种情况下，就是软件供应商。这也包括安全修复。

    +   **第三方仓库中的软件可能不会与核心发行版仓库同时更新**：这意味着有时软件所需的包版本与发行版提供的包版本之间会存在冲突。而且，随着你向服务器添加更多第三方仓库，发生冲突的概率也会增加。

## Debian 和 Ubuntu

Debian 发行版及其衍生版 Ubuntu 使用 `DEB` 包格式。它是专为 Debian 创建的。我们这里不会深入探讨其历史，只会根据需要触及一些技术细节。

直接操作包文件的命令是 `dpkg`。它用于安装、移除、配置，并且重要的是，构建 `.deb` 包。它只能安装存在于文件系统上的包，并且不理解远程仓库。让我们看看 `dpkg` 的一些可能操作：

+   `dpkg -i package_file.deb`：安装包文件。该命令会经过多个阶段，之后安装该软件。

+   `dpkg –unpack package_file.deb`：解包意味着它会将所有重要文件放入相应的位置，但不会配置包。

+   `dpkg –configure package`：请注意，这里需要的是包名，而不是包文件名。如果因为某些原因包已解包但未配置，你可以使用 `-a` 或 `–pending` 标志来处理它们。

+   `dpkg -r package`：此操作会移除软件，但不会删除配置文件及其可能包含的数据。如果你打算将来重新安装该软件，这可能会很有用。

+   `dpkg -p package`：此操作会清除该包并移除一切：软件、数据、配置文件和缓存。一切。

在以下示例中，我们正在从一个物理下载到系统的包中安装 nano 编辑器，这个包可能是通过点击网页上的下载按钮获得的。请注意，这并不是一种常见的做法：

```
root@myhome:~# dpkg -i nano_5.4-2+deb11u1_amd64.deb
(Reading database ... 35904 files and directories currently installed.)
Preparing to unpack nano_5.4-2+deb11u1_amd64.deb ...
Unpacking nano (5.4-2+deb11u1) over (5.4-2+deb11u1) ...
Setting up nano (5.4-2+deb11u1) ...
Processing triggers for man-db (2.9.4-2) ...
```

通常，你会需要使用 `apt` 工具集来安装和移除软件：

+   `apt-cache search NAME` 将搜索包含给定字符串的软件包。在以下示例中，我正在寻找一个包含 `vim` 字符串的软件包（vim 是几个流行的命令行文本编辑器之一）。为了简洁，我已将输出进行了缩短：

```
root@myhome:~# apt-cache search vim
acr - autoconf like tool
alot - Text mode MUA using notmuch mail
[...]
vim - Vi IMproved - enhanced vi editor
[...]
```

+   `apt-get install NAME` 将安装你指定名称的包。你可以在一行中安装多个软件包。在以下示例中，我正在安装一个 C 编译器、一个 C++编译器和一个 Go 语言套件。请注意，输出中还包含了一些为了确保我的软件正常工作而需要的依赖包，并且它们将会被安装以提供该功能：

```
root@myhome:~# apt-get install gcc g++ golang
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
gcc is already the newest version (4:10.2.1-1).
The following additional packages will be installed:
  bzip2 g++-10 golang-1.15 golang-1.15-doc golang-1.15-go golang-1.15-src golang-doc golang-go golang-src libdpkg-perl libfile-fcntllock-perl libgdbm-compat4 liblocale-gettext-perl libperl5.32 libstdc++-10-dev perl perl-modules-5.32
  pkg-config
Suggested packages:
  bzip2-doc g++-multilib g++-10-multilib gcc-10-doc bzr | brz git mercurial subversion debian-keyring gnupg patch bzr libstdc++-10-doc perl-doc libterm-readline-gnu-perl | libterm-readline-perl-perl libtap-harness-archive-perl dpkg-dev
The following NEW packages will be installed:
  bzip2 g++ g++-10 golang golang-1.15 golang-1.15-doc golang-1.15-go golang-1.15-src golang-doc golang-go golang-src libdpkg-perl libfile-fcntllock-perl libgdbm-compat4 liblocale-gettext-perl libperl5.32 libstdc++-10-dev perl
  perl-modules-5.32 pkg-config
0 upgraded, 20 newly installed, 0 to remove and 13 not upgraded.
Need to get 83.9 MB of archives.
After this operation, 460 MB of additional disk space will be used.
Do you want to continue? [Y/n]
```

安装程序在这里停止并等待我们的输入。默认操作是接受所有附加包并继续安装。通过输入 `n` 或 `N` 并按下 *Enter*，我们可以停止这个过程。安装操作的 `-y` 开关将跳过这个问题，并自动继续到下一步：

+   `apt-get update` 将刷新包数据库，获取新的可用包和新版本。

+   `apt-get upgrade` 将所有已安装的包升级到数据库中列出的最新版本。

+   `apt-get remove NAME` 将删除给定名称的包。在以下示例中，我们正在卸载 C++ 编译器：

```
root@myhome:~# apt-get remove g++
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages were automatically installed and are no longer required:
  g++-10 libstdc++-10-dev
Use 'apt autoremove' to remove them.
The following packages will be REMOVED:
  g++
0 upgraded, 0 newly installed, 1 to remove and 13 not upgraded.
After this operation, 15.4 kB disk space will be freed.
Do you want to continue? [Y/n]
(Reading database ... 50861 files and directories currently installed.)
Removing g++ (4:10.2.1-1) ...
```

## CentOS、RHEL 和 Fedora

另一类流行的发行版使用 `RPM` 包格式。与包交互的基本工具是 `rpm`。使用这种格式的主要发行版是由 Red Hat 公司制作的 RHEL。包的文件扩展名始终是 `.rpm`。

它们使用 `dnf` 命令来管理包。还有 `yum` 命令，它是 RHEL 发行版的原始包管理器（并且，扩展到 Fedora 和 CentOS 发行版），但已被移除。`dnf` 是 `yum` 的下一代重写，具有许多改进，使其更强大、更现代：

+   `dnf install package_name` 将安装给定名称的包及其依赖项。

+   `dnf remove package_name` 将删除包。

+   `dnf update` 将所有包更新到包数据库中的最新版本。你可以指定包名，之后 `yum` 将更新该包。

+   `dnf search NAME` 将搜索包含 `NAME` 字符串的包名。

+   `dnf check-update` 将刷新包数据库。

让我们来看看另一款广泛使用的 Linux 发行版，特别是作为 Docker 镜像的基础——Alpine Linux。

## Alpine Linux

Alpine Linux 尤其受到主要使用 Docker 和 Kubernetes 的工程师喜爱。正如该发行版的主页所宣称，**小巧、简单、安全**。它没有像基于 Debian 或 Red Hat 的发行版中那样的花里胡哨，但输出的 Docker 镜像非常小，而且由于它专注于安全，你可以假设如果你已经将所有包更新到最新版本，那么没有重大安全漏洞。

Alpine Linux 的主要缺点（也可以说是优点，视角不同而定）是它使用 `musl` 库而非广泛使用的 `libc` 库进行编译，尽管它确实使用了 `libc`，因此在安装任何 Python 依赖之前，你需要执行额外的步骤，确保已安装编译时依赖。

与包交互的命令是 `3.16`）和 edge，这是一个滚动更新版本（它始终拥有最新版本的包）。

此外，还有三个仓库可以用来安装包：`main`、`community` 和 `testing`。

你将会在主仓库中找到官方支持的包；所有经过测试的包都放在社区仓库中，而测试版则用于测试，这意味着可能存在一些损坏、过时或带有安全漏洞的包。

### 搜索包

在搜索或安装任何包之前，建议先下载最新的包缓存。你可以通过调用`apk`的`update`命令来实现：

```
root@myhome:~# apk update
fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/community/x86_64/APKINDEX.tar.gz
v3.16.2-376-g3ff8974e73 [https://dl-cdn.alpinelinux.org/alpine/v3.16/main]
v3.16.2-379-g3c25b38306 [https://dl-cdn.alpinelinux.org/alpine/v3.16/community]
OK: 17037 distinct packages available
```

如果你在构建一个用于以后使用的 Docker 镜像，建议在构建过程的最后一步使用`apk cache`的`clean`命令来删除此缓存。

有时，在我们创建新的 Docker 镜像时，不知道包的确切名称。此时，最简单的方式是使用 Web 界面：[`pkgs.alpinelinux.org/packages`](https://pkgs.alpinelinux.org/packages)。

使用命令行界面（CLI），你将能够搜索部分库名和二进制文件名，尽管你可以使用`so:`前缀来指定你正在搜索的是库。其他有用的前缀包括`cmd:`用于命令，`pc:`用于`pkg-config`文件：

```
root@myhome:~# apk search libproc
libproc-3.3.17-r1
libksysguard-5.24.5-r0
process-cpp-3.0.1-r3
samba-dc-libs-4.15.7-r0
procps-dev-3.3.17-r1
root@myhome:~# apk search vim
neovim-doc-0.7.0-r0
gvim-8.2.5000-r0
vim-tutor-8.2.5000-r0
faenza-icon-theme-vim-1.3.1-r6
notmuch-vim-0.36-r0
kmymoney-5.1.2-r3
faenza-icon-theme-gvim-1.3.1-r6
meson-vim-0.62.1-r0
runvimtests-1.30-r1
graphviz-3.0.0-r0
neovim-0.7.0-r0
py3-pynvim-0.4.3-r3
nftables-vim-0_git20200629-r1
vim-doc-8.2.5000-r0
vim-editorconfig-0.8.0-r0
apparmor-vim-3.0.4-r0
geany-plugins-vimode-1.38-r1
vimdiff-8.2.5000-r0
vimb-3.6.0-r0
neovim-lang-0.7.0-r0
u-boot-tools-2022.04-r1
fzf-neovim-0.30.0-r7
nginx-vim-1.22.1-r0
msmtp-vim-1.8.20-r0
protobuf-vim-3.18.1-r3
vimb-doc-3.6.0-r0
icinga2-vim-2.13.3-r1
fzf-vim-0.30.0-r7
vim-sleuth-1.2-r0
gst-plugins-base-1.20.3-r0
mercurial-vim-6.1.1-r0
skim-vim-plugin-0.9.4-r5
root@myhome:~# apk search -e vim
gvim-8.2.5000-r0
root@myhome:~# apk search -e so:libproc*
libproc-3.3.17-r1
libksysguard-5.24.5-r0
process-cpp-3.0.1-r3
samba-dc-libs-4.15.7-r0
```

### 安装、升级和卸载包

你可以使用`add`（安装）、`del`（卸载）和`upgrade`命令对包执行基本操作。在安装过程中，你也可以使用在搜索操作中可用的特殊前缀，但建议使用包的准确名称。请注意，当向系统添加新包时，`apk`将选择该包的最新版本：

```
root@myhome:~# apk search -e postgresql14
postgresql14-14.5-r0
root@myhome:~# apk add postgresql14
(1/17) Installing postgresql-common (1.1-r2)
Executing postgresql-common-1.1-r2.pre-install
(2/17) Installing libpq (14.5-r0)
(3/17) Installing ncurses-terminfo-base (6.3_p20220521-r0)
(4/17) Installing ncurses-libs (6.3_p20220521-r0)
(5/17) Installing readline (8.1.2-r0)
(6/17) Installing postgresql14-client (14.5-r0)
(7/17) Installing tzdata (2022c-r0)
(8/17) Installing icu-data-en (71.1-r2)
Executing icu-data-en-71.1-r2.post-install
*
* If you need ICU with non-English locales and legacy charset support, install
* package icu-data-full.
*
(9/17) Installing libgcc (11.2.1_git20220219-r2)
(10/17) Installing libstdc++ (11.2.1_git20220219-r2)
(11/17) Installing icu-libs (71.1-r2)
(12/17) Installing gdbm (1.23-r0)
(13/17) Installing libsasl (2.1.28-r0)
(14/17) Installing libldap (2.6.3-r1)
(15/17) Installing xz-libs (5.2.5-r1)
(16/17) Installing libxml2 (2.9.14-r2)
(17/17) Installing postgresql14 (14.5-r0)
Executing postgresql14-14.5-r0.post-install
*
* If you want to use JIT in PostgreSQL, install postgresql14-jit or
* postgresql-jit (if you didn't install specific major version of postgresql).
*
Executing busybox-1.35.0-r17.trigger
Executing postgresql-common-1.1-r2.trigger
* Setting postgresql14 as the default version
OK: 38 MiB in 31 packages
```

你还可以选择安装特定版本的包，而不是最新版本。不幸的是，无法从同一仓库安装旧版本的包，因为每当部署新版本时，旧版本会被移除。然而，你可以从其他仓库安装包的旧版本：

```
root@myhome:~# apk add bash=5.1.16-r0 --repository=http://dl-cdn.alpinelinux.org/alpine/v3.15/main
(1/4) Installing ncurses-terminfo-base (6.3_p20220521-r0)
(2/4) Installing ncurses-libs (6.3_p20220521-r0)
(3/4) Installing readline (8.1.2-r0)
(4/4) Installing bash (5.1.16-r0)
Executing bash-5.1.16-r0.post-install
Executing busybox-1.35.0-r17.trigger
OK: 8 MiB in 18 packages
```

你还可以使用以下命令安装你事先准备好的自定义包：

```
root@myhome:~# apk add --allow-untrusted your-package.apk
```

要升级系统中所有可用的包，你可以简单地调用`apk upgrade`命令。但是，如果你只想升级某个特定的包，你需要在升级选项后添加该包的名称。记得在此之前刷新包缓存：

```
root@myhome:~# apk update
fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/main/x86_64/APKINDEX.tar.gz
fetch https://dl-cdn.alpinelinux.org/alpine/v3.16/community/x86_64/APKINDEX.tar.gz
v3.16.2-376-g3ff8974e73 [https://dl-cdn.alpinelinux.org/alpine/v3.16/main]
v3.16.2-383-gcca4d0a396 [https://dl-cdn.alpinelinux.org/alpine/v3.16/community]
OK: 17037 distinct packages available
root@myhome:~# apk upgrade
(1/2) Upgrading alpine-baselayout-data (3.2.0-r22 -> 3.2.0-r23)
(2/2) Upgrading alpine-baselayout (3.2.0-r22 -> 3.2.0-r23)
Executing alpine-baselayout-3.2.0-r23.pre-upgrade
Executing alpine-baselayout-3.2.0-r23.post-upgrade
Executing busybox-1.35.0-r17.trigger
OK: 8 MiB in 18 packages
```

你可以通过不带任何其他选项地调用`apk`来查找所有可能的操作。最有用的操作之一是`apk info`命令。它会打印出关于包或仓库的信息（以下输出已被缩略）：

```
root@myhome:~# apk info --all bash
bash-5.1.16-r2 description:
The GNU Bourne Again shell
bash-5.1.16-r2 webpage:
https://www.gnu.org/software/bash/bash.xhtml
bash-5.1.16-r2 installed size:
1308 KiB
bash-5.1.16-r2 depends on:
/bin/sh
so:libc.musl-x86_64.so.1
so:libreadline.so.8
bash-5.1.16-r2 provides:
cmd:bash=5.1.16-r2.
```

在这一部分，我们介绍了包管理工具——在 Linux 发行版中管理软件的标准方式。在接下来的部分中，我们将介绍用户账户管理。

# 用户管理

在 Linux 系统中，用户由三组文件定义：

+   `/etc/passwd`：此文件包含关于用户的信息 —— 即用户的名称、**唯一的数字 ID**（**UID**）、用户所属的主要组的 GID、主目录的路径以及用户登录时加载的 shell。一个典型的条目如下所示：

```
   admin:x:1000:1000:Debian:/home/admin:/bin/bash
```

每一行描述一个用户。字段通过冒号分隔。第二个字段在非常特殊的情况下才会包含除了 `x` 以外的其他内容。在这里，`x` 表示密码存储在 `/etc/shadow` 文件中。原因是，`/etc/passwd` 文件的权限必须稍微宽松一些，以便登录过程能够正常工作。`/etc/shadow` 只能被 root 用户和 root 组读取，并且只能由 root 用户写入：

```
    root@myhome:~# ls -ahl /etc/passwd
    -rw-r--r-- 1 root root 1.6K Aug 22 18:38 /etc/passwd
    root@myhome:~# ls -ahl /etc/shadow
    -rw-r----- 1 root shadow 839 Aug 22 18:38 /etc/shadow
```

+   `/etc/shadow -`：此文件包含加密的密码。正如前面提到的，为了安全起见，只有 root 用户可以读取和写入此文件。

+   `/etc/group -`：此文件包含有关用户所属组的信息。组就是那些已被归类在一起的账户，以便可以管理它们的权限。

你永远不应该手动修改这些文件，特别是 `/etc/shadow` 文件。正确的方法是使用 `passwd` 命令来更改它的内容。我们建议你阅读手册页获取更多信息。

有三个命令参与用户修改，另外有三个命令用于管理组：`useradd`、`userdel`、`usermod`、`groupadd`、`groupdel` 和 `groupmod`。

## 添加用户

`useradd` 向系统添加一个用户账户。不同的选项修改该命令的行为。调用 `useradd` 命令的最常见版本会添加一个用户，创建其主目录，并指定默认的 shell：

```
root@myhome:~# useradd -md /home/auser -s /bin/bash auser
```

`-m` 告诉命令创建主目录，`-d`（这里它和 `-m` 一起传递了一个减号）告诉命令主目录应该是什么（注意是绝对路径），`-s` 指定默认的 shell。可以指定更多的参数，再次建议你查看手册页获取更多详细信息。

## 修改用户

`usermod` 修改现有的用户账户。它可以用来更改用户的组成员、主目录、锁定账户等。这里有一个有趣的选项是 `-p` 标志，它允许你非交互式地应用新密码。这在自动化中非常有用，当我们可能希望通过脚本或工具更新用户密码，而不是通过命令行。然而，这也存在安全风险：在命令执行期间，系统中的任何人都可以列出正在运行的进程及其参数，并查看密码条目。虽然这不会自动危及密码，因为密码必须通过 `crypt (3)` 函数加密后提供，但如果攻击者获得了加密的密码版本，他们可以运行密码破解程序进行攻击，最终通过暴力破解得到明文密码：

```
root@myhome:~# usermod -a -G admins auser
```

上述命令将把 `auser` 用户添加到名为 `admins` 的组中。`-a` 选项表示该用户将被添加到附加组（不会从其他已有的组中删除）。

## 删除用户

`userdel` 命令用于从系统中删除用户。它只能从系统文件中删除用户条目，并保持主目录不变，或者删除带有主目录的用户：

```
root@myhome:~# userdel -r auser
```

上述命令将删除用户及其主目录和所有文件。注意，如果用户仍然登录，它将不会被删除。

## 管理组

与管理用户类似，你可以在 Linux 系统中添加、删除和修改组。为此有等效的命令：

+   `groupadd`：在系统中创建一个组。组可以用于将用户归类，并指定其执行、目录或文件访问权限。

+   `groupdel`：从系统中删除该组。

+   `groupmod`：更改组定义。

你还可以使用 `who` 命令查看当前登录系统的用户：

```
admin@myhome:~$ who
ubuntu   tty1         2023-09-21 11:58
ubuntu   pts/0        2023-09-22 07:54 (1.2.3.4)
ubuntu   pts/1        2023-09-22 09:25 (5.6.7.8)
trochej  pts/2        2023-09-22 11:21 (10.11.12.13)
```

你还可以使用 `id` 命令查看当前登录用户的 UID 和 GID：

```
admin@myhome:~$ id
uid=1000(ubuntu) gid=1000(ubuntu) 
groups=1000(ubuntu),4(adm),20(dialout), 24(cdrom),25(floppy),27(sudo),29(audio), 30(dip),44(video),46(plugdev),119(netdev),120(lxd),123(docker)
```

执行此命令时不带任何选项，它将显示你的用户 ID 和用户所在的所有组。或者，你可以提供一个名称，查看该用户的 UID 或 GID：

```
admin@myhome:~$ id dnsmasq
uid=116(dnsmasq) gid=65534(nogroup) groups=65534(nogroup)
```

要查看主组的 ID 和 UID，你可以分别使用 `-u` 和 `-g` 选项：

```
admin@myhome:~$ id -u
1000
admin@myhome:~$ id -g
1000
```

在本节中，我们介绍了用于管理 Linux 系统中用户帐户和组的命令。下一节将解释如何使用 SSH 安全地连接到远程系统。

# 安全外壳（SSH）协议

在 DevOps 世界中，几乎没有什么东西在你的笔记本电脑或 PC 上本地运行。要连接到远程系统，SSH 协议是公认的黄金标准。SSH 于 1995 年开发，作为一种安全的加密远程 shell 访问工具，用来取代像 **telnet** 或 **rsh** 这样的明文工具。其主要原因是，在分布式网络中，监听通信过于容易，任何明文传输的内容都容易被截获。这包括诸如登录信息等重要数据。

Linux 世界中最常用的 SSH 服务器（和客户端）是 **OpenSSH** ([`www.openssh.com/`](https://www.openssh.com/))。截至撰写时，其他仍在维护的开源服务器包括 **lsh** ([`www.lysator.liu.se/~nisse/lsh/`](http://www.lysator.liu.se/~nisse/lsh/))、**wolfSSH** ([`www.wolfssl.com/products/wolfssh/`](https://www.wolfssl.com/products/wolfssh/)) 和 **Dropbear** ([`matt.ucc.asn.au/dropbear/dropbear.xhtml`](https://matt.ucc.asn.au/dropbear/dropbear.xhtml))。

SSH 主要用于登录远程机器执行命令。但它也能够传输文件（`22`）。

## 配置 OpenSSH

安装 OpenSSH 服务器后，你的发行版将在 `/etc/ssh/sshd_config` 文件中放置一个基本配置。最基本的配置如下所示：

```
AuthorizedKeysFile .ssh/authorized_keys
AllowTcpForwarding no
GatewayPorts no
X11Forwarding no
Subsystem sftp /usr/lib/ssh/sftp-server
```

在继续选项之前，让我们先研究一下它们：

+   `AuthorizedKeysFIle` 告诉我们的服务器在用户目录中查找存储所有可以用来作为指定用户连接到该机器的公钥的文件。因此，如果你将公钥放在 `AlphaOne` 的主目录中，即 `/home/AlphaOne/.ssh/authorized_keys`，你将能够使用对应的私钥作为该用户进行连接（关于密钥的更多内容将在 *创建和管理 SSH* *密钥* 小节中讨论）。

+   `AllowTCPForwarding` 将启用或禁用所有用户转发 TCP 端口的能力。**端口转发**用于访问那些无法直接在互联网上访问的远程机器，但你可以访问另一个可以连接的机器。这意味着你正在使用 SSH 主机作为所谓的跳跃主机来连接到私有网络，类似于使用 VPN。

+   `GatewayPorts` 是另一个直接与端口转发功能相关的选项。通过允许 `GatewayPorts`，你不仅可以将转发的端口暴露给你的机器，还可以暴露给你连接的网络中的其他主机。由于安全原因，不建议将此选项设置为 `yes`；你可能会不小心将私有网络暴露给你所在的网络，例如在咖啡馆中。

+   `X11Forwarding` 有一个非常特定的使用场景。通常情况下，你不希望在服务器上运行完整的图形界面，但如果你有这个需求，启用此选项后，你将能够登录到远程机器并启动远程图形应用程序，这些应用程序将像在本地主机上运行一样显示。

+   `Subsystem` 使你能够通过附加功能扩展 OpenSSH 服务器，例如在此案例中使用 SFTP。

在前面的命令块中没有指定的一个非常重要的选项是 `PermitRootLogin`，默认情况下设置为 `prohibit-password`。这意味着你将能够以 root 用户身份登录，但只有在需要使用公钥和私钥对进行身份验证时才可以登录。我们建议将此选项设置为 `no`，并且仅通过 `sudo` 命令允许 root 用户访问。

就是这些。当然，你可以添加更多高级配置，例如使用 `man` `sshd_config` 命令。

以同样的方式，你可以了解如何配置 SSH 客户端——也就是说，运行 `man ssh_config`。

以下是一些非常有用的客户端选项：

```
# Show keys ascii graphics
VisualHostKey yes
# Offer only one Identity at a time
Host *
  ForwardAgent yes
  IdentitiesOnly yes
  IdentityFile ~/.ssh/mydefaultkey
# Automatically add all used keys to agent
AddKeysToAgent yes
```

`VisualHostKey` 设置为 `yes` 时，将显示服务器公钥的 ASCII 图形。你将连接的每个服务器都有一个唯一的密钥，因此会有独特的图形。它很有用，因为作为人类，我们非常擅长识别模式，因此如果你连接到 `1.2.35.2` 服务器，但打算进入不同的系统，你很可能通过看到与预期不同的图形来察觉到不对劲。

这里有一个示例：

```
root@myhome:~# ssh user@hosts
Host key fingerprint is SHA256:EppY0d4YBvXQhCk0f98n1tyM7fBoyRMQl2o3ife1pY
+--[ED25519 256]--+
|    .oB++o       |
|     +o*o ..     |
|    +..o*.+ .    |
|    .o +.= . +   |
|    ..o.S.= = .  |
|      .ooooE. .  |
|        .o.o     |
|                 |
|                 |
+----[SHA256]-----+
```

主机选项允许你为一个或多个服务器设置特定的选项。在这个示例中，我们启用了 SSH 代理转发，并禁用了所有服务器的密码登录。此外，我们还设置了一个默认的私钥，用于连接任何服务器。这将引出我们对 SSH 密钥和加密算法的讨论。

最后一个选项，`AddKeysToAgent`，意味着每当你使用（并解锁）一个密钥时，它也会被添加到 SSH 代理中，以供将来使用。这样，你就不需要在连接时指定密钥，也不必在每次连接尝试时解锁密钥。

## 创建和管理 SSH 密钥

SSH 由三个组件组成：传输层（**传输控制协议** (**TCP**)/**互联网协议** (**IP**))，用户身份验证层，以及一个连接层，可以有效地是多个独立传输数据的连接。

就不同的身份验证选项而言，你有一种基本的密码身份验证方式，证明它不够安全。还有公钥身份验证，我们将在这里讨论。剩下的两个是 `keyboard-interactive` 和 **通用安全服务应用程序接口** (**GSSAPI**)。

公钥身份验证要求我们生成一个密钥，它将有两个对应的部分：私钥和公钥。你将把公钥放在服务器的 `authorized_keys` 文件中；私钥则用于身份验证。

在编写本书时，RSA 密钥是与 SSH 一起使用的标准。它是安全的，但建议使用更大的密钥，长度为 4,096 位，但 3,072 位（默认）也被认为足够。更大的密钥意味着更慢的连接速度。

当前，更好的选择是使用 `ed25519` 类型的密钥，它具有固定长度。

此外，所有密钥都可以用密码进行保护。

以下代码展示了如何生成这两种密钥类型：

```
root@myhome:~# ssh-keygen -b 4096 -o -a 500 -t rsa
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Created directory '/root/.ssh'.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa
Your public key has been saved in /root/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:eMu9AMUjpbQQ7DJIl4MB2UnpfiUqwZJi+/e9dsKWXZg root@614bbd02e559
The key's randomart image is:
+---[RSA 4096]----+
|o=+++.. .        |
|.o++.o =         |
|o+... + +        |
|*o+ o .+ .       |
|+o.+ oo S   o    |
|..o .  + o E .   |
| ...    = + .    |
|   . .  .B +     |
|    . ..oo=      |
+----[SHA256]-----+
root@myhome:~# ssh-keygen -o -a 100 -t ed25519
Generating public/private ed25519 key pair.
Enter file in which to save the key (/root/.ssh/id_ed25519):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_ed25519
Your public key has been saved in /root/.ssh/id_ed25519.pub
The key fingerprint is:
SHA256:EOCjeyRRIz+hZy6Pbtcnjan2lucJ2Mdih/rFc/ZnthI
root@614bbd02e559
The key's randomart image is:
+--[ED25519 256]--+
|  . =..          |
|   * o .         |
|  o B .          |
|   * o .         |
|  + o   S        |
|   B o +    E    |
|  o +.*=B o  .   |
| ...ooB*+= .. +  |
| ..oo=o=o   .=.. |
+----[SHA256]-----+
```

现在，要将这个新创建的密钥放到服务器上，你需要手动将其复制到 `authorized_keys` 文件中，或者使用 `ssh-copy-id` 命令，如果你已经有其他访问手段（如密码身份验证），它会为你完成此操作：

```
root@myhome:~# ssh-copy-id -i ~/.ssh/id_ed25519 user@remote-host
```

你只能通过使用密钥执行下次登录到此服务器的操作。

到目前为止，你应该已经很好地理解了 SSH 的工作原理以及如何使用它的最常用功能。你现在知道如何创建密钥以及如何在远程系统上保存它们。

# 总结

本章结束了我们对日常工作中所需的基本 Linux 操作的介绍。虽然这并没有全面解释你管理 Linux 系统所需了解的所有内容，但足以帮助你入门，并帮助你管理系统。在下一章中，我们将从头开始讲解如何编写 shell 脚本，并指导你学习基本以及更高级的主题。

# 练习

通过以下练习来测试你对本章内容的掌握：

1.  在 Debian/Ubuntu 中，安装`vim`软件包。

1.  创建一个定时任务，每周六上午 10:00 生成一个名为`/tmp/cronfile`的文件。

1.  创建一个名为`admins`的组，并将一个现有用户添加到该组中。
