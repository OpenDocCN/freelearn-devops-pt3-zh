

# 提升 CI/CD 管道的速度和可维护性

本章将介绍你可以利用的不同工具和方法，来提升 CI/CD 管道的速度和可维护性。本章的目标是介绍三种主要的方法来加速你的管道。我们不会涵盖 GitLab 中加速 CI 管道的所有方法，而是专注于对管道速度影响最大的几种方法。

本章将讨论以下内容：

+   使用有向无环图（DAG）和父子架构加速管道

+   为多种架构构建代码

+   何时以及如何利用缓存或工件

+   使用锚点和扩展减少重复的配置代码

+   通过结合多个管道并利用父子管道提升可维护性

+   使用专用容器保障和加速任务

# 使用有向无环图（DAG）和父子架构加速管道

GitLab 支持使用**有向无环图**（**DAG**）模式构建 CI 管道。在 GitLab 的正常使用中，每个阶段代表一系列需要完成的任务。每个阶段由多个并行执行的任务组成。当一个阶段完成后，下一个阶段开始，直到整个管道完成。这是 GitLab 用于完成 CI 管道的典型处理循环。

然而，完全可以创建一个内部循环的 CI 任务，这些任务按特定顺序执行，但不会形成循环。这个模式叫做**DAG**，即有向无环图。*有向*指的是操作的顺序，*无环*指的是这个过程只会发生一次，*图*则表示步骤的顺序。

如果正确使用，DAG 模式可以显著减少管道完成所需的时间。这是因为你在 CI 管道中创建了处理循环，并且明确地定义了 CI 任务的执行顺序，而这些任务位于阶段之外。一个好的例子是，如果一个任务的执行依赖于另一个任务的执行，你可以使用`needs:`关键字来确保它们一个接一个地执行，而不受阶段顺序的影响。这样就创建了一个简单的 DAG 过程。

## 如何在 GitLab CI 中创建 DAG

以下是一个正常 GitLab CI 管道的示例。以下是我们在介绍中描述的示例的 YAML 实现：

```
Job1:
  stage: A
Job2:
  stage: A
Job3:
  stage: B
Job4:
  stage: B
```

在传统管道中，如下所示，`stage: B`中定义的任务仅在所有`stage: A`中的任务完成后才会开始执行。以下是设置为使用 DAG 的相同管道示例：

```
Job1:
  stage: A
Job2:
  stage: A
Job3:
  stage: B
  needs: ["Job1"]
Job4:
  stage: B
```

通过将 `needs` 关键字添加到 `Job3`，我们在 `Job1` 和 `Job3` 之间建立了依赖关系。所以，一旦 `Job1` 完成，`Job3` 将开始执行。这种依赖关系超出了常规的阶段顺序。如果未定义 `needs`，则会遵循阶段顺序。如我们在 `Job4` 中所见，要让 `Job4` 开始执行，`stage: A` 必须先完成。然而，如果定义了 `needs: []` 且为空，则 GitLab 会在管道开始时立即执行该作业，并且不会分配给任何阶段执行。

# 为多个架构构建代码

GitLab CI 使你能够一次为多个架构构建工件。这可以轻松地加速软件构建，速度提升可达两到三倍，因为在多个架构下，通常需要为每个架构多次构建软件。在这里，我们将使用 CI 作业和管道同时执行多个架构的软件构建。

在为多个架构构建软件时，有一些特殊要求。第一个要求是，必须在每个架构运行的机器上安装并配置 GitLab Runners。对于这一部分，我们将以三个平台作为示例：`x86_64`、`arch64` 和 `powerpc`。在这种情况下，预期是你需要为这三种架构中的每一种准备一台机器，并在其上安装 GitLab Runner。该 GitLab Runner 还需要为它所运行的架构分配一个标签。

第二个要求是，你用来构建软件的工具链必须能够支持多架构构建。为了演示，我们将使用 GCC 和多架构的 Docker 镜像。我们还将使用 GitLab CI 的 `parallel:` 和 `matrix:` 关键字。

`parallel:` 关键字旨在允许 CI 作业并行运行多次。例如，如果你在 CI 作业中设置 `parallel: 5`，该 CI 作业将在管道中并行执行 5 次。

`matrix:` 关键字旨在与 `parallel:` 关键字一起使用，在同一时间启动多个 CI 作业，并为每个作业分配不同的变量。以下是当在多架构构建的管道中使用时的示例：

