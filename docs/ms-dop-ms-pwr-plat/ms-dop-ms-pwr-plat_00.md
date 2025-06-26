# 前言

Microsoft Power Platform 是全球领先的低代码、无代码平台——一个现代化的应用运行时，能够实现无限数量的业务解决方案。 随着这些场景变得越来越复杂且业务关键，专业的 DevOps 流程需求也愈加迫切。 本书聚焦于定制软件开发项目中的著名实践，并将这些常见活动映射到 Microsoft Power Platform 工具集。 我们探讨软件开发生命周期的每个阶段，以及此软件即服务（SaaS）产品所提供的工具和功能，详细讨论诸如打包、构建、部署、测试和发布解决方案等常见的 DevOps 活动。 此外，我们深入探讨 DevSecOps 流程，并介绍在 Microsoft Power Platform 中融入安全性的开发实践。 您将了解现代 DevOps 工具，如 Azure DevOps 服务和 GitHub，以及如何为 Microsoft Power Platform 实施 DevOps 流程的不同方式。 通过正确的 DevOps 实施，我们的解决方案可以在高度监管的行业中以受控且 有序的方式运行。

我们坚信，低代码平台是为专业开发人员设计的。 Microsoft Power Platform 是一个快速应用开发框架，是唯一为公民开发者（创作者）提供 UI 界面的框架。 该平台是为专业开发人员和工程团队创建的，旨在减少业务应用程序的上市时间。 但是，这只有在专业的 DevOps 流程支持下才有效。

作为一种独特的方法，本书将这两个正交的领域结合在一起：一方面是能够构建复杂解决方案的低代码/无代码爱好者，另一方面是了解 DevOps 和应用生命周期管理的专业开发人员。 这些知识对您来说是一个独特的机会，亲爱的读者，能够帮助您建立起有力的竞争力， 在就业市场中占据优势。

作为最后的思考，随着生成性 AI 解决方案的崛起，开发者的工作将发生显著变化，重点将转向提示工程和更大的构建模块。 由于 AI 代理可以充当不同的角色（如开发者、测试人员或项目经理），并且仅凭提供的提示就能独立合成应用程序，因此我们不再编写代码行，而是制作组件。 这些组件可以与 Power Platform 提供的构建模块相对应。 考虑到 Power Platform 的 Copilot，新的应用程序构建时代刚刚开始，只有在与定制开发项目相同的 DevOps 流程得以实施时，才能获得成功。 开发项目。

# 本书适合的人群

本书面向对软件开发和 IT 运维领域感兴趣的专业人士，特别是那些希望了解**应用程序** **生命周期** **管理** (**ALM**)以及针对 Microsoft Power Platform 的 DevOps 流程的人士。 适合以下人群：

+   软件开发人员

+   DevOps 工程师

+   云架构师

+   站点可靠性工程师

+   测试人员

+   低代码工程师

本书对那些已经具备基本软件开发流程和生命周期工具理解的读者尤为有益。 此外，它也是那些对 Power Platform 的 ALM 和 DevOps 方面感兴趣的专业开发人员的绝佳资源。

# 本书内容

*第一章*，*掌握 DevOps 和 ALM 以提高软件开发效率*，概述了软件开发行业中最重要的突破，如敏捷、精益和 DevOps，并解释了先进的应用程序开发流程和模式，以及前沿的 DevOps 和 ALM 实践。

*第二章*, *Microsoft Power Platform 入门*，首先介绍了低代码/无代码开发方法，概述了微软 Power Platform 服务，并描述了该平台如何遵循各种治理和合规标准，使其成为一个适合企业的开发平台。 它展示了如何配置一个试用环境，可以用来配合书中的示例进行操作。 本章还深入探讨了管理 Power Platform 的工具以及开始构建业务应用程序的选项。  

