

# 第十二章：从传统系统迁移到云原生解决方案

本章将深入探讨云迁移为何可能偏离轨道，更重要的是，如何避免这些陷阱。我们将分析导致云采用失败的最常见错误，比如在没有坚实战略的情况下贸然开始、缺乏关键利益相关者的支持，以及试图将过时的传统安全实践强行套用到云中。

以下是我们将讨论的内容：

+   如何制定清晰的迁移战略，使其与业务目标一致，这样你不仅仅是在迁移工作负载，而是确实在创造价值。

+   在整个云迁移过程中，利益相关者的参与至关重要，确保所有人保持一致，且都致力于迁移的成功。

+   为什么传统的安全实践无法适用于云环境，以及如何转向适应当今动态环境的云原生安全模型。

+   如何识别并弥补技能差距，确保你的团队为顺利高效的迁移做好充分准备。

本章结束时，你将拥有一份避免破坏云迁移的陷阱的路线图，并为顺利迁移到云做好更充分的准备。￼

# 规划有效的迁移

云迁移遇到障碍或完全失败的最大原因之一就是缺乏清晰的战略和适当的规划。许多企业未充分考虑全局，贸然开始迁移，导致混乱、意外的费用和低效问题。成功的迁移需要从一开始就进行深思熟虑，并与业务目标保持一致。

在本节中，我们将首先讨论为什么一个完善的迁移计划对于确保进度至关重要。我们将探讨如何通过为应用程序设定明确的优先级、选择合适的云平台以及决定迁移策略——无论是“提升迁移”（lift-and-shift）还是完全重构——来做出关键的决策。通过了解这些基础要素，我们可以以更清晰的思路、更专注的态度来进行迁移，从而提高成功的可能性。

## 没有计划地跳入：为何规划如此重要

一种反复出现的常见错误是将云迁移视为老旧基础设施的快速修复。团队通常匆忙推进这一过程，认为只需将应用程序或整个服务器迁移到云上，而不加深思。这种缺乏规划的做法会导致：

+   应用程序没有针对云环境进行优化。

+   工作负载与云服务不匹配。

+   由于资源扩展不当，导致的意外成本。

+   因未充分利用云原生服务而导致的投资回报未能实现。

+   早期未能解决合规性和安全要求。

+   这些问题中的每一个都可能是导致运营成本上升、迁移和云采用延误，甚至工作负载性能下降的根本原因。

+   如果没有适当的规划，我们可能会陷入混乱，形成一个效率低下、成本高昂且易于失败的拼凑式云环境。这正是“*仓促行事，事后悔* *慢慢感受*”的经典案例。

## 制定清晰的迁移策略

成功的云迁移的关键在于一个坚实的策略。迁移到云端不仅仅是移动工作负载；它是一个重大转变，需要精心规划和明确的方向。如果没有强有力的计划，过程可能迅速演变为意外的成本、延误和不必要的复杂性。本节将深入探讨如何制定一个与业务目标对齐并使团队保持专注的迁移策略。从评估当前环境到选择正确的云模型，我们将覆盖一系列基础步骤，帮助您顺利进行未来准备的迁移。

### 评估当前环境

在开始之前，首先要清楚自己在处理什么。识别关键应用程序、依赖关系和集成。弄清楚哪些可以退休，哪些需要重大重构。别忘了查看网络依赖关系和延迟需求。

为了正确评估我们当前的环境，我们需要利用云服务提供商或第三方的云迁移评估工具。例如：

+   AWS 迁移评估工具可以帮助我们通过映射工作负载、依赖关系和集成来分析本地基础设施。它将提供详细的成本分析，并展示在 AWS 环境中的效果。

+   类似地，Azure Migrate 可以绘制出依赖关系和网络交互，展示哪些内容适合迁移到云端以及可能出现的延迟问题。

+   Google Cloud 的 Migrate for Compute Engine 以类似的方式工作，提供对虚拟机、应用程序和基础设施的洞察，确保我们为每个工作负载有清晰的迁移路径。

使用这些工具之一可以详细列出关键应用程序、它们如何交互，以及任何网络或延迟需求。这种可见性使我们能够做出更明智的决策，了解哪些可以退休、哪些需要重构或保留不变，帮助避免常见的导致迁移失败的陷阱。

在上述每个案例中，输出应该为您提供一个有用的迁移资产清单，以及它们可能需要的资源。以下是此数据的示例，但不局限于此，也不全面：

| **主机名** | **CPU 核心数** | **操作系统** | **操作系统版本** | **总内存** |
| --- | --- | --- | --- | --- |
| app-server01.local | 4 | Windows Server 2019 | 1809 | 8192 |
| linux-db01.local | 4 | Ubuntu 20.04 | Focal Fossa | 16384 |
| mssql-db01.local | 8 | Windows Server 2016 | 1607 | 32768 |
| app-server02.local | 2 | Windows Server 2019 | 1809 | 8192 |
| backup-server.local | 4 | Windows Server 2012 R2 | 9600 | 8192 |
| web-server01.local | 2 | RHEL 8 | Ootpa | 4096 |
| dev-server.local | 4 | Windows 10 Pro | 21H1 | 16384 |
| linux-app01.local | 2 | CentOS 7 | Core | 8192 |
| storage-server.local | 16 | Windows Server 2019 | 1809 | 65536 |
| dns-server.local | 1 | Ubuntu 18.04 | Bionic Beaver | 2048 |
| mail-server.local | 4 | Windows Server 2016 | 1607 | 8192 |
| log-server.local | 8 | Ubuntu 22.04 | Jammy Jellyfish | 16384 |

表 12.1 - 迁移评估数据

### 优先排序应用：使用灯塔应用

我们不需要一次性迁移所有内容。明智的做法是根据应用的业务影响、迁移的难易程度以及能够从云中获得最大收益的因素来优先处理应用。我们从那些较简单的应用开始，逐步提升，随着我们变得更加熟练，再处理更复杂的应用。我们有时将这一概念称为“*灯塔*”。