```
my-multiarch-ci-job:
  stage: build
  image: multiarch/crossbuild
  script:
    - make helloworld
  parallel:
    matrix:
      - ARCH: x86
        CROSS_TRIPLE: " "
      - ARCH: arch64
        CROSS_TRIPLE: "arch64-linux-gnu"
      - ARCH: powerpc
        CROSS_TRIPLE: "powerpcle-linux-gnu"
  tags:
    - $ARCH
```

上述示例将执行三个关键操作：

+   首先，通过使用 `parallel:` 和 `matrix:`，GitLab 将启动三个独立的作业 —— 每个作业对应我定义的 `ARCH` 和 `CROSS_TRIPLE` 的变量对。

+   其次，每个作业将会分配一个标签，反映 `ARCH:` 中定义的值。这将导致该作业分配到与标签对应的适当 runner。

+   第三，GitLab 将把 `CROSS_TRIPLE:` 环境变量暴露给 `multiarch/crossbuild` 容器。这个环境变量用于容器正确配置 GCC 工具链，使其为目标架构做好构建准备。

当容器启动并执行 CI 作业时，`make helloworld` 命令将被执行。该命令将调用预配置的 GCC 工具链，并使其开始构建我们的应用程序二进制文件。生成的二进制文件将被构建为支持指定架构的版本。

通过这种方式构建多架构二进制文件，我们简化了多架构构建的复杂性。我们使得这些多架构构建变得可重复且易于理解。我们还将这些构建并行运行，这样它们就不会相互等待，我们可以快速看到每个架构构建的结果。

重要提示

`parallel:` 和 `matrix:` 关键字可以在任何需要多个具有相同配置但变量不同的 CI 作业的情况下使用。`matrix:` 关键字还可以接受数组作为环境变量的值。然而，环境变量的键*不能*是数组。

# 何时以及如何利用缓存或工件

在使用 GitLab 时，关于缓存和工件的使用经常会引起困惑。许多用户都很想知道在什么时候使用哪种功能。本章的目标是为您澄清这两者的区别，并提供实现这两种方法的工具，同时解释每种模式的优缺点。

缓存应该被视为一种保存 CI 作业或阶段中常用项目的方法。它不应被视为在阶段或作业之间传递项目的手段——这正是工件的作用。这个区别很重要，因为每种方式的实现和配置不同。缓存并不是为了作为一种在 CI 作业之间移动项目的方法而构建或设计的。因此，未来对缓存的任何功能或更改都将围绕该功能进行调整。

工件是支持将 CI 作业创建的项目永久存储，并将其作为依赖项传递给其他 CI 作业的主要方式。使用工件时，您可以了解存储了哪些内容，并且可以将具有硬依赖关系的 CI 作业关联起来，以确保工件得以共享。与缓存不同，缓存只有软链接。使用缓存的作业即使缓存不存在也能继续工作；然而，使用工件的作业则无法继续执行。

缓存和工件的下一个考虑因素是它们的存储方式以及涉及的网络调用。缓存包由 GitLab Runner 处理，GitLab Runner 的配置决定了缓存的存储位置。在默认配置参数下，缓存会存储在运行器所在的机器上。对于容器而言，缓存会被销毁。通过为 GitLab Runner 添加 S3 配置，Runner 会将所有缓存包推送到 S3 存储。在 CI 作业开始之前，任何使用该缓存的 CI 作业都会从 S3 存储中拉取缓存。

对于工件，Runner 在其处理过程中不发挥作用。每个工件包都会直接上传到 GitLab 实例。然后，GitLab 的配置决定该工件何时以及存储在何处。最常见的配置也是 S3 存储提供商。任何需要此工件的未来 CI 作业都会在 CI 作业开始之前下载它。像缓存包一样，工件包会被解压到工作目录中以供使用。

重要提示

在本章中，我们将讨论两种类型的依赖关系。第一种是 CI 作业依赖关系，应通过工件进行处理。CI 作业依赖关系本质上将一个 CI 作业的结果链接到另一个作业。第二种依赖关系类型是工具链依赖关系，即应用程序的依赖项——通常是第三方库。这些依赖项由多个 CI 作业或多个 CI 管道使用，因此应该利用缓存。

## 缓存特性

这些是区分缓存与工件的特点：

+   你可以通过使用`cache`关键字为每个作业定义缓存。否则，缓存将被禁用。

+   后续管道可以使用缓存。

+   如果依赖项相同，同一管道中的后续作业可以使用缓存。

+   不同的项目不能共享缓存。

