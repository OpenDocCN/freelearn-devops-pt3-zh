# 前言

DevOps 作为一种运动要求团队中的人员检查和改变他们工作和互动的三个方面：人员、流程和工具。人员的变化可能包括组织结构或文化的变化。流程变化可能包括引入精益思想中的实践，以确保持续交付。工具的变化则通过自动化实现更快的周转。

Atlassian 生产的工具无疑能帮助我们的团队改进流程并促进自动化。像 Jira 和 Confluence 这样的关键工具让团队能够建立促进敏捷和精益思想的标准流程。通过引入 Bitbucket Pipelines，Bitbucket Cloud 不仅促进了版本控制，还通过自动化实现了持续集成和持续部署。

但 Atlassian 工具的标志性特点不是工具本身，而是它们能与彼此以及其他不同制造商的工具轻松连接的能力。Atlassian 的 Open DevOps 平台提供了创建轻松互联的能力，无论供应商是谁。

本书的重点不是任何单一的 Atlassian 产品，如 Jira 或 Confluence。相反，本书旨在突出 Atlassian 工具与其他 Atlassian 工具或其他开发工具（如 GitLab、GitHub、Snyk 和 LaunchDarkly）之间的轻松互联。希望通过阅读本书，您能将 Atlassian 的投资与其他投资结合起来，创建一个强大的工具链。

我们假设您组织当前使用的工具包括 Atlassian 和其他供应商的工具。因此，我们不建议按顺序阅读本书。我们认为本书的最佳使用方式是阅读与连接您工具相关的章节，而无需担心创建一个仅由 Atlassian 工具构成的技术栈。这些渐进式的变化遵循精益方法，是采用 DevOps 的关键。

本书的食谱以在*第十三章*中展示的理想 DevOps 工具链示例结束。虽然示例主要使用 Atlassian 工具，但它可以由任何组合的工具构成。

# 本书适用对象

本书面向 Atlassian 工具和 DevOps、DevSecOps 实践者（如开发人员和网站可靠性工程师）的管理员。管理员需要了解如何创建团队结构，如 Jira 项目、Confluence 空间和 Bitbucket 工作区。

# 本书内容

*第一章*，*DevOps 和 Atlassian 生态系统简介*，简要介绍了 DevOps 的历史，并展示了如何从 Jira 安装 Open DevOps 或其他 Atlassian 工具的步骤。

*第二章*，*通过 Jira 产品发现发掘客户需求*，介绍了 Jira 产品发现功能，帮助产品经理开发并优先考虑产品和功能的种子，直到它们准备好由团队开发。

*第三章*, *通过 Confluence 进行规划和文档管理*，展示了将 Jira 与 Confluence 连接的功能，以实现最新的报告和文档。

*第四章*, *为设计、源代码管理和持续集成启用连接*，包含将 Jira 与其他源代码管理和持续集成工具连接的示例。

*第五章*, *理解 Bitbucket 和 Bitbucket Pipelines*，介绍了 Bitbucket Cloud。我们查看了源代码管理功能，并对 Bitbucket Pipelines 进行了初步介绍。

*第六章*, *扩展和执行 Bitbucket Pipelines*，继续探索 Bitbucket Pipelines，了解如何创建和编辑其主要控制文件`bitbucket-``pipelines``.yml`。

*第七章*, *利用测试用例管理和安全工具进行 DevSecOps*，演示了如何连接到测试和安全扫描工具，并介绍了 DevSecOps 的概念。

*第八章*, *通过 Bitbucket Pipelines 进行部署*，探讨了如何使用 Bitbucket Pipelines 配置持续部署。我们展示了多个环境的部署示例。

*第九章*, *利用 Docker 和 Kubernetes 进行高级配置*，展示了 Bitbucket Pipelines 如何利用 Docker 和 Kubernetes 来辅助构建和部署。

*第十章*, *通过持续部署和可观测性与运维团队协作*，探讨了如何将 Jira 与提供持续部署和监控的工具连接。