选择一个灯塔应用

灯塔应用是证明云采用价值的第一步。它是一个小型、低风险的应用，设定了其他一切的基调。选择时，要选一个足够重要以展示实际影响，但又不至于复杂到让团队陷入困境的应用。理想的灯塔应用易于迁移，能够明显从云原生功能（如自动扩展或无服务器架构）中受益，并且为您带来一个快速的成功，积累动力。关键是聪明地开始，为后续更大、更复杂的迁移奠定基础。

并非所有工作负载都需要相同的处理方式，按大小将它们分为“小型”、“中型”和“大型”——有助于我们简化规划和资源分配。T 恤尺寸法为我们提供了一种快速、实用的方式，根据复杂性和迁移工作量对工作负载进行分类。

+   小型工作负载

    这些是简单、低影响的应用，可以轻松进行迁移，适合早期的迁移，几乎不需要更改，并且能帮助我们迅速积累动力。

+   中等工作负载

    这些应用可能需要一些平台迁移或调整，以便在云中获得最佳性能。它们通常有特定的延迟或集成要求，需要更多的规划。

+   大型工作负载

    大型、关键任务应用通常需要进行大规模重构，以充分利用云的优势。它们的迁移是分阶段进行的，并涉及详细的规划，以确保与业务需求保持一致。

通过提前为工作负载确定大小，我们可以清楚地了解资源、时间表和依赖关系，从而专注于快速成功，并以可管理的方式为更复杂的迁移做好准备。

通过利用像 AWS 应用发现服务或 Azure Migrate 的评估等工具，我们可以自动化确定哪些工作负载已经准备好迁移到云端，哪些则需要更多工作。这些工具为我们提供了清晰的视图，帮助我们决定从哪里开始以及哪些工作负载可以延后处理。

这种分阶段的方法帮助我们降低风险，避免团队感到不堪重负，并且在过程中保持动力。首先从较简单的迁移开始，可以确保更平稳的过渡，并为处理更关键的应用奠定长期成功的基础。

### 定义您的云模型

在我们开始迁移之前，最重要的决策之一是选择哪种云模型最适合我们的需求：*单云*、*多云*或*混合云*。每种模式都有其优缺点，因此我们需要根据每个工作负载的目标明智选择。

#### 单云

如果我们坚持使用单一的云服务提供商，无论是 AWS、Azure 还是 GCP，这样可以简化操作。管理一个环境可以让团队更轻松，因为我们只需要专注于一套工具、API 和服务。对于不太复杂或内部应用来说，这种方法通常能提供所需的可靠性和性能。

然而，我们必须考虑供应商锁定的风险以及停机的潜在影响。如果我们的整个操作都依赖于一个供应商，他们任何一次停机都可能对我们造成严重影响。这就是权衡，简单性与灵活性和风险缓解之间的取舍。

云服务提供商的全球基础设施

在决定我们的云模型时，评估我们所考虑的**云服务提供商**（**CSP**）的全球基础设施非常重要。这包括了解他们的接入点数量以及他们确保高可用性的能力。

选择单一的云服务提供商并不意味着要牺牲高可用性，因为大多数云服务提供商都提供多个可用区和区域，可以用于冗余和故障转移。

关于如何实现高可用性的更多信息，请参考*第十一章*，我们将在其中深入探讨这个话题。

#### 多云

对于关键任务应用程序，多云策略为我们提供了额外的韧性。通过将工作负载分布在多个云服务提供商之间，我们减少了受单一故障点影响的风险。

例如，我们可能会将主应用程序部署在 AWS 上，同时在 Azure 上有一个备用副本，随时准备接管以防万一。这样，如果某个服务提供商发生故障，我们的业务不会受到影响。多云策略还可以帮助我们跨不同区域或行业应对合规要求。然而，我们需要为管理多个平台上的不同工具、API 和配置所增加的复杂性做好准备。这要求我们的团队熟练掌握所有使用的平台，并且需要跟上各个平台的更新和变化。

#### 混合云

混合模式让我们能够结合两者的优势，将本地基础设施与云资源相结合，甚至将多个云服务提供商混合使用。如果因为遗留应用、数据驻留法或严格的监管要求，我们无法完全迁移到云端，这种方式尤其有用。

在混合环境中，我们可能将敏感数据保留在本地，而将不太关键的工作负载迁移到云中。像 AWS Outposts、Azure Arc 和 Google Anthos 这样的工具帮助我们缩小本地环境和云环境之间的差距。挑战在于确保两种环境中的一切能够无缝协作，尤其是在网络、安全性和数据一致性方面。

混合云也可以看作是迁移过程中从两个云服务提供商之间或从本地迁移到云的一种过渡阶段。这种迁移通常需要几个月到几年时间，具体取决于规模或复杂性。

总结来说，我们选择的模型需要反映出我们所追求的技术和商业目标。对于关键任务应用，可能需要多云的冗余和可用性，而较简单的应用则可以单云环境中运行。如果我们需要考虑遗留系统或特定的合规要求，混合模型可能是最好的选择。最终，我们做出的选择必须与我们的运营能力和长期目标相一致。

### 使用迁移处理计划

在这里要问的第一个问题是：“在云原生迁移的背景下，什么是处理计划？”简而言之，处理计划是一个框架或模型，帮助组织做出关于如何迁移每个工作负载的重要决策。它可以被视为一种决策工具，用于对工作负载进行分类和优先级排序，以确定将基础设施、应用程序和数据迁移到云的最佳方法。

当我们做出工作负载决策时，使用正确的框架至关重要，无论是 AWS 的 7 个 R、Azure 的云采用框架，还是 GCP 的 6 个 R。这些框架帮助我们与目标保持一致，并确保我们为每个应用选择最佳的迁移方法。

虽然不同的云服务提供商（CSP）对它们的处理计划有不同的命名，但它们都围绕着相同的基本原则，具体如下：