*第三章*, *在微软 Power Platform 中探索 ALM 与 DevOps*，致力于建立 ALM 与 DevOps 与 Power Platform 之间的联系。 本章简要介绍了 Azure DevOps 和 GitHub。 为了帮助组织理解如何利用 Power Platform 来改善现有的应用程序组合，本章讨论了应用现代化选项。 本章最后通过连接能力成熟度模型指数（CMMI）和 Power Platform 采纳成熟度模型，帮助组织制定计划，在不同领域提高成熟度。  

*第四章*, *理解 Power Platform 环境与解决方案*，涵盖了 Power Platform 的基本构建块：环境和解决方案。 本章还讨论了环境策略、托管环境和 Power Platform 流水线。 本章最后通过一个实践实验室，指导我们如何利用 Power Platform 流水线构建第一个持续集成和持续交付流水线。  

*第五章*, *通过 DevOps 工具简化 Power Platform 开发*，更进一步，释放了超越 Power Platform 流水线的工具。 本章讨论了 Git、PAC CLI 和 Azure DevOps Services 流水线，结合 Power Platform 特定的构建任务和 GitHub 动作，来进行跨环境的 CI/CD 解决方案。 最后，本章将前一章中的 Power Platform 托管流水线结果与专业 DevOps 工具相结合，并直接从托管流水线中查看版本控制集成。 

*第六章*，*持续集成/持续部署（CI/CD）管道深度剖析*，描述了 DevOps CI/CD 流程的高级模式，如 Git 分支策略、Power Platform 的自动化测试框架以及 Power Platform Catalog 的包管理。 它讲解了 Azure DevOps 中的 YAML 管道和 GitHub 工作流，这些工作流分别通过使用各种 PAC CLI 命令、GitHub 动作和 Azure DevOps 构建任务，自动启动开发者分支和开发者环境。 此外，它讨论了 Power Platform 的 ALM 加速器，以及该解决方案如何利用管道模板、分支策略和环境管理等作为可复用的解决方案应用于 我们的项目。

*第七章*，*Power Platform 中的 DevSecOps 概述*，介绍了软件开发项目中 DevSecOps 的理论，并将与安全相关的活动映射到我们 Power Platform 项目的 DevOps 流程中。 它深入探讨了 GitHub 高级安全性和静态应用程序安全测试的 CodeQL。 它介绍了解决方案检查器，并展示了如何在有 Entra ID 组、服务主体和其他安全防护措施的情况下大规模启动 DevSecOps 项目。 最后，它通过对已建立流程的表面攻击和风险分析，给出了解决方案管理的建议 ，跨租户管理。

*第八章*，*展示 ALM 和 DevOps 实施*，深入探讨了 DevOps 和 ALM 原则的实际应用。 本章提供了一个现实世界示例——Power Platform 企业模板公共仓库中的 Kudos 应用——的动手练习，从 Git 分支策略到 CI 和 CD 管道，再到自动化测试、待办事项管理以及生产环境中应用的监控。 最后，本章介绍了功能标志来控制 功能发布。

*第九章*, *实施融合开发方法*，强调了构建融合团队的重要性，这些团队帮助组织实现目标。 本章展示了融合开发方法的示例，并讨论了 InnerSource 实践的重要性。 然后，继续探讨与 Microsoft Azure 云服务集成的可能性。 最后，通过展示一个常见的 Azure 和 Power Platform 集成场景来结束本章。

*第十章*, *在 Power Platform 中启用专业开发者扩展性*，继续解释专业开发者在为 Power Platform 开发时的各种可能性。 本章涵盖了集成场景中连接器的强大功能以及将配置与应用程序解耦的选项。 它深入探讨了可重用组件和自定义代码组件的强大功能。 本章展示了如何开发代码组件，并如何将 ALM 实践应用于代码组件。 本章最后通过探索 Power Pages 的应用生命周期管理来结束。

