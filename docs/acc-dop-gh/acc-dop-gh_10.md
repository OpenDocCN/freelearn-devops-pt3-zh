# 第八章：使用 GitHub Packages 管理依赖关系

使用包注册表来管理依赖关系应该是显而易见的选择。如果你编写 .NET 代码，你会使用 NuGet；如果你编写 JavaScript 代码，可能会使用 npm；如果你使用 Java，通常是 Maven 或 Gradle。然而，我遇到过许多团队仍然使用文件系统或 Git 子模块在多个代码库中重用代码文件，或者构建程序集并将它们存储在源代码管理中。转向使用带有**语义版本控制**的包是简单且低成本的，而且能够提升共享代码的质量和可发现性。

在本章中，我将向你展示如何像管理软件供应链一样使用 GitHub Packages 管理内部依赖关系。主要内容如下：

+   GitHub Packages

+   使用 npm 包与 Actions

+   使用 Docker 和 Packages

+   Apache Maven、Gradle、NuGet 和 RubyGems 包

    语义版本控制

    `1.0.0` 或 `1.5.99-beta`。格式如下：

    `<major>.<minor>.<patch>-<pre>`

    `[0-9A-Za-z-]`）。文本越长，预发布版本越小（意味着 `-alpha` < `-beta` < `-rc`）。预发布版本始终比正常版本小（`1.0.0-alpha` < `1.0.0`）。

    请参阅 [`semver.org/`](https://semver.org/) 获取完整规范。

使用包并不意味着你自动使用了松耦合架构。在大多数情况下，包仍然是强依赖关系。是否能真正解耦发布节奏取决于你如何使用这些包。

# GitHub Packages

**GitHub Packages** 是一个托管和管理你的包、容器及其他依赖项的平台。

你可以将 GitHub Packages 与 GitHub Actions、GitHub APIs 和 Webhooks 集成。这使你能够创建一个端到端的工作流来发布和使用你的代码。

GitHub Packages 目前支持以下注册表：

+   **容器** 注册表支持 **Docker** 和 **OCI** 镜像

+   `package.json`)

+   `nupkg`)

+   `pom.xml`)

+   `build.gradle`)

+   `Gemfile`)

## 定价

公共包是免费的。对于私有包，每个 GitHub 版本包括一定量的存储和数据传输。超过该限额的任何使用将单独收费，并且可以通过支出限制进行控制。

月度计费的客户默认有 $0 美元的支出限额，这会阻止额外使用存储或数据传输。开具发票的客户则有无限的默认支出限额。

每个产品的包含存储和传输数据量列在 *表 8.1* 中：

![表 8.1 – GitHub 产品中包的包含存储和数据传输](img/Table_8.1.jpg)

](img/Table_8.1.jpg)

表 8.1 – GitHub 产品中包的包含存储和数据传输

所有由 GitHub Actions 触发的外部数据传输都是免费的。任何来源的内部数据传输也都是免费的。

当达到包含的限制时，将收取以下费用：

+   **存储**：每 GB $0.25 美元

+   **数据传输**：每 GB $0.50 美元