+   **重新托管（提升与迁移）**：将工作负载迁移到云中，所需的变更最少。这是我们在时间紧迫或需要保持简单时的首选方法，适用于 AWS、Azure 和 GCP。

+   **重新平台化**：进行小范围的调整以优化云环境，而不是进行全面的改造。我们可能会使用 AWS Elastic Beanstalk、Azure App Services 或 GCP App Engine 为我们的应用提供更好的平台，同时保持熟悉的操作环境。

+   **回购（SaaS）**：在合适的情况下，将本地软件替换为 SaaS 解决方案。比如 Microsoft 365 或 GCP 的 Workspace，管理减少，效率提高，而且由他人处理繁重的工作。

+   **重构**：当我们需要重新设计一个应用，使其真正云原生时，我们会使用像 AWS Lambda、Azure Functions 或 Google Cloud Functions 这样的工具进行无服务器处理，或者将应用拆分为微服务并使用 Kubernetes。

+   **淘汰**：这涉及到剔除我们不再需要的应用程序。无论是为了简化复杂性还是节省成本，关闭过时的系统对于保持我们的云环境精简至关重要。

+   **保留**：有时候，最好将某些应用保留在本地，尤其是它们不适合云环境或合规要求需要这样做。我们可以通过 AWS Outposts、Azure Arc 或 Google Anthos 等混合解决方案来管理这些应用。

+   **迁移**：根据需要在不同云之间迁移工作负载。灵活性至关重要，云之间的切换使我们能够根据业务发展优化性能、成本或合规性。

注意事项

我们也在*第七章*第一部分中简要讨论了 7R 理论，特别是提升与迁移。

### 选择你的云平台

无论我们是在使用 AWS、Azure、GCP，还是它们的混合组合，确保平台符合我们的业务和技术需求是至关重要的。每个云服务提供商都有其优势，因此了解我们真正需要什么是关键，以避免可能限制性能或增加成本的不匹配情况。以下是在选择云平台之前需要考虑的一些因素。

#### 服务和功能评估

我们需要从审视每个提供商提供的核心服务开始，特别是在对我们业务至关重要的领域。例如：

+   AWS 以其广泛而成熟的生态系统而著称，尤其是在计算（EC2）、无服务器计算（Lambda）和数据湖（S3）等领域。

+   另一方面，如果我们已经在 Microsoft 生态系统中，Azure 可能是更好的选择，因为它与 Active Directory、Office 365 和 Windows Server 等工具的无缝集成。

+   GCP 在其 AI/ML 能力、Kubernetes（GKE）和大数据解决方案（如 BigQuery）方面通常表现出色。

决定选择哪个提供商适合我们的需求，需要评估这些核心服务如何与我们当前和未来的工作负载相匹配。

#### 成本考虑

虽然云服务提供商通常采用按需付费模式，但各个平台的定价模型差异巨大。我们不仅需要评估服务的前期成本，还需要根据可扩展性、存储和数据传输费用来评估长期的财务影响。AWS、Azure 和 GCP 在计算、网络和存储层级等方面有独特的定价结构。

在多云环境中，定价变得更加复杂。我们需要考虑不同提供商之间的数据外流成本，确保平台间的数据传输不会导致意外的费用。

另外需要注意的是，混合云策略可能会带来不同的费用，例如从私有数据中心到云服务提供商的直接连接费用、VPN 成本和额外的数据传输费用。

定价工具

工具如 AWS 价格计算器、Azure 价格计算器或 Google Cloud 价格计算器可以帮助我们根据使用模式估算这些费用。

#### 合规性与安全

如果合规性是一个主要因素（例如 GDPR、HIPAA、PCI-DSS 等），我们需要确保所选平台具备必要的认证和数据驻留选项。AWS、Azure 和 GCP 都提供强大的安全性和合规性服务，但这些服务的深度和地区可用性可能有所不同。

例如，如果我们在欧洲处理敏感数据，AWS 和 Azure 提供了与 GDPR 更加符合的特定区域。如果我们专注于具备隐私要求的 AI/ML 工作负载，Google Cloud 可能会更具吸引力。

注意事项

正如在 *第六章* 中讨论的，合规性和认证是云服务提供商与客户之间的共享责任。

#### 多云与混合云策略

如果我们计划采用多云战略，我们需要仔细评估互操作性。AWS、Azure 和 GCP 之间的服务能否轻松协同工作？我们需要决定是否使用像 Terraform 这样的云中立工具进行基础设施即代码管理，或使用 Kubernetes 进行容器编排来标准化。这可以确保我们不会被锁定在某一平台上，从而在云之间迁移工作负载或扩展操作时具有更大的灵活性。

对于混合云设置，AWS Outposts、Azure Stack 和 Google Anthos 等解决方案使我们能够将云延伸到本地数据中心，从而无缝管理跨环境的工作负载。选择哪种解决方案取决于我们如何管理本地与云之间的连接，以及我们所运行的特定工作负载。

#### 延迟和区域布局

我们决策过程中的另一个关键因素是云服务提供商的地理覆盖范围。如果低延迟对用户体验至关重要，我们需要选择一个在客户群附近有强大区域布局的平台。AWS 拥有最广泛的全球基础设施，但 Azure 和 GCP 也提供了强大的覆盖范围。我们需要分析每个提供商的可用区域和可用性区域，并确定它们与我们地理需求的契合度。

最终，选择合适的云平台归根结底在于理解技术需求和更广泛的商业目标。无论我们是全力投入单一云服务提供商，还是将工作负载分散到多个云服务提供商之间，每个决策都应通过对服务、成本、合规性和可扩展性选项的深入评估来支持。

### 确定明确的时间表和里程碑

在规划云迁移时，设定现实的时间表和明确的里程碑是确保项目按计划推进、避免遗漏任何步骤的关键。我们可以这样分解任务，添加必要的技术深度，以保持流程紧凑且可预测。

