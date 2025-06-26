

# 第七章：Git，您的 DevOps 之门

Git 是一个免费的开源**版本控制系统**（**VCS**），被软件开发者和团队广泛使用，用于跟踪代码库的更改并进行协作。它允许多人在同一个代码库上工作而不会覆盖彼此的更改，并记录每次对代码所做的更改，使得在必要时可以轻松回滚到以前的版本。

Git 是由 Linus Torvalds 于 2005 年为 Linux 内核的开发而创建的，后来它成为了软件行业版本控制的事实标准。全球数百万开发者使用 Git，且它得到了一个庞大且活跃的开源社区的支持。

在本章中，我们将介绍最常用的 Git 命令以及如何使用它们。我们将从设置 Git 仓库并进行首次提交的基础开始，然后继续讲解更高级的话题，如分支和合并。

本章将涵盖以下内容：

+   基本的 Git 命令

+   本地与远程 Git 仓库

+   GitFlow 和 GitHub Flow

# 技术要求

在本章中，您需要一台具有 Bash shell 的系统。您需要确保该系统已安装 Git 命令或能够安装它。我们推荐使用 Linux 或 macOS 系统，但也可以在 Windows 上设置功能齐全的 Bash 和 Git。本书并不涉及该环境的安装。

# 基本的 Git 命令

您可以使用很多 Git 命令，但最常用的一些命令包括：

+   `git config`：此命令用于配置本地 Git 环境。配置可以是全局性的，这时配置值会保存在 `.gitconfig` 文件中，位于您的家目录下。值也可以仅在某个仓库中设置，并保存在该仓库内。

+   `git init`：此命令初始化一个新的 Git 仓库。当您在某个目录中运行此命令时，它会在项目根目录下创建一个新的 `.git` 目录，用于跟踪项目文件所做的更改。

+   `git clone`：此命令创建一个远程 Git 仓库的本地副本。当您运行此命令时，它会创建一个与仓库同名的新目录，并将所有文件及其历史记录克隆到该目录中。

+   `git add`：此命令用于暂存文件以供提交。当您在 Git 仓库中修改文件时，这些更改不会被自动跟踪。您必须使用 `git add` 命令告诉 Git 跟踪您所做的更改。

+   `git commit`：此命令将您的更改保存到 Git 仓库中。当您运行此命令时，它会打开一个文本编辑器，让您编写提交信息，即您所做更改的简短描述。在编写并保存提交信息后，您所做的更改将被保存到仓库中，并创建一个新的提交。

+   `git push`：此命令将本地提交推送到远程仓库。当你运行此命令时，它会将所有本地提交推送到远程仓库，更新项目历史，并使你的更改对其他开发者可见。

+   `git pull`：此命令从远程仓库获取更新并将其合并到本地仓库。当你运行此命令时，它会从远程仓库获取最新的更改，并将它们合并到本地仓库中，使你的项目副本保持最新。

+   `git branch`：此命令用于在 Git 仓库中创建、列出或删除分支。分支允许你同时处理项目的多个版本，通常用于功能开发或修复 bug。

+   `git checkout`或`git switch`：此命令用于在分支之间切换或恢复工作目录中的文件。当你运行此命令时，它会将工作目录切换到你指定的分支，或将指定的文件恢复到先前提交的状态。

+   `git merge`：此命令将一个分支合并到另一个分支。当你运行此命令时，它会将指定分支中的更改合并到当前分支，并创建一个新的提交，表示合并后的更改。

+   `git stash`：此命令用于暂时存储你还未准备好提交的更改。当你运行此命令时，它会将你的更改保存到一个临时区域，并将工作目录恢复到上一次提交的状态。稍后，你可以使用`git stash apply`命令将存储的更改恢复到工作目录。

## 配置本地 Git 环境

在使用 Git 命令之前，至少有两个选项需要设置。它们是你的姓名和电子邮件地址。可以通过`git config`命令来完成此操作。接下来，我将演示如何设置 Git 用户的姓名和电子邮件地址。我们将设置全局变量。除非专门为某个仓库单独设置，否则这些全局变量将在每个本地克隆的仓库中默认使用：