+   默认情况下，受保护的分支和非受保护的分支不会共享缓存。但是，你可以更改此行为。

## 工件特性

这些是区分工件与缓存的特点：

+   你可以为每个作业定义工件。

+   同一管道后续阶段的作业可以使用工件。

+   不同的项目不能共享工件。

+   工件默认在 30 天后过期。你可以定义自定义的过期时间。

+   如果启用了**保持最新工件**，则最新工件不会过期。

+   你可以使用依赖关系来控制哪些作业获取工件。

## 使用缓存

利用缓存的第一步是将配置添加到你的 GitLab CI 文件中。以下是如何执行此操作的代码示例。如果你查看此示例，你会看到我们定义了一个名为`MyCIJob`的 CI 作业。在此基础上，我们定义了一个`cache`块，并添加了一个`path`参数，列出了我们希望缓存的目录和文件。这是将项目缓存作为 CI 作业的一部分所需的最小有效代码：

```
MyCIJob:
  cache:
    paths:
      - theDirectoryToSave/*
      - myFileToSave.js
```

现在，每次此 CI 作业启动时，它都会使用相同的缓存包。这在你有每次 CI 作业运行时都会下载的依赖项时非常有用。通过将这些依赖项放入缓存中，它们就不需要每次都下载。相反，GitLab 会在作业开始时插入这些依赖项。这大大缩短了 CI 作业的时间，最终减少了 CI 管道完成的时间。

现在，我们将进一步深入。假设你有四个独立的 CI 作业，其中每两个 CI 作业需要共享一个缓存。在前面的示例中，我们创建了一个通用的缓存，可以用于所有作业。但是在这个示例中，我们希望在两个 CI 作业之间链接一个缓存。这可以通过添加 `key` 值来完成：

```
MyCIJob:
  cache:
    key: cache1
    paths:
      - theDirectoryToSave/*
      - myFileToSave.js
```

通过添加 `key` 值，GitLab 将拥有一个标识符，用来确定何时以及下载哪个缓存包。这个 `key` 值可以是一个字符串、一个变量或两者的任意组合。许多用户最终会使用 GitLab 预定义的变量作为 key，这样 GitLab 就可以更有效地管理缓存。例如，如果你使用 `$CI_PIPELINE_ID` 变量，那么这个缓存包只会在 ID 与 `$CI_PIPELINE_ID` 匹配的流水线中被使用。这意味着每个流水线都会有自己全新的缓存包。

你可以利用缓存的最终配置是改变缓存包的上传和下载的时机和方式：

```
MyCIJob:
  cache:
    key: cache1
    policy: pull
    paths:
      - theDirectoryToSave/*
      - myFileToSave.js
```

如你所见，缓存的 GitLab CI 对象有一个名为 `policy` 的值。默认情况下，这个值设置为 `push-pull`，这意味着缓存的包会在作业开始时被拉取，在作业结束时被推送。然而，你也可以将其配置为仅推送或仅拉取。设置为推送时，它会推送缓存对象，但从不拉取；而设置为拉取时，它会拉取缓存，但不会推送更新到包中。

## 使用构件

在前一部分中，我们讨论了缓存。缓存的许多相同概念将应用于构件，但构件具有更丰富的功能集。由于构件用于报告、作业之间传递构件和作业依赖管理等，因此它们需要更深层次的功能集。让我们从一个基本的构件对象开始，并从中构建一个更复杂的构件：

```
MyArtifactJob:
  artifacts:
    paths:
      - theDirectoryToSave/*
      - myFileToSave.js
```

从前面的示例中，你可以看到它的核心与缓存非常相似。我们添加了一个构件作业，分配了一个 `artifacts:` 对象给它，向其中添加了一个 `paths:` 对象，并列出了我们希望进行构件化的文件。从这里开始，当 `MyArtifactJob` 执行时，流水线运行结束时，它将把 `paths:` 中列出的所有内容发送到 GitLab 并进行存储。

类似于缓存的 `policy` 选项，它定义了如何处理缓存，构件有一个 `when` 对象，允许你指定当构件创建时会发生什么。其选项为 `on_success`、`on_failure` 和 `always`。第一个选项 `on_success` 只有在作业成功执行时才会上传构件。第二个选项 `on_failure` 只有在作业失败时才会上传构件。最后，`always` 会在任何结果下始终上传构件。应用到我们的作业时，结果作业将如下所示：

```
MyArtifactJob:
  artifacts:
    when: 'on_success'
    paths:
      - theDirectoryToSave/*
      - myFileToSave.js
```

