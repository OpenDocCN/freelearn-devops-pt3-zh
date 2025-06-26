# 第十二章：使用 Packer 创建不可变基础设施

即使在云计算时代，管理计算基础设施仍然是一个挑战。随着容器化、**虚拟机** (**VMs**) 和无服务器计算的创新，开发人员可能认为计算基础设施已经是一个解决了的问题。

事实远非如此。对于云服务提供商或其他运营自己的数据中心的公司，裸金属机器（操作系统未在虚拟化中运行）仍然需要管理。在云计算时代，这变得更加复杂。你的服务提供商不仅需要管理其操作系统的发布和修补，云客户在运行大量虚拟机和容器时也需要这样做。像 Kubernetes 这样的容器编排系统仍然需要提供包含操作系统镜像的容器镜像。

在云中，就像在物理数据中心一样，强制所有容器和虚拟机遵守操作系统合规性是非常重要的。允许任何人运行任何他们想要的操作系统是安全漏洞的入口。为了为开发人员提供一个安全的平台，你必须提供一个在所有部署中都标准化的最小化操作系统。

在一个集群中标准化操作系统的使用，不仅带来全是好处，且几乎没有缺点。当你的公司还很小的时候，标准化操作系统镜像是最容易的。对于那些在早期未能做到这一点的大公司，包括云服务提供商，他们经历了大规模的项目来在后期实现操作系统镜像的标准化。

在本节中，我们将讨论如何使用 Packer，这是由 HashiCorp 编写的一个用 Go 语言开发的软件包，用于管理虚拟机和容器镜像的创建和修补。HashiCorp 是推动 **基础设施即代码** (**IaC**) 趋势的行业领导者。

Packer 让我们使用 YAML 和 Go 提供一种一致的方式，在多个平台上构建镜像。无论是虚拟机镜像、Docker 镜像，还是裸金属镜像，Packer 都可以为你的工作负载提供一致的运行环境。

当我们编写 Packer 配置文件并使用 Packer 二进制文件时，你将开始看到 Packer 的编写方式。Packer 定义的许多交互都是使用我们之前讨论过的 `os`/`exec` 等库编写的。也许你将编写下一个在 DevOps 社区中广泛应用的 Packer！

本章将涵盖以下主题：

+   构建亚马逊机器镜像

+   使用 Goss 验证镜像

+   使用插件自定义 Packer

# 技术要求

本章的先决条件如下：

+   一个 AWS 账户

+   在 AMD64 平台上运行的 AWS Linux 虚拟机

+   一个具有管理员访问权限和访问其秘密的 AWS 用户账户

+   在 AWS Linux 虚拟机上安装 Packer

+   在 AWS Linux 虚拟机上安装 Goss

+   访问本书的 GitHub 仓库

