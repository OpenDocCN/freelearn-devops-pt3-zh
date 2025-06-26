# 第十五章：编程云

你可能听过“*云就是别人家的电脑*”这句话。虽然有一定的道理，但也有些偏离实际。云服务提供商提供的是运行在其数据中心的虚拟机，你可以付费使用，从这个角度看，确实是在使用别人家的电脑。然而，这并没有呈现出云服务提供商的整体面貌。云服务提供商是由数百个应用托管、数据、合规性和计算基础设施服务组成的，这些服务分布在全球数百个数据中心，并通过完全可编程的 API 进行暴露。

在本章中，我们将学习如何使用 Microsoft Azure 与云 API 进行交互。我们将从了解 API 的本质开始，包括如何描述 API 以及在哪里找到有关它们的更多文档。接下来，我们将学习身份、身份验证和授权的基础知识。然后，我们将通过使用 Azure SDK for Go 的一系列示例，应用我们学到的知识，构建云基础设施并利用其他云服务。

到本章结束时，你将掌握有效使用 Microsoft Azure 的知识，并获得与其他云服务提供商合作的可转移技能。

本章将涵盖以下主题：

+   什么是云？

+   学习 Azure API 的基础知识

+   使用 Azure 资源管理器构建基础设施

+   使用已配置的 Azure 基础设施

# 技术要求

本章将需要以下工具：

+   Go

+   Azure CLI：[`docs.microsoft.com/en-us/cli/azure/install-azure-cli`](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli)