总结一下，前面的任务只有在其他 CI 任务成功时才会执行。然后，它将把对象存储在`theDirectoryToSave/`文件夹中的`myFileToSave.js`文件里。

## 利用产物作为任务依赖

现在我们已经介绍了产物的基础知识，让我们通过在实际场景中将任务与产物链接在一起，来进一步应用它们。通过在 CI 任务块中使用`dependencies`关键字，我们可以从另一个任务中提取产物，并在两个独立的 CI 任务之间建立内在的联系。

我们的应用程序将由`MyBuildJob`任务构建，该任务将在稍后定义。你可能会从之前的章节中认出很多语法。这个第一个任务是一个构建基于 Node.js 的应用程序的示例。首先，我们使用`npm install`命令来拉取构建此应用所需的任何依赖项。为了保持这个示例的简洁，我们没有使用依赖项缓存，但在生产环境中应该使用缓存。

其次，我们运行`npm build`命令来构建 Node.js 应用程序。所有 Node.js 项目的通用标准是，它们的构建文件都会生成在一个名为`dist`的文件夹中。Node.js 的依赖项通常存储在`node_modules`文件夹中。

最后，我们有一个产物定义，列出了`node_modules`和`dist`文件夹作为需要归档的内容。现在，当这个任务执行并完成时，GitLab 会存储这两个文件夹中所有的项目：

```
MyBuildJob:
  image: nodejs:latest
  script:
    - npm install
    - npm build
  artifacts:
    when: 'on_success'
    paths:
      - node_modules/*
      - dist/*
```

以下 CI 任务，定义为`MyDependentJob`，被构建用来从我们之前的任务中提取构建产物。然后，它将这些产物用于 Dockerfile 的构建。`dependencies`关键字是连接这两个任务的桥梁：

```
MyDependentJob:
  image: docker:latest
  dependencies:
    - MyBuildJob
  script:
    - docker build
```

正如我们所看到的，通过使用`dependencies`关键字，我们指示 GitLab 确保`MyDependentJob`拥有执行所需的一切。如果没有这个功能，`MyDependentJob`将需要自己拉取这些产物，这将需要额外的时间和配置。

# 使用锚点和扩展来减少重复的配置代码

所有 GitLab CI 管道文件必须是有效的 YAML。这也意味着它们支持 YAML 语言支持的各种模板和可重复的代码模式。为每个 CI 任务编写多个 CI 任务定义可能会耗时，并且会导致严重的可维护性问题。如果你在管道中多个地方使用相同的变量，最好只定义一次，然后多次引用。这样，当你需要更改变量时，只需在一个地方进行更改，而不是在多个地方进行更改。这不仅更容易操作，而且更安全，因为在大型管道中，多次替换变量可能会导致错误。

GitLab 和 YAML 提供的创建可重用 CI 管道的三种方法是`extends:`关键字和`!reference`标签。在接下来的章节中，我们将解释这三种方法的优点和使用方式。每种方法在功能上都有优缺点。

## 锚点

我们要回顾的第一种方法是**锚点**。YAML 锚点允许你将一个 CI 作业的属性复制或继承到另一个作业。使用锚点，你可以定义整个 GitLab CI 作业或几个属性，然后重复使用它们。

这里，我们将使用一个基本的 YAML 锚点。正如你所看到的，`&job_definition`是我们将要设置的锚点。然后，我们将在`jobOne`和`jobTwo`中使用它，以引入来自`&job_definition`的内容：

```
.job_definition: &job_definition
  image: node:latest
  services:
    - postgres
jobOne:
  <<: *job_definition
  script:
    - npm build
jobTwo:
  <<: *job_definition
  script:
    - npm test
```

一旦 GitLab 处理了 CI 文件，结果将类似于下面的样子。在这里，`&job_definition`的内容已合并到我们的两个 CI 作业中：

```
jobOne:
  image: node:latest
  services:
    - postgres
  script:
    - npm build
jobTwo:
  image: node:latest
  services:
    - postgres
  script:
    - npm test
```

在前面的示例中，我们定义了一个以点（`.`）开头的作业。这个点意味着该 CI 作业不会作为 CI 作业执行，而只是作为一个引用存在。这个作业定义为`.job_definition:`。在这个定义之后，我们添加了一个 YAML 锚点。这里是`&job_definition`语句，您可以看到它。所有在`&`符号之后的内容都定义了锚点的名称。接着，我们定义了一个正常的作业应该是什么样子。