```
admin@myhome:~$ git config –global user.name "Damian Wojsław"
admin@myhome:~$ git config –global user.email damian@example.com
```

现在，当我们查看`~/.gitconfig`文件时，会看到以下部分：

```
# This is Git's per-user configuration file
[user]
    name = Damian Wojsław
    email = damian@example.com
```

还有更多配置选项，如默认编辑器等，但它们超出了本节内容的范围。

## 设置本地 Git 仓库

在你开始使用 Git 之前，需要创建一个**仓库**（也称为**repo**）。仓库是 Git 存储项目所有文件和元数据的目录。

要创建一个新的仓库，你可以使用`git init`命令。此命令会创建一个新的目录，并在其中生成一个`.git`子目录，包含仓库所需的所有文件。

例如，要在当前目录中创建一个新的仓库，你可以运行以下命令：

```
admin@myhome:~$ mkdir git-repository && cd git-repository
admin@myhome:~/git-repository$ git init
```

这会在当前目录中创建一个新的仓库，并设置必要的文件和元数据。一旦你设置了 Git 仓库，就可以开始向其中添加和提交文件。

要将一个文件添加到代码库中，你可以使用`git add`命令。这个命令会将文件添加到暂存区，暂存区是一个包含下次提交将包括的更改的列表。

要将一个名为`main.c`的文件添加到暂存区，你可以运行以下命令：

```
admin@myhome:~/git-repository$ git add main.py
```

你也可以通过用空格分隔文件或目录的名称来一次性添加多个文件：

```
admin@myhome:~/git-repository$ git add main.py utils.py directory/
```

要提交暂存区中的更改，你可以使用`git commit`命令。这会创建一个新的提交，其中包括暂存区中的所有更改。

每个提交都需要有提交信息。这是对你所做更改的简短描述。要指定提交信息，你可以使用`-m`选项，后跟消息内容：

```
admin@myhome:~/git-repository$ git commit -m "Added main.py and additional utils"
```

你也可以使用`git commit`命令而不加`-m`选项，这样会打开一个文本编辑器，你可以在其中编写更详细的提交信息。

在你进行 Git 操作的每个步骤中，你都可以使用`git status`命令。这用于查看 Git 代码库的当前状态。它显示哪些文件已被修改、添加或删除，以及它们是否已准备好提交。

有许多种格式化提交信息的方法，几乎每个项目都有自己的约定。一般来说，一个好的做法是添加一个来自问题跟踪系统的`issue` ID，然后是一个简短的描述，描述字符数不超过 72 个。第二行应留空，第三行跟随更详细的描述。以下是这样的提交信息示例：

```
[TICKET-123] Adding main.py and utils.py
main.py contains the root module of the application and is a default entry point for a Docker image.
utils.py contains helper functions for setting up the environment and reading configuration from the file.
Co-authors: other-commiter@myproject.example
```

当你运行`git status`时，它将显示已修改的文件列表，以及代码库中任何未跟踪的文件。它还会显示当前分支和暂存区的状态。

以下是`git status`命令输出的示例：

```
admin@myhome:~/git-repository$ git status
On branch main
Your branch is up to date with 'origin/main'.
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        modified:   main.py
        deleted:    root/__init__.py
Untracked files:
  (use "git add <file>..." to include in what will be committed)
        ui/main.py
no changes added to commit (use "git add" and/or "git commit -a")
```

在这个例子中，`git status`显示了两个已修改的文件（`main.py`和`__init__.py`）以及一个未跟踪的文件（`main.py`）。它还指出当前分支是`main`，并且该分支与`origin/main`分支保持同步。

`git status`是一个有用的工具，可以帮助你了解当前代码库的状态，并识别哪些文件已经被修改并需要提交。你可以使用它快速查看已做的更改，并看到哪些文件已经准备好提交。它还可以显示你当前所在的分支，以及你是否与远程代码库保持同步。

`git revert`命令用于撤销已经对代码库所做的更改。它通过创建一个新的提交来撤销之前提交所引入的更改。

例如，假设你已经向你的代码库提交了几次更改，现在你想撤销上次提交所引入的更改。你可以使用`git revert`命令来实现：

```
admin@myhome:~/git-repository$ git revert HEAD
```

