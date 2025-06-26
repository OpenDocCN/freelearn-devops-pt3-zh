# *第十六章*：模拟测试

AWS 认证云从业者考试是一个 90 分钟的多选多答题型考试。AWS 收取 100 美元的报名费用，报名可以通过他们的网站[`www.aws.training/certification`](https://www.aws.training/certification)。考试可以在多个国家的测试中心进行，或者可以在家在线参加。

本章将为你提供两个完整的模拟考试，你可以用它们来评估自己是否准备好参加正式的 AWS 认证云从业者考试。

这些测试包括 65 道问题，每道题目后都有答案解释，位于本章末尾。为了有效使用这些测试，确保你遵循以下步骤：

+   每次测试请留出 90 分钟时间，确保不受干扰。你可以使用手机设置闹钟。

+   拿出纸和笔，对于每个问题，写下问题编号和正确答案。

+   确保如果你在完成测试后还有剩余时间，能再检查一遍你的答案。

+   在 90 分钟结束时，检查你的答案，并与本章最后的答案解释部分进行对比。

你希望在进入考场前能正确回答所有问题。你应该利用这些测试来找出自己没有答对的问题，然后回去复习涵盖该问题概念的章节。这样，你就能弥补知识的空白，并能更好地为正式考试取得高分做准备。

# 模拟测试 1

1.  哪种类型的云服务模型最像本地环境，你可以在其中配置虚拟基础设施组件，如计算、网络和存储服务，以便托管你的应用程序？

    1.  **软件即服务** (**SaaS**)

    1.  **平台即服务** (**PaaS**)

    1.  **基础设施即服务** (**IaaS**)

    1.  **功能即服务** (**FaaS**)

1.  你的公司计划将所有应用程序和服务迁移到云上，但希望分阶段迁移工作负载。这将要求你确保本地基础设施与在 AWS 上部署的应用程序之间有一段时间的连接。你需要建立哪种云部署模型？

    1.  私有云

    1.  公有云

    1.  混合云

    1.  多云

1.  以下哪些陈述是选择特定 AWS 区域来部署应用程序的有效理由？（选择两个）

    1.  你的组织将选择一个特定的 AWS 区域，以确保你的应用程序更靠近终端用户，从而减少延迟。

    1.  如果你的组织有特定的合规性或数据驻留法律要求，那么你选择 AWS 区域时将受到此要求的制约。

    1.  您的组织应选择离其位置较近的区域，因为您的 IT 员工需要访问 AWS 数据中心以设置服务器和网络设备。

    1.  您的组织会选择一个与业务已有合法存在的 AWS 区域位置。这是因为，除非您在该区域有合法的成立，否则无法访问其他区域。

    1.  您的组织会选择一个 AWS 区域，该区域提供更高的可变成本，但较低的前期成本。

1.  以下哪个 AWS 全球基础设施组件可以让您缓存内容（视频、图片和文档），并在用户尝试下载时提供低延迟访问？

    1.  AWS 区域

    1.  可用区

    1.  边缘位置

    1.  本地区域

1.  以下哪些 AWS 服务可以帮助您设计混合云架构，并使您的本地应用程序能够访问 Amazon S3 云存储？

    1.  Amazon Snowball Edge

    1.  AWS Storage Gateway

    1.  Amazon 弹性块存储

    1.  Amazon CloudFront

1.  您计划使用 AWS 服务托管一个仍在开发中的应用程序，并且您需要决定订阅哪个 AWS 支持计划。目前您不需要生产级别的支持，任何系统问题您都能接受 12 小时的响应时间。您应该选择哪个最具成本效益的支持计划？

    1.  基本支持计划

    1.  开发者支持计划

    1.  商业支持计划

    1.  企业支持计划

1.  以下哪些被视为 AWS 上的全球服务？（选择两个）

    1.  AWS IAM

    1.  Amazon Route53

    1.  Amazon EC2

    1.  Amazon EFS

    1.  Amazon RDS

1.  以下哪项陈述最 closely 与讨论能够*在几分钟内实现全球化*的云计算优势相关？

    1.  能够将资本支出转化为可变支出，从而避免巨大的资本支出（CAPEX）。

    1.  能够利用 Auto Scaling 等工具按需即时提供资源。

    1.  能够通过少数几次点击在多个区域部署应用程序。

    1.  能够专注于应用程序的实验和开发，而不是基础设施构建、管理和维护。

1.  您可以配置哪个 AWS 服务来在您的总支出超过预定的月度费用时向电子邮件地址发送警报？

    1.  在 Amazon CloudWatch 中设置计费警报

    1.  在 Amazon CloudTrail 中设置计费警报

    1.  在 Amazon Config 中设置计费警报

    1.  在 Amazon Trusted Advisor 中设置计费警报

1.  以下哪种资源类型与其启动所在的可用区相关联？

    1.  **弹性块存储**（**EBS**）

    1.  **弹性文件存储**（**EFS**）

    1.  Amazon Route53 托管区域

    1.  Amazon DynamoDB

1.  为了增强 AWS 账户的安全性，您需要确保所有 IAM 用户使用包含至少一个大写字母、一个数字、一个符号以及最少 9 个字符的复杂密码。您可以使用哪个 AWS IAM 功能来配置这些要求？

    1.  密码策略

    1.  权限边界

    1.  服务控制策略（SCPs）

    1.  资源策略

1.  作为推荐的最佳实践，你可以为根用户和 IAM 用户实施什么额外的身份验证安全措施？

    1.  实施 MFA（多因素认证）。

    1.  实施 LastPass。

    1.  实施 AWS WAF。

    1.  实施 AWS Shield。

1.  为需要访问你的 AWS 账户的多个共享相同工作职能的 IAM 用户分配权限的最简便方法是什么？

    1.  创建一个客户管理的 IAM 策略，并将相同的策略附加到所有共享相同工作职能的 IAM 用户。

    1.  创建一个 IAM 组，将共享相同工作职能的 IAM 用户添加到该组，并对该组应用具有必要权限的 IAM 策略。

    1.  创建一个 SCP（服务控制策略）来限制共享相同工作职能的用户执行特定权限。

    1.  创建一个具有必要权限的 IAM 角色，并将该角色分配给所有共享相同工作职能的 IAM 用户。

1.  你已将应用程序的开发外包给第三方提供商。此提供商将需要临时访问你的 AWS 账户以设置必要的基础设施并部署应用程序。你应该为该提供商配置哪种身份以便他们能够访问？

    1.  IAM 用户

    1.  IAM 组

    1.  IAM 角色

    1.  根用户

1.  AWS 上哪个工具可以用来估算你的月度成本？

    1.  AWS 定价计算器

    1.  AWS TCO 计算器

    1.  AWS 免费套餐计算器

    1.  AWS 月度计算器

1.  你需要根据业务单元和部门区分在 AWS 账户中运行的不同工作负载的成本。你如何在 AWS 生成的账单报告中识别你的资源及其所有者？

    1.  在 AWS 账单和成本管理控制台中指定特定标签作为成本分配标签。

    1.  为每个部门设置 SNS 警报。

    1.  创建账单警报。

    1.  在 AWS Organizations 中配置联合账单。

1.  哪个 AWS 工具可以让你查看你的**预留实例**（**RI**）使用情况？

    1.  AWS 成本探测器

    1.  AWS Config

    1.  AWS CloudTrail

    1.  AWS 个人健康仪表盘

1.  对于需要通过**命令行界面**（**CLI**）访问你的 AWS 账户的 IAM 用户，你需要配置哪一组凭证？

    1.  IAM 用户名和密码

    1.  IAM 访问密钥 ID 和密钥访问密钥

    1.  IAM MFA（多因素认证）

    1.  IAM 密钥对

1.  一个应用程序将部署在 EC2 实例上，且需要访问一个 Amazon S3 存储桶以上传创建的任何工件。哪种安全选项被认为是最佳实践，来授予在 EC2 实例上运行的应用程序上传文件到 Amazon S3 存储桶的必要权限？

    1.  创建一个带有一组访问密钥的 IAM 用户账户，并使用 IAM 策略分配所需的权限级别。将访问密钥硬编码到应用程序中。

    1.  创建一个带有用户名和密码的 IAM 用户账户，并使用 IAM 策略分配所需的权限级别。将用户名和密码硬编码到应用程序中。

    1.  使用 IAM 策略创建一个具有所需权限级别的 IAM 角色。将角色附加到运行在 EC2 实例上的应用程序。

    1.  创建一个 IAM 角色，并使用 IAM 策略授予所需的权限级别。将角色附加到将托管应用程序的 EC2 实例上。

1.  哪项 AWS 服务可以帮助您通过识别允许的权限集和被拒绝的权限集来排查 IAM 策略？

    1.  AWS 策略模拟器

    1.  AWS 策略管理器

    1.  AWS CloudTrail

    1.  AWS **SCPs**

1.  作为常规合规性流程的一部分，您需要定期审计 IAM 用户的列表，并查看信息，例如是否已配置密码和访问密钥，以及是否已为这些帐户启用 MFA。哪项 AWS IAM 服务使您能够生成包含上述信息的定期报告？

    1.  IAM 凭证报告

    1.  IAM MFA 报告

    1.  AWS CloudWatch

    1.  AWS Config

1.  哪种 AWS 策略可以帮助您定义 IAM 用户或 IAM 角色在您的 AWS 帐户中可以执行的权限边界？

    1.  IAM 策略

    1.  基于资源的策略

    1.  **SCPs**

    1.  权限边界

1.  哪种 AWS 策略可以帮助您控制可以为组织的 AWS 成员帐户定义的最大权限集？

    1.  IAM 策略

    1.  基于资源的策略

    1.  **SCPs**

    1.  权限边界

1.  以下哪种 Amazon S3 存储类别可以帮助您减少不经常访问的对象存储成本，并且在需要时仍然可以立即访问这些对象？

    1.  Amazon S3 标准-IA

    1.  Amazon S3 Glacier

    1.  Amazon S3 Glacier 深度归档

    1.  Amazon S3 标准

1.  您正在托管一个包含重要文件的 Amazon S3 存储桶，您希望增强安全性，使得尝试访问对象的 IAM 用户只能从公司办公网络内访问。您如何配置 S3 存储桶以满足此要求？

    1.  创建一个资源策略，授予必要的访问权限，并添加条件语句来定义和指定公司办公的 IP 地址块。

    1.  创建一个资源策略，授予必要的访问权限，并添加条件语句来指定公司 IAM 用户的帐户。

    1.  创建一个 SCP，并附加条件语句，指定公司办公的 IP 地址块。

    1.  创建一个 Amazon S3 **访问控制列表**（**ACL**），并添加条件语句来指定公司 IAM 用户的帐户。

1.  哪种 Amazon S3 存储类别在您不确定 S3 存储桶中数据的访问模式时具有成本效益？

    1.  Amazon S3 标准存储类别

    1.  Amazon S3 标准-IA 存储类别

    1.  Amazon S3 单区 IA

    1.  Amazon S3 智能分层

1.  您的初级同事不小心删除了存储在 Amazon S3 存储桶中的一些财务数据。您如何防止在 Amazon S3 中意外删除数据？

    1.  不要给初级管理员访问 Amazon S3 的权限。

    1.  在您的 S3 存储桶上设置 Amazon S3 版本控制。

    1.  设置 Amazon S3 生命周期管理。

    1.  设置 Amazon S3 终止保护。

1.  Amazon S3 的哪项功能允许你创建一个存储在不同区域中的对象副本，以满足合规性要求？

    1.  Amazon S3 **跨区域复制**（**CRR**）

    1.  Amazon S3 同区域复制

    1.  Amazon S3 版本控制

    1.  Amazon S3 多副本

1.  公司政策要求存储在 Amazon S3 中的对象必须进行静态加密。还规定，你选择的加密方式应该提供审计功能，显示何时使用了 **客户主密钥**（**CMK**）以及使用者是谁。你需要配置哪种类型的 Amazon S3 加密选项以满足要求？

    1.  **使用 Amazon S3 管理的密钥进行服务器端加密**（**SSE-S3**）

    1.  客户端加密

    1.  **使用 AWS 密钥管理服务存储的 KMS 密钥进行服务器端加密**（**SSE-KMS**）Bitlocker

1.  你需要紧急检索某些归档数据的一个小子集，以解决一个待处理的调查。数据存储在 Amazon S3 Glacier 存储类中。你可以选择哪个检索选项来紧急访问数据？

    1.  标准检索选项

    1.  加速检索选项

    1.  批量检索选项

    1.  强力检索选项

1.  你有一支远程工作团队，他们需要将研究文档和视频上传到你位于 us-east-1 区域的 Amazon S3 存储桶。你希望确保你的远程员工可以以低延迟上传研究材料。你能做些什么来减少上传过程中的速度波动，这通常是由于公共互联网架构引起的？

    1.  为你的存储桶启用 Amazon **S3 传输加速**（**S3TA**）。

    1.  在你远程员工的网络与 us-east-1 区域的 VPC 之间配置 IPSec 站点到站点 VPN 连接。

    1.  使用 Amazon Storage Gateway 服务。

    1.  设置 Amazon Express Route。

1.  你需要将大量数据从本地网络转移到 Amazon S3 平台。总数据容量大约为 400 TB。你决定选择 Amazon Snowball Edge 服务来完成转移。此过程中不需要数据计算或处理。你会推荐哪种 *版本* 的 Amazon Snowball Edge 服务？

    1.  Snowball Edge 计算优化

    1.  Snowball Edge 存储优化

    1.  Snowball Edge 数据优化

    1.  Snowball Edge 功能优化

1.  你在本地托管了几个 Microsoft Windows 应用程序，这些应用程序需要低延迟访问大量存储。你希望使用 Amazon Storage Gateway 服务来托管所有应用程序级的数据。你会推荐哪种网关选项？

    1.  Amazon S3 文件网关

    1.  Amazon FSx 文件网关

    1.  卷网关缓存模式

    1.  磁带网关

1.  按照最佳实践，你已将应用程序服务器部署在 VPC 的私有子网内。然而，这些服务器需要互联网访问以下载更新和安全补丁。哪种类型的资源可以让你在不为这些实例分配公有 IP 地址的情况下，允许 EC2 实例在私有子网中访问互联网？

    1.  Internet 网关

    1.  NAT 网关

    1.  子网

    1.  路由表

1.  关于安全组，以下哪项陈述是正确的？

    1.  安全组是有状态的，您需要为流量双向流动配置入站规则和相应的出站规则。

    1.  安全组是无状态的，您不需要为流量双向流动配置入站规则和相应的出站规则。

    1.  安全组可以用来明确拒绝来自特定 IP 地址范围的入站流量。

    1.  安全组用于限制 IAM 用户（该组的成员）可以执行的操作。

1.  AWS VPC 服务的哪项功能允许您连接多个 VPC，以便在这些 VPC 之间通过私有 IP 地址空间发送流量？

    1.  VPC 对等连接

    1.  VPC 流日志

    1.  子网

    1.  VPC 端点

1.  哪项服务可以帮助您减少建立多个 VPC 对等连接所涉及的复杂性？

    1.  AWS Transit Gateway

    1.  AWS VPC 管理器

    1.  AWS Direct Connect

    1.  IPSec VPN 隧道

1.  哪项 AWS 服务允许您使用专用的私有连接将您的本地网络连接到 AWS 账户，从而完全绕过互联网？

    1.  IPSec VPN

    1.  Express Route

    1.  Direct Connect

    1.  Snowball

1.  哪项 AWS 功能可以帮助您通过 IPSec 隧道在本地网络与 AWS VPC 之间建立连接？

    1.  Direct Connect

    1.  **虚拟专用网络**（**VPN**）

    1.  AWS Outposts

    1.  亚马逊 SNS

1.  您即将使用**应用程序负载均衡器**（**ALB**）发布您的 web 应用程序，并希望使用友好的域名而不是 ALB 的 DNS 名称来向用户宣传该站点。您可以使用哪个 AWS 服务来配置别名的名称，使得用户在浏览器中输入友好的域名时，能够直接访问 ALB 的 DNS URL？

    1.  亚马逊 Route53

    1.  亚马逊 CloudFront

    1.  亚马逊 S3

    1.  亚马逊 Direct Connect

1.  哪项 AWS 服务允许您购买并注册新的域名，可以用来在互联网上发布您的网站？

    1.  Route53

    1.  VPC

    1.  RDS

    1.  弹性 Beanstalk

1.  您开发了一个 web 应用程序，想要为其提供冗余性和弹性。亚马逊 Route53 服务的哪项功能可以帮助您设计 web 应用程序，在默认情况下，所有用户的流量会被引导到主站点，如果主站点离线，则用户会被重定向到位于不同区域的备用站点？

    1.  简单路由策略

    1.  加权路由策略

    1.  故障转移路由策略

    1.  地理位置路由策略

1.  您计划通过托管一个新的亚马逊 S3 静态网站，提供免费的食谱指南。该网站将被全球用户访问，内容包括大量有关您提供的食谱的视频和图片。哪个 AWS 服务可以帮助您在用户所在地本地缓存数字资产，从而减少用户访问网站内容时的延迟？

    1.  亚马逊 Route53

    1.  亚马逊 VPC

    1.  亚马逊 CloudFront

    1.  亚马逊 Cloud9

1.  您已创建了一个包含基础操作系统和所有必要企业设置/配置的 EC2 AMI。您在另一个区域的同事正尝试启动新的 EC2 实例，但他们无法访问您的 AMI。您需要做什么才能让您的同事使用这个新镜像？

    1.  将 AMI 复制到其他区域。

    1.  在区域之间设置 VPC 端点，允许您的同事下载 AMI。

    1.  将 AMI 复制到 S3 存储桶。

    1.  使用 Amazon Snowball 服务将 AMI 的副本发送给您的同事。

1.  哪种 EC2 实例类型是专为浮点数计算、图形处理或数据模式匹配设计的？

    1.  通用型

    1.  内存优化型

    1.  计算优化型

    1.  加速计算

1.  您需要在 EC2 实例上部署某个第三方应用程序，其许可条款是基于每个 CPU 核心/插槽计算的。您需要使用哪个 EC2 定价选项来满足这一要求？

    1.  按需实例

    1.  预留实例

    1.  Spot 实例

    1.  专用主机

1.  您目前正在运行一个正在内部开发的新应用程序的测试阶段。您的 UAT 测试人员每天需要访问测试服务器 3 小时，每周 3 次。测试阶段预计持续 5 周。在测试期间，应用程序不能出现任何中断。哪个 EC2 定价选项将是最具成本效益的？

    1.  按需实例

    1.  预留实例

    1.  Spot

    1.  专用主机

1.  哪种 EBS 卷类型是专为关键的、I/O 密集型的数据库和应用程序工作负载设计的？

    1.  `gp2`

    1.  `st1`

    1.  `sc1`

    1.  `io1`

1.  以下哪种支付选项将帮助您获得 RIs 的最大折扣？

    1.  使用**部分预付**选项支付的 1 年承诺。

    1.  使用**全额预付**选项支付的 1 年承诺。

    1.  使用**无预付**选项支付的 1 年承诺。

    1.  使用**全额预付**选项支付的 3 年承诺。

1.  哪个 AWS 服务使您能够快速部署一个**虚拟私人服务器**（**VPS**），该服务器预先配置了常见的应用程序堆栈、SSD 存储和固定的 IP 地址，并按服务器配置收取固定的月费？

    1.  亚马逊 EC2

    1.  亚马逊 Lightsail

    1.  亚马逊 ECS

    1.  亚马逊 ECR

1.  您计划在 AWS 上部署一个 Docker 应用程序。您希望无需管理 EC2 实例（如配置和扩展集群，或自己进行虚拟服务器的修补和更新）即可部署您的 Docker 镜像。哪个服务可以满足这一要求？

    1.  使用 EC2 启动类型部署的亚马逊 ECS

    1.  使用 Fargate 启动类型部署的亚马逊 ECS

    1.  使用 ECR 部署的亚马逊 ECS

    1.  使用 Lambda 函数部署的亚马逊 ECS 以管理您的服务器

1.  以下哪项服务是 AWS 无服务器产品的一部分，允许您在响应触发器或事件时运行代码？

    1.  亚马逊 ECS

    1.  AWS Lambda

    1.  亚马逊 EC2

    1.  AWS CloudFront

1.  哪个 AWS 存储选项旨在为 Windows 感知应用程序提供文件共享功能，并提供与 Microsoft Active Directory 集成的选项？

    1.  AWS FSx for Lustre

    1.  Amazon FSx for Windows 文件服务器

    1.  AWS 弹性文件系统

    1.  AWS 实例存储卷

1.  您计划在两个可用区部署 10 个 EC2 实例，这些实例将承载新的业务应用程序。所有服务器需要共享公共文件，并将运行 Amazon Linux 2 操作系统。您会推荐哪种存储架构来托管应用程序服务器的共享文件？

    1.  Amazon **弹性文件系统** (**EFS**)

    1.  Amazon FSx Lustre

    1.  Amazon S3

    1.  Amazon EBS

1.  您刚刚启动了一个 Windows EC2 实例。您如何获取 Windows 本地管理员密码？

    1.  向 Amazon 提交支持请求以获取密码。

    1.  密码会通过电子邮件自动发送给您。

    1.  密码通过短信发送到您注册的手机。

    1.  使用密钥对解密密码。

1.  哪个 AWS 服务使您能够通过扩展 AWS 基础设施来配置混合解决方案，从而使 EC2 和 EBS 服务能够托管在您的本地数据中心？

    1.  AWS RDS

    1.  AWS Direct Connect

    1.  AWS Outposts

    1.  AWS Route53

1.  您的公司提供差额投注服务。您希望对当天的交易成本进行日终分析，并进行必要的市场分析。哪个 AWS 服务可以动态配置所需的计算服务，这些服务会根据提交的作业的数量和资源需求进行扩展？

    1.  AWS Batch

    1.  AWS CloudFront

    1.  AWS Lambda

    1.  AWS 区块链

1.  哪个 AWS 服务可以帮助您使用 Kubernetes 在 AWS 上部署、管理和扩展容器化应用程序？

    1.  Amazon ECS

    1.  Amazon EKS

    1.  Amazon MFA

    1.  Amazon EC2

1.  以下哪个陈述是使用 Amazon RDS 相对于在 EC2 实例上安装数据库的优势示例？

    1.  Amazon RDS 是一个完全托管的数据库，AWS 管理底层计算和存储架构，以及修补和更新。

    1.  Amazon RDS 允许您访问操作系统，使您能够针对操作系统进行数据库的细致调优。

    1.  Amazon RDS 比在 EC2 实例上运行 Microsoft SQL Server 数据库更快。

    1.  Amazon RDS 会自动启用 Amazon RDS 中数据的加密。

1.  Amazon RDS 的哪个功能可以让您创建数据库的备用副本，并在主副本失败时提供故障转移功能？

    1.  只读副本

    1.  多可用区（Multi-AZ）

    1.  故障转移策略

    1.  快照

1.  您的公司计划将其本地的 MySQL 数据库迁移到 Amazon RDS。哪个服务可以帮助您进行迁移？

    1.  Amazon **服务器迁移服务** (**SMS**)

    1.  Amazon **数据库迁移服务** (**DMS**)

    1.  Amazon 虚拟机导入导出

    1.  Amazon Redshift 迁移工具

1.  AWS Redshift 的哪个功能允许您对存储在 Amazon S3 存储桶中的数据执行 SQL 查询？

    1.  Redshift 主节点

    1.  Redshift Spectrum

    1.  Redshift 复制

    1.  Redshift 流

1.  哪种 Amazon RDS 引擎提供高可用性，并将数据库副本分布在至少三个可用区？

    1.  MySQL

    1.  PostgreSQL

    1.  微软 SQL Server

    1.  亚马逊 Aurora

1.  哪个 AWS 托管的数据库服务允许您使用复杂结构存储数据，并且支持嵌套属性选项，例如 JSON 风格的文档？

    1.  亚马逊 RDS

    1.  亚马逊 Redshift

    1.  亚马逊 DynamoDB

    1.  亚马逊 Aurora

1.  哪个 AWS 数据库服务旨在存储不可变的敏感数据，并且其事务日志是加密可验证的？

    1.  AWS QLDB

    1.  亚马逊 Neptune

    1.  亚马逊 Aurora

    1.  亚马逊 RDS

# 模拟测试 2

1.  您目前每 4 小时手动对单实例 MySQL Amazon RDS 数据库进行快照。一些用户抱怨，当备份过程初始化时，连接到数据库的应用程序会经历短暂的中断。您可以采取什么措施解决这个问题？

    1.  配置您的 Amazon RDS 数据库，启用只读副本。

    1.  配置您的 Amazon RDS 数据库，启用 Multi-AZ。

    1.  配置 AWS 备份来执行 RDS 数据库备份。

    1.  使用 DMS 服务将 MySQL 数据库迁移到 Microsoft SQL Server。

1.  您所在的组织位于纽约的医疗行业。您计划使用内存缓存引擎来减轻 Amazon RDS 数据库的负载，以应对频繁的查询。哪个 AWS 内存缓存引擎提供 Multi-AZ 能力、数据加密，并且符合 **健康保险流通与问责法案** (**HIPAA**)？

    1.  亚马逊 Elasticache for Redis

    1.  亚马逊 Elasticache for Memcached

    1.  亚马逊 CloudFront

    1.  亚马逊 DynamoDB DAX

1.  哪个 AWS 服务提供完全托管的数据仓库功能，并且允许您使用标准 SQL 和 **商业智能** (**BI**) 工具分析大型数据集？

    1.  亚马逊 RDS

    1.  亚马逊 QLDB

    1.  亚马逊 Redshift

    1.  亚马逊 Aurora

1.  以下哪些服务会进一步增加 EC2 实例的成本？（选择两个）

    1.  详细监控

    1.  使用弹性负载均衡器

    1.  您连接的 S3 存储桶

    1.  您查询的 DynamoDB 表

    1.  设置多个密钥对

1.  您的开发团队需要部署一个弹性负载均衡器，该负载均衡器将基于 URL 路径和 HTTPS 协议将流量引导到您的 Web 服务器。您会推荐哪种弹性负载均衡器？

    1.  网络负载均衡器

    1.  ALB

    1.  网关负载均衡器

    1.  经典负载均衡器

1.  弹性负载均衡器服务的哪个功能适用于 **传输控制协议** (**TCP**)、**用户数据报协议** (**UDP**) 和 **传输层安全协议** (**TLS**) 类型的流量，并且在 **开放系统互联** (**OSI**) 模型的第 4 层运行？

    1.  网络负载均衡器

    1.  ALB

    1.  网关负载均衡器

    1.  经典负载均衡器

1.  以下关于弹性负载均衡器的哪一项是正确的？

    1.  弹性负载均衡器充当防火墙，以保护运行在 EC2 实例上的应用程序。

    1.  弹性负载均衡器使你能够通过将传入的 web 流量分配到位于多个区域的目标来实现跨多个区域的高可用性。

    1.  弹性负载均衡器使你能够通过将传入的 web 流量分配到位于多个可用区的目标来实现单一区域内的高可用性。

    1.  弹性负载均衡器使你能够通过根据资源的需求来水平扩展，配置或终止 EC2 实例。

1.  配置弹性负载均衡器的哪个组件，确保你能在指定端口上接收流量，并将该流量转发到负载均衡器后面的 EC2 实例的特定端口？

    1.  端口转发器

    1.  NAT 网关

    1.  监听器

    1.  Echo

1.  你正在构建一个多层架构，其中 web 服务器部署在公有子网，应用服务器部署在 VPC 的私有子网中。你会选择哪种类型的负载均衡器来分配流量到应用服务器？

    1.  面向互联网

    1.  内部负载均衡器

    1.  动态负载均衡器

    1.  静态负载均衡器

1.  AWS Auto Scaling 服务的哪个配置功能可以帮助你定义在你的队列中可以启动的 EC2 实例的最大数量？

    1.  自动扩展组

    1.  自动扩展启动配置

    1.  自动扩展最大队列大小

    1.  自动扩展策略

1.  哪个 AWS 服务可以帮助你仅配置满足应用需求的必要数量的 EC2 实例，从而节省通常与资源过度配置相关的成本？

    1.  弹性负载均衡器

    1.  自动扩展

    1.  成本管理器

    1.  EC2 启动器

1.  你最近推出了一个新的*免费优惠券* web 应用程序，该应用程序配置在一个自动扩展组中的一组 EC2 实例上。在黑色星期五促销前流量大幅增加，你发现自动扩展服务并没有启动更多 EC2 实例，尽管 CloudWatch 中的阈值指标已经被突破。你的同事告诉你，你可能已经超过了可以启动的 EC2 实例数量的配额或限制。哪个 AWS 服务可以快速查看并确定是否确实是这种情况？

    1.  个人健康仪表板

    1.  AWS 系统管理器

    1.  AWS Config

    1.  AWS Trusted Advisor

1.  ALB 提供哪种防火墙保护服务来帮助防御常见的 web 攻击，如跨站脚本和 SQL 注入？

    1.  AWS WAF

    1.  AWS Shield

    1.  亚马逊 Guard Duty

    1.  **网络访问控制列表**（**NACLs**）

1.  亚马逊 Auto Scaling 服务提供的哪种动态扩展策略可以帮助你根据特定指标的目标值启动或终止 EC2 实例？

    1.  目标追踪扩展策略

    1.  步骤扩展策略

    1.  简单扩展策略

    1.  可预测扩展策略

1.  你计划使用 Amazon CloudWatch 发送警报，任何时候生产 EC2 实例的 CPU 使用率超过 80% 并持续 15 分钟时都能触发该警报。你可以使用哪个 AWS 服务来发送警报通知？

    1.  亚马逊 SES

    1.  亚马逊 SNS

    1.  亚马逊 SQS

    1.  亚马逊 MQ

1.  Amazon SNS 服务的哪个功能可以让您将通知消息并行推送到多个端点？

    1.  您可以使用 SNS Fanout 场景来帮助您将通知推送到多个端点。

    1.  您可以使用 SNS FIFO 主题来帮助您将通知推送到多个端点。

    1.  您可以更改超时期限，以确保通知能够发送到多个端点。

    1.  为了将通知发送到多个端点，您需要配置 Amazon SQS 与 Amazon SNS 进行集成。

1.  哪个 AWS 服务使您能够通过将应用组件解耦成分布式系统，并促进微服务架构的设计来设计您的应用架构？

    1.  Amazon SNS

    1.  Amazon Simple Queue Service (SQS)

    1.  Amazon MQ

    1.  Amazon Redshift

1.  您计划使用 Amazon SQS 来帮助解耦您的应用组件。哪个队列类型将帮助您确保从一个组件到另一个组件的消息顺序得以保留？

    1.  配置 Amazon SQS 为标准队列。

    1.  配置 Amazon SQS 为 FIFO 队列。

    1.  配置 Amazon SQS 为 LIFO 队列。

    1.  配置 Amazon SQS 为 DLQ（死信队列）。

1.  您计划将一个应用迁移到云端。哪个消息代理服务可以让您继续使用 Apache ActiveMQ，并促进应用组件之间的通信？

    1.  Amazon SQS

    1.  Amazon MQ

    1.  Amazon SNS

    1.  Amazon SES

1.  哪个 AWS 服务可以帮助您基于事件（例如从 Amazon S3 存储桶中删除对象）触发 Lambda 函数？

    1.  AWS ECS

    1.  AWS Batch

    1.  AWS EventBridge

    1.  Amazon CloudTrail

1.  您的保险理赔解决方案的应用架构包含一个工作流过程，该过程可能需要最多 30 天才能完成，并且需要通过手动审批过程进行人工干预。您会推荐哪个 AWS 服务来构建该工作流过程？

    1.  Amazon SQS

    1.  Amazon Step Functions

    1.  AWS CloudFormation

    1.  AWS Lambda

1.  您计划配置一个 Lambda 函数，该函数将在工作日的开始和结束时自动启动和停止 EC2 实例。您如何根据指定的计划自动启动和停止 EC2 实例？

    1.  配置 Amazon SNS 以触发警报，启动 Lambda 函数。

    1.  配置 Amazon CloudTrail 以根据指定的计划触发 Lambda 函数。

    1.  配置 Amazon CloudWatch Events 并设定规则，以根据指定的计划触发 Lambda 函数。

    1.  配置 Amazon Scheduler 服务。

1.  您需要运行某些 SQL 查询来分析来自流媒体源的数据并进行分析。以下哪个服务可以让您实时分析流数据？

    1.  Amazon SQS

    1.  Amazon Kinesis Data Streams

    1.  Amazon Kinesis Analytics

    1.  Amazon Athena

1.  您需要对存储在 Amazon S3 中的每周报告运行临时测试查询。您可以使用哪个 AWS 服务使用标准 SQL 查询 Amazon S3 中的原始数据？

    1.  Amazon Athena

    1.  Amazon Kinesis

    1.  Amazon RDS

    1.  Amazon Redshift

1.  哪个 AWS 服务可以用来将大量流式数据实时加载到你的 Redshift 数据仓库解决方案中？

    1.  Amazon Kinesis 数据流

    1.  Amazon Kinesis Firehose

    1.  Amazon Kinesis 视频流

    1.  Amazon Athena

1.  哪个 AWS 服务可以用来创建和发布交互式 BI 仪表板，并通过 Amazon 提供的 API 和 SDK 嵌入到你的应用程序、网站和门户中？

    1.  Amazon Athena

    1.  Amazon QuickSight

    1.  Amazon Config

    1.  Amazon Glue

1.  哪个 AWS 服务提供无服务器的**提取、转换和加载**（**ETL**）解决方案，用于从各种来源发现和提取数据，对数据仓库和数据湖中的数据进行清洗或规范化，然后将其加载到数据库中？

    1.  AWS QuickSight

    1.  Amazon Athena

    1.  Amazon Glue

    1.  Amazon CloudTrail

1.  作为迁移到云的一部分，你需要重新托管一个使用 Apache Spark 处理大量数据的大数据项目应用程序。你可以使用 AWS 上的哪个服务来帮助进行数据转换，并执行排序、聚合、连接等 ETL 工作？

    1.  AWS QuickSight

    1.  Amazon EFS

    1.  Amazon EMR

    1.  Amazon S3

1.  你需要定期为当前正在开发的新应用程序构建测试环境。你可以使用哪个 AWS 服务自动构建测试环境的基础设施，从而减少配置所需基础设施的时间？

    1.  Amazon Elastic Beanstalk

    1.  Amazon CloudFormation

    1.  AWS OpsWorks

    1.  AWS Systems Manager

1.  哪个服务可以用来编排和配置环境，通过 Chef 和 Puppet 企业工具部署应用程序？

    1.  Amazon CloudFormation

    1.  AWS OpsWorks

    1.  Amazon Elastic Beanstalk

    1.  Amazon Cloud9

1.  哪个服务允许开发人员将代码上传到 AWS，并为该应用程序配置和管理所需的基础设施？

    1.  Amazon Elastic Beanstalk

    1.  Amazon CloudFormation

    1.  Amazon Cloud9

    1.  AWS OpsWorks

1.  Elastic Beanstalk 架构中，以下哪个环境层是为了支持后台操作而设计的？

    1.  Web 服务层

    1.  Worker 层

    1.  后端层

    1.  数据库层

1.  以下哪种格式是 CloudFormation 模板编写的？（选择两个）

    1.  YAML

    1.  XML

    1.  CSV

    1.  JSON

    1.  JAVA

1.  以下哪项是自定义 CloudWatch 指标的例子？

    1.  CPU 利用率

    1.  磁盘读取

    1.  网络字节入

    1.  内存

1.  CloudWatch 的哪个功能可以在特定阈值在指定时间段内被突破时，通过 Amazon SNS 发送通知警报？

    1.  仪表板

    1.  警报

    1.  日志

    1.  事件

1.  你计划使用 CloudWatch Logs 来监控进入 AWS 环境并专门指向 EC2 实例的网络流量。你希望记录所有通过的端口 `80` 的入站网络流量。你可以配置哪个服务来帮助你实现这一要求？

    1.  ALB 访问日志

    1.  VPC 流日志

    1.  CloudTrail 日志

    1.  Config 日志

1.  哪个 AWS 服务可以让你跟踪 AWS 账户中的用户活动和 API 使用情况，用于审计目的？

    1.  AWS Config

    1.  AWS CloudWatch

    1.  AWS CloudTrail

    1.  AWS Trusted Advisor

1.  哪项 AWS 服务可以用来查看资源如何相互关联，它们过去是如何配置的，以及查看这些资源随时间的历史变化？

    1.  AWS Trusted Advisor

    1.  AWS Systems Manager

    1.  AWS Config

    1.  AWS IAM

1.  AWS Systems Manager 服务的哪个功能可以帮助你在 EC2 实例和本地服务器上推出安全补丁？

    1.  Patch Manager

    1.  Microsoft WSUS

    1.  AWS Config

    1.  SCCM

1.  你计划部署一个三层应用架构，包含一个数据库后端。你的应用程序中硬编码了数据库连接字符串和秘密信息，如用户名和密码。公司安全政策规定这种做法不可接受，并希望你以更安全的方式管理这些秘密信息。你会推荐什么方案？

    1.  将配置信息存储在 SSM 参数存储中，并通过代码引用参数名称以动态检索连接信息。

    1.  将配置信息存储在 Amazon Redshift 中，并通过代码引用连接详细信息以动态检索连接信息。

    1.  将配置信息存储在 Amazon S3 中，并通过代码动态引用连接详细信息。

    1.  将配置信息存储在 EBS 卷上，并通过代码动态引用连接详细信息。

1.  哪项 AWS 服务可以用来管理和解决影响 AWS 托管应用程序的事件？

    1.  AWS Systems Manager 事件响应管理器

    1.  AWS Systems Manager 事件管理器

    1.  Amazon EventBridge

    1.  AWS **个人健康仪表盘** (**PHD**)

1.  哪项 AWS 服务可以用来识别那些没有按照安全最佳实践配置的资源？

    1.  AWS CloudWatch

    1.  AWS Trusted Advisor

    1.  AWS IAM

    1.  AWS CloudTrail

1.  你正在尝试查看 AWS Trusted Advisor 服务，以分析你在 AWS 上部署的各种工作负载的潜在成本节约机会。然而，你注意到成本优化类别显示为灰色，且没有关于当前配置状态的报告。是什么原因导致你无法查看成本优化报告？

    1.  你没有足够的权限访问 AWS Trusted Advisor 中的成本优化类别。

    1.  你未订阅任何商业或企业支持计划。

    1.  你已使用 IAM 账户登录，只有 root 用户可以访问定价和成本信息。

    1.  AWS 账户没有与之关联的有效借记卡/信用卡。

1.  哪个 Well-Architected Framework 支柱建议，替换失败的资源通常比试图找出故障原因更好？找出失败的原因可以稍后进行，但专注于替换失败的资源有助于你快速恢复运行。

    1.  成本优化

    1.  容错

    1.  可靠性

    1.  性能

1.  以下哪项服务可以帮助实现性能支柱中关于确保全球范围内低延迟访问单一 S3 桶中视频内容的指南？

    1.  使用 AWS CloudFront 将视频内容缓存到更接近终端用户的地方。

    1.  使用 AWS DynamoDB DAX 将视频内容缓存到更接近终端用户的地方。

    1.  使用 Amazon Elasticache 将视频内容缓存到更接近终端用户的地方。

    1.  使用 Amazon Kinesis 将视频内容缓存到更接近终端用户的地方。

1.  Well-Architected 框架中的哪个支柱指的是选择适当的定价选项，以便您能够采用按需模型来配置各种资源？

    1.  性能支柱

    1.  可靠性支柱

    1.  容错支柱

    1.  成本优化支柱

1.  关于 AWS 共享责任模型，谁负责修补 Amazon RDS 数据库实例？

    1.  AWS

    1.  客户

    1.  数据库引擎供应商

    1.  客户和 AWS 都负责

1.  哪项 AWS 服务为客户提供各种合规性报告，确认 AWS 提供的服务是否符合特定要求和监管要求？

    1.  AWS CloudTrail

    1.  AWS **可接受使用政策** (**AUP**)

    1.  AWS Artifacts

    1.  AWS 合规性程序

1.  AWS 允许客户进行漏洞扫描和渗透测试。然而，某些类型的测试是禁止的。以下哪项行为是客户禁止执行的？

    1.  通过尝试猜测您的 Amazon RDS 数据库密码进行暴力破解攻击。

    1.  在 EC2 实例上运行恶意软件检测程序。

    1.  通过您的 ALB 进行跨站脚本或 SQL 注入测试的尝试。

    1.  执行模拟 **分布式拒绝服务** (**DDoS**) 攻击。

1.  哪项 AWS 服务可以让您使用 **CMK** 加密存储在 Amazon S3 桶中的数据，并提供审计功能？

    1.  **Amazon S3 管理密钥的服务器端加密** (**SSE-S3**)

    1.  **存储在 AWS KMS 中的 CMK 的服务器端加密** (**SSE-KMS**)

    1.  **客户提供密钥的服务器端加密** (**SSE-C**)

    1.  使用 Amazon 管理密钥的客户端加密

1.  为了满足严格的合规性和监管要求，您需要使用专用的 FIPS 140-2 级别 3 验证设备加密存储在 EC2 实例上的应用数据。您可以使用哪项 AWS 服务来满足这一要求？

    1.  AWS KMS

    1.  AWS CloudHSM

    1.  AWS TPM 硬件模块

    1.  AWS 证书管理器

1.  哪项 AWS 安全解决方案提供防护，以应对 **DDoS** 攻击，并配备 24/7 的 AWS **Shield 响应团队** (**SRT**) 协助您处理此类攻击？

    1.  AWS WAF

    1.  AWS X-Ray

    1.  AWS Detective

    1.  AWS Shield Advanced

1.  哪种类型的防火墙解决方案与 Amazon CloudFront 和 ALB 集成，提供对常见 Web 攻击（如跨站脚本和 SQL 注入）的保护？

    1.  AWS WAF

    1.  AWS Shield

    1.  AWS X-Ray

    1.  AWS 防火墙管理器

1.  您计划在 Amazon S3 上存储大量数据，并希望监控您的数据访问方式，特别是突出显示任何敏感信息，如**个人身份信息**（**PII**）。哪种 AWS 服务可以帮助您满足此需求？

    1.  Amazon Macie

    1.  AWS GuardDuty

    1.  AWS Detective

    1.  AWS X-Ray

1.  您正在构建一个将对外公开的移动应用程序，并希望集成第三方身份提供商进行身份验证，如 Facebook 或 Google。哪种 AWS 服务可用于为您的 Web 和移动应用程序设置身份与访问控制解决方案？

    1.  AWS Cognito

    1.  AWS IAM

    1.  Active Directory

    1.  AWS Certificate Manager

1.  哪种 AWS 服务可以通过分析来自 CloudTrail 事件日志、Amazon VPC Flow Logs 和 DNS 日志的数据来帮助检测恶意活动？

    1.  AWS Shield

    1.  AWS Detective

    1.  AWS GuardDuty

    1.  Amazon Macie

1.  哪种 AWS 服务可以通过提取基于时间的事件（如登录、来自 Amazon VPC Flow Logs 的网络流量以及来自 GuardDuty 发现的数据）来帮助您确定安全问题的根本原因？

    1.  AWS Shield

    1.  AWS WAF

    1.  AWS Detective

    1.  Amazon Macie

1.  您计划将本地工作负载和应用程序迁移到云中。哪种 AWS 服务可以让您捕获与您的 IT 环境相关的数百万个实时数据点，并查看适当调整和正确计费工作负载的建议？

    1.  AWS 定价计算器

    1.  AWS Migration Evaluator

    1.  AWS Hybrid Calculator

    1.  AWS 成本探测器

1.  哪种 EC2 实例定价模型可以提供高达 90% 的按需价格折扣，并适用于在实例中断不会影响应用程序工作流的场景？

    1.  预留实例

    1.  Spot 实例

    1.  专用实例

    1.  专用主机

1.  哪种 Amazon S3 存储类可以让您作为 S3 存储容量的一部分托管 48 TB 或 96 TB，并提供在本地创建最多 100 个 S3 桶的选项？

    1.  标准存储类

    1.  标准单区（IA）

    1.  Glacier

    1.  Amazon S3 on Outposts

1.  您可以创建哪种类型的策略来授予匿名访问权限，以便访问存储在 S3 桶中的对象，这些对象可用于托管网站资产？

    1.  IAM 策略

    1.  IAM 权限边界

    1.  资源策略

    1.  SNS 策略

1.  哪种 AWS 服务使您能够注册新的域名以满足企业业务需求？

    1.  AWS DNS

    1.  AWS Route53

    1.  AWS VPC

    1.  Amazon Macie

1.  哪种 AWS 服务提供图像和视频分析，可以用于识别物体、人物、文本、场景和其他活动？

    1.  Amazon Rekognition

    1.  Amazon Kinesis Video Streams

    1.  Amazon Prime

    1.  Amazon Athena

1.  哪种 AWS 服务提供文本搜索和分析功能，可以在接近实时的情况下存储、分析和执行大数据量的搜索操作？

    1.  Amazon Redshift

    1.  Amazon ElastiCache

    1.  Amazon Elastisearch

    1.  Amazon Search

1.  你计划将整个本地网络迁移到云端，并且决定放弃物理桌面和工作站，转而采用完整的 VDI 解决方案。AWS 上哪个服务可以让你在云端配置虚拟桌面，并通过网页浏览器访问？

    1.  Amazon EC2

    1.  Amazon Lightsail

    1.  Amazon WorkSpaces

    1.  Amazon EKS
