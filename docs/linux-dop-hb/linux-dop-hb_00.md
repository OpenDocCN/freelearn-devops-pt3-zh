# 前言

DevOps 已经成为现代软件开发和交付的关键组成部分。它彻底改变了我们构建、测试、部署和运营软件系统的方式。DevOps 不仅是一套工具和实践，它更是一种文化和心态，强调协作、沟通和自动化。

本书旨在成为一本全面的 DevOps 指南，涵盖从选择合适的 Linux 发行版到避免 DevOps 中的陷阱的所有内容。本书的每一章都提供了详细的信息和实际的示例，帮助你理解这些概念并将其应用于实际场景中。

# 本书适合对象

本书面向那些已经在软件开发和 IT 运维领域积累了一定知识和经验的人，旨在进一步扩展他们对 DevOps 和 Linux 系统的理解。

如果你对 Linux 系统不太熟悉，本书将为你提供必要的指导和工具，帮助你快速学习并掌握 Linux 基础设施的管理。你将了解 Linux 操作系统、其架构及基本概念。

此外，本书还强调学习公共云技术，重点介绍 AWS。如果你有兴趣了解如何使用 AWS 构建和管理可扩展、可靠的系统，本书将为你提供必要的知识和工具，帮助你入门。

无论你是 DevOps 新手还是已经有一定经验，本书都为学习更复杂的概念提供了坚实的基础。它涵盖了从 Linux 系统基础到更高级的 DevOps 实践（如配置与基础设施即代码、CI/CD 等）的一系列主题。

# 本书涵盖的内容

*第一章*，*选择合适的 Linux 发行版*，讨论了 GNU/Linux 的历史以及流行发行版之间的差异。

*第二章*，*命令行基础*，引导你了解命令行的使用以及我们在全书中将使用的常用工具。

*第三章*，*进阶 Linux*，描述了 GNU/Linux 中的一些高级功能，这些功能将对你有用。

*第四章*，*使用 Shell 脚本自动化*，解释了如何使用 Bash shell 编写自己的脚本。

*第五章*，*Linux 中的服务管理*，讨论了管理 Linux 中服务的不同方式，并向你展示如何使用 systemd 定义自己的服务。

*第六章*，*Linux 中的网络*，描述了网络是如何工作的，如何控制网络配置的不同方面，以及如何使用命令行工具。

*第七章*，*Git，通往 DevOps 的门户*，讨论了 Git 是什么，以及如何使用 Git 的版本控制系统，包括一些不太为人知的 Git 特性。

*第八章*，*Docker 基础*，探讨了如何将你的服务或工具容器化，以及如何运行和管理容器。

*第九章*，*深入探索 Docker*，讨论了 Docker 的更多高级功能，包括 Docker Compose 和 Docker Swarm。

*第十章*，*监控、追踪和分布式日志记录*，讨论了如何监控你的服务、可以在云中使用的工具以及如何进行基础设置。

*第十一章*，*使用 Ansible 进行配置即代码（Configuration as Code）*，介绍了如何使用 Ansible 实现配置即代码；它将引导你完成 Ansible 的基本设置及更多高级功能。

*第十二章*，*利用基础设施即代码（Infrastructure as Code）*，讨论了**基础设施即代码**（**IaC**）的概念、流行工具以及如何使用 Terraform 管理基础设施。

*第十三章*，*使用 Terraform、GitHub 和 Atlantis 实现 CI/CD*，通过使用 Terraform 和 Atlantis 对基础设施进行**持续集成**（**CI**）和**持续部署**（**CD），将 IaC 推向更高水平。

*第十四章*，*避免 DevOps 中的陷阱*，讨论了你在 DevOps 工作中可能遇到的挑战。

# 为了最大限度地从本书中受益

你需要在虚拟机或计算机的主操作系统中安装 Debian Linux 或 Ubuntu Linux。我们使用的其他软件要么已作为默认工具集预装，要么我们会告诉你如何获取并安装它。

假设你具备一些基本的 Linux 知识及其命令行界面（CLI）操作经验。熟悉 shell 脚本和基本的编程概念也将对你有帮助。此外，建议你具备一定的 IT 基础设施管理知识，并对软件开发实践有一定的了解。

