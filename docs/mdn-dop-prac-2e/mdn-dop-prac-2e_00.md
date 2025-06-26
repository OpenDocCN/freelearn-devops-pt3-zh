# 前言

本书的新修订版超越了 DevOps 工具及其部署的基础知识，涵盖了实际示例，帮助你快速掌握容器、基础设施自动化、无服务器容器服务、持续集成和交付、自动化部署、部署流水线安全性以及在生产中运行服务的技能，所有这些都围绕容器和 GitOps 展开。

# 本书适用对象

如果你是软件工程师、系统管理员或运维工程师，想要进入公共云平台中的 DevOps 世界，那么本书适合你。当前的 DevOps 工程师也会发现本书非常有用，因为它涵盖了实施 DevOps 时的最佳实践、技巧和窍门，并以云原生的思维方式为重点。虽然不要求有容器化经验，但对软件开发生命周期和交付的基本理解将帮助你更好地从本书中获益。

# 本书涵盖内容

*第一章*，*现代 DevOps 方法*，深入探讨了现代 DevOps 的领域，强调其与传统 DevOps 的区别。我们将探讨推动现代 DevOps 的核心技术，特别强调容器在其中的核心作用。鉴于容器是相对较新的技术发展，我们将深入了解开发、部署和安全管理基于容器的应用程序的最佳实践和技术。

*第二章*，*使用 Git 和 GitOps 进行源代码管理*，介绍了 Git 这一领先的源代码管理工具及其在通过 GitOps 管理软件开发和交付中的应用。

*第三章*，*使用 Docker 进行容器化*，开启了我们对 Docker 的探索，包括安装、Docker 存储配置、启动初始容器，并通过 Journald 和 Splunk 进行 Docker 监控。

*第四章*，*创建和管理容器镜像*，剖析了 Docker 镜像，这是 Docker 使用中的一个关键组件。我们将了解 Docker 镜像、分层模型、Dockerfile 指令、镜像扁平化、镜像构建以及构建镜像的最佳实践。此外，我们还将探讨无发行版镜像及其在 DevSecOps 中的相关性。

*第五章*，*使用 Kubernetes 进行容器编排*，介绍了 Kubernetes。我们将使用 minikube 和 kinD 安装 Kubernetes，深入研究 Kubernetes 的架构基础，并探索 Kubernetes 的基本构建模块，如 Pods、容器、ConfigMaps、密钥、探针以及多容器 Pods。

*第六章*，*管理高级 Kubernetes 资源*，深入探讨了复杂的 Kubernetes 概念，包括网络、DNS、服务、部署、水平 Pod 自动缩放器以及有状态集。

*第七章*，*容器即服务（CaaS）和容器的无服务器计算*，探讨了 Kubernetes 的混合特性，架起了 IaaS 和 PaaS 的桥梁。此外，我们还将研究 AWS ECS 等无服务器容器服务，以及 Google Cloud Run 和 Azure 容器实例等替代方案。最后，我们将讨论 Knative，一种开源、云原生、无服务器技术。

*第八章*，*使用 Terraform 的基础设施即代码（IaC）*，介绍了使用 Terraform 的 IaC，阐明了其核心原则。我们将通过动手实践的例子，使用 Terraform 从头开始在 Azure 上创建资源组和虚拟机，同时掌握 Terraform 的基本概念。

*第九章*，*使用 Ansible 的配置管理*，使我们熟悉通过 Ansible 进行配置管理及其基本原理。我们将通过在 Azure 虚拟机上配置 MySQL 和 Apache 应用，来探索 Ansible 的关键概念。

*第十章*，*使用 Packer 的不可变基础设施*，深入探讨了使用 Packer 实现不可变基础设施的内容。我们将结合*第八章*，*使用 Terraform 的基础设施即代码（IaC）*，以及*第九章*，*使用 Ansible 的配置管理*，在 Azure 上启动基于 IaaS 的 Linux、Apache、MySQL 和 PHP（LAMP）栈。

*第十一章*，*使用 GitHub Actions 和 Jenkins 的持续集成*，从容器中心的角度解释了持续集成，评估了各种工具和方法，以持续构建基于容器的应用程序。我们将研究 GitHub Actions 和 Jenkins 等工具，分析何时以及如何使用每个工具，并在部署示例微服务架构的分布式应用——博客应用时加以应用。

*第十二章*，*使用 Argo CD 的持续部署/交付*，深入探讨了持续部署/交付，使用了 Argo CD 这一基于 GitOps 的现代持续交付工具。Argo CD 简化了容器应用的部署和管理。我们将利用它的功能来部署我们的示例博客应用。

*第十三章*，*保护和测试部署管道*，探讨了多种保护容器部署管道的策略，包括容器镜像分析、漏洞扫描、机密管理、存储、集成测试和二进制授权。我们将整合这些技术，以增强我们现有的 CI/CD 管道的安全性。

*第十四章*，*理解生产服务的关键性能指标（KPIs）*，介绍了站点可靠性工程，并调查了一系列关键性能指标，这些指标对于有效管理生产中的分布式应用至关重要。

*第十五章*，*使用 Istio 在生产中操作容器*，将带你了解广泛采用的服务网格技术 Istio。我们将探索在生产环境中进行日常操作的各种技术，包括流量管理、安全措施以及增强可观察性的技术，应用于我们示例中的博客应用。

# 如何最大限度地利用本书

本书所需的资源包括：

