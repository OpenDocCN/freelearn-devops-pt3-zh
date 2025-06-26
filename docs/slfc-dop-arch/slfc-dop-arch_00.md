# 前言

随着 Salesforce 平台向更加复杂的领域发展，架构师面临着对高级解决方案日益增长的需求。成功的 Salesforce 项目的关键在于有效的 DevOps 实践，本书通过提供有关 Salesforce 组件的战略性和实用性见解，帮助你实现这一目标。

本书首先从培养 DevOps 思维方式开始，重点关注协作、协调和沟通。它帮助你学习如何高效地展示治理、可视性和问责制。在此架构基础上，你将深入了解如何利用 SFDX 的功能规划你的策略。一旦你掌握了 Salesforce 的打包机制，你将学会如何使用免费的软件构建 CI/CD 堆栈，并将其配置为自动化交付变更。接着，你将解决成熟 DevOps 流程的运营问题，探索诸如票务管理、备份、变更监控和数据播种等主题。这些都是保持 Salesforce 组织健康和整洁的关键。最后，你将了解第三方解决方案的生态系统，这些解决方案提供开箱即用的能力，加速你的 Salesforce DevOps 之旅。

在本书结束时，你将彻底弄懂 Salesforce DevOps，能够以 DevOps 专业人士的身份交付 Salesforce 项目。

# 本书适合谁阅读

如果你是 Salesforce 架构师或资深开发人员，想要将 DevOps 最佳实践应用到你的项目中，那么这本书适合你。为了从本书中获益，你应该对 Salesforce 平台的开发（包括代码和低代码）有深入的了解，理解元数据、JSON 和 XML 等概念，并且熟练使用命令行操作。

# 本书内容概览

*第一章*，*Salesforce 变更部署简史*，讨论了在 Salesforce 平台上开发的独特挑战，并回顾了过去解决这些挑战的尝试。

*第二章*，*构建 DevOps 文化*，通过强调在技术之前建立文化思维方式的必要性，为 DevOps 的成功奠定了基础。

*第三章*，*源代码管理的价值*，探讨了实现 DevOps 的基础技术——源代码管理。

*第四章*，*测试你的变更*，重申了定期和早期测试的重要性，并探讨了实现这一目标的方法。

*第五章*，*使用 SFDX 的日常交付*，通过 Salesforce 的 SFDX 工具集，讲解了一个相对典型的开发和部署场景。

*第六章*，*探索打包*，介绍了 Salesforce 中另一种变更交付方式，回顾了不同类型的 Salesforce 打包。

*第七章*, *CI/CD 自动化*，深入探讨构建完整 CI/CD 流水线的强大 DevOps 自动化技术，并为流行平台提供示例。

*第八章*, *工单系统*，探讨了工单系统如何增强您的流程，并通过可管理的分块支持快速交付。

*第九章*, *数据和元数据备份*，强调了备份数据和元数据的重要性，并探讨了各种备份方法。

*第十章*, *监控变更*，讨论了跟踪整个 Salesforce 环境变更的重要性，确保没有遗漏任何变更。

*第十一章*, *在开发环境中进行数据播种*，演示了在 Salesforce 开发生命周期中准确测试数据的需求，以帮助准确测试。

*第十二章*, *Salesforce DevOps 工具 – Gearset*，开始我们对第三方 Salesforce DevOps 平台的探索，首先看看 Gearset。

*第十三章*, *Copado*，继续探讨 Salesforce DevOps 生态系统，我们转向 Copado。

*第十四章*, *Salesforce DevOps 工具 – Flosum*，转向 Salesforce DevOps 的下一个重要参与者，概述了 Flosum。

*第十五章*, *AutoRABIT*，完成了我们对最常见的第三方 Salesforce DevOps 工具的评估，重点介绍了 AutoRABIT 的功能。

*第十六章*, *其他 Salesforce DevOps 工具*，了解其他较不常见但仍然可选的 Salesforce DevOps 解决方案。

*第十七章*, *结论*，总结了我们学到的内容，并重申了重要的收获。

# 要充分利用本书

本书的性质假定您对 Salesforce 开发有所了解，这表明您可能已以某种方式部署了变更。本书不假设您具有现有的 DevOps 知识，因为这在书中有所涵盖。