*第十一章*, *环境管理* *生命周期* *与设计最佳实践*，基于设计最佳实践。 本章讨论了 Power Platform 的架构设计及 Power Platform 的着陆区，解释了它们如何帮助组织构建遵循最佳实践并部署在有治理和安全环境中的应用工作负载。 接着，本章转向环境生命周期管理，并讨论了不同方法的自动化环境管理，包括基础设施即代码和 Terraform。 最后，本章通过探讨 Power Platform 卓越中心及 CoE 启动工具包如何帮助组织管理 Power Platform 租户。

*第十二章*，*与 Copilots、ChatOps 和 AI 注入应用程序展望未来*，是本书的最后一章，讨论了人工智能如何帮助组织丰富其应用程序。 它有助于理解如何使用各种 Copilot 来提高开发者的生产力，以及 AI Builder 或 Azure OpenAI 如何现代化现有的业务流程。 我们通过讨论 Copilot Studio 和它能够创建定制化 Copilot 来结束这一章和本书，Copilot 可以帮助组织使用定制的 AI 助手自动化 DevOps 流程。

# 要充分利用本书

你需要具备 Microsoft Power Platform 产品系列的基本了解，以及软件开发实践和 IT 应用管理、支持和监控的基础知识。 除了这些理论知识，你还需要安装以下软件组件，以便能够在本地执行提供的脚本 代码片段：

| **本书涉及的** **软件** | **操作系统要求** |
| --- | --- |
| Windows 子系统 适用于 Linux | Windows |
| Bash | Linux |
| .NET Framework 4.7.2 或更高版本 | Windows |
| Windows PowerShell 版本 5.x | Windows |
| Dotnet 6 | Windows 或 Linux |
| PAC CLI | Windows 或 Linux |
| Visual Studio Code | Windows 或 Linux |

请注意，脚本是为在 Azure DevOps 或 GitHub 的云托管代理上运行而创建的，本地执行仅用于 演示目的。

为了充分利用本书，你需要拥有 Power Platform 开发者许可证、Azure DevOps 或 GitHub 组织（公共仓库的免费计划足矣），以及一个 Microsoft Azure 租户。 不需要 Microsoft Azure 订阅，因为我们将仅使用 Microsoft Entra ID 功能来将 Power Platform 世界与我们首选的 DevOps 工具连接起来。 当然，如果你的组织有这些在线服务的企业级套餐，使用示例的体验将更加顺畅 且直接。

书中有许多示例，是为 Azure DevOps 服务和 GitHub 创建的。 这些章节中阐述的概念在这些 DevOps 工具之外是相同的，脚本可以轻松地在这些平台之间迁移。 我们强烈建议你在自己选择的 DevOps 工具中创建工作流，因此可以选择使用 Azure DevOps 或 GitHub 来进行书中的示例。 书中的样例。

在 *第七章*中，建议为 GitHub 或 Azure DevOps Services 安装 GitHub 高级安全插件。 尽管可以在本地执行 CodeQL 命令，但 Azure DevOps 和 GitHub 的高级安全功能仅在公共项目和公共仓库中可用。 也是如此。

**如果你正在使用本书的数字版本，我们建议你亲自输入代码，或者通过 GitHub 仓库（下一个章节提供链接）访问并 fork 代码。 这样做有助于你避免因复制粘贴代码而可能出现的错误。** **代码。**

你可以将现实世界的示例放在 *第八章* 的单独仓库中，将你在本书中学到的所有内容放入同一个解决方案中。 这样，你将拥有一个端到端的、生产就绪的 DevSecOps 参考实现，适用于 你的组织。

## 下载示例代码文件

你可以从 GitHub 下载本书的示例代码文件，地址为 [https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform](https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform)。如果代码有更新，将会在现有的 GitHub 仓库中更新。

