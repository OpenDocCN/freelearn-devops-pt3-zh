# 前言

随着 DevOps 和平台工程推动了对强大内部开发平台的需求，基础设施自动化解决方案的需求比以往任何时候都更加迫切。Puppet 是全球最大企业使用的最强大基础设施自动化解决方案之一，并且拥有强大的开源社区。本书全面介绍了 Puppet 语言和平台。从 Puppet 作为一种有状态语言的基本概念和工作方式入手，逐步讲解如何构建 Puppet 代码，使其具备可扩展性，并允许团队间的灵活性与协作。接下来，将探讨 Puppet 平台如何实现基础设施配置的管理和报告，展示如何将 Puppet 平台与 ServiceNow 和 Splunk 等其他工具集成。最后，本书还将讨论如何将 Puppet 实现应用于高度监管和审计的环境，以及现代混合云环境。

通过本书，你将全面了解 Puppet 语言和平台的功能，并能够构建和扩展 Puppet，以创建一个提供企业级基础设施自动化的平台。

# 本书适用对象

本书非常适合希望使用 Puppet 自动化基础设施配置的 DevOps 工程师。它专门聚焦于 Puppet 的配置管理能力，但也涉及到一般的其他基础设施管理实践。无论是初学者还是当前的 Puppet 用户，都能通过本书全面了解 Puppet 语言和平台的全部功能。

# 本书内容

*第一章*，*Puppet 概念与实践*，重点介绍了为什么要开发 Puppet，它是如何随着时间变化的，以及 Puppet 的核心概念和实践。同时，还探讨了 Puppet 如何帮助实现 DevOps 转型以及我们对此的具体做法。

*第二章*，*重大变化、有用工具与参考资料*，讨论了 Puppet 5 之后出现的重大变化，如有害术语、敏感值、延迟函数等高层次的内容，还会介绍一些已被 Puppet 抛弃的项目。此外，本章将介绍一些有助于开发的工具，如 VS Code 和**Puppet 开发工具包**（**PDK**），并展示实验室和开发环境如何为本书的内容提供支持。还将展示各种 Puppet 和社区的参考资料，供进一步学习。

*第三章*，*Puppet 类、资源类型与提供者*，介绍了 Puppet 的最基本构建模块以及如何使用它们，使你能够理解编写 Puppet 代码的初步阶段，展示了资源类型和提供者如何协同工作，创建与底层操作系统实现无关的有状态代码，以及类如何将这些资源进行分组。 

*第四章*，*变量和数据类型*，详细介绍了如何在 Puppet 中为变量分配数据类型，如何在数组和哈希中管理它们，如何使用敏感数据类型来保护变量，以及如何管理变量的作用域。然后，我们将提供一些关于如何在 Puppet 中有效使用这些变量和数据结构的最佳实践建议。

*第五章*，*事实与函数*，探讨了 Puppet 提供的事实和因素，如何在 Puppet 代码中使用它们，以及如何自定义它们。它还将讨论函数：它们是什么，如何与 lambda 一起使用，以及如何使用相对较新的延迟函数。

*第六章*，*关系、排序与作用域*，讲解了 Puppet 如何处理关系和排序，以及作用域和包含。这些问题结合在一起，帮助用户理解跨模块或跨类的资源和变量是如何交叉的。

*第七章*，*模板、迭代和条件语句*，展示了如何使用模板、迭代、循环和各种条件语句，如`if`语句和选择器，来影响代码的流动和管理。

*第八章*，*开发和管理模块*，讨论了模块的结构，使用 PDK 创建模块的方法，以及如何测试模块。还将讨论如何有效使用 Puppet Forge 来使用和分享代码，并了解共享模块的质量。

*第九章*，*使用 Puppet 处理数据*，介绍了 Puppet 如何处理数据，讨论了什么是 Hiera，在哪些层级存储数据，以及在结构和方法上要避免的一些陷阱和错误。

*第十章*，*Puppet 平台的组成部分和功能*，帮助你了解 Puppet 作为一个平台的构成，各个组件如何协同工作和通信，以及常见的架构方法以实现扩展性。

*第十一章*，*分类与发布管理*，讨论了 Puppet 如何在环境中管理服务器和代码，如何对服务器进行分类，以及这种分类的 Puppet 运行实际是如何执行的。还将讨论部署代码到这些环境中的工具。

*第十二章*，*用于编排的 Bolt*，讨论了如何使用 Bolt 作为程序任务的编排工具，展示了通过 Puppet 代理使用的各种传输选项——SSH、WinRM 和 PCP。你将看到任务和计划如何补充 Puppet 代码，以及如何通过 Bolt 本身编排和部署 Puppet 代码。

*第十三章*，*深入 Puppet 服务器*，探讨了更高级的话题，确保你能够监控和扩展基础设施，处理常见问题，并整合外部数据源。

*第十四章*，*Puppet Enterprise 简介*，突出了 Puppet Enterprise 与开源版本的差异，以及可用的集成和服务，以帮助扩展和调整基础设施。