#### 评估阶段（发现与规划）

在我们迁移任何工作负载之前，我们需要为当前环境的深入评估分配足够的时间。这意味着需要使用像 AWS 迁移评估工具、Azure Migrate 或 Google Cloud Migrate 这样的发现工具，来绘制出我们所有的依赖关系、应用连接性和性能指标。这些工具为我们提供了一个完整的视图，避免后续出现意外情况。

如果我们回顾一下“评估当前环境”章节中的表格，我们可以看到这为我们设定了明确的要求，以确保我们可以开始规划我们的实例类型或需求。

| **主机名** | **CPU 核心数** | **操作系统** | **操作系统版本** | **总内存** |
| --- | --- | --- | --- | --- |
| app-server01.local | 4 | Windows Server 2019 | 1809 | 8192 |
| linux-db01.local | 4 | Ubuntu 20.04 | Focal Fossa | 16384 |
| mssql-db01.local | 8 | Windows Server 2016 | 1607 | 32768 |
| app-server02.local | 2 | Windows Server 2019 | 1809 | 8192 |
| backup-server.local | 4 | Windows Server 2012 R2 | 9600 | 8192 |
| web-server01.local | 2 | RHEL 8 | Ootpa | 4096 |
| dev-server.local | 4 | Windows 10 Pro | 21H1 | 16384 |
| linux-app01.local | 2 | CentOS 7 | Core | 8192 |
| storage-server.local | 16 | Windows Server 2019 | 1809 | 65536 |
| dns-server.local | 1 | Ubuntu 18.04 | Bionic Beaver | 2048 |
| mail-server.local | 4 | Windows Server 2016 | 1607 | 8192 |
| log-server.local | 8 | Ubuntu 22.04 | Jammy Jellyfish | 16384 |

表格 12.2 - 迁移评估数据

在这一阶段，技术审计至关重要。我们需要识别可能会受到迁移影响的网络配置、数据库、安全策略和存储设置。设定一个里程碑来完成这一发现阶段，确保我们在做出任何决策之前，手中有所有关键数据。可以把它当作为设定一个两周的窗口期，用来运行迁移评估，并在数据收集完毕后与关键利益相关者进行审查。

#### 概念验证（PoC）/ 试点测试

在我们完成评估后，不能直接跳入全面迁移。我们需要一个**概念验证**（**PoC**）或试点测试。在这里，我们将选择一个或两个非关键应用，并将其作为试验进行迁移过程。例如，我们可能会使用 AWS 应用迁移服务将一个小型应用提升并迁移，或者尝试在 Azure 或 GCP 上使用 Kubernetes 重构某个组件。

这里的关键里程碑是成功完成 PoC。这可以帮助我们验证所选择的工具和流程是否能够在大规模环境下工作。根据我们测试应用的复杂性，迁移测试大概需要 2 到 4 周的时间。在这个过程中，定期检查非常重要，以跟踪迁移的效果，并在规模扩展之前解决任何问题。

#### 全面迁移

一旦 PoC 得到批准并运行顺利，我们就进入全面迁移阶段。这是技术性迅速发展的时候。我们会设立检查点，例如确保我们的 IAM 角色、网络设置和安全组在我们开始迁移任何内容之前得到正确配置。像 AWS 数据库迁移服务、Azure 数据库迁移或 GCP 的数据传输这样的工具在处理数据库迁移时起到关键作用。

在这个阶段，我们的里程碑应涉及分批迁移。我们不会一次性将所有内容都转移到云端。相反，我们会设定目标，比如在 4-6 周内迁移我们应用的 20%，然后重新评估、回顾性能，并在推进之前进行微调。

#### 优化和微调

当应用程序在云端时，工作并没有结束。这个阶段涉及回顾性能，合理调整资源大小，并实施自动缩放以有效满足需求。像 CloudWatch（AWS）、Azure Monitor 或 GCP Operations Suite 这样的监控工具在这里至关重要，用于跟踪性能并识别我们设置中的任何低效性。

我们将为这个阶段设定里程碑，以确保我们的云架构在性能和成本上都得到了优化。这意味着定期性能审查和关注资源利用率，以便我们可以根据需要进行调整。与所有利益相关者进行的迁移后审查让我们评估什么有效、什么不行，以及我们如何优化未来的迁移。

#### 定期的利益相关者检查

在整个过程中，与利益相关者（无论技术性或非技术性）的定期沟通至关重要。我们将安排每周或每两周更新，以确保一切与更广泛的业务目标保持一致，并确保技术团队达到其关键的里程碑。详细的迁移运行手册、架构图表和定期的进展更新让所有人都保持在同一页面上，并且允许我们在需要时进行调整。

通过为每个阶段设定清晰的里程碑，评估、PoC、迁移和优化，我们保持组织性，避免问题变成更大的问题，并确保整个过程按计划进行。定期与利益相关者的沟通意味着没有人被置于黑暗中，我们可以调整时间表或流程，以确保从头到尾的平稳迁移。 

没有战略的云迁移是在找麻烦。一个扎实、深思熟虑的计划确保你不只是把一切都扔到云端然后指望一切都顺利。优先考虑你的应用程序，使用像 AWS 的 7 个 R 框架，并选择正确的云模型，为成功奠定基础。有了清晰的计划，你将避免不必要的延迟、低效以及因为毫无准备就潜水而来的昂贵错误。

策略的另一部分是确保业务各个方面的利益相关者都对迁移活动有所了解。我们将在下一节更详细地介绍这一点。

# 最低利益相关者承诺

在云采纳方面，普遍存在一种误解，认为这仅仅是一个技术性的工作，IT 团队可以独立完成。然而，现实情况远非如此。云迁移不仅仅是另一个技术项目，它是业务运作方式的根本转变。为了成功，它需要各级利益相关者的全程参与，从 C-suite 高层到部门负责人，再到技术负责人。

注意事项

*第五章*更详细地介绍了文化转变。