| **本书涵盖的软件/硬件** | **操作系统要求** |
| --- | --- |
| Salesforce | Windows，macOS 或 Linux |
| Git |  |
| 其他软件或平台将根据章节逐个讨论 |  |

**如果您使用本书的数字版，我们建议您自行输入代码或从书的 GitHub 存储库中访问代码（下一节中提供链接）。这样做将有助于避免与复制粘贴代码相关的潜在错误。**

# 下载示例代码文件

您可以从 GitHub 上下载本书的示例代码文件，链接为[`github.com/PacktPublishing/Salesforce-DevOps-for-Architects`](https://github.com/PacktPublishing/Salesforce-DevOps-for-Architects)。如果代码有更新，它将在 GitHub 库中更新。

我们还提供来自丰富书籍和视频目录的其他代码包，链接为[`github.com/PacktPublishing/`](https://github.com/PacktPublishing/)。赶快去看看！

# 使用的约定

本书中使用了多种文本约定。

`文本中的代码`：表示文本中的代码单词、数据库表名、文件夹名称、文件名、文件扩展名、路径名、虚拟 URL、用户输入和 Twitter 用户名。以下是一个示例：“相关的`ContactId`和`OpportunityId`字段需要正确填写，以便关联记录。”

代码块设置如下：

```
from salesforce_bulk import SalesforceBulk
import csv
# Salesforce credentials
username = 'your_username'
password = 'your_password'
security_token = 'your_security_token'
```

当我们希望将注意力集中在代码块的特定部分时，相关的行或项会以粗体显示：

```
from salesforce_bulk import SalesforceBulk
import csv
# Salesforce credentials
username = 'your_username'
password = 'your_password'
security_token = 'your_security_token'
```

任何命令行输入或输出都如下所示：

```
sf force package create –n "My Managed Package" -t Managed –r force-app
```

**粗体**：表示一个新术语、一个重要的词汇，或者是你在屏幕上看到的单词。例如，菜单或对话框中的词语通常以**粗体**显示。以下是一个示例：“通过点击**立即获取**按钮在 AppExchange 列表中安装包，这将启动客户的 Salesforce 环境中的安装过程。”

提示或重要事项

显示如下。

# 联系我们

我们始终欢迎读者的反馈。

**一般反馈**：如果你对本书的任何方面有疑问，请通过电子邮件联系我们，地址是 customercare@packtpub.com，并在邮件主题中注明书名。

**勘误表**：尽管我们已尽力确保内容的准确性，但错误总会发生。如果您在本书中发现错误，我们将非常感谢您向我们报告。请访问[www.packtpub.com/support/errata](http://www.packtpub.com/support/errata)并填写表单。

**盗版**：如果你在互联网上发现我们作品的任何非法复制品，任何形式的都可以，如果能提供相关位置或网站名称，我们将不胜感激。请通过电子邮件联系版权@packt.com，并附上材料的链接。

**如果你有兴趣成为作者**：如果你在某个领域有专业知识，并且有兴趣编写或参与书籍创作，请访问[authors.packtpub.com](http://authors.packtpub.com)。

# 分享您的想法

一旦你阅读了*Salesforce DevOps for Architects*，我们很想听听你的想法！请[点击这里直接进入本书的 Amazon 评价页面](https://packt.link/r/1837636052)，并分享你的反馈。

您的评价对我们和技术社区都很重要，能帮助我们确保提供优质的内容。

# 下载本书的免费 PDF 副本

感谢您购买本书！

您是否喜欢随时随地阅读，但无法携带实体书？

你的电子书购买与选择的设备不兼容吗？

别担心，现在每本 Packt 书籍都附带免费的无 DRM 限制的 PDF 版本

随时随地，任意设备上阅读。可以搜索、复制并粘贴你最喜爱的技术书中的代码，直接应用到你的程序中

福利不仅仅如此，你还可以独享折扣、新闻通讯，并每天在邮箱中收到精彩的免费内容

按照以下简单步骤即可享受福利：

1.  扫描二维码或访问以下链接

![下载此书免费 PDF 版本的二维码](https://packt.link/free-ebook/9781837636051)

[`packt.link/free-ebook/9781837636051`](https://packt.link/free-ebook/9781837636051)

1.  提交你的购买凭证

1.  就这样！我们将直接把你的免费 PDF 及其他福利发送到你的邮箱