+   一个 Azure 订阅来进行一些练习：目前，Azure 提供一个为期 30 天、价值 $200 的免费试用；请在 [`azure.microsoft.com/en-in/free`](https://azure.microsoft.com/en-in/free) 注册。

+   一个 AWS 订阅：目前，AWS 提供一些产品的免费套餐。你可以在 [`aws.amazon.com/free`](https://aws.amazon.com/free) 注册。本书使用了一些付费服务，但我们将尽量在练习中最小化使用这些付费服务的数量。

+   一个 Google Cloud Platform 订阅：目前，Google Cloud Platform 提供一个免费的 $300 试用，试用期为 90 天，你可以在 [`console.cloud.google.com/`](https://console.cloud.google.com/) 注册。

+   对于某些章节，你需要克隆以下 GitHub 仓库以继续进行练习： [`github.com/PacktPublishing/Modern-DevOps-Practices-2e`](https://github.com/PacktPublishing/Modern-DevOps-Practices-2e)

    | **本书中涵盖的软件/硬件** | **操作系统要求** |
    | --- | --- |
    | Google Cloud Platform | Windows、macOS 或 Linux |
    | AWS | Windows、macOS 或 Linux |
    | Azure | Windows、macOS 或 Linux |
    | Linux 虚拟机 | Ubuntu 18.04 LTS 或更高版本 |

**如果你使用的是本书的数字版，我们建议你自己输入代码，或者从本书的 GitHub 仓库获取代码（链接将在下一节中提供）。这样做有助于避免与复制粘贴代码相关的潜在错误。**

# 下载示例代码文件

你可以从 GitHub 上下载本书的示例代码文件，链接为 [`github.com/PacktPublishing/Modern-DevOps-Practices-2e`](https://github.com/PacktPublishing/Modern-DevOps-Practices-2e)。如果代码有更新，将会在 GitHub 仓库中更新。

我们还提供了其他来自丰富图书和视频目录的代码包，链接为 [`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。快来看看吧！

# 使用的约定

本书中使用了若干文本约定。

`文本中的代码`：表示文本中的代码词汇、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 账号。示例如下：“让我们尝试发起一个拉取请求，将我们的代码从 `feature/feature1` 分支合并到 `master` 分支。”

一段代码块如下所示：

```
import os
import datetime
from flask import Flask
app = Flask(__name__)
@app.route('/')
def current_time():
  ct = datetime.datetime.now()
  return 'The current time is : {}!\n'.format(ct)
if __name__ == "__main__":
  app.run(debug=True,host='0.0.0.0')
```

任何命令行输入或输出如下所示：

```
$ cp ~/modern-devops/ch13/install-external-secrets/app.tf \
terraform/app.tf
$ cp ~/modern-devops/ch13/install-external-secrets/\
external-secrets.yaml manifests/argocd/ 
```

**粗体**：表示新术语、重要词汇或屏幕上显示的文字。例如，菜单或对话框中的文字通常会以 **粗体** 显示。示例如下：“点击 **创建拉取请求** 按钮来创建拉取请求。”

提示或重要说明

以这种方式出现。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请通过 [customercare@packtpub.com](https://customercare@packtpub.com) 发送电子邮件，并在邮件主题中注明书名。

**勘误表**：虽然我们已尽一切努力确保内容的准确性，但错误仍然可能发生。如果你在本书中发现错误，我们将非常感激你能向我们报告。请访问 [www.packtpub.com/support/errata](https://www.packtpub.com/support/errata) 并填写表单。

**盗版**：如果你在互联网上遇到任何形式的非法复制品，我们将非常感激你能提供位置地址或网站名称。请通过 copyright@packt.com 与我们联系，并附上材料链接。

**如果你有兴趣成为作者**：如果你在某个领域有专长，并且有兴趣写作或为一本书做贡献，请访问 [authors.packtpub.com](https://authors.packtpub.com)。

# 分享你的想法

一旦你阅读完 *现代 DevOps 实践*，我们很想听听你的想法！请 [点击这里直接进入亚马逊评论页面](https://packt.link/r/1805121820) 并分享你的反馈。

你的评论对我们和技术社区都非常重要，它将帮助我们确保提供优质的内容。

# 下载本书的免费 PDF 版本

感谢你购买本书！

你喜欢随时阅读，但无法随身携带印刷版书籍吗？

你的电子书购买无法兼容你选择的设备吗？

不用担心，现在每本 Packt 书籍都会附带免费的无 DRM PDF 版本。

随时随地，在任何设备上阅读。直接从你最喜爱的技术书籍中搜索、复制并粘贴代码到你的应用程序中。

优惠不仅仅如此，你还可以获得独家折扣、新闻通讯以及每日送到你收件箱的精彩免费内容。

按照以下简单步骤获得福利：

1.  扫描二维码或访问以下链接

![](img/B19877_QR_Free_PDF.jpg)

[`packt.link/free-ebook/9781805121824`](https://packt.link/free-ebook/9781805121824)

1.  提交你的购买凭证

1.  就这些！我们将直接通过电子邮件发送您的免费 PDF 和其他福利

# 第一部分：现代 DevOps 基础

本部分将向您介绍现代 DevOps 和容器的世界，并为容器技术打下坚实的知识基础。在本节中，您将了解容器如何帮助组织在云中构建分布式、可扩展且可靠的系统。

本部分包含以下章节：

+   *第一章*，*现代 DevOps 方法*

+   *第二章*，*使用 Git 和 GitOps 进行源代码管理*

+   *第三章*，*使用 Docker 进行容器化*

+   *第四章*，*创建和管理容器镜像*