但我们常常看到的是最小利益相关者承诺，这是一种不干预的方法，决策者只有在问题出现时才会参与。他们可能在开始时提供高层次的支持，但随后逐渐淡出背景，假设技术团队能处理一切。这种脱节导致了延迟、目标不一致、预算超支，最坏的情况是迁移失败。如果利益相关者从一开始就没有积极参与，整个项目就可能失去方向，直到你意识到，迁移已经偏离了原定轨道。

让我们来分析一下为什么这是一种反模式，以及利益相关者应如何承担云采纳的责任。

## 了解反模式：最小利益相关者承诺

了解在这些情况中，什么可能出错与什么可能正确一样重要。下面我们回顾一些关于最小利益相关者承诺的关键要点。

### 被动利益相关者参与的问题

问题在于，如果利益相关者没有充分参与，就会带来各种麻烦。IT 团队可能专注于运营效率或成本节约，而业务方可能更关注可扩展性或客户体验。如果没有利益相关者的定期参与，这些目标最终可能会发生冲突。

当关键决策者未参与时，延迟开始悄然出现。关于工作负载优先级、资源分配或迁移范围变更的决策往往会被拖延。这是因为没有人积极地掌舵。

那么会发生什么呢？项目拖延，挑战出现时，没人感到有责任去解决。这种缺乏责任感导致了缓慢的反应和糟糕的问责制。

### 被动而非主动的方法

利益相关者的最低参与度导致了一种被动的做法，只有当某些问题出现时才做决策。

结果如何？你最终会得到匆忙的“*搬迁与转换*”迁移，即将工作负载迁移到云端，而没有进行优化。这是一次错失的机会，最终你会为一个比原来环境差不多的云环境支付更多费用。

如果没有主动规划，迁移工作往往集中在短期收益上，比如迁移工作负载以避免硬件更新费用，或者为了满足监管截止日期。但这种做法忽略了更大的图景，忽视了优化、可扩展性和长期的云效益。这样我们就错失了云原生功能的全部潜力，最终云架构可能和我们遗弃的旧架构一样低效和昂贵。

这两种做法在许多失败的迁移中已经是常见做法，导致了成本增加、延误，以及企业因缺乏方向而重新回到传统技术。

## 利益相关者应如何参与云采用

为了让云迁移真正成功，光是让利益相关者站在旁边是不够的。积极、持续的参与才是关键。以下是利益相关者从一开始就能在推动云采用成功中发挥关键作用的方式。

谁是“利益相关者”

在云迁移中，利益相关者是对迁移成功具有切身利益的个人或团体。这些可以是高层领导团队、IT 管理层和架构师、应用程序所有者或开发人员、运营团队、财务团队或最终用户。这份名单既不详尽也不最终，任何与应用、数据或基础设施有接触的人都可以是利益相关者。

### 清晰和定期的沟通渠道

从一开始，利益相关者就需要参与其中。我们说的是建立定期会议，让技术和业务团队共同讨论进展、挑战和即将做出的决策。通过这种方式，大家保持一致，避免了阻碍项目进展的隔离式沟通。

尝试以下做法：

+   **指导委员会**：成立一个由技术和业务团队成员组成的指导委员会。这样，决策可以通过合作方式做出，确保迁移既符合运营目标，又符合业务目标。

+   **检查会议**：每周或每两周的检查会议有助于推动迁移进程。这些会议不仅仅是进度更新，更是调整战略的机会，帮助在问题变得更严重之前解决潜在的障碍。

### 定义业务目标

云采用需要被视为一种业务转型，而不仅仅是技术迁移。利益相关者应定义与迁移相关的明确、可衡量的业务目标，无论是提高敏捷性、降低成本，还是实现更快速的产品发布。

利益相关者需要负责这些业务目标，确保在整个迁移过程中定期回顾这些目标，以确认项目仍然在正确的轨道上。一些实用的步骤可能包括：

+   **目标对齐**：确保云迁移与更广泛的业务目标紧密相连，比如改善客户体验或简化运营。

+   **关键绩效指标（KPIs）**：设立可衡量的成功指标，无论是减少停机时间、更快速的部署，还是成本节省。这些指标应在整个迁移过程中指导决策。

### 资源分配与优先排序

资源分配是利益相关者真正需要亲自参与的地方。延误和预算问题通常是因为对资源流向缺乏足够的监督。利益相关者需要参与工作负载的优先排序，确保关键应用程序获得所需的关注。这里的关键步骤可能包括：

+   **预算监督**：利益相关者应积极管理迁移预算，确保财务资源得到恰当分配，特别是对于关键工作负载和迁移后的云优化。

+   **工作负载优先排序**：让利益相关者参与决定优先迁移哪些工作负载。这有助于技术团队集中精力处理对业务带来即时价值的高影响应用程序。

### 定期审查与灵活性

云迁移不是一条直线，而是一个迭代过程。利益相关者需要定期审查进展，并准备根据需要做出调整。这不是一个“设定然后忘记”的情况。会有挑战，利益相关者需要保持参与，迅速做出调整。可以帮助的实际步骤有：

+   **季度战略审查**：除了定期的检查，利益相关者应举行季度审查，以评估迁移是否达到了预期的业务成果。

+   **迁移后优化**：迁移完成后，工作并未结束。利益相关者应该继续参与迁移后的优化，确保云设置在性能和成本效率上得到微调。

通过这种方式的参与，我们终于能够将云项目与真正的商业目标对齐，带来我们在传统、不干预方式中失去的灵活性和创新。

## 利益相关者的技术考虑因素

当涉及到利益相关者的参与时，需要在利益相关者层面考虑一些问题。

### 使用云原生治理工具

治理在云环境中至关重要。利益相关者应推动使用云原生工具来执行政策、管理多个账户，并确保安全性和合规性始终得到保障。例如：

+   **AWS 控制塔或 Azure 蓝图**：这些工具帮助在不同环境中管理账户并应用治理政策，确保我们始终保持合规和安全。

