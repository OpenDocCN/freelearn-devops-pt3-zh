# <st c="0">7</st>

# <st c="2">Power Platform 中的 DevSecOps 概述</st>

<st c="44">DevSecOps 是一种将安全实践融入整个</st> <st c="137">**软件开发生命周期**</st> <st c="174">(</st>**<st c="176">SDLC</st>**<st c="180">)的软件开发方法论，包括需求分析、规划与设计、开发、测试和质量保证，直到部署和维护。</st> <st c="314">Power Platform 是一个现代化的应用程序运行平台，使开发者能够构建自定义应用程序、自动化工作流、创建聊天机器人、设计和发布网站等。</st> <st c="504">该平台拥有强大的安全模型，确保数据保护并遵守行业标准（</st>*<st c="614">超过 90 个合规证书</st>*<st c="652">）；由 Azure DevOps 或 GitHub 建立的应用程序生命周期管理和 DevOps 流程，在处理任何类型的</st> <st c="838">开发项目时，默认部分考虑到这种安全整合。</st>

<st c="859">本章将介绍设置安全的 DevOps 开发项目所需的步骤和行动，涵盖多个环境以及</st> **<st c="1009">Microsoft Entra ID</st>**<st c="1027">支持的身份和访问管理。</st> <st c="1068">我们还将学习如何使用 DevOps 工具大规模启动基于 Power Platform 的开发项目，使开发人员和贡献者能够快速构建和部署应用程序，同时保持安全性和合规性标准。</st> <st c="1309">该平台还提供静态分析工具和报告，用于监控某些类型的安全威胁。</st> <st c="1416">这有助于我们在减少网络风险的同时实现高效的生产力。</st> <st c="1492">我们还将讨论使用开源库的自定义代码的安全影响，并将从</st> <st c="1634">安全的角度检查我们的 DevOps 工具。</st>

<st c="1655">本章我们将涵盖以下</st> <st c="1709">主要主题：</st>

+   <st c="1721">什么是 DevSecOps？</st>

+   <st c="1740">Power Platform 的</st> <st c="1759">安全模型</st>

+   <st c="1773">机密扫描和静态代码</st> <st c="1806">分析工具</st>

+   <st c="1820">解决方案检查器</st>

+   <st c="1837">大规模启动 DevSecOps 项目</st> <st c="1869">的实施</st>

+   <st c="1877">DevOps 流程的</st> <st c="1890">安全性</st>

# <st c="1906">技术要求</st>

<st c="1929">要深入了解 DevSecOps 方法和工具，我们需要具备</st> <st c="2007">以下内容：</st>