之后，我们定义了两个 CI 作业，每个作业都有不同的`script:`块。然而，我们使用`<<:`关键字告诉 YAML 处理器，我们希望将`.job_definition:`的属性和关键字与此作业合并。`*`字符后跟锚点名称，引用了我们用`&`符号定义的锚点。

结果是合并后的代码块中包含了这两个作业。

重要提示

YAML 锚点只能在定义它们的同一个 GitLab CI 文件中使用。这意味着，如果你正在利用`includes:`关键字将 CI 文件拆分成多个文件，你必须使用`extends:`或`!reference`，而不是 YAML 锚点。

## extends: 关键字

在 GitLab CI 文件中复用 CI 作业和配置的第二种方式是通过`extends:`关键字。`extends:`关键字和 YAML 锚点在操作方式上非常相似。一个主要的区别是，YAML 锚点通常用于在整个 CI 管道文件中复制单个值或属性，而`extends:`则更常用于复用整个配置块。在前面的锚点示例中，`extends:`更适合使用。

在下面的示例中，我们定义了一个`.rules_definition`块。然后，我们将其包含在`.job_definition`块中，并在`jobOne`和`jobTwo`中使用`.job_definition`块。任何以点（`.`）开头的作业定义不会被 GitLab 作为实际的作业处理。相反，它们被当作模板来处理：

```
.rule_definition:
  rules:
    - if: $CI_PIPELINE_SOURCE =="push"
.job_definition:
  extends: .rule_definition
  image: node:latest
  services:
    - postgres
jobOne:
  extends: .job_definition
  script:
    - npm build
jobTwo:
  extends: .job_definition
  script:
    - npm test
```

在 GitLab 处理完前面的 CI 文件后，最终合并的结果如下所示。在这里，`.rule_definition`和`.job_definition`的内容（这两个内容只定义了一次）现在包含在`jobOne`和`jobTwo`中：

```
jobOne:
  image: node:latest
  services:
    - postgres
  script:
    - npm build
  rules:
    - if: $CI_PIPELINE_SOURCE =="push"
jobTwo:
  image: node:latest
  services:
    - postgres
  script:
    - npm test
  rules:
    - if: $CI_PIPELINE_SOURCE =="push"
```

正如你所看到的，这个示例与我们用锚点的示例略有不同。这是因为锚点和`extends:`之间的另一个主要区别是，`extends:`可以从多个 CI 作业定义中继承配置。

因此，你可以看到合并后的作业定义增加了一个`rules:`属性。这个属性是通过`.job_definition` CI 作业继承自`.rules_definition` CI 作业的。

## 引用标签

复用 CI 文件配置的第三种方法是使用`!reference`标签。`!reference`标签是自定义的 YAML 标签，用于从其他 CI 作业部分选择关键字配置，并在当前部分中重用它们。它们的使用方式与 YAML 锚点类似，但你可以在多个 CI 文件中使用引用标签。另一方面，YAML 锚点只能在定义它们的同一个文件中使用。让我们来看一个例子。

创建一个看起来像这样的`Build.gitlab-ci.yml`文件：

```
.build-node:
  stage: deploy
  before_script:
    - npm install
  script:
    - npm build
```

创建一个看起来像这样的`.gitlab-ci.yml`文件：

```
include:
  - local: Build.gitlab-ci.yml
Build-My-App:
  stage: build
  script:
    - !reference [.build-node, script]
    - echo "Application is Built"
```

在 GitLab 处理这两个文件后，结果应该是这样的：

```
Build-My-App:
  stage: build
  script:
    - npm build
    - echo "Application is Built"
```

在前面的示例中，我们在`Build.gitlab-ci.yml`中创建了一个作业定义。然后，我们将这个`Build` CI 文件包含到主`.gitlab-ci.yml`文件中。之后，我们使用`!reference`关键字直接从`.build-node`作业定义中提取脚本块。通过利用`!reference`而不是`extends:`，我们可以只从该作业定义中提取我们想要的配置，而不是整个作业定义。如果我们使用了`extends:`，我们还会将该 CI 作业定义的`stage:`和`before_script:`属性一并引入。

# 通过合并多个流水线和利用父子流水线来提高可维护性