完成本章练习需要一个 AWS 账户。这将使用 AWS 上的计算时间和存储资源，这会产生费用，尽管你可能能够使用 AWS 免费套餐账户（[`aws.amazon.com/free/`](https://aws.amazon.com/free/)）。目前写本书时，作者们与 Amazon 没有任何关系。我们没有任何经济利益。如果有的话，开发本章内容所需的 AWS 费用由我们自己承担。

在运行 Packer 时，我们建议在 Linux 上运行，无论是云镜像还是 Docker 镜像。Windows 是云计算的特殊领域，Microsoft 为处理 Windows 镜像提供了自己的工具集。我们不建议在 Mac 上运行这些工具，因为转向 Apple Silicon 及其与多个工具之间的兼容性可能会导致较长的调试时间。虽然 macOS 是 POSIX 兼容的，但它仍然不是 Linux，而 Linux 才是这些工具的主要目标。

设置 AWS 账户、配置 Linux 虚拟机和设置用户账户超出了本书的范围。有关帮助，请参阅 AWS 文档。在本练习中，请选择 Amazon Linux 或 Ubuntu 发行版之一。

用户设置是使用 AWS IAM 工具完成的，用户名可以是你选择的任何名称。你还需要为此用户获取访问密钥和密钥对。请勿将这些信息存储在代码仓库或任何公开可访问的地方，因为它们相当于用户名/密码。用户需要执行以下操作：

+   属于一个设置了`AdministratorAccess`权限的组。

+   附加现有的策略`AmazonSSMAutomationRole`。

我们建议为本次练习使用个人账户，因为这个访问权限相当广泛。你也可以设置特定的权限集，或者使用权限不那么开放的其他方法。有关这些方法的说明可以在这里找到：[`www.packer.io/docs/builders/amazon`](https://www.packer.io/docs/builders/amazon)。

登录到你的虚拟机后，你需要安装 Packer。这将取决于你使用的 Linux 版本。

以下内容适用于 Amazon Linux：

```
sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/AmazonLinux/hashicorp.repo
sudo yum -y install packer
```

以下内容适用于 Ubuntu：

```
curl -fsSL https://apt.releases.hashicorp.com/gpg | sudo apt-key add -
sudo apt-add-repository "deb [arch=amd64] https://apt.releases.hashicorp.com $(lsb_release -cs) main"
sudo apt-get update && sudo apt-get install packer
```

对于其他 Linux 版本，请参见以下内容：

[`learn.hashicorp.com/tutorials/packer/get-started-install-cli`](https://learn.hashicorp.com/tutorials/packer/get-started-install-cli).

要测试 Packer 是否安装成功，请运行以下命令：

```
packer version
```

这应该会输出你所安装的 Packer 版本。

安装 Packer 后，执行以下命令：

```
mkdir packer
cd packer
touch amazon.pkr.hcl
mkdir files
cd files
ssh-keygen -t rsa -N "" -C "agent.pem" -f agent
mv agent ~/.ssh/agent.pem
wget https://raw.githubusercontent.com/PacktPublishing/Go-for-DevOps/rev0/chapter/8/agent/bin/linux_amd64/agent
wget https://raw.githubusercontent.com/PacktPublishing/Go-for-DevOps/rev0/chapter/12/agent.service
cd ..
```

这些命令执行了以下操作：

+   在你的用户主目录中设置一个名为`packer`的目录

+   创建一个`amazon.pkr.hcl`文件，用于存储我们的 Packer 配置

+   创建一个`packer/files`目录

+   为用户`agent`生成一个 SSH 密钥对，我们将把它添加到镜像中

+   将`agent.pem`私钥移动到我们的`.ssh`目录中

+   从 Git 仓库复制我们的系统代理

+   从 Git 仓库复制一个`systemd`服务配置文件，用于系统代理

既然我们已经处理完了先决条件，接下来看看如何为 AWS 构建自定义**Ubuntu**镜像。

本章的代码文件可以从[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/12`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/12)下载。

# 构建亚马逊机器镜像

Packer 支持多种插件，程序使用这些插件来定位特定的镜像格式。对于我们的示例，我们将针对**亚马逊机器镜像**（**AMI**）格式。

还有其他的构建目标，适用于 Docker、Azure、Google Cloud 等。你可以在这里找到其他构建目标的列表：[`www.packer.io/docs/builders/`](https://www.packer.io/docs/builders/)。

对于用于云环境中的镜像，Packer 插件通常会获取一个现有的云提供商镜像，允许你重新打包并将该镜像上传到服务中。

如果你需要为多个云提供商构建多个镜像，Packer 可以同时进行多个构建。

对于 Amazon，目前有四种方法来构建 AMI：

+   亚马逊**弹性块存储**（**EBS**）启动一个源 AMI，进行配置，然后重新打包它。

+   亚马逊实例虚拟服务器，它启动一个实例虚拟机，重新打包它，然后将其上传到 S3（一个亚马逊对象存储服务）。

另外两种方法适用于高级用例。由于这是介绍如何使用 AWS 的 Packer，因此我们将避免使用这些方法。不过，你可以在这里阅读所有这些方法：[`www.packer.io/docs/builders/amazon`](https://www.packer.io/docs/builders/amazon)。

Packer 使用两种配置文件格式：

+   **JavaScript 对象表示法**（**JSON**）

+   **HashiCorp 配置语言 2**（**HCL 2**）

由于 JSON 已被弃用，我们将使用`HCL2`。此格式由 HashiCorp 创建，你可以在这里找到他们的 Go 解析器：[`github.com/hashicorp/hcl2`](https://github.com/hashicorp/hcl2)。如果你希望围绕 Packer 编写自己的工具，或者想在`HCL2`中支持自己的配置，解析器将非常有用。

现在，让我们创建一个 Packer 配置文件，用来访问亚马逊插件。

打开我们创建的`packer/`目录中的`amazon.pkr.hcl`文件。

添加以下内容：

```
packer {
  required_plugins {
    amazon = {
      version = ">= 0.0.1"
      source = "github.com/hashicorp/amazon"
    }
  }
}
```

这告诉 Packer 以下内容：

+   我们需要`amazon`插件。

+   我们需要的插件版本，即必须比版本`0.0.1`更新的最新插件。

+   `source`位置，用于获取插件。

由于我们使用的是云提供商，因此需要设置 AWS 源信息。

## 设置 AWS 源

我们将使用**Amazon EBS**构建方法，因为这是在 AWS 上部署的最简单方法。将以下内容添加到文件中：

```
source "amazon-ebs" "ubuntu" {
  access_key = "your key"
  secret_key = "your secret"
  ami_name      = "ubuntu-amd64"
  instance_type = "t2.micro"
  region        = "us-east-2"
  source_ami_filter {
    filters = {
      name                = "ubuntu/images/*ubuntu-xenial-16.04-amd64-server-*"
      root-device-type    = "ebs"
      virtualization-type = "hvm"
    }
    most_recent = true
    owners      = ["099720109477"]
  }
  ssh_username = "ubuntu"
}
```

这里有一些关键信息，我们将逐步进行：

```
source "amazon-ebs" "ubuntu" {
```

这设置了我们基础镜像的源。由于我们使用的是 `amazon` 插件，因此源将包含与该插件相关的字段。你可以在这里找到完整的字段列表：[`www.packer.io/docs/builders/amazon/ebs`](https://www.packer.io/docs/builders/amazon/ebs)。

这一行将我们的源命名为包含两部分，`amazon-ebs` 和 `ubuntu`。当我们在 `build` 块中引用此源时，它将被称为 `source.amazon-ebs.ubuntu`。

现在，我们有几个字段值：

+   `access_key` 是要使用的 IAM 用户密钥。

+   `secret_key` 是要使用的 IAM 用户的密钥。

+   `ami_name` 是 AWS 控制台中生成的 AMI 名称。

+   `instance_type` 是用于构建 AMI 的 AWS 实例类型。

+   `region` 是构建实例所在的 AWS 区域。

+   `source_ami_filter` 用于过滤 AMI 镜像，以找到要应用的镜像。

+   `filters` 包含了一种过滤我们基础 AMI 镜像的方法。

+   `name` 给出了 AMI 镜像的名称。它可以是该 API 返回的任何匹配名称：[`docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeImages.html`](https://docs.aws.amazon.com/AWSEC2/latest/APIReference/API_DescribeImages.html)。

+   `root-device-type` 指定我们使用 `ebs` 作为我们的源。

+   `virtualization-type` 指示使用哪种 AMI 虚拟化技术，`hvm` 或 `pv`。由于 `hvm` 的增强，现在推荐使用它。

+   `most_recent` 表示使用找到的最新镜像。

+   `owners` 必须列出我们使用的基础镜像 AMI 所有者的 ID。`"099720109477"` 是对 Canonical（Ubuntu 的制造商）的引用。

+   `ssh_username` 是用来通过 SSH 登录镜像的用户名。`ubuntu` 是默认用户名。

作为此处身份验证方法的替代方案，你可以使用 IAM 角色、共享凭证或其他方法。然而，其他方法过于复杂，本书无法涵盖。如果你希望使用其他方法，请参见 *技术要求* 部分中的链接。

`secret_key` 需要像密码一样安全。在生产环境中，你将希望使用 IAM 角色来避免使用 `secret_key`，或者从安全密码服务（如 AWS Secrets Manager、Azure Key Vault 或 GCP Secret Manager）中获取密钥，并使用环境变量方法让 Packer 使用该密钥。

接下来，我们需要定义一个 `build` 块，以允许我们将镜像从基础镜像更改为我们定制的镜像。

## 定义一个 build 块并添加一些 provisioners

Packer 定义了一个 `build` 块，引用我们在前一节中定义的源，并对该镜像进行我们想要的更改。

为此，Packer 在 `build` 中使用 `provisioner` 配置。Provisioners 让你通过使用 shell、Ansible、Chef、Puppet、文件或其他方法对镜像进行更改。

你可以在这里找到完整的 provisioners 列表：

[`www.packer.io/docs/provisioners`](https://www.packer.io/docs/provisioners)。

对于长期维护你的运行基础设施，Chef 或 Puppet 已经是许多安装的选择。这样可以在不等待实例重启的情况下更新整个群集到最新的镜像。

通过与 Packer 集成，你可以确保在构建过程中应用最新的补丁到你的镜像上。

尽管这确实有所帮助，但我们无法在本章中探索这些内容。设置 Chef 或 Puppet 只是超出我们目前能做的范围。但是对于长期维护，值得探索这些配置器。

作为我们的示例，我们将执行以下操作：

+   安装 Go 1.17.5 环境。

+   添加一个用户，`agent`，到系统中。

+   复制 SSH 密钥到系统中对应的用户。

+   从前面的章节中添加我们的系统代理。

+   设置 systemd 以 `agent` 用户运行代理。

让我们从使用 `shell` 配置器开始，使用 `wget` 安装 Go 的 1.17.5 版本。

让我们添加以下内容：

```
build {
  name    = "goBook"
  sources = [
    "source.amazon-ebs.ubuntu"
  ]
  provisioner "shell" {
    inline = [
      "cd ~",
      "mkdir tmp",
      "cd tmp",
      "wget https://golang.org/dl/go1.17.5.linux-amd64.tar.gz",
      "sudo tar -C /usr/local -xzf go1.17.5.linux-amd64.tar.gz",
      "echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.profile",
      ". ~/.profile",
      "go version",
      "cd ~/",
      "rm -rf tmp/*",
      "rmdir tmp",
    ]
  }
}
```

我们的 `build` 块包含以下内容：

+   `name`，给这个块命名。

+   `sources`，这是一个包含的源块列表。这包括我们刚刚定义的源。

+   `provisioner "shell"` 表明我们将使用 `shell` 配置器，通过 shell 登录执行工作。你可以有多个这种类型或其他类型的配置器块。

+   `inline` 设置要在一个 Shell 脚本中依次运行的命令集合。这组 Shell 命令下载 Go 版本 1.17.5，安装它，测试它，并移除安装文件。

应注意，你也可以使用 `file provisioner`（稍后我们将展示），从本地复制文件而不是使用 `wget` 检索它。但我们想展示如何仅使用标准的 Linux 工具从可信库中拉取。

接下来，我们将在 `build` 中添加另一个配置 *内部* 的提供程序，用于向系统添加一个用户：

```
// Setup user "agent" with SSH key file
provisioner "shell" {
  inline = [
    "sudo adduser --disabled-password --gecos '' agent",
  ]
}
provisioner "file" {
  source = "./files/agent.pub"
  destination = "/tmp/agent.pub"
}
provisioner "shell" {
  inline = [
    "sudo mkdir /home/agent/.ssh",
    "sudo mv /tmp/agent.pub /home/agent/.ssh/authorized_keys",
    "sudo chown agent:agent /home/agent/.ssh",
    "sudo chown agent:agent /home/agent/.ssh/authorized_keys",
    "sudo chmod 400 .ssh/authorized_keys",
  ]
}
```

前面的代码块结构如下：

+   第一个 `shell` 块：添加一个名为 `agent` 的用户，禁用密码。

+   第二个 `file` 块：复制一个本地文件 `./files/agent.pub` 到 `/tmp`，因为我们不能直接使用 `file provisioner` 将其复制到 `ubuntu` 以外的用户。

+   第三个 shell 块：

    +   创建我们新用户的 `.ssh` 目录。

    +   将 `agent.pub` 文件从 `/tmp` 移到 `.ssh/authorized_keys`。

    +   修改所有目录和文件以具备正确的所有者和权限。

现在，让我们添加配置器来安装我们的系统代理并设置 `systemd` 来管理它。以下部分使用 `shell` 配置器安装 `dbus`，它用于与 `systemd` 通信。我们设置了一个环境变量，以防止在使用 `apt-get` 安装时出现一些讨厌的 Debian 交互式问题：

```
// Setup agent binary running with systemd file.
provisioner "shell" { // This installs dbus-launch
     environment_vars = [
       "DEBIAN_FRONTEND=noninteractive",
     ]
     inline = [
       "sudo apt-get install -y dbus",
       "sudo apt-get install -y dbus-x11",
     ]
}
```

使用文件配置器将我们想要运行的代理从源文件复制到镜像的 `/tmp/agent` 位置：

```
provisioner "file" {
  source = "./files/agent"
  destination = "/tmp/agent"
}
```

接下来的部分在用户代理的主目录中创建一个名为`bin`的目录，并将我们在前一部分复制过来的代理文件移动到其中。剩下的是一些必要的权限和所有权更改：

```
provisioner "shell" {
  inline = [
    "sudo mkdir /home/agent/bin",
    "sudo chown agent:agent /home/agent/bin",
    "sudo chmod ug+rwx /home/agent/bin",
    "sudo mv /tmp/agent /home/agent/bin/agent",
    "sudo chown agent:agent /home/agent/bin/agent",
    "sudo chmod 0770 /home/agent/bin/agent",
  ]
}
```

这将把`systemd`文件从源目录复制到我们的镜像中：

```
provisioner "file" {
  source = "./files/agent.service"
  destination = "/tmp/agent.service"
}
```

最后一部分将`agent.service`文件移动到最终位置，告诉`systemd`启用`agent.service`中描述的服务，并验证它是否处于活动状态。`sleep`参数的作用是允许守护进程在检查之前启动：

```
provisioner "shell" {
  inline = [
    "sudo mv /tmp/agent.service /etc/systemd/system/agent.service",
    "sudo systemctl enable agent.service",
    "sudo systemctl daemon-reload",
    "sudo systemctl start agent.service",
    "sleep 10",
    "sudo systemctl is-enabled agent.service",
    "sudo systemctl is-active agent.service",
  ]
}
```

最后，让我们添加 Goss 工具，我们将在下一部分中使用它：

```
provisioner "shell" { 
    inline = [ 
        "cd ~", 
        "sudo curl -L https://github.com/aelsabbahy/goss/ releases/latest/download/goss-linux-amd64 -o /usr/local/bin/ goss", 
        "sudo chmod +rx /usr/local/bin/goss", 
        "goss -v", 
    ] 
} 
```

这将下载最新的 Goss 工具，设置其为可执行，并测试它是否能正常工作。

现在，让我们看看如何执行 Packer 构建来创建一个镜像。

## 执行 Packer 构建

Packer 构建有四个阶段：

+   初始化 Packer 以下载插件

+   验证构建

+   格式化 Packer 配置文件

+   构建镜像

第一步是初始化我们的插件。为此，只需输入以下内容：

```
packer init .
```

注意

如果你看到类似`Error: Unsupported block type`的消息，很可能是你把`provisioner`块放在了`build`块外面。

安装插件后，我们需要验证我们的构建：

```
packer validate .
```

这应该会显示`The configuration is valid`。如果没有，你需要编辑文件以修复错误。

现在，让我们格式化 Packer 模板文件。这是我相信 HashiCorp 借用自 Go 的`go fmt`命令的概念，工作方式也相同。让我们尝试一下：

```
packer fmt .
```

最后，是时候进行我们的构建了：

```
packer build .
```

这里会有相当多的输出。如果一切顺利，你将看到如下信息：

```
Build 'goBook.amazon-ebs.ubuntu' finished after 5 minutes 11 seconds.
==> Wait completed after 5 minutes 11 seconds
==> Builds finished. The artifacts of successful builds are:
--> goBook.amazon-ebs.ubuntu: AMIs were created:
us-east-2: ami-0f481c1107e74d987
```

注意

如果你看到关于权限的错误，这将与你的用户账户设置有关。请参见本章早些时候列出的必要权限。

现在，你已经在 AWS 上拥有了一个 AMI 镜像。你可以启动使用该镜像的 AWS 虚拟机，它们将运行我们的系统代理。随意启动一个 VM 并设置为你的新 AMI，玩玩代理。你可以通过`ssh agent@[host]`从你的 Linux 设备访问该代理，其中`[host]`是 AWS 主机的 IP 或 DNS 地址。

既然我们可以使用 Packer 来打包镜像，让我们看看如何使用 Goss 来验证镜像。

# 使用 Goss 验证图片

Goss 是一个用于检查服务器配置的工具，使用的是 YAML 编写的规格文件。通过这种方式，你可以测试服务器是否按预期工作。这可以是测试通过 SSH 使用预期密钥访问服务器，也可以是验证各种进程是否正在运行。

Goss 不仅可以测试你的服务器是否符合要求，它还可以与 Packer 集成。这样，我们就可以在提供服务的步骤和部署之前，测试服务器是否按预期运行。

让我们看看如何创建一个 Goss 规格文件。

## 创建规格文件

规格文件是一组指令，告诉 Goss 需要测试什么。

有几种方法可以为 Goss 创建规范文件。规范文件用于告诉 Goss 需要测试什么。

虽然你可以手动编写，但最有效的方法是使用 Goss 的两个命令之一：

+   `goss add`

+   `goss autoadd`

使用 Goss 的最有效方法是启动一个包含自定义 AMI 的机器，使用 `ubuntu` 用户登录，并使用 `autoadd` 生成 YAML 文件。

登录到你的 AMI 实例后，让我们运行以下命令：

```
goss -g process.yaml autoadd sshd
```

这将生成一个 `process.yaml` 文件，内容如下：

```
service:
  sshd:
    enabled: true
    running: true
user:
  sshd:
    exists: true
    uid: 110
    gid: 65534
    groups:
    - nogroup
    home: /var/run/sshd
    shell: /usr/sbin/nologin
process:
  sshd:
    running: true
```

这表示我们预期以下内容：

+   一个名为 `sshd` 的系统服务应该通过 systemd 启用并运行。

+   服务应该以用户 `sshd` 运行：

    +   用户 ID 为 `110`。

    +   组 ID 为 `65534`。

    +   不属于其他任何组。

    +   用户的主目录应该是 `/var/run/sshd`。

    +   用户应该没有登录 shell。

+   一个名为 `sshd` 的进程应该正在运行。

让我们添加我们部署的代理服务：

```
goss -g process.yaml autoadd agent
```

这将向 YAML 文件中添加类似的行。

现在，让我们验证代理位置：

```
goss -g files.yaml autoadd /home/agent/bin/agent
```

这将添加如下所示的部分：

```
file:
  /home/agent/bin/agent:
    exists: true
    mode: "0700"
    size: 14429561
    owner: agent
    group: agent
    filetype: file
    contains: []
```

这表示以下内容：

+   `/home/agent/bin/agent` 文件必须存在。

+   必须是模式 `0700`。

+   必须有 `14429561` 字节的大小。

+   必须由 `agent:agent` 拥有。

+   是文件，而不是目录或 `symlink`。

让我们添加另一个，但更具体一些，使用 `goss add`：

```
goss -g files.yaml add file /home/agent/.ssh/authorized_keys 
```

与 `autoadd` 自动猜测参数不同，我们必须明确指定它是一个文件。这将生成与 `autoadd` 相同的条目。对于这个文件，我们来验证 `authorized_keys` 文件的内容。为此，我们将使用 `SHA256` 哈希。首先，我们可以通过运行以下命令来获取哈希：

```
sha256sum /home/agent/.ssh/authorized_keys
```

这将返回文件的哈希值。在 YAML 文件中 `authorized_keys` 的 `file` 条目里，添加以下内容：

```
sha256: theFileHashJustGenerated
```

不幸的是，Goss 没有简单地添加整个目录文件或自动将 `SHA256` 添加到条目的功能。一个例子可能是验证 Go 的 1.17.5 版本的所有文件是否按预期出现在我们的镜像中。

你可能会想尝试如下操作：

```
find /usr/local/go -print0 | xargs -0 -I{} goss -g golang.yaml add file {}
```

然而，这样做速度相当慢，因为 `goss` 每次运行时都会读取 YAML 文件。你可能会想使用 `xargs -P 0` 来加速，但这样会导致其他问题。

如果你需要包含大量文件和 SHA256 哈希，你需要编写一个自定义脚本/程序来处理这些内容。幸运的是，我们有 Go，因此编写一个可以做到这一点的程序非常容易。而且因为 Goss 是用 Go 编写的，我们可以重用程序中的数据结构。你可以在这里看到一个示例工具：[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/12/goss/allfiles`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/12/goss/allfiles)。

你可以直接针对目录结构（编译后）运行它，如下所示：

```
allfiles /usr/local/go > goinstall_files.yaml
```

这将输出一个 `goinstall_files.yaml` 文件，该文件提供了一个 Goss 配置，用于检查这些文件及其 SHA256 哈希。

还记得我们安装了`dbus`吗？让我们验证一下我们的`dbus`包是否已安装：

```
goss -g dbus.yaml add package dbus
goss -g dbus.yaml add package dbus-x11
```

这现在确保我们的`dbus`和`dbus-x11`包已安装。`-g dbus.yaml`文件将此写入另一个名为`dbus.yaml`的文件，而不是默认的`goss.yaml`。

现在我们需要创建一个`goss.yaml`文件，引用我们创建的其他文件。我们本可以在不加`-g`选项的情况下运行`goss`，但这样可以使事情更有条理。让我们创建我们的根文件：

```
goss add goss process.yaml
goss add goss files.yaml
goss add goss dbus.yaml
```

这会创建一个`goss.yaml`文件，该文件引用我们所有的其他文件。

让我们用它来验证所有内容：

```
goss validate
```

这将输出类似于以下内容的文本：

```
..........................
Total Duration: 0.031s
Count: 26, Failed: 0, Skipped: 0
```

请注意，是的，它确实在不到一秒钟的时间内运行完成！

## 添加 Packer 预配置器

我们能够验证已有的内容很棒，但我们真正想要的是验证每一个镜像构建。为此，我们可以使用耶鲁大学开发的自定义 Packer 预配置器。

为了做到这一点，我们需要将 YAML 文件从镜像中提取并传送到我们的构建机器上。

从构建机器上，执行以下命令（替换`[]`中的内容）：

```
cd /home/[user]/packer/files
mkdir goss
cd goss
scp ubuntu@[ip of AMI machine]:/home/ubuntu/*.yaml ./
```

你需要将`[user]`替换为构建机器上的用户名，将`[ip of AMI machine]`替换为你启动的 AMI 机器的 IP 地址或 DNS 条目。你还可能需要在`scp`之后提供`-i [pem 文件的位置]`。

由于 Goss 预配置器没有内建，我们需要从耶鲁大学的 GitHub 仓库下载该版本并进行安装：

```
mkdir ~/tmp
cd ~/tmp
wget https://github.com/YaleUniversity/packer-provisioner-goss/releases/download/v3.1.2/packer-provisioner-goss-v3.1.2-linux-amd64.tar.gz
sudo tar -xzf packer-provisioner-goss-v3.1.2-linux-amd64.tar.gz
cp sudo packer-provisioner-goss /usr/bin/packer-provisioner-goss
rm -rf ~/tmp
```

安装完预配置器后，我们可以将配置添加到`amazon.pkr.hcl`文件中：

```
// Setup Goss for validating an image.
provisioner "file" {
  source = "./files/goss/*"
  destination = "/home/ubuntu/"
}
provisioner "goss" {
     retry_timeout = "30s"
     tests = [
      "files/goss/goss.yaml", 
      "files/goss/files.yaml", 
      "files/goss/dbus.yaml", 
      "files/goss/process.yaml", 
     ]
}
```

你可以在[`github.com/YaleUniversity/packer-provisioner-goss`](https://github.com/YaleUniversity/packer-provisioner-goss)找到更多 Goss 的`provisioner`设置。

让我们重新格式化我们的 Packer 文件：

```
packer fmt .
```

我们还不能构建`packer`镜像，因为它将与我们已上传到 AWS 的镜像同名。我们有两个选择：从 AWS 中删除我们之前构建的 AMI 镜像，或者将我们 Packer 文件中的名称更改为以下内容：

```
ami_name      = "ubuntu-amd64"
```

两种选择都可以。

现在，让我们构建我们的 AMI 镜像：

```
packer build .
```

当你这次运行它时，输出中应该会看到类似以下的内容：

```
==> goBook.amazon-ebs.ubuntu: Running goss tests...
==> goBook.amazon-ebs.ubuntu: Running GOSS render command: cd /tmp/goss &&  /tmp/goss-0.3.9-linux-amd64    render > /tmp/goss-spec.yaml
==> goBook.amazon-ebs.ubuntu: Goss render ran successfully
==> goBook.amazon-ebs.ubuntu: Running GOSS render debug command: cd /tmp/goss &&  /tmp/goss-0.3.9-linux-amd64    render -d > /tmp/debug-goss-spec.yaml
==> goBook.amazon-ebs.ubuntu: Goss render debug ran successfully
==> goBook.amazon-ebs.ubuntu: Running GOSS validate command: cd /tmp/goss &&   /tmp/goss-0.3.9-linux-amd64    validate --retry-timeout 30s --sleep 1s
    goBook.amazon-ebs.ubuntu: ..........................
    goBook.amazon-ebs.ubuntu:
    goBook.amazon-ebs.ubuntu: Total Duration: 0.029s
    goBook.amazon-ebs.ubuntu: Count: 26, Failed: 0, Skipped: 0
==> goBook.amazon-ebs.ubuntu: Goss validate ran successfully
```

这表示 Goss 测试成功运行。如果 Goss 失败，将会下载调试输出到本地目录。

你可以在这里找到最终版的 Packer 文件：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/12/packer/amazon.final.pkr.hcl`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/12/packer/amazon.final.pkr.hcl)。

你现在已经看到了如何使用 Goss 工具为你的镜像构建验证并将其集成到 Packer 中。还有更多的功能可以探索，你可以在这里阅读：[`github.com/aelsabbahy/goss`](https://github.com/aelsabbahy/goss)。

现在我们已经使用了 Goss 作为预配置器，那么如何编写我们自己的呢？

# 使用插件自定义 Packer

我们使用的内建提供程序非常强大。通过提供 shell 访问和文件上传，几乎可以在 Packer 提供程序中做任何事情。

对于大规模构建，这可能会非常繁琐。而且，如果这种情况是常见的，您可能想让自己的 Go 应用程序为您完成这项工作。

Packer 允许构建可以用于以下场景的插件：

+   一个 Packer 构建器

+   一个 Packer 提供程序

+   一个 Packer 后处理器

构建器在您需要与将使用您图像的系统进行交互时使用：Docker、AWS、GCP、Azure 或其他系统。由于这种用法在云提供商或像 VMware 这样的公司增加支持之外并不常见，因此我们将不做详细介绍。

后处理器通常用于将图像推送到上传之前生成的工件。由于这不是常见的用法，我们将不做详细介绍。

提供程序是最常见的，因为它们是构建过程中输出图像的一部分。

Packer 有两种编写这些插件的方式：

+   单一插件

+   多插件

单一插件是编写插件的旧方式。Goss 提供程序就是用旧方式编写的，这也是我们手动安装它的原因。

使用更新的方式，`packer init` 可以用来下载插件。此外，一个插件可以在一个插件中注册多个构建器、提供程序或后处理器。这是编写插件的推荐方式。

不幸的是，截至撰写本文时，关于多插件和支持 `packer init` 的发布的官方文档不完整。按照这些说明操作，无法生成可以通过他们建议的过程发布的插件。

这里包含的说明将填补空白，允许构建一个多插件，用户可以通过 `packer init` 安装它。

现在让我们来看看如何编写自定义插件。

## 编写您自己的插件

提供程序是 Packer 应用程序的强大扩展。它们允许我们自定义应用程序，做任何我们需要的事情。

我们已经看到提供程序如何执行 Goss 来验证我们的构建。这使我们能够确保未来的构建遵循图像的规范。

要编写一个自定义 `provisioner`，我们必须实现以下接口：

```
type Provisioner interface { 
    ConfigSpec() hcldec.ObjectSpec 
    Prepare(...interface{}) error 
    Provision(context.Context, Ui, Communicator, 
        map[string] interface{}) error 
}
```

上面的代码描述如下：

+   `ConfigSpec()` 返回一个表示您提供程序 HCL2 规范的对象。Packer 将使用它将用户的配置转换为 Go 语言中的结构化对象。

+   `Prepare()` 准备您的插件运行，并接收一个 `interface{}` 切片，表示配置。通常，配置作为单个 `map[string]interface{}` 传递。`Prepare()` 应该执行诸如从源拉取信息或验证配置等准备工作，应该在尝试运行之前就导致失败。这不应有副作用，也就是说，它不应通过创建文件、实例化虚拟机或对系统进行任何其他更改来改变任何状态。

+   `Provision()`执行大部分工作。它接收一个`Ui`对象，用于与用户进行通信，还有一个`Communicator`对象，用于与正在运行的机器进行通信。提供了一个`map`，其中包含由构建器设置的值。然而，依赖于其中的值可能会将你绑定到一个`builder`类型。

对于我们的示例提供程序，我们将打包 Go 环境并将其安装到机器上。虽然 Linux 发行版通常会打包 Go 环境，但它们通常会落后几个版本。之前，我们可以通过使用`file`和`shell`（这些实际上几乎可以做任何事情）来完成，但如果你是应用程序提供商，想要为其他 Packer 用户在多个平台上实现可重复的操作，那么自定义提供程序是最佳选择。

### 添加我们的提供程序配置

为了让用户配置我们的插件，我们需要定义一个配置。我们希望支持的配置选项如下：`Version (string)[optional]`，下载的特定版本默认为`latest`。

我们将在子包中定义这个：`internal/config/config.go`。

在该文件中，我们将添加以下内容：

```
package config
//go:generate packer-sdc mapstructure-to-hcl2 -type Provisioner
// Provisioner is our provisioner configuration.
type Provisioner struct {
	Version string
}
// Default inputs default values.
func (p *Provisioner) Defaults() {
	if p.Version == "" {
		p.Version = "latest"
	}
}
```

不幸的是，我们现在需要能够从`hcldec.ObjectSpec`文件中读取这些内容。这比较复杂，因此 HashiCorp 创建了一个代码生成器来为我们完成这项工作。要使用它，你必须安装他们的`packer-sdc`工具：

```
go install github.com/hashicorp/packer-plugin-sdk/cmd/packer-sdc@latest
```

为了生成文件，我们可以在`internal/config`目录中执行以下操作：

```
go generate ./
```

这将输出一个`config.hcl2spec.go`文件，其中包含我们需要的代码。它使用文件中定义的`//go:generate`行。

### 定义插件的配置规范

在插件的位置根目录，我们创建一个名为`goenv.go`的文件。

所以，我们首先定义用户将输入的配置：

```
package main 
import ( 
    ... 
    "[repo location]/packer/goenv/internal/config" 
    "github.com/hashicorp/packer-plugin-sdk/packer" 
    "github.com/hashicorp/packer-plugin-sdk/plugin" 
    "github.com/hashicorp/packer-plugin-sdk/version" 
    packerConfig "github.com/hashicorp/packer-plugin-sdk/ template/config" 
    ... 
)
```

这将导入以下内容：

+   我们刚才定义的`config`包

+   构建我们插件所需的三个包：

    +   `packer`

    +   `plugin`

    +   `version`

+   一个用于处理 HCL2 配置的`packerConfig`包

    注意

    `...`表示标准库包和一些其他包，为了简洁起见省略了它们。你可以在仓库版本中看到它们的全部内容。

现在，我们需要定义我们的提供程序：

```
// Provisioner implements packer.Provisioner. 
type Provisioner struct{
     packer.Provisioner // Embed the interface.
     conf *config.Provisioner
     content []byte
     fileName string
}
```

这将保存我们的配置、一部分文件内容以及 Go tarball 文件名。我们将在这个结构体上实现我们的`Provisioner`接口。

现在，是时候添加所需的方法了。

### 定义`ConfigSpec()`函数

`ConfigSpec()`是为 Packer 的内部使用而定义的。我们只需要提供规格，以便 Packer 可以读取配置。

让我们使用之前生成的`config.hcl2spec.go`来实现`ConfigSpec()`：

```
func (p *Provisioner) ConfigSpec() hcldec.ObjectSpec {
     return new(config.FlatProvisioner).HCL2Spec()
}
```

这将返回`ObjectSpec`，用于处理读取我们的 HCL2 配置。

现在这些工作已经完成，我们需要准备好插件以供使用。

### 定义`Prepare()`方法

请记住，`Prepare()`方法仅需要解释 HCL2 配置的中间表示并验证条目。它不应该改变任何事物的状态。

以下是该示例的样子：

```
func (p *Provisioner) Prepare(raws ...interface{}) error { 
    c := config.Provisioner{} 
    if err := packerConfig.Decode(&c, nil, raws...); err != nil {
            return err
    }
    c.Defaults()
    p.conf = &c
    return nil
}
```

这段代码执行了以下操作：

+   创建我们的空配置。

+   将原始配置项解码为我们的内部表示形式。

+   如果没有设置值，默认值会被放入配置中。

+   验证我们的配置。

我们还可以利用这段时间连接服务或进行任何其他所需的准备工作。最重要的是不要改变任何状态。

经过所有准备工作后，是时候迎接大结局了。

### 定义 `Provision()` 方法。

`Provision()` 是所有魔法发生的地方。让我们将其分成一些逻辑部分：

+   获取我们的版本信息。

+   将一个 tarball 推送到镜像中。

+   解压 tarball 文件。

+   测试我们的 Go 工具安装情况。

以下代码封装了其他方法，以相同的顺序执行逻辑部分：

```
func (p *Provisioner) Provision(ctx context.Context, u packer. Ui, c packer.Communicator, m map[string]interface{}) error { 
    u.Message("Begin Go environment install") 
    if err := p.fetch(ctx, u, c); err != nil { 
            u.Error(fmt.Sprintf("Error: %s", err))
            return err
    }
    if err := p.push(ctx, u, c); err != nil {
            u.Error(fmt.Sprintf("Error: %s", err))
            return err
    }
    if err := p.unpack(ctx, u, c); err != nil {
            u.Error(fmt.Sprintf("Error: %s", err))
            return err
    }
    if err := p.test(ctx, u, c); err != nil {
            u.Error(fmt.Sprintf("Error: %s", err))
            return err
    }
    u.Message("Go environment install finished")
    return nil
}
```

这段代码调用了所有阶段（我们稍后会定义）并将一些消息输出到用户界面。`Ui` 接口定义如下：

```
type Ui interface {
     Ask(string) (string, error)
     Say(string)
     Message(string)
     Error(string)
     Machine(string, ...string)
     getter.ProgressTracker
}
```

不幸的是，UI 在代码或文档中没有很好的记录。以下是详细说明：

+   你可以使用 `Ask()` 向用户提问并获得回应。一般来说，应该避免使用这个方法，因为它会破坏自动化流程。最好让用户将其放入配置中。

+   `Say()` 和 `Message()` 都是将字符串打印到屏幕上。

+   `Error()` 输出一条错误信息。

+   `Machine()` 只是通过 `fmt.Printf()` 将一条语句输出到机器生成的日志中，并以 `machine readable:` 为前缀。

+   `getter.ProgressTracker()` 被 `Communicator` 用来跟踪下载进度，你不需要担心它。

现在我们已经涵盖了 UI，接下来讲解 `Communicator`：

```
type Communicator interface {
  Start(context.Context, *RemoteCmd) error
  Upload(string, io.Reader, *os.FileInfo) error
  UploadDir(dst string, src string, exclude []string) error
  Download(string, io.Writer) error
  DownloadDir(src string, dst string, exclude []string) error
}
```

前面代码块中的方法如下所示：

+   `Start()` 在镜像上运行一个命令。你传递 `*RemoteCmd`，它类似于我们在前面章节中使用的 `os/exec` 中的 `Cmd` 类型。

+   `Upload()` 将文件上传到机器镜像。

+   `UploadDir()` 递归地将本地目录上传到机器镜像。

+   `Download()` 从机器镜像中下载文件。这允许你捕获调试日志，例如。

+   `DownloadDir()` 从机器递归地下载一个目录到本地目的地。你可以排除某些文件。

你可以在这里查看完整的接口注释：[`pkg.go.dev/github.com/hashicorp/packer-plugin-sdk/packer?utm_source=godoc#Communicator`](https://pkg.go.dev/github.com/hashicorp/packer-plugin-sdk/packer?utm_source=godoc#Communicator)。

让我们来看一下构建第一个助手 `p.fetch()`。以下代码决定了下载 Go 工具使用的 URL。我们的工具面向 Linux，但我们支持为多个平台安装不同版本。我们使用 Go 的 runtime 包来确定我们当前运行的架构（386、ARM 或 AMD 64），以此决定下载哪个包。用户可以指定一个特定版本或 `latest`。对于 `latest`，我们查询 Google 提供的 URL，该 URL 返回 Go 的最新版本。然后，我们利用这个版本信息构造下载 URL：

```
func (p *Provisioner) fetch(ctx context.Context, u Ui, 
c Communicator) error {
     const (
          goURL = `https://golang.org/dl/go%s.linux-%s.tar.gz`
          name  = `go%s.linux-%s.tar.gz`
    )
    platform := runtime.GOARCH
    if p.conf.Version == "latest" {
          u.Message("Determining latest Go version")
          resp, err := http.Get("https://golang.org/VERSION?m=text")
          if err != nil {
                  u.Error("http get problem: " + err.Error())
                  return fmt.Errorf("problem asking Google for latest Go version: %s", err)
          }
          ver, err := io.ReadAll(resp.Body)
          if err != nil {
                  u.Error("io read problem: " + err.Error())
                  return fmt.Errorf("problem reading latest Go version: %s", err)
          }
          p.conf.Version = strings.TrimPrefix(string(ver), "go")
          u.Message("Latest Go version: " + p.conf.Version)
    } else {
          u.Message("Go version to use is: " + p.conf.Version)
    }
```

这段代码发起 Go tarball 的 HTTP 请求，并将其存储在 `.content` 中：

```
    url := fmt.Sprintf(goURL, p.conf.Version, platform)
    u.Message("Downloading Go version: " + url)
    resp, err := http.Get(url)
    if err != nil {
        return fmt.Errorf("problem reaching golang.org for version(%s): %s)", p.conf.Version, err)
    }
    defer resp.Body.Close()
    p.content, err = io.ReadAll(resp.Body)
    if err != nil {
        return fmt.Errorf("problem downloading file: %s", err)
    }
    p.fileName = fmt.Sprintf(name, p.conf.Version, platform)
    u.Message("Downloading complete")
    return nil
}
```

现在我们已经获取了 Go `tarball`内容，让我们将其推送到机器上：

```
func (p *Provisioner) push(ctx context.Context, u Ui, 
c Communicator) error {
     u.Message("Pushing Go tarball")
     fs := simple.New()
     fs.WriteFile("/tarball", p.content, 0700)
     fi, _ := fs.Stat("/tarball")
     err := c.Upload(
             "/tmp/"+p.fileName,
             bytes.NewReader(p.content),
             &fi,
     )
     if err != nil {
             return err
     }
     u.Message("Go tarball delivered to: /tmp/" + p.fileName)
     return nil
}
```

上述代码将我们的内容上传到镜像中。`Upload()`要求我们提供`*os.FileInfo`，但我们没有一个，因为我们的文件在磁盘上并不存在。所以，我们使用一个技巧，将内容写入内存中的文件系统中，然后获取`*os.FileInfo`。这样，我们就避免了将不必要的文件写入磁盘。

注意

`Communicator.Upload()`的一个奇怪之处在于它接受一个指向`interface (*os.FileInfo)`的指针。这几乎总是作者的一个错误。不要在你的代码中这样做。

接下来需要做的是在镜像中解压此内容：

```
func (p *Provisioner) unpack(ctx context.Context, u Ui, 
c Communicator) error {
     const cmd = `sudo tar -C /usr/local -xzf /tmp/%s`
     u.Message("Unpacking Go tarball to /usr/local")
     b := bytes.Buffer{}
     rc := &packer.RemoteCmd{
          Command: fmt.Sprintf(cmd, p.fileName),
          Stdout: &b,
          Stderr: &b,
     }
     if err := c.Start(rc); err != nil {
          return fmt.Errorf("problem unpacking tarball(%s):\n%s", err, b.String())
     }
     u.Message("Unpacked Go tarball")
     return nil
}
```

这段代码执行以下操作：

+   定义一个命令来解压我们的 tarball 并安装到`/usr/local`。

+   将该命令包装在`*packerRemoteCmd`中并捕获`STDOUT`和`STDERR`。

+   使用`Communicator`运行命令：如果失败，返回错误和`STDOUT`/`STDERR`用于调试。

`Provisioner`的最后一步是测试它是否已安装：

```
func (p *Provisioner) test(ctx context.Context, u Ui, 
c Communicator) error {
     u.Message("Testing Go install")
     b := bytes.Buffer{}
     rc := &packer.RemoteCmd{
          Command: `/usr/local/go/bin/go version`,
          Stdout: &b,
          Stderr: &b,
     }
     if err := c.Start(rc); err != nil {
          return fmt.Errorf("problem testing Go install(%s):\n%s", err, b.String())
     }
     u.Message("Go installed successfully")
     return nil
}
```

这段代码执行以下操作：

+   运行`/usr/local/go/bin/go version`来获取输出。

+   如果失败，返回错误和`STDOUT`/`STDERR`用于调试。

现在，插件的最后部分是编写`main()`：

```
const (
        ver     = "0.0.1"
        release = "dev"
)
var pv *version.PluginVersion
func init() {
     pv = version.InitializePluginVersion(ver, release)
}
func main() { 
    set := plugin.NewSet() 
    set.SetVersion(pv) 
    set.RegisterProvisioner("goenv", &Provisioner{}) 
    err := set.Run() 
    if err != nil { 
        fmt.Fprintln(os.Stderr, err.Error()) 
        os.Exit(1) 
    } 
} 
```

这段代码执行以下操作：

+   将我们的发布版本定义为`"0.0.1"`。

+   将发布定义为`"dev"`版本，但这里可以使用任何名称。生产版本应使用`""`。

+   初始化`pv`，它保存插件版本信息。这样做是在`init()`中，因为包注释指出应该这样做，而不是在`main()`中，这样如果存在问题，可以在最早时触发 panic。

+   创建一个新的 Packer `plugin.Set`：

    +   设置版本信息。如果未设置，所有 GitHub 发布将失败。

    +   使用`"goenv"`插件名称注册我们的配置器：

        +   可用于注册其他配置器。

        +   可用于注册构建器，`set.RegisterBuilder()`，以及后处理器，`set.RegisterPostProcessor()`。

+   运行我们创建的`Set`并在任何错误时退出。

我们可以使用常规名称进行注册，这样名称会附加到插件名称上。如果使用`plugin.DEFAULT_NAME`，我们的配置器可以简单地通过插件名称来引用。

因此，如果我们的插件命名为`packer-plugin-goenv`，我们可以将插件称为`goenv`。如果使用`plugin.DEFAULT_NAME`以外的名称，例如`example`，则我们的插件将被称为`goenv-example`。

我们现在有了一个插件，但要使其有用，我们必须允许人们初始化它。让我们来看一下如何通过 GitHub 发布我们的插件。

测试插件

在这个练习中，我们不讨论测试 Packer 插件。发布时，尚无相关的测试文档。然而，Packer 的 GoDoc 页面有公开的类型，可以模拟 Packer 中的各种类型，帮助测试你的插件。

这包括模拟`Provisioner`、`Ui`和`Communicator`类型，以便进行测试。你可以在这里找到这些内容：[`pkg.go.dev/github.com/hashicorp/packer-plugin-sdk/packer`](https://pkg.go.dev/github.com/hashicorp/packer-plugin-sdk/packer)。

## 发布插件

Packer 对允许`packer`二进制文件查找和使用插件有严格的发布要求。为了使插件可下载，必须满足以下要求：

+   必须在 GitHub 上发布；不允许使用其他来源。

+   你的仓库名称必须是`packer-plugin-*`，其中`*`是你的插件名称。

+   只能使用连字符，而不是下划线。

+   必须有一个插件发布，其中包括我们将描述的某些资产。

官方发布文档可以在这里找到：[`www.packer.io/docs/plugins/creation#creating-a-github-release`](https://www.packer.io/docs/plugins/creation#creating-a-github-release)。

HashiCorp 还有一个 30 分钟的视频，展示如何将发布文档发布到 Packer 网站，视频链接如下：[`www.hashicorp.com/resources/publishing-packer-plugins-to-the-masses`](https://www.hashicorp.com/resources/publishing-packer-plugins-to-the-masses)。

生成发布的第一步是创建一个**GNU 隐私保护**（**GPG**）密钥以签署发布版本。GitHub 的相关指令可以在这里找到（但请先阅读下面的注意事项）：[`docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key`](https://docs.github.com/en/authentication/managing-commit-signature-verification/generating-a-new-gpg-key)。

在遵循该文档之前，请记住在执行指令时注意以下事项：

+   确保将公钥添加到你的 GitHub 个人资料中。

+   请不要在密码短语中使用`$`或任何其他符号，因为这会导致问题。

一旦完成，你需要将私钥添加到你的仓库中，这样我们定义的 GitHub Actions 才能签署发布版本。你需要进入 GitHub 仓库的`|` **Secrets**。点击提供的**New Repository Secret**按钮。

选择名称`GPG_PRIVATE_KEY`。

在值部分，你需要粘贴你的 GPG 私钥，你可以使用以下命令导出该私钥：

```
gpg --armor --export-secret-keys [key ID or email]
```

`[key ID 或 email]`是你为密钥提供的身份，通常是你的电子邮件地址。

现在，我们需要添加你的 GPG 密钥的密码短语。你可以将其作为一个名为`GPG_PASSPHRASE`的密钥添加。值应该是 GPG 密钥的密码短语。

一旦完成，你需要下载 HashiCorp 提供的 GoReleaser 脚手架。你可以通过以下方式完成：

```
curl -L -o ".goreleaser.yml" \
https://raw.githubusercontent.com/hashicorp/packer-plugin-scaffolding/main/.goreleaser.yml
```

现在，我们需要在你的仓库中设置 HashiCorp 提供的 GitHub Actions 工作流。你可以通过以下方式完成：

```
mkdir -p .github/workflows &&
 curl -L -o ".github/workflows/release.yml" \
 https://raw.githubusercontent.com/hashicorp/packer-plugin-scaffolding/main/.github/workflows/release.yml
```

最后，我们需要下载`GNUmakefile`，这是脚手架使用的文件。我们来下载它：

```
curl -L -o "GNUmakefile" \
https://raw.githubusercontent.com/hashicorp/packer-plugin-scaffolding/main/GNUmakefile
```

我们的插件仅适用于 Linux 系统。`.goreleaser.yml`文件定义了多个平台的发布版本。你可以通过修改`.goreleaser.yml`中的`builds`部分来限制它。你可以在这里查看一个示例：[`github.com/johnsiilver/packer-plugin-goenv/blob/main/.goreleaser.yml`](https://github.com/johnsiilver/packer-plugin-goenv/blob/main/.goreleaser.yml)。

当你的代码可以构建并且这些文件已包含时，你需要将这些文件提交到你的仓库中。

下一步将是创建一个发布版本。这个版本需要使用语义化版本标记，类似于你在插件的`main`文件中设置的`ver`变量。稍有不同的是，虽然`ver string`中将严格使用数字和点，但在 GitHub 上标记时会加上`v`。例如，`ver = "0.0.1"`将成为 GitHub 发布版本`v0.0.1`。GitHub 发布的文档可以在此查看：[`docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository`](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository)。

一旦你创建了发布版本，你可以通过查看**Actions**标签来查看正在执行的操作。这将展示结果并详细说明操作过程中遇到的任何问题。

## 在构建中使用我们的插件

要在构建中使用我们的插件，我们需要修改 HCL2 配置。首先，我们需要修改`packer.required_plugins`以要求我们的插件：

```
packer {
  required_plugins {
    amazon = {
      version = ">= 0.0.1"
      source = "github.com/hashicorp/amazon"
    }
    installGo = {
      version = ">= 0.0.1"
      source = "github.com/johnsiilver/goenv"
    }
  }
}
```

这做了几件事：

+   创建一个新的变量`installGo`，该变量提供访问我们多插件中定义的所有插件的权限。这里只有一个插件：`goenv`。

+   设置使用的版本必须大于或等于`0.0.1`。

+   提供插件的源路径。你会注意到路径中没有`packer-plugin-`，因为这是每个插件的标准命名，它们移除了这个部分的输入。

    注意

    你会发现源地址与我们代码的位置不同。这是因为我们希望将代码保留在常规位置，但 Packer 要求插件必须有自己的仓库。代码在这两个位置都有。你可以在这里查看代码副本：[`github.com/johnsiilver/packer-plugin-goenv`](https://github.com/johnsiilver/packer-plugin-goenv)。

现在，我们需要删除`build.provisioner`下安装 Go 的`shell`部分，并用以下内容替换：

```
provisioner "goenv-goenv" {
  version = "1.17.5"
}
```

最后，你需要更新 AMI 名称，以便将其存储到新的位置。

作为替代方案，你也可以在此下载修改后的 HCL2 文件：[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/12/packer/amazon.goenv.pkr.hcl`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/12/packer/amazon.goenv.pkr.hcl)。

在终端中，格式化文件并使用以下命令下载我们的插件：

```
packer fmt .
packer init .
```

这应该会导致我们的插件下载，并输出类似于以下内容的文本：

```
Installed plugin github.com/johnsiilver/goenv v0.0.1 in "/home/ec2-user/.config/packer/plugins/github.com/johnsiilver/goenv/packer-plugin-goenv_v0.0.1_x5.0_linux_amd64"
```

我们最终可以通过以下命令构建我们的镜像：

```
packer build .
```

如果成功，您应该在 Packer 输出中看到以下内容：

```
goBook.amazon-ebs.ubuntu: Begin Go environment install
goBook.amazon-ebs.ubuntu: Go version to use is: 1.17.5
goBook.amazon-ebs.ubuntu: Downloading Go version: https://golang.org/dl/go1.17.5.linux-amd64.tar.gz
goBook.amazon-ebs.ubuntu: Downloading complete
goBook.amazon-ebs.ubuntu: Pushing Go tarball
goBook.amazon-ebs.ubuntu: Go tarball delivered to: /tmp/go1.17.5.linux-amd64.tar.gz
goBook.amazon-ebs.ubuntu: Unpacking Go tarball to /usr/local
goBook.amazon-ebs.ubuntu: Unpacked Go tarball
goBook.amazon-ebs.ubuntu: Testing Go install
goBook.amazon-ebs.ubuntu: Go installed successfully
goBook.amazon-ebs.ubuntu: Go environment install finished
```

这个插件已经过预先测试。让我们来看看如果插件失败，您可以做些什么。

## 调试 Packer 插件

当`packer build .`失败时，您可能会在 UI 输出中获得或没有获得相关信息。这取决于问题是恐慌（panic）还是错误（error）。

恐慌会返回一个`Unexpected EOF`消息，因为插件崩溃，而 Packer 应用程序只知道它没有在 Unix 套接字上接收到 RPC 消息。

我们可以通过运行时提供这个选项来请求 Packer 帮助我们：

```
packer build -debug
```

如果构建崩溃，它将输出一个`crash.log`文件。它还会在每一步之间使用`press enter`，并且一次只允许运行一个`packer`构建。

您可能会看到其他文件出现，因为一些插件（如 Goss）检测到`debug`选项并输出调试配置文件和日志。

您可能还想启用日志记录，以便记录您或其他插件写入的日志消息。这可以通过设置几个环境变量来完成：

```
PACKER_LOG=1 PACKER_LOG_PATH="./packerlog.txt" packer build .
```

这解决了大多数调试需求。然而，有时所需的调试信息是系统日志的一部分，而不是插件本身。在这种情况下，您可能希望在检测到错误时使用通信器的`Download()`或`DownloadDir()`方法来检索文件。

获取更多调试信息，请访问官方调试文档：[`www.packer.io/docs/debugging`](https://www.packer.io/docs/debugging)。

在本节中，我们详细介绍了如何构建一个 Packer 多插件，展示了如何在 GitHub 上设置插件以与`packer init`一起使用，并更新了我们的 Packer 配置以使用该插件。此外，我们还讨论了调试 Packer 插件的基础知识。

# 摘要

本章教会了您使用 Packer 构建机器镜像的基础知识，以 Amazon AWS 为目标。我们介绍了 Packer 提供的最重要插件，以自定义 AMI。然后，我们构建了一个自定义镜像，通过 `apt` 工具安装了多个软件包，下载并安装了其他工具，设置了目录和用户，最后设置了一个系统代理来与 systemd 一起运行。

我们已经介绍了如何使用 Goss 工具验证您的镜像，以及如何通过耶鲁大学开发的插件将 Goss 集成到 Packer 中。

最后，我们展示了如何创建您自己的插件，以扩展 Packer 的功能。

现在，是时候谈谈 IaC 以及 HashiCorp 的另一个工具如何在 DevOps 世界中掀起风潮了。我们来谈谈 Terraform。
