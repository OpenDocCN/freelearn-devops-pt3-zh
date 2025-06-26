# <st c="0">10</st>

# <st c="3">在 Power Platform 中启用专业开发人员扩展性</st>

<st c="52">本章重点介绍了 Power Platform 为专业开发人员提供的能力，以扩展在 Power Platform 上构建的业务解决方案的体验。</st> <st c="227">我们将探讨 Power Platform 如何确保这些扩展选项与 ALM/DevOps 流程的顺利集成，以支持这些组件的软件开发生命周期。</st> <st c="416">这些组件。</st>

<st c="433">我们将通过了解连接器开始本章内容，并继续研究自定义连接器。</st> <st c="531">在上一章中，我们展示了专业开发人员如何从 Visual Studio 构建 Web API 并直接部署到 Power Platform；在本章中，我们将探讨使用 PAC CLI 的概念，以确保我们为自定义连接器启用 ALM。</st> <st c="769">我们还将探讨自定义连接器的其他方面，如环境变量和连接引用，并解释它们在跨不同目标环境移动解决方案时的作用。</st> <st c="957">目标环境。</st>

<st c="977">之后，我们将讨论 Power Apps 中的 Canvas 组件以及使用它们的主要优势。</st> <st c="1074">接下来，我们将深入研究 Power Platform 的代码组件，特别是 Power Apps 组件框架。</st> <st c="1195">对于 Canvas 组件和代码组件，我们将解释如何将它们包含在 ALM 流程中。</st>

<st c="1308">我们将在本章结束时介绍 Power Pages，专业开发人员如何通过自定义代码扩展 Power Pages，以及如何实现 ALM 流程。</st> <st c="1454">实现。</st>

<st c="1469">我们将涵盖以下</st> <st c="1498">主要主题：</st>

+   <st c="1510">启用集成的力量 –</st> <st c="1547">连接器</st>

+   <st c="1557">Canvas 组件概览及</st> <st c="1596">组件库</st>

+   <st c="1613">了解</st> <st c="1634">代码组件</st>

+   <st c="1649">Power Pages 的 ALM</st>

# <st c="1669">技术要求</st>

<st c="1692">要跟随本章内容，您需要</st> <st c="1726">以下内容：</st>