这将创建一个新的提交，撤销上次提交所引入的更改。你的代码库的提交历史现在看起来就像是上次提交从未发生过一样。

你也可以使用 `git revert` 命令撤销特定提交所引入的更改。为此，你需要指定你想撤销的提交的哈希值：

```
admin@myhome:~/git-repository$ git revert abc123
```

这将创建一个新的提交，撤销由 `abc123` 哈希值的提交所引入的更改。

需要注意的是，`git revert` 并不会删除提交或销毁历史记录。相反，它会创建一个新的提交，撤销之前提交所引入的更改。这意味着更改仍然存在于仓库中，但在当前分支的状态下不可见。

如果你需要永久删除提交，可以使用 `git reset` 命令或 `git filter-branch` 命令。然而，这些命令可能会永久销毁历史，因此使用时需要非常小心。

现在我们已经了解如何在本地仓库上工作，接下来我们可以讨论远程和本地仓库的副本。

# 本地与远程 Git 仓库

在 Git 中，**仓库**是文件及其历史的集合，以及用于管理仓库的配置文件。仓库可以是本地的，也可以是远程的。

执行 `git init` 命令或使用 `git clone` 命令克隆一个现有仓库时，你正在创建一个本地仓库。本地仓库对于在没有互联网连接时或当你希望将项目的副本保存在自己机器上时非常有用。

当你执行 `git push` 命令时，你正在用本地更改更新远程仓库。远程仓库对于与其他开发者合作非常有用，因为它们允许多人共同工作并互相共享更改。

Git 使用分布式版本控制系统（VCS），这意味着每个开发者的本地机器上都有仓库的完整副本。这允许开发者在本地工作，完成后将更改推送到远程仓库，与他人共享。此外，即使开发者没有连接到互联网，也可以在项目上进行协作。

`git clone` 命令用于创建一个远程 Git 仓库的本地副本。当你运行此命令时，它会创建一个与仓库同名的新目录，并将所有文件及其历史克隆到该目录中。

这是你可能如何使用 `git` `clone` 命令的示例：

```
admin@myhome:~/git-repository$ git clone https://github.com/user/repo.git
```