+   **GCP 组织策略**：这确保了对资源的集中控制，使得在多云环境中治理变得更加容易。

### 设定安全与合规基准

云迁移带来了新的安全挑战，利益相关者需要关注这一点。从第一天起，像 IAM、加密和日志记录等安全功能应该就开始实施，合规基准也需要作为迁移计划的一部分。可以考虑应用：

+   **定期合规审计**：通过定期审计，确保云环境符合行业标准。

+   **安全评审**：安全应该融入到迁移过程中，而不是事后的考虑。利益相关者应该确保在迁移的每个阶段都进行安全评审。

无论你采取什么方法，积极的利益相关者承诺对成功至关重要。

最小化利益相关者的参与是导致延误、预算超支和错失机会的根源。为了确保云迁移成功，必须从头到尾让利益相关者参与其中，确保决策与业务目标一致，资源分配有效，并根据需要进行调整。通过保持参与和积极主动，利益相关者能够确保云采用发挥其全部价值，并为企业的长期成功奠定基础。

如果利益相关者没有参与并保持信息通畅，由于缺乏明确的方向或仓促的决策，很容易在云中复制本地的概念。

# 复制本地安全控制

组织在云迁移过程中犯的最大错误之一就是试图将旧的本地安全控制移植到云中。这看起来可能是一个安全的选择，毕竟这些控制措施在旧环境中运作良好，那为什么还要重新发明轮子呢？

但现实是，云的运作方式完全不同，拖着那些传统的控制措施走上云端，可能弊大于利。这不仅会造成安全漏洞，还会增加运营负担，拖慢团队的效率并消耗资源。

本地安全模型基于静态环境、固定的网络边界以及需要手动配置的工具。但云环境是动态的，随着资源的增加和减少而不断变化，以满足需求。安全必须像云本身一样灵活和可扩展，这就是云原生工具的作用所在。坚持使用熟悉的做法可能看起来像是安全的选择，但它是一种反模式，可能在安全性、效率和运营复杂性上带来代价。

让我们深入探讨为什么复制本地安全控制是一个糟糕的主意，并探讨如何调整你的安全策略，以充分利用云原生解决方案的强大功能。

## 了解反模式：复制本地安全控制

在迁移到云时，容易回归我们熟悉的做法，使用多年在本地环境中依赖的相同安全控制。但现实是，在传统环境中有效的做法并不适用于以云为优先的世界。试图在云中复制这些旧的控制措施通常会导致低效、漏洞和运营上的头痛问题。我们不仅没有增强安全性，反而最终得到了一堆过时的控制措施，未能充分发挥云的优势。

在这一部分，我们将分析这种方法为何不足，并探讨转向云原生安全实践的真正价值。

### 本地安全在云中的问题

在传统的本地安全中，假设很简单，一旦进入网络，便被信任。网络边界内的所有内容都被认为是安全的。但在云中，这种模式不再适用。随着工作负载分布在各个地区，资源按需扩展和收缩，旧的思维方式很快就崩溃了。这时，**零信任**理念应运而生。

在云环境中，我们不能仅仅因为用户、设备或应用程序的网络位置就信任它们。零信任颠覆了这种假设，要求每次都进行验证，无论是用户、设备还是工作负载。所有事物都必须证明自己是安全的，才能获得访问权限。转向零信任模型对于拥抱真正的云原生安全至关重要。

零信任的关键支柱包括：

+   **身份验证**：始终确认请求访问的是谁或什么。

+   **最小权限访问**：仅限每个用户或服务所必需的访问权限。

+   **持续监控**：实时跟踪活动，检测威胁并在风险出现时加以缓解。

+   **微分段**：将网络划分为更小的部分，以隔离资源并限制攻击面。

+   **设备安全**：确保所有设备在授权访问之前符合严格的安全标准。

通过采用这些核心原则，我们摒弃了“网络边界内的任何东西都是安全的”这一过时的观念。零信任确保云环境中的每个组件都被持续验证，从而提供对内部威胁、配置错误和可能在传统本地部署中遗漏的安全漏洞的更强保护。这是我们在日益复杂的云环境中保持安全所需的思维方式转变。

尝试在云中应用传统的安全控制往往意味着错失云所提供的灵活性和可扩展性。传统工具无法跟上云环境变化的步伐，通常需要手动调整，导致操作变慢。更糟糕的是，如果这些工具与云原生服务集成不良，可能会使你的云基础设施暴露于风险之中。

例如，虽然本地环境高度依赖防火墙来阻止未经授权的流量，但云环境要求更加细粒度的安全控制。这时，身份和访问管理（IAM）发挥作用。在云中，不仅仅是阻止不良流量；而是确保只有正确的用户、服务和应用程序才能访问它们所需的资源——没有多余，也没有不足。仅依赖传统的基于网络的安全工具可能会在访问管理上留下危险的漏洞。

### 常被忽视和误解的共享责任模型。

迁移到云时，最重要的概念之一是共享责任模型。在传统的本地环境中，你控制一切，从物理硬件到其上运行的应用程序。但在云中，安全责任由你和云服务提供商共同承担。提供商负责基础设施，如数据中心的物理安全和它们之间的网络，而你则负责保护构建在其上的应用程序、数据和身份管理。

未能理解这一区别通常会导致安全配置薄弱。例如，认为云服务提供商会为你处理加密或访问控制，可能会导致数据泄露或未经授权的访问。传统的本地安全模型没有考虑到这种共享责任，试图将其直接复制到云中，往往会导致安全防护存在严重漏洞。

## 如何从本地安全转向云原生安全

在云中复制传统的本地安全方法是行不通的。为了充分利用云基础设施，我们需要调整思路，采用云原生的方法。在本节中，我们将详细阐述如何摆脱这些过时的安全模型，并开始全面利用云原生工具和实践。

在审查你的云原生安全实施时，请考虑以下提示

