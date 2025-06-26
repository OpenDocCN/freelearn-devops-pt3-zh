# 11

# 使用 Ansible 实现配置即代码

本章我们将介绍 **配置管理**（**CM**）、**配置即代码**（**CaC**）以及我们选择的工具：Ansible。

我们将覆盖以下主题：

+   CM 系统与 CaC

+   Ansible

+   Ansible Galaxy

+   处理机密信息

+   Ansible Tower 和替代方案

+   高级话题

# 技术要求

对于本章内容，你需要一个可以通过 `ssh` 访问的 Linux 系统。如果你的主要操作系统是 Windows，你将需要另外一台 Linux 系统来充当控制节点。到目前为止，Ansible 项目尚不支持 Windows 作为控制节点。

# CM 系统与 CaC

设置和维护一个非业余服务器的系统（甚至是业余服务器，可能也需要这样做）是一个严峻的挑战：如何确保系统按照预期正确安装和配置？当你需要安装一台与现有配置完全相同的服务器时，如何确保这一点？过去，一种做法是在安装过程完成后记录当前的配置。这将是一份描述硬件、操作系统、已安装软件、创建的用户以及应用的配置的文档。任何想要重建该配置的人，都需要按照文档中的步骤操作以达到描述的配置。

接下来的合乎逻辑的步骤是编写 shell 脚本，达到与手动过程相同的目标，但有一个额外的改进：这些脚本——只要编写得当、经过测试并且维护良好——不需要人工操作，除非，可能，在初始系统安装时需要人工干预。但一个正确设置的环境将自动处理这一点。

然而，这些脚本也存在一些缺陷或不足。其一是你需要在脚本中考虑未完成执行的情况。这可能由于各种原因发生，导致系统处于部分配置状态。再次执行脚本时，所有配置操作将从头开始，有时会导致意外结果。应对未完成的执行的一种方式是将每个配置操作都包装在检查中，查看该操作是否之前已执行过。这将导致配置脚本变得更大，最终演变成一个配置函数和检查函数的库。

开发和维护这样一个工具的任务可能非常艰巨，并且可能需要一个完整的团队。但最终的结果可能值得付出这些努力。

编写和维护描述系统期望状态的文档，乍一看可能比前面提到的自动化方法更简单且更具吸引力。脚本无法从未完成的执行中恢复过来。它所能做的最好的事情就是通知系统管理员失败，记录错误并优雅地停止。手动配置允许系统管理员绕过程序中的任何障碍和不足，并实时编辑文档以反映当前状态。

尽管如此，一个经过良好开发和测试的脚本最终还是更好。让我们列举出一些原因：

+   如果脚本执行没有出错，保证会执行正确的操作。一次又一次地证明，在 IT 领域，最容易出错的元素是人类。

+   如果脚本提前退出，更新它以满足新需求的行为与更新文档的过程完全相同。

+   众所周知，人类在维护文档方面很差。编程的圣杯是自文档化代码，使注释变得不再必要，从而消除了注释与代码不同步的风险。

+   脚本可以在多个系统上同时执行，扩展性非常好，几乎是无限的。而人类只能一次配置一个系统，且犯错的风险较小。

+   以脚本或程序形式保存的配置得益于典型的编程技巧，例如自动化测试、演练和静态分析。更重要的是，将代码保存在版本库中让我们能够轻松追踪更改历史，并与问题跟踪工具集成。

+   代码是明确的，而书面语言却不能如此。文档可能留有解释的空间，但脚本不会。

+   自动化配置让你可以转向其他更有趣的任务，留给计算机去做它们最擅长的事情——执行重复性且枯燥的任务。

编程和系统管理的世界倾向于将小项目转变为更大的项目，并且有一个充满活力的开发者和用户社区。CM（配置管理）系统的诞生只是时间问题。它们将开发和管理负责配置操作的代码部分的负担从你肩上移走。CM 系统的开发者编写代码、测试并认为它稳定。剩下的工作就是编写配置文件或指令，告诉系统应该做什么。这些系统大多数能够覆盖最流行的平台，使你能够只描述一次配置，并在商业 Unix 系统（如 AIX 或 Solaris）与 Linux 或 Windows 上获得相同的预期结果。

这些系统的配置文件可以轻松地存储在 Git 等版本控制系统中。它们易于理解，这使得同事可以轻松进行审查。它们可以通过自动化工具检查语法错误，并使你能够专注于整个工作中的最重要部分：配置。

这种将配置保留为一组脚本或其他数据，而不是手动遵循的过程的方法，被称为 CaC。

随着需要管理的系统数量不断增加，以及对快速高效配置的需求不断扩大，CaC（配置即代码）方法变得越来越重要。在 DevOps 的世界里，通常需要每天设置数十个或数百个系统：包括开发人员、测试人员和生产系统，以应对服务需求的新水平。手动管理将是一项不可能完成的任务。良好实现的 CaC 允许通过点击按钮来完成这项任务。因此，开发人员和测试人员可以自行部署系统，而不需要打扰系统运维人员。你的任务将是开发、维护和测试配置数据。

如果编程世界里有一件事是可以确定的，那就是永远不会只有一个解决方案。CM 工具也不例外。

Ansible 的替代方案包括**SaltStack**、**Chef**、**Puppet** 和 **CFEngine**，后者是最古老的，它的首次发布是在 1993 年，因此截至本书撰写时已存在 30 年。通常，这些解决方案通过强制配置的方法（拉取或推送）和描述系统状态的方法（命令式或声明式）有所不同。

**命令式**（Imperative）方法意味着我们通过命令描述服务器的状态，让工具执行。命令式编程侧重于一步步描述给定工具的操作方式。

**声明式**（Declarative）方法意味着我们关注 CaC 工具应该完成什么，而不是指定它应该如何实现结果的所有细节。

## SaltStack

**SaltStack** 是一个开源的 CM 工具，允许在大规模的 IT 基础设施中进行管理。它使得日常任务的自动化成为可能，如软件包安装、用户管理和软件配置，并设计用于跨多种操作系统和平台工作。SaltStack 由 Thomas Hatch 于 2011 年创立。SaltStack 的第一个版本 0.8.0 也发布于 2011 年。

SaltStack 通过利用主从架构工作，其中一个中央的 salt-master 与运行在远程机器上的 salt-minion 进行通信，执行命令并管理配置。它采用拉取方式来强制执行配置：minion 从主服务器拉取最新的清单。

一旦 minion 被安装并配置好，我们可以使用 SaltStack 来管理服务器的配置。以下是一个安装并配置 `nginx` 的示例 `nginx.sls` 文件：

```
nginx:
  pkg.installed
/etc/nginx/sites-available/yourdomain.tld.conf:
  file.managed:
    - source: salt://nginx/yourdomain.tld.conf
    - user: root
    - group: root
    - mode: 644
```

在这个示例中，第一行指定应该在目标服务器上安装`nginx`包。接下来的两行定义了一个假设网站`example.com`的配置文件，该配置文件将被复制到`/etc/nginx/sites-available/yourdomain.tld.conf`。

要将此状态文件应用到服务器，我们需要在 SaltStack 命令行界面中使用`state.apply`命令，并指定状态文件的名称作为参数：

```
admin@myhome:~$ salt 'webserver' state.apply nginx
```

这将把`nginx.sls`文件中的指令发送到运行在 Web 服务器上的 salt-minion，salt-minion 将执行必要的步骤以确保`nginx`正确安装和配置。

## Chef

**Chef**是一个强大的开源配置管理（CM）工具，允许用户自动化基础设施、应用程序和服务的部署与管理。它最初由 Opscode 于 2009 年发布，后被 Chef Software Inc.收购。从那时起，Chef 被 IT 专业人员和 DevOps 团队广泛采用，以简化工作流程并减少管理复杂系统所需的时间和精力。

Chef 通过在一组代码文件中定义基础设施的期望状态来工作，这些代码文件称为食谱（cookbooks）。**食谱（cookbook）**是描述如何安装、配置和管理特定软件或服务的一组指令。每个食谱包含一系列资源，资源是预构建的模块，可以执行特定任务，例如安装软件包或配置文件。Chef 使用声明式的配置管理（CM）方法，意味着用户定义系统的期望状态，而 Chef 负责处理如何实现该状态的细节。

要使用 Chef 安装`nginx`，你首先需要创建一个包含安装`nginx`食谱的 cookbook。这个食谱将使用`package`资源来安装`nginx`包，并使用`service`资源确保`nginx`服务正在运行。根据需求，你还可以使用其他资源，例如`file`、`directory`或`template`，来配置`nginx`的设置。

