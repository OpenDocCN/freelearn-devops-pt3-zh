

# 第二章：构建 DevOps 文化

成功实施 DevOps 的核心不在于使用的技术和工具。相反，建立并与新工作方式保持一致的团队文化是 DevOps 的最关键因素。

在本章中，我们将讨论 DevOps 的线下方面的重要性，以及协作和沟通文化如何是 DevOps 成功的基础。我们将探讨在组织中推动采纳和与最佳实践保持一致的方法。过程中，我们将探索以下内容：

+   为什么文化是 DevOps 转型的关键，以及我们如何开始构建它

+   努力推动强有力的沟通以促进协作

+   推动采纳 DevOps 方法并与其保持一致的方式

# 需要建立 DevOps 文化

软件开发及交付的历史悠久且不断变化。随着技术领域的变革，企业对将这些技术交到客户手中的需求也在不断增加。DevOps 代表着根据这一需求进行交付的推动力，它取代了传统的单体软件发布、漫长的项目周期和不透明的瀑布式开发方法。

当我们把 DevOps 视为一种变革交付方式时，很容易首先关注软件工具，但这不应该是你 DevOps 之旅的起点。同样重要的是要牢记，任何 DevOps 转型都不应该是具有指导性的；它应该与你和你所在组织的工作方式保持一致。对于那些曾经在其他团队或系统中有过 DevOps 经验的人来说，这一点尤为重要——DevOps 没有“万能”的方法，虽然经验可以帮助在新团队中建立 DevOps 文化，但你应该注意根据团队的特点进行定制。

然而，成功的 DevOps 团队有一些共同的特征，这些特征对高效的团队始终有效。因此，你应该考虑将这些特征纳入你的计划中，以将 DevOps 文化付诸实践。让我们先来看看成功的 DevOps 团队的特征及其采纳的 DevOps 文化要素，然后再深入探讨如何实现它们。

## 明确的团队定义

顾名思义，DevOps 团队是 IT 开发团队和 IT 运维团队的结合体，但现实情况并不像表面那么简单。成功的 DevOps 团队由软件交付全流程的各个团队组成，从业务分析师收集需求和架构师设计解决方案，到开发人员实现这些解决方案，再到运维人员在 Salesforce 环境中交付这些需求。

正是在这种跨职能团队结构中，你需要为 DevOps 建立强有力的支持。一个不理解或不重视过程价值的团队，不太可能采纳 DevOps——而且只需要几次偷工减料或不按流程发布，就能破坏你们 DevOps 团队的良好工作。至关重要的是，整个团队要与 DevOps 的工作方式保持一致并参与其中，才能让这一举措取得成功。

与 DevOps 实践对齐的团队对整个应用生命周期负责，从规划到部署和维护，减少了在修复 bug 或测试失败时的对立和推卸责任。此外，DevOps 鼓励产品团队更加参与开发过程，确保他们的意见和专业知识在整个应用生命周期中得到考虑。

作为架构师，我们需要传达 DevOps 带来的价值，因为对大多数团队来说——无论是技术团队还是业务团队——这通常是让人们加入的关键因素。通过展示 DevOps 如何在我们概述的开发旅程中使每个人受益，相比于强制执行流程，我们更有可能让团队接受 DevOps。

尚未采用 DevOps 文化进行软件交付的公司，可能已经失去了对交付团队的信任，试图通过引入重型流程来防止未来失败的风险。采用 DevOps 文化的一部分就是通过提供能够帮助团队成功的工具和流程来恢复信任，并让任何失败保持在较小的范围内，而不是通过避免失败完全来拖慢一切进展。

一般来说，DevOps 和敏捷的一个好处是能够安全地迈出小步伐。DevOps 和敏捷方法提倡小步、增量发布，而非大型的、单一的部署。这种方法可以让团队更快地识别并解决问题，降低灾难性失败的风险。它还使团队能更有效地应对不断变化的需求或市场条件。因此，团队在交付准确结果和适应变化方面的信任度会增加。