+   **<st c="2021">Power Platform 订阅</st>**<st c="2051">：我们可以注册 Power Apps 开发者计划（</st> [<st c="2102">https://www.microsoft.com/en-us/power-platform/products/power-apps/free</st>](https://www.microsoft.com/en-us/power-platform/products/power-apps/free)<st c="2174">），如果我们已经有 Microsoft Entra ID 工作帐户，或者我们可以加入 Microsoft 365 开发者</st> <st c="2275">计划（</st> [<st c="2284">https://developer.microsoft.com/en-us/microsoft-365/dev-program</st>](https://developer.microsoft.com/en-us/microsoft-365/dev-program)<st c="2348">）</st>

+   **<st c="2350">Azure DevOps 服务组织</st>**<st c="2387">：我们可以随时创建一个免费的 DevOps 组织（</st> [<st c="2445">https://learn.microsoft.com/en-us/azure/devops/user-guide/sign-up-invite-teammates</st>](https://learn.microsoft.com/en-us/azure/devops/user-guide/sign-up-invite-teammates)<st c="2528">）。</st>

+   **<st c="2531">GitHub 账户和公共</st>** **<st c="2561">仓库</st>**<st c="2571">：</st> [<st c="2574">https://github.com/signup</st>](https://github.com/signup)

+   **<st c="2599">GitHub 高级安全</st>**<st c="2624">：此功能对公共仓库免费开放；</st> 参见 [<st c="2679">https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security#about-advanced-security-features</st>](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security#about-advanced-security-features)<st c="2803">。</st>

+   **<st c="2804">示例和演练</st>** <st c="2830">本章讨论的内容可以在以下位置找到</st> <st c="2869">： </st> [<st c="2875">https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform/tree/main/Chapter07</st>](https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform/tree/main/Chapter07)

# <st c="2974">什么是 DevSecOps？</st>

**<st c="2993">DevSecOps</st>** <st c="3003">代表</st> <st c="3010">的是</st> **<st c="3015">开发、安全和运维</st>** <st c="3052">以及这些领域的协作，以交付安全的应用程序。</st> <st c="3122">它的原则是在定义 DevOps 过程和方法论后不久建立的。</st> <st c="3213">这个理念相同，就是将</st> **<st c="3248">左移</st>** <st c="3258">的思维方式应用于安全领域，这与我们在质量保证和后来的运维中应用左移思维方式类似。</st> <st c="3270">它是一个框架，将安全性整合到 SDLC 的所有阶段，从这个角度看，它是软件开发组织在处理安全问题时的一次演进，旨在通过引入“安全第一”的思维文化，并在软件开发生命周期的每个阶段（从设计到交付）自动化安全。</st> <st c="3391">组织广泛采用这种方法，减少发布含有安全漏洞的代码的风险，不仅避免了声誉损失，还避免了因</st> <st c="3899">此类失败而造成的高额财务损失。</st>

<st c="3913">如果我们回顾在</st> *<st c="3986">第一章</st>* <st c="3995">中学习到的 SDLC 的主要阶段（需求分析、规划与设计、开发、测试与质量保证、部署和维护），那么我们可以将安全活动和自动化映射到这些阶段，如下图所示：</st>

![图 7.1 – SDLC 中的安全性](img/B22208_07_1.jpg)

<st c="4873">图 7.1 – SDLC 中的安全性</st>

<st c="4902">这些步骤和活动</st> <st c="4930">以及它们的持续改进构成了</st> **<st c="4977">安全开发生命周期</st>** <st c="5008">(</st>**<st c="5010">SDL</st>**<st c="5013">)。</st> <st c="5017">SDL 是一种</st> <st c="5030">软件开发过程，帮助开发人员构建和设计更安全的软件，满足安全合规要求，同时降低开发成本。</st> <st c="5201">它在各项服务中标准化了安全最佳实践，并最终交付更</st> <st c="5288">安全的产品。</st>

<st c="5304">让我们详细查看这些活动，并通过 SDLC 分析相应的</st> <st c="5400">安全范围。</st>

## <st c="5416">设置</st>

<st c="5422">在项目的</st> <st c="5433">初始阶段，我们重点关注两个主要的</st> <st c="5486">安全话题：</st>

+   **<st c="5502">库存管理</st>** <st c="5523">是</st> <st c="5531">安全性的重要元素之一，因为它 catalogizes 我们为客户提供的应用程序和服务。</st> <st c="5648">这是我们的</st> **<st c="5658">应用目录</st>** <st c="5669">，其中包含</st> <st c="5674">我们的应用程序（</st>**<st c="5702">产品</st>**<st c="5711">）和</st> <st c="5718">服务（</st>**<st c="5728">服务树</st>**<st c="5742">）在生产中的资产（源代码位置、发布的二进制文件、版本号、Azure DevOps 项目、Azure 管道、GitHub 项目、GitHub 工作流等）在我们组织内部。</st> <st c="5941">借助库存管理，我们可以轻松找到受安全漏洞影响的组件的应用程序所有者。</st> <st c="6083">在具有成熟 DevSecOps 流程的组织中，库存管理是完全自动化的，这意味着每当启动一个新项目时，就会创建一个新的目录项，并触发工作流来启动 Azure DevOps 或 GitHub 项目、Git 仓库、Microsoft Entra ID 组、服务主体、自托管代理、管道等。</st>

+   **<st c="6429">访问控制管理</st>** <st c="6455">指的是</st> <st c="6462">从工程角度访问应用程序工件，以及相应的底层 DevOps 工具。</st> <st c="6586">我们需要定义框架</st> <st c="6617">和</st> **<st c="6626">身份与访问管理</st>** <st c="6656">(</st>**<st c="6658">IAM</st>**<st c="6661">) 解决方案，用于授予产品所有者、工程师、测试人员、架构师、发布经理和运营团队在与开发项目交互时的访问权限。</st> <st c="6835">我们还需要定义如何处理条件访问策略，例如</st> <st c="6924">强制</st> **<st c="6935">多因素认证</st>** <st c="6962">(</st>**<st c="6964">MFA</st>**<st c="6967">) 以访问生产环境，以及我们在与特权身份管理</st> <st c="7077">（</st>**<st c="7081">PIM</st>**<st c="7111">）合作时的程度， 例如，向高特权用户或全局管理员提供“即时”特权访问 Microsoft Entra ID 和 Azure 资源。</st> <st c="7270">启用 PIM 后，这些高度特权的活动会被监控并</st> <st c="7344">审计。</st> <st c="7358">值得注意的是，这种访问控制并非我们要开发的应用程序或服务用于身份验证和授权的控制方式。</st>

## <st c="7511">计划和设计</st>

在设置好项目的适当访问权限和库存管理后，我们从安全角度深入探讨*计划与设计*阶段：

+   **<st c="7695">威胁建模</st>** 被用来识别我们系统的潜在威胁，通过潜在攻击者的视角，以及他们对我们的应用程序的可能性和影响，在实施任何缓解措施之前进行评估。<st c="7887">它是一个典型的风险管理过程，专注于安全。</st> <st c="7949">市场上有多种工具可用于建模威胁，例如</st> <st c="8022">微软威胁建模工具</st>（请参见*进一步阅读*部分）。<st c="8093">作为冲刺计划过程的一部分，我们可以进行威胁建模演练，并根据风险分析来决定是否应缓解某些威胁，并将这些活动作为产品待办事项，标记为安全类别。</st>

+   `<st c="8730">SecureString</st>` 类型用于处理我们代码中的凭证。

+   **<st c="8782">同行评审指南</st>** 用于制定同行评审过程：评审将何时执行，执行的频率以及是否会涉及安全专家。</st> <st c="8959">如今，DevOps 流程已经规定了在有拉取请求时必须进行代码审查，DevOps 工具也会详细记录和支持这些发现。</st>

+   **<st c="9108">预提交钩子</st>** 定义了在提交代码更改到代码库之前，我们执行哪些安全检查，例如，在提交之前检查提交内容是否包含敏感信息。</st> <st c="9300">由于时间限制，预提交钩子只能执行静态代码分析规则集的一小部分，但它们是</st> <st c="9424">防止安全漏洞的必要部分，能立即阻止不安全的提交。</st> <st c="9499">我们可以</st> <st c="9505">使用</st> **<st c="9510">IDE 安全插件</st>** 来执行本地的轻量级安全分析。</st>

## 提交（CI）

在通过了<st c="9610">预提交钩子和代码审查结果后，下一阶段是执行 CI 构建和相关的安全活动，作为拉取请求验证过程的一部分：</st>

+   **<st c="9778">依赖管理</st>**<st c="9800">：如今，定制开发的应用程序在很大程度上依赖于</st> <st c="9859">开源软件包。</st> <st c="9881">目前应用程序中，平均 90%的代码包含基于外部软件包或</st> <st c="9982">库的组件。</st> <st c="9994">术语</st> **<st c="10003">软件供应链管理</st>** <st c="10035">非常常见，因为我们在应用中使用的这些软件包也使用其他第三方组件，而这些组件又使用其他包，依此类推，形成了一个链条。</st> <st c="10212">如果不刷新我们的供应链，识别和暴露这些第三方包中的漏洞将使我们的应用变得脆弱。</st> <st c="10353">像</st> **<st c="10367">GitHub Dependabot</st>** <st c="10384">这样的工具可以</st> <st c="10388">识别我们供应链中的漏洞（</st>*<st c="10439">继承的漏洞</st>*<st c="10465">），并自动创建 Dependabot 警报，发起拉取请求，将更改提交回我们的主分支。</st> <st c="10579">Dependabot 检查可以在</st> <st c="10620">CI 构建过程中执行。</st>

+   **<st c="10630">静态应用安全测试</st>** <st c="10666">(</st>**<st c="10668">SAST</st>**<st c="10672">)：这是对代码进行静态分析和安全扫描</st> <st c="10723">以确保安全性。</st> <st c="10737">市场上有几种工具可以用来对我们的代码库进行这种分析，例如 SonarCloud、Fortify 或带有 CodeQL 的 GitHub 高级安全。</st> <st c="10914">这些工具可以集成到我们的 CI 管道中，以便进一步自定义。</st> <st c="10993">静态代码分析结果的输出为</st> **<st c="11040">静态分析结果交换格式</st>** <st c="11082">(</st>**<st c="11084">SARIF</st>**<st c="11089">)开放标准数据。</st> <st c="11112">基于 SARIF 结果，系统会自动创建代码扫描警报，</st> <st c="11132">并且它们可以作为待办事项进行跟踪。</st> <st c="11224">在使用 GitHub 企业版高级安全功能的情况下，我们还可以使用 GitHub Copilot 来自动修复我们的安全问题，</st> <st c="11371">实现完全自动化。</st>

+   **<st c="11387">安全单元和功能测试</st>**<st c="11424">：这些是维护我们代码库高质量的其他方式，旨在避免实现</st> <st c="11520">安全漏洞。</st>

+   **<st c="11545">安全管道</st>**<st c="11562">：我们需要谨慎关注自动化管道的安全性：谁可以访问哪些管道，管道机密信息存储在哪里（Azure Key Vault），以及我们是否可以使用托管标识来执行工作流程。</st> <st c="11930">在 CI 构建期间，所谓的</st> **<st c="11958">软件材料清单</st>** <st c="11985">(</st>**<st c="11987">SBOMs</st>**<st c="11992">)会被创建，以跟踪构建管道中使用的组件。</st> <st c="12059">SBOM 是软件组件的列表，包括它们的层次关系、版本号、依赖关系和许可要求，通常以 JSON 格式呈现。</st> <st c="12225">SBOM 有助于透明度和供应链的可追溯性声明，并帮助符合要求的启用。</st> <st c="12251">托管池可以用于构建和签署代码，确保构建的完整性和组件治理。</st> <st c="12365">构建成功后，代码会存储在安全的</st> <st c="12469">构件存储中。</st>

+   **<st c="12543">凭证扫描</st>**<st c="12563">：我们</st> <st c="12568">可以使用工具扫描我们的代码库，以减少凭证泄露的风险。</st>

## <st c="12673">部署（CD）</st>

<st c="12685">在这个阶段，我们</st> <st c="12703">查看代码库的整体健康状况，除了在提交阶段检查的内容（这些内容也可以在部署阶段重复检查）。</st> <st c="12855">我们在发布过程中引入了额外的安全检查：</st>

+   **<st c="12922">动态应用安全测试</st>** <st c="12959">(</st>**<st c="12961">DAST</st>**<st c="12965">)是一种评估软件应用安全性的技术，它在应用程序运行时进行。</st> <st c="12999">它涉及使用 DAST 工具对 Web 应用程序进行模拟的网络攻击，以识别 SQL 注入、跨站脚本攻击和不安全配置等漏洞。</st> <st c="13061">这种安全测试模拟恶意攻击者的行为，以发现实时应用程序中的安全弱点，使开发人员能够在应用发布之前解决这些问题。</st> <st c="13280">安全管道</st> <st c="13497">是为了避免凭证泄露或在生产环境中执行恶意代码。</st>

+   **<st c="13508">云配置验证</st>** <st c="13539">是验证云资源配置是否正确、安全，并符合最佳实践和政策的过程。</st> <st c="13590">这可以通过自动化工具完成，这些工具扫描云资源的配置文件和设置，并报告任何问题或违规行为（例如生产环境中的配置更改，或</st> <st c="13893">IaC 脚本与实际运行配置之间的不一致）。</st> <st c="13941">验证有助于防止配置错误、安全漏洞和合规问题，确保云资源的正确设置和</st> <st c="14084">管理。</st>

+   **<st c="14102">基础设施即代码</st>** <st c="14125">(</st>**<st c="14127">IaC</st>**<st c="14130">)</st> **<st c="14133">扫描</st>** <st c="14141">是分析 IaC 文件以查找</st> <st c="14183">安全漏洞、配置错误以及是否符合最佳实践和政策的过程。</st> <st c="14278">IaC 扫描工具可以检测政策违规并提出修复建议。</st> <st c="14373">甚至可以使用基于 LLM 的工具提示 GPT-4 模型，检测 IaC 文件中的政策违规，并为任何违规行为提供修复建议。</st> <st c="14519">违规和修复会以</st> <st c="14562">SARIF 文件形式报告。</st>

+   **<st c="14573">安全验收测试</st>** <st c="14601">是验证系统在发布或部署前是否满足已定义的</st> <st c="14661">安全要求和标准的过程。</st> <st c="14733">这种测试通常在新系统、升级或部署时进行，包括对信息安全要求的测试以及遵循安全系统开发实践的测试。</st> <st c="14934">验收测试活动可以在单个组件或集成系统上进行，验证活动可以在测试环境中进行，以确保系统不会引入漏洞。</st> <st c="15158">在选择测试数据或来自运营中信息系统的数据时，必须小心，确保没有捕获任何个人身份信息、密码等机密数据或哈希值。</st> <st c="15368">该数据集。</st>

<st c="15381">在成功部署到测试环境并随后部署到生产环境后，SBOM 和证据存储用于跟踪</st> <st c="15505">版本发布。</st>

## <st c="15518">操作与监控</st>

<st c="15538">在这个阶段，成功将我们的应用程序发布到生产环境后，我们需要关注以下</st> <st c="15648">安全相关的活动：</st>

+   **<st c="15676">持续监控</st>** <st c="15698">是</st> <st c="15702">持续跟踪、评估和审查信息系统的安全控制和风险的过程。</st> <st c="15823">这一过程有助于确保安全控制随时间保持有效，并确保系统或其环境的任何变化不会引入新的漏洞。</st> <st c="15998">持续监控可以通过自动化工具和过程来实现，这些工具和过程收集并分析来自各种来源的数据，如日志、网络流量和系统配置。</st> <st c="16179">然后，这些数据被用来识别潜在的安全问题，并生成可以用来提高系统安全性的警报或报告。</st> <st c="16318">该系统的安全性。</st>

+   **<st c="16329">审计日志</st>** <st c="16340">是</st> <st c="16345">用于安全检测的监控对象。</st> <st c="16380">进行管理员监控和扫描，以确保系统的安全性。</st> <st c="16462">我们可以考虑我们解决方案的审计日志，就像我们的 DevOps 工具一样。</st> <st c="16540">审计日志对于理解和检测可能来自可疑来源的变化至关重要。</st> <st c="16619">可疑来源。</st>

+   **<st c="16638">威胁情报</st>** <st c="16658">是</st> <st c="16665">收集、分析和传播当前及潜在网络安全威胁信息的实践。</st> <st c="16782">它帮助组织主动检测和应对安全威胁。</st> <st c="16865">Microsoft 威胁情报是一个云服务，利用全球收集的信号上的机器学习算法来检测和缓解安全风险。</st> <st c="17025">该基础设施用于支持 Azure 服务、传统产品、端点（设备）以及其他企业资产。</st> <st c="17151">其目的是保护全球基础设施，支持如 Microsoft Azure 和 Power Platform 等服务。</st> **<st c="17268">安全事件和事件管理</st>** <st c="17306">(</st>**<st c="17308">SIEM</st>**<st c="17312">)工具，如 Microsoft Sentinel 或 Microsoft Defender for Cloud，利用审计日志通过不同的机器学习算法来检测异常和不寻常的模式。</st> <st c="17470">学习算法。</st>

+   **无责事后分析** 是一种以非惩罚性的方式分析事件或故障的过程，目的是识别根本原因并改进系统和流程，以防止类似事件再次发生。重点在于学习和改进，而不是指责或惩罚。无责事后分析鼓励开放和诚实的沟通，允许团队成员分享他们的错误并从中学习，而不必担心遭受报复。这种方法培养了持续改进的文化，帮助团队构建更具弹性的系统。

在接下来的部分中，我们将看到我们可以为 Power Platform 解决方案应用哪些活动，以及哪些任务由 Power Platform 的运维团队提供。

现在让我们继续了解 Power Platform 的安全模型，学习其安全设计和原则。

# Power Platform 的安全模型

微软 Power Platform 运行在微软 Azure 上，受益于在该超大规模平台上可用的所有 PaaS 安全服务，如 Microsoft Sentinel、Microsoft Defender for Cloud、Microsoft Entra 的全面审计日志等。

微软 Power Platform 的安全模型建立在**最小权限访问**（**LPA**）的原则之上，这有助于创建具有更精细访问控制级别的应用程序。Power Platform 利用微软身份平台来授权所有 API 请求，采用广泛认可的 OAuth 2.0 协议。此外，作为 Power Platform 基础数据层的 Dataverse 拥有一个全面的安全模型，涵盖了环境级安全、基于角色的安全以及记录和字段级安全、安全角色、通过安全组和应用用户的业务单元和团队，正如我们在*第四章*中所了解的。

此外，Power Platform 构建在不同的环境中，这些环境是我们应用程序、流程、连接和其他组件的容器，并具有在创建这些环境时定义的地理位置的安全性和数据访问管理能力。

<st c="19633">Power Platform 的架构基于零信任安全原则，</st> *<st c="19720">永不信任，总是验证</st>* <st c="19746">的哲学。</st> <st c="19759">零信任策略</st> **<st c="19761">要求对每个用户和</st> <st c="19780">设备进行严格的身份验证，无论其位置如何（公司内网或外部边界），并且每次访问公司资源时都会进行此检查。</st> <st c="19985">这种方法通过微软身份平台和 Microsoft Entra ID 的能力得以实现。</st> <st c="20100">每次访问 Power Platform 服务端点时，都需要有效的访问令牌（承载令牌）才能执行该端点。</st> <st c="20229">在服务器端，每次请求到达端点时，都会将访问令牌发送到身份提供者（即 Microsoft Entra ID）以验证其有效性。</st> <st c="20386">这些经过数字签名的访问令牌包含关于调用者的声明，用于身份验证和授权目的。</st> <st c="20516">在识别出调用者后，Power Platform 会结合该端点确定其授权级别。</st> <st c="20635">在此，考虑到自定义和内置安全角色、业务单元层级以及团队分配，这些都是</st> <st c="20742">应用级授权。</st>

<st c="20775">下图</st> <st c="20797">展示了 Power Platform 在一个 Microsoft</st> <st c="20859">Azure 区域中的架构：</st>

![图 7.2 – Power Platform 架构](img/B22208_07_2.jpg)

<st c="21283">图 7.2 – Power Platform 架构</st>

<st c="21323">一组网页</st> <st c="21341">前端服务器包含托管在 Azure 应用服务环境中的 ASP.NET 网站。</st> <st c="21430">应用服务环境</st> <st c="21456">提供了底层的专用计算单元，应用服务计划在其上部署。</st> <st c="21547">应用服务计划可以托管一个或多个应用程序（网站）。</st> <st c="21611">这些都是 Azure 应用服务的一部分。</st> <st c="21648">当访问 Power Platform 服务或应用时，客户端的 DNS 会将请求定向到最近的数据中心，具体由 Azure Traffic Manager 确定。</st> <st c="21811">Azure Traffic Manager 提供了几种路由和负载均衡选项，在 OSI 模型的第 4 层进行，例如性能路由（按延迟选择最接近的端点）、加权轮询和基于地理位置的路由（按</st> <st c="22048">地理位置选择最接近的端点）。</st>

<st c="22071">前端集群负责处理登录和身份验证过程。</st> <st c="22161">一旦用户通过验证，他们会收到一个 Microsoft Entra ID 访问令牌（OAuth2 令牌）。</st> <st c="22249">ASP.NET 系统会检查令牌，以识别用户的组织（后台使用的是</st> *<st c="22390">多租户</st>* <st c="22401">帐户类型的 Azure 应用注册）。</st> <st c="22421">然后，它与 Power Platform 全球后端服务进行通信，通知浏览器该组织租户所在的具体后端集群。</st> <st c="22584">之后，客户端与后端集群之间的所有进一步交互都会直接发生，绕过</st> <st c="22683">网页前端。</st>

<st c="22696">对于静态资源，如 JavaScript、CSS 和图像文件，浏览器通常会从</st> <st c="22797">**Azure 内容分发**</st> **<st c="22802">网络</st>** <st c="22832">(</st>**<st c="22834">CDN</st>**<st c="22837">) 获取它们。</st>

<st c="22840">Power Platform 服务的核心</st> <st c="22857">运行在</st> <st c="22891">**后端集群**</st> <st c="22915">中，提供服务端点、后台服务、数据库、缓存和其他各种元素。</st> <st c="23018">后端集群遍布多个 Azure 区域。</st> <st c="23068">单个区域可能容纳多个集群（在前面的图中表示为</st> **<st c="23134">规模组</st>** <st c="23145">），使 Power Platform 服务能够超越单个集群的垂直和水平扩展能力，进行水平扩展。</st>

<st c="23309">这些后端集群旨在</st> *<st c="23352">保持状态</st>*<st c="23360">，承载它们所服务的所有租户的完整数据集。</st> <st c="23423">存储某个租户数据的特定集群被称为该租户的主集群。</st> <st c="23511">Power Platform 全球后端服务将有关用户主集群的详细信息提供给 Web 前端集群，后者会将请求定向到适当的主后端集群。</st> *<st c="23699">每个环境都驻留在一个规模组内</st>*<st c="23745">，这是一个集体基础设施，旨在提供可扩展且可维护的资源集合。</st> <st c="23848">一个规模组容纳多个客户组织，每个组织拥有自己的数据库，同时共享服务基础设施。</st> <st c="23983">此设置利用了多种 Azure 服务，例如 Azure SQL、Azure 虚拟机和 Azure Cache for Redis。</st> <st c="24103">这些规模组成对建立，分别对应客户选择的区域。</st> <st c="24199">例如，选择美国作为区域时，会在西部美国和东部美国区域形成规模组。</st> <st c="24338">虽然租户元数据和数据通常驻留在集群内，但数据也可以复制到位于同一 Azure 地理区域中配对区域的次要后端集群。</st> <st c="24540">该次要集群在区域性中断期间作为应急备用，否则保持不活动状态。</st> <st c="24644">此外，分散在集群虚拟网络内各种机器上的微服务也有助于</st> <st c="24761">后端的功能。</st>

<st c="24785">DevSecOps 流程和活动由 Power Platform 产品团队实现并运营，我们将其视为</st> <st c="24913">这部分</st> **<st c="24921">软件即服务</st>** <st c="24942">(</st>**<st c="24944">SaaS</st>**<st c="24948">)提供的一部分。</st> <st c="24961">这意味着产品工程团队已经建立了从库存管理、访问控制、提交前钩子、同行评审、SAST 和 DAST 分析、安全管道、IaC 扫描到通过威胁情报进行的依赖性分析等一系列控制和防护措施，涵盖了他们的开发流程。</st> <st c="25271">这一方法经过第三方独立审计员的定期审计和认证。</st> <st c="25366">在开发过程中，这些团队还使用 Microsoft SDL 来确保编码最佳实践和威胁建模。</st> <st c="25480">由此得出的结论是，我们需要专注于平台的定制化和可扩展性点，因为这些是</st> <st c="25617">我们可能引入</st> <st c="25648">漏洞的地方。</st> <st c="25682">这些包括但不限于</st> <st c="25715">以下内容：</st>

+   <st c="25729">环境</st> <st c="25742">访问管理</st>

+   <st c="25759">自定义开发的</st> **<st c="25777">Power Apps 组件框架</st>** <st c="25807">（</st>**<st c="25809">PCF</st>**<st c="25812">）组件</st>

+   <st c="25825">Dataverse 自定义工作流活动和 Dataverse Web 资源（HTML</st> <st c="25897">和 JavaScript）</st>

+   <st c="25912">Dataverse 配置和 Dataverse</st> <st c="25952">自定义开发的插件</st>

+   <st c="25976">Power</st> <st c="25983">Fx 表达式</st>

+   <st c="25997">Power Pages 自定义代码（HTML、JavaScript、</st> <st c="26041">和 Liquid）</st>

+   <st c="26052">我们自己的 DevOps 流程与 CI/CD 自动化脚本</st>

+   <st c="26111">解决方案和</st> <st c="26125">库存管理</st>

+   <st c="26145">Fusion 架构组件</st> <st c="26180">在 Microsoft Azure 中</st>

<st c="26195">在了解 Power Platform 的安全模型及从安全角度看该 SaaS 产品的优势后，让我们进一步了解可以用于静态代码分析、依赖性检查和</st> <st c="26414">密钥扫描的工具。</st>

# <st c="26430">密钥扫描和静态代码分析工具</st>

<st c="26477">尽管市场上有许多其他 SAST 工具，</st> **<st c="26534">GitHub 高级安全性</st>** <st c="26558">（</st>**<st c="26560">GHAS</st>**<st c="26564">）提供了</st> <st c="26576">最全面的静态安全代码分析功能，同时支持 Copilot 安全功能。</st> <st c="26685">GHAS 功能</st> <st c="26699">对于私有</st> <st c="26729">GitHub 代码库如下：</st>

+   **<st c="26749">代码扫描</st>** <st c="26763">使用</st> **<st c="26769">CodeQL</st>** <st c="26775">或其他</st> <st c="26785">您喜欢的工具来查找漏洞</st> <st c="26830">和编码错误。</st> <st c="26849">结果以 SARIF 格式存储，并在 GitHub 的代码库级别进行管理。</st>

+   **<st c="26938">CodeQL CLI</st>** <st c="26949">是一个独立的</st> <st c="26965">工具，我们可以用它扫描代码库中的漏洞和编码错误。</st> <st c="27048">该</st> **<st c="27052">CodeQL CLI</st>** <st c="27062">与前述的（代码扫描）一起使用，或者可以在其他 DevOps 工具中使用，例如 GitLab</st> <st c="27182">或 Jenkins。</st>

+   **<st c="27193">密钥扫描</st>** <st c="27209">查找</st> <st c="27216">代码库中的秘密、密钥和敏感令牌。</st> **<st c="27275">预提交钩子</st>** <st c="27291">也可用来阻止本地提交</st> <st c="27334">在到达代码库并创建新</st> <st c="27384">历史记录之前。</st>

+   **<st c="27399">自定义自动分类规则</st>** <st c="27424">有助于</st> <st c="27431">大规模协调</st> <st c="27448">你的</st> **<st c="27454">Dependabot 提示</st>** <st c="27471">。使用自定义自动分类规则，你可以决定忽略、推迟或触发 Dependabot 安全</st> <st c="27610">更新。</st>

+   **<st c="27621">依赖性审查</st>** <st c="27639">帮助我们</st> <st c="27649">在将不安全的依赖引入我们的代码库之前进行识别</st> <st c="27704">并提供有关许可证、依赖项和依赖项年龄的信息。</st>

<st c="27800">这些高级安全功能在每个公共的 GitHub 仓库中都可以使用，无需额外的</st> <st c="27894">费用。</st>

<st c="27909">GHAS for Azure DevOps 于 2023 年 9 月正式发布。</st> <st c="27978">此产品提供的代码扫描、密钥扫描和依赖扫描功能与 GitHub 企业版中的功能相同。</st> <st c="28107">GitHub 工程团队已与 Azure DevOps 团队分享这一功能集，就像几年前 Azure DevOps 团队将构建代理（托管运行器）功能提供给 GitHub</st> <st c="28316">产品组一样。</st>

<st c="28330">哪种工具，何时使用？</st>

<st c="28348">我们只能在 Azure DevOps 中使用 GHAS 来处理 Azure Git 仓库。</st> <st c="28415">如果我们有一个附加到 Azure DevOps 流水线的 GitHub 仓库，我们需要在 GitHub 端使用 GHAS。</st> <st c="28522">Azure DevOps 中的 GHAS 构建任务将</st> <st c="28565">无法工作。</st>

<st c="28574">GHAS 基于</st> <st c="28592">的</st> **<st c="28596">代码查询语言</st>** <st c="28615">(</st>**<st c="28617">CodeQL</st>**<st c="28623">)，可以用于构建不同编程语言的代码分析数据库。</st> <st c="28717">CodeQL 支持 C、C++、C#、Java、Go、Kotlin、JavaScript、Python、Ruby、Swift 和 TypeScript。</st> <st c="28811">CodeQL 将代码视为数据，比传统的静态分析工具更有保障地发现潜在的安全漏洞。</st> <st c="28955">通过创建一个与代码库相匹配的 CodeQL 数据库，你可以对这个数据库执行 CodeQL 查询，以查明代码中的问题。</st> <st c="29104">查询结果将显示为</st> *<st c="29135">代码扫描警报</st>* <st c="29155">，当你在 GitHub 和 Azure DevOps 中使用 CodeQL 进行</st> <st c="29208">代码扫描时。</st>

<st c="29222">CodeQL 在底层创建了一个关系数据库。</st> <st c="29276">每种语言都有自己的架构，CodeQL</st> <st c="29321">使用</st> **<st c="29326">提取器</st>** <st c="29336">(每种语言都有独特的提取器)来读取代码文件和编译后的二进制文件（脚本语言除外），并建立代码表达式、抽象语法树、数据流图和控制流图的层次表示。</st> <st c="29568">这些构建块存储在数据库中，安全查询会针对这些表执行，以发现特定语言的漏洞和编码错误。</st> <st c="29733">CodeQL 的数据库是我们执行分析时的快照。</st> <st c="29817">GitHub 提供了适用于任何支持语言的查询包。</st> **<st c="29874">查询包</st>** <st c="29885">由</st> <st c="29893">组成</st> **<st c="29897">CodeQL 套件</st>** <st c="29910">(</st>**<st c="29912">qls</st>**<st c="29915">) 是查询语言文件集（</st>**<st c="29957">ql 文件</st>**<st c="29966">）。</st> <st c="29970">这些查询套件帮助我们立即受益于 GitHub 的安全知识和专业技术，并在我们的代码库上执行这些代码扫描规则。</st> <st c="30136">我们还可以使用 Visual Studio Code 和 CodeQL 扩展编写自己的查询，并可以从现有查询套件中派生，增加我们自己的检查（Windows 驱动验证团队就是这样做的，引入了他们自己的自定义</st> <st c="30375">规则：</st> [<st c="30381">https://docs.microsoft.com/en-us/windows-hardware/drivers/devtest/static-tools-and-codeql</st>](https://docs.microsoft.com/en-us/windows-hardware/drivers/devtest/static-tools-and-codeql)<st c="30471">）。</st>

<st c="30474">CodeQL CLI 是 GHAS 服务的一部分，对学生和学术人员免费。</st> <st c="30573">作为独立工具的 CodeQL CLI 支持 macOS、Linux 和 Windows 操作系统，我们可以将其作为二进制文件下载（请参见</st> *<st c="30710">进一步阅读</st>*<st c="30725">）。</st>

<st c="30728">我们可以使用 CodeQL CLI 在本地、Azure DevOps 构建代理，或我们的</st> <st c="30838">GitHub 执行器上执行代码分析。</st>

<st c="30853">Azure DevOps 和 GitHub 都提供围绕 CodeQL CLI 的构建任务和操作，便于将代码扫描功能轻松集成到现有的流水线</st> <st c="31008">和工作流中：</st>

![图 7.3 – DevOps 工具中的 CodeQL CLI 包装器](img/B22208_07_3.jpg)

<st c="31065">图 7.3 – DevOps 工具中的 CodeQL CLI 包装器</st>

<st c="31113">由于</st> <st c="31120">在 Power Platform 中有许多地方可以引入自定义代码，建议首先了解如何使用 CodeQL CLI 以及如何设置管道和工作流来执行不同的查询包。</st> <st c="31355">最好的地方是</st> [<st c="31373">找到</st><st c="31376">Power Platform 的自定义代码在</st> `<st c="31417">Pow</st>`](https://github.com/microsoft/PowerApps-Samples)`<st c="31420">erApps-Samples</st>` <st c="31435">仓库，地址是</st> [<st c="31450">https://github.com/microsoft/PowerApps-Samples</st>](https://github.com/microsoft/PowerApps-Samples)<st c="31496">。让我们检查一下本章中的 GitHub 仓库中的自定义</st> `<st c="31629">runcodeql_javascript.ps1</st>` <st c="31654">文件：</st>

1.  <st c="31691">我们可以下载 CodeQL CLI 二进制文件（见</st> *<st c="31737">深入阅读</st>* <st c="31752">|</st> *<st c="31755">CodeQL CLI 二进制文件</st>*<st c="31774">），并将解压后的文件夹位置添加到我们的</st> <st c="31821">环境路径中。</st>

1.  <st c="31838">然后我们克隆该仓库并导航到</st> <st c="31882">目标文件夹：</st>

    ```
    <st c="31896">git clone https://github.com/microsoft/PowerApps-Samples</st>
    ```

1.  <st c="31953">让我们下载 JavaScript</st> <st c="32000">和 TypeScript 的查询包：</st>

    ```
    <st c="32015">codeql.exe pack download codeql/javascript-queries --dir ./codeql-pack/javascript</st>
    ```

1.  <st c="32097">让我们为 JavaScript/TypeScript 创建 CodeQL 数据库（目前只有一个 JS/TS 提取器）：</st>

    ```
    <st c="32356">codeql-pack</st> and export the results in a CSV file:

    ```

    codeql.exe database analyze .\codeql-database-js .\codeql-pack\javascript\codeql\javascript-queries\0.8.12\codeql-suites\<st c="32527">javascript-code-scanning.qls</st> --format=csv --output=results.csv

    ```

    ```

1.  <st c="32590">当然，我们可以分析代码并以</st> <st c="32649">SARIF 格式获取结果：</st>

    ```
    <st c="32900">codeql github upload-results</st> to upload our SARIF file to one of our GitHub repositories (we cannot upload back the results to the original repository due to lack of privileges). We need to create a PAT token that grants us access to the repo (<st c="33143">security_event</st>):

    ```

    # 上传结果到 GitHub 仓库

    $env:GH_PAT = "ghp_PAT TOKEN"

    $env:GH_PAT | & codeql.exe github upload-results `

            --repository=ourrepo/test `

            --ref=refs/heads/main `

            --commit 18cd21585b94dd16c48dc13bc1365269696a75a4 `

            --sarif=javascript.sarif --github-auth-stdin

    ```

    <st c="33429">We also need to have a commit hash ID to which we upload the</st> <st c="33491">SARIF result.</st>
    ```

<st c="33504">对于 C# 项目和解决方案，CodeQL 命令略有不同。</st> <st c="33595">例如，当</st> <st c="33614">我们需要实现 Dataverse 插件时，可以使用 C# 查询包，但在这种情况下，我们还需要在分析过程中构建 Visual Studio 项目（见</st> `<st c="33776">runcodeql_dotnet.ps1</st>` <st c="33796">文件，位于</st> `<st c="33804">Chapter07</st>` <st c="33813">文件夹下，在</st> <st c="33828">GitHub 仓库中）：</st>

```
 codeql.exe pack download codeql/csharp-queries --dir ./codeql-pack/csharp
codeql.exe database create --language=csharp --source-root . codeql-database <st c="33993">--command "dotnet build .\dataverse\DiscoveryService\DiscoveryService.sln"</st> --overwrite
codeql.exe database analyze .\codeql-database .\codeql-pack\csharp\codeql\csharp-queries\0.8.12\codeql-suites\csharp-code-scanning.qls --format=csv --output=csharp-results.csv
```

<st c="34255">我们使用</st> `<st c="34267">--command</st>` <st c="34276">参数来为一种或多种编译语言创建数据库。</st> <st c="34343">它需要我们用来构建解决方案的</st> `<st c="34358">build</st>` <st c="34363">命令，例如</st> `<st c="34406">dotnet build <<sln 文件路径>></st>`<st c="34443">。如果我们使用的是 Python</st> <st c="34502">或 TS/JS，则不需要此选项。</st>

<st c="34511">在大规模代码库的情况下，我们可以使用以下技巧来优化 CodeQL</st> <st c="34596">的运行时：</st>

+   **<st c="34610">查询包</st>**<st c="34622">：我们</st> <st c="34627">可以创建自己的查询包，基于公开可用的查询包，也可以创建自己的查询套件，专注于不同的代码扫描和安全扫描分析，这取决于我们在流水线中的时间。</st> <st c="34854">例如，在拉取请求验证构建期间，我们希望通过专注于最关键的安全漏洞来最小化分析的执行时间，而构建主分支时则会执行所有可用的</st> <st c="35092">查询套件。</st>

+   `<st c="35135">codeql database analyze</st>`<st c="35158">：如果我们希望使用多个线程运行查询，可以使用</st> `<st c="35176">--thread</st>` <st c="35184">参数</st> <st c="35195">。默认值为</st> `<st c="35271">1</st>`<st c="35272">，这意味着分析仅在一个线程中运行。</st> <st c="35324">我们可以指定更多的线程来加速查询执行。</st> <st c="35381">要将线程数设置为逻辑处理器的数量，我们可以将此参数设置为</st> `<st c="35458">0</st>` <st c="35459">。</st>

+   **<st c="35477">仅针对部分代码库执行 CodeQL</st>**<st c="35531">：理想情况下，我们可以拥有拉取请求流水线/工作流，这些工作流不仅包含拉取请求发生的触发条件，还可以</st> <st c="35672">包含</st> **<st c="35677">路径过滤器</st>** <st c="35689">来过滤源代码中的变更。</st> <st c="35719">这样，CodeQL 只会针对源代码层次结构中已变更的部分执行。</st> <st c="35726">通过这种方式，CodeQL 仅在源代码中已更改的区域运行。</st>

<st c="35857">最后，我们深入了解可用的 Azure 流水线构建任务</st> <st c="35921">在</st> `<st c="35952">.pipelines/codeql.yml</st>` <st c="35973">文件中，该文件位于</st> `<st c="35981">Chapter07</st>` <st c="35990">文件夹下的</st> <st c="36005">GitHub 仓库中：</st>

1.  <st c="36018">让我们</st> <st c="36025">导入</st> [<st c="36033">https://github.com/microsoft/PowerApps-Samples</st>](https://github.com/microsoft/PowerApps-Samples) <st c="36079">到我们的 Azure DevOps 项目中。</st> <st c="36119">我们需要进入</st> **<st c="36136">代码库</st>** <st c="36141">并点击包含所有仓库的下拉列表，在该列表中我们可以找到</st> **<st c="36237">导入仓库</st>** <st c="36254">操作项：</st>

![图 7.4 – 导入 Git 仓库](img/B22208_07_4.jpg)

<st c="36429">图 7.4 – 导入 Git 仓库</st>

<st c="36465">在这里，我们可以添加 GitHub 公共仓库的 URL 并</st> <st c="36521">导入</st> `<st c="36528">PowerApps-Samples</st>`<st c="36545">。</st>

1.  <st c="36546">通过打开项目设置启用</st> **<st c="36558">Advanced Security</st>** <st c="36575">功能（需要项目管理员权限），然后选择最近导入的仓库。</st> <st c="36713">我们</st> <st c="36716">需要启用</st> **<st c="36731">Advanced Security</st>** <st c="36748">，方法是切换</st> <st c="36761">按钮：</st>

![图 7.5 – 在 Azure DevOps 中启用高级安全功能](img/B22208_07_5.jpg)

<st c="37430">图 7.5 – 在 Azure DevOps 中启用高级安全功能</st>

1.  <st c="37499">启用此安全功能后，我们可以导航到</st> **<st c="37557">Pipelines</st>**<st c="37566">，然后我们可以创建一个新的管道 YAML 文件，该文件将存储在新的</st> <st c="37642">Git 仓库中。</st>

1.  <st c="37657">YAML 文件应如下所示：</st>

    ```
     trigger:
    - none
    pool:
      vmImage: ubuntu-latest
    steps: <st c="37748">- task: PowerPlatformToolInstaller@2</st>
     <st c="37784">inputs:</st>
     <st c="37792">DefaultVersion: true</st>
    <st c="37813">- task: AdvancedSecurity-Codeql-Init@1</st>
     <st c="37852">inputs:</st>
     <st c="37860">languages: 'javascript'</st>
     <st c="37884">querysuite: 'code-scanning'</st>
    <st c="37912">- task: AdvancedSecurity-Codeql-Analyze@1</st>
     <st c="37954">inputs:</st>
     <st c="37962">WaitForProcessing: true</st>
    ```

    <st c="37986">此</st> <st c="37991">YAML 文件将执行与之前使用 PowerShell 对 JavaScript 进行的相同的 CodeQL 分析，通过</st> <st c="38108">CodeQL CLI。</st>

1.  <st c="38119">代码扫描和密钥扫描警报列在</st> **<st c="38182">Repos</st>** <st c="38187">下的</st> **<st c="38195">Advanced Security</st>** <st c="38212">选项卡中，位于</st> <st c="38220">Azure DevOps 中：</st>

![图 7.6 – 在 Azure DevOps 中的代码和密钥扫描结果](img/B22208_07_6.jpg)

<st c="39290">图 7.6 – 在 Azure DevOps 中的代码和密钥扫描结果</st>

<st c="39351">在 GitHub 的情况下，我们可以通过分叉</st> `<st c="39414">PowerApps-Samples</st>` <st c="39431">并创建</st> <st c="39445">与代码分析相关的操作（请参阅</st> `<st c="39484">.github/workflows/codeql.yml</st>` <st c="39512">文件，位于</st> `<st c="39520">Chapter07</st>` <st c="39529">文件夹中的</st> <st c="39544">GitHub 仓库中）：</st>

```
 name: "CodeQL"
on:
  workflow_dispatch:
jobs:
  analyze:
    name: Analyze javascript
    runs-on: ubuntu-latest
    timeout-minutes: 120
    permissions:
      # required for all workflows
      security-events: write
      # only required for workflows in private repositories
      actions: read
      contents: read
    steps: <st c="39835">- name: Checkout repository</st>
 <st c="39862">uses: actions/checkout@v4</st>
 <st c="39888">- name: Initialize CodeQL</st>
 <st c="39914">uses: github/codeql-action/init@v3</st>
 <st c="39949">with:</st>
 <st c="39955">languages: javascript-typescript</st>
 <st c="39988">build-mode: none</st>
 <st c="40005">- name: Perform CodeQL Analysis</st>
 <st c="40037">uses: github/codeql-action/analyze@v3</st>
 <st c="40075">with:</st>
 <st c="40081">category: "/language:javascript-typescript"</st>
```

<st c="40125">使用</st> <st c="40135">Azure DevOps 管道模板和可重用的 GitHub 工作流，我们可以轻松地在每个项目中引入这些额外的代码扫描和安全分析步骤。</st> <st c="40312">幸运的是，还有另一种方法可以在 Power Platform 解决方案中注入安全验证和代码扫描检查，那就是</st> **<st c="40443">解决方案检查器</st>**<st c="40459">。</st>

# <st c="40460">解决方案检查器</st>

<st c="40477">解决方案检查器</st> <st c="40498">利用 Power Apps 检查服务，通过提交任务到 Power Platform 后端执行代码分析。</st> <st c="40628">有</st> <st c="40638">预定义的</st> **<st c="40649">规则集</st>** <st c="40657">和</st> **<st c="40663">规则</st>** <st c="40668">用于覆盖我们解决方案的某些安全建议和编码最佳实践。</st> <st c="40755">解决方案检查器可以以 SARIF 格式报告找到的问题，我们可以轻松将其上传到我们的 DevOps 工具中，例如使用 GitHub 的 GHAS 或 Azure DevOps 的 GHAS。</st> <st c="40920">解决方案检查器检查以下 Power Platform 资产</st> <st c="40988">中的非托管解决方案：</st>

+   <st c="41008">Dataverse 自定义</st> <st c="41026">工作流活动</st>

+   <st c="41045">Dataverse Web 资源（HTML</st> <st c="41076">和 JavaScript）</st>

+   <st c="41091">Dataverse 配置，例如 SDK</st> <st c="41130">消息步骤</st>

+   <st c="41143">Power Automate 流程（通过 Power Automate</st> <st c="41185">Flow Checker）</st>

+   <st c="41198">Power Fx 表达式（通过应用检查器 – Power Apps</st> <st c="41262">检查服务的一部分）</st>

<st c="41278">规则集及其规则是预定义并分类的，涵盖了之前列出的组件：插件或工作流活动、Web 资源以及画布应用。</st> <st c="41443">我们可以通过多种方式执行</st> <st c="41461">解决方案检查器，以检查</st> <st c="41500">非托管解决方案：</st>

+   <st c="41520">在 Power Apps 创建者门户的</st> **<st c="41562">解决方案</st>** <st c="41571">面板中。</st>

+   <st c="41578">使用</st> `<st c="41589">pac solution check</st>` <st c="41607">命令，带上正确的参数，例如</st> <st c="41652">如下：</st>

    ```
    <st c="41833">pac solution check</st> call on that blob using <st c="41944">Microsoft.PowerApps.Checker.PowerShell</st>, and respectively the <st c="42005">Get-PowerAppsCheckerRulesets</st> cmdlet for fetching the pre-built rulesets and the <st c="42085">Invoke-PowerAppsChecker</st> cmdlet for submitting the analysis job to the Power Platform backbone. However, these cmdlets are in the pre-release version and they might change.
    ```

+   <st c="42256">使用</st> <st c="42263">以下</st> **<st c="42267">Power Apps 检查器 Web API</st>**<st c="42293">：Power Apps 创建者门户、PAC CLI 和 PowerShell 模块都依赖于这些 REST API 端点。</st> <st c="42412">我们可以直接调用这些端点，前提是有适当的访问令牌；例如，</st> [<st c="42467">美国端点可通过以下</st>](https://unitedstates.api.advisor.powerapps.com) <st c="42514">URL 访问：</st> <st c="42559">以下是</st> [<st c="42564">https://unitedstates.api.advisor.powerapps.com</st>](https://unitedstates.api.advisor.powerapps.com)<st c="42610">。</st>

+   <st c="42611">使用</st> **<st c="42618">托管环境</st>** <st c="42638">在导入解决方案到目标环境之前强制执行解决方案检查器运行。</st> <st c="42723">在此，我们可以阻止导入解决方案，如果遇到违反这些规则的情况</st> <st c="42800">。</st>

+   <st c="42816">利用</st> `<st c="43186">PowerPlatformChecker@2</st>`<st c="43209">) 在从 Git 仓库导入解决方案的过程中。</st> <st c="43274">此任务在后台使用 PAC CLI。</st>

<st c="43319">让我们来看一下在 Azure 管道中执行静态代码分析的构建任务，该任务分析我们的</st> <st c="43425">解决方案（见</st> `<st c="43439">.pipelines/solution-checker.yml</st>` <st c="43470">在</st> `<st c="43478">第七章</st>` <st c="43487">文件夹中的</st> <st c="43502">GitHub 仓库）：</st>

```
 trigger:
- none
pool:
  vmImage: ubuntu-latest
variables:
 solutionName: "PacktCopilotSolution"
steps:
- task: PowerPlatformToolInstaller@2
  inputs:
    DefaultVersion: true
- task: PowerPlatformPackSolution@2
  inputs:
    SolutionSourceFolder: '$(System.DefaultWorkingDirectory)/src/$(solutionName)'
    SolutionOutputFile: '$(Build.ArtifactStagingDirectory)/Solution/$(solutionName).zip'
    SolutionType: 'Unmanaged' <st c="43915">- task: PowerPlatformChecker@2</st>
 <st c="43945">inputs:</st>
 <st c="43953">authenticationType: 'PowerPlatformSPN'</st>
 <st c="43992">PowerPlatformSPN: 'dev-US_XXX_Y'</st>
 <st c="44025">FilesToAnalyze: '$(Build.ArtifactStagingDirectory)/Solution/$(solutionName).zip'</st>
 <st c="44106">RuleSet: '0ad12346-e108-40b8-a956-9a8f95ea18c9'</st>
<st c="44328">PowerPlatformChecker@2</st> task to perform the solution check. The <st c="44391">RuleSet</st> GUID is the solution checker ruleset. There is only one other and that is AppSource certification for submitting solutions for Microsoft AppSource. We need to define a service connection, as we did in *<st c="44600">Chapter 6</st>*, to connect to an environment because the <st c="44652">AppChecker</st> service job executes solution validations in environments. Although we reference here a local unmanaged solution, that solution is uploaded to the environment before the analysis. The results are not stored in the environment but exported as a SARIF file. The build task also publishes the SARIF result to the artifacts of pipeline results.
			<st c="45003">In the case of GitHub, we can use the following workflow to achieve the same result (see</st> `<st c="45093">.github/workflows/solution-checker.yml</st>` <st c="45131">in the</st> `<st c="45139">Chapter07</st>` <st c="45148">folder of the</st> <st c="45163">GitHub repo):</st>

```

名称：解决方案检查器

触发：

workflow_dispatch:

作业：

solutioncheck:

    在：ubuntu-latest

    环境：

    解决方案：mpa_ITBase

    步骤：

    - 使用：actions/checkout@v3

    - 名称：安装 Power Platform 工具

        使用: microsoft/powerplatform-actions/actions-install@v1 <st c="45422">- 名称: 打包未管理的解决方案</st>

<st c="45453">使用: microsoft/powerplatform-actions/pack-solution@v1</st>

<st c="45508">使用：</st>

<st c="45514">解决方案文件夹: ${{ env.Solution }}</st>

<st c="45551">解决方案文件: ${{ env.Solution }}_unmanaged.zip</st>

<st c="45600">解决方案类型：未管理</st>

<st c="45625">- 名称：检查解决方案</st>

<st c="45648">使用: microsoft/powerplatform-actions/check-solution@v1</st>

<st c="45704">使用：</st>

<st c="45710">环境网址: https://yourorg.crm.dynamics.com</st>

<st c="45760">应用程序 ID: 862e5a17-d38b-BBBB-FFFF-88a77f59623f</st>

<st c="45805">客户端密钥：“${{ secrets.CLIENTSECRET_DEV }}”</st>

<st c="45854">租户 ID：4ae51f31-033a-XXXX-YYYY-5ece14d2c081</st>

<st c="45962">microsoft/powerplatform-actions/pack-solution@v1</st>，还将 SARIF 结果上传到 GitHub 工作流运行的工件中。

            <st c="46086">管理发现</st>

            <st c="46108">无论是使用带有 SARIF 结果的解决方案检查器、GHAS，还是其他代码和依赖扫描工具执行 SAST，在收集结果后，我们还需要管理这些发现。</st> <st c="46302">我们可以通过接受这些风险来减轻风险，例如在测试代码中的安全警报不被修正，或者我们可以引入新的问题或工作项，根据发现和相应描述的漏洞</st> <st c="46547">在</st> **<st c="46555">常见弱点枚举</st>** <st c="46582">(</st>**<st c="46584">CWE</st>**<st c="46587">) 数据库中。</st> <st c="46600">现代 DevOps 工具支持将代码扫描、依赖扫描和密钥扫描警报链接到工作项或问题，并在开发</st> <st c="46781">生命周期中引入修复需求。</st>

            <st c="46792">现在我们已经熟悉了解决方案检查器、GHAS 功能以及与 SDLC 方法论相关的 DevSecOps 流程中的任务，我们将这些知识与来自</st> *<st c="47002">第六章</st>*<st c="47011">的自动化 DevOps 流程结合起来。</st>

            <st c="47012">大规模启动 DevSecOps 项目</st>

            <st c="47052">在</st> *<st c="47056">第六章</st>*<st c="47065">中，我们学习了如何为我们的 Git 仓库分配解决方案并相应地设置底层分支来启动 Power Platform 环境。</st> <st c="47213">我们在 GitHub 中创建了管道模板和可重用的工作流，以提供以下自动化功能：</st>

                +   <st c="47317">启动新的 Power Platform</st> <st c="47349">开发者环境</st>

                +   <st c="47371">为这些 Power</st> <st c="47416">Platform 环境创建服务连接</st>

                +   <st c="47437">从开发者环境导出解决方案到</st> <st c="47489">Git 分支</st>

                +   <st c="47501">通过拉取请求或直接将解决方案从 Git 仓库和分支导入目标环境，或导入到我们的</st> <st c="47617">开发分支</st>

            <st c="47633">我们可以在这段旅程中迈出一步，自动化整个 Power Platform 解决方案的开发过程。</st> <st c="47752">首先，让我们引入一个新术语：工作负载。</st> **<st c="47798">工作负载</st>** <st c="47807">由一个或多个相互配合的解决方案组成，旨在实现复杂的业务需求。</st> <st c="47915">我们可以将它们视为包部署器包的输入解决方案，例如我们在</st> **<st c="48002">企业模板</st>** <st c="48022">中学到的内容，在</st> *<st c="48043">第四章</st>*<st c="48052">中学到的内容。要为工作负载或独立解决方案设置一个企业级的 DevSecOps 项目，我们需要完成以下额外的自动化任务：</st>

                1.  <st c="48207">为所有者和贡献者创建 AAD 组（Microsoft Entra ID 组）。</st> <st c="48283">必须至少有两个所有者在</st> `<st c="48324">所有者</st>` <st c="48330">组中。</st> <st c="48338">在大多数情况下，我们建议指定一个预算所有者和一个技术所有者来负责</st> <st c="48422">项目</st><st c="48431">。</st>

                1.  <st c="48432">为每个工作负载创建测试和生产环境，或者使用现有的共享环境来托管更多的解决方案</st> <st c="48565">。</st>

                1.  <st c="48573">为这些环境创建</st> <st c="48581">服务主体，使用</st> `<st c="48629">pac admin create-service-principal</st>` <st c="48663">和之前创建的</st> `<st c="48938">所有者</st>` <st c="48944">AAD 组</st> <st c="48955">。</st>

                1.  <st c="48971">创建一个 Azure DevOps 项目，并将</st> `<st c="49018">所有者</st>` <st c="49024">AAD 组分配给</st> `<st c="49081">贡献者</st>` <st c="49093">组，</st> **<st c="49107">贡献者</st>** <st c="49119">组分配给</st> <st c="49129">Azure DevOps。</st>

                1.  <st c="49142">为工作负载或解决方案在 Azure DevOps 项目中创建一个 Git 仓库，并设置分支策略</st> <st c="49245">。</st>

                1.  <st c="49254">为这些环境创建服务连接，使用我们创建的服务主体。</st> <st c="49356">这正是我们在</st> *<st c="49379">第六章</st>*<st c="49388">中所做的。</st>

                1.  <st c="49389">创建使用位于单独专用库中的管道模板的 CI/CD 管道，并配置它们以使用这些新的服务连接</st> <st c="49543">作为参数。</st>

            <st c="49557">最终，我们可以实现以下的</st> <st c="49599">项目设置：</st>

            ![图 7.7 – DevSecOps Power Platform 项目](img/B22208_07_7.jpg)

            <st c="49942">图 7.7 – DevSecOps Power Platform 项目</st>

            <st c="49987">我们可以在此看到</st> <st c="49998">这里的</st> `<st c="50320">所有者</st>` <st c="50327">和</st> `<st c="50332">贡献者</st>`<st c="50344">) 可以在项目中工作。</st> <st c="50372">成员变更被记录到</st> **<st c="50407">Azure Log Analytics 工作区</st>** <st c="50437">并</st> <st c="50442">由 SIEM 解决方案监控，如 Microsoft Sentinel（</st><st c="50499">威胁情报</st>）。</st>

            <st c="50521">我们可以借助</st> <st c="50574">的帮助</st> **<st c="50578">Azure 管道</st>** <st c="50593">位于一个单独的 Azure DevOps 项目中。</st> <st c="50638">让我们详细查看这些步骤</st> <st c="50661">：</st>

                1.  <st c="50671">创建</st> <st c="50679">AAD 组：</st>

    <st c="50690">要创建 AAD 组，我们可以使用以下脚本（参见</st> `<st c="50752">create-aad-group.sh</st>` <st c="50771">在</st> `<st c="50779">第七章</st>` <st c="50788">文件夹中的</st> <st c="50803">GitHub 仓库）：</st>

    ```
     #!/bin/bash
    set -e
    # Variables
    SERVICE_PRINCIPAL_APP_ID="<your-service-principal-app-id>"
    SERVICE_PRINCIPAL_SECRET="<your-service-principal-password>"
    TENANT_ID="<your-tenant-id>"
    GROUP_NAME="<your-group-name>"
    # Login as the service principal <st c="51061">az login --service-principal -u $SERVICE_PRINCIPAL_APP_ID -p $SERVICE_PRINCIPAL_SECRET --tenant $TENANT_ID</st> # Create the AAD group <st c="51191">az ad group create --display-name $GROUP_NAME --mail-nickname $GROUP_NAME</st> # get the user object id <st c="51290">AADObjectID=$(az ad user show \</st>
     <st c="51321">--id $userPrincipalName \</st>
     <st c="51347">--query id \</st>
     <st c="51360">--output tsv)</st> # add a member to the group <st c="51727">pac admin create</st> commands to create Power Platform developer environments. To create a sandbox or a production environment, we can use the following script (see <st c="51888">create-powerplatform-env.sh</st> in the <st c="51923">Chapter07</st> folder of the GitHub repo):

    ```

    #!/bin/bash

    set -e

    # 定义环境变量

    SERVICE_PRINCIPAL_APP_ID="<your-service-principal-app-id>"

    SERVICE_PRINCIPAL_SECRET="<your-service-principal-password>"

    TENANT_ID="<your-tenant-id>"

    ENV_NAME="YOUR_ENV_NAME"

    REGION="YOUR_REGION"

    CURRENCY="YOUR_CURRENCY"

    LANGUAGE="YOUR_LANGUAGE"

    # 使用服务主体登录到 Power Apps <st c="52302">pac auth create --applicationId $SERVICE_PRINCIPAL_APP_ID --clientSecret $SERVICE_PRINCIPAL_SECRET \</st>

    <st c="52402">--tenant $TENANT_ID</st> # 创建一个新环境 <st c="52450">pac admin create --name $ENV_NAME --region $REGION \</st>

    <st c="52502">--currency $CURRENCY --language $LANGUAGE \</st>

    <st c="52546">--type 生产环境</st>

    <st c="52564">rawOutput=$(pac admin list --name $ENV_NAME | tail -n 2)</st>

    <st c="52621">environmentId=$(echo $rawOutput | cut -d ' ' -f 2)</st> #启用托管环境 <st c="52701">pac admin set-governance-config \</st>

    <st c="52734">--environment $environmentId \</st>

    <st c="52848">pac admin set-governance-config</st> 命令以启用托管环境功能。

    ```

    ```

                    1.  <st c="52927">创建</st> <st c="52934">服务主体</st> <st c="52954">用于环境：</st>

    <st c="52971">正如我们在</st> *<st c="52985">第五章</st>*<st c="52994">中所做的，我们可以使用以下 PAC CLI 命令来创建一个分配给新创建环境的服务主体：</st>

    ```
    <st c="53424">AzureKeyVault@2</st> build task.
    ```

                    1.  <st c="53451">创建一个 Azure</st> <st c="53468">DevOps 项目：</st>

    <st c="53483">在这里，我们需要使用</st> <st c="53504">一个</st> `<st c="54040">System.AccessToken</st>` <st c="54058">，并且我们可以引用这个令牌来执行管理或全组织范围的操作。</st> <st c="54146">通过这种方式，我们可以消除</st> <st c="54187">个人访问令牌的使用，并且我们可以在集合级别为构建服务帐户授予访问权限和特权。</st> <st c="54318">为此，我们需要禁用</st> **<st c="54351">将作业授权范围限制为当前项目（针对非发布管道）</st>** <st c="54425">和</st> **<st c="54430">将作业授权范围限制为当前项目（针对发布管道）</st>** <st c="54500">选项，位于</st> **<st c="54515">项目设置</st>** <st c="54531">中的</st> **<st c="54550">设置</st>** <st c="54558">面板，在将托管我们管理管道的项目中，如下图所示：</st>

            ![图 7.8 – System.AccessToken：项目与组织范围](img/B22208_07_8.jpg)

            <st c="55598">图 7.8 – System.AccessToken：项目与组织范围</st>

            <st c="55664">如果这些切换按钮被禁用，我们需要在组织级别关闭它们。</st> <st c="55760">最后，我们需要在我们的管道中授予权限，使用</st> `<st c="55966">$(System.AccessToken)</st>` <st c="55987">系统变量来执行以下脚本（请参见</st> `<st c="56058">.pipelines/setup-azure-devops-project.yml</st>` <st c="56099">，位于</st> `<st c="56107">Chapter07</st>` <st c="56116">文件夹中的</st> <st c="56131">GitHub 仓库）：</st>

```
<st c="56144">echo $(System.AccessToken) | az devops login --organization $(System.CollectionUri)</st>
<st c="56228">az devops project create --name ${{ parameters.projectName }} --org $(System.CollectionUri)</st>
<st c="56320">aadAdminGroupId=$(az ad group show --group $(AADGroupForAdmin) --query id -o tsv)</st> echo "Admin group id: $aadAdminGroupId" <st c="56443">azureDevopsAdminGroupDescriptor=$(az devops security group list --organization $(System.CollectionUri) --project ${{parameters.projectName}} | jq -r '.graphGroups[] | select(.displayName=="Project Administrators") | .descriptor')</st> echo "Azure DevOps Admin Group Descriptor: $azureDevopsAdminGroupDescriptor"
# Add the AAD group to the Administrators groups in Azure DevOps
collectionUri=$(System.CollectionUri)
orgnamewithouthttps=${collectionUri//https:\/\//} <st c="56903">curl -u :$(System.AccessToken) \</st>
 <st c="56935">-H "Content-Type: application/json" \</st>
 <st c="56973">-d "{\"originId\": \"$aadAdminGroupId\"}" \</st>
<st c="57306">az devops security group membership add --group-id $azureDevopsAdminGroupId --member-id $aadAdminGroupId --org $(System.CollectionUri)</st> command doesn’t work across projects.

				1.  <st c="57478">Create a</st> <st c="57488">Git repository:</st>

    <st c="57503">When</st> <st c="57509">creating a new Azure DevOps project, a default Git repository with the name of</st> `<st c="57588">project</st>` <st c="57595">and a default team is created.</st> <st c="57627">To create a repository, we can use the</st> `<st c="57666">az repos create</st>` <st c="57681">command with the appropriate parameters.</st> <st c="57723">We need the following variables in our</st> <st c="57762">Bash script:</st>

    ```

    # 变量

    organizationURL="<Your-Azure-DevOps-Organization-URL>"

    project="<Your-Azure-DevOps-Project>"

    repository="<Your-New-Repository-Name>"

    pat="<Your-Personal-Access-Token>"

    <st c="58230">main</st> 分支：

    ```

    ```

    organizationName=$(basename $organizationURL) <st c="58289">git clone https://$organizationName@dev.azure.com/$organizationName/$project/_git/$repository</st> cd $repository <st c="58398">git checkout -b main</st> echo "# $repository" >> README.md <st c="58453">git add .</st>

    <st c="58462">git commit -m "初始提交"</st>

    <st c="58573">main</st> 分支通过引入对拉取请求中关联工作项的检查，防止直接提交（请参见 GitHub 仓库中<st c="58674">create-gitrepo-and-branch-policy.sh</st>，位于<st c="58717">Chapter07</st>文件夹）：

    ```
    <st c="58754">az repos policy work-item-linking create</st> \ <st c="58798">....--branch "refs/heads/main" \</st>
     <st c="58830">--enabled true --organization $organization --project $project \</st>
     <st c="58895">--repository $repositoryId --blocking true --detect false</st>
    ```

    ```

    				2.  <st c="58953">Create service connections for Power</st> <st c="58991">Platform environments:</st>

    <st c="59013">We did this exercise in</st> *<st c="59038">Chapter 6</st>*<st c="59047">, in the</st> *<st c="59056">Branches and</st>* *<st c="59069">environments</st>* <st c="59081">subsection</st><st c="59092">.</st>

				3.  <st c="59093">Create CI/CD pipelines on</st> <st c="59120">pipeline templates:</st>

    <st c="59139">To create an Azure pipeline that uses a pipeline template from a different Git repository in a different Azure DevOps project, we can use the</st> `<st c="59282">resources</st>` <st c="59291">keyword in our pipeline YAML file to specify the repository containing the template.</st> <st c="59377">Then, we can reference the template in our pipeline.</st> <st c="59430">Here is an example for this (see</st> `<st c="59463">.pipelines/azure-pipelines-using-template.yml</st>` <st c="59508">in the</st> `<st c="59516">Chapter07</st>` <st c="59525">folder of the</st> <st c="59540">GitHub repo):</st>

    ```

    <st c="59553">资源：</st>

    <st c="59564">仓库：</st>

    <st c="59578">- 仓库：templates</st>

    <st c="59602">类型：git</st>

    <st c="59612">名称：OtherProject/TemplateRepo</st>

    <st c="59644">引用：refs/heads/main</st>

    <st c="59665">端点：MyServiceConnection</st> 步骤： #或阶段：

    - 模板：.pipelines\ include-paccli-steps.yml@templates

    <st c="61119">--yml-path</st> 选项。要创建这些 YAML 文件，我们可以使用先前讨论过的 Git 命令提交基线流水线，这些流水线已经引用了丰富了秘密扫描和静态代码分析的流水线模板。

    ```

			<st c="61357">At the end of these steps, we have the new Azure DevOps project and the new Git repository with pipelines in place that use the service connections to connect to the Power Platform</st> <st c="61539">environments and leverage pipeline templates located in a centralized repository governed by Microsoft</st> <st c="61642">Entra ID.</st>
			<st c="61651">We can realize the same approach in GitHub.</st> <st c="61696">The only difference here is that we need to configure our GitHub organization to use Microsoft Entra ID through the</st> **<st c="61812">SAML protocol</st>**<st c="61825">. GitHub</st> <st c="61833">can provision enterprise accounts with the help of</st> <st c="61884">a</st> **<st c="61887">system for cross-domain identity management</st>** <st c="61930">(</st>**<st c="61932">SCIM</st>**<st c="61936">) configuration.</st> <st c="61954">This means we can add Microsoft Entra ID accounts to GitHub Enterprise organizations and, respectively, repositories, and those users can</st> <st c="62092">sign in.</st>
			<st c="62100">Microsoft Entra ID</st>
			<st c="62119">Regarding</st> <st c="62130">the DevOps tools we use, the main security takeaway is that we need to embrace Microsoft Entra ID accounts and, respectively, groups to manage access to projects, repositories, pipelines, work items, workflows, and so on.</st> <st c="62352">From the administration perspective, we want to manage only memberships of Entra ID groups to grant or remove access to DevOps services and we do not allow individuals account-level access to</st> <st c="62544">our services.</st>
			<st c="62557">Of course, there are many other tasks that we can introduce in our development life cycle stages.</st> <st c="62656">One of the most crucial activities is Inventory Management as discussed in the</st> *<st c="62735">Setup</st>* <st c="62740">section of this chapter.</st> <st c="62766">Inventory Management can be realized within Power Platform on Dataverse as well.</st> <st c="62847">The setup of new development projects can be managed with the help of Power Automate cloud flows that execute those previously discussed Azure DevOps pipelines behind the scenes.</st> <st c="63026">The Inventory Management repository can store the metadata about our projects, such as who the owners are, where the source code/solution packages are located, which Entra ID groups are created, which Power Platform environments are spun up, and</st> <st c="63272">so on.</st>
			<st c="63278">However, as illustrated in the following figure, there are numerous additional tasks, including those activities we discussed at the beginning of the chapter in conjunction with custom development projects, that occur at different stages of the SDLC of Power</st> <st c="63538">Platform solutions:</st>
			![Figure 7.9 – DevSecOps activities in Power Platform solutions](img/B22208_07_9.jpg)

			<st c="64669">Figure 7.9 – DevSecOps activities in Power Platform solutions</st>
			<st c="64730">For instance, we</st> <st c="64748">can introduce more security analysis tasks in our pipelines, such as using GHAS or CodeQL for SAST, we can create our own build farms for our dedicated builds, apply code signing for our binary components or even for our Power Apps native mobile apps (wrap the app), perform DAST, execute test automation during CI/CD steps, and also set up advanced monitoring to read and analyze the DevOps audit logs or even the application logs.</st> <st c="65181">This is only the tip of the iceberg since projects and organizations are continuously improving their DevSecOps practices and, with that, their security postures</st> <st c="65343">as well.</st>
			<st c="65351">We now delve into our final topic, the security of our established</st> <st c="65419">DevOps processes.</st>
			<st c="65436">Security of DevOps processes</st>
			<st c="65465">Creating DevSecOps processes</st> <st c="65495">and setting up whole development projects fully automated from scratch is necessary to infuse security tasks in every phase and stage of the development life cycle.</st> <st c="65660">On the other hand, we need to consider our established DevSecOps processes for security vulnerabilities and the attack surface as well.</st> <st c="65796">In the previous sections, we have seen that with the help of Microsoft Entra ID groups, we can control and guardrail access to our running projects, but it is only one of the first steps to create more secure methods in those processes.</st> <st c="66033">We need to continuously monitor the activities of our engineers and DevOps teams, such as accessing repositories and executing workflows/pipelines by logging them to Log Analytics</st> <st c="66213">workspaces.</st> <st c="66225">We can then use threat intelligence tools such as Microsoft Azure Sentinel to discover unusual patterns throughout the application life cycle.</st> <st c="66368">We can also create threat models using Microsoft’s Threat Modeling Tool on our own CI/CD pipelines to play around with security scenarios, such as how to avoid vulnerable code injections through DevOps tools, or how to avoid granting access to a developer in production environments via DevOps pipelines or GitHub workflows from risk and</st> <st c="66706">probability perspectives.</st>
			<st c="66731">Mission-critical workloads operating in highly regulated industries and environments, such as the US government cloud or deployments in the financial sector, require even more guardrails in place.</st> <st c="66929">We can fully isolate Power Platform development tenants from production ones.</st> <st c="67007">The handover between these separate tenants can happen through</st> **<st c="67070">Azure Service Bus</st>** <st c="67087">or</st> <st c="67090">other messaging queue services, as the following</st> <st c="67140">figure shows:</st>
			![Figure 7.10 – Isolated tenants](img/B22208_07_10.jpg)

			<st c="67188">Figure 7.10 – Isolated tenants</st>
			<st c="67218">On the development side, Azure pipelines or GitHub workflows build the managed and unmanaged solutions and place them in Azure Blob Storage.</st> <st c="67360">They also queue new messages in the message queue that contain the metadata information about the deployment, such as which solution from which blob container with which version the production side pipelines should deploy to.</st> <st c="67586">They also contain the SAS tokens to access Blob Storage from the production tenant.</st> <st c="67670">On the production side, Power Automate cloud flows pick up the queue message triggered by the Service Bus condition and execute the deployment based on the queue message to the targeted environments.</st> <st c="67870">With that, there is no direct, synchronous connection between development and production tenants.</st> <st c="67968">We</st> <st c="67971">apply</st> <st c="67977">the</st> **<st c="67981">asynchronous design pattern</st>** <st c="68008">here (pulling model) to terminate the original synchronous deployment call to our</st> <st c="68091">production environment.</st>
			<st c="68114">Summary</st>
			<st c="68122">In this chapter, we explored the DevSecOps process, delving into its evolution, the concept of a shift-left mindset, and the integration of security tasks within our SDLC.</st> <st c="68295">We then acquainted ourselves with the security architecture and guiding principles underpinning Power Platform.</st> <st c="68407">Our journey continued with an examination of GHAS, leveraging CodeQL to conduct SAST.</st> <st c="68493">We also investigated the solution checker, utilizing Azure DevOps build tasks and GitHub Actions to perform the platform’s built-in analysis.</st> <st c="68635">Our deep dive extended into the realms of the Azure CLI, Azure DevOps scripts, pipelines, and pipeline templates, enabling us to construct an Azure DevOps project embedded with security from the ground up in a fully automated manner.</st> <st c="68869">Lastly, we dedicated time to understanding the security threats that could potentially compromise our DevOps pipelines or workflows and discussed strategies to mitigate</st> <st c="69038">these risks.</st>
			<st c="69050">In the next chapter, we will craft a tangible solution, applying every aspect of DevOps and ALM through practical,</st> <st c="69166">hands-on walkthroughs.</st>
			<st c="69188">Further reading</st>

				*   <st c="69204">DevSecOps</st> <st c="69215">controls:</st> [<st c="69225">https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/secure/devsecops-controls</st>](https://learn.microsoft.com/en-us/azure/cloud-adoption-framework/secure/devsecops-controls)
				*   **<st c="69315">Security Development Life Cycle</st>**<st c="69347">(</st>**<st c="69349">SDL</st>**<st c="69352">)</st> <st c="69355">Practices:</st> [<st c="69366">https://www.microsoft.com/en-us/securityengineering/sdl/practices</st>](https://www.microsoft.com/en-us/securityengineering/sdl/practices)
				*   <st c="69431">Threat</st> <st c="69439">Modeling:</st> [<st c="69449">https://www.microsoft.com/en-us/securityengineering/sdl/threatmodeling</st>](https://www.microsoft.com/en-us/securityengineering/sdl/threatmodeling)
				*   <st c="69519">Microsoft threat modeling</st> <st c="69546">tool:</st> [<st c="69552">https://aka.ms/threatmodelingtool</st>](https://aka.ms/threatmodelingtool)
				*   <st c="69585">GitHub</st> <st c="69593">Dependabot:</st> [<st c="69605">https://docs.github.com/en/code-security/dependabot</st>](https://docs.github.com/en/code-security/dependabot)
				*   <st c="69656">CodeQL:</st> [<st c="69665">https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning-with-codeql</st>](https://docs.github.com/en/code-security/code-scanning/introduction-to-code-scanning/about-code-scanning-with-codeql)
				*   <st c="69781">Security in Power</st> <st c="69800">Platform:</st> [<st c="69810">https://learn.microsoft.com/en-us/power-platform/admin/security/overview</st>](https://learn.microsoft.com/en-us/power-platform/admin/security/overview)
				*   <st c="69882">GitHub Advanced Security for Azure</st> <st c="69918">DevOps:</st> [<st c="69926">https://azure.microsoft.com/en-us/products/devops/github-advanced-security</st>](https://azure.microsoft.com/en-us/products/devops/github-advanced-security)
				*   <st c="70000">CodeQL CLI</st> <st c="70012">binaries:</st> [<st c="70022">https://github.com/github/codeql-cli-binaries</st>](https://github.com/github/codeql-cli-binaries)
				*   <st c="70067">CodeQL query</st> <st c="70081">packs:</st> [<st c="70088">https://github.com/github/codeql</st>](https://github.com/github/codeql)
				*   <st c="70120">CodeQL</st> <st c="70128">analysis:</st> [<st c="70138">https://docs.github.com/en/code-security/codeql-cli/getting-started-with-the-codeql-cli/preparing-your-code-for-codeql-analysis</st>](https://docs.github.com/en/code-security/codeql-cli/getting-started-with-the-codeql-cli/preparing-your-code-for-codeql-analysis)
				*   <st c="70265">Solution</st> <st c="70275">checker:</st> [<st c="70284">https://learn.microsoft.com/en-us/power-apps/maker/data-platform/use-powerapps-checker</st>](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/use-powerapps-checker)
				*   <st c="70370">PAC CLI for solution</st> <st c="70392">checker:</st> [<st c="70401">https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/solution#pac-solution-check</st>](https://learn.microsoft.com/en-us/power-platform/developer/cli/reference/solution#pac-solution-check)
				*   <st c="70501">Microsoft.PowerApps.Checker.PowerShell</st> <st c="70541">module:</st> [<st c="70549">https://learn.microsoft.com/en-us/powershell/module/microsoft.powerapps.checker.powershell/invoke-powerappschecker?view=pa-ps-latest</st>](https://learn.microsoft.com/en-us/powershell/module/microsoft.powerapps.checker.powershell/invoke-powerappschecker?view=pa-ps-latest)
				*   <st c="70681">System access</st> <st c="70696">token:</st> [<st c="70703">https://learn.microsoft.com/en-us/azure/devops/pipelines/process/access-tokens?view=azure-devops&tabs=yaml</st>](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/access-tokens?view=azure-devops&tabs=yaml)
				*   <st c="70809">Work item branch</st> <st c="70827">policy:</st> [<st c="70835">https://learn.microsoft.com/en-us/rest/api/azure/devops/policy/configurations/create?view=azure-devops-rest-7.1&tabs=HTTP#work-item-policy</st>](https://learn.microsoft.com/en-us/rest/api/azure/devops/policy/configurations/create?view=azure-devops-rest-7.1&tabs=HTTP#work-item-policy)
				*   <st c="70973">GitHub enterprise</st> <st c="70992">accounts:</st> <st c="71002">https://docs.github.com/en/enterprise-cloud@latest/admin/managing-your-enterprise-account/about-enterprise-accounts</st>

```

```

```