我们还提供了其他来自我们丰富书籍目录的代码包，地址为 [https://github.com/PacktPublishing/](https://github.com/PacktPublishing/)。快来看看吧！

# 使用的约定

本书中使用了许多文本约定。

`文本中的代码`：指示文本中的代码词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟网址、用户输入和 Twitter 用户名。 例如： “我们将通过更改 `SingleLine.Text` 为数字（`Whole.None`）来更改 高亮的属性。”

代码块设置如下 ：

```
 pac admin help
pac admin list help
```

当我们希望特别引起您对代码块中特定部分的注意时，相关的行或项会以 粗体显示：

```
 codeql.exe database analyze .\codeql-database-js .\codeql-pack\ javascript\codeql\javascript-queries\0.8.12\codeql-suites\ javascript-code-scanning.qls --format=csv --output=results.csv
```

任何命令行输入或输出如下所示：

```
 pac solution create-settings --solution-zip .\EnvConnRef_1_1_managed.zip
```

**粗体**：表示一个新术语、一个重要的词汇或您在屏幕上看到的词汇。 例如，菜单或对话框中的词汇会这样显示。 例如：“这次我们选择 **新建** | **更多** | **环境变量**。”

提示或重要说明

显示如下。

# 联系我们

来自读者的反馈 总是欢迎的。

**一般反馈**：如果您对本书的任何部分有疑问，请在邮件主题中注明书名，并通过邮件联系我们 至 customercare@packtpub.com。

**勘误**：虽然我们已尽一切努力确保内容的准确性，但错误偶尔会发生。 如果您发现本书中的错误，我们将感激您向我们报告。 请访问 [www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)，选择您的书籍，点击勘误提交表单链接并填写 详细信息。

**盗版**：如果您在互联网上遇到任何我们作品的非法复制版本，我们将非常感激您提供该地址或网站名称。 请通过 copyright@packtpub.com 与我们联系，并提供相关材料的链接。

**如果您有兴趣成为作者**：如果您在某个主题上有专业知识，并且有兴趣撰写或参与撰写一本书，请 访问 [authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

阅读完 *《Mastering DevOps on Microsoft Power Platform》*后，我们非常希望听到您的反馈！ 请 [点击此处直接访问亚马逊的本书评论页面](https://packt.link/r/1835880851) 并分享 您的反馈。

您的评论对我们和技术社区都非常重要，它将帮助我们确保提供优质的 内容。

# 下载本书的免费 PDF 副本

感谢您购买 本书！

喜欢在路上阅读，但又无法随身携带印刷版 书籍吗？

您的电子书购买是否与您选择的设备不兼容？ 您的选择设备无法使用该电子书吗？

不用担心，现在购买每一本 Packt 图书，你都能免费获得该书的无 DRM PDF 版本 ，无需额外费用。

随时随地，任何设备都能阅读。 从你最喜欢的技术书籍中直接搜索、复制和粘贴代码到 你的应用程序中。

福利不止于此，您每天可以在 收件箱中获得独家折扣、新闻通讯和精彩的免费内容

按照这些简单的步骤获取 福利：

1.  扫描二维码或访问以下 链接

![](img/B22208_QR_Free_PDF.jpg)

[https://packt.link/free-ebook/978-1-83588-084-5](https://packt.link/free-ebook/978-1-83588-084-5)

1.  提交您的购买证明 购买证明

1.  就这样！ 我们将直接把您的免费 PDF 和其他福利发送到您的 电子邮件中

# 第一部分：理解微软 Power Platform 中的 DevOps

在本节中，我们将建立现代 DevOps 和软件开发方法所需的基础知识，并探讨过去十年低代码/无代码框架兴起的原因。 我们将深入探讨微软 Power Platform，概述其丰富的功能集，并将这些功能与已建立的软件开发生命周期方法进行关联。 此外，我们将研究 CMMI 模型的理论及其对 Power Platform 采纳成熟度模型发展的影响，强调这一模型在 我们实践中的重要性。

本部分包括以下章节： 以下是本部分的章节：

+   *第一章*, *高效软件开发中的 DevOps 与 ALM 精通*

+   *第二章*, *微软 Power Platform 入门*

+   *第三章*, *在微软 Power Platform 中探索 ALM 与 DevOps*