## 紧密合作

与强大的团队并行的是彼此协作和沟通的需求。这看起来是所有工作团队中的显而易见需求，不仅仅是 DevOps 团队，但在 DevOps 中，清晰、可见性和合作的原则真正突显出来，对于其顺利运行至关重要。

为了打破软件交付的孤岛式做法并朝着共同目标努力，整个团队需要了解项目是如何交付的。像敏捷（Agile）这样的技术和像 Jira 或 Asana 这样的工具无疑会有所帮助，但这只是协作图景的一部分，正如我们稍后会探讨的那样。

## 持续演进

无论 DevOps 团队多么成熟，表现最好的团队总是愿意接受变化和改进。通过不断的测量、改进和重新测量，这些团队能够找出可以提高性能和准确性的领域，并加以解决。它们通常关注的最常见指标基于 DORA 指标，具体如下：

+   **部署频率**：团队向生产环境发布的频率

+   **变更交付时间**：特定功能到达生产环境所需的时间

+   **变更失败率**：部署失败或在生产中引起错误的部署比例

+   **恢复平均时间**：从生产错误或其他问题中恢复所需的时间

在 Salesforce 的背景下，衡量这些指标可能稍有不同，因为它是一个基于云的平台，具有特定的功能和限制。像部署频率、变更交付时间和恢复平均时间这样的指标很容易确定，特别是如果你有像 Jira 或 Asana 这样的工单系统来管理新工作。

变更失败率可能稍微复杂一些，因为它涉及跟踪失败的部署以及与这些部署相关的事件或缺陷数量。你可以用几种方法来处理这个问题——我们将在后续章节中讨论 Salesforce 特定的 DevOps 解决方案和平台，但作为一个使用平台内特性的示例，你可以尝试以下方法：

+   使用 Salesforce 的部署历史，可以在 **部署状态** 页面上查看部署的成功率和失败率。识别失败的部署及导致失败的具体组件。

+   记录所有生产事件，包括由最近部署引起的事件。你可以使用 Salesforce Case 对象来记录和跟踪事件。

+   对于每一个失败的部署或生产事件，分析根本原因并确定是否与最近的变化有关。你可以使用 Salesforce Developer Console、调试日志和测试结果来找出问题的根本原因。

+   将失败的变更数量（导致事件或缺陷的部署）除以特定期间内所做变更的总数。将结果乘以 100 得到变更失败率的百分比。

DORA 指标的起源

DORA 指标来自一个名为 **DevOps Research and Assessment** 的小组，该小组由 Nicole Forsgren、Gene Kim 和 Jez Humble 于 2015 年创立（并在 2018 年被 Google 收购），旨在更好地理解哪些因素导致高效能的 DevOps 团队。自从最初的研究以来，这四个指标已成为衡量 DevOps 成功的行业标准。

现在我们已经确定了强大 DevOps 文化的需求和要素，让我们更详细地看看一些创建这种文化的技巧。

# 协作与沟通

在理想的 DevOps 团队中，整个团队以相同的方式工作，朝着相同的目标努力——应对变更交付的成功共同承担责任。这个协作方法的核心是强有力的沟通，这种沟通可以采取多种形式，从治理整体变更管理流程所需的更正式的方式，到日常互动，这些互动构成了你常规工作流程的一部分。

沟通应该清晰、信息丰富，并且贯穿于交付生命周期的每个步骤。例如，在使用版本控制时，团队应努力始终提供*有意义的*提交信息和同行评审的评论。这些有助于团队以*背景*为基础进行下一步变更交付过程，而不仅仅是具体的变更内容。通常需要在提供足够详细的信息和*相关*信息之间取得平衡，你应该不断调整这种细节层次，以找到适合你和团队的最佳平衡点。