这将创建一个名为 `repo` 的新目录，并将位于 [`github.com/user/repo.git`](https://github.com/user/repo.git) 的仓库克隆到该目录中。

你还可以通过将目录名作为参数添加，来指定本地目录的不同名称：

```
admin@myhome:~/git-repository$ git clone https://github.com/user/repo.git my-local-repo
```

这将创建一个名为 `my-local-repo` 的新目录，并将仓库克隆到其中。

如果你使用像 GitHub 这样的 Git 托管服务，你也可以使用仓库 URL 的简写版本：

```
admin@myhome:~/git-repository$ git clone git@github.com:user/repo.git
```

这将使用 SSH 协议克隆该仓库，这要求你为你的账户配置了一个 SSH 密钥。

`git clone`命令是创建远程仓库本地副本的有用方法，无论是开始一个新项目还是为现有项目做贡献。它允许你在本地工作，准备好后将更改推送回远程仓库。

## 与远程仓库交互

`git pull`命令用于从远程仓库获取更新，并将其合并到你的本地仓库中。它是`git fetch`命令（用于从远程仓库下载更新）和`git merge`命令（用于将更新合并到本地仓库）的一种组合。

`git fetch`命令从远程仓库下载更新，但不会将其合并到本地仓库中。相反，它将更新存储在一个名为*remote-tracking branches*的临时区域中。你可以使用`git fetch`命令更新你的远程跟踪分支，查看有哪些更改可用，但你需要使用`git merge`命令将这些更改实际合并到本地仓库中。

下面是一些你在使用`git fetch`时可能会用到的常见关键字：

+   `origin`：这是你从中克隆的远程仓库的默认名称。你可以使用`origin`来指定你想要从中获取更新的远程仓库。也可以更改默认名称并添加多个远程仓库。

+   `main`或`master`：`master`是 Git 仓库中默认分支的名称。`main`是 GitHub 平台引入的新默认名称。

+   `REMOTE_HEAD`：这是一个特殊的引用，指向远程仓库中分支的`head`提交。你可以使用`REMOTE_HEAD`来获取远程仓库当前检出的分支的更新。

+   `HEAD`：这是一个特殊的引用，指向当前分支在本地仓库中的`head`提交。

这是使用`git fetch`更新`origin`仓库中`main`分支的一个示例：

```
admin@myhome:~/git-repository$ git fetch origin main
From gitlab.com:gstlt/git-repository
 * branch            main       -> FETCH_HEAD
```

这将为`origin`仓库中的`master`分支下载更新，并将其存储在`origin/master`远程跟踪分支中。

然后，你可以使用`git merge`命令将更新合并到本地仓库中：

```
admin@myhome:~/git-repository$ git merge origin/main
Already up to date.
```

这将把`origin/master`远程跟踪分支的更新合并到你的本地`master`分支中。在这种情况下，我们没有需要合并的内容。

或者，你可以使用`git pull`命令将这两个步骤合并为一个命令来执行：

```
admin@myhome:~/git-repository$ git pull origin main
Already up to date.
```

这将从`origin`仓库获取更新，并将它们合并到本地仓库中，除非你已经按照本章开头的建议配置了 Git 客户端——在这种情况下，Git 将尝试在远程仓库的`main`分支上进行变基。

`git rebase`和`git merge`是用于将一个分支的更改合并到另一个分支中的命令。然而，它们的工作方式略有不同，对你的仓库也有不同的影响。

`git rebase`是一个用于将一个分支的一系列提交应用到另一个分支的命令。当你运行`git rebase`时，它会查看目标分支中有而源分支中没有的提交，并将它们逐个应用到源分支。这种操作的效果是*重新播放*源分支上的提交，并将其应用到目标分支之上。

例如，假设你在代码库中有两个分支：`main`和`develop`。`main`代表主开发分支，`develop`代表你正在处理的某个功能。如果你对`develop`分支进行几次提交，然后运行`git rebase main`，Git 会将这些提交一个接一个地应用到`main`分支，就好像你直接在`main`分支上做了这些提交一样。

然而，`git merge`是一个用于将一个分支的更改合并到另一个分支的命令。当你运行`git merge`时，它会查看源分支中的更改，并将它们以单个提交的形式应用到目标分支。这种操作的效果是，在目标分支上创建一个新的提交，将源分支中的所有更改合并进来。

例如，你有与 rebase 示例中相同的两个分支：`main`和`develop`。如果你对`develop`分支进行几次提交，然后运行`git merge develop`，Git 会在`main`分支上创建一个新的提交，将`develop`分支中的所有更改合并进来。

`git rebase`和`git merge`都是将一个分支的更改合并到另一个分支的有用工具，但它们对代码库有不同的影响。`git rebase`保持线性历史，并避免不必要的额外（合并）提交，但如果目标分支自源分支创建以来已有修改，它也可能会引发冲突。

`git merge`是合并更改的更直接方式，但它可能会创建许多合并提交，这会使得代码库的历史记录变得更加难以阅读，即使使用图形化工具也一样。

在合并或将本地代码库与远程版本进行 rebase 之前，检查两者之间的差异是很有用的。为此，我们可以使用`git` `diff`命令。

## 什么是 git diff 命令？

`git diff`命令用于比较对代码库所做的更改。它显示文件的两个版本或代码库中两个分支（本地或远程）之间的差异。

以下是`git` `diff`命令的一些常见用法。

比较文件当前状态和最新提交之间的差异：

```
admin@myhome:~/git-repository$ git diff path/to/file
```

这将显示文件当前状态与最后一次提交的版本之间的差异。

比较两个提交之间的差异：

```
admin@myhome:~/git-repository$ git diff hash1 hash2
```

这将显示你提供的两个提交之间的差异。

比较一个分支和其上游分支之间的差异：

```
admin@myhome:~/git-repository$ git diff branch-name @{u}
```

这将显示指定分支与其上游对应分支之间的差异。

比较暂存区与最新提交之间的差异：

```
admin@myhome:~/git-repository$ git diff --staged
```

这将显示已添加到暂存区与最新提交之间的差异。

如果你已经添加了一些文件准备提交，但在创建提交之前想再次确认，可以使用以下命令：

```
admin@myhome:~/git-repository$ git diff --cached
```

这将显示你执行了`git add`命令的所有文件的更改。

要比较同一仓库的本地和远程分支，你需要引用远程分支，可以通过执行`git diff`来实现：

```
admin@myhome:~/git-repository$ git diff main origin/main
```

`origin/main`指的是一个远程分支，其中远程仓库被命名为“origin”。

`git diff`对于理解 Git 仓库中的变化以及这些变化如何在提交时被合并非常有用。你可以使用`git diff`命令的各种选项和参数来指定要比较的文件版本或分支。

既然我们已经讨论了如何比较仓库中的更改，这里有一个特别的场景，这项技能非常有用：在重新基准（rebase）或合并（merge）更改时解决仓库中的冲突。

## 查看提交历史

Git 会记录每一次对仓库进行的提交，你可以使用`git log`命令查看提交历史。这将显示仓库中所有提交的列表，包括提交信息和提交日期。

`git log`命令的常见用法包括以下几种：

+   使用`git log`查看仓库的完整提交历史，包括已做的更改及这些更改的原因。当你在一个项目中工作并且需要理解它是如何随着时间演变时，这非常有帮助。

+   使用`git log`查看受影响文件的提交历史，并查看哪个提交引入了这个 bug。

+   使用`git log`查看受影响文件的历史，并查看哪些更改可能导致了问题。

你可以使用多种选项配合`git log`来定制其输出并筛选显示的提交。例如，你可以使用`--author`选项仅显示由特定人员提交的内容，或者使用`--since`选项仅显示过去一个月内的提交。

下面是一些使用`git log`命令的示例。

要显示当前仓库的提交历史，请调用`git log`命令：

```
admin@myhome:~/git-repository$ git log
commit 9cc1536fb04be3422ce18a6271ab83f419840ae3 (HEAD -> main, origin/main, origin/HEAD)
Author: Grzegorz Adamowicz <grzegorz@devopsfury.com>
Date:   Wed Jan 4 10:13:26 2023 +0100
    Add first version of the table schema
commit fb8a64f0fd7d21c5df7360cac427668c70545c83
Author: Grzegorz Adamowicz <grzegorz@devopsfury.com>
Date:   Tue Jan 3 22:02:25 2023 +0100
    Add testing data, mock Azure class, remove not needed comments
commit ae0ac170f01142dd80bafcf40fd2616cd1b1bc0b
Author: Grzegorz Adamowicz <grzegorz@devopsfury.com>
Date:   Tue Dec 27 14:50:47 2022 +0100
    Initial commit
```

你可以通过运行以下命令查看特定文件的提交历史：

```
admin@myhome:~/git-repository$ git log path/to/file
```

使用此命令，你可以显示特定分支的提交历史：

```
admin@myhome:~/git-repository$ git log branch-name
```

这将显示指定分支的提交历史，仅显示该分支上的提交。

你可以使用此命令显示一段提交范围内的提交历史：

```
admin@myhome:~/git-repository$ git log hash1..hash2
```

这将显示`hash1`提交 ID 和`hash2`提交 ID 之间的提交历史，仅显示发生在该范围内的提交。

`git log --oneline`将以紧凑的格式显示提交历史，仅显示每个提交的提交哈希值和消息：

```
admin@myhome:~/git-repository$ git log --oneline
```

使用此命令，你可以显示包含差异的提交历史：

```
admin@myhome:~/git-repository$ git log -p
```

这将显示提交历史，并展示每个提交的差异，显示每个提交中所做的更改。

这些只是你可以使用`git log`命令的一些示例。你可以在 Git 文档中找到更多关于可用选项的信息。

在下一部分，我们将探讨在将更改合并回`main`分支之前如何缩短我们的 Git 历史记录。如果我们在本地开发分支中有很长的提交历史，这会非常有用。

## 分支管理

`git branch`和`git switch`是两个用于管理 Git 仓库中分支的命令。

`git branch`是一个用于创建、列出或删除仓库中分支的命令。你可以通过指定分支名称作为参数来创建一个新的分支，像这样：

```
admin@myhome:~/git-repository$ git branch new-branch
```

这将创建一个名为`new-branch`的新分支，该分支基于当前分支。

你可以使用`git branch`命令配合`-a`选项列出仓库中的所有分支，包括本地分支和远程跟踪分支：

```
admin@myhome:~/git-repository$ git branch -a
```

你可以使用`git branch`命令配合`-d`选项删除一个分支，示例如下：

```
admin@myhome:~/git-repository$ git branch -d old-branch
```

这将删除`old branch`，但仅在它已经完全合并到上游分支的情况下。如果你希望删除此分支，可以添加`--force`选项，或使用`-D`选项，它是`--delete --force` Git 分支的别名。

最后，要删除一个远程分支，我们需要使用`git push`命令：

```
admin@myhome:~/git-repository$ git push origin --delete old-branch
```

这是一个具有破坏性的命令，无法撤销，因此在使用时应该谨慎操作。

`git switch`是一个用于在仓库中切换分支的命令。你可以通过指定分支名称作为参数来切换到不同的分支，像这样：

```
admin@myhome:~/git-repository$ git switch new-branch
```

这将切换到`new-branch`分支，并将其设置为当前分支。

`git branch`和`git switch`命令都允许你创建新分支、列出可用分支，并根据需要在分支之间切换。在接下来的部分，我们将介绍两种与 Git 工作相关的方法，称为工作流：`git workflow`和`github workflow`。

## 使用交互式变基合并提交

要使用`git rebase`压缩提交，你首先需要确定你想要压缩的提交范围。通常通过指定范围内第一个提交的哈希值和最后一个提交的哈希值来完成此操作。例如，如果你想压缩仓库中的最后三个提交，可以使用以下命令：

```
admin@myhome:~/git-repository$ git rebase -i HEAD~3
```

这将打开一个编辑器窗口，显示最近三次提交的列表，并附带一些说明。每个提交在文件中占据一行，行首是`pick`。要压缩某个提交，你需要将`pick`改为`squash`或`s`。

以下是文件可能的示例：

```
pick abc123 Add feature X
pick def456 Fix bug in feature X
pick ghi789 Add test for feature X
```

若要将第二个和第三个提交压缩到第一个提交中，你需要将文件更改为如下所示：

```
pick abc123 Add feature X
squash def456 Fix bug in feature X
squash ghi789 Add test for feature X
```

在做出更改后，你可以保存并关闭文件。Git 将应用这些更改，并向你展示另一个编辑器窗口，你可以在此输入合并提交的新提交消息。

需要注意的是，使用`git rebase`压缩提交可能是一个具有破坏性的操作，因为它会永久改变仓库的提交历史。一般建议在使用`git rebase`时小心，并确保在使用前有仓库的备份。

如果你想撤销`git rebase`操作所做的更改，可以使用`git rebase --abort`命令来丢弃这些更改，并将仓库恢复到先前的状态。

在成功压缩提交后，你可以将其推送到远程仓库，但必须使用`git push --force`命令，这将忽略你刚才重写的提交历史。这是一个破坏性操作，无法撤销，因此在使用`--force`选项推送更改前，请务必进行多次确认。

## 解决 Git 冲突

当你尝试合并或变基（rebase）存在冲突更改的分支时，可能会发生冲突。这通常是因为两个分支中相同的代码行都有修改，而 Git 无法自动解决这些冲突。

当在合并或变基过程中发生冲突时，Git 会在受影响的文件中标记冲突的行，你需要手动解决这些冲突才能继续操作。

下面是一个在合并过程中解决冲突的示例。

运行`git merge`来合并两个分支：

```
admin@myhome:~/git-repository$ git merge branch-to-merge
```

Git 将检测到任何冲突，并标记受影响文件中的冲突行。冲突的行将被`<<<<<<<`、`=======`和`>>>>>>>`标记包围：

```
<<<<<<< HEAD
something = 'this is in our HEAD';
=======
something = 'this is in our branch-to-merge';
>>>>>>> branch-to-merge
```

打开受影响的文件并解决冲突，选择你想保留的代码版本。你可以保留当前分支（`HEAD`）的版本，或是你正在合并的分支（`branch-to-merge`）的版本：

```
something = 'this is in our branch-to-merge';
```

使用`git add`暂存已解决的文件：

```
admin@myhome:~/git-repository$ git add path/to/file
```

继续合并或变基操作，运行`git rebase --continue`或`git merge --continue`：

```
admin@myhome:~/git-repository$ git merge --continue
```

在合并或变基过程中解决冲突可能是一个繁琐的过程，但它是使用 Git 时非常重要的一部分。通过仔细审查冲突的更改并选择正确的代码版本，你可以确保仓库的一致性并避免错误。

另外，定期将本地仓库与源分支同步，在合并或变基之前是一个好主意，以防您遇到无法解决的冲突或其他问题。这有助于您在过程中恢复任何可能发生的错误或意外。

还有一些非常棒的图形化工具，可以以更互动的方式解决冲突。如果您使用的是`KDiff3`、`WinMerge`、`Meld`或其他类似工具。

在本节中，我们已解释了分支、仓库、本地和远程仓库、合并和变基的概念。接下来，我们将研究浏览仓库的历史更改并进行修改。

# GitFlow 和 GitHub Flow

`develop`和`master`。`develop`分支用于开发新特性，而`master`分支表示当前的生产就绪状态。还有几个支持分支，如`feature`、`release`和`hotfix`，用于管理软件的开发、发布和维护过程。

使用拉取请求（pull requests）向`master`分支进行合并。没有单独的`develop`分支，`master`分支始终被认为是当前的生产就绪版本。

另一种非常相似的分支模型是**GitLab Flow**。它用于管理软件项目的开发和维护，专门设计用于与 GitLab 一起使用，GitLab 是一个基于 Web 的 Git 仓库管理工具，提供**源代码管理**（**SCM**）、**持续集成**（**CI**）等功能。

在 GitLab Flow 模型中，所有开发都在分支中进行，新特性通过合并请求合并到`master`分支。没有单独的`develop`分支，`master`分支始终被认为是当前的生产就绪版本。然而，GitLab Flow 确实包含一些附加功能和工具，如使用受保护分支和合并请求审批的功能，这可以帮助团队执行开发最佳实践并保持高水平的代码质量。

在下一节中，我们将深入研究如何使用配置文件根据我们的需求来配置 Git。

## 全局 Git 配置 - .gitconfig

`.gitconfig`文件是一个用于设置 Git 全局选项的配置文件。它通常位于用户的`home`目录中，可以使用任何文本编辑器进行编辑。

您可能想在`.gitconfig`中包含的一些常见选项如下：

+   `user.name` 和 `user.email`：这些选项指定将与您的 Git 提交关联的用户名和电子邮件地址。正确设置这些选项非常重要，因为它们将用于识别您作为提交的作者。

+   `color.ui`：此选项启用或禁用 Git 中的彩色输出。您可以将此选项设置为`auto`，以便在 Git 在支持彩色输出的终端中运行时启用彩色输出，或者将其设置为`true`或`false`，分别表示始终启用或禁用彩色输出。

+   `core.editor`：此选项指定 Git 在需要你输入提交信息或其他输入时使用的文本编辑器。你可以将此选项设置为你喜欢的文本编辑器命令，如`nano`、`vi`或`emacs`。

+   `merge.tool`：此选项指定 Git 在合并分支时解决冲突的工具。你可以将此选项设置为可视化合并工具的命令，如`kdiff3`、`meld`或`tkdiff`。

+   `push.default`：此选项指定当你没有指定分支时，`git push`命令的行为。你可以将此选项设置为`simple`，它会将当前分支推送到远程的同名分支，或者设置为`upstream`，它会将当前分支推送到它正在跟踪的远程分支。

+   `alias.*`：这些选项允许你为 Git 命令创建别名。例如，你可以将`alias.st`设置为`status`，这将允许你使用`git st`命令代替`git status`。

以下是一个示例`.gitconfig`文件，使用了前面提到的选项，并在每个部分后面添加了注释：

```
[user]
name = Jane Doe
email = jane@doe.example
[color]
    ui = always
```

`user`部分定义了每个提交的作者用户名和电子邮件。`color`部分启用颜色，以便提高可读性。`ui = always`将始终启用颜色，适用于所有输出类型（无论是机器消费还是其他）。其他可能的选项有`true`、`auto`、`false`和`never`。

`alias`部分允许你在使用 Git 时简化一些长命令。它将在左侧定义一个别名，用于 Git 命令，你可以将它添加到等号后面。

这是一个示例：

```
[alias]
  ci = commit
```

我们将`ci`命令定义为`commit`。在将此内容添加到你的`.gitconfig`文件中后，你将获得另一个 Git 命令：`git ci`，它实际上会执行`git commit`命令。你可以为日常使用的常见 Git 命令添加别名。以下命令告诉 Git 去哪里查找默认的`.gitignore`文件：

```
[core]
    excludesfile = ~/.gitignore
```

以下的`push`设置更改了默认的推送行为，这将要求你指定要将本地分支推送到哪个远程分支：

```
[push]
  default = current
```

通过指定`current`选项，我们指示 Git 尝试将本地分支推送到一个与本地分支名称完全相同的远程分支。其他选项在这里列出：

+   `nothing`：不尝试推送任何内容。

+   `matching`：将所有本地和远程名称相同的分支视为匹配并推送所有匹配的分支。

+   `upstream`：将当前分支推送到上游分支。

+   `simple`：这是默认选项。如果本地分支的名称与远程上游分支的名称不同，它将拒绝推送到上游。

当运行`git pull`命令时，Git 会尝试将远程提交合并到你的本地分支。默认情况下，它会尝试进行上游合并，这可能会导致不必要的合并提交。这将改变默认行为为`rebase`：

```
[pull]
rebase = false
```

更多信息请参见 *本地与远程 Git 仓库* 部分。

## 使用 `.gitignore` 配置文件忽略某些文件

`.gitignore` 文件是一个配置文件，用来告诉 Git 在跟踪仓库中的更改时，哪些文件或目录需要被忽略。如果你有一些由构建过程生成的文件，或者是特定于本地环境的文件，或者是与项目无关且不需要被 Git 跟踪的文件，这个功能会非常有用。

下面是一些你可能会在 `.gitignore` 文件中包含的文件和目录类型的示例：

+   `*.tmp`、`*.bak` 或 `*.swp` 文件。

+   `*.exe`、`*.jar`、`*.war` 文件，以及 `bin/`、`obj/` 或 `dist/` 目录。

+   如果你使用 `npm`，则为 `node_modules/` 目录；如果你使用 Composer，则为 `vendor/` 目录。

+   如果你使用的是 JetBrains IDEs，则为 `idea/` 目录；如果你使用的是 Visual Studio Code，则为 `.vscode/` 目录。

+   **敏感信息**：包含密码或 API 密钥的文件。我们强烈建议特别小心忽略这些文件，以防它们被提交到仓库中。这将为你避免许多麻烦和不必要的风险。现在，像 GitHub 的 *Dependabot* 这样的机器人会在这些敏感文件进入你的仓库时发出警报（甚至阻止提交），但最好还是在开发过程早期就避免这些问题。

这是一个示例 `.gitignore` 文件，它会忽略一些常见类型的文件：

```
# Temporary files or logs
*.tmp
*.bak
*.swp
*.log
# Build artifacts
bin/
obj/
dist/
venv/
*.jar
*.war
*.exe
# Dependencies
node_modules/
vendor/
requirements.txt
# IDE-specific files
.idea/
.vscode/
# Sensitive information
secrets.txt
api_keys.json
```

也可以使用通配符和 `.gitignore` 文件来处理某些目录中需要特殊规则的情况，而这些规则不希望在全局范围内生效。

最后，你可以将你全站范围的 `.gitignore` 文件放入你的主目录，以确保不会提交任何本地开发所需的文件。

# 总结

Git 是一个非常强大的工具，很难将其所有功能浓缩到一本书中。在这一章中，我们学习了作为 DevOps 工程师在日常工作中最基本的任务，掌握了处理大多数问题所需的技能。

在下一章，我们将重点介绍 Docker 和容器化，并将在那里将你迄今为止获得的所有技能付诸实践。
