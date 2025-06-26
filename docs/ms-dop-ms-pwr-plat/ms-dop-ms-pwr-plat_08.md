# <st c="0">8</st>

# <st c="2">展示 ALM 和 DevOps 实现</st>

<st c="45">在本章中，我们将通过一个动手练习，使用真实的例子，结合代码片段和逐步说明。</st> <st c="184">我们将使用</st> <st c="199">Power Platform 企业模板</st> **<st c="234">Power Platform 企业模板</st>** <st c="268">来演示 DevOps 流程的端到端场景。</st> <st c="329">该模板是</st> **<st c="350">员工奖励模板</st>**<st c="373">，我们可以用它来表扬</st> <st c="401">他人出色的成就。</st> <st c="445">可用的解决方案</st> <st c="467">提供了一个基于模型的应用程序（</st>**<st c="505">Kudos 管理应用</st>**<st c="530">），用于管理奖励、选择退出用户并创建徽章，这些徽章可以在画布应用中使用，填写奖励内容。</st> <st c="649">这个画布应用，称为</st> **<st c="684">Kudos 应用</st>** <st c="693">在解决方案中，提供了一个用户界面</st> <st c="736">供组织中的用户使用。</st> <st c="767">由于 Kudos</st> <st c="782">解决方案依赖于</st> **<st c="807">员工体验基础解决方案</st>** <st c="831">，我们将学习如何通过引入多个</st> **<st c="927">GitHub 工作流</st>** <st c="943">在发布火车中同时管理两个解决方案，并利用部署包。</st> <st c="1004">我们将为此应用定义分支策略，并深入了解不同的 GitHub 工作流，以及</st> **<st c="1120">DevSecOps</st>** <st c="1129">任务来管理这些解决方案的开发。</st> <st c="1161">我们将介绍待办事项管理，并使用分支策略来保护我们的主分支，避免意外更改。</st> <st c="1299">我们将为 Kudos 应用创建测试，并引入</st> **<st c="1352">监控</st>** <st c="1362">到我们的应用和流程中。</st> <st c="1385">最后，我们将学习</st> **<st c="1414">功能开关</st>** <st c="1427">以及如何使用它们启用或禁用</st> <st c="1461">我们应用中的某些功能。</st>

<st c="1510">在本章中，我们将涵盖以下</st> <st c="1564">主要主题：</st>

+   <st c="1576">练习 – 存储库管理和分支策略</st> <st c="1636">对于应用程序</st>

+   <st c="1652">练习 – 构建 CD 流水线和一个</st> <st c="1692">发布火车</st>

+   <st c="1705">练习 – 待办事项管理</st> <st c="1736">在 GitHub 中</st>

+   <st c="1745">练习 –</st> <st c="1757">测试解决方案</st>

+   <st c="1774">练习 –</st> <st c="1786">监控应用程序</st>

+   <st c="1809">练习 – 引入</st> <st c="1833">功能开关</st>

# <st c="1846">技术要求</st>

<st c="1869">要深入了解 DevSecOps 方法和工具，我们需要具备以下内容：</st>