尽管本书并未深入探讨敏捷原则，但成功的 DevOps 团队与敏捷实践者之间确实存在着强烈的关联，因为这两个领域都强调定期、清晰和简洁的沟通原则，以推动项目向前发展。这些技巧涵盖了所有参与变更交付的团队成员，以确保每个人都了解并意识到工作流程和进度。

同样，工具将帮助提高日常工作的可见性和清晰度。用于管理特性通过 DevOps 流程的软件，如 Jira、Asana、Azure DevOps 等，在正确使用时可以为你的流程提供概览，并且它们与大多数 DevOps 工具以某种方式集成，以完善整个画面。许多团队已经开始摒弃电子邮件作为内部沟通媒介，而是更倾向于使用即时消息平台，如 Slack 或 Teams，作为进一步打破孤岛并消除跨职能团队之间障碍的方式。

适应远程工作的必要性导致了对数字沟通工具的依赖增加，并且在许多方面改变了团队互动的动态。由于团队成员分布在不同地点和时区，拥有能够实现实时协作的工具是至关重要的，这些工具提供即时沟通、文件共享以及与其他工具的集成。在远程工作中，并非总是能够在同一时间召集所有人进行讨论。异步沟通工具，如项目管理平台、共享文档以及消息应用中的线程讨论，使团队成员能够在方便的时候做出贡献，并让每个人了解进展。

在向远程工作转变的过程中，每个需要适应的变化都需要保持平衡。随着数字通信的普及，远程工作者可能会面临大量的消息和通知涌入。消息平台通过提供频道、线程和打盹选项等功能进行适应，帮助团队成员有效地优先处理和管理通讯。然而，保持团队成员之间的连接和参与感同样重要。消息平台促进了非正式的互动，如虚拟水冷器聊天、快速签到和社交活动，帮助团队保持联系并促进积极的团队文化。

远程工作使得团队必须在没有面对面交流所提供的背景下有效沟通。现代的远程团队沟通方法鼓励团队成员在沟通时更加简洁和清晰，同时在回应时也更加有意识。

最后，由于远程工作依赖于数字通信工具，确保数据安全性和符合行业规定变得至关重要。技术解决方案已经通过提供端到端加密、数据存储选项和针对不同行业的合规功能来做出响应。

# 采纳与对齐

正如我们所见，DevOps 文化的采纳应该先于 DevOps 技术的采纳。然而，在每个方面，最佳的做法始终是慢慢开始——人们常说 DevOps 是一段旅程，而不是一个目的地，为此，我们应该从一些规划开始。

## 开始时的问题

开始任何旅程的最佳方式是先看看我们想要去哪里，并提出一些问题：

+   *预期的流程是* *什么样的？*

了解你的目标场景有助于集中精力，避免对业务造成不必要的干扰。例如，最终目标是能够更快地、逐步地交付业务需求，还是想采用更有计划的、基于冲刺的方式，但又能更好地看到并控制冲刺中的各个元素？事先明确目标有助于团队专注于实现这些目标。

+   *新流程的目标受众是* *谁？*

一个新的 DevOps 过程将影响的不仅仅是开发和运维团队。如果你真的想要采纳端到端的 Salesforce DevOps 方法，你不仅需要对齐技术团队，还需要协调那些参与任务收集和分配、项目管理、发布管理、整体架构、业务审批等工作的人员。我们稍后将讨论一些治理方面的内容。

+   *我们需要在当前的方法中* *做出哪些改变？*

虽然并非没有先例，但要完全替换现有流程的情况是比较罕见的。先评估一下你当前的交付模式，并记录下哪些有效，哪些无效。哪些瓶颈在拖慢你的进度？交付一个生产版本需要多少次尝试？如果我们重新审视之前讨论过的 DORA 指标，我们的起点在哪里？在开始 DevOps 转型之前，设定一组基准指标是衡量进展和改进的坚实方式——最终，它将帮助你衡量投资回报率，尤其在你开始采用 Salesforce DevOps 时。此外，能够向高层管理者展示问题（以及后来所做的改进）是赢得他们支持 DevOps 转型项目的无价之宝。