大多数 GitLab 用户只使用一个`.gitlab-ci.yml`文件来配置他们的流水线。这种方法完全可以接受，但在许多情况下，这个文件中的代码量可能会变得非常庞大且难以维护。GitLab 引入了将多个`gitlab-ci`文件作为一个文件包含在一起的功能。在本节中，我们将讨论如何将一个`.gitlab-ci.yml`文件拆分为多个部分。接下来，我们将讲解如何使用第二个`.gitlab-ci.yml`文件并执行一个子流水线，然后讨论为什么你可能会这样做的原因。

## 利用包含来提高可维护性

创建一个看起来像这样的`.gitlab-ci.yml`文件：

```
"Build Application":
  stage: build
  script:
    - code here
"Build Container":
  stage: build
  script:
    - code here
"Deploy Container":
  stage: deploy
  script:
    - code here
"Deploy Production":
  stage: deploy
  script:
    - code here
```

上述代码展示了一个传统的`.gitlab-ci.yml`文件的示例。在这个示例中，我们包含了四个作业——两个构建作业和两个部署作业。在普通的`.gitlab-ci.yml`文件中，这可能会包含数十个作业，并且它们之间有大量的逻辑，代码行数可能会达到数百行。这是完全可以接受的做法，但它难以维护和管理。与所有源代码形式一样，我们希望确保我们编写的代码在第一眼就能被理解，以保证其可维护性。

为了拆分这段代码并提高可维护性，我们可以利用 GitLab CI 语法中的 `include:` 关键字。该关键字用于告诉 GitLab 的 YAML 处理器何时将多个 YAML 文件合并为一个单一的上下文。让我们使用这个工具将我们的 CI 文件拆分成多个独立的 CI 文件，以便于重用。

创建一个 `Build.gitlab-ci.yml` 文件，其内容如下：

```
"Build Application":
  stage: build
  script:
    - code here
"Build Container":
  stage: build
  script:
    - code here
```

创建一个 `Deploy.gitlab-ci.yml` 文件，其内容如下：

```
"Deploy Container":
  stage: deploy
  script:
    - code here
"Deploy Production":
  stage: deploy
  script:
    - code here
```

创建一个 `.gitlab-ci.yml` 文件，其内容如下：

```
include: "Build.gitlab-ci.yml"
include: "Deploy.gitlab-ci.yml"
```

在这里，我们将各个作业根据阶段拆分到不同的文件中。然后，我们将这些文件包含在 `.gitlab-ci.yml` 文件中。当 GitLab 的 CI 处理器遍历 `.gitlab-ci.yml` 文件时，它会将每个 YAML 文件从上到下合并成一个单一的上下文。这意味着，如果我在 `Build.gitlab-ci.yml` 和 `Deploy.gitlab-ci.yml` 中都写了相同的作业，因为 `Deploy.gitlab-ci.yml` 最后被包含，它将覆盖 `Build` CI 文件中的内容。这是一个简单的示例和方法，用于将 CI 文件拆分以提高可维护性。接下来，我们将结合之前的示例与这种方法进行讲解。

## 利用包含以提高重用性

之前，你已经学习了如何使用包含来帮助维护性。在本节中，我们将结合使用 `include:` 和本章中关于锚点和扩展的知识，向你展示如何创建可重用的流水线。我们将从一个示例开始。

创建一个 `Templates.gitlab-ci.yml` 文件，其内容如下：

```
.npm-build:
  stage: build
  variables:
    NPM_CLI_OPTS: ""
  before_script:
    - npm install
  script:
    - npm rebuild $NPM_CLI_OPTS
```

在前面的 GitLab CI 文件中，我们创建了一个非常简单的 CI 作业定义。由于该 CI 作业的名称以 `.` 开头，因此这个作业不会单独运行。我们包含了一个 `variables:` 块，并插入了一个空的 `NPM_CLI_OPTS` 变量作为占位符。然后，我们在 `script:` 块中使用了这个变量，当我们执行 `rebuild` 命令时。设置这个默认变量的原因是，我们希望在使用该作业时，能够有一个合理的默认值。

我们将这个 CI 文件命名为 `Templates.gitlab-ci.yml`，以表明它包含的是 CI 作业模板，而不是实际的 CI 作业定义。然而，我们希望在 CI 流水线中利用它。以下示例展示了如何实现这一点。

创建一个 `.gitlab-ci.yml` 文件，其内容如下：

```
include: Templates.gitlab-ci.yml
My-NPM-Job:
  extends: .npm-build
  variables:
    NPM_CLI_OPTS: '--global'
```

在这里，我们包含了来自 `Templates.gitlab-ci.yml` 文件的所有作业定义，并基于这些定义启动了我们自己的作业。然而，在 `variables:` 块中，我们添加了自己的变量来控制模板 CI 作业定义的执行方式。