+   **<st c="1961">Microsoft Azure 订阅</st>**<st c="1992">：我们可以通过</st> [<st c="2053">https://azure.microsoft.com/en-us/free</st>](https://azure.microsoft.com/en-us/free)<st c="2091">注册 Microsoft Azure 订阅。如果我们拥有 Visual Studio 订阅或是 Microsoft 认证培训师，可以通过 MSDN 订阅加入，每月获得 150 美元的信用额度。</st>

+   **<st c="2244">Power Platform 订阅</st>**<st c="2274">：如果我们已经拥有 Microsoft Entra ID 工作账户，可以注册 Power Apps 开发者计划（</st>[<st c="2325">https://www.microsoft.com/en-us/power-platform/products/power-apps/free</st>](https://www.microsoft.com/en-us/power-platform/products/power-apps/free)<st c="2397">），或者可以加入 Microsoft 365 开发者计划（</st>[<st c="2507">https://developer.microsoft.com/en-us/microsoft-365/dev-program</st>](https://developer.microsoft.com/en-us/microsoft-365/dev-program)<st c="2571">）。</st>

+   **<st c="2574">GitHub 账户和公共</st>** **<st c="2604">仓库</st>**<st c="2614">：（</st>[<st c="2618">https://github.com/signup</st>](https://github.com/signup)<st c="2644">）</st>

+   **<st c="2646">GitHub 高级安全功能</st>** <st c="2679">对于公共</st> <st c="2714">仓库免费提供：</st> [<st c="2728">https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security#about-advanced-security-features</st>](https://docs.github.com/en/get-started/learning-about-github/about-github-advanced-security#about-advanced-security-features)<st c="2852">。</st>

+   **<st c="2853">示例和操作指南</st>** <st c="2878">将在本章中讨论，并位于</st> <st c="2925">以下位置：</st> [<st c="2928">https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform/tree/main/Chapter08</st>](https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform/tree/main/Chapter08)<st c="3027">。</st>

+   **<st c="3028">Azure CLI</st>**<st c="3042">：我们可以通过安装指南直接在机器上安装 Azure CLI（</st>[<st c="3135">https://learn.microsoft.com/en-us/cli/azure/install-azure-cli#install</st>](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli#install)<st c="3205">），或者可以通过 Azure 门户使用</st> **<st c="3223">Azure Cloud Shell</st>** <st c="3240">获取交互式 Bash 或</st> <st c="3293">PowerShell 会话。</st>

+   **<st c="3312">GitHub Codespaces</st>**<st c="3330">：GitHub 每月为个人提供 60 小时的计算资源，免费使用。</st> <st c="3413">要创建并启动 GitHub Codespaces，只需拥有一个</st> **<st c="3475">GitHub 账户</st>**<st c="3488">。</st>

+   `<st c="3683">.devcontainer/devcontaine</st><st c="3708">r.json</st>`<st c="3715">。</st>

# <st c="3716">练习 – 应用程序的仓库管理和分支策略</st>

<st c="3792">在我们深入实际操作之前，</st> <st c="3826">值得先了解一下 Power Platform 企业模板中的 Kudos 应用。</st> <st c="3857">我们可以通过访问企业模板的官方文档</st> <st c="3929">了解更多关于该解决方案的信息：</st> [<st c="4034">https://learn.microsoft.com/en-us/power-platform/enterprise-templates/hr/employee-kudos/install-and-set-up</st>](https://learn.microsoft.com/en-us/power-platform/enterprise-templates/hr/employee-kudos/install-and-set-up)<st c="4140">。我们为本书提供的仓库中包含了此解决方案的源代码，包含了我们将在整个练习过程中使用的额外扩展和工作流。</st>

<st c="4319">在本练习中，我们</st> <st c="4340">将创建一个公共的</st> **<st c="4370">GitHub 仓库</st>** <st c="4387">，该仓库将托管本章的应用程序，并且我们将</st> <st c="4486">设置我们的分支策略并创建</st> <st c="4527">部署 Kudos 应用到 Power Platform 生产环境的前置条件。</st> <st c="4627">我们将采取以下步骤：</st>

| **<st c="4660">步骤</st>** | **<st c="4665">描述</st>** |
| --- | --- |
| <st c="4677">1.</st> | <st c="4680">克隆示例仓库到我们自己的 GitHub</st> <st c="4727">公共仓库中。</st> |
| <st c="4745">2.</st> | <st c="4748">创建我们的 Power Platform 生产环境，该环境将托管 Kudos 应用以及我们将在 GitHub 工作流中使用的服务主体。</st> <st c="4891">该服务主体的凭证存储在</st> <st c="4946">GitHub secrets 中。</st> |
| <st c="4961">3.</st> | <st c="4964">创建一个 Microsoft Entra ID 组（Azure AD 组），该组将包含可以使用</st> <st c="5056">该应用程序的用户。</st> |
| <st c="5072">4.</st> | <st c="5075">在生产环境中创建 Kudos 应用的连接。</st> |
| <st c="5146">5.</st> | <st c="5149">创建部署设置文件，以管理 CI/CD 工作流中的连接引用，并执行`Release to Production` <st c="5282">流程，将 Kudos 应用部署到</st> <st c="5315">生产环境中。</st> |
| <st c="5338">6.</st> | <st c="5341">基于 GitHub flow 定义我们的分支策略。</st> <st c="5382"> |
| <st c="5394">7.</st> | <st c="5397">创建我们的第一个分支保护规则，以强制对主分支的拉取请求</st> <st c="5463">进行保护。</st> |

<st c="5471">表 8.1 – 练习步骤</st>

<st c="5506">为了简化接下来的</st> <st c="5522">脚本执行</st> <st c="5540">和</st> <st c="5547">后续步骤</st> <st c="5562">及章节的操作，我们将使用</st> **<st c="5589">GitHub Codespaces</st>**<st c="5606">。GitHub Codespaces 提供的计算资源</st> <st c="5654">和开发容器托管在 GitHub 云端。</st> <st c="5717">我们已经准备了一个包含各种工具的容器，例如 GitHub CLI、</st> **<st c="5795">Git CLI</st>**<st c="5802">、</st> **<st c="5804">PAC CLI</st>**<st c="5811">以及 Azure CLI，这些工具已预装在其中。</st> <st c="5845">其配置文件位于以下位置：</st> <st c="5867">直接位于</st> [<st c="5908">https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform/blob/main/.devcontainer/devcontainer.json</st>](https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform/blob/main/.devcontainer/devcontainer.json)<st c="6029">。</st>

<st c="6030">要启动容器，我们需要在网页浏览器中导航至本书的 GitHub 仓库（</st>[<st c="6127">https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform</st>](https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform)<st c="6207">），然后点击</st> **<st c="6227">代码</st>** <st c="6231">按钮，再点击</st> **<st c="6252">在主分支上创建代码空间</st>** <st c="6276">按钮，如下图所示：</st>

![图 8.1 – 在 GitHub 中创建代码空间](img/B22208_08_1.jpg)

<st c="7233">图 8.1 – 在 GitHub 中创建代码空间</st>

<st c="7276">点击按钮后，</st> <st c="7291">系统会解析</st> `<st c="7308">devcontainer.json</st>` <st c="7325">文件，</st> <st c="7331">该文件位于</st> `<st c="7351">.devcontainer</st>` <st c="7364">文件夹中，然后</st> <st c="7386">基于该配置，</st> <st c="7392">在后台创建一个新的</st> **<st c="7422">Docker 镜像</st>** <st c="7434">。</st> <st c="7465">该镜像将作为容器运行，并且在新的浏览器窗口中，Visual Studio Code 将很快启动，预装了在</st> <st c="7611">JSON 文件中描述的功能：</st>

![图 8.2 – 浏览器中的代码空间](img/B22208_08_2.jpg)

<st c="8188">图 8.2 – 浏览器中的代码空间</st>

<st c="8225">在</st> <st c="8232">终端</st> <st c="8242">窗口（通常是</st> **<st c="8258">Bash</st>**<st c="8262">，但也可以使用</st> **<st c="8268">PowerShell</st>** <st c="8278">）中，我们</st> <st c="8315">将开始创建我们自己的</st> <st c="8354">代码库，按步骤进行：</st>

1.  `<st c="8801">copilot suggest</st>` <st c="8816">或</st> `<st c="8820">copilot explain</st>` <st c="8835">参数，用于在不切换上下文的情况下与 GitHub Copilot 进行交互。</st> `<st c="8906">Gh copilot suggest</st>` <st c="8924">将我们的自然语言提示转换为正确的 GitHub CLI 命令，并附带适当的参数，而</st> `<st c="9035">gh copilot explain</st>` <st c="9053">则用自然语言描述我们感兴趣的命令。</st> <st c="9124">我们可以使用以下脚本，在我们自己的 GitHub 企业组织中创建一个名为</st> `<st c="9186">Kudos</st>` <st c="9191">的仓库（如果可用），或者在登录后使用我们自己的 GitHub 账户（GitHub Codespaces 默认提供有限访问 GitHub</st> <st c="9363">管理端点的权限）：</st>

    ```
     export GITHUB_TOKEN=
    gh auth login
    gh auth switch
    # move out of the book repoto "/workspaces"
    cd .. gh repo create Kudos --public --clone
    git clone https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform.git
    ```

    <st c="10110">克隆后，我们可以将</st> `<st c="10164">Chapter08</st>` <st c="10173">文件夹中的文件和文件夹复制到我们的</st> `<st c="10188">Kudos</st>` <st c="10193">文件夹中（即</st> <st c="10206">git 仓库）：</st>

    ```
    cp -rT ./Mastering-DevOps-on-Microsoft-Power-Platform/Chapter08/ ./Kudos
    ```

    <st c="10289">然后，我们可以使用以下脚本将文件添加、提交并推送到</st> <st c="10371">新仓库：</st>

    ```
    # use the credentials of gh CLI in Git commands
    gh auth setup-git
    git config --global user.email "ouremail@address.com"
    git config --global user.name "Our Name"
    git add . git commit -m "Baseline Kudos app"
    git branch -M main
    git push -u origin main
    ```

    <st c="10635">这样，我们</st> <st c="10640">就拥有了新</st> <st c="10650">仓库中的一切，将在</st> <st c="10676">我们的</st> <st c="10707">动手操作练习中使用：</st>

![图 8.3 – 我们自己的仓库与 Kudos 应用程序](img/B22208_08_3.jpg)

<st c="11338">图 8.3 – 我们自己的仓库与 Kudos 应用程序</st>

<st c="11388">在我们的新仓库中，</st> <st c="11417">也有一个</st> `<st c="11428">.devcontainer</st>` <st c="11441">文件夹，里面包含 GitHub Codespace 定义。</st> <st c="11491">在这里，让我们在主分支上创建自己的 codespace，并再次通过 GitHub 进行身份验证，以便获得对</st> <st c="11613">仓库的写权限：</st>

```
 export GITHUB_TOKEN=
gh auth login
gh auth switch
```

<st c="11678">此外，如果</st> <st c="11695">我们已经正确完成了每个准备步骤，并且现在打开</st> **<st c="11759">Actions</st>** <st c="11766">标签</st> <st c="11770">，我们应该看到可用的 GitHub 工作流，如下图所示：</st>

![图 8.4 – Kudos 应用程序的 GitHub 工作流](img/B22208_08_4.jpg)

<st c="12492">图 8.4 – Kudos 应用程序的 GitHub 工作流</st>

<st c="12539">我们将很快使用这些工作流，将我们的 Kudos 应用程序部署到 Power Platform</st> <st c="12617">生产环境中。</st>

1.  `<st c="13033">pac admin create</st>` <st c="13049">命令，而创建服务主体时，我们应用</st> `<st c="13111">pac admin create-service-principal</st>` <st c="13145">命令。</st> <st c="13155">所以，让我们使用交互式登录在 GitHub Codespaces</st> <st c="13303">或本地创建我们的 Kudos 应用程序的生产环境：</st>

    ```
     pac auth create <st c="13331">pac admin create --name Kudos-Prod --region Europe \</st>
     <st c="13383">--currency EUR \</st>
    <st c="14796">gh repo create</st> command in our GitHub codespace, the default repository is our Kudos repository. Otherwise, we can use <st c="14914">gh repo set-default owner/repo</st> command to set it as default.
    ```

1.  **<st c="14974">创建 AAD 组</st>**<st c="14994">：如果我们使用服务主体将解决方案部署到目标环境，那么解决方案中的应用将归服务主体所有。</st> <st c="15155">为了让其他人访问这些应用，我们需要</st> <st c="15205">创建一个</st> **<st c="15218">Microsoft Entra ID 安全组</st>** <st c="15251">（我们在</st> *<st c="15283">第七章</st>*<st c="15292">中学习了如何操作）。</st> <st c="15296">我们需要被分配</st> *<st c="15323">组管理员</st>* <st c="15343">Microsoft Entra 内置角色，才能管理组的创建和成员分配。</st> <st c="15422">让我们执行以下</st> <st c="15449">脚本来创建 Microsoft Entra</st> <st c="15485">ID 组：</st>

    ```
    <st c="15494">az login</st>
    <st c="15503">az ad group create --display-name $GROUP_NAME --mail-nickname $GROUP_NAME</st> # get the user object id <st c="15603">AADObjectID=$(az ad user show \</st>
     <st c="15634">--id $userPrincipalName \</st>
     <st c="15660">--query id \</st>
     <st c="15673">--output tsv)</st> # add a member to the group <st c="15827">\.github\workflows\share-app.ps1</st>, we can share the Kudos app with the created security group. At the time of writing, there is no PAC CLI command that can be used to share an app with Entra ID groups; that’s why we had to use <st c="16079">Set-AdminPowerAppRoleAssignment -PrincipalType "Group" -PrincipalObjectId $GroupID -RoleName CanView -AppName $AppName -EnvironmentName $EnvironmentName</st> <st c="16231">cmdlet</st> here. The security group’s object ID needs to be provided as an input parameter to the script. The GitHub workflows available in the repository also expect this AAD group ID as input.
    ```

1.  **<st c="16422">准备 Power Platform 生产环境</st>**<st c="16472">：Kudos 应用程序包含四个 Power Automate 云流和 Power Apps 画布应用，这些应用使用连接引用及其对应的连接，已为</st> *<st c="16637">Dataverse</st>*<st c="16646">、</st> *<st c="16648">Office 365 Outlook</st>*<st c="16666"> 和 </st> *<st c="16672">Office 365 Users</st>* <st c="16688">连接器创建。</st> <st c="16701">这些</st> **<st c="16705">连接引用</st>** <st c="16726">用于在其他环境中使连接</st> <st c="16759">可调整。</st> <st c="16794">这些</st> **<st c="16798">连接器</st>** <st c="16808">是连接的定义；就像</st> **<st c="16865">面向对象编程</st>** <st c="16892">（</st>**<st c="16894">OOP</st>**<st c="16897">）中的类一样，它们是基于</st> **<st c="16910">OpenAPI</st>** **<st c="16923">REST API</st>** <st c="16932">规范，</st> <st c="16952">并在环境中实例化为</st> **<st c="16973">连接</st>** <st c="16984">（即面向对象编程中的对象）。</st> <st c="17028">当我们使用解决方案创建流、应用及其他 Power Platform 资产时，每次在流或应用中启动新的连接时，都会在解决方案中自动创建一个</st> **<st c="17178">连接引用</st>** <st c="17198">。</st> <st c="17241">如果我们在解决方案外工作并在</st> **<st c="17300">我的流</st>**<st c="17308">下创建流，则会创建直接连接，而不是连接引用。</st> <st c="17380">然而，PAC CLI 可以代表服务主体在我们的生产环境中创建 Dataverse 连接，使用以下脚本：</st>

    ```
     pac connection create [--environment] --tenant-id --name --application-id --client-secret
    ```

    <st c="17621">Office 365 Outlook 和 Office 365 用户</st> <st c="17626">连接器需要服务账户</st> <st c="17665">和真实用户账户，而不是服务主体，用于在 Outlook 和 Office 365 API 中进行身份验证。</st> <st c="17702">我们不能将 O365 或 M365 许可证分配给服务主体。</st> <st c="17860">这就是为什么我们需要在部署解决方案之前，在目标环境中创建这些</st> <st c="17895">连接。</st> <st c="17965">我们可以在</st> <st c="17978">**PowerApps 制作门户**</st> <st c="18026">中的</st> **<st c="18037">连接</st>** <st c="18048">面板，通过点击</st> **<st c="18071">+ 新建连接</st>** <st c="18087">按钮，如下图所示：</st>

![图 8.5 – Kudos 应用的连接](img/B22208_08_5.jpg)

<st c="18589">图 8.5 – Kudos 应用的连接</st>

<st c="18639">创建三个</st> <st c="18665">连接后，我们应该看到与图中显示相同的结果。</st> <st c="18736">最后，我们需要将这些连接与服务主体和之前在</st> *<st c="18860">步骤 2</st>*<st c="18866">中创建的应用用户共享，通过点击三个点并选择</st> **<st c="18913">共享</st>** <st c="18918">菜单项</st> <st c="18932">在</st> **<st c="18935">连接</st>**<st c="18946">：</st>

![图 8.6 – 与服务主体共享连接](img/B22208_08_6.jpg)

<st c="19269">图 8.6 – 与服务主体共享连接</st>

<st c="19331">让我们通过</st> <st c="19338">引入部署</st> <st c="19380">设置文件</st> <st c="19408">来在部署中使用这些连接。</st>

1.  **<st c="19422">部署设置文件</st>**<st c="19447">：为了在解决方案部署期间使用这些连接，我们需要使用一个所谓的</st> **<st c="19541">部署设置文件</st>**<st c="19564">。该文件已经为解决方案生成，脚本如下：</st>

    ```
     pac solution create-settings --solution-zip .\mpa_Kudos_1_0_0_36.zip
    ```

    <st c="19715">此命令提取</st> `<st c="19842">\src\mpa_Kudos\deploymentSettings.json</st>` <st c="19880">文件，该文件位于</st> <st c="19893">GitHub 仓库中：</st>

    ```
    {
      "EnvironmentVariables": [], <st c="19942">"ConnectionReferences": [</st> {
          "LogicalName": "mpa_KudosDataverse",
          "ConnectionId": "[Dataverse]",
          "ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_commondataserviceforapps"
        }, <st c="20126">{</st>
     <st c="20127">"LogicalName": "mpa_KudosO365",</st>
     <st c="20159">"ConnectionId": "[O365]",</st>
     <st c="20185">"ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_office365users"</st>
     <st c="20260">},</st> {
          "LogicalName": "mpa_KudosOutlook",
          "ConnectionId": "[Outlook]",
          "ConnectorId": "/providers/Microsoft.PowerApps/apis/shared_office365"
        }]}
    ```

    <st c="20403">缺失的</st> `<st c="20416">ConnectionId</st>` <st c="20428">值会在</st> <st c="20443">GitHub 工作流运行时</st> <st c="20450">设置。</st> <st c="20477">这些值</st> <st c="20490">是我们工作流的输入参数。</st> <st c="20536">要从我们的 Power Platform 环境中获取这些值，我们</st> <st c="20595">需要点击 Power Apps maker portal 中的连接</st> **<st c="20636">Power Apps 创建者门户</st>** <st c="20659">并复制连接的 ID</st> <st c="20695">从相应的 URL 中。</st> <st c="20725">下图展示了 Dataverse 连接的连接 ID，突出显示在</st> <st c="20814">URL 中：</st>

![图 8.7 – URL 中的连接 ID](img/B22208_08_7.jpg)

<st c="21339">图 8.7 – URL 中的连接 ID</st>

<st c="21380">第一个</st> <st c="21390">GUID 在 URL 中是环境 ID，而</st> <st c="21438">第二个 GUID 是连接</st> <st c="21472">ID。</st> <st c="21476">以下示例突出了</st> <st c="21509">第二个 GUID：</st>

```
 https://make.powerapps.com/environments/<<Environment GUID>>/connections/shared_commondataserviceforapps/<st c="21877">\.github\workflows\cd-to-prod.yml</st> contains the default values of these connection IDs, which we can overwrite and commit back to the main branch. With everything in place, we can deploy the Kudos app from the main branch to our production Power Platform environment, by starting this workflow with the gathered connection IDs:
			![Figure 8.8 – The Release to Production workflow with parameters](img/B22208_08_8.jpg)

			<st c="22548">Figure 8.8 – The Release to Production workflow with parameters</st>
			<st c="22611">To</st> <st c="22615">store these connection IDs, we can</st> <st c="22649">alternatively</st> <st c="22663">use</st> **<st c="22668">GitHub environments</st>**<st c="22687">. In the case of our developer</st> <st c="22717">branch, we will use a</st> <st c="22740">dedicated GitHub environment to pre-configure these values later in</st> <st c="22808">this chapter.</st>

				1.  **<st c="22821">Create our branch strategy</st>**<st c="22848">: At the beginning of our development project, we need to design which branching and merging strategy we will use.</st> <st c="22964">In</st> *<st c="22967">Chapter 5</st>*<st c="22976">, we learned about the</st> **<st c="22999">GitHub flow</st>**<st c="23010">, and this is what we</st> <st c="23031">are going to create for the Kudos application.</st> <st c="23079">The following figure shows our strategy and the direct mapping between Power Platform environments and</st> <st c="23182">Git branches:</st>

			![Figure 8.9 – The branch strategy](img/B22208_08_9.jpg)

			<st c="23301">Figure 8.9 – The branch strategy</st>
			<st c="23333">The</st> <st c="23337">arrows</st> <st c="23344">between the Power Platform environments and Git branches represent the code and deployment flows.</st> <st c="23443">A production environment</st> <st c="23467">can handle only</st> `<st c="23738">dev/DEV-US_XXX_Z</st>` <st c="23754">branch represents a short-lived feature branch containing the implementation</st> <st c="23832">of a user story or a</st> <st c="23853">bug fix.</st>

				1.  `<st c="24332">main</st>` <st c="24336">in the</st> **<st c="24344">Branch name pattern</st>** <st c="24363">field, and we need to select the</st> **<st c="24397">Require a pull request before merging</st>** <st c="24434">checkbox, as shown in the</st> <st c="24461">following figure:</st>

			![Figure 8.10 – The branch protection rule for main](img/B22208_08_10.jpg)

			<st c="26022">Figure 8.10 – The branch protection rule for main</st>
			<st c="26071">At the</st> <st c="26079">bottom of the page, we need</st> <st c="26106">to click</st> <st c="26115">on the</st> **<st c="26123">Create</st>** <st c="26129">button.</st> <st c="26138">With this setting, we ensure that only pull requests are allowed to our main branch.</st> <st c="26223">Note that, by default,</st> **<st c="26246">Require approvals</st>** <st c="26263">is checked, which means at least someone else should review our pull requests.</st> <st c="26343">Since we cannot assign a pull request to ourselves, if we</st> <st c="26401">work</st> <st c="26405">alone, it is recommended to uncheck</st> **<st c="26442">Require approvals</st>**<st c="26459">. However, it is obviously not best practice to do so in a real-world project.</st> <st c="26538">Besides this setting, we need to define the type of merge that we want to allow developers to do.</st> <st c="26636">It is recommended to use</st> **<st c="26661">squash merging</st>**<st c="26675">, which combines all commits</st> <st c="26703">from the head branch into a single commit in the target branch.</st> <st c="26768">This significantly reduces the commit history, since all changes we commit in our developer branch sequentially will be combined into one commit after a successful pull request to the parent branch.</st> <st c="26967">We can force this type of merge by going to the</st> **<st c="27015">Settings</st>** <st c="27023">menu in GitHub and, under the</st> **<st c="27054">General</st>** <st c="27061">blade, removing every other</st> <st c="27090">merge type:</st>
			![Figure 8.11 – Enforcing squash merging](img/B22208_08_11.jpg)

			<st c="27759">Figure 8.11 – Enforcing squash merging</st>
			<st c="27797">Checking</st> <st c="27806">only</st> **<st c="27812">Allow squash merging</st>** <st c="27832">will force</st> <st c="27844">the pull</st> <st c="27852">requests to be squash commits.</st> <st c="27884">The other two options,</st> **<st c="27907">Allow merge commits</st>** <st c="27926">and</st> **<st c="27931">Allow rebase merging</st>**<st c="27951">, will copy the commit history of the child</st> <st c="27995">branches into the main branch, leading to large history nodes in the</st> <st c="28064">commit history.</st>
			<st c="28079">If we encounter any issue in the</st> <st c="28113">previous steps, we can use the following settings to get verbose logging of</st> <st c="28189">the commands.</st>

				*   `<st c="28245">GH_DEBUG</st>` <st c="28253">environment variable.</st> <st c="28276">If its value is</st> `<st c="28292">1</st>`<st c="28293">, it provides more insights, but if we set this variable to</st> `<st c="28353">api</st>`<st c="28356">, then we will see every REST API call to the</st> <st c="28402">GitHub endpoints:</st>

    ```

    export GH_DEBUG=<st c="28488">gh</st> 命令执行将会在标准输出中打印详细的跟踪信息。

    ```

    				*   `<st c="28632">--debug</st>` <st c="28639">flag to the end of every command that we want</st> <st c="28686">to troubleshoot:</st>

    ```

    az login <st c="28712">--debug</st>

    ```

    				*   **<st c="28719">The PAC CLI</st>**<st c="28731">: At the</st> <st c="28741">time of writing, the PAC CLI</st> <st c="28770">doesn’t offer any flag or environment variable to enrich the verbose logs in the standard output.</st> <st c="28868">If we need to troubleshoot PAC CLI commands, we can</st> <st c="28919">only use the log available under</st> <st c="28953">the</st> <st c="28956">following:</st>

    ```

    <st c="28967">%userprofile%\.dotnet\tools\.store\microsoft.powerapps.cli.tool\1.30.7\microsoft.powerapps.cli.tool\1.30.7\tools\net6.0\any\logs\pac-log.txt</st>

    ```

			<st c="29108">Now, we are fully prepared to make some changes to the Kudos application.</st> <st c="29183">Let’s understand the existing GitHub workflows and their roles in the</st> <st c="29253">DevOps processes.</st>
			<st c="29270">Exercise – building CD pipelines and a release train</st>
			<st c="29323">In the GitHub repository, we will find the following prebuilt GitHub workflows to manage the life cycle</st> <st c="29427">of our development project, end</st> <st c="29460">to end:</st>

				1.  `<st c="29525">.github/workflows/setup-dev-environment.yml</st>`<st c="29568">, this workflow creates a developer branch with the name provided before the workflow execution (the branch name follows the naming rule,</st> `<st c="29706">dev/branch_name</st>`<st c="29721">).</st> <st c="29725">The pipeline spins up a Power Platform developer environment with the same name used for the branch.</st> <st c="29826">We need to add the work account that we use in the Power Platform tenant to create the developer environment on behalf of our account.</st> <st c="29961">At the end of the workflow, we can see</st> <st c="29999">additional steps that grant</st> **<st c="30028">System Administrator rights</st>** <st c="30055">to our work account and to the service principal that we created for the production environment earlier.</st> <st c="30161">Let’s execute this workflow with the</st> <st c="30198">following parameters:</st>

			![Figure 8.12 – Workflow inputs of “Setup dev environment”](img/B22208_08_12.jpg)

			<st c="30366">Figure 8.12 – Workflow inputs of “Setup dev environment”</st>
			<st c="30422">After a successful run, we will see a new Power Platform developer environment with the name</st> `<st c="30516">DEV-US_XXX_Z</st>` <st c="30528">and a branch with the</st> <st c="30551">name</st> `<st c="30556">dev/DEV-US_XXX_Z</st>`<st c="30572">.</st>

				1.  `<st c="30861">dev</st>` <st c="30864">prefix.</st> <st c="30873">In our case, it is</st> `<st c="30892">dev/DEV-US_XXX_Z</st>`<st c="30908">. We can create the GitHub environment by opening the</st> `<st c="31204">DATAVERSE_CONNECTION_ID</st>`2.  `<st c="31228">O365_CONNECTION_ID</st>`3.  `<st c="31247">OUTLOOK_CONNECTION_ID</st>`

<st c="31269">We can see how we’ve introduced the variables in the</st> <st c="31323">following screenshot:</st>

			![Figure 8.13 – The GitHub environment with connection IDs](img/B22208_08_13.jpg)

			<st c="31808">Figure 8.13 – The GitHub environment with connection IDs</st>
			<st c="31864">As a next step, we need to create these three</st> `<st c="32003">KudosSPN</st>`<st c="32012">), and copy the connection IDs from the appropriate URLs into the</st> `<st c="32079">deploymentSettings.json</st>` <st c="32102">file.</st> <st c="32109">Our workflow expects to find this information in the assigned GitHub environment with the matching name.</st> <st c="32214">Let’s execute the</st> *<st c="32232">Import to dev</st>* <st c="32245">workflow located under</st> `<st c="32269">github/workflows/import-to-dev.yml</st>` <st c="32303">from the dev</st> <st c="32316">branch, to import the</st> **<st c="32339">unmanaged solutions</st>** <st c="32358">to the</st> <st c="32366">developer environment:</st>
			![Figure 8.14 – The Import to dev workflow](img/B22208_08_14.jpg)

			<st c="33124">Figure 8.14 – The Import to dev workflow</st>
			<st c="33164">We need to select</st> <st c="33182">our recently created developer branch to use its name to find the right environment in the Power</st> <st c="33280">Platform tenant:</st>
			![Figure 8.15 – Executing the workflow on the dev branch](img/B22208_08_15.jpg)

			<st c="33342">Figure 8.15 – Executing the workflow on the dev branch</st>
			<st c="33396">When everything is set up right according to the previous steps, we will see the GitHub workflow running during the execution of the job.</st> <st c="33535">The</st> `<st c="33539">import-to-dev</st>` <st c="33552">job will run in the</st> `<st c="33573">dev/DEV-US_XXX_Z</st>` <st c="33589">environment, displaying the GitHub environment name in the job’s rectangular box of the workflow run (</st>`<st c="33692">Import to dev</st>`<st c="33706">), as shown in the</st> <st c="33726">following figure:</st>
			![Figure 8.16 – The GitHub workflow import-to-dev job](img/B22208_08_16.jpg)

			<st c="33923">Figure 8.16 – The GitHub workflow import-to-dev job</st>
			<st c="33974">After the successful run, we have only one task</st> <st c="34022">left, which is to turn on the</st> **<st c="34053">Power Automate cloud flow</st>**<st c="34078">,</st> *<st c="34080">Kudos App – Notification email</st>*<st c="34110">, by opening the Kudos solution and viewing the flow under the</st> **<st c="34173">Cloud flows</st>** <st c="34184">solution asset.</st> <st c="34201">We will not see the flows under</st> **<st c="34233">My flows</st>** <st c="34241">because these flows are intentionally not shared</st> <st c="34291">with us.</st>
			<st c="34299">When we examine</st> <st c="34315">the YML file of the workflow in detail, we can see that this pipeline uses a</st> `<st c="34606">deploymentSettings.json</st>` <st c="34629">file with the right connection IDs.</st> <st c="34666">Finally, we execute the</st> `<st c="34690">microsoft/powerplatform-actions/import-solution@v1</st>` <st c="34740">action to import the unmanaged solutions with the right deployment configurations to our Power Platform environment.</st> <st c="34858">We also</st> <st c="34865">import data to the</st> `<st c="34899">Badge</st>`<st c="34904">, with the help of the</st> `<st c="34927">microsoft/powerplatform-actions/import-data@v1</st>` <st c="34973">GitHub action.</st>
			*<st c="34988">The branch name is the glue that ties the Power Platform environment, the Git branch, and the GitHub environment together to help developers easily find their</st>* *<st c="35148">own environments/configurations.</st>*

				1.  `<st c="35238">\.github\workflows\commit-to-dev.yml</st>`<st c="35274">, this workflow has been built to easily commit the changes made in the Power Platform developer environment back to the developer branch.</st> <st c="35413">We can make changes by going directly</st> <st c="35450">to the Kudos solution in the</st> **<st c="35480">maker portal</st>** <st c="35492">and opening the canvas app, (the Kudos app) for editing.</st> <st c="35550">On the main screen of the applica</st><st c="35583">tion, let’s change the textbox to include the current date, as shown in the</st> <st c="35660">following figure:</st>

			![Figure 8.17 – Editing the Kudos App in the developer environment](img/B22208_08_17.jpg)

			<st c="36216">Figure 8.17 – Editing the Kudos App in the developer environment</st>
			<st c="36280">We need to save the app</st> <st c="36304">and publish the customizations before executing the</st> *<st c="36357">Commit to dev branch</st>* <st c="36377">GitHub workflow.</st> <st c="36395">If we want to run the app locally in this environment, we also need to share the application within the solution with our account.</st> <st c="36526">The</st> *<st c="36530">Commit to dev branch</st>* <st c="36550">expects no input parameter; we only need to set the branch to our developer branch.</st> <st c="36635">If we started the workflow directly on the main branch, it would fail because of the branch protection rules that are applied.</st> <st c="36762">Under the hood, this workflow exports both solutions as managed and unmanaged (</st>`<st c="36841">microsoft/powerplatform-actions/export-solution@v1</st>`<st c="36892">) and also unpacks them to the right folders in the developer branch (</st>`<st c="36963">microsoft/powerplatform-actions/unpack-solution@v1</st>`<st c="37014">).</st> <st c="37018">Finally, the flow commits the changes to the dev branch.</st> <st c="37075">As we did earlier, we use the branch name to find the</st> **<st c="37129">Power Platform environment URL</st>** <st c="37159">to export the solutions from the right Dataverse instance, with the help of the following Bash</st> <st c="37255">script snippet:</st>

```

ref=${{ github.ref }}

branch="${ref#refs/heads/dev/}"

echo "$branch"

# 工作流在 dev 分支上执行，所以我们需要从 dev 分支获取环境 URL <st c="37439">rawOutput=$(pac admin list --name $branch | tail -n 2)</st>

<st c="37493">environmentURL=$(echo $rawOutput | cut -d ' ' -f 3)</st> echo "Environment URL: $environmentURL"

# 设置 env.devEnvironmentURL

echo "devEnvironmentURL=$environmentURL" >> "$GITHUB_ENV"

```

			<st c="37679">In the final line, we create</st> <st c="37708">an environment variable with the value of the URL to use in the upcoming actions within the GitHub job.</st> <st c="37813">We hand over this URL to other jobs in the workflow by using the</st> `<st c="37878">outputs</st>` <st c="37885">and</st> `<st c="37890">needs</st>` <st c="37895">keywords in the</st> <st c="37912">YML file.</st>

				1.  `<st c="38197">dev/DEV-US_XXX_Z</st>` <st c="38213">branch to</st> `<st c="38224">main</st>`<st c="38228">. The pull request will compare the two branches and list every change that has been made since the creation of the developer branch.</st> <st c="38362">We can use</st> **<st c="38373">GitHub Copilot</st>** <st c="38387">to generate a summary</st> <st c="38409">of these changes by clicking the Copilot icon on the pull</st> <st c="38468">request page:</st>

			![Figure 8.18 – A Copilot-generated pull request summary](img/B22208_08_18.jpg)

			<st c="38863">Figure 8.18 – A Copilot-generated pull request summary</st>
			<st c="38917">By clicking the</st> **<st c="38934">Merge</st>** <st c="38939">button, our pull request</st> <st c="38964">will close and the changes will merge</st> <st c="39002">back to the parent branch.</st> <st c="39030">Usually, we delete developer branches after a successful pull request, but this time, let’s keep our dev branch to execute some additional exercises in the</st> <st c="39186">upcoming sections.</st>

				1.  `<st c="39263">\.github\workflows\cd-to-prod.yml</st>`<st c="39296">, this workflow deploys the latest version of the main branch to the production environment, as we discussed in the previous section.</st> <st c="39430">We can execute this workflow to deploy the latest version of the Kudos app to the</st> <st c="39512">production environment.</st>

			<st c="39535">In</st> *<st c="39539">Chapter 6</st>*<st c="39548">, we learned about the</st> `<st c="39707">0.0.0.0.</st>` <st c="39715">in Azure DevOps Services during the development phase.</st> <st c="39771">We can apply the same concept in GitHub by introducing a reusable workflow located under</st> `<st c="39860">\.github\workflows\set-version-number.yml</st>`<st c="39901">. This workflow has two parameters; one is the source folder of our solutions, and the other is the version number to be set.</st> <st c="40027">The workflow checks out the repository, searches for the</st> `<st c="40084">Solution.xml</st>` <st c="40096">file under the source folder, and replaces the inline version tags with the version number, provided as an</st> <st c="40204">input parameter:</st>

```

- shell: bash

run: |

# 在解决方案目录及其子目录中查找所有 Solution.xml 文件 <st c="40321">find</st> ${{ inputs.source_folder }} -type f -name "Solution.xml" | while read -r file; do

# 使用输入的版本号替换<Version>标签中的内容 <st c="40493">sed</st> -i 's|<Version>.*</Version>|<Version>${{ inputs.version_number }}</Version>|g' "$file"

    done

```

			<st c="40588">The final action in the flow commits</st> <st c="40625">back the changes to the branch on which the</st> <st c="40670">workflow runs.</st>
			<st c="40684">We need to add this reusable workflow to the</st> *<st c="40730">Commit to dev branch</st>* <st c="40750">workflow to set the version number of the solutions to</st> `<st c="40806">0.0.0.0</st>`<st c="40813">. Let’s update the workflow in the dev branch (</st>`<st c="40860">dev/DEV-US_XXX_Z</st>`<st c="40877">) with the following lines, directly adding them to the end of</st> <st c="40941">the workflow:</st>

```

set-version-number:

    needs: [ commit-to-dev-kudos ]

    name: 将版本号设置为 0.0.0.0

    uses: jovadker/ppdemo/.github/workflows/set-version-number.yml@main

    with:

        source_folder: src/

        version_number: "0.0.0.0"

```

			<st c="41166">Let’s commit the changes locally and push</st> <st c="41208">them back to the remote repository.</st> <st c="41245">We can work in</st> `<st c="41465">dev/DEV-US_XXX_Z</st>`<st c="41482">) and create</st> <st c="41496">another codespace:</st>
			![Figure 8.19 – A GitHub codespace on the dev branch](img/B22208_08_19.jpg)

			<st c="42312">Figure 8.19 – A GitHub codespace on the dev branch</st>
			<st c="42362">After launching the newly created codespace, we can add the workflow snippet (the GitHub job with the name</st> `<st c="42470">set-version-number</st>`<st c="42488">) to the end of the file, as shown in the</st> <st c="42531">following figure:</st>
			![Figure 8.20 – The commit-to-dev.yml file in the GitHub codespace](img/B22208_08_20.jpg)

			<st c="43595">Figure 8.20 – The commit-to-dev.yml file in the GitHub codespace</st>
			<st c="43659">After saving the file in the VS Code editor</st> <st c="43703">in the browser, we can navigate to the source control icon on the left menu and commit our changes, by providing a commit message and clicking on the</st> **<st c="43854">Commit</st>** <st c="43860">button:</st>
			![Figure 8.21 – A Git commit in a GitHub codespace](img/B22208_08_21.jpg)

			<st c="44605">Figure 8.21 – A Git commit in a GitHub codespace</st>
			<st c="44653">After clicking on the</st> **<st c="44676">Commit</st>** <st c="44682">button, we should not forget to click on</st> **<st c="44724">Sync Changes</st>** <st c="44736">to push back the changes to the</st> <st c="44769">remote origin.</st>
			<st c="44783">To test the GitHub workflow</st> <st c="44811">upon our changes, we can start it on the branch,</st> `<st c="44861">dev/DEV-US_XXX_Z</st>`<st c="44877">, and see how the two solutions and their</st> `<st c="44919">Solution.xml</st>` <st c="44931">files are updated.</st> <st c="44951">Similarly, we can introduce this job to other flows if we plan to maintain a homogenous versioning in</st> <st c="45053">our solutions.</st>
			<st c="45067">Version number – 0.0.0.0</st>
			<st c="45092">If we use the</st> `<st c="45107">0.0.0.0</st>` <st c="45114">version number, then the solutions imported to Power Platform developer environments will also have this version.</st> <st c="45229">This approach is ideal for multiple developers working on the same solution or project, since they will not override the versions by committing their changes back to developer branches and later, through pull requests, to the</st> <st c="45455">main branch.</st>
			<st c="45467">Finally, we can leverage the releases feature of GitHub to publish</st> <st c="45534">our new versions.</st> **<st c="45553">GitHub releases</st>** <st c="45568">offer an easy way to package our software, along with release notes and links to binary files, for other people to use.</st> <st c="45689">We can manage these releases in GitHub workflows; the platform provides a special trigger that we can use to add our solutions to a release, and there</st> <st c="45839">are actions available in the</st> **<st c="45869">GitHub marketplace</st>** <st c="45887">to create releases within workflows.</st> <st c="45925">We will create our release by executing a new GitHub workflow that builds</st> <st c="45998">our solutions, using</st> **<st c="46020">MSBuild</st>**<st c="46027">, creates a deployment package, and publishes the generated artifacts as part of the new</st> <st c="46116">release version.</st>
			<st c="46132">The Kudos application provides</st> `<st c="46262">MSBuild</st>` <st c="46269">and</st> `<st c="46274">dotnet</st>` <st c="46280">CLIs.</st> <st c="46287">We can create these</st> `<st c="46307">.cdsproj files</st>` <st c="46321">at any time by executing the following PAC CLI command in the</st> `<st c="46384">solution</st>` <st c="46392">folder:</st>

```

pac solution init --publisher-name developer --publisher-prefix dev

```

			<st c="46468">This command creates a wrapper around our solution, if empty, and then it will create</st> <st c="46555">an empty solution under the</st> `<st c="46583">src</st>` <st c="46586">folder, into which we can copy our solution files (XML and</st> <st c="46646">JSON files).</st>
			<st c="46658">In</st> *<st c="46662">Chapter 4</st>*<st c="46671">, we learned about</st> <st c="46689">deployment packages and the</st> `<st c="46923">.csproj</st>` <st c="46930">file under the</st> `<st c="46946">DeploymentPackage</st>` <st c="46963">folder:</st>

```

pac package init --outputDirectory DeploymentPackage

cd .\DeploymentPackage <st c="47048">pac package add-solution --path <<PATHTORELEASE>>\mpa_EmployeeExperienceBase_managed.zip</st>

<st c="47252">pac package add-solution</st> 命令将之前由 <st c="47342">MSBuild</st> 构建的托管解决方案作为引用添加到此 <st c="47358">.csproj</st> 文件中。设置部署包后，我们可以使用 <st c="47442">dotnet publish -c Release</st> 命令来构建部署包的发布版本。

            <st c="47531">部署包</st>

            <st c="47554">我们在 GitHub 仓库中创建了一个文件夹结构，其中两个解决方案和</st> `<st c="47647">DeploymentPackage</st>` <st c="47664">文件夹位于</st> `<st c="47693">src</st>` <st c="47696">文件夹下。</st> <st c="47705">在</st> `<st c="47709">csproj</st>` <st c="47715">文件中，</st> `<st c="47728">DeploymentPackage</st>` <st c="47745">文件夹引用了两个解决方案的发布构建，分别是 Employee Experience Base</st> <st c="47830">和 Kudos。</st>

            <st c="47840">最后，我们可以通过使用</st> *<st c="47896">GitHub Release</st>* <st c="47910">工作流来创建我们的 GitHub 发布，该工作流位于</st> `<st c="47935">/.github/workflows/create-release.yml</st>`<st c="47972">。该工作流的高层步骤和关键要点如下：</st>

                1.  `<st c="48376">/.github/actions/set-version-number-action/action.yml</st>`<st c="48429">，它设置了 Power Platform 解决方案的版本号。</st> <st c="48491">正如我们之前所看到的，可重用的工作流需要作为作业运行，这意味着 GitHub 运行器将在完成可重用工作流作业并继续下一个作业后清理本地仓库。</st> <st c="48691">如果我们不希望提交版本号，我们需要将所有操作都运行在同一个作业中。</st> <st c="48787">这就是为什么我们创建了这个</st> <st c="48819">复合操作。</st>

                1.  `<st c="48932">MSBuild</st>` <st c="48939">和</st> `<st c="48944">dotnet</st>` <st c="48950">CLI 可用于我们的构建操作。</st> <st c="48989">我们使用以下命令构建解决方案：</st>

    ```
    <st c="49098">proj</st> file contains the references to the <st c="49139">cdsproj</st> files under the <st c="49163">solutions</st> folders. We generate the deployment package with the <st c="49226">dotnet publish -c Release /p:Version=${{inputs.release_version}}</st> command by setting the version of the package to the one provided as the workflow parameter. After having the binaries generated, we upload every build artifact to the GitHub artifact store.
    ```

                    1.  `<st c="49610">gh cli</st>` <st c="49616">命令用于创建一个</st> <st c="49637">GitHub 发布：</st>

    ```
     gh release create ${{inputs.release_version}} --title "${{inputs.release_title}}" --generate-notes ${{ env.solution_release_folder}}/*.*
    ```

    <st c="49789">通过使用</st> `<st c="49803">generate-notes</st>` <st c="49817">参数，GitHub 发布说明将自动生成。</st> <st c="49886">我们还通过引用文件夹及其内容将二进制文件附加到创建的发布中，</st> `<st c="49981">${{</st>` `<st c="49985">env.solution_release_folder}}/*.*</st>`<st c="50018">。</st>

            <st c="50019">现在，让我们使用默认参数执行此流程，并在完成后检查是否能在 GitHub 仓库的主页上看到</st> **<st c="50172">Releases</st>**<st c="50180">下的一个名为</st> **<st c="50199">Initial release</st>**<st c="50214">的发布。打开它后，我们应该能发现关于第一次发布的更多细节——例如，</st> <st c="50306">附加的资产：</st>

            ![图 8.22 – 包含 Power Platform 包的 GitHub 发布](img/B22208_08_22.jpg)

            <st c="50590">图 8.22 – 包含 Power Platform 包的 GitHub 发布</st>

            <st c="50654">包部署器</st>

            <st c="50675">该</st> `<st c="50680">.pdpkg</st>` <st c="50686">（</st> `<st c="50899">pac package deploy --package</st>` `<st c="50928">.\bin\Release\mpa_Kudos_DeploymentPackage.1.0.0.pdpkg.zip</st>` <st c="50985">在另一个租户上进行部署。</st> <st c="51007">这些包</st> <st c="51021">还可以上传数据并为</st> <st c="51082">解决方案准备目标环境。</st>

            <st c="51096">通过 GitHub 发布，我们可以</st> <st c="51125">将我们的开发项目结果分发给负责生产租户的 IT 运维团队，而无需直接将我们的开发租户和工作流与</st> <st c="51296">生产租户连接。</st>

            <st c="51311">如果在尝试运行</st> <st c="51358">工作流时遇到问题，请考虑以下</st> <st c="51397">故障排除选项：</st>

                +   **<st c="51421">检查运行日志</st>**<st c="51440">：第一步是检查工作流运行的日志。</st> <st c="51500">GitHub 为每个工作流步骤提供详细日志，这可以帮助我们识别错误发生的地方。</st>

                +   `<st c="51699">ACTIONS_STEP_DEBUG</st>` <st c="51717">密钥用于启用步骤调试日志，提供每个步骤的更详细输出。</st> <st c="51791">步骤日志的详细输出可以帮助我们诊断问题。</st>

                +   **<st c="51801">在本地运行工作流</st>**<st c="51826">：我们可以使用诸如</st> **<st c="51854">act</st>** <st c="51857">(</st>[<st c="51859">https://github.com/nektos/act</st>](https://github.com/nektos/act)<st c="51888">)等工具在本地机器上运行工作流。</st> <st c="51936">这可以帮助我们在受控的环境中调试工作流。</st>

                +   `<st c="52045">.yml</st>` <st c="52049">工作流文件格式正确，且所有必需字段都已包含。</st> <st c="52129">语法错误或缺失字段可能导致工作流失败。</st> <st c="52190">我们需要非常小心地更改 YML 文件中的行缩进，因为仅一个额外的空格就可能导致语法错误。</st>

                +   **<st c="52320">检查外部更改</st>**<st c="52347">：有时，外部依赖项或环境的变化会导致工作流失败。</st> <st c="52441">我们需要确保所有外部服务和依赖项</st> <st c="52503">都是正常运行的。</st>

            <st c="52519">现在，让我们进入下一个话题，深入了解</st> <st c="52572">待办事项管理。</st>

            <st c="52591">练习 – GitHub 中的待办事项管理</st>

            <st c="52631">在</st> *<st c="52635">第一章</st>*<st c="52644">中，我们了解了为何跟踪我们的活动、用户故事、变更请求和 bug 修复在任何代码库中都是至关重要的。</st> <st c="52771">让我们回顾一下</st> <st c="52788">关键要点：</st>

                +   <st c="52802">我们希望在每个冲刺前规划好开发人员的工作</st> <st c="52859">以便集中精力</st> <st c="52874">处理最关键的</st> <st c="52881">功能</st> **<st c="52896">特性</st>**<st c="52904">,</st> **<st c="52906">缺陷</st>**<st c="52910">和</st> **<st c="52916">问题</st>**<st c="52923">。整个</st> **<st c="52936">冲刺规划过程</st>** <st c="52959">基于一个健康的</st> **<st c="52982">产品待办事项</st>** <st c="52997">，每个开发人员</st> <st c="53018">和产品负责人</st> <st c="53036">都需要</st> <st c="53043">维护它。</st>

                +   <st c="53055">健康的待办事项管理只允许计划中的源代码更改</st> <st c="53114">，以避免</st> **<st c="53141">黄金镶嵌</st>** <st c="53153">(开发人员添加不属于活动范围的额外功能) 和</st> **<st c="53227">范围蔓延</st>** <st c="53241">(当项目团队在不调整项目成本或时间表的情况下，处理客户请求的功能)</st> <st c="53280">。</st>

                +   **<st c="53366">待办事项管理</st>** <st c="53385">提供</st> **<st c="53395">向后追溯性</st>** <st c="53416">以及追溯</st> <st c="53448">从生产环境中运行的应用程序到生成该应用程序的源代码</st> <st c="53510">的能力。</st> **<st c="53534">二进制文件</st>** <st c="53542">的生成。</st> <st c="53563">我们可以利用这些信息进行根本原因分析，发现哪个工作项跟踪的更改导致了生产环境中的问题。</st>

            <st c="53693">Azure DevOps 服务</st> <st c="53720">和 GitHub 提供了这些需求工程</st> <st c="53769">和问题管理功能，具备先进的项目管理特性，例如安排我们的活动并分配给团队成员</st> <st c="53917">在冲刺中。</st>

            <st c="53928">为了保持 GitHub 中健康的待办事项管理，我们可以</st> <st c="53985">改进最小</st> `<st c="54251">pull_request</st>` <st c="54263">操作，目标是 main 分支并执行以下操作：</st>

            ![图 8.23 – 触发拉取请求的工作流，用于分支保护](img/B22208_08_23.jpg)

            <st c="54475">图 8.23 – 触发拉取请求的工作流，用于分支保护</st>

            <st c="54544">我们的工作流关键部分如下所示（位于</st> <st c="54600">`/.github/workflows/pr-check.yml`</st> <st c="54637">）：</st>

```
<st c="54640">name: Pull request check</st>
<st c="54665">on:</st>
 <st c="54669">pull_request:</st>
 <st c="54683">types: [edited, synchronize, opened, reopened]</st>
 <st c="54730">branches: [ "main" ]</st> jobs:
  prcheck:
    runs-on: ubuntu-latest
    steps:
      - name: Check for comments in PR <st c="54830">id: check-comments</st> run: |
          # every pull request is an issue as well
          #- we can address them through /issues/ endpoint
          comments=$(<st c="54957">curl</st> -s -H "Authorization: token ${{secrets.GITHUB_TOKEN}}" \
             "<st c="55021">https://api.github.com/repos/${{ github.repository }}/issues/${{ github.event.pull_request.number }}/comments")</st> if [ $(echo "$comments" | jq '. | length') -eq 0 ]; then
            echo "There is no comment added to the PR." <st c="55235">echo "no_comments=true" >> $GITHUB_OUTPUT</st> else
            echo "Comments are added to the PR." <st c="55319">echo "no_comments=false" >> $GITHUB_OUTPUT</st> fi
        shell: bash
      - name: Fail if no comments
        run: |
          if [[ <st c="55418">"${{ steps.check-comments.outputs.no_comments }}</st>" == "true" ]]; then
           echo "No comments added to the pull request. Failing the build." exit 1
          fi
        shell: bash
```

            <st c="55574">我们使用</st> `<st c="55582">curl</st>` <st c="55586">通过 REST API 端点查询属于该问题的评论</st> <st c="55608">（</st><st c="55631">https://api.github.com/repos/${{ github.repository }}/issues/${{ github.event.pull_request.number }}/comments</st> <st c="55769">），因为每个拉取请求也被建模为一个</st> `<st c="55910">no_comments</st>`<st c="55921">，因此。</st> <st c="55936">下一步操作会使用前一步操作的输出，判断构建是否通过。</st> <st c="56026">如果工作流失败，拉取请求将被阻止，无法合并。</st> <st c="56108">工作流还会在开始时检查拉取请求是否添加了描述，采用相同的方法。</st> <st c="56231">我们可以通过回溯写描述并添加评论来修正我们的拉取请求。</st> <st c="56321">此外，我们还可以使用 GitHub Copilot 根据子分支和</st> <st c="56438">父分支之间的更改生成拉取请求描述：</st>

            ![图 8.24 – 使用 GitHub Copilot 生成 PR 描述](img/B22208_08_24.jpg)

            <st c="57005">图 8.24 – 使用 GitHub Copilot 生成 PR 描述</st>

            <st c="57067">我们只需要点击</st> <st c="57089">Copilot</st> <st c="57104">图标，然后底层的</st> **<st c="57135">GPT-4 模型</st>** <st c="57146">会生成拉取请求的摘要</st> <st c="57168">——在我们的案例中，</st> **<st c="57205">该拉取请求对</st>** **<st c="57253">Readme.md 文件进行了微小的更改……</st>**<st c="57268">。</st>

            <st c="57269">如果我们想要在拉取请求中引入更复杂的检查，以配合待办事项管理</st>，我们可以访问<st c="57365">这个</st> `<st c="57439">verify-linked-issue</st>` <st c="57458">（</st>[<st c="57460">https://github.com/marketplace/actions/verify-linked-issue</st>](https://github.com/marketplace/actions/verify-linked-issue)<st c="57518">），它会检查拉取请求是否至少关联了一个</st> <st c="57582">问题。</st>

            <st c="57597">现在我们已经建立了严格的仓库和工作管理控制，接下来我们进入下一个话题，</st> **<st c="57714">质量保证</st>**<st c="57731">。</st>

            <st c="57732">练习 – 测试解决方案</st>

            <st c="57761">在</st> *<st c="57765">第六章</st>*<st c="57774">中，我们深入探讨了</st> **<st c="57795">质量保证</st>** <st c="57812">(</st>**<st c="57814">QA</st>**<st c="57816">) 主题，并了解了</st> <st c="57837">可用于端到端 UI 测试的工具和框架</st> <st c="57868">，例如</st> **<st c="57936">Power Apps 测试引擎</st>**<st c="57958">，或开源的</st> <st c="57968">Web</st> <st c="57974">测试框架，如</st> **<st c="58007">Selenium</st>**<st c="58015">、</st> **<st c="58017">Playwright</st>**<st c="58027">、</st> **<st c="58029">Appium</st>** <st c="58035">或</st> **<st c="58039">Cypress</st>**<st c="58046">。我们还得出结论，Power Automate 云流和</st> **<st c="58102">桌面流</st>** <st c="58115">被视为我们的业务逻辑层，我们可以通过 UI 组件进行端到端测试。</st> <st c="58226">在本节中，我们将进行以下操作：</st>

                +   <st c="58276">在我们的 Power Platform 开发环境中，为 Kudos 应用创建一个 Power Apps 测试工作室的测试</st> <st c="58367">（</st>`<st c="58380">DEV-US_XXX_Z</st>`<st c="58393">）。</st>

                +   <st c="58396">将其作为测试套件下载</st> **<st c="58425">YAML 文件</st>** <st c="58434">并提交到我们的开发</st> <st c="58465">分支（记得这个分支</st> <st c="58503">仍然存在）。</st>

                +   <st c="58516">在本地运行，借助</st> <st c="58553">PAC CLI。</st>

                +   <st c="58561">将这一步引入到我们的</st> *<st c="58589">提交到开发分支</st>* <st c="58609">GitHub 工作流中。</st>

            <st c="58626">我们还有一些先决条件</st> <st c="58658">用于这个测试</st> <st c="58673">自动化场景：</st>

                +   <st c="58693">为了能够在 CI/CD 过程中执行我们的测试，我们还</st> <st c="58758">需要一个没有</st> **<st c="58787">多因素认证</st>** <st c="58813">(</st>**<st c="58815">MFA</st>**<st c="58818">) 的用户在我们的开发租户中；可以通过</st> *<st c="58887">进一步阅读</st>* <st c="58902">部分中的链接了解更多信息（Power Apps</st> <st c="58923">测试引擎）。</st>

                +   <st c="58936">我们还需要在</st> **<st c="58983">解决方案</st>** <st c="58992">页面下共享 Kudos 应用</st> **<st c="59009">Microsoft Entra</st>** **<st c="59025">ID 用户</st>**<st c="59032">。</st>

                +   我们必须将此用户添加到开发环境（`<st c="59086">DEV-US_XXX_Z</st>`）中，**<st c="59109">Power Platform 管理中心</st>**，并且我们需要为该用户分配内置的**<st c="59173">安全角色</st>**，**<st c="59188">基本用户</st>**和自定义角色**<st c="59220">Kudos 员工</st>**，这样才能访问由 Kudos 解决方案创建的自定义表格，其中包含徽章以及已经<st c="59378">共享的 Kudos。</st>

                +   我们需要代表测试用户第一次交互式启动该应用，且不启用 MFA，授予他们对 Kudos 应用、Office 365 用户和 Office 365 Outlook 中使用的连接的访问权限。</st>

                +   最后，我们需要与该用户共享*<st c="59611">Kudo 应用 – 与发送者共享 Kudo</st>*<st c="59644">，</st>*<st c="59646">分配给接收者</st>*<st c="59665">，以及</st>*<st c="59671">Kudos 应用 - 通知邮件</st>*<st c="59701">云流程，作为</st>*<st c="59732">仅限运行的用户</st>*<st c="59746">在 Power</st><st c="59759">Automate 云</st><st c="59775">流程 UI 中。</st>

            我们可以通过使用<st c="59783">`/test/SmokeTestSuite.yaml`</st>文件来轻松记录<st c="59804">我们的测试，继续进行这项练习。</st>

            一旦我们有了 YAML 文件，我们需要进行一些<st c="60571">更改，以便能够在</st>`<st c="60924">0x0</st>`<st c="60927">中运行，像是</st>`<st c="60937">102x768</st>`<st c="60944">像素（</st>`<st c="60953">screenWidth</st>` `<st c="60966">X screenHeight</st>`<st c="60980">）：</st>

```
 testSettings:
  filePath:
  browserConfigurations:
  - browser: Chromium
    device: <st c="61059">screenWidth: 1024</st><st c="61076">screenHeight: 768</st> locale: en-US <st c="61109">recordVideo: true</st><st c="61126">headless: true</st> enablePowerFxOverlay: false
  timeout: 30000
```

            除了这些更新之外，我们可以将<st c="61184">`headless`</st> <st c="61219">参数设置为</st>`<st c="61241">false</st>`<st c="61246">，以便在 Chromium 浏览器中进行本地测试，跟踪 UI 操作。</st><st c="61328">要在本地执行此测试 YAML 文件，我们可以使用以下 Bash 脚本：</st>

```
 export user1Email="USEREMAIL"
export user1Password="PASSWORD"
pac test run --test-plan-file ./test/SmokeTestSuite.yaml -env 4d3c1075-FFFF-GGGG-VVVV-40c6f5edd705 --tenant 4ae51f31-XXXX-YYYY-ZZZZ-5ece14d2c081
```

            我们需要将测试用户的电子邮件地址和密码设置为环境变量，并且还需要提供测试文件位置、环境 ID 和租户 ID。</st><st c="61785">成功执行后，我们将在</st>`<st c="61850">TestOutput</st>`<st c="61860">文件夹中找到测试结果及视频录制，文件格式也将是</st>`<st c="61909">.webm</st>`<st c="61915">。</st>

            <st c="61923">要在 GitHub 中执行此测试，我们需要</st> <st c="61963">创建两个额外的</st> `<st c="62092">TESTUSER</st>` <st c="62100">和</st> `<st c="62105">TESTUSERPSW</st>`<st c="62116">，并且 – 与我们在本章开始时创建的其他三个（</st>`<st c="62157">PPAPPID</st>`<st c="62165">,</st> `<st c="62167">PPAPPSECRET</st>`<st c="62178">, 和</st> `<st c="62184">PPTENANTID</st>`<st c="62194">）类似。</st>

            <st c="62246">我们提前创建了一个 GitHub 工作流，以便在开发分支上轻松执行我们的</st> `<st c="62307">SmokeTestSuite.yaml</st>` <st c="62326">文件。</st> <st c="62358">此工作流位于</st> `<st c="62391">/.github/workflows/run-test.yml</st>` <st c="62422">并使用以下 Bash 脚本运行</st> <st c="62465">测试：</st>

```
 - name: Run test <st c="62492">shell: bash</st><st c="62503">run: |</st> set -e <st c="62518">ref=${{ github.ref }}</st> branch="${ref#refs/heads/dev/}"
       echo "$branch"
       # Get the environment Id <st c="62612">rawOutput=$(pac admin list --name $branch | tail -n 2)</st> environmentId=$(echo $rawOutput | cut -d ' ' -f 2) <st c="62718">export user1Email="${{secrets.TESTUSER}}</st>" <st c="62761">export user1Password="${{secrets.TESTUSERPSW}}</st>" <st c="63003">DEV-US_XXX_Z</st>). Based on the branch name, the workflow finds our developer environment and then calls <st c="63106">pac test run</st> with the appropriate parameters.
			<st c="63151">To avoid feature regression and maintain</st> <st c="63192">the high quality of our solution, we can introduce this step in our</st> *<st c="63261">Commit to dev branch</st>* <st c="63281">GitHub workflow to fail fast and early in the development process.</st> <st c="63349">All we need to do is to append the entire job, called</st> `<st c="63403">test</st>`<st c="63407">, from the</st> `<st c="63418">/.github/workflows/run-test.yml</st>` <st c="63449">as the first job in the workflow, as we want to only allow new commits landing in the branch when our automated</st> <st c="63562">tests pass:</st>
			![Figure 8.25 – The Commit to dev branch with a placeholder for “test” job](img/B22208_08_25.jpg)

			<st c="64234">Figure 8.25 – The Commit to dev branch with a placeholder for “test” job</st>
			<st c="64306">Additionally, we can introduce this quality check in our pull request triggered workflow (</st>*<st c="64397">Pull request check</st>*<st c="64416">), in the production workflow (</st>*<st c="64448">Release to Production</st>*<st c="64470">), or even in the release workflow (</st>*<st c="64507">GitHub Release</st>*<st c="64522">) based on our preferences.</st> <st c="64551">Some of these workflows need to be extended, for instance, to be able to spin up new Power Platform environments, deploy the release candidate, and execute the tests.</st> <st c="64718">Only our imagination and project costs can limit our</st> <st c="64771">QA investments.</st>
			<st c="64786">Now, our application is ready</st> <st c="64816">to run in production.</st> <st c="64839">There is only one task left, which is to get real-time insights and telemetry data about our application’s runtime characteristics and behavior.</st> <st c="64984">Let’s discover what monitoring options we have in</st> <st c="65034">Power Platform.</st>
			<st c="65049">Exercise – monitoring the applications</st>
			<st c="65088">After publishing our application</st> <st c="65121">to the production environment, we want to understand how it performs, how users interact with the application, and how far the application is stable and can run without errors.</st> <st c="65299">As</st> <st c="65301">Microsoft Power Platform runs on</st> **<st c="65335">Microsoft Azure</st>**<st c="65350">, it can leverage the existing Azure</st> **<st c="65387">platform-as-a-service</st>** <st c="65408">(</st>**<st c="65410">PaaS</st>**<st c="65414">) services to provide real-time</st> <st c="65446">telemetry data collection and analysis for the Power Platform portfolio – Power Apps, Power Automate, Copilot Studio, Power Pages, and even Dataverse.</st> <st c="65598">Azure’s</st> <st c="65605">PaaS service is</st> **<st c="65622">Azure Application Insights</st>**<st c="65648">, which is</st> <st c="65659">tidily</st> <st c="65665">connected to</st> **<st c="65679">Azure Monitor</st>** <st c="65692">and</st> **<st c="65697">Azure Log Analytics workspaces</st>**<st c="65727">. Azure Application Insights is an</st> **<st c="65762">Application Performance Management</st>** <st c="65796">(</st>**<st c="65798">APM</st>**<st c="65801">) solution that can be used</st> <st c="65829">in live production monitoring scenarios.</st> <st c="65871">Application Insights provides application dashboards, application maps, live metrics, transaction search, availability view, failures view, performance view, monitoring alerts, workbooks, and so on.</st> <st c="66070">With Application Insights, we can also discover the usage patterns of our users, how people interact with the app, and how the churn rate or the conversion rate looks.</st> <st c="66238">It also visualizes the user journey on web applications.</st> <st c="66295">The Application Insights service offers machine learning-based</st> <st c="66357">analysis of telemetry data (called</st> **<st c="66393">Smart Detection</st>**<st c="66408">) to identify anomalies or performance degradation before outages or blackouts occur.</st> <st c="66495">If the built-in detection features are not enough, we can write our own queries to look</st> <st c="66582">for anomalies with the help of</st> **<st c="66614">Kusto Query Language</st>** <st c="66634">(</st>**<st c="66636">KQL</st>**<st c="66639">).</st> <st c="66643">We can introduce our custom alerting and notifications</st> <st c="66697">based on</st> <st c="66706">the</st> <st c="66710">query</st> <st c="66717">results</st> <st c="66724">of KQL scripts that can trigger</st> **<st c="66757">Azure Playbooks</st>**<st c="66772">,</st> **<st c="66774">Azure Logic Apps</st>**<st c="66790">,</st> **<st c="66792">Azure Functions</st>**<st c="66807">,</st> **<st c="66809">Azure EventHub</st>**<st c="66823">, and</st> **<st c="66829">custom Webhooks</st>**<st c="66844">. The custom webhooks can trigger Power Automate cloud flows to react to the anomalies and outriders in the Power Platform.</st> <st c="66968">Last but not least, Application Insights provides SDKs, available</st> <st c="67033">in JavaScript, Java, C#, Node.js, and Python, based on the</st> **<st c="67093">OpenTelemetry framework</st>**<st c="67116">.</st>
			<st c="67117">Since Power Apps are browser-based applications and the web</st> <st c="67177">player that hosts the apps in the browser is based on</st> `<st c="67282">react-native</st>` <st c="67294">in the native mobile apps through</st> **<st c="67329">wrap functionality</st>**<st c="67347">, it is a very straightforward</st> <st c="67377">approach to embrace Application Insights’ capabilities in our low-code/no-code platform.</st> <st c="67467">We can add Application Insights’ endpoint directly to the canvas app – in our case, to the Kudos app.</st> <st c="67569">We just need to edit the app in the developer environment through the Kudos solution and select the</st> **<st c="67669">App</st>** <st c="67672">node in the tree view on the</st> <st c="67702">left side:</st>
			![Figure 8.26 – Application Insights in Power Apps Studio](img/B22208_08_26.jpg)

			<st c="68566">Figure 8.26 – Application Insights in Power Apps Studio</st>
			<st c="68621">On the right side, among the properties</st> <st c="68661">of</st> **<st c="68665">App</st>**<st c="68668">, we will find the</st> **<st c="68687">Instrumentation key</st>** <st c="68706">field, and here, we should provide the instrumentation key of our Application Insights instance.</st> <st c="68804">Let’s create an Application Insights instance in our</st> <st c="68857">Azure subscription:</st>

```

# 交互式登录

az login

# 选择正确的订阅

az account set --subscription baa70448-593c-4dc7-8a91-c92cf7eaf66e

az group create --location westeurope --resource-group KudosApp.AI.RG <st c="69077">workspace=$(az monitor log-analytics workspace create \</st>

<st c="69132">--resource-group KudosApp.AI.RG \</st>

<st c="69166">--workspace-name KudosWorkspace \</st>

<st c="69200">--location westeurope --query id --output tsv)</st>

<st c="69247">az monitor app-insights component create \</st>

<st c="69290">--app KudosAppInsights \</st>

<st c="69315">--location westeurope \</st>

<st c="69339">--workspace $workspace \</st>

<st c="69450">instrumentationKey</st> 密钥。我们现在可以将该密钥添加到 Kudos 应用并试用，查看数据如何被导入到<st c="69574">应用程序</st> <st c="69586">洞察</st> 仪表板。

            <st c="69605">Canvas 应用洞察</st>

            <st c="69625">要查看遥测</st> <st c="69643">信息，我们需要在 Power Platform 管理中心启用 Canvas 应用洞察。</st> <st c="69731">转到</st> **<st c="69737">设置</st>**<st c="69745">，它列出了所有租户设置，然后选择</st> **<st c="69799">Canvas 应用洞察</st>** <st c="69818">项。</st> <st c="69825">在右侧的</st> **<st c="69832">Canvas 应用洞察</st>** <st c="69851">面板中，我们可以开启此</st> <st c="69893">功能。</st>

            <st c="69904">当然，我们不希望直接在画布应用中存储仪表密钥。</st> <st c="69991">使仪表密钥与应用程序独立的最简单方法是引入一个新的环境变量，但在撰写本文时，仪表密钥属性尚不支持此方法。</st> <st c="70201">我们可以做的是更新我们的部署管道，并将仪表密钥替换为正确的值，该值存储为</st> `<st c="70580">json 文件</st>` <st c="70589">在我们的解决方案文件夹中 –</st> `<st c="70615">/src/mpa_Kudos/src/CanvasApps/src/mpa_KudosApp/AppInsightsKey.json</st>`<st c="70681">。这个文件包含了我们可以在</st> <st c="70773">GitHub 工作流中替换为正确的仪表密钥。</st>

            <st c="70789">尽管我们的解决方案中没有自定义聊天机器人，</st> **<st c="70848">Microsoft Copilot Studio</st>** <st c="70872">同样支持这种与 Azure 应用程序洞察的集成</st> <st c="70911">。</st> <st c="70945">我们可以在</st> **<st c="71033">设置</st>** <st c="71041">中配置 Azure 应用程序洞察实例的连接字符串</st> <st c="71049">，位于</st> **<st c="71064">Copilot 详细信息</st>** <st c="71081">菜单下的</st> **<st c="71089">高级</st>** <st c="71089">标签页，如下图所示：</st>

            ![图 8.27 – Microsoft Copilot Studio 中的应用程序洞察](img/B22208_08_27.jpg)

            <st c="72035">图 8.27 – Microsoft Copilot Studio 中的应用程序洞察</st>

            <st c="72097">在这里，我们需要提供</st> <st c="72121">完整的连接字符串，格式如下：</st>

```
 InstrumentationKey=XXXXXXXX-YYYY-YYYY-YYYY-XXXXXXXXXXXX;IngestionEndpoint=https://westeurope-5.in.applicationinsights.azure.com/;LiveEndpoint=https://westeurope.livediagnostics.monitor.azure.com/;ApplicationId=TTTTTTTT-ZZZZ-ZZZZ-ZZZZ-SSSSSSSSSSSS
```

            <st c="72422">我们可以在</st> **<st c="72454">概览</st>** <st c="72462">页找到此字符串，属于我们的 Azure 应用程序</st> <st c="72494">洞察实例。</st>

            <st c="72512">对于</st> **<st c="72528">Power Pages 网站</st>**<st c="72548">，我们需要注入跟踪用户在网站上操作的代码片段，并将遥测数据发送到 Azure 应用程序洞察终端。</st> <st c="72706">我们只需获取文档中提供的客户端 JavaScript 代码片段</st> <st c="72791">并将其作为内容片段添加到我们的</st> **<st c="72831">Power Pages</st>** **<st c="72843">管理</st>** <st c="72853">应用程序中：</st>

            ![图 8.28 – Power Pages 中的应用程序洞察](img/B22208_08_28.jpg)

            <st c="73599">图 8.28 – Power Pages 中的应用程序洞察</st>

            <st c="73648">代码片段本身</st> <st c="73667">可以在此链接找到：</st> [<st c="73689">https://learn.microsoft.com/en-us/azure/azure-monitor/app/javascript-sdk?tabs=javascriptwebsdkloaderscript</st>](https://learn.microsoft.com/en-us/azure/azure-monitor/app/javascript-sdk?tabs=javascriptwebsdkloaderscript)<st c="73795">，只需要更新连接字符串为</st> <st c="73851">我们自己的。</st>

            <st c="73859">除了画布应用、自定义</st> <st c="73887">聊天机器人和 Power Pages 网站外，Azure Application Insights 作为通用的 APM 框架</st> <st c="73978">可用于监控</st> **<st c="74021">模型驱动应用</st>**<st c="74046">、Power Automate 云流，以及</st> **<st c="74080">Dataverse 诊断和性能事件</st>**<st c="74124">。这可以通过使用</st> **<st c="74153">将数据导出到 Application Insights</st>** <st c="74188">功能实现，该功能可在</st> <st c="74223">Power Platform 管理中心中使用，如果我们拥有付费/高级 Dataverse 许可证的话。</st> <st c="74295">对于我们来说，这意味着我们可以创建导出包，将选定环境中的遥测数据推送到我们的 Application Insights 服务，而无需将端点或连接字符串注入到 Power Platform 资产中。</st> <st c="74523">我们无需在解决方案中准备或创建任何内容；环境和此导出作业将负责遥测数据的摄取。</st> <st c="74664">建议每个环境使用一个 Application Insights 实例，并且请注意，此功能仅在托管</st> <st c="74805">环境中开启和支持。</st>

            <st c="74823">遥测数据的延迟摄取</st>

            <st c="74859">在</st> **<st c="74889">服务级别协议</st>** <st c="74912">(</st>**<st c="74914">SLA</st>**<st c="74917">) 中规定的遥测数据流交付时间框架</st> <st c="74962">从支持此功能的 Power Platform 产品到 Application Insights 的交付时间为</st> <st c="75045">24 小时。</st>

            <st c="75054">如果我们想要收集</st> <st c="75076">来自这些 Power Platform 资产的实时遥测数据，我们可以创建自己的扩展，例如</st> <st c="75173">以下内容：</st>

                +   <st c="75187">在模型驱动应用中，我们可以创建一个</st> **<st c="75226">Power Platform 组件框架</st>** <st c="75260">(</st>**<st c="75262">PCF</st>**<st c="75265">) 控件，该控件显示在 UI 中并通过客户端 JavaScript 连接到 Application Insights 端点。</st>

                +   <st c="75382">通过 Power Automate 云流，我们可以使用在 Dataverse 中记录的有关开始时间、持续时间、结束时间、状态（例如失败、取消或成功）和执行操作</st> <st c="75584">的信息，并借助</st> **<st c="75634">Dataverse 插件</st>**<st c="75650">将其发送到 Application Insights 端点。</st> <st c="75690">监控这些流的另一个选项是使用 Power Automate 中新内置的</st> **<st c="75757">自动化中心</st>** <st c="75774">面板：</st>

            ![图 8.29 – Power Automate 自动化中心](img/B22208_08_29.jpg)

            <st c="76747">图 8.29 – Power Automate 自动化中心</st>

            <st c="76797">在这里，我们可以可视化记录在 Dataverse 中的数据，并且可以在右侧使用 Copilot 获得</st> <st c="76909">故障排除的帮助。</st>

                +   <st c="76930">使用 Dataverse，我们可以开发一个</st> **<st c="76964">自定义 Dataverse 插件</st>** <st c="76987">，将这些信息发送到 Application Insights</st> <st c="77015">端点，并借助</st> **<st c="77075">C# SDK</st>**<st c="77081">。</st>

            <st c="77082">正如我们所见，Azure Application</st> <st c="77117">Insights 是一个企业级的应用程序性能管理 PaaS 解决方案，我们可以轻松地将其集成到我们的 Power Platform</st> <st c="77252">产品组合中。</st>

            <st c="77270">现在，我们将深入探讨最后一个主题——功能标志的世界，以及它们能为我们的 Power</st> <st c="77377">Platform 解决方案带来什么。</st>

            <st c="77396">练习 - 引入功能标志</st>

            <st c="77433">在自定义开发项目中，</st> **<st c="77466">功能标志</st>** <st c="77479">用于启用或禁用应用程序的功能。</st> <st c="77515">敏捷团队非常受益于这一概念，因为具有重大影响和较长开发周期的功能，跨越多个冲刺，可以在完全开发之前对最终用户保持隐藏。</st> <st c="77743">考虑一些功能，例如启用 Microsoft Azure 中的 Copilot 功能或在公共仓库中提供 GitHub Copilot。</st> <st c="77876">这些功能在向公众发布之前，都是在功能标志下开发的。</st> <st c="77977">我们也经常使用功能标志来为一组用户（例如参与 Beta 测试活动的用户）启用新功能。</st> <st c="78117">上述的 Copilot 功能最初是作为专门客户的私人预览版提供的，之后进入了公共预览阶段，最终它们变得</st> <st c="78281">全面可用。</st>

            <st c="78301">特定于解决方案的</st> **<st c="78324">环境变量</st>** <st c="78345">在 Power Platform 中可以提供此功能标志能力，前提是我们</st> <st c="78411">在新功能前使用它。</st> <st c="78445">要在 Power Platform 解决方案中将环境变量用作功能标志，我们可以按照</st> <st c="78534">以下步骤操作：</st>

                1.  <st c="78546">在我们的解决方案中创建一个环境变量。</st> <st c="78599">这可以通过选择</st> **<st c="78629">新建</st>** <st c="78632">|</st> **<st c="78635">更多</st>** <st c="78639">| 解决方案中的</st> **<st c="78646">环境变量</st>** <st c="78657">来完成，我们正在</st> <st c="78690">进行的工作。</st>

                1.  <st c="78701">设置环境变量的数据类型，可以是</st> `<st c="78763">布尔值</st>`<st c="78770">、</st> `<st c="78772">选项集</st>`<st c="78782">，或</st> `<st c="78787">文本</st>`<st c="78791">，具体取决于我们为</st> <st c="78827">功能标志所需的类型。</st>

                1.  <st c="78840">在我们的解决方案组件中使用环境变量，例如 Power Automate 流、Power Apps 画布应用或自定义连接器。</st> <st c="78974">对于 Power Apps，我们可以使用</st> `<st c="79045">LookUp()</st>` <st c="79053">PowerFX 函数来访问环境变量的值，对于 Power Automate，我们可以使用 Dataverse 的</st> `<st c="79119">执行外部操作</st>` <st c="79145">，并选择操作</st> <st c="79162">名称</st> `<st c="79167">RetrieveEnvironmentVariableSecretValue</st>`<st c="79205">。</st>

                1.  <st c="79206">将我们的解决方案部署到不同的环境，并根据我们的功能标志设置更改环境变量值。</st> <st c="79336">这使我们能够根据</st> <st c="79421">环境设置启用或禁用某些功能或功能。</st>

            <st c="79442">让我们为我们的 Kudos 应用解决方案引入一个功能标志</st> <st c="79473">来控制登陆页面上需要显示哪个标签——是原始的标签，还是我们在</st> *<st c="79619">练习——构建 CD 管道和发布列车</st>* <st c="79671">一节中早些时候创建的标签：</st>

            ![图 8.30 – 作为功能标志的环境变量](img/B22208_08_30.jpg)

            <st c="80127">图 8.30 – 作为功能标志的环境变量</st>

            <st c="80182">我们定义这个</st> <st c="80198">环境变量，数据类型为</st> `<st c="80228">布尔</st>` <st c="80235">，并将其默认值设置为</st> `<st c="80427">featureFlagLabel</st>`<st c="80443">。</st>

            <st c="80444">要在画布应用中使用 PowerFX 读取环境变量的值，我们可以使用以下方法：</st>

                +   <st c="80553">确保我们已将</st> `<st c="80582">环境变量值</st>` <st c="80609">表添加到我们的画布应用的数据源中。</st>

                +   <st c="80649">使用</st> `<st c="80658">LookUp()</st>` <st c="80666">函数，结合我们环境变量的架构名称——例如，</st> `<st c="80740">LookUp('环境变量值', '环境变量定义'.'架构名称' = "</st>``<st c="80829">YourEnvironmentVariableSchemaName").Value</st>`<st c="80871">。</st>

                +   <st c="80872">这将检索我们指定的环境变量的当前值。</st> <st c="80938">。</st>

            <st c="80951">让我们打开我们的 Kudos 应用来编辑</st> <st c="80984">从我们的解决方案中，然后引入新的数据源，</st> `<st c="81044">环境变量值</st>`<st c="81071">。之后，我们需要将</st> `<st c="81107">文本</st>` <st c="81111">属性从</st> `<st c="81124">lblTitle_LandingScreen</st>` <st c="81146">的静态文本更改为</st> <st c="81167">以下内容：</st>

```
 If( <st c="81186">IsBlank(LookUp('Environment Variable Values', 'Environment Variable Definition'.'Schema Name' = "mpa_featureFlagLabel").Value)</st>,
    "Employee Kudos",
    If( <st c="81336">LookUp('Environment Variable Values', 'Environment Variable Definition'.'Schema Name' = "mpa_featureFlagLabel").Value = "no"</st>,
        "Employee Kudos",
        "Employee Kudos - April 2024"
    )
)
```

            <st c="81513">正如我们在本章第一节中所学到的，部署设置文件不仅包含连接引用，还包含环境变量。</st> <st c="81666">我们的</st> `<st c="81670">deploymentSettings.json</st>` <st c="81693">文件位于</st> `<st c="81716">\src\mpa_Kudos\deploymentSettings.json</st>`<st c="81754">。在</st> `<st c="81777">cat</st>` <st c="81780">命令的帮助下，我们已经在 GitHub 工作流中更新了此文件（</st>*<st c="81849">发布到生产环境</st>* <st c="81871">和</st> *<st c="81876">导入到开发环境</st>*<st c="81889">），关于连接引用：</st>

```
 newDataverseId="${{ github.event.inputs.dataverseConnectionId }}"
newO365Id="${{ github.event.inputs.o365IdConnectionId }}"
newOutlookId="${{ github.event.inputs.outlookIdConnectionId }}"
cat ${{ env.solution_source_folder}}/${{ env.kudos_solution_name }}/deploymentSettings.json | jq --arg dataverseId "$newDataverseId" --arg o365Id "$newO365Id" --arg outlookId "$newOutlookId" '<st c="82305">.ConnectionReferences[]</st> |=
         if .ConnectionId == "[Dataverse]" then .ConnectionId = $dataverseId
         elif .ConnectionId == "[O365]" then .ConnectionId = $o365Id
         elif .ConnectionId == "[Outlook]" then .ConnectionId = $outlookId
        else . end' > temp.json && mv temp.json ${{ env.solution_source_folder}}/${{ env.kudos_solution_name }}/deploymentSettings.json
```

            <st c="82653">类似地，我们可以更新此 JSON 文件中的环境变量值，以便在部署管道中完全自动化它们。</st> <st c="82785">我们只需</st> <st c="82797">将</st> `<st c="82813">.ConnectionReferences[]</st>` <st c="82836">数组替换为</st> `<st c="82852">.EnvironmentVariables[]</st>` <st c="82875">数组来调整</st> <st c="82890">我们的变量。</st>

            <st c="82904">在本节中，我们学习了如何利用环境变量将功能标志添加到我们的应用程序和</st> <st c="83015">云流中。</st>

            <st c="83027">总结</st>

            <st c="83035">在本章中，我们踏上了一段激动人心的旅程，深入探讨了 DevOps 和 ALM 原则的实际应用。</st> <st c="83157">我们通过各种实践练习，掌握了从存储库分支策略到构建稳健的 CD 管道、有效管理待办事项以及强制执行分支保护规则等内容。</st> <st c="83353">我们还深入探讨了解决方案的自动化测试，通过 APM 监控应用程序在运行时的性能，并利用功能标志的强大功能。</st> <st c="83512">这些练习不仅仅是理论上的；我们还将其应用于一个真实世界的例子，利用 GitHub 作为我们首选的 DevOps 工具。</st> <st c="83648">通过这些实践教程，我们将 DevOps 和 ALM 的每一个环节编织成了一个</st> [<st c="83735">实践</st> <st c="83745">经验的画卷。</st>](https://learn.microsoft.com/en-us/power-platform/enterprise-templates/overview)

            [<st c="83765">在接下来的章节中，我们将深入探讨</st>](https://learn.microsoft.com/en-us/power-platform/enterprise-templates/overview) <st c="83813">融合架构，并查看如何在我们的 Power</st> <st c="83894">Platform 解决方案中利用 Azure PaaS 服务。</st>

            <st c="83913">进一步阅读</st>

                +   <st c="83929">Power Platform 企业版</st> <st c="83956">模板：</st> [<st c="83966">https://learn.microsoft.com/en-us/power-platform/enterprise-templates/overview</st>](https://learn.microsoft.com/en-us/power-platform/enterprise-templates/overview)

                +   <st c="84044">Kudos</st> 应用程序： [<st c="84064">https://learn.microsoft.com/zh-cn/power-platform/enterprise-templates/hr/employee-kudos/install-and-set-up</st>](https://learn.microsoft.com/zh-cn/power-platform/enterprise-templates/hr/employee-kudos/install-and-set-up)

                +   <st c="84170">GitHub</st> CLI： [<st c="84187">https://github.com/cli/cli</st>](https://github.com/cli/cli)

                +   <st c="84213">GitHub CLI 与 GitHub</st> Copilot： [<st c="84250">https://docs.github.com/zh-cn/copilot/github-copilot-in-the-cli/using-github-copilot-in-the-cli</st>](https://docs.github.com/zh-cn/copilot/github-copilot-in-the-cli/using-github-copilot-in-the-cli)

                +   <st c="84342">预填充连接</st> 引用： [<st c="84379">https://learn.microsoft.com/zh-cn/power-platform/alm/conn-ref-env-variables-build-tools#get-the-connection-reference-information</st>](https://learn.microsoft.com/zh-cn/power-platform/alm/conn-ref-env-variables-build-tools#get-the-connection-reference-information)

                +   <st c="84507">GitHub</st> 环境： [<st c="84529">https://docs.github.com/zh-cn/actions/learn-github-actions/variables#using-the-vars-context-to-access-configuration-variable-values</st>](https://docs.github.com/zh-cn/actions/learn-github-actions/variables#using-the-vars-context-to-access-configuration-variable-values)

                +   <st c="84657">GitHub</st> 输出： [<st c="84674">https://docs.github.com/zh-cn/actions/using-jobs/defining-outputs-for-jobs</st>](https://docs.github.com/zh-cn/actions/using-jobs/defining-outputs-for-jobs)

                +   <st c="84745">GitHub 作业和</st> 需求： [<st c="84769">https://docs.github.com/zh-cn/actions/using-jobs/using-jobs-in-a-workflow</st>](https://docs.github.com/zh-cn/actions/using-jobs/using-jobs-in-a-workflow)

                +   <st c="84839">GitHub 复合</st> 操作： [<st c="84866">https://docs.github.com/zh-cn/actions/creating-actions/creating-a-composite-action</st>](https://docs.github.com/zh-cn/actions/creating-actions/creating-a-composite-action)

                +   <st c="84945">GitHub 拉取</st> 请求： [<st c="84968">https://docs.github.com/zh-cn/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests</st>](https://docs.github.com/zh-cn/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests)

                +   <st c="85111">GitHub</st> 市场： [<st c="85132">https://github.com/marketplace</st>](https://github.com/marketplace)

                +   <st c="85162">Power Apps 测试</st> 引擎： [<st c="85187">https://learn.microsoft.com/zh-cn/power-apps/developer/test-engine/overview</st>](https://learn.microsoft.com/zh-cn/power-apps/developer/test-engine/overview)

                +   <st c="85262">使用 Power Apps 测试</st> <st c="85292">工作室：</st> [<st c="85300">https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/working-with-test-studio</st>](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/working-with-test-studio)

                +   <st c="85387">Azure 应用程序洞察</st> <st c="85415">概述：</st> [<st c="85425">https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview</st>](https://learn.microsoft.com/en-us/azure/azure-monitor/app/app-insights-overview)

                +   <st c="85504">指标</st> <st c="85512">警报：</st> [<st c="85520">https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/tutorial-metric-alert</st>](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/tutorial-metric-alert)

                +   <st c="85602">操作</st> <st c="85610">组：</st> [<st c="85618">https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/action-groups</st>](https://learn.microsoft.com/en-us/azure/azure-monitor/alerts/action-groups)

                +   <st c="85692">Power Apps 与应用程序</st> <st c="85721">洞察：</st> [<st c="85731">https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/application-insights</st>](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/application-insights)

                +   <st c="85814">应用程序洞察</st> <st c="85851">集成概述：</st> [<st c="85864">https://learn.microsoft.com/en-us/power-platform/admin/overview-integration-application-insights</st>](https://learn.microsoft.com/en-us/power-platform/admin/overview-integration-application-insights)

                +   <st c="85960">将数据导出到应用程序</st> <st c="85988">洞察：</st> [<st c="85998">https://learn.microsoft.com/en-us/power-platform/admin/set-up-export-application-insights</st>](https://learn.microsoft.com/en-us/power-platform/admin/set-up-export-application-insights)

                +   <st c="86087">Power Pages 和应用程序</st> <st c="86116">洞察：</st> [<st c="86126">https://learn.microsoft.com/en-us/power-pages/go-live/telemetry-monitoring</st>](https://learn.microsoft.com/en-us/power-pages/go-live/telemetry-monitoring)

                +   <st c="86200">环境</st> <st c="86213">变量：</st> [<st c="86224">https://learn.microsoft.com/en-us/power-apps/maker/data-platform/environmentvariables</st>](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/environmentvariables)

                +   <st c="86309">Power Automate 自动化</st> <st c="86336">中心：</st> [<st c="86344">https://learn.microsoft.com/en-us/power-automate/automation-center-overview</st>](https://learn.microsoft.com/en-us/power-automate/automation-center-overview)

```

```

```

```

# <st c="0">第三部分：探索 DevOps 最佳实践及未来发展</st>

<st c="58">在本部分中，我们将探索构建融合团队的可能性，在这些团队中，专业开发者和 DevOps 工程师可以帮助低代码/无代码开发方法的实施。</st> <st c="237">我们将了解文化如何在更快的开发周期中发挥重要作用，以及构建可重用组件的重要性。</st> <st c="374">我们将看看 Microsoft Azure 云服务如何与 Power Platform 解决方案进行集成。</st> <st c="470">专业开发者将了解 Power Platform 的可扩展性，并能够利用自定义代码组件，扩展 Power Platform 的功能。</st> <st c="637">我们将通过研究人工智能如何改变我们开发业务应用程序的方式，来总结本章内容，探讨它如何帮助我们构建定制的副驾驶，不仅支持我们的 DevOps 流程，还能丰富我们的</st> <st c="877">业务解决方案。</st>

<st c="896">本部分包括以下章节：</st>

+   *<st c="934">第九章</st>*<st c="944">，</st> *<st c="946">实施融合开发方法</st>*

+   *<st c="990">第十章</st>*<st c="1001">，</st> *<st c="1003">在 Power Platform 中实现专业开发者扩展性</st>*

+   *<st c="1051">第十一章</st>*<st c="1062">，</st> *<st c="1064">通过设计最佳实践管理环境生命周期</st>*

+   *<st c="1126">第十二章</st>*<st c="1137">，</st> *<st c="1139">展望副驾驶、ChatOps 和 AI 驱动的应用程序</st>*