当这些问题摆在眼前时，开始识别采用 Salesforce DevOps 带来的潜在收益就变得更加容易，进而有助于推动团队与变化的一致性。这一步是至关重要的——所有参与者都需要成为 DevOps 转型的利益相关者，而实现这一目标的最佳方式是从两个角度来审视问题：

+   *DevOps 会带来哪些好处？*

在确定了会直接受到 DevOps 交付模式影响的团队之后，着手向他们传达这种变化的好处。变更的可见性、更易管理的小规模工作单元、更快速的交付、稳健的测试、可预测的发布周期——这些对于 Salesforce 团队来说都至关重要，而让他们的工作变得更轻松这一总体原则，是促使他们支持变革的重要动力。

+   *不采纳 DevOps 的风险是什么？*

同样，评估停滞不前、什么都不改变的风险也是非常重要的。如果你不采用更快速、更灵活的交付模式，你有可能会被更敏捷的竞争对手超越。如果你不实施一个强大的备份和恢复策略，你可能会失去你宝贵的业务数据或客户数据。如果你坚持使用更传统的交付模式，这些模式可能既冗长又繁琐，你可能会面临团队的不满和倦怠，最终导致他们离开。

## 让团队的工作更轻松

如果你的 Salesforce 团队对 DevOps 概念、技术，甚至术语都不熟悉，那么他们可能会觉得转向新的交付模式是一个令人生畏的前景。然而，像所有大型项目一样，最优的方式通常是从小的环节开始，慢慢推进，逐步扩展和迭代。

例如，由于源代码管理的概念历来是开发人员专属的领域，许多 Salesforce 管理员对这种方法并不熟悉。仅仅这一领域就是一个很好的起点——即使您不一定直接进入源代码管理的应用，单是让管理员和开发人员对齐工作方式，便有助于在建立 DevOps 文化的过程中促进沟通与协作。

让管理员和开发人员以这种方式更紧密地合作，也有助于采用另一套促进 DevOps 文化的有效技术。高效能的 Salesforce DevOps 团队经常利用辅导和指导，不仅提升团队的整体技能和信心，而且作为协作工作的助力，打破孤岛，形成多学科的 DevOps 团队。

当然，这不仅仅是关于流程，您还应确保您的团队拥有必要的工具，以帮助顺利推进 DevOps 之旅。作为架构师，您应时刻关注 Salesforce DevOps 的生态，评估各种组件，例如版本控制提供者、新工具或现有工具的更新，甚至是完整的 Salesforce DevOps 平台——其中一些我们将在后续章节中讨论。

## 治理与风险管理

DevOps 文化应时刻关注管理和减轻业务风险的需求，并应有强大的治理框架来提供保障。重要的是要认识到，虽然 DevOps 解锁了快速交付变更的潜力，但它并非没有控制的自由放任。

我们的 DevOps 过程的治理应该与您的业务，或您的客户的业务运营所采用的治理相一致。如果没有正确的流程，您有可能失去在开始 DevOps 旅程时所促成的对齐和采用的价值。您有可能重新回到使用冗长、单体发布的模式，而不满的客户正在等待那些深埋在待办事项中的变更。您还可能面临低质量变更的风险，最坏的情况是，这可能会损害您的系统、数据和声誉。

金融服务和医疗等受监管行业在软件开发和部署方面面临独特的挑战。这些行业受到广泛的法规、标准和合规要求的约束，这些法规规定了软件开发、测试和部署的方式。这些法规旨在保护敏感数据、确保数据隐私，并防止欺诈和其他犯罪活动。

在金融服务行业，像**萨班斯-奥克斯利法案**、**支付卡行业数据安全标准**（**PCI DSS**）和**反洗钱**（**AML**）等法规要求金融机构在软件开发、测试和部署过程中实施强有力的控制。同样，在医疗保健行业，像**健康保险流通与问责法案**（**HIPAA**）和**通用数据保护条例**（**GDPR**）等法规要求医疗机构保护患者数据并确保数据隐私。DevOps 可以通过为软件开发提供结构化过程，包括自动化测试、安全扫描和持续监控，帮助这些行业的组织遵守相关法规。这可以确保软件在开发时考虑到安全性和隐私，并且在软件部署之前识别并解决任何潜在的安全漏洞。

