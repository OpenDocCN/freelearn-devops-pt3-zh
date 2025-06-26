

# 使用 Bitbucket Pipelines 部署

在了解 Bitbucket Pipelines 的过程中，我们从 `bitbucket-pipelines.yml` 文件的概念开始。

我们现在处于部署阶段，需要将之前在 Bitbucket Pipelines 中创建的构建安装到我们的某个环境中，从测试环境到预发布环境，最后到生产环境。如果我们扩展 Bitbucket Pipelines 以实现自动部署，我们必须考虑 **持续部署**。

本章中，我们将查看如何在 `bitbucket-``pipelines``.yml` 中进行必要的添加以执行持续部署。为此，我们将涵盖以下配方：

+   配置部署

+   将工件推送到 Bitbucket 仓库

+   将工件推送到工件仓库工具

+   部署工件到 Bitbucket 下载

+   使用 **Secure Copy** **Protocol** (**SCP**) 部署工件

+   将工件部署到 AWS S3 存储桶

+   将工件部署到 AWS Lambda

+   部署工件到 Google Cloud

+   部署工件到 Microsoft Azure

+   在部署阶段使用 Ansible

+   在部署阶段使用 Terraform

# 技术要求

因为我们仍在使用 Bitbucket Pipelines，所以需要订阅 Bitbucket Cloud。对于将工件最终放入代码库或 Bitbucket 下载的配方，仍然需要此订阅。

将工件部署到 AWS、Google Cloud 和 Microsoft Azure 等云环境，需要在本章的配方中使用的服务拥有权限的账户。

