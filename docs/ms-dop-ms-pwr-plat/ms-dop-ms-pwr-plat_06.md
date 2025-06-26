# <st c="0">6</st>

# <st c="2">深入了解持续集成/持续部署（CI/CD）管道</st>

<st c="81">在本章中，我们将</st> <st c="106">亲身体验设计</st> **<st c="144">应用生命周期</st>** **<st c="160">C</st>** **<st c="162">ycle Management</st>** <st c="177">(</st>**<st c="179">ALM</st>**<st c="182">) 过程，适用于具有代码优先和低代码/无代码组件的应用程序。</st> <st c="266">此外，我们还将讨论 DevOps 工程师如何标准化管道</st> <st c="339">并为它们注入活力</st> <st c="350">在任何其他</st> <st c="362">新的 Power Platform 项目中，使用</st> **<st c="404">管道模板</st>** <st c="422">和</st> **<st c="427">可重用工作流</st>** <st c="445">在</st> **<st c="449">Azure DevOps</st>** <st c="461">和</st> **<st c="466">GitHub</st>** <st c="472">作为可重用的构建模块。</st> <st c="502">我们将深入了解</st> <st c="523">并对</st> **<st c="557">Power Platform 的 ALM 加速器</st>** <st c="591">包有一个扎实的理解，这个包是 Power Platform 卓越中心启动工具包的一部分，以及我们如何在自己的 DevOps 环境中重用该包中的内容。</st> <st c="733">我们还将学习</st> **<st c="758">Power Platform 管道</st>** <st c="782">、托管管道，以及它们如何利用专业 DevOps 工具的功能，比如自动化测试。</st> <st c="903">最后，本章</st> <st c="923">还将教我们如何</st> **<st c="949">使用开源框架进行自动化测试</st>** <st c="966">及其在</st> **<st c="1032">持续集成与持续部署</st>** <st c="1080">(</st>**<st c="1082">CI/CD</st>**<st c="1087">)管道中的集成。</st>

<st c="1100">在本章中，我们将涵盖以下</st> <st c="1153">主要主题：</st>

+   <st c="1165">当所有内容汇聚在一起——分支、环境和 Power</st> <st c="1233">Platform 目录</st>

+   <st c="1249">Azure 管道模板和可重用</st> <st c="1288">GitHub 工作流</st>

+   <st c="1304">Power Platform 的 ALM 加速器</st>

+   <st c="1343">DevOps 和 Power</st> <st c="1382">Platform 管道中的自动化测试</st>

# <st c="1400">技术要求</st>

<st c="1423">要通过使用专业开发 DevOps 工具深入了解我们的 CI/CD 管道，我们需要具备</st> <st c="1522">以下内容：</st>