这种将 CI 作业模板化，然后使用变量来暴露配置选项的方法，类似于传统软件开发中的组件化方式。尽管语法不同，但它遵循相同的目的、规则和目标。

## 来自远程区域的包含

`include:` 关键字并不局限于单个项目或仓库。你还可以从开放的互联网中包含 GitLab CI 文件。以下是一些远程包含的示例：

+   包括来自不同项目和分支：

    ```
    include:
    ```

    ```
      - project: "my-group/my-project"
    ```

    ```
        ref: my-branch
    ```

    ```
        file: 'Templates.gitlab-ci.yml
    ```

+   包括来自远程位置：

    ```
    include:
    ```

    ```
      - remote: 'https://www.google.com/Templates.gitlab-ci.yml'
    ```

+   从 GitLab 实例的模板中包含（`lib/gitlab/ci/templates`）：

    ```
    include:
    ```

    ```
      - template: 'Templates.gitlab-ci.yml'
    ```

重要提示

你可以从 CI 流水线启动者有访问权限的任何地方包含 CI 文件。前提是启动 CI 流水线的人具有该文件的读取权限，流水线将会成功。如果启动者没有访问权限，流水线将会失败。

## 利用父子流水线

现在我们已经讨论了如何包含多个 YAML 文件，并且如何将它们模板化以便重用，接下来让我们谈谈如何使用`trigger:`关键字来创建子流水线。

**子流水线**是由另一个流水线触发的流水线。触发流水线的流水线称为**父流水线**。一旦父流水线触发子流水线，父流水线的执行将等待子流水线完成后才会继续。这是构建多个流水线来支持单体仓库，或将一个大型复杂流水线拆分为更小、更易管理流水线的强大工具。

要触发子流水线，只需将`trigger:`关键字添加到`include:`语句的父级。以下示例将会执行`build.gitlab-ci.yml`文件作为子流水线：

```
My-Child-CI-Job:
    stage: build
    trigger:
      include:
        - project: "my-group/my-project"
          ref: my-branch
          file: 'Build.gitlab-ci.yml'
```

在前面的示例中，在 CI 流水线视图中，你会看到一个名为`My-Child-CI-Job`的 CI 作业。该 CI 作业将会附带另一个名为`Downstream Pipeline`的流水线。在这个视图中，你将能够看到来自子流水线的所有作业及其执行状态。

重要提示

被触发的子流水线接受所有正常的 CI 作业属性和关键字。像`rules:`这样的关键字可以决定何时触发子流水线。`environment:`关键字还可以将子流水线的触发与审批规则或环境跟踪关联起来。

# 使用专用容器保障和加速作业

GitLab 在正确配置的情况下，会在容器中运行流水线的所有 CI 作业。这意味着整个构建操作都发生在容器中。因此，容器的管理非常重要。如果 CI 作业发生在不安全的容器中，那么整个 CI 作业和流水线都将不安全。如果 CI 作业使用性能不佳的容器，那么该作业和流水线将会花费更多时间才能完成，导致显示结果的时间大大延长。在所有可衡量的方面，CI 作业使用的容器是流水线中最重要的部分。

重要提示

要快速设置或识别某个 CI 作业使用的容器，查找 CI 作业中的`image:`属性。该属性将定义容器镜像的来源，以及正在使用的具体容器镜像。

查找此容器镜像的第二个位置是在 CI 作业日志的顶部。那里会有一条消息，指示当前使用的容器镜像。

我们的目标是通过一种称为*专用容器*的实践来解决这些问题。这些容器的整个设计都是用于在 CI 流水线中使用。在这里，我们将概述这些容器的一些特性，并解释如何构建它们。在为 CI 流水线构建容器时，请尽可能遵循这些指导原则。即使您无法实现此处提到的每一项，尽可能遵循这些指导原则将使您的 CI 流水线容器更安全、更高效和更易于维护。

考虑的第一个项目是容器的文件大小。用于 GitLab CI 的容器应仅包含运行和执行其任务所需的最少组件。此容器还应只执行一组任务。这意味着您会为每个工具链单独创建一个容器 - 也就是说，一个用于 Java，一个用于 Node.js。您不希望混合多个工具链，因为这会增加容器的文件大小。很少有情况需要将多个工具链放在同一个容器中执行，因为它们通常在单独的 CI 作业中执行。一个好的遵循规则是，如果 CI 作业不需要某些东西，那么容器中就不应该包含该东西。