本章节的示例代码可以在本书 GitHub 仓库的`Chapter8`文件夹中找到：[`github.com/PacktPublishing/Atlassian-DevOps-Toolchain-Cookbook/tree/main/Chapter8`](https://github.com/PacktPublishing/Atlassian-DevOps-Toolchain-Cookbook/tree/main/Chapter8)

现在，让我们来探索如何为部署准备构建。

# 配置部署

尽管 Bitbucket Pipelines 可以部署到多个不同的平台，但部署的特点是相同的。这在你部署到测试环境时非常有利，你可能需要定义一些运行时测试或包测试，而在生产环境中，测试已经完成，可以发布新的功能给最终用户。

有了这个考虑，我们来看看部署的共同点是什么，以及如何统一定义它们。之后，我们将学习如何为典型 DevSecOps 流程中的不同环境配置部署。

## 准备工作

在设置管道之前，我们需要定义部署环境。你可以使用默认的环境，它们有以下预定义设置：

+   测试环境

+   Staging（预发布环境）

+   生产环境

你还可以更改环境的名称或为部署配置特定于环境的变量。要设置环境配置，请执行以下步骤：

1.  在 Bitbucket 中，进入感兴趣的代码库并选择 **Repository settings**（代码库设置）。

![图 8.1 – 选择仓库设置](img/B21937_08_01.jpg)

图 8.1 – 选择仓库设置

1.  在**流水线**部分，选择**部署**选项。

![图 8.2 – 选择部署](img/B21937_08_02.jpg)

图 8.2 – 选择部署

1.  默认环境列表将显示，分为**测试**、**预发布**和**生产**。

![图 8.3 – 部署环境](img/B21937_08_03.jpg)

图 8.3 – 部署环境

1.  要更改现有环境，点击代表该环境的条带。它将展开，显示您可以用来更改名称的选项以及一个用于环境特定变量的部分。

![图 8.4 – 环境的配置选项](img/B21937_08_04.jpg)

图 8.4 – 环境的配置选项

要更改名称，在**环境名称：**字段中覆盖名称。要添加变量，请填写**名称**和**值**字段。如果变量包含密码或密钥等秘密信息，请确保选中**已加密**复选框。最后点击**添加**按钮。

1.  要添加一个新环境，进入定义环境类型（**测试**、**预发布**或**生产**）的部分，并点击**添加** **环境**按钮。

1.  填写新环境的名称，并选择**预发布** **环境**复选框。

![图 8.5 – 添加一个环境](img/B21937_08_05.jpg)

图 8.5 – 添加一个环境

在 Bitbucket 的免费计划中，您可以定义 10 个环境。在 Bitbucket 的高级版本中，这个限制增加到 100。Bitbucket 的高级版本允许您限制哪些分支可以部署到该环境，并且可以设置只有管理员才能在该环境上执行任何部署。

现在我们已经配置好了环境，是时候配置部署了。

## 如何做...

要定义一个部署，我们必须在 `bitbucket-pipelines.yml` 文件中引入新的关键字和部分。我们将在流水线的阶段或步骤中应用这些关键字和部分。

一旦完成此配置，我们可以根据接收来自下级分支合并的分支来定义单独的部署。这使我们能够不断地进行部署和测试，直到最终的拉取请求提交到主分支，在生产环境中可能会进行部署。

让我们看看如何执行这些部署。

### 在 bitbucket-pipelines.yml 中配置部署

部署指令在 `bitbucket-pipelines.yml` 文件的阶段或步骤中通过 `deployment:` 关键字标识，并指定部署步骤或步骤的目标环境。让我们学习如何使用 `deployment:` 关键字：

1.  `deployment:` 关键字在步骤中用于标识正在部署的环境。以下代码片段中展示了一个示例：

    ```
    pipelines:
      default:
        - step:
            name: Deploy to staging
            deployment: staging
            script:
              - python deploy.py staging_1
    ```

    你可以为部署定义多个步骤，每个步骤都会标明环境。然而，Bitbucket 要求环境类型按以下顺序排列：

    1.  测试环境。

    1.  阶段环境

    1.  生产环境

1.  如果部署需要多个步骤，可以在阶段定义中使用`deployment:`关键字来标识目标环境。以下代码片段展示了这一示例：

    ```
    pipelines:
       default:
          - stage:
                name: Deploy to Production
                deployment: production
                steps:
                    - step:
                       name: Basic deploy first step
                       script:
                             - sh ./deploy1.sh
                    - step:
                        name: Basic deploy second step
                        script:
                           - sh ./deploy2.sh
    ```

1.  如果你希望部署步骤被手动触发，请在部署步骤中添加`trigger: manual`关键字。以下代码片段展示了这一操作：

    ```
    - step:
          name: Deploy to staging
          deployment: staging
          trigger: manual
          script:
             - python stage_deploy.py staging
    ```

一旦我们在`bitbucket-pipelines.yml`文件中定义了部署步骤或阶段，我们就可以在 Bitbucket 中监控部署的进度和结果。我们来看看如何操作。

### 监控部署

一旦你执行了部署，你可以在部署仪表板中跟踪其进度或手动执行部署。我们来看看部署仪表板的功能：

1.  正如我们在前面的配方中看到的，要访问部署仪表板，请在仓库级别的侧边栏中选择**部署**。

1.  部署仪表板将出现在侧边栏的右侧。它被划分为在`bitbucket-pipelines.yml`文件中定义的环境。在该环境内，将显示一个卡片，展示该环境中最后一次成功的部署。

![图 8.6 – 部署仪表板](img/B21937_08_06.jpg)

图 8.6 – 部署仪表板

1.  点击环境中的卡片，查看该环境的部署详情。详情包括导致`bitbucket-pipelines.yml`执行的提交，现有版本与提交之间的差异说明，以及推送到关联环境的内容，包括部署历史。如果 Jira 和 Bitbucket 已连接，正如我们在*第一章*的*连接 Bitbucket*配方中讨论的，你还将看到与提交相关的 Jira 问题。以下截图展示了部署到**测试**环境。

![图 8.7 – 部署到测试环境](img/B21937_08_07.jpg)

图 8.7 – 部署到测试环境

1.  如果部署被定义为手动触发，你将在最后一次成功部署到较低环境的卡片上看到一个**提升**按钮。点击**提升**按钮即可执行部署到更高的环境。

![图 8.8 – 将手动部署提升到生产环境](img/B21937_08_08.jpg)

图 8.8 – 将手动部署提升到生产环境

1.  如果部署失败，你可以从部署仪表板重新部署上次成功的部署。重新部署的能力要求满足以下条件：

    +   管道中的初始部署步骤已成功执行

    +   部署权限设置为允许重新部署该步骤（请注意，这仅在 Bitbucket 的高级计划中提供）。

    +   用于部署的产物不能过期。

1.  一旦部署到特定环境被启动，来自其他 Pipelines 的任何后续部署都会被暂停，这得益于**并发控制**，它将每个环境的部署限制为一次。原始部署完成后，您可以对暂停的部署进行以下操作：

    +   从头开始重新运行 Pipeline

    +   从暂停处恢复 Pipeline

现在我们了解了部署的机制，如果与特定分支相关联的拉取请求成功，我们就可以允许部署到特定环境。让我们看看如何做到这一点。

### 从拉取请求或分支进行部署

如你从*第六章*的*条件执行 Pipelines*的配方中所记得的，你可以设置你的 Pipelines，根据接收新提交的分支或是否有拉取请求被创建，执行不同的指令。让我们学习如何建立这些不同的条件：

1.  分支 Pipeline 指定在特定分支上发生推送时需要执行的操作。您可以使用 `branches:` 关键字来描述分支 Pipeline 上发生的操作。这些操作会在分支名称下分组。以下代码片段详细说明了使用 `default:` 关键字时，针对其他分支的部署操作，以及为暂存分支设定的独立操作：

    ```
    pipelines:
       default:
         - step:
             script:
               - echo "We do this on all branches except staging"
       branches:
         staging:
           - step:
               deployment: staging
               script:
                 - echo "Done on the staging branch"
                 - python deploy.py
    ```

1.  拉取请求 Pipeline 允许您指定在特定分支上创建拉取请求时执行的操作。这些操作在 `pull-requests:` 关键字下的分支中定义。让我们来看一下当拉取请求发生在暂存分支时完成的操作：

    ```
    pipelines:
       pull-requests:
         staging/*:
            - step:
                deployment: staging
                script:
                  - echo "Test and deploy pre-merge"
                  - python deploy.py
    ```

通过这些内容，我们已经学会了如何在功能分支发生更改时轻松进行测试和部署。当从暂存分支到生产分支的拉取请求被创建时，自动部署到生产环境是可能的。

现在我们已经了解了如何配置 Bitbucket Pipelines 进行部署，接下来我们将查看根据目标执行特定部署的示例。首先，我们将学习如何将部署内容推送到 Bitbucket git 仓库本身。

# 将构建产物推送到 Bitbucket 仓库

部署过程的一部分可能是将构建产物放置到正确的仓库中。最方便的存储构建产物的仓库可能就是 Bitbucket 本身。通常不推荐这么做，因为构建产物通常会占用大量存储空间，并可能导致底层 git 工具的性能问题。不过，如果没有其他替代方案，我们提供了这个方法。

让我们学习如何将构建产物推送回 Bitbucket 仓库，作为部署的一部分。

## 准备工作

尽管推荐将内容推送到 git 仓库的方式是使用 HTTP，但有时唯一可行的方式是使用 SSH。为了做到这一点，你需要设置你的凭据。如果你在仓库上启用了分支权限，或者想为这些操作设置一个自动化账户，那么这种情况是适用的。一个重要的考虑因素是，配置这些账户后，会取消对它们能访问的限制。

让我们学习如何创建具有不同认证方式的账户。

### 使用 OAuth 进行认证

你可以使用 OAuth 作为访问 git 仓库的认证方式。这涉及到创建一个账户，并通过分支权限授予该账户对主分支或 master 分支的写入权限。让我们学习如何使用 OAuth 设置一个账户：

1.  在 Bitbucket 的**工作区**页面，点击**设置**齿轮图标，选择**工作区设置**。

![图 8.9 – 选择工作区设置](img/B21937_08_09.jpg)

图 8.9 – 选择工作区设置

1.  在**工作区设置**的侧边栏中，找到**应用程序和功能**部分，选择**OAuth 消费者**。

![图 8.10 – 选择 OAuth 消费者](img/B21937_08_10.jpg)

图 8.10 – 选择 OAuth 消费者

1.  在**OAuth 消费者**页面，点击**添加** **消费者**按钮：

![图 8.11 – 添加消费者](img/B21937_08_11.jpg)

图 8.11 – 添加消费者

1.  在后续页面中，填写以下详细信息：

    +   **名称**

    +   设置**回调 URL**为 [`bitbucket.org`](https://bitbucket.org)

)

    +   确保勾选**这是一个私人消费者**

1.  对于权限，确保在**仓库**下勾选了**读取**和**写入**权限。点击**保存**按钮保存 OAuth 消费者：

1.  返回到消费者页面，找到你创建的新消费者。将密钥和密钥保存在安全的管道变量中。将 `CLIENT_ID` 用作密钥，`CLIENT_SECRET` 用作密钥：

![图 8.12 – OAuth 消费者密钥和密钥](img/B21937_08_12.jpg)

图 8.12 – OAuth 消费者密钥和密钥

1.  在 `bitbucket-pipelines.yml` 文件的 `script` 部分添加以下代码片段，在 git 命令之前进行更改并提交到 git 仓库。我们假设你的运行器已经安装了 `curl` 和 `jq` 工具：

    ```
    - >
      export access_token=$(curl -s -X POST -u "${CLIENT_ID}:${CLIENT_SECRET}" \
        https://bitbucket.org/site/oauth2/access_token \
        -d grant_type=client_credentials -d scopes="repository"| jq --raw-output '.access_token')
    # Configure git to use the oauth token.
    - git remote set-url origin https://x-token-auth:${access_token}@bitbucket.org/${BITBUCKET_REPO_OWNER}/${BITBUCKET_REPO_SLUG}
    ```

这样，你就配置了一个 OAuth 消费者，并允许它从 Bitbucket Pipelines 进行 git 提交。

我们也可以使用 Bitbucket UI 设置 SSH 密钥。让我们看看如何操作。

### 为 Bitbucket Pipelines 创建 SSH 密钥对

对于给定的仓库，你可以创建一个 SSH 密钥对，并将公钥保存在 Bitbucket 中，以便可以使用 SSH 将其部署回 git 仓库。让我们看看这是如何实现的：

1.  在仓库页面，选择侧边栏中的**仓库设置**：

1.  在**仓库设置**侧边栏的**Pipelines**部分，选择**SSH 密钥**：

![图 8.13 – 选择 SSH 密钥](img/B21937_08_13.jpg)

图 8.13 – 选择 SSH 密钥

1.  在**SSH 密钥**页面，选择**生成密钥**：

![图 8.14 – 生成密钥](img/B21937_08_14.jpg)

图 8.14 – 生成密钥

1.  复制生成的公钥：

![图 8.15 – 复制公钥](img/B21937_08_15.jpg)

图 8.15 – 复制公钥

1.  此时，您需要将公钥添加到您的个人 Bitbucket 设置中。首先，选择**设置**图标，然后点击**个人** **Bitbucket 设置**：

1.  在**个人设置**侧边栏的**安全性**部分，选择**SSH 密钥**：

![图 8.16 – 选择 SSH 密钥](img/B21937_08_16.jpg)

图 8.16 – 选择 SSH 密钥

1.  在**SSH** **密钥**页面上选择**添加密钥**：

![图 8.17 – 添加密钥](img/B21937_08_17.jpg)

图 8.17 – 添加密钥

1.  在**添加 SSH 密钥**弹窗中，粘贴公钥并选择**添加密钥**：

![图 8.18 – 粘贴并添加公钥](img/B21937_08_18.jpg)

图 8.18 – 粘贴并添加公钥

1.  以下代码行需要添加到 `bitbucket-pipelines.yml` 文件的 `script` 部分，在 git 命令之前，以便您可以将更改保存并提交回 git 仓库。此行配置 git 使用 SSH。`BITBUCKET_GIT_SSH_ORIGIN` 是一个默认的环境变量：

    ```
    git remote set-url origin ${BITBUCKET_GIT_SSH_ORIGIN}
    ```

完成这些配置后，您已经设置了 SSH 密钥，在将提交推送回 git 仓库时进行身份验证。

另一种身份验证方法是创建一个应用密码，并将其作为安全变量传递。让我们来看看如何操作。

### 创建应用密码

应用密码是您可以在 Bitbucket 中安全使用的个人秘密，用于自动化功能。让我们看看如何创建一个应用密码：

1.  选择**设置**图标并选择**个人** **Bitbucket 设置**。

1.  在**个人设置**侧边栏的**访问管理**部分，选择**应用密码**。

1.  在**应用密码**页面上，选择**创建** **应用密码**：

![图 8.19 – 创建应用密码](img/B21937_08_19.jpg)

图 8.19 – 创建应用密码

1.  在**添加应用密码**页面上，为应用密码命名，并确保在**仓库**部分选中**读取**和**写入**权限。完成后，点击**创建**：

1.  应用密码将在弹窗中显示。复制该值，并确保它存放在安全的位置：

![图 8.20 – 生成的应用程序密码](img/B21937_08_20.jpg)

图 8.20 – 生成的应用程序密码

一个安全的位置示例是安全变量。将应用密码设置为**安全的仓库变量**或**安全的工作区变量**：

![图 8.21 – 将应用密码添加为安全的仓库变量](img/B21937_08_21.jpg)

图 8.21 – 将应用密码添加为安全的仓库变量

1.  以下代码行需要添加到 `bitbucket-pipelines.yml` 文件的 `script` 部分，在 git 命令之前，以便您可以将更改保存并提交回 git 仓库。此行配置 git 远程 URL，使用包含的用户名和应用密码进行身份验证：

    ```
    git remote set-url origin https://<your username>:${APP_SECRET}@bitbucket.org/${BITBUCKET_REPO_OWNER}/${BITBUCKET_REPO_SLUG}
    ```

到目前为止，我们已经学习了在提交更改到 git 仓库时如何使用几种身份验证方法。现在，让我们看看需要添加哪些内容来执行实际的提交。

## 如何实现…

如果你正在使用预配置的 HTTP git 源来推送更改到 git 仓库，或者已经使用上述某种方法设置了身份验证，那么你需要做的唯一事情就是定义执行实际 git 提交和推送的脚本步骤。我们来看看如何实现：

1.  要提交更改，将 `git add`、`git commit` 和 `git push` 命令添加到 `bitbucket-pipelines.yml` 文件的 `script` 部分。请注意，关于 git 提交消息，你可以添加 `[skip ci]` 以避免管道执行的无限循环。以下代码片段展示了一个示例：

    ```
    pipelines:
       default:
          - step:
              script:
                 - git add <changed files>
                 - git commit -m "[skip ci] Updates added via Bitbucket Pipelines deploy"
                 - git push
    ```

1.  如果你使用标签，可以添加 `git tag` 命令来创建新的标签。代码片段将类似于以下内容。在此示例中，由于我们引用了 `BITBUCKET_BUILD_NUMBER` 预定义变量，标签将包含构建号：

    ```
    pipelines:
       default:
          - step:
              script:
                 - git add <changed files>
                 - git commit -m "[skip ci] Updates added via Bitbucket Pipelines deploy"
                 - git tag -am "Tag for release ${BITBUCKET_BUILD_NUMBER}" release-${BITBUCKET_BUILD_NUMBER}
                 - git push
    ```

我们现在已经看到如何使用 git 仓库作为构建工件的存储库。坦率地说，当有像 JFrog 的 Artifactory 和 Sonatype 的 Nexus 这样的专用构件库工具时，这种方法会显得有些繁琐。在接下来的步骤中，我们将学习如何将构建工件部署到这些工具中进行存储和跟踪。

# 将构件推送到构件库工具中

像 Artifactory 和 Nexus 这样的构件库工具支持配置管理，这是一种将从持续集成管道中生成的构建工件存储并与其应用的环境进行跟踪的实践。

通过 Bitbucket Pipes，基本构建可以推送到 Artifactory 或 Nexus，这使你可以轻松地从 `bitbucket-pipelines.yml` 文件进行部署。更复杂的涉及 Maven 或 npm 的部署将在这里不予讨论，但我们会提供一些参考，以创建这些类型的部署。

## 准备工作

在使用 JFrog Artifactory 或 Sonatype Nexus 的 Pipes 时，有一些先决步骤需要定义。我们将逐一查看每个工具所需的步骤。

### JFrog CLI 先决条件

JFrog Setup CLI Pipe 需要连接到 JFrog 平台服务器。让我们看看这个过程中涉及的步骤：

1.  确保运行器已安装 JFrog CLI。在编写本文时，版本应为 `2.17.0` 或更高。以下命令适用于 macOS 和 Linux 运行器，假设已安装 `curl`：

    ```
    curl -fL "https://getcli.jfrog.io?setup" | sh
    ```

1.  如果你正在使用 Windows 运行器，请使用以下命令。请注意，这里使用的是 PowerShell：

    ```
    powershell "Start-Process -Wait -Verb RunAs powershell '-NoProfile iwr https://releases.jfrog.io/artifactory/jfrog-cli/v2-jf/[RELEASE]/jfrog-cli-windows-amd64/jf.exe -OutFile $env:SYSTEMROOT\system32\jf.exe'" ; jf setup
    ```

1.  使用 JFrog CLI 连接到你的 JFrog 平台服务器。要设置 JFrog 平台实例的详细信息，请运行以下命令：

    ```
    JFROG_CLI_SERVER_ID.
    ```

1.  现在，可以方便地设置与 JFrog 环境变量相对应的 Bitbucket 安全变量，变量名以 `JF_ENV_` 开头，键为环境变量名，值为服务器令牌。你可以通过以下 JFrog CLI 命令获取服务器令牌：

    ```
    jf c export <server ID from previous instruction>
    ```

到此为止，我们已经覆盖了 JFrog CLI 的初步设置。现在，我们来看一下 Nexus 需要的内容。

### Sonatype Nexus 先决条件

Sonatype Nexus 发布器的管道需要定义一些环境变量。某些变量是必填的。需要定义的变量如下：

+   `FILENAME`（必填）：这是要发布的文件路径。

+   `ATTRIBUTES`（必填）：Nexus 发布器所需的属性。组件属性以 `-C` 表示，资产属性以 `-A` 表示。

+   `USERNAME`（必填）：Nexus 用户名。

+   `PASSWORD`（必填）：Nexus 密码。

+   `SERVER_URL`（必填）：Nexus 服务器 URL。

+   `REPOSITORY`：Nexus 中的仓库名称。默认值是 `maven-releases`。

+   `FORMAT`：制品格式。默认值是 `maven2`。

既然先决条件已经处理完毕，接下来我们来看一下如何使用管道推送我们的制品。

## 如何操作……

在 *第六章* 的 *连接到 Bitbucket 管道* 章节中，我们了解了管道。管道作为 Bitbucket 管道中的第三方工具集成点。

现在，我们已经准备好通过管道连接到 JFrog 或 Sonatype Nexus，并设置部署步骤。让我们来看看如何为每个工具执行这项操作。

### 使用 JFrog Setup CLI 管道

你可以使用 JFrog Setup CLI 管道连接到 JFrog 平台上的任何 JFrog 工具，包括 Artifactory 和 XRay。我们来看一下相关步骤：

1.  将以下几行添加到 `bitbucket-pipelines.yml` 文件的 `script` 部分：

    ```
    script:
       - pipe: jfrog/jfrog-setup-cli:2.0.0
       - source ./jfrog-setup-cli.sh
    ```

1.  一旦添加了设置行，你就可以使用任何 JFrog CLI 命令进行更改。以下代码片段是一个示例，展示了一个构建过程，该过程通过 JFrog CLI 命令将多个命令发送到 Artifactory：

    ```
    script:
       - pipe: jfrog/jfrog-setup-cli:2.0.0
       - source ./jfrog-setup-cli.sh
       # Upload artifacts to Artifactory
       - jf rt u file artifacts/
       # Collect environment variables
       - jf rt bce
       # Publish build info
       - jf rt bp
    ```

到此为止，我们已经学会了如何使用管道和 JFrog CLI 命令与 JFrog 工具进行连接和通信。现在，我们来看一下 Sonatype Nexus 管理器的过程。

### 使用 Sonatype Nexus 发布器管道

让我们学习如何使用管道连接到 Sonatype Nexus 发布器：

1.  将以下代码片段添加到 `bitbucket-pipelines.yml` 文件的 `script` 部分。你可以直接在代码中填写变量，或者使用 Bitbucket 变量：

    ```
    - pipe: sonatype/nexus-repository-publish:0.0.1
      variables:
        FILENAME: '<string>'
        ATTRIBUTES: '<string>'
        USERNAME: '<string>'
        PASSWORD: '<string>'
        SERVER_URL: '<string>'
        # REPOSITORY: '<string>' # Optional.
        # FORMAT: '<string>' # Optional.
    ```

1.  这是一个示例：

    ```
        - step:
            # set NEXUS_USERNAME and NEXUS_PASSWORD as environment variables
            name: Deploy to Nexus Repository Manager
            deployment: test   # set to test, staging or production
            # trigger: manual  # uncomment to have a manual step
            script:
              - pipe: sonatype/nexus-repository-publish:0.0.1
                variables:
                  FILENAME: 'target/myapp-1.0-SNAPSHOT.jar'
                  ATTRIBUTES: '-CgroupId=com.example -CartifactId=myapp -Cversion=1.0 -Aextension=jar'
                  USERNAME: '$NEXUS_USERNAME'
                  PASSWORD: '$NEXUS_PASSWORD'
                  SERVER_URL: 'https://nexus.example.com/'
    ```

到目前为止，我们已经看到使用管道进行轻松部署是多么简单。本章后续的其他示例也将展示管道的简便性。

## 另见

以下是一些其他参考资料链接，如果你需要了解更多关于如何连接到 JFrog 或 Sonatype Nexus 的信息：

+   JFrog CLI 文档：[`docs.jfrog-applications.jfrog.io/jfrog-applications/jfrog-cli`](https://docs.jfrog-applications.jfrog.io/jfrog-applications/jfrog-cli)

+   使用 Bitbucket Pipelines 进行 Maven 部署的示例，使用**GNU 隐私保护**（**GPG**）和**开源软件仓库托管**（**OSSRH**）：[`bitbucket.org/simpligility/ossrh-pipeline-demo/src/master/`](https://bitbucket.org/simpligility/ossrh-pipeline-demo/src/master/)

接下来，我们将看几个将构建构件安装到目标环境的示例。我们将从查看各种上传过程开始。

# 将构件部署到 Bitbucket 下载区

你可以通过配置 Bitbucket 下载管道，将 Bitbucket 本身作为构建构件的暂存区。这使你能够将构建构件推送到 Bitbucket 下载区。我们将在本食谱中更详细地了解这一过程。

## 准备就绪

使用`bitbucket-upload-file`管道需要通过用户名和应用密码或访问令牌进行身份验证。我们在*准备就绪*部分中查看了如何创建应用密码，这部分内容出现在*将构件推送到 Bitbucket* *仓库*的食谱中。

访问令牌在所有 Bitbucket 计划的仓库级别都可用，并且在 Bitbucket 的 Premium 计划下，项目和工作区级别也可以使用。令牌仅限于为其创建的仓库、项目或工作区。这些令牌是一次性使用的，如果需要替换，则会被撤销。让我们学习如何创建一个仓库访问令牌：

1.  在仓库级别，选择侧边栏中的**仓库设置**。

1.  在**仓库设置**侧边栏中，选择**安全**部分中的**访问令牌**：

![图 8.22 – 选择访问令牌](img/B21937_08_22.jpg)

图 8.22 – 选择访问令牌

1.  在**访问令牌**页面，选择**创建仓库** **访问令牌**：

1.  在弹窗中，为令牌命名并指定权限。Bitbucket Pipelines 要求**读取**和**写入**仓库权限。完成后，点击**创建**：

1.  接下来的弹窗将包含令牌的值以及相关的有用应用。通过复制令牌并粘贴到安全的位置来保存令牌值。这将是你查看令牌值的唯一机会。

1.  你可以将访问令牌作为安全的仓库变量。选择**仓库设置**侧边栏中的**仓库变量**，填写令牌的键名，并将之前复制的令牌值粘贴到**值**部分。一旦完成，勾选**已加密**复选框并点击**添加**：

![图 8.23 – 仓库变量](img/B21937_08_23.jpg)

图 8.23 – 仓库变量

现在我们已经建立了身份验证方式，无论是通过用户名/应用密码还是访问令牌，接下来让我们设置管道以将文件上传到 Bitbucket 下载区。

## 如何操作…

`bitbucket-upload-file`管道是将文件部署到 Bitbucket 下载区的主要方式。让我们来学习如何设置这个管道：

1.  如果你想使用用户名和应用密码进行身份验证，请使用以下代码片段：

    ```
    script:
      - pipe: atlassian/bitbucket-upload-file:0.7.1
        variables:
          BITBUCKET_USERNAME: $BITBUCKET_USERNAME
          BITBUCKET_APP_PASSWORD: $BITBUCKET_APP_PASSWORD
          FILENAME: 'package.json'
    ```

1.  如果改用访问令牌，请将 `BITBUCKET_USERNAME` 和 `BITBUCKET_APP_PASSWORD` 替换为 `BITBUCKET_ACCESS_TOKEN`。以下代码片段展示了这一点：

    ```
    script:
      - pipe: atlassian/bitbucket-upload-file:0.7.1
        variables:
          BITBUCKET_ACCESS_TOKEN: $BITBUCKET_ACCESS_TOKEN
          FILENAME: 'package.json'
    ```

1.  `FILENAME` 管道变量可以通过调用通配符来指定多个文件。注意，限制为 10 个文件。以下代码展示了上传所有 `.txt` 文件的示例：

    ```
    script:
      - pipe: atlassian/bitbucket-upload-file:0.7.1
        variables:
          BITBUCKET_USERNAME: $BITBUCKET_USERNAME
          BITBUCKET_APP_PASSWORD: $BITBUCKET_APP_PASSWORD
          FILENAME: '*.txt'
    ```

1.  其他可选的管道变量允许你指定另一个帐户和仓库来上传文件。`ACCOUNT` 和 `REPOSITORY` 在以下代码片段中展示：

    ```
    script:
      - pipe: atlassian/bitbucket-upload-file:0.7.1
        variables:
          BITBUCKET_USERNAME: $BITBUCKET_USERNAME
          BITBUCKET_APP_PASSWORD: $BITBUCKET_APP_PASSWORD
          FILENAME: 'package.json'
          ACCOUNT: $PROJECT_ACCOUNT
          REPOSITORY: $ALTERNATE_REPO
    ```

到此为止，我们已经学会了如何将文件推送到 Bitbucket 下载区以便取回。现在，让我们学习如何将构建产物发送到目标服务器。

# 使用 SCP 部署构建产物

**SCP** 是在两台主机之间传输文件的一种方式。该协议使用 SSH 作为基础，安全地将文件从一台计算机移动到另一台计算机。

让我们学习如何使用 SCP 将构建产物从 Bitbucket Cloud 传输到远程主机。

## 准备工作

由于 SCP 的基础是 SSH，我们需要准备一个 SSH 密钥，并在 Bitbucket 和远程主机上配置与 SSH 相关的其他设置。让我们看一下所需的步骤：

1.  我们在**推送构建产物到 Bitbucket 仓库**这一食谱的*准备工作*部分定义了一个仓库 SSH 密钥。我们可以通过复制公钥并将其放入 `~/.ssh/authorized_keys` 文件中，使用该密钥进行远程主机的传输。如果你有远程主机的 SSH 访问权限，可以在生成密钥对的机器上运行以下命令。这确保执行此操作的用户是你：

    ```
    ssh-copy-id -i <public key file to copy> user@host
    ```

1.  我们还需要更新 Bitbucket 上的已知主机。对于仓库，位置与我们用来创建 SSH 密钥的屏幕相同。在**仓库设置**中，选择**SSH 密钥**。在**SSH 密钥**屏幕上，输入远程主机的 IP 地址并点击**获取**。

![图 8.24 – 添加已知远程主机](img/B21937_08_24.jpg)

图 8.24 – 添加已知远程主机

1.  配置你的远程主机，允许在所需端口（默认是`22`）上进行 SCP/SSH 访问，并允许使用 SSH 密钥进行访问。由于存在多种系统和配置，这一步骤留给你作为练习。

一旦我们拥有必要的配置，就可以配置 `bitbucket-pipelines.yml` 文件了。

## 如何操作...

现在，我们已经在 Bitbucket Cloud 和远程主机上配置了 SSH 密钥，接下来可以设置 Bitbucket Pipelines，将文件传输过程变成一个部署过程：

1.  将以下代码片段添加到 `bitbucket-pipelines.yml` 文件的 `script` 部分。所需的管道变量包括远程主机的用户名、远程主机名称、远程主机上部署文件的路径，以及构建产物所在的本地路径：

    ```
    - pipe: atlassian/scp-deploy:1.5.0
      variables:
        USER: '<string>'
        SERVER: '<string>'
        REMOTE_PATH: '<string>'
        LOCAL_PATH: '<string>'
        # SSH_KEY: '<string>' # Optional.
        # EXTRA_ARGS: '<string>' # Optional.
        # DEBUG: '<boolean>' # Optional.
    ```

1.  你还可以使用 `EXTRA_ARGS` 管道变量添加选项。以下代码片段展示了一个示例：

    ```
    script:
      - pipe: atlassian/scp-deploy:1.5.0
        variables:
          USER: 'ec2-user'
          SERVER: '127.0.0.1'
          REMOTE_PATH: '/var/www/build/'
          LOCAL_PATH: 'build/'
          DEBUG: 'false'
          EXTRA_ARGS: ["-P", "8022"]
    ```

1.  `SSH_KEY`管道变量允许你定义一个替代的 SSH 密钥。它应该是一个 base64 编码的私钥，保存为安全的 Bitbucket 变量。以下代码片段展示了如何使用替代 SSH 密钥：

    ```
    script:
      - pipe: atlassian/scp-deploy:1.5.0
        variables:
          USER: 'ec2-user'
          SERVER: '127.0.0.1'
          REMOTE_PATH: '/var/www/build/'
          LOCAL_PATH: 'build'
          SSH_KEY: $MY_SSH_KEY
          DEBUG: 'true'
          EXTRA_ARGS: ['-o', 'ServerAliveInterval=10']
    ```

至此，我们已经通过 Bitbucket Pipelines 部署到了目标环境。目前，目标环境是物理服务器或虚拟机。接下来，我们将学习如何部署到公共云环境。

# 部署构建产物到 AWS S3 存储桶

在这个示例中，我们将看看如何部署到 AWS。根据构建产物的类型，Bitbucket Pipelines 有多种管道可以部署到特定的 AWS 组件服务。

让我们看看将构建产物部署到 AWS S3 存储桶中需要涉及的内容。这是使用 Bitbucket Pipelines 进行部署的典型用例。

## 准备工作

在设置用于部署到 AWS S3 存储桶的管道时，你需要做的唯一事情是设置以下变量：

+   `AWS_ACCESS_KEY_ID`：你的 AWS 访问密钥 ID。

+   `AWS_SECRET_ACCESS_KEY`：你的 AWS 密钥访问密钥。它应该作为一个安全变量保存。

+   `AWS_DEFAULT_REGION`：资源的默认 AWS 区域。

现在这些已经定义了，让我们来看一下如何设置管道。

## 如何操作…

此时，我们需要将管道配置添加到`bitbucket-pipelines.yml`文件的`script`部分。让我们更详细地看一下：

1.  将以下代码片段添加到`bitbucket-pipelines.yml`文件的`script`部分。这将包含你在*准备工作*部分设置的变量，并将其纳入`script`部分。这里，`S3_BUCKET`定义了目标存储桶，`LOCAL_PATH`定义了构建产物的位置：

    ```
    script:
      - pipe: atlassian/aws-s3-deploy:1.6.0
        variables:
          AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
          AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
          AWS_DEFAULT_REGION: 'us-east-1'
          S3_BUCKET: 'my-bucket-name'
          LOCAL_PATH: 'build'
    ```

1.  你还可以通过附加路径到在`S3_BUCKET`中定义的存储桶，来定义存储桶中的文件夹。以下代码片段演示了这一点：

    ```
    script:
      - pipe: atlassian/aws-s3-deploy:1.6.0
        variables:
          AWS_ACCESS_KEY_ID: $AWS_ACCESS_KEY_ID
          AWS_SECRET_ACCESS_KEY: $AWS_SECRET_ACCESS_KEY
          AWS_DEFAULT_REGION: 'us-east-1'
          S3_BUCKET: 'my-bucket-name/logs'
          LOCAL_PATH: '$(pwd)'
    ```

通过这些，你已经了解了如何轻松地将 Bitbucket Pipelines 连接到 AWS，从而部署构建产物。接下来，让我们看看如何在 Google Cloud 上做同样的事情。

# 部署构建产物到 Google Cloud

Bitbucket Pipelines 可以通过集成与这些服务的管道，向 Google 服务进行部署。以下是与 Google 服务连接的当前管道列表：

+   Firebase 部署

+   Google App Engine 部署

+   Google Cloud Storage 部署

+   **Google Artifactory 注册**（**GAR**）推送镜像

+   Google Kubernetes Engine kubectl run

让我们更详细地看一下如何使用 Google Cloud Storage 部署管道来部署构建产物。

## 如何操作…

我们可以通过执行以下步骤将构建产物部署到 Google Cloud Storage：

1.  在你打算部署到 Google Cloud Storage 的`step`部分的`script`区域中，复制并粘贴`pipe`定义。Google Cloud Storage 部署管道的定义如下所示：

    ```
    - pipe: atlassian/google-cloud-storage-deploy:2.0.0
      variables:
        KEY_FILE: '<string>'
        BUCKET: '<string>'
        SOURCE: '<string>'
        # GOOGLE_OIDC_CONFIG_FILE: "<string>" # Optional by default. Required for OpenID Connect (OIDC) authentication.
        # PROJECT: "<string>" # Optional by default. Required with GOOGLE_OIDC_CONFIG_FILE.
        # CACHE_CONTROL: '<string>' # Optional. options include no-cache,no-store,max-age=<seconds>, s-maxage=<seconds>, no-transform, public, private
        # CONTENT_DISPOSITION: '<string>' # Optional.
        # CONTENT_ENCODING: '<string>' # Optional.
        # CONTENT_LANGUAGE: '<string>' # Optional.
        # CONTENT_TYPE: '<string>' # Optional.
        # ACL: '<string>' # Optional.  Options include project-private, private, public-read, public-read-write, authenticated-read, bucket-owner-read, bucket-owner-full control
        # STORAGE_CLASS: '<string>' # Optional.  Options include multi-regional, regional, nearline, coldline
        # DEBUG: '<boolean>' # Optional.
    ```

1.  将管道添加到 `bitbucket-pipelines.yml` 文件的 `script` 部分。以下代码片段展示了只提供必需管道变量的示例：

    ```
    script:
      - pipe: atlassian/google-cloud-storage-deploy:2.0.0
        variables:
          KEY_FILE: $KEY_FILE
          BUCKET: 'my-bucket'
          SOURCE: 'myApp.jar'
    ```

1.  如果需要，请继续通过添加其他变量来定义部署操作。以下代码片段展示了一个填充了更多变量的管道：

    ```
    script:
      - pipe: atlassian/google-cloud-storage-deploy:2.0.0
        variables:
          KEY_FILE: $KEY_FILE
          BUCKET: 'my-bucket'
          SOURCE: 'myAppFile.jar'
          CACHE_CONTROL: 'max-age=60'
          CONTENT_DISPOSITION: 'attachment'
          ACL: 'public-read'
          STORAGE_CLASS: 'nearline'
    ```

我们刚刚看到了使用管道将部署集成到 Google Cloud 资源中的示例。现在，让我们看一下与 Microsoft Azure 集成的示例。

# 部署工件到 Microsoft Azure

Bitbucket Pipelines 可以通过与这些服务集成的管道执行对 Azure 服务的部署。以下是当前与 Microsoft Azure 连接的管道列表：

+   Azure CLI

+   Azure 容器应用程序部署

+   Azure ACR 推送镜像

+   Azure Functions 部署

+   Azure Kubernetes 服务部署

+   Azure Kubernetes 服务 Helm 部署

+   Azure 存储部署

+   Azure Web Apps 容器部署

+   Azure Web Apps 部署

让我们看看使用 Azure Functions 部署管道作为部署到 Microsoft Azure 的示例所需的内容。

## 准备中

在使用 Azure Functions 部署管道将部署到 Microsoft Azure 之前，我们需要建立 Azure 凭证。请按照以下步骤操作：

1.  在本地计算机上安装 Azure CLI 或使用 Azure Cloud Shell 后，通过输入以下命令来创建 Azure 安全主体：

    ```
    az ad sp create-for-rbac --name <name of your service principal>
    ```

1.  上述命令将返回以下 JSON 格式的输出：

    ```
    {
      "appId": "myAppId",
      "displayName": "myServicePrincipalName",
      "password": "myServicePrincipalPassword",
      "tenant": "myTentantId"
    }
    ```

1.  将输出保存为仓库变量。例如，您可以按如下方式将以下输出定义为变量：

    +   `appId`: `AZURE_ID`

    +   `password`: `AZURE_PASSWORD`（已加密）

    +   `tenant`: `AZURE_TENANT`

现在我们已经有了安全主体，可以使用 Bitbucket Pipelines 连接到 Azure。让我们来学习如何操作。

## 如何操作…

Azure Functions 部署将通过您最喜爱的 Microsoft 开发工具（如 Visual Studio）编写的无服务器逻辑实现，打包后在 Azure 中按需执行。开发过程的一部分可以包括通过 Bitbucket Pipelines 进行部署。让我们学习如何从 `bitbucket-pipelines.yml` 文件部署功能：

1.  将以下代码片段添加到 `bitbucket-pipelines.yml` 的 `script` 部分。所需的参数包括安全主体的属性、在 Azure 中找到的功能名称以及包含要在 Azure 部署的功能的 ZIP 文件名：

    ```
    script:
      - pipe: atlassian/azure-functions-deploy:2.0.0
        variables:
          AZURE_APP_ID: $AZURE_APP_ID
          AZURE_PASSWORD: $AZURE_PASSWORD
          AZURE_TENANT_ID: $AZURE_TENANT_ID
          FUNCTION_APP_NAME: '<string>'
          ZIP_FILE: '<string>'
          # DEBUG: '<boolean>' # Optional
    ```

1.  继续将必要的变量添加到管道定义中。完成的示例可以参考以下内容：

    ```
    script:
      - pipe: atlassian/azure-functions-deploy:2.0.0
        variables:
          AZURE_APP_ID: $AZURE_APP_ID
          AZURE_PASSWORD: $AZURE_PASSWORD
          AZURE_TENANT_ID: $AZURE_TENANT_ID
          FUNCTION_APP_NAME: 'my-function'
          ZIP_FILE: 'application.zip'
    ```

有了这些，我们已经将一个无服务器功能从本地系统部署到了 Azure。

部署的一个方面是 **基础设施即代码** (**IaC**)，即使用基于文本的配置动态创建所需资源。一种流行的工具是 Ansible。在下一个食谱中，我们将学习如何使用 Bitbucket Pipelines 执行 Ansible playbook 并部署资源。

# 在部署阶段使用 Ansible

Ansible 是执行基础设施即代码（IaC）的标准工具。通过 Ansible，你可以配置物理或虚拟服务器，并执行配置任务，如安装或升级软件、设置必要的参数、启动应用服务等。

Ansible 既可以通过原始的命令行接口（CLI）使用，也可以通过一个名为 **Ansible Tower** 的集成图形用户界面（GUI）为专用的 Ansible 应用服务器提供服务。我们将学习如何自动化部署到这两个接口。

## 准备工作

要执行 Ansible，必须在你计划用于执行 Bitbucket 流水线的运行器上安装以下程序：

+   `python`

+   `pip`

一旦它们安装完成，我们可以将 Ansible 命令添加到 Bitbucket 流水线中。

## 如何操作…

原始的 Ansible 应用程序基于 Python 构建，接受两个输入文件。这两个文件都是 YAML 格式的文本文件：

+   `playbook`：该文件包含 Ansible 执行的命令

+   `Inventory`：此文件详细列出了 Ansible 将 playbook 应用到的机器及其环境

让我们学习如何在 `bitbucket-pipelines.yml` 文件中整合这一操作：

1.  将以下行添加到 `bitbucket-pipelines.yml` 文件的 `script` 部分。这将安装 Ansible：

    ```
    - pip install ansible==2.17
    ```

1.  安装完成后，进入 Ansible playbook 所在的目录：

    ```
    -i flag to denote the inventory file:

    ```

    - ansible-playbook -i inventory playbook.yaml

    ```

    ```

1.  脚本应如下所示：

    ```
    script:
       - pip install ansible==2.17
       - cd deployment
       - ansible-playbook -i inventory playbook.yaml
    ```

在这个示例中，我们学习了如何使用社区版 Ansible 从 Bitbucket Pipelines 部署配置。来自 Red Hat 的完整版提供了一个专用的基础设施，用于运行由名为 Ansible Tower 的 GUI 控制的 Ansible 应用程序。接下来，我们将学习如何使用 Bitbucket Pipelines 部署到 Ansible Tower。

## 还有更多内容…

尽管控制 Ansible Tower 作业的主要方式是通过图形用户界面（GUI），但也有一个名为 `tower-cli` 的命令行工具，它允许你脚本化 Ansible Tower 作业。让我们来看一下 `tower-cli` 的实际操作：

1.  在你的构建环境中安装 `tower-cli`。你可能需要指定一个包含 Python 和 `pip` 的镜像。在这里，`pip` 允许你安装 `tower-cli`：

    ```
    image: python:2.7
      pipelines:
       default:
        - step:
            script: # Modify the commands below to build your repository.
                - pip install ansible-tower-cli
    ```

1.  设置所需的环境变量。对于 `tower-cli`，这些变量如下：

    +   `host`：Tower 主机

    +   `username`：Tower 用户名

    +   `password`：Tower 用户密码（保存为安全的 Bitbucket 变量）

    +   `ID`：要启动的 Tower 作业模板的 ID

1.  添加以下几行来配置环境变量并运行 Ansible 作业：

    ```
     - hostval=$(tower-cli config host $host)
     - userval=$(tower-cli config username $username)
     - passwordval=$(tower-cli config password $password)
     - tower-cli config verify_ssl false
     - tower-cli job launch --job-template $ID --monitor
    ```

有了这些，我们就触发了位于 Ansible Tower 中的 Ansible 作业，从 Bitbucket Pipelines 启动。

## 另见

以下是 Ansible 文档，提供了创建正确 Ansible 作业的指导：

+   [`access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.4`](https://access.redhat.com/documentation/en-us/red_hat_ansible_automation_platform/2.4)

+   [`docs.ansible.com/ansible/latest/index.html`](https://docs.ansible.com/ansible/latest/index.html)

# 在部署阶段使用 Terraform

Terraform 是另一种提供 IaC 功能的工具。它的流行之处在于它能够灵活地概述云资源的实例创建，然后通过指定实施细节的提供程序将这些资源实施到特定的云平台。

让我们学习如何从 Bitbucket Pipelines 部署到 Terraform。

## 准备工作

Terraform 在三个文件中描述了它将执行的配置，这些文件都需要在您的 Bitbucket 仓库中：

+   `main.tf`

+   `variables.tf`

+   `provider.tf`

此外，Terraform 配置后端平台所需的任何凭据，例如 AWS 或 Google Cloud，应该存储为 Bitbucket 变量，并在必要时将其设置为安全。

在这些步骤完成后，让我们设置我们的部署。

## 如何操作…

我们必须采取几个步骤才能使用 Terraform 部署，封装在三个命令中。让我们看一下我们需要使用的 Terraform 命令：

1.  对于定义部署的步骤，使用 Terraform Docker 镜像。我们将在*第九章*中更详细地介绍如何在 Bitbucket Pipeline 步骤中使用 Docker 镜像：

    ```
     - step:
        image: hashicorp/terraform:full
    ```

1.  在 `bitbucket-pipelines.yml` 文件的 `script` 部分中，输入以下命令来初始化 Terraform：

    ```
     - terraform init
    ```

1.  添加以下命令以执行验证。这可能不是必需的：

    ```
    -out flag:

    ```

    - terraform plan -out=plantf

    ```

    ```

1.  添加以下命令以应用计划并运行 Terraform：

    ```
    step should now look as follows:

    ```

    - step:

    image: hashicorp/terraform:full

    script:

    - terraform init

    - terraform validate

    - terraform plan -out=plantf

    - terraform apply plantf

    ```

    ```

通过这些步骤，我们在 Bitbucket Pipelines 中执行了部署，并调用 Terraform 创建了我们的实例。

## 另见

以下资源有助于理解 Terraform – [`developer.hashicorp.com/terraform/docs`](https://developer.hashicorp.com/terraform/docs)。