+   **<st c="1536">Power Platform 订阅</st>**<st c="1566">：如果我们已经有了 Microsoft Entra ID 工作账户，我们可以注册 Power Apps 开发者计划（</st>[<st c="1617">https://www.microsoft.com/en-us/power-platform/products/power-apps/free</st>](https://www.microsoft.com/en-us/power-platform/products/power-apps/free)<st c="1689">），或者我们也可以加入 Microsoft 365 开发者计划（</st>[<st c="1798">https://developer.microsoft.com/en-us/microsoft-365/dev-program</st>](https://developer.microsoft.com/en-us/microsoft-365/dev-program)<st c="1862">）。</st>

+   **<st c="1865">Azure DevOps Services 组织</st>**<st c="1903">：我们可以随时创建一个 DevOps 组织</st> *<st c="1951">免费</st>* <st c="1959">(</st>[<st c="1961">https://learn.microsoft.com/en-us/azure/devops/user-guide/sign-up-invite-teammates</st>](https://learn.microsoft.com/en-us/azure/devops/user-guide/sign-up-invite-teammates)<st c="2043">)。</st> <st c="2047">如果我们在 Azure DevOps 中创建一个公共项目，我们将获得多个免费的流水线，并免费访问服务的所有功能（请参阅</st> **<st c="2182">Azure DevOps for Open</st>** **<st c="2204">Source</st>** <st c="2210">提供）。</st>

+   <st c="2221">一个</st> **<st c="2224">GitHub 账号</st>** <st c="2237">和公共代码库（</st>[<st c="2261">https://github.com/signup</st>](https://github.com/signup)<st c="2287">），这也</st> *<st c="2305">是免费的</st>* <st c="2309">，适用于公共代码库。</st>

+   <st c="2334">本章讨论的示例和教程位于</st> <st c="2398">在</st> [<st c="2401">https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform/tree/main/Chapter06</st>](https://github.com/PacktPublishing/Mastering-DevOps-on-Microsoft-Power-Platform/tree/main/Chapter06)<st c="2500">。</st>

# <st c="2501">当一切都合而为一时</st>

<st c="2532">在上一章中，我们看到 Azure DevOps Services 流水线和 GitHub 流程如何与我们的 Dataverse 环境和 Power Platform 解决方案互动，以实现 DevOps</st> <st c="2710">CI/CD 能力。</st>

## <st c="2729">分支和环境</st>

<st c="2755">如果我们遵循</st> <st c="2772">基于主干的分支策略</st> <st c="2804">或非常相似的</st> **<st c="2824">GitHub flow</st>**<st c="2835">，我们将经常创建短生命周期的分支用于开发目的。</st> <st c="2910">这些分支就是</st> <st c="2932">所谓的</st> **<st c="2943">功能分支</st>** <st c="2959">，我们可以用它们来实现新的用户需求或修复客户或内部质量保证过程中发现的漏洞。</st> <st c="3084">。</st>

<st c="3095">我们可以使用</st> **<st c="3107">Power Platform 构建工具</st>** <st c="3137">以及相应的底层</st> **<st c="3172">Pac CLI</st>** <st c="3179">来创建</st> <st c="3191">开发者</st> <st c="3201">环境。</st> <st c="3215">一个特定解决方案的典型分支</st> <st c="3231">结构可能如下所示：</st>

```
 main
    test
        devA
        devB
        devZ
```

<st c="3314">在</st> **<st c="3332">分支策略</st>**<st c="3347">的帮助下，我们可以强制开发者</st> <st c="3372">使用拉取请求将更改合并回测试和/或主分支。</st> <st c="3446">在 Azure DevOps Services 中，如果我们为某个分支配置了任何分支策略（</st>**<st c="3541">要求</st>** **<st c="3550">最低审阅者数量</st>**<st c="3579">，</st> **<st c="3581">检查链接的工作项</st>**<st c="3608">，</st> **<st c="3610">检查评论解决</st>**<st c="3638">，或者</st> **<st c="3643">限制合并类型</st>**<st c="3660">），该分支</st> <st c="3675">将无法删除，并且要求</st> **<st c="3707">拉取请求</st>** <st c="3720">(</st>**<st c="3722">PRs</st>**<st c="3725">) 用于所有更改。</st> <st c="3745">因此，我们可以避免将意外更改直接提交到这些</st> <st c="3814">专用分支：</st>

![图 6.1 – 分支策略](img/B22208_06_1.jpg)

<st c="4609">图 6.1 – 分支策略</st>

<st c="4637">检查链接的工作项</st>

<st c="4665">我们强烈建议在您的专用主分支和测试分支上强制执行此设置。</st> <st c="4752">它仅仅确保，在没有明确定义的用户故事或</st> <st c="4834">待办事项的情况下，不会有任何更改。</st>

<st c="4848">我们也可以在 GitHub 上实现类似的配置。</st> <st c="4900">这被称为</st> **<st c="4915">分支保护规则</st>**<st c="4938">，它提供了更多</st> <st c="4965">选项。</st> <st c="4975">例如，我们可以要求在合并之前进行 PR 审核，要求在合并之前通过状态检查，并限制谁可以推送到该分支。</st> <st c="5118">我们还可以强制</st> **<st c="5136">签名提交</st>** <st c="5150">通过附加加密签名来验证</st> <st c="5177">提交的真实性。</st> <st c="5237">这个签名是使用提交者的私钥创建的，其他人可以使用提交者的公钥进行验证。</st> <st c="5362">签名提交提供了代码更改来自可信且</st> <st c="5439">授权源的保证。</st>

<st c="5457">下图显示了所有可用的保护规则</st> <st c="5509">适用于</st> <st c="5524">GitHub 分支：</st>

![图 6.2 – 分支保护规则](img/B22208_06_2.jpg)

<st c="7557">图 6.2 – 分支保护规则</st>

<st c="7593">无论我们使用哪个 DevOps 工具，我们的目标是创建一个 DevOps 框架，使得专业开发人员不仅能够相互协作，还能与每一个开发人员（公民开发者）合作。</st> <st c="7790">为了在开发过程中设置正确的</st> **<st c="7810">质量门控</st>** <st c="7823">，建议</st> <st c="7861">在专用和临时/开发人员 Power</st> <st c="7930">平台环境中映射我们的分支结构：</st>

![图 6.3 – 分支与环境](img/B22208_06_3.jpg)

<st c="8032">图 6.3 – 分支与环境</st>

<st c="8070">在前面的图中，Power Platform 环境和 Git 分支之间的箭头表示代码</st> <st c="8107">和部署流程。</st> <st c="8183">生产环境</st> <st c="8206">和测试环境只能处理 PR（即解决方案仅被导入到这些环境中），而开发环境应当是双向的，因为这些环境中的更改将被导出回 Git 仓库，并分别更新到开发分支。</st> <st c="8496">在我们的案例中，</st> `<st c="8513">US_XXX_Y</st>` <st c="8521">分支表示</st> <st c="8539">一个短期的功能分支，包含了用户故事的实现。</st> <st c="8616">具有不同触发条件的 CI/CD 流水线管理着这两个世界之间的互动，从解决方案生命周期的开始开始。</st> <st c="8769">流水线/工作流甚至可以用于在整个</st> <st c="8849">开发者旅程中创建这些自动化。</st>

<st c="8867">让我们详细看看如何</st> <st c="8908">使用 Azure Pipelines 自动化一些最常见的步骤：</st> <st c="8958">步骤：</st>

1.  **<st c="8971">创建开发者分支</st>**<st c="8997">：我们可以使用以下 Bash 脚本来创建</st> <st c="9047">一个分支：</st>

    ```
    <st c="9056">git config user.name $(Build.RequestedFor)</st>
    <st c="9099">git config user.email $(Build.RequestedForEmail)</st>
    <st c="9148">#creating the branch under "dev" folder</st>
    <st c="9188">newbranch=dev/${{ parameters.devbranch }}</st>
    <st c="9230">git checkout -b $newbranch</st>
    <st c="9257">git -c http.extraheader="AUTHORIZATION: bearer $(System.AccessToken)" push origin $newbranch</st>
    ```

1.  **<st c="9350">为 Power Platform 开发者环境提供服务</st>**<st c="9399">：首先，我们需要通过使用 Microsoft 托管的 Windows 机器上的服务主体登录到 Power Platform 环境，因为 Pac CLI 将身份验证凭证以纯文本形式存储在</st> <st c="9596">Linux 机器上：</st>

    ```
    <st c="9611">pac auth create --applicationId 862e5a17-d38b-43ff-b24f-88a77f59623f \</st>
     <st c="9682">--clientSecret $(ClientSecret) \</st>
     <st c="9715">--tenant 4ae51f31-033a-48fa-be48-5ece14d2c081</st>
    ```

    <st c="9761">我们还需要使用相同的服务主体登录到我们的 Azure 租户，以查明是谁触发了此</st> <st c="9878">Azure Pipeline：</st>

    ```
    <st c="9893">az login --service-principal \</st>
     <st c="9924">-u 862e5a17-d38b-43ff-b24f-88a77f59623f \</st>
     <st c="9966">-p $(ClientSecret) \</st>
     <st c="9987">--tenant 4ae51f31-033a-48fa-be48-5ece14d2c081</st>
    <st c="10033">AADObjectID=$(az ad user show \</st>
     <st c="10065">--id $(Build.RequestedForEmail) \</st>
     <st c="10099">--query id \</st>
     <st c="10112">--output tsv)</st>
    <st c="10165">$(Build.RequestedForEmail)</st> system variable contains the email address of the user who started the pipeline. We use the Microsoft Entra ID’s user object ID to create a Power Platform developer environment on behalf of this user (the command, per se, runs under the name of the service principal and not on behalf of our users):

    ```

    <st c="10491">pac admin create --name "dev-US_XXX_Y" \</st>

    <st c="10532">--type Developer \</st>

    <st c="10551">--user $AADObjectID</st>

    ```

    <st c="10571">After creating the Power Platform</st> <st c="10605">environment, we need to assign the developer (who has started the pipeline) to it as the</st> <st c="10695">System Administrator:</st>

    ```

    <st c="10716"># 获取环境 ID（倒数第二行包含此信息）</st>

    <st c="10796">rawOutput=$(pac admin list --name dev-us_XXX_Y | tail -n 2)</st>

    <st c="10856">environmentId=$(echo $rawOutput | cut -d ' ' -f 2)</st>

    <st c="10907">#为创建开发者环境的用户添加系统管理员角色</st>

    <st c="10995">pac admin assign-user --environment $environmentId --user $AADObjectID --role "系统管理员"</st>

    ```

    ```

<st c="11096">开发者环境</st>

<st c="11119">Power Platform 中的每个用户最多可以免费创建三个开发者环境</st> <st c="11195">。</st>

1.  `<st c="11615">导出解决方案</st>`<st c="11630">,</st> `<st c="11632">导入解决方案</st>`<st c="11647">,</st> `<st c="11649">复制环境</st>`<st c="11665">, 和</st> `<st c="11671">备份</st>`<st c="11677">，针对该环境。</st> <st c="11704">如果我们希望自动化服务连接创建过程，也是可行的。</st> <st c="11789">我们需要创建一个 JSON 文件，将其作为有效载荷发布到</st> `<st c="11857">REST</st>` <st c="11861">端点上，供 Azure DevOps 服务使用（参见</st> `<st c="11901">AzDO/.pipelines/config-tokenizer.yml</st>` <st c="11937">在</st> `<st c="11945">第六章</st>` <st c="11954">文件夹中的</st> <st c="11969">GitHub 仓库）：</st>

    ```
     {
       "authorization": {
          "parameters": {
             "tenantId": "GUID",
             "applicationId": "GUID",
             "clientSecret": "secret"
          },
          "scheme": "None"
       },
       "createdBy": {},
       "data": {},
       "isShared": false,
       "isOutdated": false,
       "name": "ConnenctionName",
       "owner": "library",
       "type": "powerplatform-spn",
       "url": "DataverseURL",
       "administratorsGroup": null,
       "description": "",
       "groupScopeId": null,
       "operationStatus": null,
       "readersGroup": null,
       "serviceEndpointProjectReferences": [
        {
             "description": "",
             "name": "ProjectName",
             "projectReference": {
                "id": "ProjectID",
                "name": "ProjectName"
             }
         }
       ]
    }
    ```

    <st c="12550">我们可以使用以下 Bash</st> <st c="12580">脚本来标记这个 JSON 文件（参见</st> `<st c="12620">AzDO/.pipelines/create-service-connection.yml</st>` <st c="12665">在</st> `<st c="12673">第六章</st>` <st c="12682">文件夹中的</st> <st c="12697">GitHub 仓库）：</st>

    ```
    # Set the filename
    filename="config_tokenizer.json"
    # Read the JSON file
    json_string=$(cat .pipelines/$filename)
    # Replace the value
    new_json_string=$(jq '.authorization.parameters.clientSecret = "$(ClientSecret)"' <<< "$json_string")
    new_json_string=$(jq '.authorization.parameters.tenantId = "$(tenantId)"' <<< "$new_json_string")
    new_json_string=$(jq '.authorization.parameters.applicationId = "$(applicationId)"' <<< "$new_json_string")
    new_json_string=$(jq '.url = "${{ parameters.ppdevenvironmentURL }}"' <<< "$new_json_string")
    new_json_string=$(jq '.name = "${{ parameters.ppdevenvironment }}"' <<< "$new_json_string")
    new_json_string=$(jq '.serviceEndpointProjectReferences[0].projectReference.id = "$(System.TeamProjectId)"' <<< "$new_json_string")
    new_json_string=$(jq '.serviceEndpointProjectReferences[0].projectReference.name = "$(System.TeamProject)"' <<< "$new_json_string")
    new_json_string=$(jq '.serviceEndpointProjectReferences[0].name = "${{ parameters.ppdevenvironment }}"' <<< "$new_json_string")
    # Write the modified JSON data back to the file
    echo $new_json_string > config.json
    ```

    <st c="13813">如果我们想要在本地执行</st> <st c="13835">这个脚本，那么我们可以使用</st> `<st c="14589">az devops</st>` <st c="14598">CLI 命令来自动创建我们的</st> <st c="14640">服务连接：</st>

    ```
    <st c="14659">echo $(PatToken) | az devops login --organization $(System.CollectionUri)</st>
    <st c="14733">az devops service-endpoint create \</st>
     <st c="14769">--organization $(System.CollectionUri) \</st>
    <st c="15175">AzDO/.pipelines/import-solution.yml</st> in the <st c="15218">Chapter06</st> folder of the GitHub repo):

    ```

    steps:

    - checkout: git://PowerPlatform/Copilot@${{parameters.sourceBranch}}

    displayName: '检出源分支'

    persistCredentials: true

    - task: PowerPlatformToolInstaller@2

    inputs:

    DefaultVersion: true

    - task: PowerPlatformPackSolution@2

    inputs:

        SolutionSourceFolder: '$(System.DefaultWorkingDirectory)/src/$(solutionName)'

        SolutionOutputFile: '$(Build.ArtifactStagingDirectory)/Solution/$(solutionName).zip'

        SolutionType: 'Unmanaged'

    - task: PublishBuildArtifacts@1

    inputs:

        PathtoPublish: '$(Build.ArtifactStagingDirectory)/Solution'

        ArtifactName: 'Solution'

        publishLocation: '容器'

    - task: PowerPlatformImportSolution@2

    inputs:

        authenticationType: 'PowerPlatformSPN'

        PowerPlatformSPN: ${{parameters.serviceconnection}}

        SolutionInputFile: '$(Build.ArtifactStagingDirectory)/Solution/$(solutionName).zip'

        AsyncOperation: true

        MaxAsyncWaitTime: '60'

    - task: PowerPlatformPublishCustomizations@2

    inputs:

        authenticationType: 'PowerPlatformSPN'

        PowerPlatformSPN: ${{parameters.serviceconnection}}

    ```

    <st c="16256">We use</st> `<st c="16264">${{parameters.serviceconnection}}</st>` <st c="16297">as a string parameter of the pipeline, which represents the name of the service connection</st> <st c="16389">previously created.</st><st c="16408">We can put these steps together</st> <st c="16440">into one pipeline or keep them separately.</st> <st c="16484">We can also chain these pipelines by using other trigger conditions, such as</st> `<st c="16561">branch created</st>`<st c="16575">. For instance, a new dev branch is created under the</st> `<st c="16629">dev/*</st>` <st c="16634">condition looks,</st> <st c="16652">as follows:</st>

    ```

    <st c="16663">trigger:</st>

    <st c="16672">-  dev/*</st>

    ```

    ```

<st c="16680">经过这些自动化步骤之后，我们最终得到了一个开发分支，可以在上面开始我们的工作。</st> <st c="16792">建议使用 Azure DevOps 中的可用工具将一个或多个工作项分配给该分支；否则，后续提交的 PR 将会被自动拒绝（如果配置了推荐的分支策略）。</st>

<st c="17014">下图显示了如何分配</st> <st c="17062">分支到</st> **<st c="17076">Issue</st>** <st c="17081">工作项：</st>

![图 6.4 – 将分支链接到工作项](img/B22208_06_4.jpg)

<st c="17272">图 6.4 – 将分支链接到工作项</st>

<st c="17316">在 Power 平台环境中进行更改</st> <st c="17365">应该在开发分支上反映出版本控制系统中的更改。</st> <st c="17452">为了实现这一点，我们需要创建另一个管道（</st>`<st c="17505">Export-to-Git</st>`<st c="17519">），导出我们的托管和非托管解决方案，解包并提交更改到我们的开发分支。</st> <st c="17625">我们在</st> *<st c="17665">第五章</st>*<st c="17674"> 中看到了这些构建模块。除了该管道外，我们还需要添加一个步骤来导出未托管的解决方案，并以</st> `<st c="17801">Both</st>`<st c="17805"> 类型解包解决方案。要将更改提交回分支，我们可以在</st> `<st c="17916">PowerPlatformExportSolution@2</st>` <st c="17945">和</st> `<st c="17950">PowerPlatformUnpackSolution@2</st>` <st c="17979">构建任务完成后，执行以下 Git 命令作为 Bash 脚本：</st>

```
 -  checkout:  git://$(System.TeamProject)/$(Build.Repository.Name)@${{parameters.targetBranch}}
    displayName:  'Checkout  Source  Branch'
    persistCredentials:  true
#  Configure  email/name  and  checkout  git  branch
-  script:  |
      git  config  user.name  $(Build.RequestedFor)
      git  config  user.email  $(Build.RequestedForEmail)
      git  checkout  origin/${{parameters.targetBranch}}  --track
[[Adding here those PowerPlatformExportSolution@2 and PowerPlatformUnpackSolution@2 steps discussed earlier.]]
-  script:  |
        #!/bin/bash
        set  -e
        #  Set  the  path  to  the  directory  containing  the  solution
        solution_dir="$(Build.SourcesDirectory)/src/$(solutionName)"
        #  Find  all  Solution.xml  files  in  the  solution  directory  and  its  subdirectories
        find  "$solution_dir"  -type  f  -name  "Solution.xml"  |  while  read  -r  file;  do
                #  Replace  the  content  of  the  <Version>  tag  with  0.0.0.0
                sed  -i  's|<Version>.*</Version>|<Version>0.0.0.0</Version>|g'  "$file"
        done
    displayName:  "Set  version  number  to  0.0.0.0"
-  script:  |
        rm  -rf  ./out/$(solutionName).zip
        rm  -rf  ./out/$(solutionName)_managed.zip
        git  add  --all
        git  commit  -am  "Solution  is  committed"  --allow-empty
        git  -c  httpO.extraheader="AUTHORIZATION:  bearer  $(System.AccessToken)"  push  origin  ${{parameters.targetBranch}}
```

<st c="19227">通过这个管道的帮助，我们的开发人员可以不断地将他们的更改提交回他们自己的</st> <st c="19329">专用开发分支，进行特性开发。</st> <st c="19384">我们有意将解决方案的版本号设置为</st> `<st c="19443">0.0.0.0</st>` <st c="19450">，以避免冲突的合并，并通过 Azure 管道管理版本号。</st> <st c="19539">当开发人员准备好时，他们可以提交 PR 回到</st> <st c="19599">父分支。</st>

<st c="19613">让我们讨论一些额外的 DevOps 设计原则和建议</st> <st c="19688">与我们的</st> <st c="19713">DevOps 过程相结合：</st>

+   `<st c="19783">PowerPlatformPackSolution@2</st>` <st c="19810">构建任务可以随时从源代码控制中打包我们的解决方案。</st> <st c="19872">这是一个压缩步骤，实际上是将我们的文件夹打包成 ZIP 文件，因此不需要长时间的编译时间。</st> <st c="19973">当然，如果我们有</st> <st c="19994">需要构建的代码组件，例如</st> **<st c="20032">PowerApps 组件框架</st>** <st c="20061">(</st>**<st c="20063">PCF</st>**<st c="20066">) 控件或 Dataverse 插件，或者我们针对同一清单（解决方案）目标多个环境，则值得创建专门的 CI 管道。</st>

+   `<st c="20361">Export-to-Git</st>` <st c="20374">管道，我们可以提交 PR 来将我们的更改合并到测试分支，然后再合并到生产分支。</st> <st c="20472">我们可以引入 CD 管道，该管道在目标分支上的 PR 完成后执行。</st> <st c="20581">分支触发：</st>

    ```
    <st c="20596">trigger:</st>
    <st c="20969">SolutionChecker</st> or performing <st c="20999">automated tests</st>. We can introduce approval processes for staging environments (test and production) to control when changes are approved by release managers or environment administrators, with the help of <st c="21204">Azure DevOps</st> <st c="21217">Services environments</st>.
    ```

+   **<st c="21239">解决方案的版本号 - 何时需要设置它们？</st>** <st c="21299">如前所述，随着 DevOps 的兴起，我们希望只维护一个版本，即我们应用程序在生产中的最新版本。</st> <st c="21431">我们使用版本控制和版本号来追踪更改到</st> <st c="21509">源代码：</st>

    ```
     #  Solution  version  in  source  control  is  not  used. Instead,  create  version  at  build  time  from  the  current  build  number. -  pwsh:  |
          Get-ChildItem  -Path  "$(Build.SourcesDirectory)\$(RepoName)\${{parameters.solutionName}}\SolutionPackage\**\Solution.xml"  |
          ForEach-Object  {
                    (Get-Content  $_.FullName)  `
                            -replace  '<Version>0.0.0.0<\/Version>',  '<Version>$(Build.BuildNumber)</Version>'  |
                    Out-File  $_.FullName
          }
        displayName:  'Update  Solution  XML  with  Build  Number'
    name:  1.0.$(Date:yyyyMMdd)$(Rev:.r)
    ```

    <st c="22099">这意味着在构建/部署过程中，版本</st> <st c="22156">是实时计算的，使用 Azure 管道的构建名称。</st> <st c="22227">这提供了解决方案版本与生成该版本的管道之间的一对一关系。</st> <st c="22328">该版本。</st>

+   `<st c="22505">导出到 Git</st>` <st c="22518">管道中，我们可以引入一个参数，该参数期望部署父分支当前版本的分支名称。</st> <st c="22659">如果我们不想以这种方式覆盖我们的工作，我们可以使用诸如</st> `<st c="22744">git rebase</st>` <st c="22754">等 Git 命令将我们的开发分支恢复到父分支的最新版本。</st> <st c="22831">之后，我们需要使用另一个管道将解决方案导入到我们的</st> <st c="22908">开发环境。</st>

<st c="22931">总的来说，我们可以通过以下方式管理我们的开发工作</st> <st c="22995">四个管道：</st>

![图 6.5 - 管理解决方案的管道](img/B22208_06_5.jpg)

<st c="23412">图 6.5 - 管理解决方案的管道</st>

<st c="23455">我们可以以同样的方式创建 GitHub 工作流。</st> <st c="23504">那些 Bash 和 PowerShell 脚本可以轻松迁移到 GitHub 工作流，还可以建立与 Azure 类似的分支和仓库结构，并分配 Power Platform 环境。</st> <st c="23699">DevOps 服务。</st>

## <st c="23715">Power Platform 目录</st>

<st c="23742">除了这种</st> <st c="23767">CI/CD 交付到 Power Platform 环境之外，Power Platform 还提供了一个新的概念，用于管理我们可重用的软件包。</st> <st c="23892">该</st> **<st c="23896">Power Platform 目录</st>** <st c="23918">是一个软件包管理器，包含解决方案模板、PCF 组件、端到端解决方案以及其他可重用资产。</st> <st c="24049">它基于我们在专业开发场景中发布软件包到不同的包管理库时使用的相同概念，例如 NuGet、npm、Maven 和 Pip。</st> <st c="24225">Power Platform 中的目录是一个功能，允许开发者和创建者轻松发现并使用组织内的 Power Platform 模板和代码组件。</st> <st c="24401">它提供了一个私有的中央位置，用于查找和安装最新且最可靠的组件版本，提供能够立即产生价值的模板。</st> <st c="24570">对于管理员和业务审批人而言，目录作为一个唯一的权威来源，用于存储和维护 Power Platform 工件，允许他们策划和控制内容，加速创作者和开发者的价值。</st> <st c="24811">它还为在敏感的监管和法定场景中使用批准的组件和模板启用了审批工作流，提供具有设置</st> <st c="24994">和元数据的管理功能。</st>

<st c="25007">为了将目录带入一个专用环境（建议提供一个单独的环境，例如</st> `<st c="25122">目录</st>`<st c="25129">），我们需要将其作为 Dynamics 365 应用从</st> <st c="25151">AppSource 安装：</st> [<st c="25199">https://appsource.microsoft.com/product/dynamics-365/powerappssvc.catalogmanager-preview?flightCodes=dde212e5c66047c59bf2b346c419cef6</st>](https://appsource.microsoft.com/product/dynamics-365/powerappssvc.catalogmanager-preview?flightCodes=dde212e5c66047c59bf2b346c419cef6)<st c="25332">。</st>

<st c="25333">安装此 D365 应用后，我们可以提交并部署以下资产</st> <st c="25420">到其中：</st>

+   **<st c="25426">Dataverse 解决方案</st>** <st c="25446">(已管理或未管理) 或</st> **<st c="25473">软件包部署器软件包</st>** <st c="25498">（企业模板也可以</st> <st c="25524">发布到</st> <st c="25546">目录中）</st>

+   **<st c="25558">模板</st>** <st c="25568">用于 Power App 或 Power</st> <st c="25596">Automate 流程</st>

+   **<st c="25609">Power Platform 代码优先组件</st>** <st c="25646">如自定义连接器</st> <st c="25672">或</st> **<st c="25676">Power Apps 组件框架</st>** <st c="25706">(</st>**<st c="25708">PCF</st>**<st c="25711">) 控件</st>

<st c="25722">Power Platform 目录</st> <st c="25749">引入了额外的安全角色来管理目录项的生命周期——</st> **<st c="25827">目录提交者</st>** <st c="25844">用来提交项目到目录，</st> **<st c="25877">目录只读成员</st>** <st c="25901">用来发现和安装</st> <st c="25925">目录中的项目，</st> **<st c="25950">目录批准者</st>** <st c="25966">用来批准提交到</st> <st c="25989">目录的项目，</st> <st c="25996">以及</st> **<st c="26010">目录管理员</st>** <st c="26031">用来管理目录。</st> <st c="26059">我们也从目录中获得了这种企业级的安全性。</st>

<st c="26119">我们的 Azure 管道和 GitHub 工作流不仅可以面向环境，还可以面向目录。</st> <st c="26213">Pac CLI 提供了适当的命令，将我们的解决方案、自定义连接器和 PCF 组件提交到目录。</st> <st c="26338">以下脚本创建一个提交到我们的</st> `<st c="26387">mpa_ITBase_managed</st>` <st c="26405">解决方案（我们在</st> *<st c="26430">第四章</st>* <st c="26439">用于管理的管道）</st>，并从企业模板提交一个新的</st> <st c="26511">目录项：</st>

```
 # Power Platform Catalog Manager application's GUID
pac admin list --application 83a35943-cb41-4266-b7d2-81d60f383695
# environment is the "Catalog" environment
pac catalog list --environment 0b90d036-e017-e840-9287-b1de3d95e252
# create a submission json file
pac catalog create-submission
# update the generated json file
pac catalog submit --environment 0b90d036-e017-e840-9287-b1de3d95e252 --path .\submission.json --solution-zip .\mpa_ITBase_managed.zip
```

<st c="26983">这些脚本也可以代表服务</st> <st c="27026">主体运行。</st> <st c="27047">默认情况下，提交需要由目录管理员批准，但我们可以更改配置，使某些发布者，例如在 Azure 管道或 GitHub 工作流中运行的服务主体，获得</st> <st c="27261">自动批准。</st>

<st c="27280">下图展示了</st> `<st c="27410">mpa_ITBase_managed</st>` <st c="27428">解决方案：</st>

![图 6.6 – Power Platform 目录管理器](img/B22208_06_6.jpg)

<st c="27540">图 6.6 – Power Platform 目录管理器</st>

<st c="27583">Power Platform 目录</st>

<st c="27607">Power Platform 目录</st> <st c="27631">将在它们变为</st> **<st c="27648">管理环境的一部分</st>** <st c="27668">后，</st> <st c="27687">正式</st> <st c="27696">发布。</st>

<st c="27707">现在让我们继续讨论管道模板和可重用的工作流，它们可以在大规模上提供我们的 Azure 管道和 GitHub 工作流的可重用性。</st>

# <st c="27861">Azure 管道模板和可重用的 GitHub 工作流</st>

<st c="27916">回顾我们前一节中的 Azure 流水线</st> <st c="27952">，我们可以说，这其实是一个非常好的起点，可以让所有工作自动化。</st> <st c="28059">然而，如果我们需要支持数百甚至数千个应用程序，那么在每个包含这些解决方案的存储库中保留三到四个流水线的方法将变得更加复杂。</st> <st c="28283">随着我们开发的应用程序越来越多，这些 Azure 流水线的不同版本将会并行存在。</st> <st c="28412">如果能有一个集中式存储库，包含这些流水线的业务逻辑，且只在调用它们之前定制一些参数，那就太好了。</st> <st c="28584">这就是 Azure 流水线模板和可重用</st> <st c="28646">GitHub 工作流背后的理念。</st>

**<st c="28663">Azure 流水线模板</st>** <st c="28688">使我们能够创建可重用的业务逻辑，这些逻辑可以在我们的 YAML 流水线实例中使用。</st> <st c="28796">这些模板还可以用来控制流水线中允许的内容，通过定义调用者应遵循的策略。</st> <st c="28924">例如，我们可以通过使用一个充当保护措施的模板来强制执行任务执行约束，确保执行的任何任务都符合我们组织的</st> <st c="29090">安全指南。</st>

<st c="29110">例如，以下</st> `<st c="29199">AzDO/.pipelines/template/include-paccli-steps.yml</st>` <st c="29248">位于</st> `<st c="29256">Chapter06</st>` <st c="29265">文件夹中的</st> <st c="29280">GitHub 仓库）：</st>

```
 # File: templates/include-paccli-steps.yml
steps:
-  bash:  |
        echo  "Installing  Pac  CLI"
        dotnet  tool  install  --global  Microsoft.PowerApps.CLI.Tool
    displayName:  "Installing  Pac  CLI"
```

<st c="29471">使用此模板的流水线文件会在 Linux 上执行，然后在 Windows 代理上执行（</st><st c="29570">参见</st> `<st c="29575">AzDO/.pipelines/azure-pipeline-template.yml</st>`<st c="29618">）：</st>

```
 # File: azure-pipelines.yml
jobs:
- job: Linux
  pool:
    vmImage: 'ubuntu-latest'
  steps:
  - template: templates/include-paccli-steps.yml
- job: Windows
  pool:
    vmImage: 'windows-latest'
  steps:
  - template: templates/include-paccli-steps.yml
```

<st c="29854">我们还可以对 Azure 流水线模板</st> <st c="29907">文件进行参数化，调用者流水线可以根据其</st> <st c="29984">定制需求设置这些参数。</st>

<st c="30004">相同的</st> <st c="30013">概念，称为</st> `<st c="30161">workflow_call</st>`<st c="30174">。以下可重用的工作流将在目标平台上安装 Pac CLI。</st> <st c="30253">该工具的版本号和目标平台是此流程的参数（</st><st c="30337">参见</st> `<st c="30342">GitHub/.github/workflows/install-paccli.yml</st>`<st c="30385">）：</st>

```
 on:
  workflow_call:
    inputs:
      version:
        required: true
        type: number
      platform:
       required: true
       type: string
       description: "platform"
jobs:
  installpac:
    runs-on: ${{inputs.platform}}
    steps:
     - name: install pac cli
       shell: bash
       run: |
          echo "version number: ${{inputs.version}}"
          dotnet tool install --global Microsoft.PowerApps.CLI.Tool
```

<st c="30713">要从其他工作流中调用我们的工作流</st> <st c="30734">，我们需要在我们的</st> *<st c="30798">调用者</st>* <st c="30804">GitHub 工作流中创建一个专门的任务（</st><st c="30823">参见</st> `<st c="30828">GitHub/.github/workflows/Deploy.yml</st>`<st c="30863">）：</st>

```
 name: Call a reusable workflow
on:
  workflow_dispatch:
jobs:
  call-paccli-workflow:
    uses: jovadker/ppmanaged/.github/workflows/install-paccli.yml@main
    with:
      version: 8
      platform: ubuntu-latest
  call-workflow-windows:
    uses: jovadker/ppmanaged/.github/workflows/install-paccli.yml@main
    with:
      version: 8
      platform: windows-latest
```

<st c="31188">这个概念在 Azure DevOps 和 GitHub 中的真正力量在于，我们可以将模板</st> <st c="31281">和可复用的工作流放置在同一代码库中的不同位置，甚至放在不同的 Git 代码库中。</st> <st c="31391">如果我们查看为启动开发人员环境、创建服务连接、以及分别从/向环境导出/导入解决方案而创建的管道，我们现在可以将这些模板或可复用的工作流放置在专用的 Git 代码库中。</st> <st c="31652">不同的项目和解决方案可以通过使用这个可复用的概念来调用我们的模板。</st> <st c="31744">相反，我们可以通过减少管理大量项目的整体工作量来集中改进和维护这些工作流。</st> <st c="31865">项目。</st>

<st c="31877">除了可复用的</st> <st c="31898">工作流，GitHub 还提供了所谓的</st> **<st c="31934">GitHub 启动工作流</st>**<st c="31958">。GitHub 启动工作流也是存储在组织的</st> *<st c="32041">GitHub 代码库</st>*<st c="32058">中的模板。</st> <st c="32138">当我们在 GitHub 用户界面中创建新工作流时，这些模板会显示出来。</st> <st c="32272">创建工作流后，模板会被复制到我们的代码库中，至此，我们将实例与模板解耦。</st> <st c="32272">我们可以使用 GitHub 启动工作流帮助项目基于我们的</st> <st c="32362">可复用工作流来设置它们的工作流。</st>

<st c="32381">现在我们已经了解了管道模板和可复用工作流，接下来让我们学习一下 Power Platform 的 ALM 加速器，它在背后大量使用了这个概念。</st> 

# <st c="32553">Power Platform 的 ALM 加速器</st>

<st c="32592">Power Platform 的 ALM 加速器</st> <st c="32631">是一个帮助开发人员自动化 Power Platform 软件开发工作流的工具。</st> <st c="32730">尽管</st> <st c="32738">它是</st> **<st c="32757">卓越中心</st>** <st c="32777">(</st>**<st c="32779">CoE</st>**<st c="32782">)</st> **<st c="32785">启动工具包</st>**<st c="32796">的一部分，该工具包是帮助组织开始采用 Power Platform 的组件和工具的集合，但它也可以作为独立的解决方案安装在我们的</st> <st c="32969">Dataverse 环境中。</st>

ALM Accelerator 包括 Azure 管道模板、Azure 管道和 Power Platform 应用程序，帮助专业开发人员在项目中实现 DevOps 和 ALM。这个加速器包被称为**ALM Accelerator for Advanced Makers**（**AA4AM**）解决方案，包含了最初由微软 CoE Starter Kit 工程团队开发的参考实现。成功实现后，团队决定将其开源，并使每个组织都能使用，证明了如何在 Power Platform 中实现健康的应用生命周期管理。我们可以直接使用这些管道和应用程序，或者根据我们的组织需求对其进行自定义，当然，我们也可以从这个团队几年前设计的基本概念中学到很多东西。

我们可以找到关于加速器的全面文档，了解如何安装它，如何基于可用的管道模板创建管道，以及如何在 Azure DevOps 仓库中管理我们的 Power Platform 解决方案。

本章前面讨论过的分支、环境和管道模板，对于理解 ALM Accelerator 如何在后台工作至关重要。加速器的主要功能如下：

+   有一个模型驱动的应用程序，**ALM Accelerator for Power Platform Administration**，用于创建和管理部署配置文件。**部署配置文件**描述了解决方案在我们环境中的 CD 流程。它在概念上与我们如何定义 Power Platform 管理管道相同。建议为每个 Power Platform 解决方案创建一个部署配置文件，因为该配置文件还包含有关 Azure DevOps 项目、Git 仓库和目标分支的信息，以及相应的 Power Platform 环境。

+   另有一个画布应用程序，ALM Accelerator for Power Platform，为开发人员提供了一个 UI，并在给定的 Power Platform 环境中可视化解决方案和分配的部署配置文件。

+   通过 ALM Accelerator for Power Platform 画布应用程序，我们可以轻松创建一个分支，并将我们的解决方案版本（更改）多次提交到该分支。

+   <st c="35167">Canvas 应用程序提供了一种通过定义的环境移动我们更改的方式。</st> <st c="35252">每次部署（发布）都是对父分支的 PR。</st> <st c="35309">在 Azure DevOps 中完成 PR 后，管道将解决方案导入到部署配置文件中定义的分配环境。</st>

+   <st c="35444">建议为每个解决方案创建一个仓库。</st> <st c="35502">该仓库包含我们的管道，利用了加速器提供的管道模板。</st>

+   <st c="35603">我们可以添加更多环境，并创建我们在本章开头讨论的相同分支结构。</st>

+   <st c="35729">Canvas 应用程序使用自定义连接器，并调用 Azure DevOps Services 的 REST API 端点与</st> <st c="35847">定义的管道进行交互。</st>

+   <st c="35865">该</st> `<st c="35870">coe-alm-accelerator-templates</st>` <st c="35899">GitHub 仓库包含了 ALM 加速器使用的管道模板。</st> <st c="35990">我们需要将此仓库分叉到我们的 Azure DevOps 项目中，作为一个额外的</st> <st c="36061">Git 仓库。</st>

+   <st c="36076">我们可以根据我们</st> <st c="36125">组织的需求自定义管道模板。</st>

+   <st c="36146">Power Platform 的 ALM 加速器</st> <st c="36185">仅适用于 Azure</st> <st c="36214">DevOps Services。</st>

<st c="36230">下图展示了 Power Platform 的 ALM 加速器</st> <st c="36297">的工作原理：</st>

![图 6.7 – Power Platform 的 ALM 加速器](img/B22208_06_7.jpg)

<st c="37172">图 6.7 – Power Platform 的 ALM 加速器</st>

<st c="37223">我们可以看到所选环境中可用的未管理解决方案（开发者环境）。</st> <st c="37323">我们可以使用</st> **<st c="37338">提交解决方案</st>**<st c="37353">、</st> **<st c="37355">部署解决方案</st>**<st c="37370">和</st> **<st c="37376">删除解决方案</st>** <st c="37391">按钮来触发我们的管道，提交我们的更改，或</st> <st c="37449">启动 PR。</st>

<st c="37462">Power Platform 的 ALM 加速器</st>

<st c="37501">通过观察本章中前面展示的广泛灵活性</st> <st c="37543">以及管道在托管环境中的有效性，我们应该考虑将 Power Platform 的 ALM 加速器作为一个工具，以帮助我们熟悉各种设计模式和概念。</st> <st c="37801">然后，这些可以被纳入我们自定义的 Azure DevOps 管道，并随后集成到我们的</st> <st c="37894">GitHub 工作流中。</st>

<st c="37911">现在，我们已经准备好在组织中操作和管理大量的项目和解决方案。</st> <st c="38028">确保我们应用程序质量的最后一件事是</st> **<st c="38107">自动化测试</st>**<st c="38124">。</st>

# <st c="38125">DevOps 和 Power Platform 管道中的自动化测试</st>

<st c="38182">测试自动化和自动化测试</st> <st c="38218">是 ALM 和 DevOps</st> <st c="38257">流程的核心部分。</st> <st c="38268">没有自动化</st> <st c="38285">测试，我们无法</st> <st c="38308">与其他开发人员共享解决方案，无法作为独立的 SCRUM 团队共同合作解决方案，而且每次修改解决方案时都是一项巨大的挑战，因为我们无法知道我们的应用程序会在回归测试中跌落多远。</st> <st c="38557">为了避免回归并实现高测试覆盖率，我们需要建立与定制开发项目相同的质量保证流程。</st> <st c="38702">Power Platform 提供了多种工具和框架来为</st> <st c="38787">我们的解决方案创建功能测试。</st>

`<st c="39841">选择</st>`<st c="39847">,</st> `<st c="39849">设置属性</st>`<st c="39860">,</st> `<st c="39862">断言</st>`<st c="39868">, 和</st> `<st c="39874">追踪</st>`<st c="39879">. 测试表达式构建测试用例。</st> `<st c="40156">真</st>` <st c="40160">或</st> `<st c="40164">假</st>` <st c="40169">在测试中。</st> <st c="40183">如果断言返回</st> `<st c="40208">假</st>`<st c="40213">, 测试</st> <st c="40224">用例失败。</st>

<st c="40235">使用 Power Apps 测试工作室创建的测试可以通过 Azure 管道执行。</st> <st c="40314">在</st> *<st c="40354">进一步阅读</st>* <st c="40369">部分提供的逐步指南（</st>*<st c="40379">使用 Azure DevOps Pipelines 自动化测试</st>*<st c="40422">）使用测试的回放 URL 作为自动化测试执行的输入。</st> <st c="40497">在</st> [<st c="40528">https://github.com/microsoft/PowerAppsTestAutomation</st>](https://github.com/microsoft/PowerAppsTestAutomation) <st c="40580">下提供的仓库</st> `<st c="40675">MSTest</st>` <st c="40681">测试方法使用</st> `<st c="40958">AppMagic.TestStudio.GetTestExecutionInfo()</st>` `<st c="41121">MSTest</st>` <st c="41127">测试方法然后根据这些结果决定成功或失败。</st> <st c="41188">测试方法的结果还会写回我们的 Azure 管道，我们可以直接在 Azure DevOps 服务的</st> <st c="41327">构建结果中查看测试结果。</st>

**<st c="41341">Power Apps 测试引擎</st>** <st c="41364">是测试 PowerApps</st> <st c="41405">画布应用的最新方法，使用基于 Playwright 的引擎。</st> **<st c="41452">Playwright</st>** <st c="41462">因其稳定性</st> <st c="41496">和性能、简便的设置、对 Chromium、Firefox 和 WebKit 的统一 API 支持，以及 JavaScript、Python 和 C# 的官方绑定而广受欢迎。</st> <st c="41627">Playwright 最初由微软开发，现已开源并发布供</st> <st c="41723">社区贡献。</st>

<st c="41747">PowerApps 测试引擎允许我们使用直观的 Power Fx 语言以 YAML 格式编写测试，</st> **<st c="41860">文档对象模型</st>** <st c="41881">(</st>**<st c="41883">DOM</st>**<st c="41886">) 抽象化使得我们可以使用在设计阶段建立的控件名称引用来编写测试。</st> <st c="41935">这种方法意味着测试作者不需要具备</st> <st c="42072">JS 知识</st> <st c="42085">或对与应用程序视觉输出相关的浏览器 DOM 的了解。</st> <st c="42129">以下 YAML 文件</st> <st c="42181">展示了一个这样的例子：</st>

```
 testSuite:
   testSuiteName: Regression test suite
   testSuiteDescription: ''
   persona: User1
   appLogicalName: vadkerti_contentgenerator_48e05
   appId: ''
   onTestCaseStart: ""
   onTestCaseComplete: ""
   onTestSuiteComplete: ""
   networkRequestMocks:
   testCases:
   - testCaseName: Test with empty input
      testCaseDescription: ''
      testSteps: |
         =
         SetProperty(Prompt.Text, "Writing some test");
         Select(SystemPrompt);
         SetProperty(Prompt.Text, "");
         Select(SystemPrompt);
         Select(Prompt);
         Select(Button4);
         Assert(IsMatch(TextInput4.Text, "I'm sorry, I'm not sure what you are asking. Can you please provide more context or clarify your question?<|im_end|>"), "Different output");
testSettings:
   filePath:
   browserConfigurations:
   - browser: Chromium
      device:
      screenWidth: 0
      screenHeight: 0
   locale: en-US
   recordVideo: true
   headless: true
   enablePowerFxOverlay: false
   timeout: 30000
environmentVariables:
   filePath:
   users:
   - personaName: User1
      emailKey: user1Email
      passwordKey: user1Password
```

<st c="43158">我们可以从 Power Apps 测试工作室</st> <st c="43189">下载这个 YAML 文件</st> <st c="43217">并导入</st> <st c="43228">到我们的</st> <st c="43240">PowerApps</st> <st c="43251">测试引擎。</st>

<st c="43263">测试引擎还支持</st> **<st c="43290">连接器模拟</st>**<st c="43307">，允许我们在应用程序中围绕连接器和连接创建模拟。</st> <st c="43336">因此，我们可以将应用程序与第三方生产服务隔离进行测试。</st> <st c="43412">此外，测试引擎支持截图和视频录制，允许我们在测试执行的任何时候截屏，并通过视频记录文档化测试序列，这对于故障排除未通过的测试和分析实际用户体验非常有帮助。</st> <st c="43822">未通过的测试场景。</st>

<st c="43831">其他我们可以在管道执行期间调用的知名工具，借助 Pac CLI</st> <st c="43932">和 PowerShell 模块，</st> 是 `<st c="44003">AppChecker</st>` <st c="44013">(</st>`<st c="44015">pac solution check</st>`<st c="44033">) 用于</st> <st c="44040">画布应用程序。</st>

<st c="44052">在测试</st> **<st c="44066">Microsoft Copilot Studio</st>**<st c="44090">时，我们可以使用</st> **<st c="44107">PVA 测试框架</st>**<st c="44125">，一个可以在 GitHub 上找到的示例解决方案。</st> <st c="44149">它演示了如何使用 API 和 Direct Line 通道对 Microsoft Copilot Studio 聊天机器人进行测试。</st> <st c="44285">该框架确认机器人在多种情况下按预期运行，包括</st> <st c="44330">以下情况：</st>

+   <st c="44395">测试自然语言理解模型以触发</st> <st c="44460">正确的话题。</st>

+   <st c="44473">验证多个匹配话题的选项（“</st>*<st c="44527">您是指...</st>*<st c="44541">”如果用户意图</st> <st c="44597">模糊不清，副驾驶助手会提出此问题）</st>

+   <st c="44610">进行负载测试</st> <st c="44635">的大规模测试</st>

+   <st c="44643">完整的</st> <st c="44661">端到端对话测试</st>

+   <st c="44685">测试</st> <st c="44715">自适应卡片的功能</st>

+   <st c="44729">在 CI/CD 部署管道中添加一个测试步骤，以防测试失败时防止部署</st> <st c="44810">测试失败</st>

<st c="44820">我们可以手动创建</st> **<st c="44835">对话脚本</st>** <st c="44857">，或者我们可以使用</st> <st c="44881">PVA 测试框架</st> <st c="44904">从 Dataverse 导出现有对话</st> <st c="44952">，并以</st> *<st c="44960">CHAT 格式</st>*<st c="44971">进行转换。然后，我们可以借助 PVA 测试框架将这些 CHAT 文件转换为 JSON，并让它们通过 Direct Line REST API 端点执行。</st> <st c="45123">该框架可以轻松集成到我们的 Azure 管道和 GitHub 工作流中，因为它是一个.NET 6 控制台应用程序，可以在基于 Windows 的</st> <st c="45272">托管代理上运行。</st>

<st c="45286">要测试</st> **<st c="45295">基于模型的 Power Apps</st>** <st c="45318">和</st> **<st c="45323">Power Pages 网站</st>**<st c="45343">，我们可以使用其他</st> <st c="45361">网页测试</st> <st c="45370">框架，例如：</st> <st c="45391">以下内容：</st>

+   <st c="45405">**<st c="45410">Selenium 测试框架</st>**</st> <st c="45433">是一个自动化测试工具集，允许开发人员创建强大的基于浏览器的回归自动化套件和测试。</st> <st c="45458">它可以扩展并分配脚本到多个环境，并支持广泛的编程语言。</st>

+   <st c="45680">Playwright 是一个跨浏览器、跨平台、跨语言的</st> <st c="45747">可靠的端到端网页应用测试工具。</st> <st c="45797">它支持所有现代渲染引擎、本地移动模拟、自动等待、网页优先的断言、追踪和完全隔离。</st> <st c="45926">Playwright 使用其</st> <st c="45971">自有 API 与浏览器进行交互。</st>

+   **<st c="45979">Appium.io 用于移动应用程序测试</st>**<st c="46020">：Appium 是一个免费开放源代码的软件套件和生态系统，旨在促进跨各种应用平台的用户界面自动化。</st> <st c="46064">这包括移动操作系统（iOS、Android 和 Tizen）、网页浏览器（Chrome、Firefox 和 Safari）、桌面环境（macOS 和 Windows）以及电视操作系统（Roku、tvOS、Android TV 和 Samsung）。</st> <st c="46192">其目标是使任何移动应用程序的自动化成为可能，无论使用什么编程语言或测试框架，完全访问后端</st> **<st c="46558">应用程序编程接口</st>** <st c="46592">(</st>**<st c="46594">APIs</st>**<st c="46598">)和</st> **<st c="46605">数据库</st>** <st c="46614">(</st>**<st c="46616">DBs</st>**<st c="46619">)都可以在</st> <st c="46633">测试脚本中进行访问。</st>

+   <st c="46646">Cypress 是一个基于 JavaScript 的端到端测试</st> <st c="46696">框架，建立在 Mocha 上。</st> <st c="46723">它旨在简化为 Web 应用程序设置、编写、运行和调试测试的过程。</st> <st c="46837">Cypress 可以测试任何在 Web 浏览器中运行的内容，并特别擅长处理现代 JavaScript 框架。</st> <st c="46956">Cypress 的一些关键优势包括更快且更可靠的测试、减少的故障率、支持详细报告的仪表盘、并行</st> <st c="47103">测试</st> <st c="47109">执行，以及支持</st> **<st c="47136">行为驱动开发</st>** <st c="47163">(</st>**<st c="47165">BDD</st>**<st c="47168">) 和</st> **<st c="47175">测试驱动开发</st>** <st c="47198">(</st>**<st c="47200">TDD</st>**<st c="47203">) 测试。</st>

+   <st c="47214">JMeter 是由 Apache 软件基金会开发的开源工具，用于负载测试和分析各种服务的性能。</st> <st c="47254">JMeter 是用 Java 编写的，可以用于测试各种应用程序和协议的性能和功能行为，例如</st> **<st c="47499">HTTP</st>** <st c="47503">(</st>**<st c="47505">超文本传输协议</st>**<st c="47532">)，</st> **<st c="47536">SOAP</st>** <st c="47540">(</st>**<st c="47542">简单对象访问协议</st>**<st c="47571">)，</st> **<st c="47575">FTP</st>** <st c="47578">(</st>**<st c="47580">文件传输协议</st>**<st c="47602">)，以及</st> **<st c="47610">LDAP</st>** <st c="47614">(</st>**<st c="47616">轻量级目录访问协议</st>**<st c="47653">)。</st> <st c="47657">它</st> <st c="47659">提供了一个功能齐全的 IDE、动态 HTML 报告、多线程框架以及一个高度可扩展的核心，支持可插拔的采样器</st> <st c="47791">和插件。</st>

<st c="47803">测试 Power Automate 云流和桌面流是比在浏览器或移动设备上执行 UI 自动化测试更复杂的工程挑战。</st> <st c="47965">截至写作时，没有推荐的框架来实现针对我们的云流和桌面流的集成和系统测试。</st> <st c="48100">由于这些组件被视为业务逻辑层的一部分，我们可以通过它们前面的 UI 组件进行测试，作为</st> <st c="48239">端到端测试。</st>

<st c="48256">相反，我们可以通过调用其相应的 REST API 端点并将其结果与我们</st> <st c="48448">期望的结果进行比较，提供部分解决方案来测试 HTTP 触发和 Webhook 触发的云流。</st>

<st c="48458">正如我们在</st> *<st c="48476">第五章</st>*<st c="48485">中学习到的，</st> `<st c="48615">OnDeploymentCompleted</st>` <st c="48636">事件可以用于测试我们在部署到目标环境后</st> <st c="48696">的解决方案，该环境被表示为一个部署</st> <st c="48751">阶段。</st> <st c="48759">我们的 Power Automate</st> <st c="48777">云流作为该事件的事件处理程序，可以调用 Azure 管道或 GitHub 工作流执行代码优先的专业开发测试自动化脚本（类似于我们在</st> *<st c="48982">第五章</st>*<st c="48991">中如何调用 GitHub 工作流）来将我们的解决方案提交到 Git 仓库。</st> <st c="49038">下一个部署阶段的预审批门（</st>`<st c="49085">OnApprovalStarted</st>` <st c="49103">或</st> `<st c="49107">OnPredeploymentStarted</st>`<st c="49129">）可以检查前一个环境中测试执行的结果。</st> <st c="49210">如果测试通过，部署到下一个环境</st> <st c="49264">可以继续进行。</st>

<st c="49277">最后，我们可以使用其他</st> <st c="49303">Microsoft Azure 服务，如</st> **<st c="49342">Azure 负载测试</st>** <st c="49361">服务，来对我们的</st> **<st c="49383">JMeter</st>** <st c="49389">进行测试</st> <st c="49406">Power Platform 解决方案。</st> <st c="49433">JMeter 还提供录制功能，而 Azure 负载测试服务可以模拟数百个并行用户同时与</st> <st c="49593">我们的应用进行交互。</st>

<st c="49610">根据解决方案的类型，我们需要选择具有适当深度和广度的测试工具。</st> <st c="49709">我们需要设计和交付的业务或任务关键应用越多，我们就应该在这一</st> <st c="49841">DevOps 领域投入更多的时间和精力。</st>

# <st c="49859">总结</st>

<st c="49867">在本章中，我们探讨了 DevOps CI/CD 流程的高级模式，并学习了如何建立一个强大的分支策略，反映我们解决方案的开发、测试和生产环境。</st> <st c="50088">我们检查了许多自动化创建分支、使用服务连接启动 Power Platform 环境、以及使用各种 Pac</st> <st c="50271">CLI 脚本导入和导出解决方案的示例。</st>

<st c="50289">我们还深入了解了 Power Platform 目录，作为 Power Platform 的包管理解决方案。</st> <st c="50392">此外，我们还了解了 Power Platform 的 ALM 加速器，以及该解决方案如何使用管道模板、分支策略和环境管理。</st> <st c="50559">最后，我们熟悉了可以与 Power</st> <st c="50651">Platform 解决方案一起使用的测试自动化框架。</st>

<st c="50670">在下一章中，我们将更进一步，学习在</st> <st c="50787">Power Platform 上实现 DevSecOps 流程。</st>

# <st c="50802">进一步阅读</st>

+   <st c="50818">`az devops`</st> <st c="50832">CLI：</st> [<st c="50838">https://learn.microsoft.com/en-us/cli/azure/devops?view=azure-cli-latest</st>](https://learn.microsoft.com/en-us/cli/azure/devops?view=azure-cli-latest)

+   <st c="50910">Windows Subsystem for</st> <st c="50933">Linux：</st> [<st c="50940">https://learn.microsoft.com/en-us/training/modules/wsl-introduction/</st>](https://learn.microsoft.com/en-us/training/modules/wsl-introduction/)

+   <st c="51008">Azure DevOps 批准</st> <st c="51031">流程：</st> [<st c="51037">https://learn.microsoft.com/en-us/azure/devops/pipelines/process/approvals?view=azure-devops&tabs=check-pass</st>](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/approvals?view=azure-devops&tabs=check-pass)

+   <st c="51145">Power Platform</st> <st c="51165">目录：</st> [<st c="51174">https://learn.microsoft.com/en-us/power-platform/developer/catalog</st>](https://learn.microsoft.com/en-us/power-platform/developer/catalog)

+   <st c="51240">Azure DevOps 管道</st> <st c="51263">模板：</st> [<st c="51274">https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops&pivots=templates-includes</st>](https://learn.microsoft.com/en-us/azure/devops/pipelines/process/templates?view=azure-devops&pivots=templates-includes)

+   <st c="51392">GitHub 启动</st> <st c="51408">工作流：</st> [<st c="51415">https://docs.github.com/en/actions/using-workflows/creating-starter-workflows-for-your-organization</st>](https://docs.github.com/en/actions/using-workflows/creating-starter-workflows-for-your-organization)

+   <st c="51514">GitHub 可重用</st> <st c="51531">工作流：</st> [<st c="51542">https://docs.github.com/en/actions/using-workflows/reusing-workflows</st>](https://docs.github.com/en/actions/using-workflows/reusing-workflows)

+   <st c="51610">Power</st> <st c="51641">Platform 的 ALM 加速器：</st> [<st c="51651">https://learn.microsoft.com/en-us/power-platform/guidance/alm-accelerator/overview</st>](https://learn.microsoft.com/en-us/power-platform/guidance/alm-accelerator/overview)

+   <st c="51733">ALM 的源代码</st> <st c="51757">加速器：</st> [<st c="51770">https://github.com/microsoft/coe-starter-kit/tree/main/CenterofExcellenceALMAccelerator</st>](https://github.com/microsoft/coe-starter-kit/tree/main/CenterofExcellenceALMAccelerator)

+   <st c="51857">ALM 加速器管道</st> <st c="51883">模板：</st> [<st c="51894">https://github.com/microsoft/coe-alm-accelerator-templates/tree/main</st>](https://github.com/microsoft/coe-alm-accelerator-templates/tree/main)

+   Power Apps 测试<st c="51962">工作室</st>: [<st c="51987">https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/test-studio</st>](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/test-studio)

+   Visual Studio 单元<st c="52059">测试</st>: [<st c="52088">https://learn.microsoft.com/en-us/visualstudio/test/getting-started-with-unit-testing?view=vs-2022&tabs=dotnet%2Cmstest</st>](https://learn.microsoft.com/en-us/visualstudio/test/getting-started-with-unit-testing?view=vs-2022&tabs=dotnet%2Cmstest)

+   使用 Azure DevOps 自动化测试的<st c="52207">管道</st>: [<st c="52252">https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/test-studio-classic-pipeline-editor</st>](https://docs.microsoft.com/en-us/powerapps/maker/canvas-apps/test-studio-classic-pipeline-editor)

+   PowerApps 测试<st c="52348">引擎</st>: [<st c="52372">https://learn.microsoft.com/en-us/power-apps/developer/test-engine/overview</st>](https://learn.microsoft.com/en-us/power-apps/developer/test-engine/overview)

+   Azure 负载测试<st c="52447">服务</st>: [<st c="52480">https://learn.microsoft.com/en-us/azure/load-testing/overview-what-is-azure-load-testing</st>](https://learn.microsoft.com/en-us/azure/load-testing/overview-what-is-azure-load-testing)

+   Microsoft Copilot Studio 测试<st c="52568">框架</st>: [<st c="52610">https://powervirtualagents.microsoft.com/en-us/blog/automate-testing-of-your-power-virtual-agents-chatbots-with-the-pva-test-framework-sample-solution/</st>](https://powervirtualagents.microsoft.com/en-us/blog/automate-testing-of-your-power-virtual-agents-chatbots-with-the-pva-test-framework-sample-solution/)

+   Microsoft Copilot Studio 实施<st c="52761">指南</st>: [<st c="52809">https://aka.ms/CopilotStudioImplementationGuide</st>](https://aka.ms/CopilotStudioImplementationGuide)