+   **<st c="1740">Power Platform 订阅</st>**<st c="1768">：如果您已经拥有 Microsoft Entra ID 工作帐户，您可以注册 Power Platform 开发计划（</st>[<st c="1826">https://powerapps.microsoft.com/en-us/developerplan/</st>](https://powerapps.microsoft.com/en-us/developerplan/)<st c="1879">），或者您可以加入 Microsoft 365 开发者</st> <st c="1981">计划（</st>[<st c="1990">https://developer.microsoft.com/en-us/microsoft-365/dev-program</st>](https://developer.microsoft.com/en-us/microsoft-365/dev-program)<st c="2054">）。</st>

+   **<st c="2057">Visual Studio Code</st>**<st c="2076">：我们建议使用 Visual Studio Code 或带有 Power Platform Tools 扩展的 Visual Studio，或者你选择的 IDE。</st> <st c="2202">可以在此找到 Visual Studio Code：</st> [<st c="2240">https://code.visualstudio.com/</st>](https://code.visualstudio.com/)<st c="2270">。Visual Studio 也可以作为免费的 Community 版获取：</st> [<st c="2333">https://visualstudio.microsoft.com/vs/community/</st>](https://visualstudio.microsoft.com/vs/community/)<st c="2381">。</st>

+   **<st c="2382">Power Platform CLI</st>**<st c="2401">：我们将在命令行或终端中使用 PAC CLI。</st> <st c="2470">安装指南可以在此找到</st> <st c="2505">：</st> [<st c="2511">https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction?tabs=windows</st>](https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction?tabs=windows)<st c="2599">。</st>

+   **<st c="2600">Azure DevOps 或 GitHub</st>**<st c="2623">：我们可以随时创建一个 Azure DevOps 服务组织</st> *<st c="2686">免费</st>* <st c="2694">（</st>[<st c="2696">https://learn.microsoft.com/en-us/azure/devops/user-guide/sign-up-invite-teammates</st>](https://learn.microsoft.com/en-us/azure/devops/user-guide/sign-up-invite-teammates)<st c="2778">）。</st> <st c="2782">我们还可以创建 GitHub 账号和公共仓库（</st>[<st c="2840">https://github.com/signup</st>](https://github.com/signup)<st c="2866">），这对于</st> *<st c="2884">公共仓库</st>* <st c="2888">也是免费的。</st>

+   **<st c="2913">Node.js</st>**<st c="2921">：要构建 PCF 代码组件，你需要 Node.js。</st> <st c="2975">推荐使用 LTS 版本</st> <st c="2994">（</st>[<st c="3007">https://nodejs.org/en</st>](https://nodejs.org/en)<st c="3029">）。</st>

+   `<st c="3072">msbuild</st>` <st c="3079">（Visual Studio 的一部分）或</st> `<st c="3107">dotnet build</st>` <st c="3119">工具（.NET SDK 的一部分：</st> [<st c="3149">https://learn.microsoft.com/en-us/dotnet/core/sdk</st>](https://learn.microsoft.com/en-us/dotnet/core/sdk)<st c="3198">）。</st>

# <st c="3201">启用集成功能——连接器</st>

<st c="3252">本节将深入探讨如何借助 ALM 在不同环境中复用连接器和自定义连接器。</st> <st c="3387">我们将了解环境变量和连接引用的概念，这两个概念对于我们计划进行扩展时至关重要。</st> <st c="3537">我们将学习如何通过解决方案将连接器封装，以便在</st> <st c="3623">不同环境中进行潜在复用。</st>

## <st c="3646">连接器</st>

<st c="3657">我们在</st> <st c="3680">第二章</st><st c="3694">中介绍了</st><st c="3703">连接器。</st> <st c="3880">到目前为止，我们讨论的连接器使我们能够从我们的应用程序、流程和聊天机器人中连接到不同的服务和数据源。</st> <st c="3998">连接器可以是认证的，连接到第一方或第三方服务，也可以是自定义的。</st> <st c="4238">认证的连接器是预构建的，不能更改，而自定义连接器则使我们可以自由创建连接到我们自己的自定义服务/API 或其他服务的连接器，即使预构建的连接器尚未</st><st c="4359">存在。</st>

<st c="4248">连接器作为 API 操作集的包装，按触发器</st> <st c="4348">或操作分类。</st>

**<st c="4359">触发器</st>** <st c="4368">是</st> <st c="4373">连接器中响应某个事件的操作，例如创建 SharePoint 列表项。</st> <st c="4483">存在两种类型的触发器：轮询触发器</st> <st c="4528">和推送触发器。</st> **<st c="4551">轮询</st>** <st c="4558">触发器</st> <st c="4568">主动检查更改。</st> <st c="4599">它们定期执行指定服务端点的调用，以查找新数据。</st> **<st c="4708">推送</st>** <st c="4715">触发器或 Webhook 触发器能够对外部事件做出反应。</st> <st c="4785">当发生某个特定事件时，服务端点通过</st> <st c="4864">回调 URL 通知触发器。</st>

**<st c="4877">操作</st>** <st c="4885">是</st> <st c="4889">帮助我们执行在 API 定义文件中指定的方法（检索、创建、更新和删除）的操作。</st> <st c="5006">操作是在连接器连接的服务中的数据上执行的，例如从</st> <st c="5139">站点获取所有 SharePoint 列表。</st>

<st c="5146">我们在</st> *<st c="5183">第九章</st>*<st c="5192">中使用了自定义连接器。</st> <st c="5289">它们使我们能够根据我们服务的 API 操作定义自己的触发器和操作。</st>

<st c="5301">Microsoft Power Platform 的开源存储库</st>

<st c="5351">Microsoft 发布了一个开源存储库，允许任何人查看现有的认证和自定义连接器，并通过向存储库提交新连接器来进行协作。</st> <st c="5539">预定义连接器和带有相应定义文件的自定义连接器的列表已发布在 Microsoft 的 GitHub 存储库中，任何人都可以参与</st> <st c="5710">贡献：</st> [<st c="5725">https://github.com/microsoft/PowerPlatformConnectors/</st>](https://github.com/microsoft/PowerPlatformConnectors/)<st c="5778">。</st>

<st c="5779">任何想要认证连接器的人都可以按照这里描述的逐步方法进行：</st> <st c="5868">[<st c="5874">https://learn.microsoft.com/en-us/connectors/custom-connectors/submit-certification</st>](https://learn.microsoft.com/en-us/connectors/custom-connectors/submit-certification)<st c="5957">。</st>

<st c="5958">在计划将应用程序部署到不同目标环境时，我们必须确保我们的连接器使用存在于目标环境中的连接。</st> <st c="6124">为了能够通过管道自动切换到目标环境中的正确连接，我们建议使用连接引用。</st> <st c="6272">现在，让我们来看看什么是连接引用以及它们如何</st> <st c="6346">使用。</st>

## <st c="6354">连接引用</st>

<st c="6376">当连接器在应用程序或流程中用于执行某个连接器操作时，</st> **<st c="6468">连接</st>** <st c="6478">将在特定环境中为连接器创建。</st> <st c="6537">连接绑定到环境，并存储用于执行</st> <st c="6625">操作的身份验证凭据。</st>

<st c="6639">由于连接不是解决方案感知的，并且不提供将我们的业务解决方案与连接解耦的选项，</st> <st c="6789">因此引入了连接引用。</st>

**<st c="6805">连接引用</st>** <st c="6827">是</st> <st c="6832">指向特定连接器连接的解决方案组件。</st> <st c="6905">使用连接引用使我们能够构建灵活的解决方案，允许我们在应用程序和流程中通过编程方式更改连接信息。</st> <st c="7067">这简化了使用 DevOps 方法将解决方案部署到不同目标环境时的工作，因为它允许我们连接到与</st> <st c="7244">目标环境相关的资源。</st>

<st c="7263">让我们探索另一种参数化解决方案配置的方法：</st> <st c="7336">环境变量。</st>

## <st c="7358">环境变量</st>

<st c="7380">环境变量充当</st> <st c="7410">解决方案组件的配置参数，允许我们动态更改特定于目标环境的配置值。</st> <st c="7551">环境变量在传统应用程序开发中非常常见，开发人员使用它们将配置与应用程序解耦。</st> <st c="7706">这使开发人员可以在不更改应用程序代码的情况下，使应用程序适应目标环境。</st> <st c="7819">在 Power Platform 中，我们遵循相同的概念，通过仅更改环境变量来在不同目标环境中使用应用程序组件。</st> <st c="7982">这样，我们可以连接到特定于</st> <st c="8063">目标环境的数据源或一组 API。</st>

<st c="8082">环境变量通常以键值对的格式存储，值存储在应用程序源代码之外的安全位置，如</st> **<st c="8248">Azure 应用配置</st>** <st c="8271">或</st> **<st c="8275">Azure 密钥保管库</st>**<st c="8290">，以防止值被轻易访问。</st> <st c="8338">在应用程序</st> <st c="8357">运行时或 CI/CD 流水线执行过程中，这些值会从这些服务中检索并</st> <st c="8443">按需使用。</st>

<st c="8460">使用这种方法使我们能够实现 DevOps 最佳实践之一：尽可能参数化，以增强应用程序的灵活性。</st> <st c="8624">在应用程序源代码中硬编码值不仅可能导致维护问题，还可能带来潜在的</st> <st c="8737">安全漏洞。</st>

<st c="8762">总之，使用环境变量的一些</st> <st c="8796">好处包括</st> <st c="8814">以下几点：</st>

+   <st c="8836">我们可以将应用程序与配置解耦，从而轻松更改数据源和其他机密的配置值。</st> <st c="8983">例如，我们可以在将应用程序部署到不同的</st> <st c="9122">目标环境时，改变与数据源的连接，例如 API 密钥和服务器 URL。</st>

+   <st c="9142">我们可以在其他解决方案组件之间复用环境变量。</st> <st c="9212">例如，我们可以创建一个环境变量，在 Power Automate 和 Power Apps 中都使用，从而</st> <st c="9326">简化配置。</st>

+   <st c="9351">通过将机密与解决方案组件分离并</st> <st c="9435">将其存储在密钥保管库中，我们可以提高安全性，减少</st> <st c="9482">滥用的风险。</st>

<st c="9492">当我们将环境变量添加到解决方案时，Power Platform 会自动在两个 Dataverse 表中创建条目：</st> **<st c="9614">环境变量值</st>** <st c="9640">和</st> **<st c="9645">环境变量定义</st>**<st c="9676">。要在 Power Apps 应用程序中使用环境变量，我们应该使用查找函数来查询这两个表，并根据我们创建的环境变量名称获取值，如在</st> *<st c="9888">引入功能标志</st>* <st c="9913">练习中所示</st> *<st c="9926">第八章</st>*<st c="9935">。在 Power Automate 中，我们可以通过使用动态</st> <st c="10021">内容选择器来访问环境变量。</st>

<st c="10038">为了更好地理解如何简化此操作并遵循 DevOps 最佳实践，我们将通过</st> <st c="10183">一个示例来进一步了解这种方法。</st>

## <st c="10194">示例 - 将配置与应用程序解耦</st>

<st c="10250">在这个例子中，我们将</st> <st c="10278">利用连接引用和环境变量来将配置从我们的</st> <st c="10374">业务解决方案中解耦。</st>

<st c="10392">首先，我们需要创建一个解决方案并添加所有属于我们业务解决方案的组件，包括新的环境变量和连接引用。</st> <st c="10555">然后，我们将创建一个部署设置文件，该文件将把配置设置与实际应用程序解耦。</st> <st c="10680">这将使我们能够配置特定于目标环境的值。</st> <st c="10759">我们将使用部署设置文件将解决方案导入到目标环境中，并带有相关的</st> <st c="10865">配置值。</st>

### <st c="10886">准备解决方案</st>

<st c="10907">让我们来探索准备解决方案与</st> <st c="10940">配置设置的第一步：</st>

1.  <st c="10992">在 Power Apps（</st>[<st c="11012">https://make.powerapps.com</st>](https://make.powerapps.com)<st c="11039">）或 Power Automate（</st>[<st c="11061">https://make.powerautomate.com</st>](https://make.powerautomate.com)<st c="11092">）主页上，我们可以在左侧导航栏中找到</st> **<st c="11120">解决方案</st>** <st c="11129">选项。</st> <st c="11165">选择该选项将打开一个屏幕，显示当前环境中所有解决方案的列表。</st>

1.  <st c="11253">我们可以点击</st> `<st c="11541">pac solution init</st>` <st c="11558">和</st> `<st c="11563">pac solution</st>` `<st c="11576">import</st>` <st c="11582">命令。</st>

1.  <st c="11592">当我们进入解决方案对象资源管理器时，我们可以通过点击</st> **<st c="11688">新建</st>** <st c="11691">|</st> **<st c="11694">更多</st>** <st c="11698">|</st> **<st c="11701">连接引用</st>**<st c="11721">来添加新的连接引用。</st>

    <st c="11722">要添加一个</st> <st c="11732">现有的连接引用，我们点击</st> **<st c="11776">添加现有</st>** <st c="11788">|</st> **<st c="11791">更多</st>** <st c="11795">|</st> **<st c="11798">连接引用</st>**<st c="11818">，选择现有的连接引用，然后点击</st> **<st c="11872">下一步</st>**<st c="11876">，正如我们在</st> *<st c="11895">图 10</st>**<st c="11904">.1\.</st>* <st c="11908">中看到的。</st> <st c="11949">这将把任何现有的组件添加到</st> <st c="11953">解决方案中：</st>

![图 10.1 – 在解决方案对象资源管理器中添加组件](img/B22208_10_1.jpg)

<st c="12421">图 10.1 – 在解决方案对象资源管理器中添加组件</st>

1.  <st c="12484">每当我们创建新的连接引用时，</st> **<st c="12541">新建连接引用</st>** <st c="12565">屏幕将会打开，正如在</st> *<st c="12602">图 10</st>**<st c="12611">.2</st>*<st c="12613">中所见。</st>

    <st c="12614">在这里，我们必须提供</st> **<st c="12640">显示名称</st>**<st c="12652">，</st> **<st c="12654">名称</st>**<st c="12658">，</st> **<st c="12660">连接器</st>** <st c="12669">的值，以及</st> **<st c="12691">连接</st>** <st c="12701">值用于所选连接器。</st> <st c="12736">提供描述是可选的，但建议提供，以便更好地理解</st> <st c="12836">连接引用的用途：</st>

    +   <st c="12858">如果在环境中没有所选连接器的连接，我们需要通过选择</st> **<st c="12973">+ 新建连接</st>**<st c="12989">来创建一个新连接，这将打开一个新连接屏幕，弹出框中显示我们所选的连接器。</st> <st c="13074">我们需要提供所有必需的连接参数，并点击</st> **<st c="13146">创建</st>** <st c="13152">来创建</st> <st c="13163">一个连接。</st>

1.  <st c="13176">返回</st> <st c="13187">到之前的</st> **<st c="13204">新建连接引用</st>** <st c="13228">屏幕，我们现在可以点击</st> **<st c="13258">刷新</st>** <st c="13265">按钮，位于连接下拉列表旁边，如</st> *<st c="13334">图 10</st>**<st c="13343">.2</st>*<st c="13345">所示，这将更新连接列表并显示新创建的连接。</st> <st c="13428">我们可以选择它并点击</st> **<st c="13469">创建</st>**<st c="13475">，这将在</st> <st c="13510">我们的</st> <st c="13513">解决方案中创建一个新组件。</st>

![图 10.2 – 创建新的连接引用](img/B22208_10_2.jpg)

<st c="13745">图 10.2 – 创建新的连接引用</st>

1.  <st c="13794">要添加一个</st> <st c="13805">环境变量，我们遵循类似于</st> *<st c="13873">步骤 3</st>*<st c="13879">中提到的步骤。这次，我们选择</st> **<st c="13902">新建</st>** <st c="13905">|</st> **<st c="13908">更多</st>** <st c="13912">|</st> **<st c="13915">环境变量</st>**<st c="13935">。</st>

1.  <st c="13936">将出现一个新屏幕，我们需要提供更多关于环境变量的信息，例如</st> **<st c="14053">显示名称</st>**<st c="14065">，</st> **<st c="14067">名称</st>**<st c="14071">，</st> **<st c="14073">描述</st>**<st c="14084">，以及</st> **<st c="14090">数据类型</st>**<st c="14099">：</st>

    +   **<st c="14101">数据类型</st>** <st c="14110">指定了我们环境变量的类型。</st> <st c="14159">这可以是</st> **<st c="14173">十进制数</st>**<st c="14187">，</st> **<st c="14189">是/否</st>**<st c="14195">，</st> **<st c="14197">文本</st>**<st c="14201">，</st> **<st c="14203">数据源</st>**<st c="14214">，</st> **<st c="14216">机密</st>**<st c="14222">，等等。</st> <st c="14235">选择数据类型后，我们可以配置默认值和当前值，如在</st> *<st c="14330">图 10.3</st>*<st c="14341">中所示。如果我们正在创建一个类型为</st> **<st c="14394">数据源</st>**<st c="14405">的环境变量，我们可以选择</st> <st c="14433">连接器</st>**<st c="14443">。</st>

![图 10.3 – 添加新环境变量](img/B22208_10_3.jpg)

<st c="14836">图 10.3 – 添加新环境变量</st>

+   **<st c="14883">默认值</st>** <st c="14897">不是必填项；然而，当没有</st> **<st c="14969">当前值</st>** <st c="14982">设置时，它是会被使用的。</st> **<st c="15006">当前值</st>** <st c="15019">会覆盖</st> **<st c="15030">默认值</st>** <st c="15043">，并且在部署到不同的</st> <st c="15109">目标环境时非常有用。</st>

<st c="15129">一旦我们按需要配置好环境变量后，可以点击</st> **<st c="15203">保存</st>** <st c="15207">来存储配置，这将会将一个新组件添加到</st> <st c="15266">我们的解决方案中。</st>

<st c="15279">如果我们需要将现有的环境变量添加到我们的解决方案中，可以通过点击</st> **<st c="15371">添加现有</st>**<st c="15383">|</st> **<st c="15386">更多</st>** <st c="15390">|</st> **<st c="15393">环境变量</st>** <st c="15413">在解决方案对象资源管理器中，选择列表中的现有变量，然后点击</st> **<st c="15508">下一步</st>**<st c="15512">。接下来的页面会显示我们选择的环境变量，对于每个变量，我们可以选择</st> **<st c="15615">包含定义</st>** <st c="15633">和</st> **<st c="15638">包含当前值</st>** <st c="15659">复选框。</st> <st c="15672">我们可以审查已选择的现有</st> <st c="15708">环境变量，并点击</st> **<st c="15739">添加</st>** <st c="15742">将其添加到</st> <st c="15763">我们的解决方案中。</st>

<st c="15776">环境变量的当前限制</st>

<st c="15821">在使用环境变量时仍然存在一些限制；例如，目前无法使用 Microsoft Power Platform Build Tools 管理数据源环境变量。</st> <st c="16008">请确保查看限制并相应地使用环境变量。</st> <st c="16090">有关限制的更新信息，请参考</st> <st c="16145">这里：</st> [<st c="16151">https://learn.microsoft.com/en-us/power-apps/maker/data-platform/environmentvariables</st>](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/environmentvariables)<st c="16236">。</st>

<st c="16237">现在我们已经准备好了解决方案，可以将其部署到不同的环境中。</st> <st c="16322">手动导入解决方案时，导入过程会检查环境中是否已经存在定义的连接引用。</st> <st c="16483">如果存在，导入过程将其与该连接关联；如果不存在，它将提供自动创建并关联连接的机会。</st> <st c="16624">一旦连接配置完成，下一步将允许我们使用目标环境的实际值更新环境变量。</st>

<st c="16767">为了将设置解耦并允许我们的解决方案以更具程序化的方式导入，我们将查看如何生成并使用部署设置文件。</st> <st c="16930">和使用。</st>

### <st c="16939">构建部署设置文件</st>

<st c="16975">一个</st> **<st c="16977">部署设置文件</st>** <st c="17002">是一个</st> <st c="17008">存储关于连接引用和我们解决方案中使用的环境变量信息的 JSON 格式文件。</st> <st c="17148">同一个文件将在使用 Power Platform Build Tools 或 PAC CLI 进行解决方案导入时使用。</st> <st c="17259">我们在</st> *<st c="17287">第八章</st>*<st c="17296">中简要提到过它：</st>

1.  我们应该<st c="17298">导出我们的解决方案或克隆它。</st> <st c="17307">我们需要</st> <st c="17341">拥有一个解决方案 ZIP 文件或一个未打包的解决方案文件夹，我们可以通过它来生成设置文件。</st> <st c="17466">我们可以通过</st> <st c="17484">两种方式来实现：</st>

    +   <st c="17493">选择解决方案，点击</st> `<st c="17615">pac solution clone --name <solution-name></st>` <st c="17656">或</st> `<st c="17660">pac solution export --name <name></st>`<st c="17693">，这意味着解决方案文件夹将被本地克隆，或解决方案 ZIP 文件将被</st> <st c="17783">本地下载</st>

1.  <st c="17801">现在我们</st> <st c="17813">已经导出了解决方案包</st> <st c="17849">文件，我们准备好按照以下方法创建部署设置文件。</st>

    <st c="17937">生成部署设置文件可以使用以下 PAC</st> <st c="18014">CLI 命令：</st>

    ```
     pac solution create-settings --solution-zip <solution-file.zip> --solution-folder <unpacked-solution-folder> --settings-file <JSON-file-name>
    <st c="18445">deploymentSettings.json</st> file in the current folder.
    ```

    <st c="18496">下面是</st> `<st c="18532">deploymentSettings.json</st>` <st c="18555">文件结构的示例，位于本书 GitHub 仓库的</st> *<st c="18579">第十章</st>* <st c="18589">文件夹中。</st> <st c="18631">在该文件中，我们可以看到环境变量和连接引用的结构，并包含相应的值。</st>

    <st c="18751">文件中的每个环境变量都有一个</st> `<st c="18806">Value</st>` <st c="18811">属性。</st> <st c="18822">初始时，它是一个空字符串，我们将其更新为目标环境的值。</st> <st c="18913">对连接引用也应进行相同的操作，其中</st> `<st c="18978">ConnectionId</st>` <st c="18990">属性最初为空，需要</st> <st c="19032">提供。</st>

    <st c="19044">要更新连接引用值，我们可以使用 PAC CLI 连接到目标环境并列出所有连接。</st> <st c="19174">这将使我们看到给定连接器的连接是否已存在于目标环境中。</st> <st c="19276">当我们将 PAC CLI 指向目标环境时，可以使用</st> `<st c="19351">pac connection list</st>` <st c="19370">命令列出在选定环境中建立的所有连接。</st> <st c="19444">如果连接已存在，请将该连接的</st> `<st c="19477">Id</st>` <st c="19479">值复制到部署设置文件中的正确连接引用</st> `<st c="19575">ConnectionId</st>` <st c="19587">值下，如</st> *<st c="19607">第八章</st>* <st c="19616">所示；否则，创建一个新的连接。</st>

1.  <st c="19654">现在我们已经为解决方案准备了部署设置文件，可以继续进行解决方案导入过程，包括指定的部署设置文件。</st> <st c="19827">如果命令中未提供环境值或路径，将使用当前环境和当前文件夹结构进行</st> <st c="19966">解决方案</st> <st c="19976">导入：</st>

    ```
     pac solution import --path <path_to_zip_file> --settings-file .\deploymentSettings.json
    ```

<st c="20071">上述方法提供了一种编程方式来生成部署设置文件、编辑它并将解决方案导入目标环境。</st> <st c="20237">为了遵循 DevOps 方法，我们可以在流水线中使用额外任务来管理部署设置文件并部署到</st> <st c="20365">目标环境。</st>

### <st c="20384">部署设置文件和 DevOps 方法</st>

<st c="20437">在</st> *<st c="20441">第八章</st>*<st c="20450">中，我们看到如何在 DevOps 中使用部署设置</st><st c="20462">文件。</st> <st c="20486">为了保持自动化，我们可以在管道中添加一个额外的任务，</st><st c="20519">该任务将创建一个部署设置文件并将其存储在与我们导出解决方案相同的仓库中。</st> <st c="20704">我们可以使用 PAC CLI 来实现。</st> <st c="20738">我们必须确保在导出管道中使用 Power Platform Tool Installer 任务时，将 PAC CLI 添加到</st> `<st c="20865">PATH</st>` <st c="20869">环境变量中。</st> <st c="20892">以下示例演示了如何在</st> <st c="20942">Azure DevOps 中完成这项操作：</st>

```
 - task: PowerPlatformToolInstaller@2
  inputs:
    DefaultVersion: true
    AddToolsToPath: true
```

<st c="21042">为了生成部署设置文件，我们将使用一个</st> **<st c="21097">命令行</st>** <st c="21109">任务，使用我们现在</st> <st c="21147">已经熟悉的命令：</st>

```
 - task: CmdLine@2
  inputs:
    script: |
      echo 'Create Deployment Settings File'
      pac solution create-settings --solution-zip '$(Build.ArtifactStagingDirectory)/$(SolutionName).zip' --settings-file '$(SolutionName).json'
```

<st c="21375">我们在导出管道中使用的</st> `<st c="21380">git commit</st>` <st c="21390">命令</st> <st c="21399">将确保这个新的部署设置文件会被添加到我们的仓库中。</st> <st c="21534">如果我们在导出管道中创建一个制品，我们应该确保使用管道中的</st> **<st c="21678">复制文件</st>** <st c="21688">任务将部署设置文件添加到制品中。</st>

<st c="21710">现在我们已经在仓库中有了部署设置文件，接下来就是更新文件中的值。</st> <st c="21822">由于我们不希望在源代码中保留任何敏感信息，而这些值可能是可读的，我们建议将环境变量和连接参考的连接 ID 的值参数化。</st> <st c="22041">例如，在部署设置文件中，我们可以将环境变量的值写成如下形式：</st> `<st c="22154">„Value": „#{ENV_NAME}#"</st>`<st c="22177">，连接参考的连接 ID 值则可以写成如下形式：</st> `<st c="22253">„ConnectionId": „#{CONN_O365OUTLOOK}#</st>`<st c="22290">。类似的方法在</st> *<st c="22324">第八章</st>*<st c="22333">中有展示。</st>

<st c="22334">作为我们</st> <st c="22349">部署管道的一部分，在将解决方案导入到环境中的过程中，我们可以更新这些值以确保其正确。</st> <st c="22460">我们可以使用 DevOps 管道中的一个任务/操作，搜索具有正确前缀和后缀的令牌，在我们的案例中是</st> `<st c="22593">#{ … }#</st>`<st c="22600">，并用存储在 DevOps 工具或安全密钥库中的实际值替换这些令牌。</st> <st c="22720">例如，在 Azure DevOps 和 GitHub 中，我们可以通过一个</st> **<st c="22790">替换</st>** **<st c="22798">令牌</st>** <st c="22804">任务/操作来实现这一点。</st>

<st c="22817">一旦在部署管道中将令牌替换为实际值，下一步是使用</st> **<st c="22958">Power Platform 导入解决方案</st>** <st c="22988">任务导入解决方案，该任务是</st> <st c="23004">Power Platform 构建工具的一部分。</st> <st c="23040">这</st> <st c="23045">使我们能够在导入解决方案时引用部署设置文件。</st> <st c="23126">在这里，我们可以看到一个这样的任务在 Azure</st> <st c="23181">DevOps 管道中的示例：</st>

```
 - task: PowerPlatformImportSolution@2
  inputs:
    authenticationType: 'PowerPlatformSPN'
    PowerPlatformSPN: '<PP-SPN>'
    SolutionInputFile: '$(Build.ArtifactStagingDirectory)/$(SolutionName).zip'
    UseDeploymentSettingsFile: true
    DeploymentSettingsFile: '$(SolutionName).json'
    AsyncOperation: true
    MaxAsyncWaitTime: '60'
```

<st c="23509">现在，我们更清楚地了解了我们的解决方案如何拥有配置设置文件，使其能够在不同的环境中重用，这也将帮助我们构建其他组件，如画布组件和代码组件。</st> <st c="23756">我们将从画布组件和</st> <st c="23797">组件库开始。</st>

# <st c="23817">画布组件和组件库概述</st>

<st c="23871">在 Power Apps 中开发应用时，应用开发者使用各种控件来构建应用的基础模块。</st> <st c="23981">为了避免重复，并构建可重用的应用部件，这些部件可以在同一个应用或多个应用中使用，开发者应考虑使用画布组件。</st> <st c="24172">本节介绍了画布组件以及组件库，并帮助你理解它们之间的区别以及它们如何融入</st> <st c="24322">ALM 过程。</st>

## <st c="24334">画布组件</st>

**<st c="24352">画布组件</st>** <st c="24370">作为</st> <st c="24379">独立的模块化构建块，封装了特定的功能或用户界面元素。</st> <st c="24490">画布组件可以在 Power Apps 画布应用和基于模型的应用中重用。</st> <st c="24574">这些组件不仅在创建更大、更复杂的企业级应用时起着至关重要的作用，而且在多人协作开发环境中也非常重要，因为开发人员可以分配任务，并在构建业务解决方案时专注于每个单独的组件。</st> <st c="24897">创建画布组件的另一个好处是，它们可以在一个地方集中更新，所有更新都会在应用中的所有实例中反映出来。</st> <st c="25058">该应用。</st>

<st c="25074">创建画布</st> <st c="25090">组件是在 Power Apps Studio 中构建画布应用程序时完成的。</st> <st c="25172">在左侧导航栏中，点击</st> **<st c="25206">树视图</st>**<st c="25215">，我们可以看到应用程序中所有的屏幕和组件。</st> <st c="25293">点击</st> **<st c="25302">组件</st>** <st c="25312">可以查看所有现有的组件，并通过点击</st> **<st c="25394">新建组件</st>**<st c="25407">来创建新的组件。这会创建一个空白的画布组件，我们可以</st> <st c="25463">添加控件：</st>

![图 10.4 – 在 Power Apps 中创建画布组件](img/B22208_10_4.jpg)

<st c="25536">图 10.4 – 在 Power Apps 中创建画布组件</st>

<st c="25591">每个组件都可以拥有自定义属性。</st> <st c="25635">自定义属性允许组件接收应用程序传递的值（称为输入属性），或者将数据或状态从组件发送到应用程序（称为输出属性）。</st> <st c="25825">无论属性类型是输入还是输出，自定义属性都可以保存任何数据类型的数据，涵盖从传统的数据类型，如文本和数字，到更符合 Power Apps 特性的类型，如屏幕、颜色和表格。</st> <st c="26051">使用自定义属性可以在组件与应用程序主机之间共享信息。</st> <st c="26149">在画布组件层级还有一个设置，</st> **<st c="26206">访问应用程序范围</st>**<st c="26222">，可以在组件的属性中找到。</st> <st c="26287">它可以打开或关闭，以允许访问应用程序内更广泛的信息</st> <st c="26362">，例如全局变量、集合和应用程序中的控件。</st> <st c="26472">这仅适用于应用程序内的画布组件，而不适用于组件库中的组件。</st> <st c="26600">我们可以关闭此设置，并通过</st> <st c="26683">自定义属性</st> <st c="26683">将信息传递给组件。</st>

## <st c="26701">组件库</st>

<st c="26721">由于画布组件只能在一个应用程序内使用，为了在环境中跨应用程序重用组件，我们可以创建组件库。</st> <st c="26895">一个</st> **<st c="26897">组件库</st>** <st c="26914">作为组件定义的存储库。</st> <st c="26921">这使得应用程序可以管理其所依赖的组件，这意味着每当组件更新可用时，应用程序开发者将被通知。</st> <st c="27151">可用更新的信息会在 Power Apps Studio 中编辑应用程序时显示，或者通过手动点击刷新按钮来检查组件</st> <st c="27324">库更新。</st>

<st c="27340">创建组件库与创建</st> <st c="27372">画布组件不同：</st>

1.  <st c="27416">可以通过导航到 Power Apps 主屏幕并从左侧导航面板中选择</st> **<st c="27508">组件库</st>** <st c="27527">来找到组件库。</st> <st c="27560">如果在左侧导航面板中看不到此选项，请点击</st> **<st c="27629">更多</st>** <st c="27633">|</st> **<st c="27636">查看全部</st>**<st c="27648">，在这里可以找到</st> **<st c="27679">应用增强</st>**<st c="27695">：</st>

![图 10.5 – 访问组件库](img/B22208_10_5.jpg)

<st c="28983">图 10.5 – 访问组件库</st>

1.  <st c="29026">一旦我们在</st> <st c="29046">组件库屏幕上，我们可以点击</st> **<st c="29090">+ 新建组件库</st>** <st c="29113">来创建一个空白的画布，用于添加</st> <st c="29150">应用程序组件。</st>

1.  <st c="29165">在这里，我们可以按照与常规画布组件相同的过程导入现有组件或创建新组件。</st> <st c="29292">在组件库中创建或导入的每个组件都允许制作者在目标环境中进行自定义。</st> **<st c="29422">允许自定义</st>** <st c="29441">是 Power Apps Studio 内部组件的属性。</st> <st c="29499">如果允许自定义，则一旦应用制作者开始在应用程序中对组件进行更改，这将打破对组件库的引用并创建组件的本地副本。</st> <st c="29708">为了保持对组件的控制并仅允许从组件库内部进行组件更改，关闭此组件设置是</st> <st c="29861">一个良好的实践。</st>

1.  <st c="29875">一旦所有组件添加到组件库中，我们需要发布这些更改。</st> <st c="29964">发布过程与任何其他画布组件或画布应用相同。</st> <st c="30049">如果组件库没有发布，则无法</st> <st c="30102">重复使用。</st>

1.  <st c="30112">发布后，我们可以在应用程序中重新使用库中的组件。</st> <st c="30203">为此，我们在 Power Apps Studio 中打开我们的画布应用，在左侧导航面板中点击</st> **<st c="30301">插入</st>**<st c="30307">，这将打开添加控件的选项</st> <st c="30351">和组件。</st>

    <st c="30366">我们点击代表目录搜索的图标，这将打开一个</st> **<st c="30437">导入组件</st>** <st c="30454">屏幕。</st> <st c="30463">在这里，我们可以找到我们的组件库以及库中的所有组件。</st> <st c="30551">我们可以选择需要的组件然后</st> <st c="30588">点击</st> **<st c="30594">导入</st>**<st c="30600">：</st>

![图 10.6 – 从组件库导入组件](img/B22208_10_6.jpg)

<st c="31035">图 10.6 – 从组件库导入组件</st>

1.  <st c="31094">组件库中的组件将出现在左侧导航面板的</st> **<st c="31184">插入</st>** <st c="31190">选项下。</st> <st c="31199">我们可以在</st> **<st c="31221">库组件</st>** <st c="31239">组中找到它们，位于</st> **<st c="31267">自定义</st>** <st c="31273">类别下方，该类别包含本地创建的画布组件，如</st> <st c="31315">下图所示：</st>

![图 10.7 – 从添加的组件库中访问组件](img/B22208_10_7.jpg)

<st c="31417">图 10.7 – 从添加的组件库中访问组件</st>

<st c="31484">现在我们已经学习了如何构建画布组件和组件库，这些组件库支持在环境中跨画布应用程序的组件可重用性，让我们来探讨如何管理组件库的</st> <st c="31716">生命周期。</st>

## <st c="31727">管理组件库的生命周期</st>

<st c="31776">正如我们之前所看到的，</st> <st c="31799">为了在多个应用程序之间重用组件，它们需要被添加到组件库中。</st> <st c="31895">一旦添加到库中，它们就可以被插入到应用程序中。</st> <st c="31965">这样，应用程序就会创建一个来自组件库的选定组件的依赖关系，并简化我们的</st> <st c="32085">解决方案管理。</st>

<st c="32105">为了将带有组件依赖关系的应用程序部署到不同的环境中，我们必须确保在部署应用程序之前，目标环境中存在该组件库。</st> <st c="32307">否则，导入解决方案的过程将无法成功。</st> <st c="32364">我们需要确保组件库要么与应用程序一起打包在解决方案中，要么在部署带有</st> <st c="32589">组件依赖关系</st> <st c="32589">的应用程序之前，将其移至目标环境的单独解决方案中。</st>

<st c="32610">我们可以通过导航到</st> **<st c="32686">解决方案</st>**<st c="32695">，选择我们的解决方案，点击</st> **<st c="32730">对象</st>**<st c="32736">，点击应用程序名称旁边的三个点，点击</st> **<st c="32811">高级</st>**<st c="32819">，然后点击</st> **<st c="32834">显示依赖</st><st c="32847">关系</st>**<st c="32852">，如</st> *<st c="32866">图 10.8</st>*<st c="32875">所示</st><st c="32877">：</st>

![图 10.8 – 检查应用程序的依赖关系](img/B22208_10_8.jpg)

<st c="33489">图 10.8 – 检查应用程序的依赖关系</st>

<st c="33536">在这里我们会找到一个标签</st> <st c="33561">叫做</st> **<st c="33569">使用</st>**<st c="33573">，它显示了哪些对象正在被我们的应用程序使用。</st> <st c="33636">选择组件库名称旁边的三个点将会引导我们到</st> **<st c="33723">默认解决方案</st>**<st c="33739">。需要注意的是，组件库会被放置在环境中的默认解决方案中。</st> <st c="33849">当我们迁移到目标环境时，只要该环境已启用</st> <st c="33948">Dataverse</st>，情况将保持不变。</st>

<st c="33966">将组件库添加到包含我们应用程序的解决方案中，可以通过进入我们的解决方案，点击</st> **<st c="34101">对象</st>**<st c="34108">，然后创建一个新的组件库或添加一个现有的组件库。</st> <st c="34181">创建新组件库的方法是点击</st> **<st c="34220">新建</st>** <st c="34223">|</st> **<st c="34226">更多</st>** <st c="34230">|</st> **<st c="34233">组件库</st>**<st c="34250">。添加现有组件库的方法是通过</st> **<st c="34309">添加现有</st>** <st c="34321">|</st> **<st c="34324">更多</st>** <st c="34328">|</st> **<st c="34331">组件库</st>**<st c="34348">。将组件库添加到解决方案后，我们可以启用/禁用允许在目标环境中进行自定义的选项。</st> <st c="34490">该设置可以在解决方案资源管理器中找到。</st> <st c="34546">我们选择组件库，点击其名称旁边的三个点，然后点击</st> **<st c="34636">高级</st>** <st c="34644">|</st> **<st c="34647">托管属性</st>**<st c="34665">。这将打开一个编辑组件库托管属性的界面。</st> <st c="34752">在这里，我们可以切换启用或禁用允许在</st> <st c="34823">目标环境中进行自定义的选项：</st>

![图 10.9 – 在目标环境中启用/禁用自定义](img/B22208_10_9.jpg)

<st c="35736">图 10.9 – 在目标环境中启用/禁用自定义</st>

<st c="35808">一旦我们将包含所有组件的组件库添加到解决方案中，就可以按照我们已经熟悉的 ALM 流程进行操作：通过导出管道从 Power Apps 导出解决方案，然后通过导入管道将其导入目标环境，并在中间执行所有其他相关步骤</st> <st c="36114">用于测试。</st>

<st c="36126">如果我们希望</st> <st c="36144">单独更新组件库，而不是与应用程序一起更新，那么拥有一个专门用于更新组件库的管道过程是有意义的。</st> <st c="36301">我们可以创建一个单独的解决方案，并添加一个或多个我们希望单独管理其应用生命周期的组件库。</st> <st c="36447">我们需要记住，组件库必须在导入引用该组件库的解决方案之前部署到目标环境。</st> <st c="36621">一旦组件更新，应用程序创建者将通过组件更新过程（自动或主动）收到更新通知。</st>

<st c="36769">现在我们已经学习了画布组件及其可重用性，接下来我们将进一步探讨在</st> <st c="36923">代码组件中可以进行哪些额外的自定义。</st>

# <st c="36939">了解代码组件</st>

<st c="36971">在专业开发者可扩展性的基础上再进一步，</st> **<st c="37025">Power Apps 组件框架</st>** <st c="37055">(</st>**<st c="37057">PCF</st>**<st c="37060">) 使得专业开发者可以创建代码组件，在内置组件的外观和体验不足时，提升用户体验。</st> <st c="37225">PCF 是一个非常庞大的话题，足以在本书中单独开设一章。</st> <st c="37302">在本节中，我们将尽量为你提供对 PCF 的初步理解，了解 PAC CLI 如何帮助我们构建代码组件，以及如何使用</st> <st c="37480">DevOps 工具执行 ALM。</st>

<st c="37493">PCF</st> <st c="37502">是一个统一的框架，允许开发人员构建可在 Power Apps 和 Power Pages 网站中跨应用重用的自定义代码组件。</st> <st c="37661">这为组织提供了一个绝佳的机会，可以构建代码组件，并在业务解决方案中加以利用。</st> <st c="37780">通过使用 PCF，我们可以开发代码组件，这些组件不仅包含关于组件外观的信息，还包括业务逻辑。</st> <st c="37939">这使得我们能够让应用程序和网站在视觉上更具吸引力，并且根据业务需求进行定制。</st> <st c="38043">PCF 是 HTML Web 资源的继任者，HTML Web 资源曾用于在 PCF 之前，在模型驱动应用中渲染自定义 UI 组件。</st> <st c="38183">与 HTML Web 资源相比，PCF 在性能方面得到了更多的优化，使其更适用于复杂的</st> <st c="38288">业务解决方案。</st>

<st c="38307">模型驱动应用中的自定义页面</st>

<st c="38341">自定义页面</st> <st c="38354">是模型驱动应用中的一种灵活而强大的页面类型，允许我们将构建画布应用的体验带入模型驱动应用。</st> <st c="38503">我们可以向自定义页面中添加画布和代码组件，这些组件可以作为主页面、中心对话框屏幕或侧边对话框视图显示在模型驱动应用中。</st> <st c="38659">这为我们构建业务应用的体验提供了极大的灵活性。</st>

<st c="38758">目前，我们知道有两种类型的 PCF</st> <st c="38795">组件：</st> **<st c="38807">标准</st>** <st c="38815">和</st> **<st c="38820">虚拟</st>**<st c="38827">。这两种组件都利用了</st> <st c="38863">HTML、CSS 和 TypeScript。</st> <st c="38890">然而，虚拟组件使用了两个平台提供的库，React 和 Fluent UI。</st> <st c="38976">它们被添加到父虚拟</st> **<st c="39013">文档对象模型</st>** <st c="39034">(</st>**<st c="39036">DOM</st>**<st c="39039">)，这意味着不需要为每个组件单独实例化一个 React 虚拟 DOM。</st> <st c="39144">尽管 React 和 Fluent UI 可以与标准 PCF 组件一起使用，但这样做需要开发者将库单独打包到组件中。</st> <st c="39304">对于虚拟组件，我们能够使用平台提供的库，这使得我们可以在使用这些代码组件时提升应用的性能。</st> <st c="39479">需要注意的是，即使我们开始时使用标准代码组件，后续也可以将其转换为</st> <st c="39586">虚拟组件。</st>

<st c="39604">根据组件的使用方式，我们可以区分两种类型的</st> <st c="39692">PCF 组件：</st>

+   **<st c="39707">字段</st>**<st c="39713">，在这里，代码组件可以绑定到</st> <st c="39765">表单中的字段</st>

+   **<st c="39771">数据集</st>**<st c="39779">，它是一个绑定到视图、数据集或画布应用中的集合的代码组件，可以与来自</st> <st c="39908">数据集</st> 的数据行一起工作

<st c="39917">使用这两种类型的 PCF 组件，开发人员可以在数据集和仪表板上构建自定义字段、列和视图，进而替换我们业务应用中的内建组件。</st>

<st c="40109">PCF 的社区资源</st>

<st c="40141">对于那些愿意学习</st> <st c="40177">更多关于 PCF 的内容，并希望查看更广泛社区使用 PCF 开发的一些示例，微软提供了一系列链接（视频、博客和代码组件库），可以用于此目的。</st> <st c="40402">欲了解更多信息，请访问</st> <st c="40427">：</st> [<st c="40431">https://learn.microsoft.com/en-us/power-apps/developer/component-framework/community-resources</st>](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/community-resources)<st c="40525">。</st>

## <st c="40526">代码组件的组成</st>

<st c="40553">无论我们决定构建哪种控制类型或组件类型，代码组件由三个主要元素组成——清单文件、组件实现和</st> <st c="40720">资源文件。</st>

### <st c="40735">清单文件</st>

<st c="40749">控制清单</st> <st c="40768">是一个用于注册和管理代码组件的 XML 文件。</st> <st c="40840">清单文件的</st> <st c="40844">名称</st> <st c="40870">是</st> `<st c="40873">ControlManifest.Input.xml</st>`<st c="40898">。</st>

<st c="40899">它包含了关于代码组件的所有信息，例如</st> <st c="40959">以下内容：</st>

+   <st c="40973">组件</st> <st c="40988">元数据</st>

+   <st c="41008">关于控制类型的信息（标准</st> <st c="41054">或虚拟）</st>

+   <st c="41065">是否将使用任何外部服务（这将需要</st> <st c="41138">高级许可）</st>

+   <st c="41156">可用的属性</st> <st c="41187">供使用</st>

+   <st c="41194">已使用的资源</st>

+   <st c="41209">其他</st> <st c="41216">元数据和</st> <st c="41229">配置信息</st>

<st c="41240">代码组件中可用的属性可以手动输入或动态设置为来自</st> <st c="41387">应用程序中其他组件的值或数据。</st>

### <st c="41403">组件实现</st>

<st c="41428">代码组件的逻辑在一个名为</st> `<st c="41482">index.ts</st>`<st c="41490">的文件中实现。在这里，我们可以放置定义代码组件行为的代码。</st> <st c="41580">这是我们可以找到控制代码组件生命周期方法的地方。</st> <st c="41659">代码组件。</st>

<st c="41674">代码组件的生命周期可以分为四个</st> <st c="41737">主要方法：</st>

+   `<st c="41750">init</st>`<st c="41755">：此方法用于初始化组件实例。</st> <st c="41816">此功能不仅配置组件，还可以注册任何事件监听器或添加其他功能，以确保组件正确运行。</st> <st c="41973">此方法需要在代码</st> <st c="42027">实现文件中实现。</st>

+   `<st c="42047">updateView</st>`<st c="42058">：当组件的属性或组件元数据中的任何值发生变化时，会调用此方法。</st> <st c="42161">它用于反映组件 UI 中的变化。</st> <st c="42218">此方法需要在代码</st> <st c="42272">实现文件中实现。</st>

+   <st c="42694">资源文件</st>

+   `<st c="42292">getOutputs</st>`<st c="42303">：此方法在组件接收新数据之前调用。</st> <st c="42371">此方法在代码</st> <st c="42407">实现文件中是可选的。</st>

### <st c="44012">前提条件</st>

<st c="42709">在清单文件中，定义代码组件的节点之一是</st> <st c="42743">`resources`</st><st c="42790">。该节点包含关于代码资源的信息，也就是我们的</st> `<st c="42862">index.ts</st>` <st c="42870">文件，其中包含代码逻辑。</st> <st c="42904">在</st> `<st c="42916">代码</st>` <st c="42920">资源旁边，我们有静态文件，定义了代码组件的视觉外观。</st> <st c="43009">这里我们可以找到一个</st> **<st c="43028">层叠样式表</st>** <st c="43050">(</st>**<st c="43052">CSS</st>**<st c="43055">) 文件</st> <st c="43062">，其中包含控制组件视觉表现的代码。</st>

<st c="43139">由于代码组件可以在不同的应用和环境中重用，因此支持本地化非常有利，因为它允许我们在有其他语言需求的应用之间共享组件。</st> <st c="43360">PCF 支持本地化。</st> <st c="43391">本地化字符串可以存储在 RESX Web 资源文件中，并在清单文件中注册。</st> <st c="43487">其他文件，如图像、图标或任何其他需要的文件，都可以添加。</st> <st c="43603">文件通常存储在一个单独的文件夹结构中，注册在清单文件中，并在</st> <st c="43715">代码逻辑中引用。</st>

<st c="43726">现在我们已经了解了代码组件的文件结构，接下来让我们创建一个简单的</st> <st c="43849">代码组件。</st>

## <st c="43864">创建代码组件</st>

<st c="43893">在这个练习中，我们将创建一个</st> <st c="43934">虚拟（React）控件类型，因为这是未来推荐的控件类型。</st>

### `<st c="42427">destroy</st>`<st c="42435">：当组件从 DOM 树中移除时，会调用此方法。</st> <st c="42517">它应被用于执行任何清理操作，并释放组件所使用的任何内存。</st> <st c="42620">此方法需要在代码</st> <st c="42674">实现文件中实现。</st>

<st c="44040">为了构建代码</st> <st c="44068">组件，我们需要安装在</st> *<st c="44132">技术</st>* *<st c="44142">要求</st>* <st c="44154">部分中提到的工具。</st>

<st c="44163">要获取 .NET 构建工具，应安装 Visual Studio 2019 或更高版本（Visual Studio 2022）。</st> <st c="44264">我们可以选择仅在安装向导中选择 .NET 构建工具作为工作负载，而不是安装完整的 Visual Studio。</st> <st c="44402">另一种选择是安装 .NET 8.0 SDK。</st> <st c="44457">目前，.NET 8.0 是最新的长期</st> <st c="44501">支持版本。</st>

### <st c="44517">初始化项目</st>

<st c="44542">首先，我们需要初始化</st> <st c="44576">项目。</st> <st c="44585">为此，我们将使用 PAC CLI 以及其特定于 PCF 的命令集。</st> <st c="44669">我们可以使用以下命令来实现这一点：</st>

```
 pac pcf init --name <COMPONENT_NAME> --namespace <COMPONENT_NAMESPACE> --template <COMPONENT_TYPE> --framework <RENDERING_FRAMEWORK> -npm
```

<st c="44855">有两个参数非常突出—</st> `<st c="44883">template</st>` <st c="44891">和</st> `<st c="44896">framework</st>`<st c="44905">。现在，</st> `<st c="44912">template</st>` <st c="44920">描述我们是否会创建一个用于字段或数据集的代码组件。</st> `<st c="45019">framework</st>` <st c="45028">定义了我们将使用的框架。</st> <st c="45071">如果使用</st> `<st c="45080">React</st>`<st c="45085">，那么它将标记为虚拟控制类型；否则，它将是标准控制类型。</st> <st c="45186">如果我们没有在命令中提供</st> `<st c="45209">--run-npm-install</st>` <st c="45226">或</st> `<st c="45230">--npm</st>` <st c="45235">开关，我们将需要单独运行</st> `<st c="45283">npm install</st>` <st c="45294">命令，以安装</st> <st c="45330">项目依赖。</st>

<st c="45351">在我们的示例中，我们使用了</st> <st c="45380">以下命令来初始化一个包含一些</st> <st c="45456">支持文件的文件夹结构的项目：</st>

```
 pac pcf init --name SimpleReactPCF --namespace SimpleReactNS --template field --framework react -npm
```

<st c="45574">以下是</st> <st c="45587">文件夹结构：</st>

![图 10.10 – PCF 初始化项目的文件夹结构](img/B22208_10_10.jpg)

<st c="45716">图 10.10 – PCF 初始化项目的文件夹结构</st>

<st c="45778">现在，我们已经初始化了项目，并且有了文件和文件夹结构，接下来可以开始实现</st> <st c="45900">代码逻辑。</st>

### <st c="45911">实现代码组件</st>

<st c="45943">正如我们在</st> <st c="45964">代码组件的文件夹结构中看到的那样，我们的清单文件也存在于</st> <st c="46046">PCF 项目中。</st>

<st c="46058">在</st> `<st c="46070">ControlManifest.Input.xml</st>` <st c="46095">文件中，我们可以看到</st> `<st c="46117">control</st>` <st c="46124">元素，其中包括关于我们的代码组件的信息，包括控制类型设置为</st> <st c="46224">virtual</st><st c="46234">：</st>

```
 <control namespace="SimpleReactNS" constructor="SimpleReactPCF" version="0.0.1" display-name-key="SimpleReactPCF" description-key="SimpleReactPCF description" <st c="46461">property</st> node, where we can have multiple properties for our code components.
			<st c="46538">We will change the property from</st> `<st c="46572">SingleLine.Text</st>` <st c="46587">to a number (</st>`<st c="46601">Whole.None</st>`<st c="46612">) by changing the</st> <st c="46631">highlighted property:</st>

```

<property name="sampleProperty" display-name-key="Property_Display_Key" description-key="Property_Desc_Key" <st c="46761">of-type="Whole.None"</st> usage="bound" required="true" />

```

			<st c="46814">During this step, extensive coding may be done to develop the PCF component.</st> <st c="46892">Making changes to the code of the component requires knowledge of TypeScript</st> <st c="46969">and React.</st>
			<st c="46979">Debugging a developed code component</st>
			<st c="47016">With a simple</st> `<st c="47031">npm start watch</st>` <st c="47046">command, we can run the project in debugging mode, which opens</st> <st c="47110">a browser with our local PCF</st> <st c="47139">test environment.</st> <st c="47157">On the right side of the test application, we can see all the configured data properties that are specified in the manifest file, as shown in</st> *<st c="47299">Figure 10</st>**<st c="47308">.11</st>*<st c="47311">. Here we can adjust the values of the properties and check whether the state of the code component reflects the</st> <st c="47424">desired behavior.</st>
			<st c="47441">Additionally, we</st> <st c="47458">can use the test environment</st> <st c="47487">to understand how our code component adjusts to different screen sizes.</st> <st c="47560">We can use the</st> **<st c="47575">Form Factor</st>** <st c="47586">settings as well as</st> **<st c="47607">Component Container Width</st>** <st c="47632">and</st> **<st c="47637">Co</st><st c="47639">mponent Container Height</st>** <st c="47664">to adjust the</st> <st c="47679">screen size:</st>
			![Figure 10.11 – The PCF test environment](img/B22208_10_11.jpg)

			<st c="47867">Figure 10.11 – The PCF test environment</st>
			<st c="47906">Packaging the code component</st>
			<st c="47935">Code components can</st> <st c="47956">be added to solutions to enable reusability across environments.</st> <st c="48021">If we already have solutions available, we can simply upload the code component to the solution using the</st> `<st c="48127">pac pcf push</st>` <st c="48139">command, as shown in the next section.</st> <st c="48179">In other cases, we can utilize a set of commands to first create a solution and then add a reference to</st> <st c="48283">that solution.</st>
			<st c="48297">We will be adding a reference for our code component project to a newly created solution project.</st> <st c="48396">The solution project (</st>`<st c="48418">cdsproj</st>`<st c="48426">) can contain multiple code component references, while the code component project can only contain a single code component.</st> <st c="48552">Adding references to multiple code components within a single solution project allows us to reference all the code components required for a particular business solution.</st> <st c="48723">They can also be packaged together with other solution objects</st> <st c="48786">if needed.</st>
			<st c="48796">First, we will initialize a solution using the</st> <st c="48844">following command:</st>

```

pac solution init

```

			<st c="48880">Then we will add a reference to the code</st> <st c="48922">component project:</st>

```

pac solution add-reference --path <LOCATION_TO_PCFPROJ_FILE>

```

			<st c="49001">This will create a reference inside the</st> `<st c="49042">.cdsproj</st>` <st c="49050">file,</st> <st c="49057">like this:</st>

```

<ItemGroup>

    <ProjectReference Include="..\SamPCF\SamPCF.pcfproj" />

</ItemGroup>

```

			<st c="49148">When we are ready to generate a ZIP file from the solution project, we can use</st> <st c="49228">the following:</st>

```

dotnet build /t:build /restore

```

			<st c="49273">This will restore all code component projects that are referenced in the solution project and build the project.</st> <st c="49387">The final outcome of the process is a ZIP package artifact that can be imported with the code components solutions.</st> <st c="49503">Such a solution can now be imported into the</st> <st c="49548">downstream environment.</st>
			<st c="49571">If we have a</st> <st c="49585">business application in another solution that is referencing our code components, we need to make sure that the solution with the code components is deployed to the environment first, before importing the business application that</st> <st c="49816">references them.</st>
			<st c="49832">Importing the solution into Power Platform</st>
			<st c="49875">Importing the solution into</st> <st c="49903">Power Platform (Dataverse) can be done manually using the Power Platform web portal or the</st> `<st c="49995">pac pcf push</st>` <st c="50007">command.</st> <st c="50017">It can also be done automatically using Power Platform Build Tools and the DevOps tool of choice.</st> <st c="50115">In the next section, we will take a look at how this can be done automatically.</st> <st c="50195">In this section, we will complete the</st> <st c="50233">process manually.</st>
			<st c="50250">First, we should make sure that we are in the right Dataverse environment.</st> <st c="50326">To select the right environment, we can proceed with</st> `<st c="50379">pac env list</st>` <st c="50391">and</st> `<st c="50396">pac env select -env <ENVIRONMENT_ID></st>`<st c="50432">. After confirming the environment, we can proceed with the</st> `<st c="50492">push</st>` <st c="50496">command.</st>
			<st c="50505">It is worth noting that the</st> `<st c="50534">pac pcf push</st>` <st c="50546">command also has a</st> `<st c="50566">-env</st>` <st c="50570">setting, which allows us to push to a selected environment if we are operating across</st> <st c="50657">multiple environments.</st>
			<st c="50679">As we are</st> <st c="50689">still in the folder of our code component, we can run the command, in which we can either use a publisher prefix from our environment or the unique name of our solution.</st> <st c="50860">In both cases, the PAC CLI will check whether the publisher or solution already exists.</st> <st c="50948">If it does, it will use it to push the component – here is</st> <st c="51007">one option:</st>

```

pac pcf push –solution-unique-name <SOLUTION_NAME>

```

			<st c="51069">Here is</st> <st c="51078">the other:</st>

```

pac pcf push -pp <PUBLISHER_PREFIX>

```

			<st c="51124">Without the name of the solution, this code component will be imported to a temporary</st> `<st c="51211">PowerAppsTools_<prefix></st>` <st c="51234">solution.</st>
			<st c="51244">Preparing the component for release</st>
			<st c="51280">When preparing our</st> <st c="51299">code component for release to the production environment, we should add the</st> `<st c="51376"><PcfBuildMode>production</PcfBuildMode></st>` <st c="51415">property inside the</st> `<st c="51436">pcfproj</st>` <st c="51443">file underneath</st> `<st c="51460">OutputPath</st>`<st c="51470">, as in the</st> <st c="51482">following example:</st>

```

<PropertyGroup>

    <Name>PCF</Name>

    <ProjectGuid>e71d2e10-908c-4123-9f29-3283cbd224ab</ProjectGuid>

    <OutputPath>$(MSBuildThisFileDirectory)out\controls</OutputPath>

    <PcfBuildMode>production</PcfBuildMode>

</PropertyGroup>

```

			<st c="51719">Development mode produces a larger bundle that holds debugging information, which might impact performance in production.</st> <st c="51842">This is why the change should be made to the</st> `<st c="51887">pcfproj</st>` <st c="51894">file before building and deploying</st> <st c="51930">to production.</st>
			<st c="51944">Additionally, when</st> <st c="51964">preparing the solution project,</st> `<st c="51996">SolutionPackagerType</st>`<st c="52016">, which is part of</st> `<st c="52035">PropertyGroup</st>` <st c="52048">in the</st> `<st c="52056">cdsproj</st>` <st c="52063">file, defines the solution type, stating whether we would like to go for managed, unmanaged, or both</st> <st c="52165">solution types:</st>

```

<PropertyGroup>

    <SolutionPackageType>Managed</SolutionPackageType>

</PropertyGroup>

```

			<st c="52264">When running the</st> `<st c="52282">dotnet build</st>` <st c="52294">command, this information will be taken to generate a ZIP file of the defined solution type.</st> <st c="52388">When we are preparing for release to production, we can use</st> `<st c="52448">dotnet build /p:configuration=Release</st>` <st c="52485">to create a release build for the selected</st> <st c="52529">solution type.</st>
			<st c="52543">Adding code components to applications</st>
			<st c="52582">Code components can be used across canvas applications, model-driven apps, and websites.</st> <st c="52672">The following are some examples of adding components</st> <st c="52725">to apps.</st>
			<st c="52733">Model-driven apps</st>
			<st c="52751">Adding components to</st> <st c="52773">model-driven apps</st> <st c="52790">is</st> <st c="52794">very easy:</st>

				1.  <st c="52804">When editing the form, click on</st> **<st c="52837">Components</st>** <st c="52847">in the left navigation bar and click</st> **<st c="52885">Get</st>** **<st c="52889">more components</st>**<st c="52904">.</st>
				2.  <st c="52905">This will</st> <st c="52915">open a</st> **<st c="52923">Get more components</st>** <st c="52942">sidebar, where newly created code components will appear.</st> <st c="53001">Select the</st> <st c="53012">necessary components and click</st> **<st c="53043">Add</st>**<st c="53046">, as can be seen in</st> *<st c="53066">Figure 10</st>**<st c="53075">.12</st>*<st c="53078">:</st>

			![Figure 10.12 – Adding code component to a form in a model-driven app](img/B22208_10_12.jpg)

			<st c="53866">Figure 10.12 – Adding code component to a form in a model-driven app</st>

				1.  <st c="53934">After adding the code components to the list of components, they appear under the</st> **<st c="54017">More components</st>** <st c="54032">group inside the</st> **<st c="54050">Components</st>** <st c="54060">option.</st> <st c="54069">Simply drag and drop the components to the right places in the form and connect the component values with the</st> <st c="54179">correct values.</st>

			<st c="54194">Canvas apps</st>
			<st c="54206">Adding code</st> <st c="54219">components to</st> <st c="54233">canvas applications has to be enabled before it’s possible.</st> <st c="54293">The setting for enabling it is currently available in the Power Platform</st> <st c="54366">admin center:</st>

				1.  <st c="54379">Go to</st> **<st c="54386">Environments</st>**<st c="54398">, select the environment, and then click</st> **<st c="54439">Settings</st>** <st c="54447">|</st> **<st c="54450">Product</st>** <st c="54457">|</st> **<st c="54460">Features</st>**<st c="54468">.</st>
				2.  <st c="54469">Here we will find the</st> **<st c="54492">Power Apps component framework for canvas apps</st>** <st c="54538">setting, which is turned off by default.</st> <st c="54580">If needed,</st> <st c="54591">enable it.</st>
				3.  <st c="54601">Once done, we can</st> <st c="54619">go to our canvas application and click</st> **<st c="54659">Insert</st>** <st c="54665">to see a list of all controls.</st> <st c="54697">Then we can proceed to the</st> **<st c="54724">Code</st>** <st c="54728">tab, sele</st><st c="54738">ct</st> <st c="54742">the components, and click</st> **<st c="54768">Import</st>**<st c="54774">, as shown in</st> *<st c="54788">Figure 10</st>**<st c="54797">.13</st>*<st c="54800">.</st>

			![Figure 10.13 – Adding a code component to the canvas app](img/B22208_10_13.jpg)

			<st c="55336">Figure 10.13 – Adding a code component to the canvas app</st>
			<st c="55392">Power Pages</st>
			<st c="55404">Adding code components to</st> <st c="55431">Power Pages</st> <st c="55442">is similar to the process of doing so for</st> <st c="55485">model-driven apps:</st>

				1.  <st c="55503">Your components can be found by going to</st> **<st c="55545">Data</st>** <st c="55549">in the left</st> <st c="55562">navigation bar.</st>
				2.  <st c="55577">Select the table of choice, click</st> **<st c="55612">Forms</st>** <st c="55617">(or</st> **<st c="55622">V</st><st c="55623">iews</st>**<st c="55627">, depending on the use case), and click</st> **<st c="55667">Get</st>** **<st c="55671">more components</st>**<st c="55686">.</st>

			![Figure 10.14 – Adding code components in Power Pages](img/B22208_10_14.jpg)

			<st c="56268">Figure 10.14 – Adding code components in Power Pages</st>

				1.  <st c="56320">This will open a</st> <st c="56337">sidebar to help you find a custom</st> <st c="56372">code component.</st> <st c="56388">Select the one you require and click</st> **<st c="56425">Add</st>**<st c="56428">, as can be seen in</st> *<st c="56448">Figure 10</st>**<st c="56457">.14.</st>*

			<st c="56461">Now that we have learned how code components are created and added to an application, let’s understand how the application life cycle is managed for</st> <st c="56611">these components.</st>
			<st c="56628">ALM for code components</st>
			<st c="56652">As we saw in the</st> <st c="56670">previous section, the development life cycle of a code component consists of the</st> <st c="56751">following steps:</st>

				1.  <st c="56767">The initialization of the</st> <st c="56794">PCF project</st>
				2.  <st c="56805">Working on the code</st> <st c="56826">component’s implementation</st>
				3.  <st c="56852">Testing/local debugging of the developed code component and pushing it to the Power Platform</st> <st c="56946">development environment</st>
				4.  <st c="56969">Adding the component project reference to the solution in the</st> <st c="57032">development environment</st>
				5.  <st c="57055">Testing the business solution with the included</st> <st c="57104">code components</st>
				6.  <st c="57119">Preparing the code component</st> <st c="57149">for release</st>
				7.  <st c="57160">Importing the solution to other environments</st> <st c="57206">and testing</st>

			<st c="57217">We recommend building automated pipelines by leveraging Power Platform Build Tools in Azure DevOps or GitHub, together with the PAC CLI, to perform these</st> <st c="57372">tasks automatically.</st>
			<st c="57392">As with any other development project, when developing code components, it is also recommended that developers use a source code repository, such as GitHub or Azure DevOps, from the beginning, in which to collaborate.</st> <st c="57611">Using</st> `<st c="57617">pac pcf init</st>` <st c="57629">also creates a</st> `<st c="57645">.gitignore</st>` <st c="57655">file, which instructs the version control system as to which files and folders should be left out since they will be either created or restored during the</st> <st c="57811">build process.</st>
			<st c="57825">It is worth noting that when we have a code component in a solution, we can export the solution together with the code component using Power Platform Build Tools or</st> `<st c="57991">SolutionPackager /action: Extract</st>`<st c="58024">, which incrementally exports changes to the solution metadata.</st> <st c="58088">When a code component is extracted from a solution package, a</st> `<st c="58150">Controls</st>` <st c="58158">subfolder will be created for each code component that is included in</st> <st c="58229">the solution.</st>
			<st c="58242">Using</st> `<st c="58249">pac pcf push</st>` <st c="58261">to push a</st> <st c="58271">code component to an environment, and then using commands such as</st> `<st c="58338">msbuild</st>` <st c="58345">and</st> `<st c="58350">SolutionPackage /action: Pack</st>` <st c="58379">to build a solution, allows us to build and repack the solution and make it ready</st> <st c="58462">for import.</st>
			<st c="58473">Solution strategy for code components</st>
			<st c="58511">Another consideration</st> <st c="58533">with code components is the solution strategy.</st> <st c="58581">As we have learned, our business applications can have dependencies on various solution components.</st> <st c="58681">Power Platform allows us to deploy components together, as part of</st> <st c="58747">a</st> **<st c="58750">single solution</st>**<st c="58765">, or separately, in</st> **<st c="58785">segmented solutions</st>**<st c="58804">, where</st> <st c="58811">functionalities of our business applications are split between</st> <st c="58875">multiple solutions.</st>
			<st c="58894">The segmented approach allows us to be more agile with the development approach of our business solution.</st> <st c="59001">Teams can each focus on their own parts of the business solution and can develop and test independently of the final solution.</st> <st c="59128">Such an approach allows us to decouple and share code components between multiple applications and across environments.</st> <st c="59248">Single solutions have everything in one solution, which means that it is important to follow a good branching strategy if you want to have control over updates to the</st> <st c="59415">complete solution.</st>
			<st c="59433">Versioning and updates to code components</st>
			<st c="59475">Whenever we</st> <st c="59487">deliver changes to our code components that should be</st> <st c="59542">pushed to our business applications, we should make sure that we follow the standard versioning strategy (</st>`<st c="59648">MAJOR.MINOR.PATCH</st>`<st c="59666">), which will ensure that Power Apps applications detect and update dependencies to the</st> <st c="59755">latest version.</st>
			<st c="59770">Inside the manifest file (</st>`<st c="59797">ControlManifest.Input.Xml</st>`<st c="59823">), under the</st> `<st c="59837">control</st>` <st c="59844">element, there is a</st> `<st c="59865">version</st>` <st c="59872">property that should get at least a</st> `<st c="59909">PATCH</st>` <st c="59914">version increase whenever an update to a code component is deployed.</st> <st c="59984">This will ensure that the change can be detected and that both canvas and model-driven apps will receive the latest version of</st> <st c="60111">the component:</st>

```

<control namespace="SimpleReactNS" constructor="SimpleReactPCF" <st c="60190">version="0.0.1"</st> display-name-key="SimpleReactPCF" description-key="SimpleReactPCF description" control-type="virtual" >

```

			<st c="60309">An increase of the version can be done manually by changing the number in the manifest file, or it can be done automatically with the</st> <st c="60444">following command:</st>

```

pac pcf version --strategy manifest

```

			<st c="60498">There are also other strategies, such as</st> `<st c="60540">gittags</st>`<st c="60547">; however, the manifest strategy is by far the simplest, since it does</st> <st c="60619">everything automatically.</st>
			<st c="60644">If we would like to specify the exact value of a version, we can do</st> <st c="60713">the following:</st>

```

pac pcf version --patchversion <PATCH VERSION>

```

			`<st c="60774">MAJOR</st>` <st c="60780">and</st> `<st c="60785">MINOR</st>` <st c="60790">versions</st> <st c="60799">should be aligned with the version of our</st> <st c="60841">Dataverse solution.</st> <st c="60862">If a significant change is made to a solution, then the</st> `<st c="60918">MAJOR</st>` <st c="60923">and/or</st> `<st c="60931">MINOR</st>` <st c="60936">version of the Dataverse solution should be incremented, which should lead also to incrementing the</st> `<st c="61037">MAJOR</st>` <st c="61042">and/or</st> `<st c="61050">MINOR</st>` <st c="61055">version of the</st> <st c="61071">code component.</st>
			<st c="61086">Code component updates for canvas apps</st>
			<st c="61125">In order to update a code component to a newer version inside canvas applications, app makers must open the canvas app in Power Apps Studio and click</st> **<st c="61276">Update</st>** <st c="61282">on the</st> **<st c="61290">Update code components</st>** <st c="61312">popup.</st> <st c="61320">If the update is not done, then the canvas app will continue to use the old version of the</st> <st c="61411">code component.</st>
			<st c="61426">Automated build pipelines for segmented solutions</st>
			<st c="61476">To create a code</st> <st c="61493">component from scratch and keep control over it, we start by creating the code component locally and committing the changes to the</st> <st c="61625">Git repository:</st>

				1.  <st c="61640">Using the</st> `<st c="61651">pac pcf init</st>` <st c="61663">command, we will create a</st> `<st c="61690">pcfproj</st>` <st c="61697">folder.</st> <st c="61706">Once the project is created, we will be committing the changes to the Git repository.</st> <st c="61792">Using</st> `<st c="61798">pac pcf push</st>`<st c="61810">, we will deploy our code component to</st> <st c="61849">the environment.</st>
				2.  <st c="61865">Separately, we will be creating a solution project using</st> `<st c="61923">pac solution init</st>`<st c="61940">, to create a new blank solution, or</st> `<st c="61977">pac solution clone</st>`<st c="61995">, if we already have a solution that we would like to reuse.</st> <st c="62056">The solution is also version-controlled in the</st> <st c="62103">Git repository.</st>
				3.  <st c="62118">Next, we need to add a reference to the solution using the</st> `<st c="62178">pac solution add-reference</st>` <st c="62204">command with the path to our PCF</st> <st c="62238">component project.</st>
				4.  <st c="62256">Now we need to update the solution version in</st> `<st c="62303">Solution.xml</st>` <st c="62315">to reflect the version of the current build.</st> <st c="62361">This can be done by formulating the</st> `<st c="62397">MAJOR.MINOR.BUILD.REVISION</st>` <st c="62423">version with the variables in Azure DevOps/GitHub.</st> <st c="62475">For</st> `<st c="62479">MAJOR</st>`<st c="62484">, we can create</st> `<st c="62500">$(majorVersion)</st>` <st c="62515">variables; for</st> `<st c="62531">MINOR</st>`<st c="62536">, we can create</st> `<st c="62552">$(minorVersion)</st>` <st c="62567">variables; for</st> `<st c="62583">BUILD</st>`<st c="62588">, we can use the predefined</st> `<st c="62616">$(Build.BuildId)</st>`<st c="62632">; and for</st> `<st c="62643">REVISION</st>`<st c="62651">, we can</st> <st c="62660">use</st> `<st c="62664">$(Rev:r)</st>`<st c="62672">.</st>
				5.  <st c="62673">We will also need to change the version value</st> <st c="62720">inside</st> `<st c="62727">ControlManifest.Input.xml</st>`<st c="62752">.</st>
				6.  <st c="62753">Once the version numbers are updated, we can run a task, as part of our build pipeline, using the</st> `<st c="62852">dotnet build /restore /p:configuration=Release</st>` <st c="62898">command (or</st> `<st c="62911">msbuild</st>`<st c="62918">, depending on what tools are we using).</st> <st c="62959">We</st> <st c="62962">configure the task in a way that means it builds only the</st> `<st c="63020">cdsproj</st>` <st c="63027">project.</st> <st c="63037">We can achieve this by setting the</st> `<st c="63072">*.</st>``<st c="63074">cdsproj</st>` <st c="63082">wildcard.</st>

			<st c="63092">Once the build has been completed, we will store the produced ZIP file in the pipeline release artifact using a</st> **<st c="63205">Copy file</st>** <st c="63214">task, which can be used for deployment.</st> <st c="63255">The deployment process is the same as with other solution</st> <st c="63313">import processes.</st>
			<st c="63330">Automated build pipelines for single mixed solutions</st>
			<st c="63383">The single</st> <st c="63395">mixed solutions approach needs to be configured in the solution object explorer, where all the required solution components are added.</st> <st c="63530">Then, we utilize</st> `<st c="63547">SolutionPackager /action: Extract</st>` <st c="63580">to extract the components into the source control system.</st> <st c="63639">Just as earlier, we need to update the solution version in</st> `<st c="63698">Solution.xml</st>` <st c="63710">to reflect the version of the current build.</st> <st c="63756">We also need to change the version value</st> <st c="63797">inside</st> `<st c="63804">ControlManifest.Input.xml</st>`<st c="63829">.</st>
			<st c="63830">We will add a task to the pipeline called</st> **<st c="63873">Power Platform</st>** **<st c="63888">Tool Installer</st>**<st c="63902">.</st>
			<st c="63903">Next, we will restore</st> `<st c="63926">node_modules</st>` <st c="63938">using</st> `<st c="63945">npm task</st>` <st c="63953">with the</st> `<st c="63963">npm ci</st>` <st c="63969">command.</st> <st c="63979">For the production release mode, we will use</st> `<st c="64024">npm task</st>` <st c="64032">with a custom command parameter:</st> `<st c="64066">npm run build -- --buildMode release</st>`<st c="64102">. The output of the build needs to be stored separately in a folder.</st> <st c="64171">Then we will use a</st> `<st c="64227">SolutionPackager /action: Pack</st>` <st c="64257">to package the files and collect the</st> <st c="64294">built solution ZIP in the</st> <st c="64321">pipeline artifact.</st>
			<st c="64339">More information on ALM for PCF components</st>
			<st c="64382">To support this section, we recommend visiting the documentation on ALM for PCF components</st> <st c="64473">here:</st> [<st c="64480">https://learn.microsoft.com/en-us/power-apps/developer/component-framework/code-components-alm</st>](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/code-components-alm)<st c="64574">.</st>
			<st c="64575">We have mentioned many times that code components can also be used with Power Pages.</st> <st c="64661">Since Power Pages allows developers to extend websites greatly with custom code, it is important to take a look at how we can apply ALM to</st> <st c="64800">Power Pages.</st>
			<st c="64812">ALM for Power Pages</st>
			<st c="64832">Power Pages allows</st> <st c="64852">app makers and professional developers to build engaging, scalable, and secure websites.</st> <st c="64941">Since Power Pages websites are built using the HTML, CSS, JavaScript, and Liquid templating languages, developers can follow common web development practices and use known tools, such as Visual Studio Code.</st> <st c="65148">Once changes are made to their websites, developers should follow a process of committing those changes to the repository and applying them to different environments.</st> <st c="65315">This section focuses on describing how ALM is applied to Power Pages websites.</st> <st c="65394">We will take a look at how Power Platform pipelines enable easier and faster configuration of deployment pipelines and how to use Power Platform Build Tools in Azure DevOps to create the pipelines necessary for exporting and importing solutions into</st> <st c="65644">different environments.</st>
			<st c="65667">Power Pages was previously known as Power Apps Portals.</st> <st c="65724">The older data model, known also as the standard data model, used custom tables to store the configurations of Power Apps Portals websites.</st> <st c="65864">As configurations were stored separately, this prevented us from using solutions as was possible with other Power Platform services.</st> <st c="65997">However, in the second half of 2023, Microsoft introduced a new and enhanced data model that was built on system tables, non-config tables, and virtual tables.</st> <st c="66157">This new approach allows us to contain website configurations in solutions, which simplifies the</st> <st c="66254">ALM approach.</st>
			<st c="66267">Information about each site, including whether it uses the enhanced or standard data model, can be found inside the</st> **<st c="66384">Site details</st>** <st c="66396">|</st> **<st c="66399">Data model</st>** <st c="66409">setting for each website in Power Pages.</st> <st c="66451">The same website information can also be found on</st> **<st c="66501">Power Platform Admin center</st>** <st c="66528">|</st> **<st c="66531">Resources</st>** <st c="66540">|</st> **<st c="66543">Power Pages sites</st>**<st c="66560">, where you can select a site and then click</st> **<st c="66605">Manage</st>**<st c="66611">. The information about the data model is found under</st> **<st c="66665">Site Details</st>**<st c="66677">.</st>
			<st c="66678">Migrating from the standard model to the enhanced data model</st>
			<st c="66739">Although all newly created sites in Power Pages use the enhanced data model by default, you might still come across legacy Power Pages websites that are built on the standard data model.</st> <st c="66927">Customers who have websites that use the previous data model should consider migrating their websites to the new model.</st> <st c="67047">The enhanced data model provides benefits over the standard one, such as faster website provisioning, solution support,</st> <st c="67167">and ALM.</st>
			<st c="67175">Microsoft has introduced a migration guide that helps customers to migrate from one data model to the</st> <st c="67278">other:</st> [<st c="67285">https://learn.microsoft.com/en-us/power-pages/admin/migrate-enhanced-data-model</st>](https://learn.microsoft.com/en-us/power-pages/admin/migrate-enhanced-data-model)<st c="67364">.</st>
			<st c="67365">Now that we’ve had a short introduction to Power Pages, we can take a look at how the PAC CLI can support website life</st> <st c="67485">cycle management.</st>
			<st c="67502">Use of the PAC CLI for Power Pages</st>
			<st c="67537">The PAC CLI offers tools for</st> <st c="67566">managing the configuration of Power Pages websites, which includes downloading and uploading website content from and to</st> <st c="67688">Power Pages.</st>
			<st c="67700">There are two specific commands that we will be using in this section.</st> <st c="67772">The first one is the</st> `<st c="67793">pac pages download</st>` <st c="67811">command, which is able to download website content from Dataverse.</st> <st c="67879">The other one is the</st> `<st c="67900">pac pages upload</st>` <st c="67916">command, which can upload website content together with all the manifest files and the deployment profile that is appropriate for the</st> <st c="68051">target environment.</st>
			<st c="68070">These two commands support ALM for Power Pages and can be used together in our CI/CD pipelines, wherever we are not using Power Platform</st> <st c="68208">Build Tools.</st>
			<st c="68220">Deployment profiles</st>
			<st c="68240">A deployment profile</st> <st c="68261">is a file in YAML format that holds a set of values and settings</st> <st c="68326">that are relevant to the target environment.</st> <st c="68372">Deployment profiles are used in a similar way as deployment settings files are in Power Apps.</st> <st c="68466">It is possible to have a deployment profile file for each target environment that we are deploying a website to.</st> <st c="68579">Each of these files contains values that match the configuration settings of the</st> <st c="68660">target environment.</st>
			<st c="68679">Deployment profiles should be added in the</st> `<st c="68723">deployment-profiles</st>` <st c="68742">folder, in the root of the downloaded website folder.</st> <st c="68797">A new folder will need to be created if it has not been created yet.</st> <st c="68866">There we can add the</st> `<st c="68887"><</st>``<st c="68888">profileTag>.deployment.yml</st>` <st c="68914">files.</st>
			<st c="68921">These files will then be used later on, during the upload process, with a</st> `<st c="68996">--deploymentProfile</st>` `<st c="69016">test</st>` <st c="69020">argument:</st>
			![Figure 10.15 – Deployment profiles and manifest files in Power Pages](img/B22208_10_15.jpg)

			<st c="69446">Figure 10.15 – Deployment profiles and manifest files in Power Pages</st>
			<st c="69514">Manifest files</st>
			<st c="69529">When downloading a website from Power Pages, the PAC CLI creates two additional manifest files: an environment</st> <st c="69641">manifest file (</st>`<st c="69656">org-url-manifest.yml</st>`<st c="69677">) and a delete-tracking manifest file (</st>`<st c="69717">manifest.yml</st>`<st c="69730">).</st> <st c="69734">Both files are in YAML format and stored with the</st> `<st c="69784">.portalconfig</st>` <st c="69797">root folder structure of the</st> <st c="69827">downloaded website.</st>
			<st c="69846">The environment manifest file is created with the purpose of optimizing the upload process.</st> <st c="69939">After each download method is run with the PAC CLI, the download method creates a new manifest file, in case one doesn’t exist yet.</st> <st c="70071">If the manifest file exists, it reads the existing manifest file and updates the changes, such as any entries that were removed.</st> <st c="70200">When uploading content, only the updated and new entries are uploaded to</st> <st c="70273">Power Pages.</st>
			<st c="70285">The environment manifest file is environment-specific, intended mainly for development environments, and it should be added to the</st> `<st c="70417">git</st>` `<st c="70421">ignore</st>` <st c="70427">list.</st>
			<st c="70433">The</st> <st c="70438">delete-tracking manifest file is used to keep track of entries removed from the environment.</st> <st c="70531">Whenever a download method is called, all the deleted entries are added to the delete-tracking manifest file.</st> <st c="70641">Once an upload method is called, the method removes the unnecessary entries from Power Pages.</st> <st c="70735">This file is important and should be added to the source control system and transferred to the</st> <st c="70830">target environment.</st>
			<st c="70849">Example of using the PAC CLI for Power Pages</st>
			<st c="70894">We will take a look at the example</st> <st c="70929">of using the PAC CLI for the download and upload of Power</st> <st c="70988">Pages content:</st>

```

# 显示所有 Power Pages 网站

pac pages list

# 下载 Power Pages 网站的内容并将其传输到选定位置。 pac pages download -id <PAGES_WEB_ID> -p <DOWNLOAD_LOCATION>

# 我们对文件进行更改，并决定上传这些更改，使用部署配置文件。 pac pages upload –path <PATH_LOCATION> --environment <ENVIRONMENT_ID> --deploymentProfile <PROFILETAG>

```

			<st c="71400">Using Power Platform Build Tools with Power Pages</st>
			<st c="71450">Power Platform Build Tools</st> <st c="71477">offers two tasks that we have not yet explored:</st> **<st c="71526">Power Platform Download PAPortal</st>** <st c="71558">and</st> **<st c="71563">Power Platform Upload PAPortal</st>**<st c="71593">. Both tasks are intended to operate with</st> <st c="71635">Power Pages.</st>
			<st c="71647">We will first</st> <st c="71662">create an export pipeline that will download the website content and commit everything to the main branch.</st> <st c="71769">The following is a snippet of steps with the tasks needed to enable such</st> <st c="71842">a pipeline:</st>

```

步骤:

- 任务: PowerPlatformToolInstaller@2

inputs:

    默认版本: true

- 任务: PowerPlatformDownloadPaportal@2

inputs:

    认证类型: 'PowerPlatformSPN'

    PowerPlatformSPN: 'PP-DevUS-SPN'

    下载路径: 'Portal/'

    WebsiteId: '$(WebsiteID)'

- 任务: CmdLine@2

inputs:

    脚本: |

    echo 提交所有更改

    git config user.email "<EMAIL@DOMAIN.COM>"

    git config user.name "<USER_NAME>"

    git init

    git checkout -B main

    git add --all

    git commit -m "代码提交"

    git push --set-upstream origin main

    git -c http.extraheader="AUTHORIZATION: bearer $(System.AccessToken)" push -f origin HEAD:main

```

			<st c="72437">After</st> <st c="72444">running the export pipeline, we now have the source code in</st> <st c="72504">Azure Repos.</st>
			<st c="72516">As a next step, we should create deployment profiles with the necessary configuration values, as mentioned in the previous section (on creating a</st> `<st c="72663">deployment-profiles</st>` <st c="72682">folder with corresponding deployment YAML files).</st> <st c="72733">The deployment profile files might not change that often, as they might be environment-specific, but it is worth setting them up and updating them when necessary; it is a good practice to use them from the beginning in order to keep the configuration settings separate</st> <st c="73002">between environments.</st>
			![Figure 10.16 – Deployment profiles in Azure Repos](img/B22208_10_16.jpg)

			<st c="73602">Figure 10.16 – Deployment profiles in Azure Repos</st>
			<st c="73651">Whenever developers have to work on their website, we recommend using a branching strategy and pull request mechanism, in order to keep the main branch protected and always in a</st> <st c="73830">deployable state.</st>
			<st c="73847">After the pull request is validated and approved, branches can be merged to the</st> <st c="73928">main branch.</st>
			<st c="73940">Merging branches can automatically trigger the deployment pipeline.</st> <st c="74009">The following is a snippet of tasks from the</st> <st c="74054">deployment pipeline:</st>

```

步骤:

- 任务: PowerPlatformToolInstaller@2

inputs:

    默认版本: true

- 任务: PowerPlatformUploadPaportal@2

inputs:

    认证类型: 'PowerPlatformSPN'

    PowerPlatformSPN: 'PP-DevUS-SPN'

    上传路径: 'Portal/$(WebsiteName)'

    DeploymentProfile: '$(profileTag)'

```

			<st c="74336">Here we have two variables,</st> `<st c="74365">WebsiteName</st>` <st c="74376">and</st> `<st c="74381">profileTag</st>`<st c="74391">. The first is the name of our Power Pages instance as it appears in our Azure Repos;</st> `<st c="74477">profileTag</st>` <st c="74487">should match the tag of the</st> <st c="74516">deployment profile.</st>
			<st c="74535">We should</st> <st c="74546">not forget to perform proper testing of our Power Pages websites in order to validate our work and confirm the rollout to our</st> <st c="74672">target environment.</st>
			<st c="74691">We familiarized ourselves with Power Platform pipelines in</st> *<st c="74751">Chapter 5</st>*<st c="74760">, where we talked about managed pipelines.</st> <st c="74803">Let’s take a look at how we can utilize Power Platform pipelines to deploy our Power</st> <st c="74888">Pages websites.</st>
			<st c="74903">Using Power Platform pipelines with Power Pages</st>
			**<st c="74951">Power Platform pipelines</st>** <st c="74976">allow</st> <st c="74983">the quick and easy configuration of deployment pipelines that enable organizations to automate the release of a build from the development environment to other environments, such as test</st> <st c="75170">and production.</st>
			<st c="75185">In order to</st> <st c="75197">utilize Power Platform pipelines, all target environments must be enabled as</st> **<st c="75275">Managed Environments</st>**<st c="75295">, and all environments that are used in pipelines must be Dataverse-enabled.</st> <st c="75372">All websites have to be created using the enhanced</st> <st c="75423">data model.</st>
			<st c="75434">Once the prerequisites are met, we are able to proceed with deploying our websites to</st> <st c="75521">target environments.</st>
			<st c="75541">First, we need to prepare the solution.</st> <st c="75582">In Power Apps, we open</st> **<st c="75605">Solutions</st>**<st c="75614">, where we either create a new solution or open an existing one.</st> <st c="75679">Then, we open the solution and add our existing website to the solution by selecting</st> **<st c="75764">Add existing</st>** <st c="75776">|</st> **<st c="75779">Site</st>**<st c="75783">. In the sidebar that opens, we select the website that we would like to add to the solution.</st> <st c="75877">Once the solution objects are added to the solution, we can proceed with setting up the pipeline.</st> <st c="75975">While still in the newly created solution, we click</st> **<st c="76027">Pipelines</st>**<st c="76036">, which can be found in the left navigation bar.</st> <st c="76085">In</st> **<st c="76088">Pipelines</st>**<st c="76097">, we can select existing pipelines that the DevOps or IT operations team has created for us in advance, or we can click</st> **<st c="76217">Create new pipeline</st>**<st c="76236">. The new popup screen allows us to define the deployment pipeline, which will take our solutions from the source to the</st> <st c="76357">target environment:</st>
			![Figure 10.17 – Power Platform pipelines](img/B22208_10_17.jpg)

			<st c="77208">Figure 10.17 – Power Platform pipelines</st>
			<st c="77247">Once the</st> <st c="77256">pipeline is configured, we can press the</st> **<st c="77298">Deploy here</st>** <st c="77309">button and the deployment process</st> <st c="77344">will start.</st>
			<st c="77355">This is a more simplified kind of deployment process.</st> <st c="77410">Still, it is suitable for many scenarios.</st> <st c="77452">It opens the door for app makers who might not be skilled in DevOps processes and CI/CD pipelines but are still aware of the importance of the automated approach of deploying business solutions and keeping</st> <st c="77658">environments separate.</st>
			<st c="77680">Summary</st>
			<st c="77688">This chapter continued building on the possibilities that Microsoft Power Platform provides to pro-developers.</st> <st c="77800">The platform definitely shouldn’t be taken lightly, even though it is a part of the low-code development toolset.</st> <st c="77914">As we have seen, it offers many opportunities for pro-developers to extend the user experience and integrate solutions with complex backend and legacy systems.</st> <st c="78074">Now we can see how important it is to form agile fusion teams where pro-developers have the ability to customize solutions using different custom components, in order to deliver on</st> <st c="78255">project requirements.</st>
			<st c="78276">We looked at examples of how connection references and environment variables can be used together with the DevOps approach.</st> <st c="78401">We continued with a look at canvas components and component libraries.</st> <st c="78472">We then covered the Power Apps component framework and code components.</st> <st c="78544">All these components can be added to solutions as solution objects, and DevOps pipelines can be used to deploy them securely to other environments as development progresses.</st> <st c="78718">We closed this chapter by taking a look at Power Pages ALM.</st> <st c="78778">We have investigated how we can use Power Platform Build Tools to support ALM process for our websites, as well as checked a simpler approach using managed pipelines in</st> <st c="78947">Power Platform.</st>
			<st c="78962">The next chapter will focus on best practices for managing environments in a governed and secure way and how the IT operations team can describe environments using code.</st> <st c="79133">This is a very important part of the DevOps life cycle that we have to address, even though it is not</st> <st c="79235">very pro-dev-oriented.</st>
			<st c="79257">Further reading</st>

				*   <st c="79273">Connectors</st> <st c="79285">architecture:</st> [<st c="79299">https://learn.microsoft.com/en-us/connectors/connector-architecture</st>](https://learn.microsoft.com/en-us/connectors/connector-architecture)
				*   <st c="79366">Connection</st> <st c="79378">references:</st> [<st c="79390">https://learn.microsoft.com/en-us/power-apps/maker/data-platform/create-connection-reference</st>](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/create-connection-reference)
				*   <st c="79482">Environmental</st> <st c="79497">variables:</st> [<st c="79508">https://learn.microsoft.com/en-us/power-apps/maker/data-platform/environmentvariables</st>](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/environmentvariables)
				*   <st c="79593">Component</st> <st c="79604">library:</st> [<st c="79613">https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/component-library</st>](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/component-library)
				*   <st c="79693">Power Apps component</st> <st c="79715">framework:</st> [<st c="79726">https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview</st>](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/overview)
				*   <st c="79809">Power Pages</st> <st c="79822">ALM:</st> [<st c="79827">https://learn.microsoft.com/en-us/power-pages/configure/portals-alm</st>](https://learn.microsoft.com/en-us/power-pages/configure/portals-alm)

```