考虑的第二个项目是确保您的容器是多用途的。您仍然希望将工具链保持在单独的容器中；但是，您应避免将配置嵌入到只能用于多个 CI 作业或多个流水线的容器中。一个好的例子是包括加密证书，以便容器可以与其需要的任何资源通信。不应包括的示例是仅与单个 CI 作业或流水线相关的任何配置或设置。这两个示例之间的区别在于，第一个示例（证书）用于使容器能够正常工作，而第二个示例（配置或设置）将限制容器可以运行的 CI 作业和流水线。

考虑的第三个项目是防止容器在正常使用时运行。专为 GitLab CI 构建的容器应有效地成为僵尸或外壳。容器执行除了 CI 作业指示其执行的任何内容之外的情况都不应该存在。这可以通过确保容器的入口点为空来实现。如果 Docker 容器在 GitLab CI 中启动时执行任何操作，可能会导致冲突，并且启动时间也会更长。

第四个需要考虑的事项是避免在 Docker 容器中添加不必要的层。每当 Dockerfile 中使用`RUN`或`ADD`命令时，Docker 都会创建一个新层。不必要的层可能会显著增加 Docker 容器的大小，从而违反第一个考虑事项。当你在 Dockerfile 中运行命令时，应充分使用`&`操作符将多个命令链接在一起。我们在 Dockerfile 示例中提供了这种用法的例子。

最后一个需要考虑的事项是权限。在 GitLab CI 中运行的 Docker 容器通常不需要提升的权限来运行和执行操作。在 OpenShift 和一些 Kubernetes 平台上，任何具有提升权限的 Docker 容器可能都不允许运行和执行。创建容器时设置随机用户 ID 有助于防止这些平台赋予容器任何形式的特权。如果需要为文件或文件夹提供权限，应将权限授予组，而不是单个用户。

## 一个定制容器示例

在这里，我们提供了一个定制容器的示例。该示例遵循了我们之前列出的所有考虑事项。这是一个基于`alpine:3.12.0`基础容器构建的 Docker 镜像。在此基础上，我们有一个`RUN`命令，该命令将多个命令合并为一行。它利用`&`操作符将多个 APK 包管理器命令连接在一起。这减少了 Docker 文件中的层数。在该命令行的末尾，我们将一个文件夹分配给运行用户的组。通过这样做，我们保留了操作该文件夹的能力，但防止它通过错误的组分配获得任何特权：

```
FROM alpine:3.12.0
RUN apk update && apk add –no-cache nodejs npm && mkdir ~/.npm && chmod -R g=u ~/.npm
USER 1001
CMD ["echo", "This is a purpose built container. It is meant to be used in a pipeline and not executed."]
```

对于第一个考虑事项，你可以看到它通过`apk add –no-cache nodejs npm`来满足。Node.js 是该容器中唯一安装的工具链。

对于第二个考虑事项，你可以看到 Dockerfile 中没有嵌入任何配置。这意味着容器将从 CI 作业的配置中获取所有配置。

对于第三个考虑事项，你可以查看以`CMD ["echo"`开头的那一行。该行防止容器在 CI 作业之外运行。它显示错误信息并终止容器。

对于第四个考虑事项，请查看以`RUN apk`开头的那一行。该行将多个命令通过`&`连接在一起。每个容器中的`RUN`命令都会创建一个新的层。这里，我们尽量使用尽可能少的`RUN`命令。

对于第五个考虑事项，我们使用`USER 1001`关闭容器。这一行强制容器中的所有命令以随机用户 ID 运行。这意味着没有任何命令会以任何形式的提升权限运行。

# 总结

在本章中，我们学习了许多可以与 GitLab CI 一起使用的工具，用于创建快速且可重用的流水线。我们从 DAG（有向无环图）开始，它可以使流水线执行得更快。接着，我们学习了如何为多种架构构建代码。随着主流 ARM 平台的出现，这一点将随着时间的推移变得越来越重要。随后，我们学习了在流水线中何时利用缓存或工件进行依赖管理。

我们讨论的最后两个主题可能是最重要的。我们学习了三种不同的方法来构建可重用的流水线定义，这样你就不必在多个地方重复编写相同的逻辑。最后，我们了解了一个名为*专用流水线*的概念，它使你能够构建快速、安全且稳定的容器来执行你的 CI 工作负载。

在下一章中，你将学习如何扩展你的 CI/CD 流水线的覆盖范围。