一个好的治理框架通过在整个生命周期中实施必要的制衡措施来解决这些问题。从优先排序工作和决定推进哪些项目，到技术设计和实施考虑，治理使各方利益相关者能够为成功提供反馈。

这一方法的核心是卓越中心（CoE），它负责监督这一过程。它指导和通知与 Salesforce 相关的业务目标、交付方法和使用的技术。它还负责与组织内部的利益相关者和最终用户沟通，识别和管理项目的业务风险，并确保项目能够交付价值。

因此，CoE 通常包含或与具有特定职责的不同小组合作。例如，变更管理小组将负责批准进入 Salesforce 的变更，并确保变更质量合适，且在允许其发布到生产环境之前已进行充分的测试。这通常通过定义所需的流程和行为来确保交付质量，而不是由变更管理小组直接执行测试，测试仍将是技术团队的责任。

然而，在变更管理小组方面，应该注意一些问题。在 Nicole Forsgren、Jez Humble 和 Gene Kim 合著的《加速：精益软件与 DevOps 的科学：构建和扩展高效能技术组织》一书中，作者强调了快速反馈回路、持续实验以及学习和改进文化的重要性——这些因素表明，传统的变更管理实践可能并不总是与高效能 DevOps 团队的需求对接。

另一方面，指导委员会是一个由业务主导的团队，确保变更始终与业务战略、愿景和价值观保持一致。在所有这些领域中，应该有一位执行赞助人，能够并且有权做出决策并解锁业务瓶颈。

### 向领导层阐明 CoE 的必要性

希望向组织或客户的领导层提出建立 CoE 案例的架构师，通常会借鉴展示任何提案给利益相关者时的相同技巧。然而，某些具体元素应被视为该提案的基本组成部分。以下是一些典型的关注领域：

| **主题** | **详情** |
| --- | --- |
| 定义目标和宗旨 | 阐明 CoE 的目标，如推动持续改进、分享最佳实践、促进协作以及加速 DevOps 在组织中的采用。 |
| 建立商业案例 | 创建一个有说服力的商业案例，展示 CoE 的好处，包括潜在的成本节省、提高的运营效率、更快的市场时间和更高的客户满意度。展示行业示例和相关的成功故事。 |
| 确定关键利益相关者 | 确定并与关键利益相关者互动，如高层管理、开发和运维团队。将他们纳入决策过程，并在随后的 CoE 建立过程中发挥作用。 |
| 提出 CoE 结构 | 提出 CoE 的结构，包括角色、职责和汇报关系。估算设立和维持 CoE 所需的预算和资源。职位可能包括 DevOps 教练、产品负责人和持续改进专家。 |
| 制定路线图 | 制定 CoE 实施的路线图，包括里程碑、时间表和**关键绩效指标**（**KPIs**）。为领导层提供一个清晰的计划，以便跟踪和监控进展。 |
| 变更管理计划 | 认识到实施 CoE 可能涉及重大文化和组织变革。提出一项变更管理策略，以应对潜在的抗拒、沟通和培训需求。 |
| 促进协作 | 强调跨职能协作和知识共享的重要性。提出促进沟通和协作的工具和平台，如聊天平台、维基或视频会议系统。 |
| 启动试点并迭代 | 提议从一个试点项目开始，涉及一个或多个团队来测试和完善 CoE 方法。让组织从试点中学习，进行调整，并在更广泛的 DevOps 采用过程中逐步扩展 CoE。 |
| 定期报告进展 | 确保将 CoE 的进展定期报告给领导层，包括成功、挑战和收获。通过透明化保持高层管理的支持和承诺。 |
| 展示持续价值 | 持续突出 CoE 对组织的积极影响，包括效率、质量和创新方面的可量化改进。维护并扩大对 CoE 的支持，以及其在更广泛 DevOps 采用中的角色。 |