+   从 GitHub 下载的代码文件：[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/15`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/15)

# 什么是云？

亚马逊、微软和谷歌云物理计算基础设施的资本投资规模是巨大的。试想一下，建设 200 多个物理数据中心，配备多个冗余的电力和冷却系统，并且具备先进的物理安全设施，需要多少投资。这些数据中心在面临自然灾害时仍能保持韧性。即便如此，你也只是在触及冰山一角。

这些数据中心需要全球最大的互联网络之一将它们连接起来。所有这些基础设施在没有大量电力和冷却的支持下无法运作，最好是来自可持续的能源来源。例如，Azure 自 2012 年以来一直实现碳中和，并承诺到 2030 年实现碳负排放。当人们谈论超大规模云时，他们指的是这些云服务提供商的全球范围运营。

有没有想过访问这些数据中心会是怎样的体验？例如，要访问 Azure 的数据中心，你必须通过多个安全层级。你首先需要申请进入数据中心并提供有效的商业理由。如果获得批准，当你到达数据中心的外围访问点时，你会注意到周围有众多摄像头、高大的钢铁围栏以及混凝土围墙。你需要验证身份，并进入建筑物的入口。在建筑物入口，你会遇到安保人员，他们会再次通过双重身份验证（包括生物识别）来确认你的身份。通过生物识别扫描后，他们会引导你进入数据中心的特定区域。在进入数据中心的过程中，你还需要通过全身金属探测器筛查，确保你没有携带不该带出的物品。这些数据中心的安全措施非常严格。

仍然觉得这像是*别人的电脑*吗？

云服务提供商的物理基础设施令人敬畏。然而，我们应该将焦点从云服务提供商的运营规模转向云服务是如何暴露给开发者的。正如我们最初提到的，云服务提供商通过 API 暴露云的功能，开发者可以使用这些 API 来管理运行在云上的基础设施和应用程序。我们可以利用这些 API 构建应用程序，借助超大规模的云基础设施，使其具备全球规模。

# 学习 Azure API 的基础知识

现在我们知道，编程云的路径是通过 API，让我们深入了解一下这些 API。了解如何将大量的 API 组合在一起，形成一致的编程接口是非常重要的。我们还将学习在遇到挑战时，如何找到相关的代码和文档。

在这一部分，我们将讨论主要云服务如何定义 API，并为云 API 编程提供**软件开发工具包**（**SDK**）。我们将了解在哪里可以找到这些 SDK，以及如何查找 API 和 SDK 的文档。

我们还将学习关于身份、**基于角色的访问控制**（**RBAC**）以及**资源层级**的知识，特别是在微软 Azure 中。最后，我们将创建并登录一个免费的 Azure 账户，在后续章节中我们将使用它来进行云编程。

## 云 API 和 SDK 的背景知识

正如我们在上一节中讨论的那样，云服务提供商会暴露用于管理和访问数百个服务的 API，这些服务分布在众多区域内。这些 API 通常采用 **表述性状态转移** (**REST**) 或 **谷歌远程过程调用** (**gRPC**) 实现。在每个云服务提供商内部，很可能有同等数量的工程团队负责构建这些 API。为了提供一致的资源表示，至关重要的是这些 API 在整体上能为每个服务提供相似的行为。每个云服务提供商在解决这个问题时都有自己的方法。例如，在微软 Azure 中，定义 REST API 的规则由 **微软 Azure REST API 指南**（[`github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md`](https://github.com/microsoft/api-guidelines/blob/vNext/azure/Guidelines.md)）明确规定。这些规则为服务团队提供了指导。

开发者通常不会直接通过 HTTP 使用云 API，而是通过使用 SDK。这些 SDK 是一组库，提供了对特定编程语言的 API 访问。

例如，Azure ([`github.com/Azure/azure-sdk-for-go`](https://github.com/Azure/azure-sdk-for-go))、AWS (https://github.com/aws/aws-sdk-go) 和 Google ([`github.com/googleapis/google-api-go-client`](https://github.com/googleapis/google-api-go-client)) 都为它们的云服务提供了 Go SDK 以及多种其他语言的 SDK。这些 SDK 力求消除访问云 API 所需的样板代码，简化开发者编写与之交互的程序代码。除了云服务提供商发布的文档外，始终记得 GoDocs 是你的朋友。例如，Azure Blob 存储服务的 GoDocs ([`github.com/Azure/azure-kusto-go`](https://github.com/Azure/azure-kusto-go)) 提供了使用该 SDK 的有用信息。

这些 SDK 大多数是基于机器可读的 API 规范生成的。当你拥有数百个服务和多种编程语言时，依靠大量人工编写 SDK 无法有效扩展。每个云服务提供商都有自己解决这个问题的方法。

例如，微软 Azure 几乎所有的 Azure API 参考文档（[`docs.microsoft.com/en-us/rest/api/azure/`](https://docs.microsoft.com/en-us/rest/api/azure/)）和 SDK 都是使用 Azure REST API 规范库中的 OpenAPI 规范生成的（[`github.com/Azure/azure-rest-api-specs`](https://github.com/Azure/azure-rest-api-specs)）。生成文档和 SDK 的整个过程托管在 GitHub 上，并由 AutoRest 代码生成器等开源工具提供支持（[`github.com/Azure/autorest`](https://github.com/Azure/autorest)）。

趣味提示

本书的其中一位作者 David Justice 在 Azure 建立了这一过程，并首次提交了 Azure REST API 规格仓库的代码（[`github.com/Azure/azure-rest-api-specs/commit/8c42e6392618a878d5286b8735b99bbde693c0a2`](https://github.com/Azure/azure-rest-api-specs/commit/8c42e6392618a878d5286b8735b99bbde693c0a2)）。

## Microsoft Azure 身份、RBAC 和资源层次结构

为了准备与 Azure API 进行交互，我们需要了解一些基础知识——身份、RBAC 和资源层次结构。**身份**确定了与 API 交互的用户或主体。RBAC 定义了身份在 API 内可以做什么。资源层次结构描述了 Azure 云中资源之间的关系。RBAC 角色和权限描述了主体可以对给定的资源或资源层次结构做什么。例如，用户可以被分配 Azure 订阅的贡献者权限，从而能够修改该订阅中的资源。

Azure 中的身份存储在**Azure Active Directory**（**AAD**）中。这是一个企业身份与访问管理服务，提供单点登录、多因素认证和条件访问等功能。AAD 中的身份存在于一个或多个租户中，租户包含多个身份。身份可以是用户身份，表示人类，并具有交互式身份验证流程，或者可以是服务主体，表示非人类身份，如没有交互式身份验证流程的应用程序。

Azure 中资源的根源是 Azure 订阅。订阅是一个逻辑容器，包含 Azure 资源组。每个资源，如虚拟机、存储账户或虚拟网络，都位于某个资源组内。资源组是一个逻辑实体，将多个 Azure 资源关联在一起，方便你将它们作为一个整体进行管理。

身份会被授予 RBAC 角色和权限，以便与 Azure 订阅和资源进行交互。你可以将 AAD 和 Azure 看作是两个由 RBAC 权限和角色绑定在一起的独立系统。我们不会深入探讨每个 RBAC 角色或权限，但你可以在 Azure 内置角色文档中找到更多信息（[`docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles`](https://docs.microsoft.com/en-us/azure/role-based-access-control/built-in-roles)）。

现在我们对将要使用的云环境有了一些基本了解，接下来我们开始吧。

## 创建 Azure 账户并访问 API

为了运行本章其余的示例，你需要一个 Azure 账户。如果你没有 Azure 账户，你可以注册一个免费账户，并获得 200 美元的 Azure 积分（[`azure.microsoft.com/en-us/free/`](https://azure.microsoft.com/en-us/free/)）。

一旦你有了账户，使用 Azure CLI 登录：

```
$ az login
```

该命令将登录到您的 Azure 账户，并为您的主 Azure 订阅设置默认上下文。默认情况下，当您创建 Azure 账户时，您的身份将被授予订阅中的 `owner` 角色。`owner` 角色授予完全访问权限来管理所有资源，包括在 Azure RBAC 中分配角色的能力。要查看当前活动的订阅，请运行以下命令：

```
$ az account show
{
  "environmentName": "AzureCloud",
  "isDefault": true,
  "managedByTenants": [],
  "name": "mysubscription",
  "state": "Enabled",
  "tenantId": "888bf....db93",
  "user": {
      ...
  }
}
```

上述命令的输出显示了订阅的名称以及当前 Azure CLI 上下文的其他详细信息。在接下来的命令中，我们将使用 `az` CLI 直接与 Azure API 进行交互：

```
az rest --method get --uri "/subscriptions?api-version=2019-03-01"
```

上述命令将列出您的身份通过 RBAC 权限访问的订阅。请注意，作为 Azure REST API 指南的一部分，所有 Azure API 必须使用 `api-version` 查询参数。这是强制性的，确保 API 消费者始终可以依赖于请求和响应格式的稳定性。API 更新频繁，如果没有指定某个 API 的 `api-version` 查询参数，消费者可能会面临 API 的重大变化。

接下来，让我们使用 `debug` 标志运行相同的请求：

```
az rest --method get --uri "/subscriptions?api-version=2019-03-01" --debug
```

使用 Azure CLI 执行任何命令时，添加`--debug`标志将输出 HTTP 请求的详细信息，显示类似以下内容的输出：

```
 Request URL: 'https://management.azure.com/subscriptions?apiversion=2019-03-01'
 Request method: 'GET'
 Request headers:
     'User-Agent': 'python/3.10.2 (macOS-12.3.1-arm64-arm-64bit) AZURECLI/2.34.1 (HOMEBREW)'
    urllib3.connectionpool: Starting new HTTPS connection (1): management.azure.com:443
urllib3.connectionpool: https://management.azure.com:443 "GET /subscriptions?api-version=2019-03-01 HTTP/1.1" 200 6079
 Response status: 200
 Response headers:
     'Content-Type': 'application/json; charset=utf-8'
     'x-ms-ratelimit-remaining-tenant-reads': '11999'
     'x-ms-request-id': 'aebed1f6-75f9-48c2-ae0b-1dd18ae5ec46'
     'x-ms-correlation-request-id': 'aebed1f6-75f9-48c2-ae0b-
     'Date': 'Sat, 09 Apr 2022 22:52:32 GMT'
     'Content-Length': '6079'
```

该输出对于查看发送到 Azure API 的 HTTP 内容非常有用。另外，注意 URI `https://management.azure.com/...` 对应于**Azure 资源管理器**（**ARM**）。ARM 是由每个 Azure 资源的资源提供服务组成的复合服务，负责在其中变更资源。

在这一部分，我们了解了主要云平台如何定义 API 并为 API 提供 SDK。我们还特别学习了 Azure 身份、RBAC 和资源层次结构。尽管这些信息可能特定于 Azure，但所有主要云平台遵循相同的模式。一旦你了解了某个云平台如何处理**身份与访问管理**（**IAM**），它的大致方法也可以迁移到其他云平台。最后，我们登录到 Azure 账户，供后续章节使用，并学习了如何通过 Azure CLI 直接访问 Azure REST API。

在下一部分，我们将使用 Azure SDK for Go 来变更云基础设施。让我们开始用 Go 编程操作 Azure 云。

# 使用 Azure 资源管理器构建基础设施

云 API 分为两类：管理平面和数据平面。管理平面是一个控制基础设施创建、删除和变更的 API。数据平面是由配置好的基础设施暴露的 API。

例如，管理平面将用于创建 SQL 数据库。SQL 数据库资源的数据平面则是用于操作数据库内数据和结构的 SQL 协议。

管理平面由云资源 API 提供服务，数据平面由已配置服务暴露的 API 提供服务。

在本节中，我们将学习如何使用 Azure Go SDK 在 Azure 中配置基础设施。我们将学习如何创建和销毁资源组、虚拟网络、子网、公有 IP、虚拟机和数据库。本节的目标是让大家熟悉 Azure Go SDK 以及如何与 ARM 进行交互。

## Azure Go SDK

正如我们在上一节中讨论的，云 SDK 简化了指定语言与云服务提供商 API 之间的交互。对于 Azure，我们将使用 Azure Go SDK（[`github.com/Azure/azure-sdk-for-go/`](https://github.com/Azure/azure-sdk-for-go/)）来与 Azure API 进行交互。具体来说，我们将使用该 SDK 的最新版本（[`github.com/Azure/azure-sdk-for-go#management-new-releases`](https://github.com/Azure/azure-sdk-for-go#management-new-releases)），该版本已经重新设计，以遵循 Azure 为 Go 语言制定的设计指南（[`azure.github.io/azure-sdk/golang_introduction.html`](https://azure.github.io/azure-sdk/golang_introduction.html)）。有关最新的包和文档信息，请务必查看 Azure SDK 发布页面（[`azure.github.io/azure-sdk/releases/latest/mgmt/go.html`](https://azure.github.io/azure-sdk/releases/latest/mgmt/go.html)）。

本节的代码位于 GitHub 的本章代码文件夹中：[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/15`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/15)。

## 设置本地环境

要运行本节的代码，您需要设置一个 `.env` 文件。在仓库的 `./chapter/15` 目录下运行以下命令：

```
$ mkdir .ssh
$ ssh-keygen -t rsa -b 4096 -f ./.ssh/id_rsa -q -N ""
$ chmod 600 ./.ssh/id_rsa*
```

此命令将在 `./chapter/15` 中创建一个 `.ssh` 目录，在该目录下生成一个 SSH 密钥对，并确保对该密钥对设置了正确的权限。

注意

上述命令会创建一个没有密码的 SSH 密钥。我们仅使用这个密钥对作为示例。在实际使用中，您应该为密钥设置强密码。

接下来，让我们设置一个本地的 `.env` 文件，用于存储示例中使用的环境变量：

```
echo -e "AZURE_SUBSCRIPTION_ID=$(az account show --query 'id' -o tsv)\nSSH_PUBLIC_KEY_PATH=./.ssh/id_rsa.pub" >> .env
```

现在，这个命令将创建一个 `.env` 文件，文件中包含两个环境变量，`AZURE_SUBSCRIPTION_ID` 和 `SSH_PUBLIC_KEY_PATH`。我们通过 Azure CLI 的当前活动订阅来推导出 Azure 订阅 ID 的值。

现在我们已经设置好了本地环境，接下来构建一个 `cloud-init` 配置脚本，并通过公有 IP 提供 SSH 访问。

## 构建 Azure 虚拟机

我们先从运行示例开始，然后再深入研究构建基础设施的代码。要运行示例，请执行以下命令：

```
$ go run ./cmd/compute/main.go
Staring to build Azure resources...
Building an Azure Resource Group named "fragrant-violet"...
Building an Azure Network Security Group named "fragrant-violet-nsg"...
Building an Azure Virtual Network named "fragrant-violet-vnet"...
Building an Azure Virtual Machine named "fragrant-violet-vm"...
Fetching the first Network Interface named "fragrant-violet-nic-6d8bb6ea" connected to the VM...
Fetching the Public IP Address named "fragrant-violet-pip-6d8bb6ea" connected to the VM...
Connect with: `ssh -i ./.ssh/id_rsa devops@20.225.222.128`
Press enter to delete the infrastructure.
```

在运行 `go run ./cmd/compute/main.go` 后，你应该看到与前一个命令块中显示的类似的内容。从输出中可以看到，程序构建了多个基础设施组件，包括一个 Azure 资源组、网络安全组、虚拟网络和虚拟机。稍后我们将更详细地讨论这些基础设施的每个部分。

正如输出所示，你还可以使用 SSH 访问虚拟机，具体操作见输出内容。我们将使用此方法来检查虚拟机的预配置状态，以确认 `cloud-init` 配置脚本是否按预期运行。

如果你访问 Azure 门户，应该能看到以下内容：

![图 15.1 – Azure 门户虚拟机基础设施](img/B17626_15_001.jpg)

图 15.1 – Azure 门户虚拟机基础设施

在前面的截图中，你可以看到资源组以及所有已创建的基础设施。接下来，我们来看看为这些基础设施提供服务的代码。

### 使用 Go 配置 Azure 基础设施

在这些示例中，你将看到如何构建 Azure API 客户端，查询用于访问 API 的凭证，并修改基础设施。这些示例中的许多使用了简化的错误处理方式，以使代码尽可能简洁，便于说明。`panic` 不是你的朋友。请根据需要适当地包装和传递错误。

让我们从 `go run ./cmd/compute/main.go` 的入口点开始，学习如何使用 Go 来配置云基础设施：

```
func main() {
	_ = godotenv.Load()
	ctx := context.Background()
	subscriptionID := helpers.MustGetenv(
		"AZURE_SUBSCRIPTION_ID",
	)
	sshPubKeyPath := helpers.MustGetenv("SSH_PUBLIC_KEY_PATH")
	factory := mgmt.NewVirtualMachineFactory(
		subscriptionID,
		sshPubKeyPath,
	)
	fmt.Println("Staring to build Azure resources...")
	stack := factory.CreateVirtualMachineStack(
		ctx,
		"southcentralus",
	)
	admin := stack.VirtualMachine.Properties.OSProfile.AdminUsername
	ipAddress := stack.PublicIP.Properties.IPAddress
	sshIdentityPath := strings.TrimRight(sshPubKeyPath, ".pub")
	fmt.Printf(
		"Connect with: `ssh -i %s %s@%s`\n\n",
		sshIdentityPath, *admin, *ipAddress,
	)
	fmt.Println("Press enter to delete the infrastructure.")
	reader := bufio.NewReader(os.Stdin)
	_, _ = reader.ReadString('\n')
	factory.DestroyVirtualMachineStack(context.Background(), stack)
}
```

在前面的代码中，我们使用 `godotenv.Load()` 加载本地 `.env` 文件中的环境变量。在 `main` 函数中，我们创建一个新的 `VirtualMachineFactory` 来管理 Azure 基础设施的创建和删除。基础设施在 `factory.CreateVirtualMachineStack` 中创建后，我们打印出 SSH 连接详情，并提示用户确认是否删除基础设施堆栈。

接下来，让我们深入了解虚拟机工厂，看看虚拟机堆栈中包含了哪些内容：

```
type VirtualMachineFactory struct {
     subscriptionID string
     sshPubKeyPath  string
     cred           azcore.TokenCredential
     groupsClient   *armresources.ResourceGroupsClient
     vmClient       *armcompute.VirtualMachinesClient
     vnetClient     *armnetwork.VirtualNetworksClient
     subnetClient   *armnetwork.SubnetsClient
     nicClient      *armnetwork.InterfacesClient
     nsgClient      *armnetwork.SecurityGroupsClient
     pipClient      *armnetwork.PublicIPAddressesClient
}
```

这段代码定义了 `VirtualMachineFactory` 的结构，它负责创建和访问 Azure SDK API 客户端。我们使用 `NewVirtualMachineFactory` 函数来实例化这些客户端，如下所示：

```
func NewVirtualMachineFactory(subscriptionID, sshPubKeyPath string) *VirtualMachineFactory {
     cred := HandleErrWithResult(azidentity.NewDefaultAzureCredential(nil))
     return &VirtualMachineFactory{
          cred:           cred,
          subscriptionID: subscriptionID,
          sshPubKeyPath:  sshPubKeyPath,
          groupsClient:   BuildClient(subscriptionID, cred, armresources.NewResourceGroupsClient),
          vmClient:       BuildClient(subscriptionID, cred, armcompute.NewVirtualMachinesClient),
          vnetClient:     BuildClient(subscriptionID, cred, armnetwork.NewVirtualNetworksClient),
          subnetClient:   BuildClient(subscriptionID, cred, armnetwork.NewSubnetsClient),
          nsgClient:      BuildClient(subscriptionID, cred, armnetwork.NewSecurityGroupsClient),
          nicClient:      BuildClient(subscriptionID, cred, armnetwork.NewInterfacesClient),
          pipClient:      BuildClient(subscriptionID, cred, armnetwork.NewPublicIPAddressesClient),
     }
}
```

这段代码构建了一个新的默认 Azure 身份凭证。该凭证用于验证客户端对 Azure API 的身份。默认情况下，该凭证会检查多个来源以寻找可用的身份。默认凭证首先会检查环境变量，然后尝试使用 Azure 托管身份（[`docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview`](https://docs.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/overview)），最后，如果没有找到身份，则会回退到使用 Azure CLI 的用户身份。对于本示例，我们依赖 Azure CLI 身份与 Azure API 交互。这对开发非常方便，但不应在已部署的应用程序或脚本中使用。非交互式身份验证需要使用 Azure 服务主体（[`docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals`](https://docs.microsoft.com/en-us/azure/active-directory/develop/app-objects-and-service-principals)）或 Azure 托管身份。

VM 工厂使用 `subscriptionID`、凭证以及每个客户端的 `New*` 函数构建每个 Azure API 客户端。`BuildClient()` 构建每个客户端。

既然我们已经了解了如何实例化凭证和 API 客户端，接下来让我们深入了解 `CreateVirtualMachineStack` 中的基础设施创建：

```
func (vmf *VirtualMachineFactory) CreateVirtualMachineStack(ctx context.Context, location string) *VirtualMachineStack {
     stack := &VirtualMachineStack{
          Location:   location,
          name:       haiku.Haikunate(),
          sshKeyPath: HandleErrWithResult(homedir.Expand(vmf.sshPubKeyPath)),
     }
     stack.ResourceGroup = vmf.createResourceGroup(ctx, stack.name, stack.Location)
     stack.SecurityGroup = vmf.createSecurityGroup(ctx, stack.name, stack.Location)
     stack.VirtualNetwork = vmf.createVirtualNetwork(ctx, stack)
     stack.VirtualMachine = vmf.createVirtualMachine(ctx, stack)
     stack.NetworkInterface = vmf.getFirstNetworkInterface(ctx, stack)
     stack.PublicIP = vmf.getPublicIPAddress(ctx, stack)
     return stack
}
```

在前面的代码中，我们创建了一个堆栈的概念——一组相关的基础设施。我们使用给定的位置、一个人类可读的名称和 SSH 公钥路径的内容创建了一个新的堆栈。随后，我们创建了每个 Azure 资源，以便为虚拟机提供公共 SSH 访问权限。

让我们来探讨一下 `CreateVirtualMachineStack` 中的 `create` 和 `get` 函数：

```
func (vmf *VirtualMachineFactory) createResourceGroup(ctx context.Context, name, location string) armresources.ResourceGroup {
     param := armresources.ResourceGroup{
          Location: to.Ptr(location),
     }
     fmt.Printf("Building an Azure Resource Group named %q...\n", name)
     res, err := vmf.groupsClient.CreateOrUpdate(ctx, name, param, nil)
     HandleErr(err)
     return res.ResourceGroup
}
```

在前面的代码中，`createResourceGroup` 在 `groupsClient` 上调用 `CreateOrUpdate`，在指定位置创建一个 Azure 资源组。Azure 资源组是 Azure 资源的逻辑容器。我们将使用这个资源组作为其余资源的容器。

接下来，让我们深入了解网络安全组创建函数 `createSecurityGroup`：

```
func (vmf *VirtualMachineFactory) createSecurityGroup(ctx context.Context, name, location string) armnetwork.SecurityGroup {
     param := armnetwork.SecurityGroup{
          Location: to.Ptr(location),
          Name:     to.Ptr(name + "-nsg"),
          Properties: &armnetwork.SecurityGroupPropertiesFormat{
               SecurityRules: []*armnetwork.SecurityRule{
                    {
                         Name: to.Ptr("ssh"),
                         Properties: &armnetwork.SecurityRulePropertiesFormat{
                              Access:                   to.Ptr(armnetwork.SecurityRuleAccessAllow),
                              Direction:                to.Ptr(armnetwork.SecurityRuleDirectionInbound),
                              Protocol:                 to.Ptr(armnetwork.SecurityRuleProtocolAsterisk),
                              Description:              to.Ptr("allow ssh on 22"),
                              DestinationAddressPrefix: to.Ptr("*"),
                              DestinationPortRange:     to.Ptr("22"),
                              Priority:                 to.Ptr(int32(101)),
                              SourcePortRange:          to.Ptr("*"),
                              SourceAddressPrefix:      to.Ptr("*"),
                         },
                    },
               },
          },
     }
     fmt.Printf("Building an Azure Network Security Group named %q...\n", *param.Name)
     poller, err := vmf.nsgClient.BeginCreateOrUpdate(ctx, name, *param.Name, param, nil)
     HandleErr(err)
     res := HandleErrPoller(ctx, poller)
     return res.SecurityGroup
}
```

在前面的代码中，我们构建了一个 Azure 网络安全组，该安全组包含一个单独的安全规则，允许在端口 `22` 上的网络流量，从而为虚拟机启用 SSH 访问。请注意，我们调用的是 `BeginCreateOrUpdate`，而不是 `CreateOrUpdate`，后者会向 Azure API 发送 `PUT` 或 `PATCH` 请求，并启动一个长期运行的操作。

在 Azure 中，长时间运行的操作是指—在初始变更被接受后—执行直到到达终态。例如，在创建网络安全组时，API 接收初始变更，然后开始构建基础设施。基础设施构建完成后，API 会通过操作状态或配置状态表示完成。`poller`负责跟踪这个长时间运行的操作直到完成。在`HandleErrPoller`中，我们跟踪轮询直到完成，并返回资源的最终状态。

接下来，让我们通过`createVirtualNetwork`来探讨虚拟网络的创建：

```
func (vmf *VirtualMachineFactory) createVirtualNetwork(ctx context.Context, vmStack *VirtualMachineStack) armnetwork.VirtualNetwork {
     param := armnetwork.VirtualNetwork{
          Location: to.Ptr(vmStack.Location),
          Name:     to.Ptr(vmStack.name + "-vnet"),
          Properties: &armnetwork.VirtualNetworkPropertiesFormat{
               AddressSpace: &armnetwork.AddressSpace{
                    AddressPrefixes: []*string{to.Ptr("10.0.0.0/16")},
               },
               Subnets: []*armnetwork.Subnet{
                    {
                         Name: to.Ptr("subnet1"),
                         Properties: &armnetwork.SubnetPropertiesFormat{
                              AddressPrefix:        to.Ptr("10.0.0.0/24"),
                              NetworkSecurityGroup: &vmStack.SecurityGroup,
                         },
                    },
               },
          },
     }
     fmt.Printf("Building an Azure Virtual Network named %q...\n", *param.Name)
     poller, err := vmf.vnetClient.BeginCreateOrUpdate(ctx, vmStack.name, *param.Name, param, nil)
     HandleErr(err)
     res := HandleErrPoller(ctx, poller)
     return res.VirtualNetwork
}
```

在前一个代码块中，我们为虚拟机构建了一个 Azure 虚拟网络。该虚拟网络的 CIDR 设置为`10.0.0.0/16` `10.0.0.0/24`。子网引用了我们在前一个代码块中构建的网络安全组，这导致网络安全组中的规则会被强制执行在子网上。

现在我们已经为虚拟机构建好了网络，接下来让我们通过`createVirtualMachine`来构建虚拟机：

```
func (vmf *VirtualMachineFactory) createVirtualMachine(ctx context.Context, vmStack *VirtualMachineStack) armcompute.VirtualMachine {
     param := linuxVM(vmStack)
     fmt.Printf("Building an Azure Virtual Machine named %q...\n", *param.Name)
     poller, err := vmf.vmClient.BeginCreateOrUpdate(ctx, vmStack.name, *param.Name, param, nil)
     HandleErr(err)
     res := HandleErrPoller(ctx, poller)
     return res.VirtualMachine
}
```

`createVirtualMachine()`并没有太多可展示的内容。如您所见，创建资源的相同模式通过长时间运行的 API 调用应用在这段代码中。值得注意的是在`linuxVM()`中的一些细节：

```
func linuxVM(vmStack *VirtualMachineStack) armcompute.VirtualMachine {
     return armcompute.VirtualMachine{
          Location: to.Ptr(vmStack.Location),
          Name:     to.Ptr(vmStack.name + "-vm"),
          Properties: &armcompute.VirtualMachineProperties{
               HardwareProfile: &armcompute.HardwareProfile{
                    VMSize: to.Ptr(armcompute.VirtualMachineSizeTypesStandardD2SV3),
               },
            StorageProfile: &armcompute.StorageProfile{
                    ImageReference: &armcompute.ImageReference{
                         Publisher: to.Ptr("Canonical"),
                         Offer:     to.Ptr("UbuntuServer"),
                         SKU:       to.Ptr("18.04-LTS"),
                         Version:   to.Ptr("latest"),
                    },
               },
               NetworkProfile: networkProfile(vmStack),
               OSProfile:      linuxOSProfile(vmStack),
          },
     }
}
```

在`linuxVM`中，我们指定了虚拟机的位置、名称以及属性。在属性中，我们指定了希望配置的硬件类型。在这种情况下，我们配置的是 Standard D3v2（您可以在[`docs.microsoft.com/en-us/azure/virtual-machines/dv3-dsv3-series`](https://docs.microsoft.com/en-us/azure/virtual-machines/dv3-dsv3-series)）硬件**库存单位**（**SKU**）。

我们还指定了我们的`StorageProfile`，用于指定操作系统以及我们希望附加到虚拟机的数据磁盘。在这个例子中，我们指定了要运行最新版本的 Ubuntu 18.04。由于`NetworkProfile`和`OSProfile`的复杂性太高，无法在此函数中包含，所以我们将在以下代码块中分别探讨它们：

```
func networkProfile(vmStack *VirtualMachineStack) *armcompute.NetworkProfile {
     firstSubnet := vmStack.VirtualNetwork.Properties.Subnets[0]
     return &armcompute.NetworkProfile{
          NetworkAPIVersion: to.Ptr(armcompute.NetworkAPIVersionTwoThousandTwenty1101),
          NetworkInterfaceConfigurations: []*armcompute.VirtualMachineNetworkInterfaceConfiguration{
               {
                    Name: to.Ptr(vmStack.name + "-nic"),
                    Properties: &armcompute.VirtualMachineNetworkInterfaceConfigurationProperties{
                         IPConfigurations: []*armcompute.VirtualMachineNetworkInterfaceIPConfiguration{
                              {
                                   Name: to.Ptr(vmStack.name + "-nic-conf"),
                                   Properties: &armcompute.VirtualMachineNetworkInterfaceIPConfigurationProperties{
                                        Primary: to.Ptr(true),
                                        Subnet: &armcompute.SubResource{
                                             ID: firstSubnet.ID,
                                        },
                                        PublicIPAddress Configuration: &armcompute.VirtualMachinePublicIPAddress Configuration{
                                             Name: to.Ptr(vmStack.name + "-pip"),
                                             Properties: &armcompute.VirtualMachinePublicIPAddressConfiguration Properties{
                                                  PublicIPAllocationMethod: to.Ptr(armcompute.PublicIPAllocation MethodStatic),
                                                  PublicIPAddressVersion:   to.Ptr(armcompute.IPVersionsIPv4),
                                             },
                                        },
                                   },
                              },
                         },
                         Primary: to.Ptr(true),
                    },
               },
          },
     }
}
```

在`networkProfile()`中，我们创建了`NetworkProfile`，它指定虚拟机应该使用 IPv4 并通过公共 IP 进行暴露，同时该虚拟机应只有一个网络接口。网络接口应分配到我们在`createVirtualNetwork()`中创建的子网。

接下来，让我们通过以下代码块探讨`OSProfile`配置，具体通过`linuxOSProfile()`进行配置：

```
func linuxOSProfile(vmStack *VirtualMachineStack) *armcompute.OSProfile {
     sshKeyData := HandleErrWithResult(ioutil.ReadFile(vmStack.sshKeyPath))
     cloudInitContent := HandleErrWithResult(ioutil.ReadFile("./cloud-init/init.yml"))
     b64EncodedInitScript := base64.StdEncoding.EncodeToString(cloudInitContent)
     return &armcompute.OSProfile{
          AdminUsername: to.Ptr("devops"),
          ComputerName:  to.Ptr(vmStack.name),
          CustomData:    to.Ptr(b64EncodedInitScript),
          LinuxConfiguration: &armcompute.LinuxConfiguration{
               DisablePasswordAuthentication: to.Ptr(true),
               SSH: &armcompute.SSHConfiguration{
                    PublicKeys: []*armcompute.SSHPublicKey{
                         {
                              Path:    to.Ptr("/home/devops/.ssh/authorized_keys"),
                              KeyData: to.Ptr(string(sshKeyData)),
                         },
                    },
               },
          },
     }
}
```

在`linuxOSProfile`中，我们创建了一个`OSProfile`，其中包括了管理员用户名、计算机名以及 SSH 配置等细节。请注意，`CustomData`字段用于指定 Base64 编码的`cloud-init` YAML，该 YAML 文件用于执行虚拟机的初始配置。

让我们探讨一下在`cloud-init` YAML 文件中我们正在做什么：

```
#cloud-config
package_upgrade: true
packages:
  - nginx
  - golang
runcmd:
  - echo "hello world"
```

一旦虚拟机（VM）创建完成，以下的`cloud-init`指令将被执行：

1.  首先，升级 Ubuntu 机器上的软件包。

1.  接下来，`nginx`和`golang`包通过**高级包工具**（**APT**）安装。

1.  最后，`runcmd echos "hello world"`。

`cloud-init`对于引导虚拟机非常有用。如果你以前没有使用过它，我强烈建议你进一步探索它([`cloudinit.readthedocs.io/en/latest/`](https://cloudinit.readthedocs.io/en/latest/))。

我们可以通过 SSH 访问虚拟机并执行类似下面的命令来验证`cloud-init`是否执行。记住，你的 IP 地址与这里显示的不同：

```
$ ssh -i ./.ssh/id_rsa devops@20.225.222.128
devops@fragrant-violet:~$ which go
/usr/bin/go
devops@fragrant-violet:~$ which nginx
/usr/sbin/nginx
cat /var/log/cloud-init-output.log
```

如你所见，`nginx`和`go`已经安装。你还应该能在已配置的虚拟机的`/var/log/cloud-init-output.log`中看到 APT 变更和*hello world*。

你已经配置并创建了一个 Azure 虚拟机及相关基础设施！现在，让我们销毁整个基础设施堆栈。你应该能够在运行`go run ./cmd/compute/main.go`的 shell 中按下*Enter*键。

让我们看看在调用`factory.DestroyVirtualMachineStack`时发生了什么：

```
func (vmf *VirtualMachineFactory) DestroyVirtualMachineStack(ctx context.Context, vmStack *VirtualMachineStack) {
     _, err := vmf.groupsClient.BeginDelete(ctx, vmStack.name, nil)
     HandleErr(err)
}
```

在`DestroyVirtualMachineStack`中，我们简单地在组的客户端上调用`BeginDelete()`，并指定资源组名称。然而，与其他示例不同，我们并没有等待轮询器完成。我们将`DELETE HTTP`请求发送到 Azure。我们不等待基础设施完全删除，而是相信`delete`请求的接受意味着它最终会达到删除的终端状态。

我们现在已经使用 Azure SDK for Go 构建并清理了一堆基础设施。我们学会了如何创建资源组、虚拟网络、子网、公有 IP 和虚拟机，以及如何将这种模式扩展到 Azure 中的任何资源。此外，这些技能适用于每个主要云平台，不仅仅是 Azure。AWS 和 GCP 也有类似的概念和 API 访问模式。

在下一节中，我们将构建一个 Azure 存储账户，并通过上传文件然后提供受限访问来下载这些文件，了解如何使用云服务的数据平面。

# 使用已配置的 Azure 基础设施

在上一节中，我们构建了一堆计算和网络基础设施来说明如何操作云基础设施。在这一节中，我们将把已配置的基础设施与 Azure 控制平面配对，并通过已配置服务的数据平面使用这些基础设施。

在这一部分，我们将构建一个云存储基础设施。我们将使用 Azure 存储来存储文件，并通过共享访问签名([`docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview`](https://docs.microsoft.com/en-us/azure/storage/common/storage-sas-overview))为这些文件提供受限访问。我们将学习如何使用 ARM 获取账户密钥，并使用这些密钥为存储资源提供受限访问。

## 构建一个 Azure 存储账户

让我们通过运行示例开始，然后我们将深入研究如何构建基础设施和使用配置的存储账户。要执行示例，请运行以下命令：

```
$ go run ./cmd/storage/main.go
Staring to build Azure resources...
Building an Azure Resource Group named "falling-rain"...
Building an Azure Storage Account named "fallingrain"...
Fetching the Azure Storage Account shared key...
Creating a new container "jd-imgs" in the Storage Account...
Reading all files ./blobs...
Uploading file "img1.jpeg" to container jd-imgs...
Uploading file "img2.jpeg" to container jd-imgs...
Uploading file "img3.jpeg" to container jd-imgs...
Uploading file "img4.jpeg" to container jd-imgs...
Generating readonly links to blobs that expire in 2 hours...
https://fallingrain.blob.core.windows.net/jd-imgs/img1.jpeg?se=2022-04-20T21%3A50%3A25Z&sig=MrwCXziwLLQeepLZjrW93IeEkTLxJ%2BEX16rmGa2w548%3D&sp=r&sr=b&st=2022-04-20T19%3A50%3A25Z&sv=2019-12-12
...
Press enter to delete the infrastructure.
```

如你在之前的输出中看到的，示例创建了一个资源组和一个存储账户，获取了账户密钥，然后将`./blobs`中的所有图片上传到云端。最后，示例通过共享访问签名打印出每张图片的 URI。如果你点击其中一个 URI，你应该能够下载我们上传到存储账户中的图片。

当你尝试在没有查询字符串的情况下下载`img1.jpeg`时会发生什么——例如，使用`https://fallingrain.blob.core.windows.net/jd-imgs/img1.jpeg`链接？你应该会看到访问被拒绝的消息。

让我们看看如何使用 Azure 存储上传文件并限制访问权限。

### 使用 Go 配置 Azure 存储

在这个示例中，我们将配置一个 Azure 资源组和一个 Azure 存储账户。为了让代码尽可能简洁以便说明，我们使用了简化的错误处理行为。正如我在上一节所说，panic 不是你的朋友。请适当包装和抛出错误。

让我们从 Go 的入口点`run ./cmd/storage/main.go`开始，学习如何使用 Go 来配置存储账户：

```
func init() {
     _ = godotenv.Load()
}
func main() {
     subscriptionID := MustGetenv("AZURE_SUBSCRIPTION_ID")
     factory := mgmt.NewStorageFactory(subscriptionID)
     fmt.Println("Staring to build Azure resources...")
     stack := factory.CreateStorageStack(
  context.Background(),
  "southcentralus”,
)
     uploadBlobs(stack)
     printSASUris(stack)
     fmt.Println("Press enter to delete the infrastructure.")
     reader := bufio.NewReader(os.Stdin)
     _, _ = reader.ReadString('\n')
     factory.DestroyStorageStack(context.Background(), stack)
}
```

类似于上一节中的虚拟机基础设施示例，我们使用`NewStorageFactory()`创建`StorageFactory`，然后使用它来创建和销毁存储堆栈。在中间，我们调用`uploadBlobs()`上传图片文件，并调用`printSASUris()`为每个上传的文件生成并打印共享访问签名。

首先，我们来看看如何配置存储基础设施：

```
type StorageFactory struct {
     subscriptionID string
     cred           azcore.TokenCredential
     groupsClient   *armresources.ResourceGroupsClient
     storageClient  *armstorage.AccountsClient
}
func NewStorageFactory(subscriptionID string) *StorageFactory {
     cred := HandleErrWithResult(
  azidentity. NewDefaultAzureCredential(nil),
)
     return &StorageFactory{
          cred:           cred,
          subscriptionID: subscriptionID,
          groupsClient:   BuildClient(subscriptionID, cred, armresources.NewResourceGroupsClient),
          storageClient:  BuildClient(subscriptionID, cred, armstorage.NewAccountsClient),
     }
}
```

存储工厂看起来类似于上一节中的`VirtualMachineFactory`。然而，存储工厂只使用资源组和存储客户端。

接下来，让我们探索`CreateStorageStack()`，看看我们是如何创建 Azure 存储账户的：

```
func (sf *StorageFactory) CreateStorageStack(ctx context.Context, location string) *StorageStack {
     stack := &StorageStack{
          name: haiku.Haikunate(),
     }
     stack.ResourceGroup = sf.createResourceGroup(ctx, stack.name, location)
     stack.Account = sf.createStorageAccount(ctx, stack.name, location)
     stack.AccountKey = sf.getPrimaryAccountKey(ctx, stack)
     return stack
}
```

在前面的代码中，我们为堆栈创建了一个人类可读的名称，用它来命名资源组和存储账户。然后我们将已创建的资源填充到堆栈字段中。

我将不会介绍`createResourceGroup()`，因为它已在上一节中讲解过。然而，`createStorageAccount()`和`getPrimaryAccountKey()`很有意思。让我们探讨一下它们的功能：

```
// createStorageAccount creates an Azure Storage Account
func (sf *StorageFactory) createStorageAccount(ctx context.Context, name, location string) armstorage.Account {
     param := armstorage.AccountCreateParameters{
          Location: to.Ptr(location),
          Kind:     to.Ptr(armstorage.KindBlockBlobStorage),
          SKU: &armstorage.SKU{
               Name: to.Ptr(armstorage.SKUNamePremiumLRS),
               Tier: to.Ptr(armstorage.SKUTierPremium),
          },
     }
     accountName := strings.Replace(name, "-", "", -1)
     fmt.Printf("Building an Azure Storage Account named %q...\n", accountName)
     poller, err := sf.storageClient.BeginCreate(ctx, name, accountName, param, nil)
     HandleErr(err)
     res := HandleErrPoller(ctx, poller)
     return res.Account
}
```

在前面的代码中，`createStorageAccount()`创建了一个新的块 blob，具有高级性能层，并且是本地冗余的 Azure 存储帐户。块 blob ([`docs.microsoft.com/en-us/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-block-blobs`](https://docs.microsoft.com/en-us/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-block-blobs)) 优化了大数据量的上传，正如其名称所示，它被分成了任意大小的块。本地冗余存储 ([`docs.microsoft.com/en-us/azure/storage/common/storage-redundancy#locally-redundant-storage`](https://docs.microsoft.com/en-us/azure/storage/common/storage-redundancy#locally-redundant-storage)) 意味着每个块会在同一个数据中心内复制 3 次，并且在给定的一年内保证提供 99.999999999%（11 个 9！）的耐用性。最后，Azure 存储的高级层级 ([`docs.microsoft.com/en-us/azure/storage/blobs/scalability-targets-premium-block-blobs`](https://docs.microsoft.com/en-us/azure/storage/blobs/scalability-targets-premium-block-blobs)) 表示存储帐户将针对那些持续需要低延迟和高交易吞吐量的块 blob 变更的应用程序进行优化。

除了存储帐户的配置外，其他资源的配置与我们到目前为止配置的资源类似。

为了生成上传 blob 的共享访问签名，我们需要获取一个存储帐户密钥，该密钥是在存储帐户创建时配置的。让我们看看如何请求存储帐户密钥：

```
func (sf *StorageFactory) getPrimaryAccountKey(ctx context.Context, stack *StorageStack) *armstorage.AccountKey {
     fmt.Printf("Fetching the Azure Storage Account shared key...\n")
     res, err := sf.storageClient.ListKeys(ctx, stack.name, *stack.Account.Name, nil)
     HandleErr(err)
     return res.Keys[0]
}
```

在这段代码中，我们通过在存储客户端上调用`ListKeys`来获取帐户密钥，并返回第一个返回的帐户密钥。

现在我们已经配置好了存储基础设施并获取了存储帐户密钥，我们可以使用存储服务上传文件并提供对文件的受限访问。

### 使用 Azure 存储

让我们使用`uploadBlobs`函数将`./blobs`中的文件上传到我们的存储帐户：

```
func uploadBlobs(stack *mgmt.StorageStack) {
     serviceClient := stack.ServiceClient()
     containerClient, err := serviceClient.NewContainerClient("jd-imgs")
     HandleErr(err)
     fmt.Printf("Creating a new container \"jd-imgs\" in the Storage Account...\n")
     _, err = containerClient.Create(context.Background(), nil)
     HandleErr(err)
     fmt.Printf("Reading all files ./blobs...\n")
     files, err := ioutil.ReadDir("./blobs")
     HandleErr(err)
     for _, file := range files {
          fmt.Printf("Uploading file %q to container jd-imgs...\n", file.Name())
          blobClient := HandleErrWithResult(containerClient.NewBlockBlobClient(file.Name()))
          osFile := HandleErrWithResult(os.Open(path.Join("./blobs", file.Name())))
          _ = HandleErrWithResult(blobClient.UploadFile(context.Background(), osFile, azblob.UploadOption{}))
     }
}
```

在前面的代码中，我们创建了一个服务客户端来与存储服务客户端进行交互。通过`serviceClient`，我们可以定义一个名为`jd-imgs`的新存储容器。你可以把存储容器看作是一个类似于目录的实体。在指定容器后，我们调用`create`来请求存储服务创建该容器。一旦我们有了容器，我们就可以遍历`./blobs`目录中的每个图像，并使用块 blob 客户端将它们上传。

到目前为止，我们一直在使用 Azure CLI 身份作为与 Azure 服务交互的凭证。然而，当我们实例化`serviceClient`时，我们开始使用 Azure 存储帐户密钥与我们的存储帐户进行交互。让我们看看`ServiceClient()`：

```
func (ss *StorageStack) ServiceClient() *azblob.ServiceClient {
     cred := HandleErrWithResult(azblob.NewSharedKeyCredential(*ss.Account.Name, *ss.AccountKey.Value))
     blobURI := *ss.Account.Properties.PrimaryEndpoints.Blob
     client, err := azblob.NewServiceClientWithSharedKey(blobURI, cred, nil)
     HandleErr(err)
     return client
}
```

在前面的代码中，我们使用存储账户名称和账户密钥的值创建了一个新的凭证。我们构建了 `ServiceClient`，使用存储账户的 blob 终结点和新构建的共享密钥凭证。共享密钥凭证将被用于所有从服务客户端派生的客户端。

现在我们已经将文件上传为块 blob，让我们看看如何创建签名 URI 来提供受限访问：

```
func printSASUris(stack *mgmt.StorageStack) {
     serviceClient := stack.ServiceClient()
     containerClient, err := serviceClient.NewContainerClient("jd-imgs")
     HandleErr(err)
     fmt.Printf("\nGenerating readonly links to blobs that expire in 2 hours...\n")
     files := HandleErrWithResult(ioutil.ReadDir("./blobs"))
     for _, file := range files {
          blobClient := HandleErrWithResult(containerClient.NewBlockBlobClient(file.Name()))
          permissions := azblob.BlobSASPermissions{
               Read: true,
          }
          now := time.Now().UTC()
          sasQuery := HandleErrWithResult(blobClient.GetSASToken(permissions, now, now.Add(2*time.Hour)))
          fmt.Println(blobClient.URL() + "?" + sasQuery.Encode())
     }
}
```

我们在前面的代码块中构建了 `ServiceClient` 并建立了一个容器客户端。然后，我们遍历本地 `./blobs` 目录中的每个文件，并创建一个 blob 客户端。

blob 客户端有一个有用的方法，叫做 `GetSASToken`，它会根据 blob 访问权限和有效期生成共享访问令牌。在我们的案例中，我们授予的访问权限是立即生效并在 2 小时后过期的读取权限。为了创建一个完整的 URI 以访问 blob，我们需要将 blob URL 和共享访问令牌生成的查询字符串组合起来。我们通过 `blobClient.URL()`、`"?"` 和 `sasQuery.Encode()` 来实现这一点。现在，任何拥有签名 URI 的人都可以访问该文件。

在最后一节中，我们构建并使用了云存储基础设施来存储文件，并通过使用共享访问签名（SAS）提供对这些文件的受限访问。我们学习了如何获取账户密钥，并使用它们来提供对存储资源的受限访问。通过这些技能，你可以结合权限和其他约束来定制访问方式。以这种方式提供受限访问是一个强大的工具。例如，你可以创建一个仅写的 URI，指向一个尚未创建的 blob，将 URI 传递给客户端，然后让他们上传文件，而无需访问存储账户中的任何其他文件。

# 总结

Azure 存储仅是你可以用来构建云端应用的数百项服务中的一种。每个云服务提供商都有类似的存储服务，这些服务的操作方式相似。本章中展示的示例特定于 Microsoft Azure，但可以轻松地模仿其他云服务。

Azure 存储示例有助于说明云管理平面和数据平面之间的区别。如果仔细观察，你会发现 **创建、读取、更新和删除**（**CRUD**）资源操作在使用 ARM 时与与 Azure 存储服务、容器和 blob 客户端的交互非常相似。云中的资源管理是统一的，而数据库、存储服务和内容分发网络的数据平面则很少统一，通常通过专门构建的 API 来暴露。

在本章中，我们学到了云不仅仅是“别人的计算机”。云是一个跨越行星规模的高安全性数据中心网络，里面充满了计算、网络和存储硬件。我们还学习了身份、认证和授权的基础知识，并结合了 Microsoft Azure 的具体实例。我们简要介绍了 Azure RBAC 及其与 AAD 身份的关系。最后，我们学习了如何使用 Microsoft Azure 配置和使用云资源。

你应该能够将你在这里学到的知识应用到云服务的配置和使用中，以实现你的目标。这些技能主要集中在 Microsoft Azure，但在这里学到的技能很容易转移到 AWS 或 Google 云平台。

在下一章中，我们将探讨当软件在不完美条件下运行时会发生什么。我们将学习如何为混乱设计。