+   **拥抱基于身份的安全模型**：在云中，安全围绕身份展开，无论是用户、应用程序还是服务。**IAM**（**身份和访问管理**）是你的第一道防线。通过为每个用户和资源分配特定的角色和权限，你可以控制谁可以访问什么资源，以及在什么条件下访问。你可以考虑的实际步骤包括：

    +   **采纳最小权限原则**：确保用户和服务仅拥有其绝对需要的权限。在 AWS 中，这意味着定义细粒度的 IAM 策略。在 Azure 和 GCP 中，类似的基于身份的控制允许你在粒度级别上管理访问权限。

    +   **实施多因素认证（MFA）**：MFA 应当对所有访问敏感资源的行为强制执行。这一额外的安全层确保即使凭据被泄露，攻击者也无法在没有第二因素的情况下获得访问权限。

    +   **定期审计**：定期检查权限。云环境变化迅速，上个月合理的角色今天可能已经过于宽松。定期审计帮助你保持对访问权限的掌控。

注意

有关更多信息，请查看*第六章*以及临时凭证的使用

+   **使用云原生安全工具**：与其强行让本地安全工具在云中工作，不如利用专为云环境构建的云原生工具。每个主要的云提供商都提供了一整套安全服务，旨在监控、保护和扩展你的基础设施。

    以下工具可以帮助你实施良好的云原生安全基础。

    +   **AWS Shield 和 AWS WAF**：使用 AWS Shield 和 **AWS Web Application Firewall (WAF)** 保护你的应用免受 DDoS 攻击和其他恶意流量的影响。Azure 和 GCP 提供类似的服务，如 Azure DDoS Protection 和 GCP Cloud Armor。

    +   **集中式安全仪表板**：使用像 Azure Security Center 或 Google Cloud Security Command Center 这样的集中平台来全面了解你的安全态势，并迅速识别任何漏洞或配置错误。

    +   **自动化威胁检测**：像 AWS GuardDuty、Azure Advanced Threat Protection 和 Google Cloud Security Scanner 这样的工具持续监控可疑活动，并可以在威胁升级前提醒你。

+   **自动化安全管理与监控**：云环境不断发展变化，这使得手动的安全管理变得不切实际。幸运的是，云为我们提供了以下工具，自动化了许多安全监控和事件响应的方面，从而减少了运营负担，并确保对潜在威胁能够更快响应。

    +   **CloudTrail/CloudWatch**：在 AWS 中，使用 CloudTrail 和 CloudWatch 来记录和监控云环境中的所有活动。类似地，Azure Monitor 和 GCP 的 Operations Suite 提供实时的云原生洞察和警报。

    +   **自动化响应威胁**：为威胁响应配置自动化工作流。例如，如果 AWS GuardDuty 标记出可疑活动，你可以自动撤销凭证或触发安全组更改以阻止未授权访问。

    +   **安全审计与合规性**：使用像 AWS Config、Azure Policy 和 GCP Policy Intelligence 这样的工具安排定期的自动化安全审计。这可以确保你的环境始终符合安全政策和行业法规。

+   **通过云原生解决方案减少运营负担**：坚持使用本地安全工具不仅限制了你的安全能力，还增加了大量的运营负担。在云环境中手动配置防火墙、管理静态 IP 地址或不断调整 VPN 设置既费时又容易出错。云原生解决方案减少了对手动干预的需求，自动化了许多安全管理过程。在规划迁移时，请牢记以下几点：

    +   **减少手动配置**：用动态的基于身份的访问控制替换手动的防火墙和 VPN 设置。这减少了不断更新和监控的需求。

    +   **集中式治理**：使用云原生治理工具，如 AWS Control Tower、Azure Blueprints 或 Google Cloud 组织策略，来管理多个账户中的策略，执行最佳实践，并自动化合规性管理。  

    +   **自动化事件响应**：通过云原生自动化工具，您可以消除应对安全事件中大部分的人工工作。自动撤销被盗的凭证、调整防火墙规则，或者根据需要启动备份实例——这一切无需人工干预。  

总结来说，您需要拥抱云原生安全以消除传统的负担。在云中复制本地的安全控制不仅效率低下，而且充满风险。传统工具和手动流程无法跟上云环境快速变化的节奏，导致安全漏洞和操作效率低下。  

通过采用云原生安全模型、自动化关键流程，并利用 AWS、Azure 或 GCP 提供的完整工具套件，您可以建立一个比以往更强大、更具可扩展性和更高效的安全态势。  

关键点是：云安全不仅仅是一个复制粘贴的工作。它需要思维方式的根本转变。通过正确的方法，相关方和技术团队可以顺利地从过时、劳动密集型的安全模型过渡到一个云原生环境，从而最大化安全性和操作效率。

在本章的最后部分，我们将回顾教育和知识传递对成功迁移的重要性。  

# 低估技能差距  

在向云端迁移时，最关键且常被忽视的因素之一是团队内部的技能差距。许多组织在采用云技术时，认为现有的技术团队能够顺利适应新的环境。然而，云基础设施的运作原理与传统 IT 不同，假设原有技能能够直接迁移，往往会导致延误、效率低下，甚至失败。低估技能差距可能会导致诸如配置错误、错失优化机会，或最严重的安全漏洞等问题，直到为时已晚才被发现。  

云迁移不仅仅是技术上的转变，更是思维方式的转变。如果没有适当的培训、支持和对所需技能的现实理解，组织往往会发现自己难以充分利用云的能力。让我们更深入地探讨这一反模式，并探索有效弥补技能差距的实际策略。  

## 理解反模式：低估技能差距  

在云技术采用方面，许多企业认为，如果某人在传统 IT 或数据中心管理中非常熟练，那么他们自然能够处理云操作。然而，不同的云服务提供商（CSP）有着完全不同的运作范式。  