欲了解更多关于定价的信息，请参阅 [`docs.github.com/en/billing/managing-billing-for-github-packages/about-billing-for-github-packages`](https://docs.github.com/en/billing/managing-billing-for-github-packages/about-billing-for-github-packages)。

## 权限和可见性

发布到仓库的包继承该仓库的权限和可见性。当前，只有容器包提供详细的权限和访问控制（见*图 8.1*）。

![图 8.1 – 管理对容器包的访问](img/B17827_08_01.jpg)

图 8.1 – 管理对容器包的访问

所有其他包类型遵循仓库作用域包的仓库访问权限。在组织级别，包是私有的，所有者具有写权限，成员具有读权限。

如果你对容器镜像具有管理员权限，你可以将容器镜像的访问权限设置为`private`或`public`。公开镜像允许匿名访问，无需认证。你还可以为容器镜像设置与组织和仓库级别的权限分开的访问权限。

在组织级别，你可以设置成员可以发布的容器包类型。你还可以查看和恢复已删除的包（见*图 8.2*）。

![图 8.2 – 组织级别的包权限](img/B17827_08_02.jpg)

图 8.2 – 组织级别的包权限

对于由用户帐户拥有的容器镜像，你可以为任何人分配访问角色。对于由组织发布并拥有的容器镜像，你只能为组织中的人员或团队分配访问角色。

欲了解更多关于权限和可见性的信息，请参阅 [`docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility`](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility)。

# 使用 npm 包与 Actions

使用 GitHub Actions 设置包的发布工作流非常简单。你可以使用`GITHUB_TOKEN`进行认证，并使用包管理器的原生客户端。要尝试使用 npm，你可以按照这里的逐步说明进行操作：[`github.com/wulfland/package-demo`](https://github.com/wulfland/package-demo)。

如果你的机器上已经安装了 npm，你可以使用`npm init`来创建包。否则，只需从上述仓库复制`package.json`和`package-lock.json`的内容。

发布包的工作流很简单。每次创建新版本时，它都会被触发：

```
on:
```

```
  release:
```

```
    types: [created]
```

工作流由两个作业组成。第一个作业仅使用 npm 构建和测试包：

```
  build:
```

```
    runs-on: ubuntu-latest
```

```
    steps:
```

```
      - uses: actions/checkout@v2
```

```
      - uses: actions/setup-node@v2
```

```
        with:
```

```
          node-version: 12
```

```
      - run: npm ci
```

```
      - run: npm test
```

第二个操作将图像发布到注册表。这个操作需要写入包和读取内容的权限。它使用`${{ secrets.GITHUB_TOKEN }}`来进行注册表认证：

```
  publish-gpr:
```

```
    needs: build
```

```
    runs-on: ubuntu-latest
```

```
    permissions:
```

```
      packages: write
```

```
      contents: read
```

```
    steps:
```

```
      - uses: actions/checkout@v2
```

```
      - uses: actions/setup-node@v2
```

```
        with:
```

```
          node-version: 12
```

```
          registry-url: https://npm.pkg.github.com/
```

```
      - run: npm ci
```

```
      - run: npm publish
```

```
        env:
```

```
          NODE_AUTH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

这个工作流很简单，每次在 GitHub 中创建新发布时，它都会将新包发布到你的 npm 注册表中。你可以在**Code** | **Packages**下找到包的详细信息和设置（参见*图 8.3*）。

![图 8.3 – 包的详细信息和设置](img/B17827_08_03.jpg)

图 8.3 – 包的详细信息和设置

然后，你可以在其他项目中使用`npm install @<owner-name>/<package-name>`来使用该包。

注意

请注意，包的版本不是标签或发布版本，而是`package.json`文件中的版本。如果在创建第二个发布版本之前没有更新版本，工作流将会失败。

如果你想自动化此过程，有一些操作可以帮助你。你可以使用`github.event.release.name`或标签（`github.event.release.tag_name`），并将包版本设置为此：

```
- name: 'Change NPM version'
```

```
  uses: reedyuk/npm-version@1.1.1
```

```
  with:
```

```
    version: ${{github.event.release.tag_name}}
```

如果你想要一种更灵活的方法，根据标签和分支计算语义版本号，可以使用**GitVersion**（请参见[`gitversion.net/`](https://gitversion.net/)）。**GitVersion**是**GitTools**操作的一部分（请参见[`github.com/marketplace/actions/gittools`](https://github.com/marketplace/actions/gittools)）。

对于 checkout 操作的`fetch-depth`参数并将其设置为`0`：

```
    steps:
```

```
      - uses: actions/checkout@v2
```

```
        with: 
```

```
          fetch-depth: 0
```

接下来，安装`execute`操作。如果你想获取语义版本的详细信息，请设置`id`：

```
      - name: Install GitVersion
```

```
        uses: gittools/actions/gitversion/setup@v0.9.7
```

```
        with:
```

```
          versionSpec: '5.x'
```

```
      - name: Determine Version
```

```
        id:   gitversion
```

```
        uses: gittools/actions/gitversion/execute@v0.9.7
```

计算出的最终语义版本号将作为环境变量`$GITVERSION_SEMVER`存储。你可以将其用作例如**npm-version**的输入。

注意

请注意，**GitVersion**支持配置文件，以了解它应该如何计算版本！有关更多信息，请参见[`gitversion.net/`](https://gitversion.net/)。

如果你需要访问`gitversion`任务中的详细信息：

```
  - name: Display GitVersion outputs
```

```
    run: |
```

```
      echo "Major: ${{ steps.gitversion.outputs.major }}"
```

使用**GitVersion**，你可以扩展工作流，从分支或标签创建包——不仅限于发布版本：

```
on:
```

```
  push: 
```

```
    tags:
```

```
      - 'v*'
```

```
    branches:
```

```
      - 'release/*'
```

构建一个带有自动语义版本控制的发布工作流是复杂的，并且在很大程度上取决于你使用的工作流和包管理器。本章应能帮助你入门。此技巧也可以应用于**NuGet**、**Maven**或任何其他包管理器。

# 使用 Docker 与包

GitHub 的容器注册表是`ghcr.io`。容器镜像可以由组织或个人帐户拥有，但你可以自定义每个镜像的访问权限。默认情况下，镜像继承了运行工作流的仓库的可见性和权限模型。

如果你想亲自试一下，可以在这里找到逐步指南：[`github.com/wulfland/container-demo`](https://github.com/wulfland/container-demo)。按照这些步骤了解构建过程的工作原理：

1.  创建一个名为`container-demo`的新仓库，并添加一个非常简单的`Dockerfile`（没有扩展名）：

    ```
    FROM alpine
    CMD ["echo", "Hello World!"]
    ```

该 Docker 镜像继承自 alpine 发行版，并将`Hello World!`输出到您的控制台。如果您是 Docker 新手并想尝试，可以克隆仓库并将目录切换到本地仓库的根目录。然后为容器构建镜像：

```
$ docker build -t container-demo 
```

然后运行容器：

```
$ docker run --rm container-demo
```

`--rm`参数在完成时自动移除容器。这应该会将`Hello World!`输出到您的控制台。

1.  现在在`.github/workflows/`目录下创建一个名为`release-container.yml`的工作流文件。每当创建新的发布时，工作流将被触发：

    ```
    name: Publish Docker image
    on:
      release:
        types: [published]
    ```

注册表和镜像名称被设置为环境变量。我使用仓库名称作为镜像名称。您也可以在这里设置固定名称：

```
env:
  REGISTRY: ghcr.io
  IMAGE_NAME: ${{ github.repository }}
```

该工作需要对`packages`具有写权限，并且需要克隆仓库：

```
jobs:
  build-and-push-image:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    steps:
      - name: Checkout repository
        uses: actions/checkout@v2
```

`docker/login-action`使用`GITHUB_TOKEN`对工作流进行身份验证。这是推荐的做法：

```
- name: Log in to the Container registry
  uses: docker/login-action@v1.10.0
  with:
    registry: ${{ env.REGISTRY }}
    username: ${{ github.actor }}
    password: ${{ secrets.GITHUB_TOKEN }}
```

`metadata-action`从 Git 上下文中提取元数据，并将标签应用于 Docker 镜像。当我们创建发布时，我们推送一个标签（`refs/tags/<tag-name>`）。该操作将创建一个与 Git 标签同名的 Docker 标签，并为镜像创建最新标签。请注意，元数据作为输出变量传递给下一个步骤！这就是为什么我为此步骤设置了一个`id`：

```
- name: Extract metadata (tags, labels)
  id: meta
  uses: docker/metadata-action@v3.5.0
  with:
    images: ${{ env.REGISTRY }}/${{ env.IMAGE_NAME }}
```

`build-push-action`构建镜像并将其推送到容器注册表。标签和标签从`meta`步骤的输出中提取：

```
- name: Build and push Docker image
  uses: docker/build-push-action@v2.7.0
  with:
    context: .
    push: true
    tags: ${{ steps.meta.outputs.tags }}
    labels: ${{ steps.meta.outputs.labels }}
```

1.  创建一个新的发布和标签以触发工作流。工作流完成后，您可以在**Code** | **Packages**下找到包的详细信息和设置（参见*图 8.4*）。

![图 8.4 – 容器包的详细信息和设置](img/B17827_08_04.jpg)

图 8.4 – 容器包的详细信息和设置

如果您创建了新的发布，GitHub 现在将创建一个新的 Docker 镜像并将其添加到注册表中。

1.  您可以从注册表将容器拉取到本地并运行：

    ```
    $ docker pull ghcr.io/<user>/container-demo:latest
    $ docker run --rm ghcr.io/<user>/container-demo:latest
    > Hello World!
    ```

请注意，如果您的包不是公开的，在拉取镜像之前，您必须使用`docker login ghcr.io`进行身份验证。

容器注册表是发布软件的好方法。从命令行工具到完整的微服务，您可以将软件及其所有依赖项一起发布，供其他人使用。

# Apache Maven、Gradle、NuGet 和 RubyGems 包

其他包类型与 npm 和 Docker 基本相同：如果您了解本地包管理器，它们非常容易使用。我将简要介绍每种类型。

## 使用 Apache Maven 的 Java

对于`pom.xml`文件：

```
<distributionManagement>
```

```
  <repository>
```

```
    <id>github</id>
```

```
    <name>GitHub Packages</name>
```

```
    <url>https://maven.pkg.github.com/user/repo</url>
```

```
  </repository>
```

```
</distributionManagement>
```

然后，您可以使用`GITHUB_TOKEN`在工作流中发布您的包：

```
- name: Publish package
```

```
  run: mvn --batch-mode deploy
```

```
  env:
```

```
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

要从您的开发机器获取包，您必须使用`read:packages`作用域进行身份验证。您可以在 GitHub 中的`~/.m2/settings.xml`文件下生成新的令牌。

有关更多信息，请参阅 [`docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry`](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-apache-maven-registry)。

## Gradle

在`build.gradle`文件中，你可以从环境变量中读取用户名和访问令牌：

```
repositories {
```

```
  maven {
```

```
    name = "GitHubPackages"
```

```
    url = "https://maven.pkg.github.com/user/repo"
```

```
    credentials {
```

```
      username = System.getenv("GITHUB_ACTOR")
```

```
      password = System.getenv("GITHUB_TOKEN")
```

```
    }
```

```
  }
```

```
}
```

在工作流中，你可以使用`gradle publish`进行发布：

```
- name: Publish package
```

```
  run: gradle publish
```

```
  env:
```

```
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

有关更多详细信息，请参阅 [`docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry`](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-gradle-registry)。

## RubyGems

如果你想构建并发布你仓库中的所有`.gemspec`文件，可以使用市场中的一个操作：

```
- name: Build and publish gems got .gemspec files
```

```
  uses: jstastny/publish-gem-to-github@master
```

```
  with:
```

```
    token: ${{ secrets.GITHUB_TOKEN }}
```

```
    owner: OWNER
```

要使用包，你需要至少 RubyGems 2.4.1 和 bundler 1.6.4。修改`~/.gemrc`文件并通过提供你的用户名和个人访问令牌，将注册表作为源来安装包：

```
---
```

```
:backtrace: false
```

```
:bulk_threshold: 1000
```

```
:sources:
```

```
- https://rubygems.org/
```

```
- https://USERNAME:TOKEN@rubygems.pkg.github.com/OWNER/
```

```
:update_sources: true
```

```
:verbose: true  
```

要使用**bundler**安装包，你还需要配置它与你的用户和令牌：

```
$ bundle config \
```

```
https://rubygems.pkg.github.com/OWNER \
```

```
USERNAME:TOKEN
```

有关更多详细信息，请参阅 [`docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-rubygems-registry`](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-rubygems-registry)。

## NuGet

要发布`setup-dotnet`操作。它有额外的参数`source-url`。令牌通过环境变量设置：

```
- uses: actions/setup-dotnet@v1
```

```
  with:
```

```
    dotnet-version: '5.0.x'
```

```
    source-url: https://nuget.pkg.github.com/OWNER/index.json
```

```
  env:
```

```
    NUGET_AUTH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

然后，你可以构建并测试你的项目。之后，只需打包并将包推送到注册表：

```
- run: |
```

```
  dotnet pack --configuration Release
```

```
  dotnet nuget push "bin/Release/*.nupkg"
```

要安装包，你必须将注册表作为源添加到`nuget.config`文件中，包括你的用户和令牌：

```
<?xml version="1.0" encoding="utf-8"?>
```

```
<configuration>
```

```
    <packageSources>
```

```
        <add key="github" value="https://nuget.pkg.github.com/OWNER/index.json" />
```

```
    </packageSources>
```

```
    <packageSourceCredentials>
```

```
        <github>
```

```
            <add key="Username" value="USERNAME" />
```

```
            <add key="ClearTextPassword" value="TOKEN" />
```

```
        </github>
```

```
    </packageSourceCredentials>
```

```
</configuration>
```

有关更多信息，请参阅 [`docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry`](https://docs.github.com/en/packages/working-with-a-github-packages-registry/working-with-the-nuget-registry)。

# 摘要

使用包是直接的，最大的挑战是身份验证。但通过在 GitHub Actions 中使用`GITHUB_TOKEN`，你可以轻松设置一个完全自动化的发布工作流。这就是为什么它对你的团队来说是工具箱中不可或缺的一部分。如果你将代码作为容器或包共享，并使用语义版本控制和单独的发布流程，许多发布代码的问题可以减少。

本章中，你已了解如何使用语义版本控制和包来更好地管理你的内部依赖项并共享代码。你已了解包是什么，以及如何为每种包类型设置发布工作流。

在下一章，我们将更详细地探讨环境以及如何使用 GitHub Actions 将应用部署到任何平台。

# 进一步阅读

有关本章中主题的更多信息，请参考以下内容：

+   *语义版本控制*: [`semver.org/`](https://semver.org/)

+   *计费与定价*: [`docs.github.com/en/billing/managing-billing-for-github-packages/about-billing-for-github-packages`](https://docs.github.com/en/billing/managing-billing-for-github-packages/about-billing-for-github-packages)

+   *访问控制与可见性*: [`docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility`](https://docs.github.com/en/packages/learn-github-packages/configuring-a-packages-access-control-and-visibility)

+   *与注册表合作*（容器，Apache Maven，Gradle，NuGet，npm，RubyGems）：[`docs.github.com/en/packages/working-with-a-github-packages-registry`](https://docs.github.com/en/packages/working-with-a-github-packages-registry)