本书面向 DevOps 新手，假设你渴望学习这个领域中常用的工具和概念。在阅读完本书后，你将能深入理解如何使用 IaC 工具（如 Terraform 和 Atlantis）来管理基础设施，并掌握如何使用 Ansible 和 Bash 脚本自动化重复任务。你还将学习如何设置日志记录和监控解决方案，帮助你维护和排查基础设施问题。

| **本书涉及的软件/硬件** | **操作系统要求** |
| --- | --- |
| Bash | Linux 操作系统预装 |
| Ansible | Python 3 或更新版本 |
| Terraform | Linux 操作系统 |
| AWS CLI | Python 3 或更新版本 |
| Docker | Linux 操作系统 |

如果您正在使用本书的数字版，我们建议您自己输入代码或通过本书的 GitHub 仓库获取代码（下节会提供链接）。这样可以帮助您避免与复制粘贴代码相关的潜在错误。

# 下载示例代码文件

您可以从 GitHub 上下载本书的示例代码文件，链接为 [`github.com/PacktPublishing/The-Linux-DevOps-Handbook`](https://github.com/PacktPublishing/The-Linux-DevOps-Handbook)。如果代码有更新，将会在 GitHub 仓库中进行更新。

我们还提供了丰富的书籍和视频代码包，您可以访问 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 查看。

# 使用的约定

本书中使用了多种文本约定。

`文本中的代码`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号。示例如下：“当以 root 用户登录时，您的提示符号会以 `#` 结尾。当以普通用户登录时，提示符号会显示 `$`。”

代码块如下所示：

```
docker build [OPTIONS] PATH | URL | -
```

当我们希望引起您注意某一代码块的特定部分时，相关行或项目将以粗体显示：

```
docker build [OPTIONS] PATH | URL | -
```

任何命令行输入或输出将按如下方式书写：

```
chmod ug=rx testfile
```

**粗体**：表示新术语、重要单词或屏幕上显示的单词。例如，菜单或对话框中的单词会以**粗体**显示。示例如下：“**Ansible Galaxy** 是一个由社区驱动的平台，提供了大量的 Ansible 角色和剧本。”

提示或重要说明

这样显示。

# 联系我们

我们始终欢迎读者的反馈。

**常规反馈**：如果您对本书的任何方面有疑问，请通过电子邮件联系 [customercare@packtpub.com](http://customercare@packtpub.com)，并在邮件主题中注明书名。

**勘误**：尽管我们已尽最大努力确保内容的准确性，但错误仍然可能发生。如果您在本书中发现错误，我们将非常感激您能将其报告给我们。请访问 www.packtpub.com/support/errata 并填写表格。

**盗版**：如果您在互联网上遇到我们作品的任何非法副本，请向我们提供该位置地址或网站名称。请通过 [copyright@packt.com](http://copyright@packt.com) 联系我们，并附上该材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上有专长并且有兴趣写作或为书籍贡献内容，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦您阅读完 *《Linux DevOps 手册》*，我们很想听听您的想法！[请点击这里直接进入本书的亚马逊评论页面并分享您的反馈](https://packt.link/r/1803245662)。

您的评论对我们以及技术社区非常重要，能够帮助我们确保提供优质内容。

# 下载本书的免费 PDF 版本

感谢您购买本书！

您喜欢随时阅读，但无法随身携带纸质书籍吗？

您购买的电子书与您选择的设备不兼容吗？

不用担心，现在每本 Packt 书籍都可以免费获得该书的无 DRM 版本 PDF。

随时随地，在任何设备上阅读。您可以直接从您最喜欢的技术书籍中搜索、复制并粘贴代码到您的应用程序中。

好处不仅仅是这些，您还可以获得独家的折扣、新闻通讯，并且每天会在您的邮箱中收到精彩的免费内容

按照以下简单步骤获得福利：

1.  扫描二维码或访问下面的链接

![](img/B18197_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781803245669`](https://packt.link/free-ebook/9781803245669)

1.  提交您的购买证明

1.  就是这样！我们会直接将您的免费 PDF 和其他福利发送到您的邮箱

# 第一部分：Linux 基础

在本书的开篇部分，我们将重点介绍 Linux 发行版及您需要的基本技能，以便高效使用 Linux 操作系统。您还将学习如何编写基本的 Shell 脚本来自动化日常任务。

本部分包括以下章节：

+   *第一章*，*选择合适的 Linux 发行版*

+   *第二章*，*命令行基础*

+   *第三章*，*Linux 进阶*

+   *第四章*，*使用 Shell 脚本自动化*