*第十一章*, *通过 CheckOps 在 Compass 中监控组件活动和指标*，介绍了一个新的 Atlassian 工具 Compass，用于收集和监控部署和事件信息，以衡量项目组件在 CheckOps 这一学科中的健康状况。

*第十二章*, *通过 Opsgenie 警报进行升级*，演示了如何使用 Opsgenie，当事件发生时，团队可以协作处理。

*第十三章*, *通过一个真实世界的示例整合所有内容*，将你从前面章节学到的所有内容结合起来，展示 DevOps 工具链是如何运作的。

*第十四章**, 附录 – 关键要点和 Atlassian DevOps 工具的未来*，以展望未来为结尾，并提供了其他入门建议，帮助你开始 DevOps 之旅。

# 为了从本书中获得最大的收益

工具管理员需要知道如何在 Jira 中创建项目；在 Confluence 中创建空间；以及在 Bitbucket 中创建工作区、项目和代码库。

Jira 用户需要知道如何创建问题。

Confluence 用户需要知道如何创建页面。

使用 Bitbucket 和其他 Git 服务器工具（如 GitLab 和 GitHub）的用户需要了解如何在 Git 中执行提交以及在他们选择的 Git 服务器工具中进行拉取请求/合并请求。

尽管我们讨论的 Atlassian 工具是基于云的、**软件即服务**（**SaaS**）应用程序，但某些操作或功能需要外部资源。我们在下表中概述了这些外部资源：

| **书中涵盖的操作/功能** | **外部** **资源要求** |
| --- | --- |
| 连接到 Moqups（第四章） | Moqups 账户 |
| 连接到 Figma（第四章） | Figma 账户 |
| 连接到 GitLab（第四章和第十章） | GitLab 账户 |
| 连接到 GitHub（第四章） | GitHub 账户 |
| 连接到 Jenkins（第四章） | Jenkins 服务器 |
| 连接到 CircleCI（第四章） | CircleCI 账户 |
| Bitbucket 运行器 – 自托管（第六章） | Windows、macOS 或 Linux 机器 |
| 连接到 Snyk（第七章和第十三章） | Snyk 账户 |
| 使用 JFrog CLI 部署到 JFrog | JFrog 环境中的账户 |
| 部署到 Sonatype Nexus | Nexus 环境中的账户 |
| 部署到 AWS S3（第八章） | 拥有适当角色的 AWS 账户 |
| 部署到 Google Cloud（第八章） | 拥有适当权限的 Google Cloud Platform 账户 |
| 部署到 Microsoft Azure（第八章） | 拥有适当权限的 Azure 账户 |
| 连接到监控工具（第十章） | Datadog 账户 |

**如果你使用的是本书的数字版，我们建议你亲自输入代码或通过 GitHub 仓库访问代码（下节中提供链接）。这样做将帮助你避免与复制和粘贴代码相关的潜在错误。**

## 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，网址为 [`github.com/PacktPublishing/Atlassian-DevOps-Toolchain-Cookbook`](https://github.com/PacktPublishing/Atlassian-DevOps-Toolchain-Cookbook)。如果代码有更新，GitHub 上的现有仓库也会进行更新。

我们还有来自我们丰富书籍和视频目录的其他代码包， 可以在 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/) 上查看！不要错过！

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：指文本中的代码单词、数据库表名、文件夹名、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 账号。例如：“在页面上，在搜索栏中键入 `Jira report`。”

一段代码的设置如下：

```
      pipelines: 
        default: 
          - step: 
            script: 
             - echo "Running a command"
```

任何命令行输入或输出都按如下方式编写：

```
kubectl run <my.app> --labels="app=<my.app>" --image=<my.dockerhub.username>/<my.app>:latest --replicas=2 --port=8080
```

**粗体**：表示新术语、重要词汇或你在屏幕上看到的单词。例如，菜单或对话框中的词汇会以这种方式显示在文本中。这里有一个例子：“选择 **Insights** 查看其他来源的链接，或通过点击 **Create an** **insight** 按钮自行创建这样的链接。”