一旦创建了食谱（cookbook），你需要将其上传到 Chef 服务器，Chef 服务器作为食谱及其相关元数据的中央仓库。然后，你可以使用 Chef 的命令行工具`knife`来配置目标系统使用该食谱。这涉及将系统与 Chef 环境关联，Chef 环境定义了应应用于系统的食谱及其版本。接下来，你可以使用`chef-client`命令在目标系统上运行 Chef 客户端，该客户端将下载并应用必要的食谱和配置，以使系统达到所需状态。

这是安装和配置`nginx`的示例：

```
# Install Nginx package
package 'nginx'
# Configure Nginx service
service 'nginx' do
  action [:enable, :start]
end
# Configure Nginx site
template '/etc/nginx/sites-available/yourdomain.tld.conf' do
  source 'nginx-site.erb'
  owner 'root'
  group 'root'
  mode '0644'
  notifies :restart, 'service[nginx]'
end
```

这个食谱使用了三个资源，如下所示：

+   `package`：使用系统默认的软件包管理器安装`nginx`包。

+   `service`：这将启动并启用`nginx`服务，使其在启动时自动启动并保持运行。

+   `template`：通过从模板文件生成配置文件，为`nginx`创建一个配置文件。模板文件（`nginx-site.erb`）位于食谱的`templates`目录中。`notifies`属性告诉 Chef 在配置文件更改时重新启动`nginx`服务。

一旦你在食谱中创建了这个食谱，你可以使用`knife`命令将食谱上传到 Chef 服务器。然后，你可以使用`chef-client`命令将食谱应用于目标系统，按照食谱安装和配置`nginx`。

## Puppet

`2.0`。

Puppet 通过在声明性语言中定义基础设施资源的期望状态来工作，这种语言被称为 Puppet 语言。管理员可以在 Puppet 代码中定义服务器、应用程序和其他基础设施组件的配置，然后将其一致地应用于多个系统。

Puppet 由一个主服务器和多个代理节点组成。主服务器充当 Puppet 代码和配置数据的中央存储库，而代理节点则执行 Puppet 代码并将期望的状态应用到系统中。

Puppet 有一个强大的模块生态系统，这些模块是预先编写的 Puppet 代码，可以用于配置常见的基础设施资源。这些模块可在**Puppet Forge**中找到，Puppet Forge 是一个公开的 Puppet 代码存储库。

下面是一个示例 Puppet 清单，它安装`nginx`并创建一个类似我们在 SaltStack 和 Chef 中所做的配置文件：

```
# Install Nginx
package { 'nginx':
  ensure => installed,
}
# Define the configuration template for the domain
file { '/etc/nginx/sites-available/yourdomain.tld.conf':
  content => template('nginx/yourdomain.tld.conf.erb'),
  owner   => 'root',
  group   => 'root',
  mode    => '0644',
  notify  => Service['nginx'],
}
# Enable the site by creating a symbolic link from sites-available to sites-enabled
file { '/etc/nginx/sites-enabled/yourdomain.tld.conf':
  ensure  => 'link',
  target  => '/etc/nginx/sites-available/yourdomain.tld.conf',
  require => File['/etc/nginx/sites-available/yourdomain.tld.conf'],
}
# Restart Nginx when the configuration changes
service { 'nginx':
  ensure     => running,
  enable     => true,
  subscribe  => File['/etc/nginx/sites-enabled/yourdomain.tld.conf'],
}
```

一旦你创建了清单并将其放置在 Puppet 服务器上，它将被安装在服务器上的 Puppet 代理拾取并执行。通信方式与 SaltStack 相同，通过 TLS 协议确保安全，使用与互联网上的 HTTPS 服务器相同的机制。

代理节点运行 Puppet 代理进程，通过 TCP 端口`8140`连接到主服务器。代理向服务器发送**证书签名请求**（**CSR**），管理员必须批准此请求。一旦 CSR 被批准，代理将获得对服务器 Puppet 配置的访问权限。

当代理运行时，它向主服务器发送请求以获取其配置。服务器响应并提供一个资源目录，说明应应用于节点的内容。该目录是根据存储在服务器上的 Puppet 代码和清单，以及配置的任何外部数据源或层次结构生成的。

然后，代理将目录应用到节点，这包括对节点配置进行必要的更改，以确保其与目录中定义的期望状态匹配。这可能包括安装软件包、更新配置文件或启动或停止服务。

代理在应用完目录后将报告发送回服务器，这些报告可用于监控和审计。服务器还可以使用这些信息检测通过 Puppet 未做的节点配置更改，并在必要时采取纠正措施。

## CFEngine

**CFEngine** 是一个开源的配置管理系统，允许用户自动化部署、配置和维护 IT 系统。它由 Mark Burgess 于 1993 年创立，之后成为管理大规模 IT 基础设施的流行工具。CFEngine 以其强大而灵活的语言来描述系统配置和执行策略而闻名，是处理复杂 IT 环境的理想选择。

CFEngine 的第一次发布是在 1994 年，使其成为现存最古老的配置管理工具之一。此后，CFEngine 经历了许多更新和改进，以适应不断变化的 IT 环境和新兴技术。CFEngine 最新版本 3.18 包括改进的加密功能、增强的监控能力和对云基础设施的更好支持。

多年来，CFEngine 因其强大的功能、易用性和强大的社区支持而广受欢迎。今天，许多组织仍在使用它，并且它仍在积极开发，因此它是一个安全的选择，可以用来管理你的服务器配置。

这里将展示一个 CFEngine 配置示例。由于需要，它只是一个片段，而非完整配置：

```
##############################################################
# cf.main - for master infrastructure server
##################################################################
###
# BEGIN cf.main
###
control:
   access    = ( root )        # Only root should run this
   site      = ( main )
   domain    = ( example.com )
   sysadm    = ( admin@example.com )
   repository = ( /var/spool/cfengine )
   netmask   = ( 255.255.255.0 )
   timezone  = ( CET )
#################################################################
files:
  Prepare::
      /etc/motd              m=0644 r=0 o=root act=touch
```

在本节中，我们已经解释了什么是 CaC，以及为什么它是系统管理员工具箱中的一个重要工具。我们简要描述了你可以使用的最流行的工具。在下一节中，我们将介绍我们首选的工具——Ansible。

# Ansible

在本节中，我们将向你介绍 **Ansible**，这是我们在 CaC 方面的首选工具。

Ansible 是一款用于管理系统和设备配置的工具。它是用 Python 编写的，源代码可以自由下载和修改（在 Apache License `2.0` 许可范围内）。名字“Ansible”来源于 Ursula K. Le Guin 的小说 *Rocannon’s World*，指的是一种无论距离多远都能实现瞬时通信的设备。

这里列出了 Ansible 一些有趣的特点：

+   **模块化**：Ansible 不是一个单体工具。它是一个核心程序，每个它知道如何执行的任务都被写作一个独立的模块——如果你愿意的话，可以把它看作是一个库。由于从一开始就是这种设计，它产生了一个干净的 API，任何人都可以使用这个 API 来编写自己的模块。

+   **幂等性**：无论执行多少次配置，结果始终保持不变。这是 Ansible 最重要和最基本的特点之一。你不必知道哪些操作已经执行过。当你扩展配置并再次运行工具时，它的工作是找出系统的状态并仅应用新的操作。

+   **无代理**：Ansible 不会在配置系统上安装代理。这并不意味着它完全不需要任何东西。为了执行 Ansible 脚本，目标系统需要能够连接到它的一些手段（通常是运行的 SSH 服务器）以及安装 Python 语言。这个范式带来了几个优势，包括以下几点：

    +   Ansible 与通信协议无关。它使用 SSH，但并不实现该协议，而是将细节留给操作系统、SSH 服务器和客户端。一个优势是，你可以根据需要自由地将一个 SSH 解决方案替换为另一个，而你的 Ansible 剧本应该照常工作。此外，Ansible 也不关心 SSH 配置的安全性，这让开发人员可以集中精力处理系统的真正问题：配置你的系统。

    +   Ansible 项目无需为被管理的节点开发和维护独立的程序。这不仅减轻了开发人员的不必要负担，还限制了安全漏洞被发现并用于攻击目标机器的可能性。

    +   在使用代理的解决方案中，如果由于任何原因，代理程序停止工作，就无法将新的配置传递到系统中。SSH 服务器通常使用广泛，故障的概率几乎可以忽略不计。

    +   使用 SSH 作为通信协议可以降低防火墙阻止 CM 系统通信端口的风险。

+   `nginx` 已安装，正确的配置项应该如下所示：

    ```
    - name: install nginx
    ```

    ```
      package:
    ```

    ```
        name: nginx
    ```

    ```
        state: present
    ```