表 2.1 – 提案中需要考虑的要素

### 克服抗拒和犹豫

有几个常见的原因，可能导致人们最初抗拒在他们的组织中实施 DevOps，认为“这听起来不错，但在这里做不到。”让我们解决其中的一些原因，并提供反驳，帮助消除这些担忧：

| **领域** | **抗拒** | **反驳** |
| --- | --- | --- |
| 组织结构和文化 | 现有的组织结构和文化促进了团队之间的隔阂，并不鼓励协作 | DevOps 是打破隔阂并促进协作的机会。从小的变化开始，例如创建跨职能团队，并随着组织适应新方法逐步扩大 DevOps 措施。 |
| 技能和专业知识缺乏 | 团队成员缺乏实施 DevOps 实践和工具的技能和知识 | 投资培训和提升团队成员技能，并考虑聘请或与专家合作，帮助指导 DevOps 转型。持续学习是 DevOps 的核心原则，因此发展这些技能应该被视为一个持续的过程。 |
| 资源和预算有限 | 没有预算或资源投资于新工具、技术和 DevOps 转型所需的培训 | DevOps 可以帮助提高效率，并在长期内降低成本。从利用现有工具和资源开始，随着投资回报率的展示和组织的认可，逐步扩大 DevOps 能力。 |
| 害怕失败和干扰 | 改变现有的流程可能导致干扰，并对当前项目产生负面影响 | DevOps 关注持续改进和从失败中学习。从小而低风险的项目开始，以最小化潜在干扰，并利用学到的经验在处理更大项目之前精炼方法。 |
| 遗留系统和技术债务 | 现有的基础设施和遗留系统使得采用现代 DevOps 实践和工具变得困难 | DevOps 可以通过促进渐进式改进和培养创新文化来帮助解决技术债务和现代化遗留系统。优先处理基础设施中最关键的部分，并制定引入 DevOps 实践的路线图。 |
| 缺乏管理支持 | 管理层没有看到 DevOps 的价值，或不愿意投资必要的变革 | 通过强调 DevOps 的潜在收益，建立一个强有力的商业案例。分享其他组织的成功故事和最佳实践，并考虑开展试点项目，亲自展示 DevOps 的价值。 |
| 法规和合规问题 | 采纳 DevOps 实践可能与在高度监管行业中的合规要求冲突 | DevOps 可以通过自动化流程、确保一致性和提供更好的可见性来改善合规性。与合规和安全团队合作，确保你的 DevOps 实践符合行业法规和组织政策。 |

表 2.2 – 驳斥顾虑的理由和反驳

通过解决这些常见问题并展示采纳 DevOps 的潜在好处，你可以帮助克服阻力，鼓励利益相关者接受这一变革性方法。

# 总结

有一句话常被引用（有时甚至被误引用）：“*文化吃掉战略早餐*”，这句话在 DevOps 的世界里尤其贴切。无论你制定的 DevOps 采纳战略有多么精妙，如果你的团队没有接受所需的文化和心态，它也不会成功。通过推广 DevOps 流程的优势，确保整个团队一起协作，并在过程中保持强有力的沟通，你为成功的 Salesforce DevOps 转型奠定了基础，并可以在此基础上，结合我们在本书下一部分将探讨的工具和技术继续发展。在接下来的两章中，我们将首先看一下测试在 DevOps 生命周期中的重要作用，然后再看一个考虑到这些因素的典型 SFDX 和 Git 工作流示例。

文化吃掉战略早餐——或者说，它真的是这样吗？

这句话通常归功于著名的管理专家彼得·德鲁克。虽然这种版本仍然广泛使用，并且证明了我们这里的观点，但德鲁克的原始名言是“*文化——无论如何定义——是* *特别持久*。”