*第十五章*，*采用方法*，讨论了如何在实际的棕地环境中采用并使用 Puppet，强调了在该领域和各种采用过程中获得的经验教训，并着眼于正确界定使用案例以便定期交付。它将探讨 Puppet 如何在平台工程中工作，如何与传统遗产平台以及高度监管和变更管理的环境兼容。

# 如何最大化本书的价值

需要一些 Unix 和 Windows 系统的系统管理背景知识以及应用程序部署的基础知识。此外，还需要一些核心开发概念的知识，例如版本控制工具（Git）、虚拟化和测试工具，以及编码工具（如 vi 或 Visual Studio Code）。

| **本书中涉及的软件/硬件** | **操作系统要求** |
| --- | --- |
| Puppet 7 或 8 | Windows、macOS 或 Linux |
| Bolt | Windows、macOS 或 Linux |
| Visual Studio Code | Windows、macOS 或 Linux |
| Azure |  |
| **Puppet 开发工具包** **（PDK）** | Windows、macOS 或 Linux |
| PEADM 模块 | Windows、macOS 或 Linux |

实验环境所需软件的完整配置将在 *第二章* 中进行介绍。

**如果你正在使用本书的数字版本，我们建议你自己输入代码，或者从本书的 GitHub 仓库访问代码（链接将在下一节提供）。这样做有助于避免与复制和粘贴代码相关的潜在错误。**

# 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，地址是 [`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers)。如果代码有更新，它将会在 GitHub 仓库中同步更新。

我们还有来自我们丰富书籍和视频目录中的其他代码包，可以在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 获取。快来看看吧！

下载彩色图片

我们还提供了一个 PDF 文件，其中包含本书中使用的截图和图表的彩色图片。你可以在这里下载：[`packt.link/vPsXh`](https://packt.link/vPsXh)

# 使用的约定

本书中使用了许多文本约定。

`Code in text`：表示文本中的代码词汇、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。例如： “查找函数键，`data_hash`，接受`yaml_data`、`json_data`和`hocon_data`作为值，但大多数 Puppet 实现仅使用 YAML 数据，因此本书默认使用`yaml_data`后端。”

一段代码块设置如下：

```
hierarchy:
- name: "YAML layers"
  paths:
    - "nodes/%{trusted.certname}.yaml"
    - "location/%{fact.data_center}.yaml"
    - "common.yaml"
```

当我们希望引起您注意代码块中的某个特定部分时，相关的行或项会加粗显示：

```
 type { 'title': 
   attribute1 => value1, 
   attribute2 => value2, 
 }
```

任何命令行输入或输出如下所示：

```
bolt --verbose plan run pecdm::provision @params.json
```

**加粗**：表示一个新术语、一个重要词汇或您在屏幕上看到的文字。例如，菜单或对话框中的文字显示为**加粗**。例如：“从**管理**面板中选择**系统信息**。”

提示或重要说明

显示如下。

# 联系我们

我们始终欢迎读者的反馈。

`customercare@packtpub.com`，并在邮件主题中注明书名。

**勘误**：尽管我们已尽力确保内容的准确性，但难免会有错误。如果您在本书中发现错误，我们将不胜感激，请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表格。

`copyright@packt.com`，并附上相关材料的链接。

**如果您有兴趣成为作者**：如果您在某个话题上有专业知识，且有意写书或为书籍作贡献，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

阅读完*Puppet 8 for DevOps Engineers*后，我们很想听听您的想法！请点击这里直接访问此书的 Amazon 评论页面并分享您的反馈。

您的评论对我们和技术社区非常重要，能够帮助我们确保提供高质量的内容。

# 下载本书的免费 PDF 版本

感谢购买本书！

您喜欢随时随地阅读，但无法随身携带印刷版书籍吗？

电子书购买不兼容您选择的设备吗？

不用担心，现在每本 Packt 书籍，您都能免费获得该书的无 DRM PDF 版本。

随时随地、在任何设备上阅读。直接将您最喜欢的技术书籍中的代码搜索、复制并粘贴到您的应用程序中。

优惠不仅仅是这些，您还可以获得独家折扣、时事通讯，并每日收到丰富的免费内容到您的邮箱

按照以下简单步骤获取福利：

1.  扫描二维码或访问以下链接

![](img/B18492_QR_Free_PDF.jpg)

https://packt.link/free-ebook/9781803231709

1.  提交您的购买凭证

1.  就是这样！我们将直接把免费的 PDF 文件和其他福利发送到您的邮箱

# 第一部分 – Puppet 简介及 Puppet 语言基础

本部分将建立 Puppet 的核心概念，阐述 Puppet 的功能、如何与 DevOps 方法相结合，以及本书中我们将如何处理这些内容。接下来，我们将对 Puppet 的核心组件进行高层次的概览。本书中使用的开发实验环境将被回顾，并提供有用的参考资料和进一步学习资源。然后，我们将从语言的基础开始，介绍类、资源、变量和函数。

本部分包含以下章节：

+   *第一章*，*Puppet 概念与实践*

+   *第二章*，*主要变化、实用工具与参考资料*

+   *第三章*，*Puppet 类、资源类型与提供者*

+   *第四章*，*变量与数据类型*

+   *第五章*，*事实与函数*