与 Ansible 交互的主要方式是通过编写使用一种叫做 YAML 的特殊语法的配置文件。**YAML** 是一种专为配置文件设计的语法，基于 Python 的格式化方式。缩进在 YAML 文件中起着重要作用。YAML 的主页提供了完整的备忘单卡片（[`yaml.org/refcard.xhtml`](https://yaml.org/refcard.xhtml)），其中涵盖了该语法。然而，这里将呈现最重要的部分，因为我们在本章中将主要与这些文件打交道：

+   它们是明文文件。这意味着可以使用最简单的编辑器（如 Notepad、Vim、Emacs，或任何你喜欢的文本编辑工具）查看和编辑它们。

+   缩进用于表示范围。不允许使用制表符进行缩进，通常使用空格来进行缩进。

+   新文档以三个短横线（`-`）开头。一个文件可以包含多个文档。

+   注释与 Python 中相同，以井号（`#`）开始，直到行尾。注释必须被空白字符包围，否则它将被视为文本中的字面井号（`#`）。

+   文本（字符串）可以不加引号、单引号（`'`）或双引号（`"`）括起来。

+   当指定一个列表时，每个成员由一个连字符（`-`）表示。每个项目将占一行。如果需要单行表示，可以将列表项用方括号（`[]`）括起来，并用逗号（`,`）分隔。

+   关联数组通过键值对表示，每个键和值之间用冒号和空格分隔。如果必须在一行中呈现，数组将用大括号（`{}`）括起来，键值对之间用逗号（`,`）分隔。

如果前面的规则现在还不太清楚，别担心。我们将编写正确的 Ansible 配置文件，随着过程的推进，所有问题都会明朗。

Ansible 将机器分为两组：**控制节点**是存储配置指令的计算机，它将连接到目标机器并进行配置。控制节点可以有多个。目标机器称为**库存**。Ansible 将在列出的库存中的计算机上运行操作。库存通常以**初始化**（**INI**）格式编写，这种格式简单易懂，如下所示：

+   注释以分号（`;`）开头。

+   各个部分有名称，并且名称用方括号（`[]`）括起来。

+   配置指令以键值对的形式存储，每一对独占一行，键和值之间用等号（`=`）分隔。

我们稍后会看到一个库存文件的例子。然而，也有可能存在所谓的动态库存，它是通过脚本或每次运行 Ansible 时由系统自动生成的。

我们将要交互的主要配置文件称为剧本。**剧本**是 Ansible 的入口点，工具将从这里开始执行。剧本可以包括其他文件（这通常会这么做）。

目标主机可以根据自定义标准进行分组：操作系统、组织中的角色、物理位置——任何需要的内容。这样的分组称为**角色**。

一项要执行的单一操作称为**任务**。

## 使用 Ansible 的基础知识

首先要做的是安装 Ansible。这在所有主要的 Linux 发行版上都非常简单，macOS 也同样如此。我们建议您根据所选择的操作系统找到相应的解决方案。

对于基于 Debian 的发行版，以下命令应该足够：

```
$ sudo apt-get install ansible
```

对于 Fedora Linux 发行版，您可以运行以下命令：

```
$ sudo dnf install ansible
```

然而，要安装最新版本的 Ansible，我们建议使用 Python 虚拟环境及其`pip`工具，如下所示：

```
$ python3 -m venv venv
```

在前面的代码中，我们使用`venv python3`模块激活了虚拟环境。它将创建一个特殊的`venv`目录，其中包含所有重要的文件和库，允许我们设置 Python 虚拟环境。接下来，我们有以下内容：

```
$ source venv/bin/activate
```

在前面的代码行中，我们读取了一个特殊的文件，该文件通过配置 shell 来设置环境。接下来，我们有以下内容：

```
$ pip install -U pip
Requirement already satisfied: pip in ./venv/lib/python3.11/site-packages (22.3.1)
Collecting pip
  Using cached pip-23.0.1-py3-none-any.whl (2.1 MB)
Installing collected packages: pip
  Attempting uninstall: pip
    Found existing installation: pip 22.3.1
    Uninstalling pip-22.3.1:
      Successfully uninstalled pip-22.3.1
Successfully installed pip-23.0.1
```

在上面的代码中，我们已经升级了 `pip`，一个 Python 包管理器。在下一步中，我们将实际安装 Ansible，也使用 `pip`。为了简洁起见，输出将被缩短：

```
$ pip install ansible
Collecting ansible
  Using cached ansible-7.3.0-py3-none-any.whl (43.1 MB)
Collecting ansible-core~=2.14.3
Installing collected packages: resolvelib, PyYAML, pycparser, packaging, MarkupSafe, jinja2, cffi, cryptography, ansible-core, ansible
Successfully installed MarkupSafe-2.1.2 PyYAML-6.0 ansible-7.3.0 ansible-core-2.14.3 cffi-1.15.1 cryptography-39.0.2 jinja2-3.1.2 packaging-23.0 pycparser-2.21 resolvelib-0.8.1
```

正如你在上面的代码中看到的，`pip` 告诉我们 Ansible 及其依赖项已成功安装。

这两种安装 Ansible 的方式都将下载并安装运行 Ansible 所需的所有软件包。

你可以运行的最简单命令是所谓的临时 `ping`。它如此基础，以至于它成为了 Ansible 在教程和书籍中最常见的入门用法之一。我们也不打算偏离它。以下命令尝试连接到指定主机，并打印连接结果：

```
$ ansible all -m ping -i inventory
hostone | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

在上面的命令中，我们告诉 Ansible 运行针对清单中的所有主机，使用 `ping` 模块（`-m ping`），并使用名为 `inventory` 的清单文件（`-i inventory`）。如果你没有指定清单文件，Ansible 将尝试使用 `/etc/ansible/hosts`，但通常这样做不是个好主意。

我们将在稍后的段落中深入探讨清单文件。

一个 `ping` 模块（我们也可以理解为一个 `ping` 命令）。然而，它与操作系统中的 `ping` 命令不同，后者发送一个特别制作的网络数据包来判断主机是否在线。而 Ansible 的 `ping` 命令则会尝试登录，以确定凭证是否正确。

默认情况下，Ansible 使用 SSH 协议和一对公私密钥。在正常操作中，你通常不想使用基于密码的身份验证，而是会选择基于密钥的身份验证。

Ansible 在执行结果中非常擅长提供自解释的信息。上面的输出告诉我们，Ansible 成功连接到清单中的所有节点（这里只有一个），`python3` 已安装，并且没有做任何更改。

清单文件是一个简单但强大的工具，用于列出、分组和为被管理的节点提供变量。我们的示例清单文件如下所示：

```
[www]
hostone ansible_host=192.168.1.2  ansible_ssh_private_key_file=~/.ssh/hostone.pem  ansible_user=admin
```

上面的代码有两行。第一行声明了一个名为 `www` 的节点组。第二行声明了一个名为 `hostone` 的节点。由于这是一个无法通过 DNS 解析的名称，我们通过 `ansible_host` 变量声明了它的 IP 地址。然后，我们指定了正确的 `ssh` 密钥文件，并声明了登录时应该使用的用户名（`admin`）。在这个文件中，我们还可以定义更多内容。有关编写清单文件的详细信息，可以参考 Ansible 项目的文档 ([`docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.xhtml`](https://docs.ansible.com/ansible/latest/inventory_guide/intro_inventory.xhtml))。

我们无法涵盖 Ansible 的所有方面，也没有深入探讨我们在这里将使用的大部分功能。然而，Packt 出版社有很多很好的 Ansible 书籍，如果你想深化你的知识，可能会有兴趣选择其中的一些书籍——例如 James Freeman 和 Jesse Keating 的*《Mastering Ansible，第四版》*。

## 任务（Tasks）

任务是 Ansible 配置的核心。它们正是那些要在托管节点上执行的任务。任务包含在剧本中。它们可以直接放入剧本中（写入剧本文件），也可以间接地通过角色包含。

有一种特殊类型的任务叫做**处理（handles）**。这是一个只有在另一个任务通知时才会执行的任务。

## 角色（Roles）

Ansible 的文档将**角色（roles）**定义为“*可重用 Ansible 内容（任务、处理程序、变量、插件、模板和文件）的一种有限分发，用于剧本内部使用。*”就我们而言，我们可以将它们视为一种将剧本部分进行分组的机制。一个角色可以是我们要执行剧本的主机类型：Web 服务器、数据库服务器、Kubernetes 节点等等。通过将剧本分解为角色，我们可以更容易、更高效地管理单独的任务。更不用说，这些包含文件的内容也变得更加可读，因为我们限制了其中的任务数量，并按其功能进行分组。

## 剧本和剧本集

**剧本（Plays）**是在任务执行的上下文中。虽然剧本是一个有些短暂的概念，**剧本集（playbooks）**是其物理表现：定义剧本的 YAML 文件。

让我们看一个剧本示例。以下剧本将在托管节点上安装`nginx`和`php`软件包：

```
---
- name: Install nginx  and php
  hosts: www
  become: yes
  tasks:
  - name: Install nginx
    package:
      name: nginx
      state: present
  - name: Install php
    package:
      name: php8
      state: present
  - name: Start nginx
    service:
      name: nginx
      state: started
```

第一行（三个破折号）标志着一个新的 YAML 文档的开始。下一行是整个剧本的名称。剧本的名称应简短但具有描述性。它们最终会出现在日志和调试信息中。

接下来，我们通知 Ansible 该剧本应该在`www`组中的节点上执行。我们还告诉 Ansible 在执行命令时使用`sudo`。这是必需的，因为我们在本指南中覆盖的所有发行版都需要 root 权限来安装和删除软件包。

然后，我们开始`tasks`部分。每个任务都会被命名，并指定我们将要使用的模块（命令），同时命令会附带选项和参数。如你所见，缩进声明了范围。如果你熟悉 Python 编程语言，这对你来说应该是直观的。

在运行这个剧本之前，让我们使用一个非常有用的工具，`ansible-lint`：

```
$ ansible-lint install.yaml
WARNING: PATH altered to expand ~ in it. Read https://stackoverflow.com/a/44704799/99834 and correct your system configuration.
WARNING  Listing 9 violation(s) that are fatal
yaml[trailing-spaces]: Trailing spaces
install.yaml:1
yaml[truthy]: Truthy value should be one of [false, true]
install.yaml:4
fqcn[action-core]: Use FQCN for builtin module actions (package).
install.yaml:6 Use `ansible.builtin.package` or `ansible.legacy.package` instead.
[...]
yaml[empty-lines]: Too many blank lines (1 > 0)
install.yaml:18
Read documentation for instructions on how to ignore specific rule violations.
                   Rule Violation Summary
 count tag                   profile    rule associated tags
     1 yaml[empty-lines]     basic      formatting, yaml
     1 yaml[indentation]     basic      formatting, yaml
     3 yaml[trailing-spaces] basic      formatting, yaml
     1 yaml[truthy]          basic      formatting, yaml
     3 fqcn[action-core]     production formatting
Failed after min profile: 9 failure(s), 0 warning(s) on 1 files.
```

为了简洁，我已经删除了一部分输出，但你可以看到工具打印了关于 YAML 语法和 Ansible 最佳实践违规的信息。失败是会停止剧本执行的错误类型。警告仅仅是警告：剧本将继续执行，但存在一些违反最佳实践的错误。让我们按如下方式修正我们的剧本：

```
---
- name: Install nginx  and php
  hosts: www
  become: true
  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: nginx
        state: present
    - name: Install php
      ansible.builtin.package:
        name: php8
        state: present
    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
```

现在我们可以使用`ansible-playbook`命令运行 playbook：

```
$ ansible-playbook -i inventory install.yaml
PLAY [Install nginx  and php] ********************************************************************
TASK [Gathering Facts] ********************************************************************
ok: [hostone]
TASK [Install nginx] ********************************************************************
changed: [hostone]
TASK [Install php] ********************************************************************
fatal: [hostone]: FAILED! => {"changed": false, "msg": "No package matching 'php8' is available"}
PLAY RECAP ********************************************************************
hostone     : ok=2    changed=1   reachable=0    failed=1    skipped=0    rescued=0    ignored=0
```

在前面的输出中，我们指示`ansible-playbook`命令使用名为`inventory`的清单文件，并运行名为`install.yaml`的 playbook。输出应该是自解释的：我们会看到我们运行的 play 的名称。接下来，我们会看到 Ansible 将尝试执行操作的受管节点列表。然后，我们看到任务和任务成功或失败的节点列表。`nginx`任务在`hostone`上成功执行。然而，安装`php`失败了。Ansible 给出了确切的原因：我们的受管节点上没有`php8`包。有时，解决方案很明显，但有时则需要一些挖掘。经过与我们的发行版检查后，我们发现实际可用的`php`包是`php7.4`。在快速更正有问题的行后，我们再次运行 playbook，如下所示：

```
$ ansible-playbook -i inventory install.yaml
PLAY [Install nginx  and php] ********************************************************************
TASK [Gathering Facts] ********************************************************************
ok: [hostone]
TASK [Install nginx] ********************************************************************
ok: [hostone]
TASK [Install php] ********************************************************************
changed: [hostone]
TASK [Start nginx] ********************************************************************
ok: [hostone]
PLAY RECAP ********************************************************************
hostone                    : ok=4    changed=1    unreachable=0   failed=0    skipped=0    rescued=0    ignored=0
```

注意输出中的变化。首先，Ansible 告诉我们`hostone`上的`nginx`是正常的。这意味着 Ansible 能够确认包已安装，因此没有采取任何行动。接着，它告诉我们`hostone`服务器上的`php7.4`安装成功（`changed: [hostone]`）。

前面的 playbook 很短，但我们希望它能够展示这个工具的有用性。Playbook 是以线性方式执行的，从上到下。

我们的 playbook 存在一个问题。虽然它只安装了两个包，但如果需要安装数十个甚至上百个包，你可能会担心可维护性和可读性。为每个包单独创建任务是麻烦的。幸好，有解决方案。你可以为给定任务创建一个项目列表，任务将对该列表中的每个项目执行——类似于一个循环。让我们如下编辑 playbook 的`tasks`部分：

```
  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: '{{ item }}'
        state: present
      with_items:
        - nginx
        - php7.4
       - gcc
       - g++
    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
```

我们添加了两个包，以展示一个更完整的运行过程。注意没有为`php7.4`单独创建任务。

在运行这个 play 之前，最好先用`ansible-lint`检查一下。下面是如何操作：

```
$ ansible-lint install.yaml
Passed with production profile: 0 failure(s), 0 warning(s) on 1 files.
```

现在，在`ansible-lint`给出绿色信号后，让我们运行这个 playbook：

```
$ ansible-playbook -i inventory install.yaml
PLAY [Install nginx  and php] ********************************************************************
TASK [Gathering Facts] ********************************************************************
ok: [hostone]
TASK [Install nginx] ********************************************************************
ok: [hostone] => (item=nginx)
ok: [hostone] => (item=php7.4)
changed: [hostone] => (item=gcc)
changed: [hostone] => (item=g++)
TASK [Start nginx] ********************************************************************
ok: [hostone]
PLAY RECAP ********************************************************************
hostone            : ok=3    changed=1    unreachable=0   failed=0    skipped=0    rescued=0    ignored=0
```

假设我们想在`nginx`中安装页面配置。Ansible 可以复制文件。我们可以设置它来复制`nginx`虚拟服务器配置，但我们只希望在服务设置时重启`nginx`一次。

我们可以通过`notify`和`handlers`来实现这一点。我会先粘贴整个 playbook：

```
---
- name: Install nginx  and php
  hosts: www
  become: true
  tasks:
    - name: Install nginx
      ansible.builtin.package:
        name: '{{ item }}'
        state: present
      with_items:
        - nginx
        - php7.4
        - gcc
        - g++
      notify:
        - Start nginx
    - name: Copy service configuration
      ansible.builtin.copy:
        src: "files/service.cfg"
        dest: "/etc/nginx/sites-available/service.cfg"
        owner: root
        group: root
        mode: '0640'
    - name: Enable site
      ansible.builtin.file:
        src: "/etc/nginx/sites-available/service.cfg"
        dest: "/etc/nginx/sites-enabled/default"
        state: link
      notify:
        - Restart nginx
  handlers:
    - name: Start nginx
      ansible.builtin.service:
        name: nginx
        state: started
    - name: Restart nginx
      ansible.builtin.service:
        name: nginx
        state: restarted
```

注意到与复制文件相关的整个新部分。我们还创建了一个`nginx`。为了简洁起见，我们简化了它，只是为了展示一个原理。

执行此 play 后，得到以下输出：

```
$ ansible-playbook -i inventory install.yaml
PLAY [Install nginx  and php] ********************************************************************
TASK [Gathering Facts] ********************************************************************
ok: [hostone]
TASK [Install nginx] ********************************************************************
ok: [hostone] => (item=nginx)
ok: [hostone] => (item=php7.4)
ok: [hostone] => (item=gcc)
ok: [hostone] => (item=g++)
TASK [Copy service configuration] ********************************************************************
ok: [hostone]
TASK [Enable site] ********************************************************************
ok: [hostone]
PLAY RECAP ********************************************************************
hostone                     ok=4    changed=0    unreachable=0   failed=0    skipped=0    rescued=0    ignored=0
```

一旦你开始在实际生产环境中使用 Ansible，就会明显发现，即使使用我们之前展示的*循环*技巧，playbook 也会迅速增长并变得难以管理。幸运的是，有一种方法可以将 playbook 分解成更小的文件。

其中一种方法是将任务划分为不同的角色。如前所述，角色是一个抽象的概念，因此最简单的方式是将其视为一个组。你根据对你来说相关的标准将任务分组。虽然标准完全由你决定，但通常情况下，角色会根据给定的受管节点执行的功能类型来命名。一个受管节点可以执行多个功能，但仍然最好将这些功能分开到不同的角色中。因此，HTTP 服务器将是一个角色，数据库服务器将是另一个角色，文件服务器将是另一个角色。

Ansible 中的角色有一个预定的目录结构。定义了八个标准目录，尽管你只需要创建其中一个。

以下代码摘自 Ansible 文档（[`docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.xhtml`](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_reuse_roles.xhtml)）：

```
roles/
    common/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        templates/        #  <-- files for use with the template resource
            ntp.conf.j2   #  <------- templates end in .j2
        files/            #
            bar.txt       #  <-- files for use with the copy resource
            foo.sh        #  <-- script files for use with the script resource
        vars/             #
            main.yml      #  <-- variables associated with this role
        defaults/         #
            main.yml      #  <-- default lower priority variables for this role
        meta/             #
            main.yml      #  <-- role dependencies
        library/          # roles can also include custom modules
        module_utils/     # roles can also include custom module_utils
        lookup_plugins/   # or other types of plugins, like lookup in this case
```

`roles` 目录与 playbook 位于同一级别。你可能会看到的最常见子目录是 `tasks`、`templates`、`files`、`vars` 和 `handlers`。在每个子目录中，Ansible 会查找一个 `main.yml`、`main.yaml` 或 `main` 文件（所有这些文件必须是有效的 YAML 文件）。它们的内容将自动提供给 playbook。那么，这在实践中是如何工作的呢？在与我们当前 `install.yaml` playbook 同一目录下，我们将创建一个 `roles` 目录。在该目录下，我们将再创建一个目录：`www`。在这个目录中，我们将创建：`tasks`、`files` 和 `handlers` 目录。我们将为 `development` 角色创建一个类似的结构：

```
roles/
    www/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
        handlers/         #
            main.yml      #  <-- handlers file
        files/            #
            service.cfg       #  <-- files for use with the copy resource
    development/               # this hierarchy represents a "role"
        tasks/            #
            main.yml      #  <-- tasks file can include smaller files if warranted
```

`development` 角色只有任务子目录，因为我们不需要任何额外的附加功能。那么，为什么是开发角色呢？当你回顾我们的当前 playbook 时，你会发现我们将 `www` 服务器和开发软件包（即编译器）的安装混在一起了。这是一个不好的做法，即使它们最终会安装在同一台物理服务器上。

因此，我们将编辑我们的清单文件，以便我们有两个独立的角色：

```
[www]
hostone ansible_host=192.168.1.2  ansible_ssh_private_key_file=~/.ssh/hostone.pem  ansible_user=admin
[development]
hostone ansible_host=192.168.1.3  ansible_ssh_private_key_file=~/.ssh/hostone.pem  ansible_user=admin
```

两个组只包含一个节点，这种做法整体上是不好的。我们这样做仅仅是为了本指南的目的。不要在 HTTP 服务器上安装编译器和开发软件，尤其是在生产环境中。

现在，我们需要在 playbook 中移动一些内容。`install.yml` 文件将变得更短，正如我们在这里看到的：

```
---
- name: Install nginx and php
  hosts: www
  roles:
    - www
  become: true
- name: Install development packages
  hosts: development
  roles:
    - development
  become: true
```

其实我们在这个 playbook 中有两个 play。一个是针对 `www` 主机的，另一个是针对 `development` 主机的。我们为每个 play 起个名字，并列出希望它运行的主机组，然后使用关键字 roles 列出实际使用的角色。如你所见，只要你遵循之前解释的目录结构，Ansible 会自动定位到正确的角色。当然，也可以通过指定角色的完整路径直接包含角色，但我们这里不做详细讲解。

现在，对于 `www` 角色，我们将执行以下代码：

```
---
- name: Install nginx
  ansible.builtin.package:
    name: '{{ item }}'
    state: present
  with_items:
    - nginx
    - php7.4
  notify:
    - Start nginx
- name: Copy service configuration
  ansible.builtin.copy:
    src: "files/service.cfg"
    dest: "/etc/nginx/sites-available/service.cfg"
    owner: root
    group: root
    mode: '0640'
- name: Enable site
  ansible.builtin.file:
    src: "/etc/nginx/sites-available/service.cfg"
    dest: "/etc/nginx/sites-enabled/default"
    state: link
  notify:
    - Restart nginx
```

从一开始你就应该注意到几个变化：

+   这份文档中没有 `tasks` 关键字。这是因为该文件是 `tasks` 子目录中的 `main.yaml` 文件，默认包含任务。

+   缩进已经向左移动。

+   这里没有 handlers。

+   我们已经移除了 `gcc` 和 `g++` 包的安装。

现在，让我们来看一下 handlers：

```
---
- name: Start nginx
  ansible.builtin.service:
    name: nginx
    state: started
  listen: "Start nginx"
- name: Restart nginx
  ansible.builtin.service:
    name: nginx
    state: restarted
  listen: "Restart nginx"
```

总体变化如这里所示：

+   我们已经移除了 `handlers` 关键字

+   我们已将缩进移到左侧

现在，让我们来看一下 `roles/development/tasks/main.yaml`：

```
---
- name: Install compilers
  ansible.builtin.package:
    name: '{{ item }}'
    state: present
  with_items:
    - gcc
    - g++
```

这非常简单。我们可能增加了目录结构的复杂性，但我们也简化了任务和 play 的编写。当你的 playbook 越来越大，越来越复杂时，这些好处完全值得这个权衡。

使用 CaC 工具有许多优势，具体如下：

+   配置可以进行 lint 检查——也就是说，可以通过自动化工具检查语法错误

+   它可以无限次应用，且结果相同

+   它可以在多个系统上并行运行

+   它可以存放在版本控制系统中，比如 Git，其中保存了变更的历史记录和评论，可以随时查看

+   它可以由自动化工具运行，从而不再需要人工干预，只需编写 playbook 即可

在这一小节中，我们展示了如何编写简单的 Ansible playbook。我们解释了什么是 Ansible，以及配置脚本的基本组成部分。我们介绍了 playbook、角色和清单。Ansible 还可以做很多其他事情。你可以管理设备、文件系统、用户、组、权限、网络等等。Ansible 默认自带的所有模块列表令人印象深刻。你可以随时查看这个列表，访问 [`docs.ansible.com/ansible/latest/module_plugin_guide/index.xhtml`](https://docs.ansible.com/ansible/latest/module_plugin_guide/index.xhtml)。记得查看你所使用的 Ansible 版本的列表。

所以，现在我们已经介绍了如何安装、配置以及使用 Ansible 来管理你的服务器（安装软件、创建配置文件和管理服务），接下来我们将探讨 Ansible Galaxy：一个社区开发的模块，能够提升 Ansible 的实用性。

# Ansible Galaxy

Ansible 是一个强大的自动化工具，使用户能够轻松配置、部署和管理复杂的 IT 基础设施。然而，创建和维护 Ansible playbook 可能会耗费时间，特别是在处理大规模环境时。幸运的是，Ansible Galaxy 通过提供一个预构建角色和 playbook 的中心化存储库，来帮助简化这个过程。

**Ansible Galaxy**是一个社区驱动的平台，托管着大量的 Ansible 角色和 playbook。这些角色和 playbook 由全球用户提交，并由 Ansible 的维护人员进行审查和策划。Ansible Galaxy 提供了一种简单高效的方法来查找和使用预构建的自动化内容，可以节省用户的时间和精力，同时确保质量和一致性。

使用 Ansible Galaxy，用户可以快速找到、下载和使用流行应用程序、服务和基础设施组件的预构建角色和 playbook。这些预构建组件可以帮助加快部署时间，确保遵循最佳实践，并减少错误或不一致性的可能性。Ansible Galaxy 还可以帮助用户从他人的经验中学习，并获得同行最佳实践的见解。

让我们使用一个 Galaxy 角色来在我们的`webserver`角色上安装`nginx` web 服务器。为了做到这一点，我们需要从 Ansible Galaxy 安装角色。首先，请确保在系统上安装了 Ansible，运行以下命令：

```
admin@myhome:~$ ansible-galaxy install nginxinc.nginx
```

此命令将从 Ansible Galaxy 下载并安装`nginx`角色。默认情况下，所有安装的角色都放置在`~/.ansible/roles`目录中。您可以通过在您的家目录中创建全局 Ansible 配置文件`~/.ansible.cfg`来更改这一设置。

更改`roles_path`目录的配置文件示例如下：

```
[defaults]
roles_path = /home/admin/myansibleroles
```

一个良好的做法是固定角色版本号，并将该版本保存在与 Ansible playbook 同一 Git 存储库中的 YAML 文件中。为了实现这一点，让我们创建一个`ansible_requirements.yml`文件：

```
---
- src: nginxinc.nginx
  version: 0.24.0
```

使用该文件从 Ansible Galaxy 安装角色，您可以运行以下命令：

```
admin@myhome:~$ ansible-galaxy install -r ansible_requirements.yml
```

安装角色后，可以通过将以下行添加到 playbook 中，在 Ansible playbook 中使用该角色：

```
roles:
    - nginxinc.nginx
```

这是一个示例 playbook，使用 Ansible Galaxy 中的`nginx`角色在远程服务器上安装和配置`nginx`：

```
---
- name: Install and configure Nginx
  hosts: webservers
  become: true
  roles:
    - nginxinc.nginx
  vars:
    nginx_sites:
      myapp:
        template: "{{ playbook_dir }}/templates/myapp.conf.j2"
```

在这个 playbook 中，我们将`webservers`组指定为目标主机，并使用`nginxinc.nginx`角色来安装和配置`nginx`。我们还定义了一个名为`nginx_sites`的变量，该变量指定了在 playbook 的`templates`目录中使用的 Jinja2 模板来创建一个`nginx`服务器块的配置。

通过使用 Ansible Galaxy 和预构建角色，如 `nginxinc.nginx`，用户可以快速而可靠地自动化复杂任务，确保一致性并减少错误的风险。

# 处理机密

保护诸如密码、令牌和证书等机密在任何 IT 基础设施中都至关重要。这些机密是访问敏感信息和服务的钥匙，它们的泄露可能导致严重的安全漏洞。因此，保持它们的安全至关重要。Ansible 提供了多种管理机密的方法，例如 Ansible Vault，它允许用户使用密码或密钥文件加密和解密敏感数据。此功能有助于保护机密，并确保只有授权用户才能访问它们。

将机密保存在 Git 仓库或任何其他公共地方是一个重大的安全风险。这类仓库通常对多个用户开放，其中一些用户可能没有访问敏感数据的必要权限。此外，像 Git 这样的版本控制系统会保留文件修改的历史记录，这可能导致机密不小心泄露。如果用户不小心将机密提交到仓库，或者黑客获取了仓库的提交历史，机密就可能会被暴露。因此，必须避免将机密保存到公共地方，以降低未经授权访问的风险。相反，Ansible 提供了安全的方式来管理机密，确保它们被加密且仅授权用户可访问。这样，用户可以确信他们的机密是安全的。

## Ansible Vault

**Ansible Vault** 是 Ansible 提供的一个功能，允许用户加密和解密敏感数据，如密码、密钥和证书。该保险库创建一个加密文件，只有授权用户才能解密，从而确保敏感数据的安全。保险库可用于存储文件、变量或其他 Ansible 使用的源中的机密。

Ansible Vault 使用多种加密方法来保护其中存储的机密。默认情况下，Ansible Vault 使用 AES 256 加密，这是一个广泛接受且安全的加密算法。此外，Ansible Vault 还支持其他加密算法，如 AES 192 和 AES 128，为加密强度提供了灵活性。当使用 Ansible Vault 加密数据时，用户可以选择使用密码或密钥文件进行加密。这确保了只有持有密码或密钥文件的授权用户才能解密存储在保险库中的机密。

要使用 Ansible Vault 创建一个新的保险库，可以使用以下命令：

```
admin@myhome:~$ ansible-vault create secrets.yml
New Vault password:
Confirm New Vault password:
```

这将创建一个名为 `secrets.yml` 的新加密保险库文件。系统会提示你输入一个密码来加密文件。一旦输入密码，保险库文件将被创建并在默认编辑器中打开。以下是一个示例 `secrets.yml` 文件：

```
somesecret: pleaseEncryptMe
secret_pgsql_password: veryPasswordyPassword
```

要编辑这个秘密文件，你需要使用`ansible-vault edit secrets.yml`命令，并在之后输入加密密码。

要编写一个 Ansible 任务，从保险库中读取`pgsql_password`秘密，你可以使用`ansible.builtin.include_vars`模块，并指定`vault_password_file`参数。以下是一个示例任务：

```
- name: Read pgsql_password from Ansible Vault
  include_vars:
    file: secrets.yml
    vault_password_file: /path/to/vault/password/file
  vars:
    pgsql_password: "{{ secret_pgsql_password }}"
```

在这个任务中，我们使用`include_vars`模块从`secrets.yml`保险库文件中读取变量。`vault_password_file`参数指定包含解密保险库密码的文件位置。然后，我们将`secret_pgsql_password`的值赋给`pgsql_password`变量，可以在剧本的其他地方使用该变量。

请注意，`secret_pgsql_password`变量应该在保险库中定义。`secret_`前缀表示该密码是从保险库中检索的。Ansible 不区分常规变量和秘密变量。

当你以提高的 Ansible 调试级别运行这个剧本时，你会注意到 PostgreSQL 密码暴露在调试输出中。为了防止这种情况发生，任何处理敏感信息的任务都可以启用`no_log: True`选项来执行。

## SOPS

**Secrets OPerationS**（**SOPS**）是 Mozilla 开发的一个开源工具，允许用户在各种配置文件中安全地存储和管理他们的秘密信息，包括 Ansible 剧本。SOPS 采用混合加密方法，即结合了对称加密和非对称加密，以确保最大安全性。

SOPS 使用主密钥加密秘密信息，主密钥可以是对称的或非对称的。**对称加密**使用密码或密码短语来加密和解密秘密，而**非对称加密**使用一对密钥，一个公钥和一个私钥，来加密和解密秘密。SOPS 通过密钥包装加密主密钥，密钥包装是一种使用另一个密钥加密密钥的技术。在 SOPS 中，通常用于密钥包装的密钥是 AWS **密钥管理服务**（**KMS**）密钥，但也可以是**Pretty Good Privacy**（**PGP**）密钥或 Google Cloud KMS 密钥。

SOPS 与 Ansible 无缝集成，支持多种文件格式，包括 YAML、JSON 和 INI。它还支持多种云提供商，包括 AWS、Google Cloud 和 Azure，使其成为跨不同环境管理秘密的多功能工具。

这是如何创建一个 SOPS 加密的 YAML 文件并使用`community.sops.load_vars`将其内容加载到 Ansible 剧本中的示例：

首先，创建一个名为`secrets.yaml`的 YAML 文件，内容如下：

```
postgresql_password: So.VerySecret
```

然后，使用以下命令通过 SOPS 加密文件：

```
admin@myhome:~$ sops secrets.yaml > secrets.sops.yaml
```

这将创建一个名为`secrets.sops.yaml`的加密版本的`secrets.yaml`文件。现在可以安全地删除`secrets.yaml`的明文文件。最常见的错误是忘记删除这些文件并将它们提交到 Git 仓库中。

接下来，创建一个新的 Ansible 剧本，命名为`database.yml`，并包含以下内容：

```
---
- hosts: dbserver
  become: true
  vars:
    postgresql_password: "{{ lookup('community.sops.load_vars', 'secrets.sops.yaml')['postgresql_password'] }}"
  tasks:
    - name: Install PostgreSQL
      apt:
        name: postgresql
        state: present
    - name: Create PostgreSQL user and database
      postgresql_user:
        db: mydatabase
        login_user: postgres
        login_password: "{{ postgresql_password }}"
        name: myuser
        password: "{{ postgresql_password }}"
        state: present
```

在这个示例中，我们使用 `community.sops.load_vars` 从加密的 `secrets.sops.yaml` 文件中加载 `postgresql_password` 变量。然后，使用 `{{ postgresql_password }}` Jinja2 语法将 `postgresql_password` 变量传递给 `postgresql_user` 任务。

当您使用 Ansible 运行这个 playbook 时，它会使用 SOPS 解密 `secrets.sops.yaml` 文件，并将 `postgresql_password` 变量加载到 playbook 中的 `postgresql_password` 变量中。这确保密码不会以明文形式存储在 playbook 中，为您的敏感信息提供额外的安全层。

有关 SOPS 的更多信息，请参见其官方 GitHub 仓库：[`github.com/mozilla/sops`](https://github.com/mozilla/sops)。

## 其他解决方案

有几种替代 Ansible Vault 和 SOPS 的解决方案，可以用于安全地管理敏感数据，具体如下：

+   **HashiCorp Vault**：一个开源工具，用于安全地存储和访问秘密。它提供了一种服务，用于安全和集中存储秘密、访问控制和审计。

+   **Blackbox**：一个命令行工具，使用 **GNU Privacy Guard**（**GPG**）加密和解密文件。它通过为每个需要访问加密数据的用户或团队创建一个单独的 GPG 密钥来工作。

+   **Keywhiz**：另一个开源的秘密管理系统，提供一种简单的方法来安全地存储和分发秘密。它包括一个用于管理秘密的 Web 界面和一个用于访问秘密的命令行工具。

+   **Azure Key Vault, AWS Secrets Manager 或 Google Cloud Secret Manager**：在处理云环境时，您可能想要考虑使用的解决方案来存储秘密。

当然，这不是所有可用选项的完整列表。

在本节中，我们介绍了 Ansible Vault 和 SOPS 作为处理秘密（如密码）的方法。在下一节中，我们将介绍 Ansible 的图形前端（GUI）。

# Ansible Tower 和替代方案

**Ansible Tower** 提供了一个集中平台，用于管理 Ansible 自动化工作流，使 IT 团队更容易合作、共享知识和维护基础设施。其一些关键功能包括用于管理 Ansible playbook、库存和作业运行的基于 Web 的界面、用于管理用户权限的 **基于角色的访问控制**（**RBAC**）、内置仪表板用于监控作业状态和结果，以及与其他工具和平台集成的 API。

它最早由 Ansible, Inc.（现在是 Red Hat 的一部分）于 2013 年发布，此后成为自动化 IT 工作流的最受欢迎工具之一。

自首次发布以来，Ansible Tower 经历了多次更新和增强，包括支持更复杂的自动化工作流、与 AWS 和 Azure 等云平台的集成以及改进的可扩展性和性能。Ansible Tower 是由 Red Hat 公司发布的商业产品。与 Ansible Tower 最相似的替代品是**Ansible** **WorX**（**AWX**）。

**Ansible AWX**是 Ansible Tower 的开源替代品，提供与 Tower 类似的许多功能，但具有更大的定制性和灵活性。AWX 于 2017 年首次发布，并迅速成为寻求在不需要商业许可证的情况下大规模实施 Ansible 自动化的组织的热门选择。

Ansible Tower 和 Ansible AWX 之间的主要区别之一是它们的许可模型。Ansible Tower 需要商业许可证，并作为 Red Hat Ansible Automation Platform 的一部分进行销售，而 Ansible AWX 是开源的，可以从 Ansible 官网免费下载。这意味着组织可以在自己的基础设施上部署 AWX，并根据特定需求进行定制，而无需依赖预构建的商业解决方案。

从功能上来看，Ansible Tower 和 Ansible AWX 非常相似，两者都提供基于 Web 的界面用于管理 Ansible 剧本、清单和任务运行，RBAC 用于管理用户权限，并且具有内置的仪表板来监控任务状态和结果。然而，Ansible Tower 提供了一些 Ansible AWX 没有的附加功能，比如与 Red Hat Ansible Automation Platform 的本地集成、先进的分析和报告功能，以及认证模块和集合。

另一个 Ansible Tower 的开源替代品是**Ansible Semaphore**。与 Tower 类似，它是一个基于 Web 的应用程序，旨在简化 Ansible 剧本和项目的管理。它是一个开源、免费的、易于使用的 Ansible Tower 替代品，允许用户轻松自动化其基础设施任务，而无需大量的编码知识。Ansible Semaphore 的首次发布是在 2016 年，从那时起，它已成为那些希望获得简单而强大的 Web 界面的用户的热门选择，用于管理他们的 Ansible 自动化工作流。

你可以在各自的官方网站上了解更多关于这些替代品的信息：

+   **Ansible** **Tower**: [`access.redhat.com/products/ansible-tower-red-hat`](https://access.redhat.com/products/ansible-tower-red-hat%20)

+   **Ansible** **AWX**: [`github.com/ansible/awx`](https://github.com/ansible/awx)

+   **Ansible** **Semaphore**: [`www.ansible-semaphore.com/`](https://www.ansible-semaphore.com/)

# 高级主题

在本节中，我们将展示如何处理高级 Ansible 功能和调试技术，以及如何自动检查剧本中可能存在的错误。

## 调试

为了调试 Ansible 剧本运行中的问题，通常将详细程度提高是有用的，以便获得关于 Ansible 正在执行的操作的更详细输出。Ansible 有四个详细程度：`-v`，`-vv`，`-vvv`和`-vvvv`。你添加的`v`越多，输出越详细。

默认情况下，Ansible 以`-v`运行，提供有关执行任务的基本信息。然而，如果你在剧本中遇到问题，增加详细程度可能会有帮助，以便获得更详细的输出。例如，使用`-vv`将提供有关正在执行的剧本、角色和任务的额外信息，而使用`-vvv`还会显示 Ansible 跳过的任务。

要提高 Ansible 剧本运行的详细程度，只需在`ansible-playbook`命令中添加一个或多个`-v`选项。以下是一个示例：

```
admin@myhome:~$ ansible-playbook playbook.yml -vv
```

这将以`-vv`详细程度运行剧本。如果你需要更详细的输出，可以添加额外的`-v`选项。你可以在这里看到一个示例：

```
admin@myhome:~$ ansible-playbook playbook.yml -vvv
```

除了使用`-v`选项外，你还可以通过在`ansible.cfg`文件中添加以下行来设置详细程度：

```
[defaults]
verbosity = 2
```

这将为所有`ansible-playbook`命令设置`-vv`的详细程度。你可以将值更改为`3`或`4`，以进一步增加详细程度。

如果你需要在详细模式中添加一些自定义通信（例如打印变量），你可以通过使用`debug`任务来实现。

这是一个示例剧本，演示了如何在`-vv` `debug`模式下打印一个变量：

```
---
- hosts: all
  gather_facts: true
  vars:
    my_variable: "Hello, World!"
  tasks:
    - name: Print variable in debug mode
      debug:
        msg: "{{ my_variable }}"
      verbosity: 2
```

在这个剧本中，我们定义了一个`my_variable`变量，存储了字符串`"Hello, World!"`。然后，我们使用`debug`模块通过`msg`参数打印出这个变量的值。

`verbosity: 2`这一行启用了`debug`模式。它告诉 Ansible 将详细程度设置为`-vv`，这样我们就可以看到`debug`模块的输出。

## Ansible 剧本代码检查

**Ansible 代码检查**是分析和验证 Ansible 代码语法和风格的过程，确保其符合最佳实践和标准。检查的目的是在代码执行之前捕捉潜在错误或问题，从而节省时间和精力。

最常用的 Ansible 代码检查工具是`ansible-lint`。这是一个开源命令行工具，用于分析 Ansible 剧本和角色，找出潜在问题并提供改进建议。

要运行`ansible-lint`检查示例剧本，你可以在终端中执行以下命令：

```
admin@myhome:~$ ansible-lint sample-playbook.yml
```

假设示例剧本已保存为`sample-playbook.yml`，并位于当前工作目录中，其内容如下：

```
---
- name: (Debian/Ubuntu) {{ (nginx_setup == 'uninstall') | ternary('Remove', 'Configure') }} NGINX repository
  ansible.builtin.apt_repository:
    filename: nginx
    repo: "{{ item }}"
    update_cache: true
    mode: "0644"
    state: "{{ (nginx_state == 'uninstall') | ternary('absent', 'present') }}"
  loop: "{{ nginx_repository | default(nginx_default_repository_debian) }}"
  when: nginx_manage_repo | bool
- name: (Debian/Ubuntu) {{ (nginx_setup == 'uninstall') | ternary('Unpin', 'Pin') }} NGINX repository
  ansible.builtin.blockinfile:
    path: /etc/apt/preferences.d/99nginx
    create: true
    block: |
      Package: *
      Pin: origin nginx.org
      Pin: release o=nginx
      Pin-Priority: 900
    mode: "0644"
    state: "{{ (nginx_state == 'uninstall') | ternary('absent', 'present') }}"
  when: nginx_repository is not defined
- name: (Debian/Ubuntu) {{ nginx_setup | capitalize }} NGINX
  ansible.builtin.apt:
    name: nginx{{ nginx_version | default('') }}
    state: "{{ nginx_state }}"
    update_cache: true
    allow_downgrade: "{{ omit if ansible_version['full'] is version('2.12', '<') else true }}"
  ignore_errors: "{{ ansible_check_mode }}"
  notify: (Handler) Run NGINX
```

请注意，`ansible-lint`的输出可能会根据所使用的 Ansible 版本和`ansible-lint`版本以及启用的特定规则而有所不同。

作为示例，以下是运行`ansible-lint`检查提供的示例剧本后的输出：

```
[WARNING]: empty path for ansible.builtin.blockinfile, path set to ''
[WARNING]: error loading version info from /usr/lib/python3.10/site-packages/ansible/modules/system/setup.py: __version__ = '2.10.7'
[WARNING]: 3.0.0 includes an experimental document syntax parser that could result in parsing errors for documents that used the previous parser. Use `--syntax-check` to verify new documents before use or consider setting `document_start_marker` to avoid using the experimental parser.
sample-playbook.yml:1:1: ELL0011: Trailing whitespace
sample-playbook.yml:7:1: ELL0011: Trailing whitespace
sample-playbook.yml:9:1: ELL0011: Trailing whitespace
sample-playbook.yml:17:1: ELL0011: Trailing whitespace
sample-playbook.yml:22:1: ELL0011: Trailing whitespace
sample-playbook.yml:26:1: ELL0011: Trailing whitespace
sample-playbook.yml:29:1: ELL0011: Trailing whitespace
sample-playbook.yml:35:1: ELL0011: Trailing whitespace
sample-playbook.yml:38:1: ELL0011: Trailing whitespace
sample-playbook.yml:41:1: ELL0011: Trailing whitespace
sample-playbook.yml:44:1: ELL0011: Trailing whitespace
sample-playbook.yml:47:1: ELL0011: Trailing whitespace
sample-playbook.yml:50:1: ELL0011: Trailing whitespace
sample-playbook.yml:53:1: ELL0011: Trailing whitespace
```

输出指示 Playbook 的多个行存在行尾空白，违反了 `ansible-lint` 的 `ELL0011` 规则。关于空路径和版本信息加载的警告消息并不是关键问题，可以安全地忽略。

要修复行尾空白问题，只需删除每个受影响行末尾的额外空格。问题修复后，可以重新运行 `ansible-lint` 来确保 Playbook 没有进一步的问题。

## 加速 SSH 连接

Ansible 是一款开源自动化工具，广泛用于部署和管理 IT 基础设施。Ansible 的一个关键特性是能够使用 SSH 与远程服务器进行安全通信。然而，为每个单独任务使用 SSH 可能会耗费时间并影响性能，特别是在处理大量服务器时。为了解决这个问题，Ansible 支持**SSH 多路复用**，允许多个 SSH 连接共享单个 TCP 连接。

SSH 多路复用通过重用现有的 SSH 连接而不是为每个任务创建新连接来工作。当 Ansible 建立与远程服务器的 SSH 连接时，它打开一个 TCP 套接字并为该连接创建一个控制套接字。控制套接字是用于管理 SSH 连接的特殊套接字。当请求到同一主机的另一个 SSH 连接时，Ansible 会检查是否已存在该连接的控制套接字。如果存在，则 Ansible 会重用现有的控制套接字，并在同一 SSH 连接内为新任务创建一个新通道。

SSH 多路复用的好处在于通过减少 Ansible 需要建立的 SSH 连接数量来节省时间和资源。此外，它可以通过减少为每个任务创建和拆除 SSH 连接的开销来提高 Ansible 的性能。

要在 Ansible 中启用 SSH 多路复用，需要在 SSH 客户端配置文件中配置 `ControlMaster` 和 `ControlPath` 选项。`ControlMaster` 选项启用 SSH 多路复用的使用，而 `ControlPath` 选项指定控制套接字的位置。默认情况下，Ansible 使用 `~/.ansible/cp` 目录存储控制套接字。您还可以通过设置 `ControlPersist` 选项来配置可以同时复用的最大 SSH 连接数。

要自定义 SSH 多路复用配置，可以通过在 `~/.ansible.cfg` 中添加 `[ssh_connection]` 部分来将 SSH 选项放入默认的 Ansible 配置文件中，如下所示：

```
[ssh_connection]
ssh_args = -o ControlMaster=auto -o ControlPersist=3600
control_path = ~/.ssh/multiplexing/ansible-ssh-%%r@%%h:%%p
```

在添加此配置后，确保还创建了 `~/.ssh/multiplexing` 目录。

`ControlMaster=auto` 选项会自动创建一个主会话，如果已经存在主会话，则后续会话将自动进行复用。设置 `ControlPersist=3600` 将会在后台保持主连接打开，以接受 `3600` 秒（1 小时）内的新连接。

## 动态清单

在 AWS 等云环境中，服务器可以根据需求动态创建和终止。手动管理这些服务器可能是一个艰巨的任务，这就是为什么自动化工具，如 Ansible，非常重要。Ansible 的一个关键特性是动态清单，它使其非常适合云环境。

**动态清单** 是 Ansible 的一个功能，它支持在云环境中自动发现主机（服务器）和组（标签）。在 AWS 中，Ansible 可以使用 **弹性计算云**（**EC2**）清单插件来查询 AWS API，获取有关 EC2 实例和组的信息。

要在 AWS 中使用动态清单，你需要在 Ansible 配置中配置 EC2 清单插件。

`amazon.aws.aws_ec2` 库插件是一个官方的 Ansible 插件，它支持 Amazon EC2 实例的动态清单。要使用此插件，你需要按照接下来的步骤操作。

根据你使用的 Ansible 版本，以及是否安装了完整的 Ansible（而不仅仅是 Ansible Core），你可能需要使用 Ansible Galaxy 安装 AWS 集合插件，方法如下：

```
admin@myhome:~$ ansible-galaxy collection install amazon.aws
```

在你的 Ansible 控制节点上安装 `boto3` 和 `botocore` 库。你可以使用 `pip` 包管理器安装它们，方法如下：

```
admin@myhome:~$ pip install boto3 botocore
```

创建一个具有必要权限以访问 EC2 实例和组的**身份和访问管理**（**IAM**）用户。你可以使用 AWS 管理控制台或 AWS CLI 创建 IAM 用户。确保保存 IAM 用户的访问密钥和秘密访问密钥。

在你的 Ansible 控制节点上创建一个 AWS 凭证文件（`~/.aws/credentials`），并添加 IAM 用户的访问密钥和秘密访问密钥，如下所示：

```
[default]
aws_access_key_id = YOUR_ACCESS_KEY
aws_secret_access_key = YOUR_SECRET_KEY
```

在你的项目目录中创建一个 Ansible 清单文件（`inventory.yml`），并配置它使用 `amazon.aws.aws_ec2` 插件，方法如下：

```
plugin: amazon.aws.aws_ec2
regions:
  - eu-central-1
filters:
  tag:Environment:
    - webserver
    - frontend
```

下面是配置选项的简要说明：

+   `plugin`：指定要使用的清单插件

+   `regions`：指定要搜索实例的 AWS 区域

+   `filters`：允许你按标签筛选 EC2 实例

通过运行以下命令测试清单：

```
admin@myhome:~$ ansible-inventory -i inventory.yml --list
```

该命令应输出一个 JSON 对象，列出指定区域内所有 EC2 实例，并按 Ansible 标签分组。

这样，你就不需要在每次运行 Ansible 剧本时都更新清单文件，以确保拥有最新的服务器列表。

# 总结

在本章中，我们介绍了 Ansible CaC 工具。我们已经解释并演示了如何将配置从部落知识和文档（以及描述使系统达到目标状态所需的步骤）转移到能够基于定义良好的语法实现该配置的工具，这为你的组织带来了诸如可重复性、能够并行运行多个配置、自动化测试和执行等好处。

在下一章中，我们将向你介绍**基础设施即代码**（**IaC**）。

# 进一步阅读

+   **掌握 Ansible（第四版）** 由 James Freeman 和 Jesse Keating 编写

+   **Ansible Playbook 基础** 由 Gourav Shah 编写

+   **Ansible 实际应用自动化** 由 Gineesh Madapparambath 编写