基础设施即代码、无服务器和容器编排等术语不仅仅是流行词；它们需要对新工具和方法有深入理解。没有正确的技能，云迁移很容易偏离轨道。让我们更详细地分析一下传统技能与云技能，以及它们对云采用的影响。

### 传统技能集与云技能集

在传统的本地环境中，管理基础设施意味着物理部署硬件、安装软件并手动管理一切。另一方面，云环境需要精通自动化、动态扩展和对云原生服务的深入理解。弹性、自动扩展和安全模型等概念对许多技术团队来说通常是新的，没有专注的培训，技能缺口很快就会显现。

### 对云采用的影响

未能解决技能差距不仅会导致迁移时间线延长；它还会直接影响云采用的成功。缺乏必要技能的团队可能复制旧的本地流程，这些流程并未针对云进行优化，导致工作流程低效和配置错误。更糟糕的是，糟糕的安全实践可能会开放漏洞，从而危及整个云环境的完整性。

## 如何弥补技能差距

承认存在技能差距是第一步，但这还不够。你需要一个坚实的计划来弥补它。以下是我们的做法。

### 投资于云培训和认证

最快弥补技能差距的方式是通过结构化培训和认证。每个主要云服务提供商都提供一系列认证，旨在为团队提供处理云架构、运营和安全所需的知识。AWS、Azure 和 GCP 都有针对不同角色（从架构师到开发人员再到 DevOps 工程师）的学习路径。

+   **确定关键培训领域**：不要仅仅送团队参加通用的云培训。确定团队需要提升的具体领域。是自动化？安全？容器管理？专注于能为组织带来最大价值的领域。

+   **鼓励认证**：推动进行正式的云认证。AWS 认证解决方案架构师、Azure 管理员和 Google Cloud 工程师认证为团队提供基础和高级知识。这些认证不仅验证技能，还能使你的团队始终掌握云最佳实践。

+   **持续学习**：云平台发展迅速。使学习成为持续过程，鼓励团队定期参与网络研讨会、动手实验室和新的认证考试。

### 培养跨职能协作的文化

低估技能差距通常源于将技术团队孤立在各自的领域中。当开发人员、运营和安全团队朝着共同目标合作时，云操作能够蓬勃发展。创建跨职能协作的文化不仅能弥补技能差距，还能确保整体云操作的顺利进行。请考虑以下几点。

+   **跨培训**：鼓励跨职能培训，让开发人员了解基础设施，运营人员了解更多关于代码和自动化的内容。这有助于在团队内创建 DevOps 甚至 DevSecOps 的思维方式。

+   **协作项目**：开展跨职能团队共同合作解决现实问题的项目。例如，从零开始构建一个基于云的应用程序可能需要开发、安全和运营团队齐心协力。这种方式促进了团队合作，同时也能实时提升技能。

+   **导师制度**：将熟悉云环境的资深工程师与那些仍在适应的工程师配对。导师制度加速学习，同时促进跨部门的协作。

## 平衡技术技能与软技能

这不仅仅是技术知识的问题。团队还需要具备在云环境中进行沟通、协作和解决问题的能力。云采用是一个全公司范围的举措，而不仅仅是一个技术项目，它要求业务和技术团队密切合作。弥合技能差距往往需要在这些团队之间促进更好的沟通。以下的指导对于确保平衡技术与软技能至关重要：

### 在整个组织中培养云流畅度

云采用影响着整个组织，而不仅仅是 IT 部门。为了最大化云迁移的效益，整个业务的利益相关者需要对云原则有基本了解。无论是财务跟踪云成本，安全管理合规性，还是法律处理云合同，确保各部门具备云流畅度是关键。请记住以下几点：

+   **利益相关者工作坊**：为非技术团队设立云基础知识工作坊，帮助他们理解云如何影响他们的角色，以及为什么云对业务至关重要。

+   **云特定 KPI**：为不同部门定义云特定的关键绩效指标（KPI）。这有助于确保所有业务单位理解云采用的影响和好处，并保持所有人对迁移目标的统一认识。

### 确立明确的责任归属

技能差距的最大来源之一是云项目中缺乏明确的责任归属。当角色和责任不明确时，人们往往会依赖他们熟悉的方式，这可能导致技术债务和运营低效。为确保责任和专业技能在需要的地方得以发展，必须明确云项目的责任归属。为了实现明确的责任归属：

+   **定义角色**：在云迁移过程中，明确谁负责什么。无论是基础设施、安全性，还是部署，都要明确责任，以确保没有遗漏。

+   **创建云技术冠军**：在组织内指定云技术冠军——那些负责特定云技术领域并推动跨团队最佳实践的人。

弥合知识差距是云成功的关键

低估云采纳中的技能差距是常见的反模式，但它也是最容易解决的。通过投资于针对性的云培训，培养协作文化，并在整个组织内建立云技能流利度，你可以弥补这些差距，确保顺利的迁移。云采纳不仅仅是技术问题，它关乎建立一支具备技能和心态的团队，以便在快速发展的环境中取得成功。

# 总结

云迁移远不止是迁移工作负载，它是一次完整的转型，需要精心规划、强有力的利益相关者参与，以及向现代化、云原生实践的转变。在这一章中，我们探讨了阻碍进展的一些常见反模式，从规划不足、利益相关者参与不力到从本地环境延续下来的过时安全实践。

使用像 AWS 的 7Rs 框架或 Azure 和 GCP 的等效选项，我们可以就每个工作负载做出更聪明的决策，确保每一步都符合技术和业务目标。通过战略性地优先考虑应用程序，并选择合适的云模型——无论是单一云、多云还是混合云——我们可以减少风险并制定出能够创造实际价值的迁移路线图。解决技能差距同样至关重要，因为它使团队能够接受成功在云端所需的工具和方法。

这不仅仅是迁移系统；它是为灵活性、可扩展性和创新奠定基础。有了清晰的战略、协作的团队精神和现代化的方法，我们不仅可以顺利迁移到云端，还能在那里茁壮成长。