提示或重要说明

如下所示。

# 章节

本书中有几个常见的标题（*准备开始*、*如何操作...*、*原理解析...*、*还有更多...* 和 *另见*）。

为了给出清晰的操作步骤，可以按如下方式使用这些部分。

## 准备开始

本节告诉你该在食谱中期待什么，并描述如何设置所需的软件或任何初步配置。

## 如何操作…

本节包含完成食谱所需的步骤。

## 原理解析…

本节通常会详细解释上一节发生的内容。

## 还有更多…

本节提供关于食谱的更多信息，帮助你更深入了解食谱内容。

## 另见

本节提供指向其他有用信息的链接，供食谱参考。

# 与我们联系

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何部分有疑问，请在邮件主题中提及书名，并通过 customercare@packtpub.com 给我们发送邮件。

**勘误**：虽然我们已尽力确保内容的准确性，但难免会有错误。如果你发现本书中的任何错误，请向我们报告。请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)，选择你的书籍，点击勘误提交表单链接，并填写相关信息。

**盗版**：如果你在互联网上发现任何非法的我们的作品副本，我们将不胜感激，如果你能提供该地址或网站名称。请通过 copyright@packtpub.com 联系我们，并附上相关链接。

**如果你有兴趣成为作者**：如果你在某个领域有专长，并且有意写书或为书籍贡献内容，请访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享你的想法

阅读完 *Atlassian DevOps 工具链食谱* 后，我们希望听到你的想法！请 [点击这里直接访问亚马逊评论页面](https://packt.link/r/1835463789)，分享你的反馈。

你的评论对我们和技术社区都非常重要，能帮助我们确保提供优质的内容。

# 下载本书的免费 PDF 版

感谢购买本书！

喜欢随时随地阅读，但无法携带纸质书籍？

你的电子书购买无法在你选择的设备上使用？

不用担心，现在每本 Packt 书籍都可以免费获得该书的无 DRM PDF 版本。

随时随地，在任何设备上阅读。可以从你最喜欢的技术书籍中搜索、复制并粘贴代码，直接应用到你的项目中。

优惠不仅仅是这些，你还可以每天通过邮箱独家获得折扣、新闻通讯和丰富的免费内容。

按照以下简单步骤即可获得好处：

1.  扫描二维码或访问以下链接

![](img/B21937_QR_Free_PDF.jpg)

[`packt.link/free-ebook/978-1-83546-378-9`](https://packt.link/free-ebook/978-1-83546-378-9)

1.  提交您的购买凭证

1.  就这些！我们将直接通过电子邮件发送您的免费 PDF 和其他福利

# 第一部分：开始循环

本书全面探索了 DevOps 及其工具的适配问题。特别关注的是 Atlassian 工具如何与彼此以及其他厂商的工具进行连接。

我们通过了解为何需要这种方法的背景和背景，设定了探索 DevOps 和工具链的舞台。一旦我们发现了 DevOps 的“为什么”，我们就将注意力转向 Jira 及其通过 Open DevOps 平台可以建立的连接。我们发现如何配置 Jira，以便通过该平台与其他工具连接。

在任何实施开始之前，开发人员需要与产品经理、利益相关者，甚至客户合作，确定新产品的需求、可行性和可行性。Jira Product Discovery 帮助确定哪些想法应该付诸实践，通过提供一个收集、跟踪、阐述和比较不同产品和特性想法的位置来辅助这一过程。那些准备好的想法可以与 Jira 问题链接，以监控开发进度。

在我们开始开发时，需要规划我们的工作，并确保我们的努力在正确的轨道上。同时，我们还需要确保开发工作与产品的整体大局及其价值承诺相连。为此，我们将 Jira 与 Confluence 连接，以便 Confluence 能够提供必要的项目文档。

本部分包含以下章节：

+   *第一章**，* *DevOps 介绍及 Atlassian 生态系统概述*

+   *第二章**，通过 Jira Product Discovery 发现客户需求*

+   *第三章**，* *使用 Confluence 进行规划和文档管理*
