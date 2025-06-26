

# Linux 中的网络

网络是一个复杂的话题，无论在哪个操作系统上都是如此。就灵活性而言，Linux 在配置、内核功能和命令行工具的众多可能性上可能让人感到非常不知所措，这些工具可以帮助我们配置这些选项。在本章中，我们将为这个话题奠定基础，以便您可以在其他出版物中查找有关特定主题的更多信息。在本章中，我们将涵盖以下主题：

+   Linux 中的网络

+   ISO/OSI 作为网络标准

+   防火墙

+   高级话题

# Linux 中的网络

在 Linux 中，网络是通过内核实现的，这意味着它是操作系统的一部分。内核包括几个组件，这些组件协同工作以实现网络功能，包括设备驱动程序、协议实现和系统调用。

当用户想要通过网络发送或接收数据时，他们可以使用 Linux 中任何可用的网络应用程序，如`ping`、`traceroute`、`telnet`或`ssh`。这些应用程序使用系统调用与内核通信，并请求通过网络发送或接收数据。

内核通过设备驱动程序与网络硬件进行通信，设备驱动程序是软件程序，使得内核能够访问和控制硬件。不同类型的网络硬件需要不同的驱动程序，例如以太网或 Wi-Fi。

内核还实现了几个网络协议，这些协议是定义如何在网络上传输和格式化数据的规则和标准。Linux 中常用的协议包括 TCP、UDP 和 IP（版本`4`和版本`6`）。

# ISO/OSI 作为网络标准

关于网络的任何讨论总是从**国际标准化组织/开放系统互联**（**ISO/OSI**）定义的参考模型开始。ISO/OSI 参考模型是一个概念模型，定义了一个网络框架，用于在七层中实现协议。它是一个框架，让我们可以将系统（计算机或其他）之间的通信与其实际的物理和软件结构区分开来看。

在 Linux 中，OSI 模型是通过一系列软件组件来实现的，这些组件负责执行每一层的功能。这些组件共同作用，实现 Linux 中的网络功能。

在 Linux 中实现的 OSI 模型的七层如下：

+   物理层

+   数据链路

+   网络层

+   传输层

+   会话层

+   表示层

+   应用层

在运行在云端的系统中，您将访问到 Linux 内核中实现的所有层。这些层包括网络层、传输层、会话层、表示层和应用层。为了调试网络连接、检查统计信息并找出任何其他可能的问题，Linux 提供了一个控制台工具。让我们逐一查看每一层，研究我们可以在 Linux 中使用哪些命令行工具来检查它们。

要了解更多关于 OSI 模型的信息，可以参考 [`osi-model.com/`](https://osi-model.com/)。我们将在接下来的子节中深入探讨这些层并对其进行解释。

## 物理层

这一层负责通过通信通道传输原始比特，并通过设备驱动程序来实现，这些驱动程序控制网络硬件，例如以太网标准，如 `10BASE-T`、`10BASE2`、`10BASE5`、`100BASE-TX`、`100BASE-FX`、`100BASE-T`、`1000BASE-T` 等。由于我们将专注于软件实现以及如何在 Linux 控制台上与其交互，因此在这里不会进一步讨论这一层。你可以在网上找到大量关于电缆、硬件设备和网络的信息。

## 数据链路层 – MAC，VLAN

**数据链路层**负责在网络上的设备之间提供可靠的链路。它分为**逻辑链路控制**（**LLC**）子层和**媒体访问控制**（**MAC**）子层。

数据链路层将来自网络层（第 3 层）的原始数据转换为可以通过物理链路传输的格式。它还提供错误检测和校正、流量控制以及 MAC 功能。

LLC 子层为网络层提供一致的接口，不论使用何种类型的物理网络。它还提供流量控制和错误校正服务。

MAC 子层控制对物理网络的访问并提供寻址服务。它使用 MAC 地址，这是分配给网络上每个设备的唯一标识符，确保数据能传送到正确的目的地。

数据链路层还包括使用协议，如以太网、PPP 和帧中继，以在网络上提供设备间的通信。它还提供流量和错误控制机制—例如，它使用**循环冗余校验**（**CRC**）进行错误检测，并使用滑动窗口进行流量控制。

有多个 Linux 命令行工具可用于调试数据链路层问题。以下是一些示例：

+   `ifconfig`：此命令可用于查看网络接口的状态及其相关的 IP 地址、子网掩码和 MAC 地址。它还可以用来配置网络接口，例如设置 IP 地址或启用或禁用接口。

+   `ping`：此命令可用于测试网络中主机的可达性。它向指定主机发送一个**互联网控制消息协议**（**ICMP**）回显请求数据包，并等待回显响应。如果主机响应，则表明主机可达，并且数据链路层工作正常。

+   `traceroute`、`tracepath`和`mtr`：这些命令可以用来追踪数据包从源到目的地的路径。它们还可以用来识别任何可能导致问题的网络跳数或设备。此外，`tracepath`可以测量`mtr`，而`mtr`则提供更多关于网络健康的信息。

+   `arp`：此命令可用于查看和操作**地址解析协议**（**ARP**）缓存。ARP 用于将 IP 地址映射到本地网络上的 MAC 地址。此命令可以用来验证 ARP 缓存中是否有正确的 IP-MAC 地址映射。

+   `ethtool`：此命令可用于查看和配置以太网接口的高级设置，如链路速度、双工模式和自动协商设置。

+   `tcpdump`：此命令可用于实时捕获和分析网络数据包。它可以用来排查诸如数据包丢失、延迟包和网络拥堵等问题。

我们将在本章的接下来的部分中深入探讨前述工具，因为大多数工具可以用于同时查看多个 OSI 层。

## 网络层 – IPv4 和 IPv6

每个公有（互联网）或私有（你的办公室或家里）网络上的设备都有一个唯一地址，用于识别它并与之连接。当你向一个网站发送请求时，你的设备会使用其地址向目标服务器发送消息。服务器随后会使用其地址向你的设备发送消息以响应。这一过程就是设备通过互联网互相通信的方式。我们使用的地址有两种类型：IPv6 和 IPv4。`v4`（版本`4`）和`v6`（版本`6`）是你可以使用的地址数量。

IPv4 是最广泛使用的 IP 版本，但只有大约 43 亿个唯一地址可用。然而，这些地址不足以支持不断增加的连接到互联网的设备数量。为了解决这个问题，开发了一种新的 IP 版本——IPv6，提供了更大的地址空间。

IPv4 地址是 32 位数字，通常以点分十进制表示，包含四个范围从`0`到`255`的八位字节。例如，`192.168.0.1`或`1.2.3.4`都是有效的 IPv4 地址。

IPv6 地址是 128 位数字（在十六进制中介于`zero`和`FFFF`之间，等于十进制值`65535`），以十六进制表示，采用八组四个十六进制数字，并以冒号分隔。IPv6 地址的示例为`2001:0db8:bad:f00d:0000:dead:beef:7331`。

我们在这里主要关注 IPv4，因为它通常更容易理解，但类似的原则也适用于 IPv6，因此稍后我们将在本章中学到的内容更容易重新应用到 IPv6 环境中。

### 子网、类别和网络掩码

**子网** 是通过将一个较大的网络划分成更小的网络而创建的网络片段。这么做的原因有很多，包括安全性、组织结构以及高效利用 IP 地址。在公共互联网网络中，每个组织都有自己的一部分网络——一个子网。

当一个网络被划分成子网时，IP 地址中的主机部分（标识网络中具体设备的部分）会被分成两部分：一部分标识子网，另一部分标识子网内的主机 IP 地址。子网掩码是应用于 IP 地址的二进制表示，用于确定 IP 地址中哪一部分标识子网，哪一部分标识主机。

假设一个使用 IP 地址范围 `192.168.11.0/24`（或 `192.168.11.0/255.255.255.0` 的十进制形式）的网络。`/24` 部分表示 `24` 位网络部分和 `8` 位主机部分。

这意味着该网络的 IP 地址有 24 位（或地址中的前三个数字）用于 `网络` 部分，8 位（或最后一个数字）用于 `主机` 部分。所以，对于这个网络，你将有一个可用的地址范围 `192.168.11.0 - 192.168.11.255`，其中 `192.168.11.0` 是网络地址，`192.168.11.255` 是广播地址。

**广播网络地址** 是一种特殊类型的 IP 地址，用于向特定网络或子网中的所有主机发送信息。广播地址是网络或子网 IP 地址范围中的最高地址，并与子网掩码一起使用，以标识广播域。当主机将数据包发送到广播地址时，该数据包将被送到同一网络或子网中的所有主机。需要注意的是，广播数据包不会离开当前的子网网络——它们仅在本地网络或子网内起作用。

在标准化 CIDR 之前，IP 地址根据网络需要支持的主机数量被分为不同的类别（`A`、`B` 和 `C`）。这些类别是根据 IP 地址的前导位来定义的，每个类别有不同数量的位用于网络部分和主机部分。目前我们主要使用 CIDR，但一些网络地址仍然存在。例如，`10.0.0.0`、`12.0.0.0` 或 `15.0.0.0` 通常具有 `255.0.0.0` 或 `/8` 网络掩码。以下是你可能会遇到的一些其他网络：

+   `10.0.0.0/8`、`12.0.0.0/8` 和 `15.0.0.0/8`

+   `172.16.0.0/16`、`172.17.0.0/16` 和 `172.18.0.0/16`

+   `192.168.0.0/24`、`192.168.1.0/24` 和 `192.168.2.0/24`

这些例子并不是实际的网络，而仅仅是通过查看 IP 地址的前导位和子网掩码来识别网络类别的表示——这并不是一种规则，你可以在你的基础设施中创建更小（或更大）的网络。

你可以在网上找到许多计算器，帮助你更好地理解网络地址是如何工作的。这里有一些例子：

+   [`www.calculator.net/ip-subnet-calculator.xhtml`](https://www.calculator.net/ip-subnet-calculator.xhtml)

+   [`mxtoolbox.com/subnetcalculator.aspx`](https://mxtoolbox.com/subnetcalculator.aspx)

+   [`www.subnet-calculator.com/cidr.php`](https://www.subnet-calculator.com/cidr.php)

既然我们已经学习了子网、类别和网络掩码，那么接下来让我们进入下一个小节。

### 网络配置和控制台工具

通过你目前所掌握的知识，你可以轻松地使用一些在现代 Linux 环境中都可用的命令行工具来检查你的 Linux 网络配置。

这里是一些用于网络配置的基本控制台工具：

+   `iproute2` 包，取代了 `ifconfig` 和 `route` 命令

+   `ifconfig`

+   `route`

+   `ip`

+   `netplan`

让我们一起了解这些工具的语法以及使用它们时可能实现的功能。

#### ifconfig

在 Linux 中，通常可用的命令之一是`ifconfig`。这个工具用于配置网络接口。它可以用来显示接口的状态、为接口分配 IP 地址、设置网络掩码以及设置默认网关。`ifconfig`（来自 net-tools 包）在近年来被 `iproute2` 工具集所取代；这个包中最著名的命令是 `ip`。

以下是 `ifconfig` 命令的示例输出：

```
admin@myhome:~$ ifconfig
eth0: flags=4163<UP,BRODCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.17.0.2  netmask 255.255.0.0  broadcast 172.17.255.255
        ether 02:42:ac:11:00:02  txqueuelen 0  (Ethernet)
        RX packets 17433  bytes 26244858 (26.2 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 8968  bytes 488744 (488.7 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

当没有任何额外选项时，执行 `ifconfig` 会列出系统中所有可用的接口，并显示一些基本信息，如接口名称（这里是 `eth0` 和 `lo`）、网卡的 MAC 地址（设备的物理地址）、网络配置（IP 地址、网络掩码和广播地址）、接收和发送的包，以及其他信息。这将让你一眼查看网络状态。

回环设备（在上面的示例中命名为 `lo`）是一个虚拟网络接口，用于将网络数据包发送回同一个发送源的主机。它也被称为回环接口，并由 `lo` 或 `lo0` 表示。

回环设备的主要作用是为主机提供一种稳定且一致的方式，通过网络栈与自身进行通信，而不需要依赖任何物理网络接口。

回环接口通常用于测试、故障排除和一些系统与应用程序功能，以及**进程间通信**（**IPC**），尤其是在同一主机上运行的进程之间。

使用 `ifconfig` 还可以让你启用或禁用某些接口并配置它们的设置。要使配置保持有效，你需要将其保存到`/etc/network/interfaces` 文件中：

```
auto lo
iface lo inet loopback
# eth0 network device
auto eth0
iface eth0 inet static
    address 172.17.0.2
    netmask 255.255.0.0
    gateway 172.17.0.1
    dns-nameservers 8.8.8.8 4.4.4.4
auto eth1
allow-hotplug eth1
iface eth1 inet dhcp
```

在前面的例子中，我们设置了一个自动回环设备，分别为`eth0`和`eth1`。`eth0`接口具有静态网络配置，并将在系统启动时像`lo`一样进行配置。`eth1`接口具有动态网络配置，它是通过`allow-hotplug`配置获取的，这意味着该设备将在 Linux 内核检测到后启动。

需要知道的是，在编辑`/etc/network/interfaces`文件后，您需要在 Debian Linux 或 Ubuntu Linux 中使用`ifup`或`ifdown`工具，或者在 Alpine Linux 中使用`ifupdown`工具。或者，您可以通过在 Debian Linux、Ubuntu Linux 或 RHEL/CentOS 中使用`systemctl restart`网络来重启网络。必须在 Alpine Linux 中使用`rc-service`网络`restart`命令。

要使用`ifconfig`手动配置设备，您需要运行以下命令：

```
admin@myhome:~$ ifconfig eth0 192.168.1.2 netmask 255.255.255.0
admin@myhome:~$ ifconfig eth0 up
```

这将为`eth0`接口配置一个`192.168.1.2`的 IP 地址，网络的子网掩码为`255.255.255.0`（或 CIDR 表示法中的`/24`）。

在 Debian Linux 和 Ubuntu Linux 系统中，您可以使用`ifup`和`ifdown`代替`ifconfig up`和`ifconfig down`命令，或者在 Arch Linux 系统中使用`ifupdown`。

#### route

`route`命令用于查看和操作 IP 路由表。它用于确定网络数据包在给定目标 IP 地址下的发送位置。

在不带任何选项的情况下调用`route`命令将显示系统中的当前路由表：

```
admin@myhome:~$ route
Kernel IP routing table
Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
default       172.17.0.1      0.0.0.0         UG    0      0        0 eth0
172.17.0.0      0.0.0.0       255.255.0.0     U     0      0       0 eth0
```

要向路由表添加新条目，请使用以下命令：

```
admin@myhome:~$ sudo route add default gw 172.17.0.1 eth0
```

只能有一个默认路由。要添加自定义路由，请执行以下操作：

```
admin@myhome:~$ sudo route add -net 192.168.1.0 netmask 255.255.255.0 gw 192.168.1.1 dev eth0
```

`del`命令用于从路由表中删除条目，类似于前面的示例：

```
admin@myhome:~$ route del -net 192.168.1.0 gw 192.168.1.1 netmask 255.255.255.0 dev eth0
```

最后，`flush`命令会删除路由表中的所有条目，这意味着您将失去所有网络连接，如果您连接到远程机器，您将无法再对其进行操作。

使用`ifconfig`和`route`命令还有更多的可能性，但正如我们之前所说，两个命令都被`iproute2`包（`iproute`的继任者）取代，该包包括`ip`命令。

#### iproute2

更高级的命令用于操作路由和网络设备的配置是`ip`，它可以用来执行更广泛的任务，如创建和删除接口、添加和删除路由以及显示网络统计信息。

让我们来看看使用`iproute2`时可以执行的最常见命令：

+   `ip addr`或`ip a`：此命令显示有关网络接口及其 IP 地址的信息。它还支持子命令；例如，`ip addr add`命令可用于向接口添加 IP 地址，而`ip route add`可用于向路由表中添加路由。

+   `ip link`或`ip l`：此命令显示有关网络接口及其链路层设置的信息。

+   `ip route`或`ip r`：此命令显示 IP 路由表。

+   `ip -s link`（或`ip -s l`）：这将显示关于网络接口的统计信息。

以下是运行`ip link`和`ip` `addr`命令的输出：

```
admin@myhome:~$ sudo ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/tunnel6 :: brd :: permaddr 4ec4:2248:2903::
47: eth0@if48: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
admin@myhome:~$ sudo ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: tunl0@NONE: <NOARP> mtu 1480 qdisc noop state DOWN group default qlen 1000
    link/ipip 0.0.0.0 brd 0.0.0.0
3: ip6tnl0@NONE: <NOARP> mtu 1452 qdisc noop state DOWN group default qlen 1000
    link/tunnel6 :: brd :: permaddr 4ec4:2248:2903::
47: eth0@if48: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

两个命令打印出非常相似的信息，但`ip addr`除了提供物理接口的信息外，还添加了有关网络配置的信息。`ip link`命令用于控制接口状态。与`ifconfig up eth0`启用接口类似，`ip link set dev eth0 up`也会做同样的事情。

要使用`iproute2`配置网络接口，你需要执行以下命令：

```
admin@myhome:~$ sudo ip addr add 172.17.0.2/255.255.0.0 dev eth0
admin@myhome:~$ sudo ip link set dev eth0 up
```

要将接口设置为默认路由，请使用以下命令：

```
admin@myhome:~$ sudo ip route add default via 172.17.0.1 dev eth0
```

要仅检查`eth0`接口的状态，你可以执行以下命令：

```
admin@myhome:~$ sudo ip addr show dev eth0
47: eth0@if48: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff link-netnsid
0
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
```

#### netplan

这个工具是 Ubuntu 中引入的新网络配置工具，并且在 Debian 中也得到了支持。它是一个 YAML 配置文件，可用于管理网络接口、IP 地址和其他网络设置。它首次在 Ubuntu 17.10 中作为传统`/etc/network/interfaces`文件的替代品被引入，随后也被其他发行版（如 Debian 和 Fedora）采纳。Ubuntu 18.04 及更高版本默认安装了`netplan`，其他发行版如 Debian 10 和 Fedora 29 及更高版本也默认包含了`netplan`。

要使用 Netplan，你首先需要在`/etc/netplan/`目录中创建一个配置文件。该文件应具有`.yaml`扩展名，并且应该命名为具有描述性的名称，如`01-eth0.yaml`或`homenetwork.yaml`。

样本配置如下所示：

```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      dhcp4: true
```

该配置定义了一个单一的网络接口`eth0`，它使用 DHCP 获取 IP 地址。`renderer`键指定了 netplan 使用的网络管理器（在本例中为`networkd`）。`version`键用于表示正在使用的`netplan`版本。`networkd`是`systemd`系统和服务管理器的一部分，用于网络管理的守护进程。

为`eth0`接口配置静态 IP 地址的配置文件将如下所示：

```
network:
  version: 2
  renderer: networkd
  ethernets:
    eth0:
      addresses: [192.168.0.2/24]
      gateway4: 192.168.0.1
      nameservers:
        addresses: [8.8.8.8, 4.4.4.4]
```

该配置文件定义了一个以太网接口（`eth0`）及其静态 IP 地址。此接口还定义了网关和 DNS 服务器。注意，我们使用了 CIDR 表示法，而不是十进制表示法。

一旦你保存了配置文件，为了应用更改，可以运行以下命令：

```
admin@myhome:~$ sudo netplan apply
```

你还可以使用以下命令检查配置文件是否有语法错误：

```
admin@myhome:~$ sudo netplan --debug generate
```

你可以使用以下命令检查网络接口的当前状态：

```
admin@myhome:~$ sudo netplan --debug try
```

最后，如果你想检查网络接口的状态而不应用配置，可以使用以下命令：

```
admin@myhome:~$ sudo netplan --debug networkd try
```

要查看与网络接口启动时的错误相关的日志，可以使用 `dmesg` 命令查看内核消息，包括与网络接口相关的消息。您可以使用 `dmesg | grep eth0` 来过滤与 `eth0` 接口相关的日志。其他位置包括 `/var/log/messages` 文件，`journalctl`（例如，`journalctl -u systemd-networkd.service` 命令），以及包含 `netplan` 生成的日志的 `/var/log/netplan/`。

在日常操作中，编辑 `/etc/network/interfaces` 文件或 `netplan` 配置比手动配置接口更常见，但是知道如何临时更改网络配置以进行测试或调试问题非常有用。

接下来，我们将介绍传输层。

## 传输层 – TCP 和 UDP

在 OSI 模型的**传输层**中，我们更多地关注**传输控制协议**（**TCP**）和 IP，它们是现代互联网的基础。此外，我们还将深入了解**用户数据报协议**（**UDP**）。我们在本章的*网络层 – IPv4 和 IPv6* 小节中讨论了 IP，因此这里我们只会稍微加深对该协议的了解。

TCP 用于通信，需要服务之间可靠的双向通信。UDP 是一种无状态协议，不需要持续连接。

使用 *三次握手* 建立了 TCP 连接：

1.  客户端向服务器发送一个 `SYN`（同步）包，以初始化连接。

1.  服务器接收到 `SYN` 包，并向客户端发送一个 `SYN-ACK`（同步-确认）包，以确认连接已建立。

1.  客户端接收到 `SYN-ACK` 包，并向服务器发送一个 `ACK`（确认）包，以完成三次握手。

一旦完成三次握手，设备就可以通过已建立的 TCP 连接开始互相发送数据。

TCP 和 UDP 协议的连接是通过向连接的服务器端口发送数据包来初始化的。端口号包含在 IP 数据包头中，与 IP 地址一起，以便目标设备知道将数据发送到哪个进程。

端口是一个整数，范围从`0`到`65535`。它用于标识设备上运行的特定进程，并与其他服务区分开来。端口号会附加在 IP 地址后面。因此，如果多个程序在同一 IP 地址上监听连接，另一端可以准确知道它想要与哪个程序通信。假设有两个进程正在运行并监听：一个 WWW 服务器和一个 SSH 服务器。WWW 服务器通常会监听`80`或`443`端口，SSH 服务器会监听`22`端口。假设系统的 IP 地址是`192.168.1.1`。要连接到 SSH 服务器，我们会将 IP 地址与端口地址配对（通常写作`192.168.1.1:22`）。服务器会知道该传入的连接需要由 SSH 进程处理。与 TCP 不同，UDP 连接并不会建立与服务器机器的连接。相反，设备可以在已知端口上相互发送 UDP 数据报（**数据包**）。

当设备发送 UDP 数据报时，它会在数据包头中包含目标 IP 地址和端口号。接收设备检查数据包头中的目标 IP 地址和端口号，以确定将数据发送到哪个进程。

由于 UDP 是无连接的，因此不能保证数据会被目标设备接收，或者接收的顺序与发送的顺序相同。也没有错误检查或丢失数据包的重传。UDP 通常用于需要低开销和快速通信的服务。最著名的使用 UDP 进行通信的服务是**域名系统**（**DNS**）。

端口号有三种类型：

+   `/etc/services` 文件。这些是为特定服务保留的端口号，例如`HTTP`流量的`80`端口或 DNS 流量的`53`端口。

+   `1` 和 `1024`。

+   **短暂端口**是用于临时连接的端口号，并由操作系统动态分配，例如用于 TCP 连接。

有各种工具可供查看机器上 TCP 和 UDP 流量的详细信息。你还可以设置防火墙，以便控制从网络访问你机器的权限。

### netstat

要查看从你的机器发起的所有 TCP 连接，可以使用`netstat`命令。它可以用于查看打开的连接列表，并显示系统网络流量的统计信息。

要查看系统上所有打开的连接列表，可以使用`netstat -an`命令。这将显示所有当前连接的列表，包括本地和远程 IP 地址及端口，以及连接状态（例如，监听、已建立等）。

使用`netstat -s`，你可以查看系统上每个连接的统计信息。这将显示关于系统网络流量的各种统计数据，包括发送和接收的包数量、错误数量等。

要查看 `netstat` 中的所有 UDP 连接，你可以使用 `netstat -an -u` 命令。它会显示所有当前的 UDP 连接，包括本地和远程的 IP 地址与端口，以及连接的状态。

或者，你也可以使用 `netstat -an -u | grep "udp" | grep "0.0.0.0:*"` 命令，只显示处于监听状态的 UDP 连接。这个命令过滤 `netstat -an -u` 的输出，只显示包含 `"UDP"` 和 `"0.0.0.0:*"` 的行，这表示一个监听中的 UDP 连接。

你还可以使用其他选项，例如 `-p` 显示每个连接所属进程的进程 ID 和名称，`-r` 显示路由表，`-i` 显示特定接口的统计信息。

### tcpdump

`tcpdump` 是一个命令行数据包分析器，允许你通过显示传输或接收的网络数据包来捕获和分析网络流量。`tcpdump` 可以用来排查网络问题、分析网络性能以及监控网络安全。你可以在特定接口上捕获数据包，基于各种标准过滤数据包，并将捕获的包保存到文件中以供后续分析。

要捕获并显示 `eth0` 接口上的所有网络流量，你可以使用 `-i` 选项，后跟接口名称。要捕获并显示 `eth0` 接口上的所有网络流量，你需要运行 `sudo tcpdump -i eth0`。

你还可以通过使用 `-w` 选项将捕获的数据包保存到文件中，以便以后分析——例如，`tcpdump -i eth0 -w all_traffic.pcap`。这将把 `eth0` 接口上捕获的所有数据包保存到文件中。

默认情况下，`tcpdump` 会无限期地捕获数据包。

要捕获特定时间内的数据包，你可以使用 `-c` 选项，后跟要捕获的数据包数量——例如，`sudo tcpdump -i eth0 -c 100`。该命令会捕获并显示 `100` 个数据包，然后退出。

你还可以通过使用像 `port`、`ip`、`host` 等过滤器来筛选流量——例如，`sudo tcpdump -i eth0 'src host 192.168.1.2 and (tcp or udp)'`。这个过滤器捕获所有源 IP 地址为 `192.168.1.2` 且为 `TCP` 或 `UDP` 的数据包。

为了展示 `tcpdump` 更高级的用法，让我们只捕获 `SYN` 数据包（查看所有正在建立的连接）。你可以通过使用 `tcp[tcpflags] & (tcp-syn) != 0` 过滤器来实现。这个过滤器检查数据包的 `TCP` 头部中是否设置了 `SYN` 标志。

这是一个命令示例，它捕获并显示 `eth0` 接口上的所有 `SYN` 数据包：

```
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0'
```

你还可以通过使用 `-w` 选项将捕获的数据包保存到文件中，以便后续分析，如下所示：

```
sudo tcpdump -i eth0 -w syn_packets.pcap 'tcp[tcpflags] & (tcp-syn) != 0'
```

这将把所有捕获的 `SYN` 数据包保存到 `syn_packets.pcap` 文件中。

你也可以指定一个更复杂的过滤器，如下所示：

```
sudo tcpdump -i eth0 'tcp[tcpflags] & (tcp-syn) != 0 and src host 192.168.1.2'
```

这个过滤器仅捕获源 IP 地址为 `192.168.1.2` 的 `SYN` 数据包。

### Wireshark

另一个类似于`tcpdump`的流行工具是 Wireshark。它既可以在无头模式下使用（仅在命令行中），也可以通过图形界面使用：

+   要使用 Wireshark 显示`eth0`接口上的所有流量，你可以使用`sudo wireshark -i eth0`命令。这将启动 Wireshark 并监听`eth0`接口上的流量。你还可以使用`-k`标志立即开始捕获，并使用`-w`标志将捕获的流量写入文件：`sudo wireshark -k -i eth0 -w output_file.pcap`。

+   如果你想仅显示`SYN`数据包，如同我们在`tcpdump`示例中所展示的那样，你可以运行`sudo wireshark -i eth0 -f "tcp.flags.syn == 1"`命令。前面的命令使用了一个过滤器，`"tcp.flags.syn == 1"`，这表示我们只想看到标记为`SYN`的`TCP`协议标志。你也可以在 Wireshark 的图形界面版本中使用此过滤器，方法是进入**捕获**菜单，选择**选项**，然后在**捕获过滤器**字段中输入过滤器，再开始捕获。

或者，在捕获流量后，你也可以通过点击过滤字段中的`"tcp.flags.syn==1"`并按*Enter*来应用该过滤器。

### ngrep

下一个非常有用的工具是`ngrep`，它类似于`tcpdump`和 Wireshark。与我们之前提到的其他工具不同，它的使用更加简单，而且允许你（类似于`grep`）在网络数据包中搜索字符串。

例如，要监视`GET HTTP`请求，你可以使用以下命令：

```
admin@myhome:~$ ngrep -d eth0 -q -W byline "GET" "tcp and port 80"
interface: en0 (192.168.1.0/255.255.255.0)
filter: ( tcp and port 80 ) and ((ip || ip6) || (vlan && (ip || ip6)))
match: GET
# Later output will contain actual GET requests, removed for readability
```

该命令将在`eth0`接口上监听，并仅匹配`80`端口上的`TCP`数据包（`HTTP`的默认端口）。`-q`选项告诉`ngrep`保持安静，不显示数据包摘要信息，而`-W`则告诉`ngrep`将每个数据包的数据打印在单独的行上。此时，我们可以进入会话层。

## 会话层

**会话层**，正如我们之前提到的，它负责在设备之间建立、维护和终止连接。这意味着它设置、协调和终止应用程序之间的对话、交换或连接。会话层通过使用令牌管理和检查点等技术，确保数据可靠地传输并按正确顺序传送。它还负责解决会话中可能出现的任何冲突，例如当两个应用程序试图同时发起会话时。简而言之，会话层在网络中建立、维护和终止设备之间的连接。

换句话说，会话层是我们已经讨论过的下层与接下来几节将要介绍的上层之间的粘合剂。

解决会话问题的最佳工具是与你正在使用的服务相关的日志。如果你遇到 FTP 连接问题，可能需要查看你管理的机器上运行的客户端和/或服务器的日志。如果日志不足以理解你要解决的问题，之前介绍的工具也可以提供帮助。

## 表示层 – SSL 和 TLS

`ASCII`）或 8 位（`EBCDIC`）整数。我们还在这一层处理加密和压缩。表示层还确保数据格式正确，便于应用程序处理。它充当应用程序与数据之间的中介，使得应用程序不受接收数据特定格式的影响。

对于这一层，最常见的加密标准是**安全套接字层**（**SSL**）和**传输层安全性**（**TLS**），你可能需要调试和修复这两个协议的问题。

**TLS**是一个广泛使用的协议，用于确保网络通信的安全性。它是**SSL**的继任者，用于加密和验证通过网络（如互联网）传输的数据。

TLS 通过在两个设备之间建立一个安全的*tunnel*（隧道）来工作，例如 Web 服务器和 Web 浏览器。该隧道用于在两个设备之间以加密格式传输数据，使得攻击者难以拦截和读取数据。

建立 TLS 连接的过程包括几个步骤：

1.  **握手**：客户端和服务器交换信息，以建立共享的加密方法和密钥，确保连接的安全。

1.  **认证**：服务器通过提供包含服务器身份和公钥信息的数字证书向客户端进行身份验证。客户端可以使用这些信息验证服务器的身份。

1.  **密钥交换**：客户端和服务器交换公钥，以建立一个共享的秘密密钥，该密钥将用于加密和解密数据。

1.  **数据加密**：一旦共享的秘密密钥建立，客户端和服务器就可以使用对称加密算法开始加密数据。

1.  **数据传输**：然后，数据通过安全连接进行传输，并通过握手过程中建立的加密保护。

TLS 有多个版本，其中最新的版本被认为更为安全。然而，旧系统可能不支持最新版本。可用的 TLS 版本如下：

+   **TLS 1.0**：这是该协议的第一个版本，于 1999 年发布。

+   **TLS 1.1**：该版本于 2006 年发布。

+   **TLS 1.2**：该版本于 2008 年发布，增加了对新加密算法的支持，并做了其他多个安全性增强。

+   **TLS 1.3**：该版本于 2018 年发布，采用了前向保密性（forward secrecy），使得攻击者更难解密捕获的数据。

SSL 是一种广泛用于保障网络通信安全的协议，如互联网通信。它由 Netscape 在 1990 年代开发，后来被 TLS 协议所取代。

和 TLS 一样，SSL 通过在两个设备之间建立一个安全的*tunnel*（隧道）来工作，比如网页服务器和网页浏览器。这个隧道用于在两个设备之间传输加密格式的数据，使得攻击者很难拦截和读取数据。目前不建议再使用 SSL，最好使用最新版本的 TLS，但你可能仍然会遇到使用 SSL 的系统。

调试 SSL 和 TLS 问题的最佳工具是`openssl`命令。你可以用它测试与服务器的 SSL 连接。例如，你可以使用以下命令测试与服务器在端口（通常是`443`，这是 HTTPS 的常用端口）的连接：

```
admin@myhome:~$ openssl s_client -connect myhome:443
```

你可以使用`openssl`命令检查 SSL 证书的详细信息，包括过期日期、发行机构和公钥。例如，你可以使用以下命令来检查证书的详细信息：

```
admin@myhome:~$ openssl s_client -connect myhome:443 -showcerts
```

这将允许你检查服务器提供的证书是否有效，并且是否符合你的预期。这比使用网页浏览器要快得多。

使用`openssl`命令，你还可以检查服务器支持哪些加密算法。例如，你可以使用以下命令检查服务器支持的加密算法：

```
admin@myhome:~$ openssl s_client -connect myhome:443 -cipher 'ALL:eNULL'
# Cut most of the output for readability
SSL handshake has read 5368 bytes and written 415 bytes
---
New, TLSv1/SSLv3, Cipher is AEAD-AES256-GCM-SHA384
Server public key is 2048 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : AEAD-AES256-GCM-SHA384
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Start Time: 1674075742
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---
closed
```

此外，`openssl`还内置了诊断工具，可以检测系统中已知的漏洞：

```
admin@myhome:~$ openssl s_client -connect myhome:443 -tlsextdebug -status
CONNECTED(00000006)
140704621852864:error:1404B42E:SSL routines:ST_CONNECT:tlsv1 alert protocol version:/AppleInternal/Library/BuildRoots/aaefcfd1-5c95-11ed-8734-2e32217d8374/Library/Caches/com.apple.xbs/Sources/libressl/libressl-3.3/ssl/tls13_lib.c:151:
---
no peer certificate available
---
No client certificate CA names sent
---
SSL handshake has read 5 bytes and written 303 bytes
---
New, (NONE), Cipher is (NONE)
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.3
    Cipher    : 0000
    Session-ID:
    Session-ID-ctx:
    Master-Key:
    Start Time: 1674075828
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
---
```

还值得注意的是，SSL 和 TLS 都使用公钥加密。在这种加密方法中，我们会创建两个文件：一个私钥和一个公钥。它们共同构成一对。公钥用于加密数据，而私钥用于解密数据。私钥应始终保密，正如它的名字所示。这种加密方法基于大素数的数学特性，被认为是非常安全的。

在 TLS 的情况下，公钥加密在*握手*阶段用于建立客户端与服务器之间的安全连接。这个过程如下：

1.  服务器生成公钥和私钥。公钥作为服务器数字证书的一部分发送给客户端。

1.  客户端生成一个会话密钥，用于加密发送到服务器的数据。服务器的公钥用于加密会话密钥。一旦服务器收到加密的数据，它使用私钥解密数据并恢复会话密钥。

1.  一旦会话密钥被解密，它就会用于加密客户端与服务器之间发送的数据。

接下来的最后一层是 OSI 模型的顶层。在接下来的章节中，我们将介绍应用层，涵盖诸如 HTTP 和 FTP 等协议，这些协议通常用于浏览网页和共享文件。

## 应用层 – HTTP 和 FTP

**应用层**是 OSI 网络模型的第七层，也是最高层。它提供软件应用程序与网络之间的接口，使得应用程序能够访问网络的通信服务。应用层定义了特定于应用程序的协议和服务，例如文件传输（**文件传输协议**（**FTP**））、电子邮件（**简单邮件传输协议**（**SMTP**）和**互联网消息访问协议**（**IMAP**））以及广泛使用的网络服务（**超文本传输协议**（**HTTP**））。

让我们更仔细地看看 HTTP。这个通信协议用于在万维网上传输数据。它基于客户端-服务器模型，其中 Web 浏览器（客户端）向 Web 服务器发送请求，服务器则返回响应。

当用户在浏览器中输入**统一资源定位符**（**URL**）时，浏览器会向与该 URL 关联的网络服务器发送 HTTP 请求。URL 是你在浏览器地址输入框中看到的内容，通常位于浏览器窗口的顶部，例如，[`google.com`](https://google.com)。

请求包含方法（如`GET`、`POST`、`PUT`、`PATCH`或`DELETE`），指示浏览器希望服务器执行的操作类型，并可能包含其他信息，如`POST`或`PUT`请求的数据。

然后，网络服务器处理请求，之后服务器会发送回一个 HTTP 响应，包含状态码（如`200`表示成功或`404`表示`未找到`）以及浏览器请求的任何数据，例如构成网站的 HTML 和 CSS。

一旦浏览器收到响应，它会解析 HTML、CSS，通常还包括 JavaScript，以便将网站显示给用户。

HTTP 是一种无状态协议，这意味着每个请求都是独立的，服务器不会保留任何关于先前请求的信息。然而，许多 Web 应用程序使用 cookies 或其他技术来维持多个请求之间的状态。

HTTP 1.1 版本引入了新特性，如持久连接、主机头字段和字节服务，这些改进提高了协议的整体性能，并使其更适合重负载使用场景。该协议的最新版本是 2.0，详细描述在 RFC 7540 中（https://www.rfc-editor.org/rfc/rfc7540.xhtml），该版本于 2015 年发布，并在 2020 年由 RFC 8740（[`www.rfc-editor.org/rfc/rfc8740.xhtml`](https://www.rfc-editor.org/rfc/rfc8740.xhtml)）更新。

要解决 HTTP 问题，您可以使用任何调试网络问题的工具，例如 `tcpdump` 或 `ngrep`。有多种控制台和图形界面工具可供调试 HTTP。最常见的控制台工具是 `wget` 和 `curl`，图形界面工具包括 `Postman` 或 `Fiddler`。浏览器中也会内置调试工具，例如 Firefox 或 Chrome 开发者工具。

我们现在将重点关注控制台工具，因此首先来看看 `wget`。这个工具本用于下载文件，但我们仍然可以用它来调试 HTTP。首先，我们将展示请求和响应的详细信息：

```
admin@myhome:~$ wget -d https://google.com
DEBUG output created by Wget 1.21.3 on darwin22.1.0.
Reading HSTS entries from /home/admin/.wget-hsts
URI encoding = 'UTF-8'
Converted file name 'index.xhtml' (UTF-8) -> 'index.xhtml' (UTF-8)
--2023-01-20 14:11:02--  https://google.com/
Resolving google.com (google.com)... 216.58.215.78
Caching google.com => 216.58.215.78
Connecting to google.com (google.com)|216.58.215.78|:443... connected.
Created socket 5.
Releasing 0x0000600000d2dac0 (new refcount 1).
Initiating SSL handshake.
Handshake successful; connected socket 5 to SSL handle 0x00007fca4a008c00
certificate:
  subject: CN=*.google.com
  issuer:  CN=GTS CA 1C3,O=Google Trust Services LLC,C=US
X509 certificate successfully verified and matches host google.com
---request begin---
GET / HTTP/1.1
Host: google.com
User-Agent: Wget/1.21.3
Accept: */*
Accept-Encoding: identity
Connection: Keep-Alive
---request end---
HTTP request sent, awaiting response...
# Rest of the output omitted
```

您可能需要发送带有特定头部的 HTTP 请求。为此，您可以使用 `--header` 选项：

```
admin@myhome:~$ wget --header='<header-name>: <header-value>' <url>
```

`POST` 是一种特殊的 HTTP 请求——它使用 URL，但也需要一些请求数据，因为它的目的是向您的系统发送数据，无论是用户名和密码还是文件。要发送带有准备数据的 HTTP `POST` 请求，可以使用 `--post-data` 选项，如下所示：

```
admin@myhome:~$ wget --post-data=<data> <url>
```

一个更强大的调试 HTTP 问题的工具是 `curl`。要发送 HTTP `GET` 请求并显示响应，您可以使用以下命令：

```
admin@myhome:~$ curl linux.com
<html>
<head><title>301 Moved Permanently</title></head>
<body>
<center><h1>301 Moved Permanently</h1></center>
<hr><center>nginx</center>
</body>
</html>
```

要发送 HTTP `POST` 请求并显示响应，您可以使用 `-X POST` 选项和 `-d` 选项：

```
admin@myhome:~$ curl -X POST -d <data> <url>
```

使用 `-X` 选项，您可以发送其他类型的请求，例如 `PATCH` 或 `DELETE`。

要发送带有特定头部的 HTTP 请求，您可以使用 `-H` 选项：

```
admin@myhome:~$ curl -H '<header-name>: <header-value>' <url>
```

若要仅显示响应头部，请使用以下代码：

```
admin@myhome:~$ curl -I linux.com
HTTP/1.1 301 Moved Permanently
Connection: keep-alive
Content-Length: 162
Content-Type: text/html
Location: https://linux.com/
Server: nginx
X-Pantheon-Styx-Hostname: styx-fe3-b-7f84d5c76-q4hpv
X-Styx-Req-Id: f1409a84-9832-11ed-abcb-7611ac88195f
Cache-Control: public, max-age=86400
Date: Fri, 20 Jan 2023 13:14:42 GMT
X-Served-By: cache-chi-kigq8000084-CHI, cache-fra-eddf8230050-FRA
X-Cache: HIT, HIT
X-Cache-Hits: 37, 1
X-Timer: S1674220483.602573,VS0,VE1
Vary: Cookie, Cookie
Age: 62475
Accept-Ranges: bytes
Via: 1.1 varnish, 1.1 varnish
```

您还可以通过使用 `-v` 选项显示请求和响应的详细信息，如下所示：

```
admin@myhome:~$ curl -v <url>
```

通过发送不同类型的请求并分析响应，您可以使用 `wget` 或 `curl` 来调试各种 HTTP 问题，如连接问题、响应错误和性能问题。像往常一样，您可以参考这两个工具的完整文档，以加深对它们使用方法的理解。

在本节中，我们介绍了 ISO/OSI 标准定义的几层网络层。每一层都标准化了网络通信不同元素的功能。接下来，我们将讨论防火墙。

# 防火墙

**防火墙** 是一种安全措施，根据预定义的规则和策略控制进出网络流量。它通常放置在受保护的网络和互联网之间，其主要目的是阻止未经授权的访问，同时允许授权的通信。防火墙可以是硬件基础的，也可以是软件基础的，且可以使用多种技术，如数据包过滤、状态检测和应用层过滤等来控制网络流量。在这一部分，我们将介绍一种适用于 Linux 系统的防火墙。

要控制 Linux 防火墙，您需要使用 `iptables`、`ufw`、`nftables` 或 `firewalld`。数据包过滤已内置在 Linux 内核中，因此这些命令行工具将与其交互。

## iptables

**iptables** 是控制防火墙的最冗长工具，这意味着它没有太多的抽象，但理解基本概念很重要，这样我们才能继续使用更用户友好的工具。

如前所述，`iptables` 允许你创建规则来过滤和操作网络数据包，并可以根据各种标准（如 IP 地址、MAC 地址、端口和协议）控制进出网络的流量。

`iptables` 使用多个概念来组织规则并将其划分为功能部分：表、链、规则和目标。最一般的概念是用表来组织规则。

我们可以使用三种表：`filter`、`nat` 和 `mangle`。`filter` 表用于过滤进出数据包，`nat` 表用于网络地址转换，`mangle` 表用于高级数据包修改。

每个表包含一组链，用于组织规则。例如，`filter` 表包含三条预定义链：`INPUT`、`OUTPUT` 和 `FORWARD`。`INPUT` 链用于接收的数据包，`OUTPUT` 链用于发送的数据包，`FORWARD` 链用于转发的数据包。

每个链包含一组规则，这些规则用于匹配数据包并决定如何处理它们。每条规则都有一个匹配条件和一个动作。例如，某条规则可能会匹配来自特定 IP 地址的数据包并将其丢弃，或者它可能会匹配发送到特定端口的数据包并允许通过。

每条规则都有一个目标，它是当规则的匹配条件满足时应执行的操作。最常见的目标是 `ACCEPT`、`DROP` 和 `REJECT`。`ACCEPT` 表示允许数据包通过防火墙，`DROP` 表示丢弃数据包而不向对端反馈任何信息，`REJECT` 表示主动拒绝数据包，使得远端知道该端口的访问已被拒绝。

`iptables` 的默认表会将规则添加到 `filter` 表中，默认情况下，每个链（`INPUT`、`OUTPUT` 和 `FORWARD`）的默认策略设置为 `ACCEPT`。你还可以创建额外的表，并将数据包定向到该表进行后续处理。通常的做法是将至少 `FORWARD` 和 `INPUT` 策略设置为 `DROP`：

```
admin@myhome:~$ sudo iptables -P INPUT DROP
admin@myhome:~$ sudo iptables -P FORWARD DROP
```

同时，我们可以允许所有回环接口访问并设置为 `ACCEPT`：

```
admin@myhome:~$ sudo iptables -A INPUT -i lo -j ACCEPT
```

此外，所有处于 `ESTABLISHED` 或 `RELATED` 状态的数据包应当被接受，否则我们将失去所有已建立的连接或正在建立中的连接：

```
admin@myhome:~$ sudo iptables -A INPUT -m conntrack --ctstate ESTABLISHED,RELATED -j ACCEPT
```

为了允许 HTTP 和 HTTPS 流量，我们可以执行以下操作：

```
admin@myhome:~$ sudo iptables -A INPUT -p tcp --dport 80 -j ACCEPT
admin@myhome:~$ sudo iptables -A INPUT -p tcp --dport 443 -j ACCEPT
```

允许 `SSH` 流量是一个好主意，这样我们可以远程登录到这台机器：

```
admin@myhome:~$ sudo iptables -A INPUT -p tcp --dport 22 -j ACCEPT
```

这里有一些 `iptables` 中常用的其他选项：

+   `-A` 或 `--append`：将规则追加到链的末尾

+   `-I` 或 `--insert`：在链中的特定位置插入一条规则

+   `-D` 或 `--delete`：从链中删除一条规则

+   `-P` 或 `--policy`：设置链的默认策略

+   `-j` 或 `--jump`：指定规则的目标

+   `-s` 或 `--source`：根据源 IP 地址或网络匹配数据包

+   `-d` 或 `--destination`：根据目标 IP 地址或网络匹配数据包

+   `-p` 或 `--protocol`：根据协议匹配数据包（例如，TCP、UDP 或 ICMP）

+   `-i` 或 `--in-interface`：根据入站接口匹配数据包

+   `-o` 或 `--out-interface`：根据出站接口匹配数据包

+   `--sport` 或 `--source-port`：根据源端口匹配数据包

+   `--dport` 或 `--destination-port`：根据目标端口匹配数据包

+   `-m` 或 `--match`：添加匹配扩展，允许你根据其他标准（如连接状态、数据包长度等）匹配数据包

在处理 `iptables` 时有许多其他功能可用，例如设置 NAT、接口绑定、TCP 多路径等。我们将在本章的 *高级主题* 部分讨论其中一些内容。

## nftables

`iptables` 以其冗长和缺乏内置抽象而闻名。

`nftables` 使用逻辑结构来组织规则，该结构包括表、链、规则和判决。表作为规则的顶级容器，并且在对规则进行分类时发挥重要作用。`nftables` 提供几种表类型：`ip`、`arp`、`ip6`、`bridge`、`inet` 和 `netdev`。

在每个表中，有不同的链，帮助进一步组织规则，按类别分为：`filter`、`route` 和 `nat`。

每个链包含单独的规则，这些规则作为匹配数据包和确定后续操作的标准。规则由匹配条件和判决组成。例如，规则可以匹配来自特定 IP 地址的数据包，并指示防火墙丢弃它们，或者它可以匹配前往特定端口的数据包，并决定接受它们。

让我们为传入和转发的数据包设置默认策略为“丢弃”（停止处理数据包并不作响应）：

```
sudo nft add rule ip filter input drop
sudo nft add rule ip filter forward drop
```

此外，通常做法是允许所有环回接口访问：

```
sudo nft add rule ip filter input iifname "lo" accept
```

为确保已建立的和相关的连接被允许，你可以运行以下命令：

```
sudo nft add rule ip filter input ct state established,related accept
```

你可以运行以下命令以允许 HTTP 和 HTTPS 流量：

```
sudo nft add rule ip filter input tcp dport {80, 443} accept
```

最后，为了启用远程访问的 SSH 流量，你可以使用以下命令：

```
sudo nft add rule ip filter input tcp dport 22 accept
```

以下是一些在 `nftables` 中常用的选项：

+   `add`：将规则追加到链的末尾

+   `insert`：将规则插入到链中的特定位置

+   `delete`：从链中删除规则

+   `chain`：指定规则的目标

+   `ip saddr`：根据源 IP 地址或网络匹配数据包

+   `ip daddr`：根据目标 IP 地址或网络匹配数据包

+   `ip protocol`：根据协议匹配数据包（例如，TCP、UDP 或 ICMP）

+   `iifname`：根据入站接口匹配数据包

+   `oifname`：根据出站接口匹配数据包

+   `tcp sport`：根据源端口匹配数据包

+   `tcp dport`：根据目标端口匹配数据包

+   `ct state`：添加一个匹配扩展，允许根据连接状态、数据包长度等其他标准匹配数据包

`nftables`被视为`iptables`的替代品，但在现代系统中，二者都常常被使用。接下来，我们将介绍一些更抽象、更易于使用的工具。

## ufw

`ufw`是 Linux`iptables`防火墙的前端，为其提供了一个简单易用的管理界面。`ufw`设计上非常易于使用，它会根据你指定的配置选项自动为你设置`iptables`规则。它比直接使用`iptables`更加用户友好，适用于常见任务。

在开始使用`ufw`之前，你需要启用它，这样你添加或移除的所有规则在系统重启后都会持久保存。只需运行以下命令：

```
admin@myhome:~$ sudo ufw enable
```

要使用`ufw`打开 TCP 端口`80`和`443`，你可以使用以下命令：

```
admin@myhome:~$ sudo ufw allow 80/tcp
admin@myhome:~$ sudo ufw allow 443/tcp
```

或者，你也可以使用一个命令同时打开这两个端口：

```
admin@myhome:~$ sudo ufw allow 80,443/tcp
```

一旦你打开了端口，你可以通过检查`ufw`的状态来验证更改：

```
admin@myhome:~$ sudo ufw status
```

`ufw`可在所有主要 Linux 发行版上使用，包括 Debian Linux、Ubuntu Linux、Arch Linux 和 Fedora Linux。但在某些情况下，你需要安装它，因为它并非系统的默认组成部分。

## firewalld

你可以使用的另一个管理 Linux 防火墙的工具是`firewalld`。这是一个旨在简化防火墙动态配置的程序。`firewalld`的一个重要特点是区域（zones），它允许你声明不同级别的信任关系，针对不同的接口和网络。它默认包含在许多流行的 Linux 发行版中，如 Red Hat Enterprise Linux、Fedora、CentOS 和 Debian。其他一些 Linux 发行版，如 Ubuntu，默认并不包含`firewalld`，但你可以在这些系统上安装并使用它。

要使用`firewalld`打开 TCP 端口`80`和`443`，你可以使用`firewall-cmd`命令行工具。以下是打开这些端口的命令：

```
admin@myhome:~$ sudo firewall-cmd --add-port=80/tcp --permanent
admin@myhome:~$ sudo firewall-cmd --add-port=443/tcp --permanent
```

你可以使用一个命令同时打开这两个端口：

```
admin@myhome:~$ sudo firewall-cmd --add-port=80/tcp --add-port=443/tcp --permanent
```

添加端口后，你需要重新加载防火墙以使更改生效：

```
admin@myhome:~$ sudo firewall-cmd --reload
```

你还可以使用以下命令检查端口的状态：

```
admin@myhome:~$ sudo firewall-cmd --list-ports
```

无论你使用什么工具来配置防火墙，设置默认规则策略为`DROP`，并只允许你希望系统处理的流量，始终是个好主意。本章篇幅有限，无法涵盖很多相关话题，但了解网络配置时的一些可能性总是有帮助的。

# 高级话题

在本节中，我们将介绍网络功能的更高级用法。某些功能非常常见（如端口转发或 NAT），而有些则不太为人所知。我们将从最常见的功能开始，它们是你最有可能经常遇到的，然后再介绍一些更高级和不太为人知的功能。

## NAT

**网络地址转换**（**NAT**）是一种将一个网络映射到另一个网络的技术。其最初的目的在于简化整个网络段的路由，而无需更改数据包中每个主机的地址。

**源地址转换**（**SNAT**）是一种改变数据包源 IP 地址的 NAT 类型。它用于允许私有网络上的主机通过单个公共 IP 地址访问互联网。

**目标地址转换**（**DNAT**）是一种改变数据包目标 IP 地址的 NAT 类型。它用于根据目标 IP 地址将传入的流量转发到特定的内部主机。通常用于允许外部客户端通过公共 IP 地址访问内部主机上运行的服务。

要使用`iptables`设置 NAT，你可以使用以下基本命令：

```
admin@myhome:~$ echo 1 > /proc/sys/net/ipv4/ip_forward
```

这将启用转发，使得你的机器能够将数据包从一个接口转发到另一个接口。这个特定的命令通过使用`proc`文件系统动态更新 Linux 内核配置。你也可以使用`sysctl`命令实现相同的功能：

```
admin@myhome:~$ sudo sysctl -w net.ipv4.ip_forward=1
```

要为本地网络如`192.168.10.0/24`配置 NAT，你需要以`root`用户身份运行以下命令：

```
admin@myhome:~$ sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
admin@myhome:~$ sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
admin@myhome:~$ sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
```

请注意，`eth0`是连接到互联网的接口，`eth1`是连接到内部网络的接口。

`MASQUERADE`目标用于在 Linux 路由器上实现 NAT。当数据包通过路由器并发送到互联网时，`MASQUERADE`目标会将数据包的源地址更改为路由器的公共 IP 地址。这使得内部网络中的设备能够使用路由器的公共 IP 地址作为源地址与互联网中的设备通信，有效地将内部网络隐藏于互联网之外。

`MASQUERADE`目标通常用于`nat`表的`POSTROUTING`链，通常应用于连接到互联网的接口。它仅适用于动态分配的 IP 地址（DHCP），通常用于家庭路由器的情况。

## 端口转发

**端口转发**是一种将网络流量从一个网络地址和端口定向到另一个地址和端口的技术。这对于将传入流量定向到计算机或网络设备上运行的特定服务或应用程序非常有用。这有助于从远程位置访问私有网络上的服务或应用程序，或者使私有网络上运行的服务或应用程序对外界可访问。

本质上，它是 NAT 的另一种用途，因为你将更改到达你机器的数据包的目标 IP（以及端口）。

要将进入`eth0`接口的 TCP 端口`80`的数据包转发到内部 IP`192.168.10.101`的端口`8080`，可以使用以下命令：

```
admin@myhome:~$ sudo iptables -t nat -A PREROUTING -i eth0 -p tcp --dport 80 -j DNAT --to-destination 192.168.10.101:8080
admin@myhome:~$ sudo iptables -t nat -A POSTROUTING -j MASQUERADE
```

这里我们需要使用`MASQUERADE`，因为我们希望隐藏内部 IP 地址。

## 接口绑定

`active-backup`、`balance-rr` 和 `802.3ad`。`active-backup`模式意味着当主设备发生故障时，两个绑定接口中的一个作为备用。`balance-rr`将同时使用两个接口，并采用轮询策略。`802.3ad`绑定创建共享相同技术规格（速度和双工设置）的聚合组。您可以在官方 Linux 内核网站上阅读更多关于模式和绑定设置的信息：[`www.kernel.org/doc/Documentation/networking/bonding.txt`](https://www.kernel.org/doc/Documentation/networking/bonding.txt)

## TCP 多路径

**TCP 多路径**指的是使用多个路径在网络中两个点之间发送和接收数据，而不是仅使用一个路径。这可以通过允许当一个路径不可用时进行故障切换，并通过在多个路径之间进行负载均衡，从而提高网络的可靠性和性能。可以通过多种技术实现这一点，例如使用设备上的多个接口或通过网络使用多个路由路径。

使用`iproute2`软件包配置多路径非常简单。要在 Linux 上使用`eth0`和`eth1`接口配置多路径，您需要运行以下命令：

```
admin@myhome:~$ ip route add default scope global nexthop via 192.168.1.1 dev eth0 nexthop via 192.168.2.1 dev eth1 weight 1
```

此命令创建了一个新的默认路由，使用`eth0`和`eth1`接口，并设置权重为`1`。在此示例中使用的 IP 地址（`192.168.1.1`和`192.168.2.1`）应替换为`eth0`和`eth1`接口上下一跳路由器的实际 IP 地址。

当运行`ip route show`时，您将看到一个新的多路径路由。要开始使用它，您需要更改默认路由：

```
admin@myhome:~$ ip route change default scope global nexthop via 192.168.1.1 dev eth0 nexthop via 192.168.2.1 dev eth1 weight 1
```

您可以在[`www.multipath-tcp.org/`](https://www.multipath-tcp.org/)上阅读更多关于多路径及其使用方法的信息。

## BGP

**边界网关协议** (**BGP**) 是一种路由协议，用于在单个**自治系统** (**AS**) 内部或多个自治系统之间分发路由信息。BGP 用于在互联网骨干路由器以及企业网络中构建路由表。

BGP 路由器与其邻居交换路由信息，这些邻居可以是同一自治系统或不同自治系统中的其他 BGP 路由器。当 BGP 路由器从其邻居接收路由信息时，它使用一套规则和策略来决定将哪些路由添加到其路由表中并广告给邻居。这使得 BGP 能够支持多个通向目标的路径，并根据距离、成本或优先级等各种因素选择最佳路径。

BGP 被认为是一种路径向量协议，因为它交换关于通往目标的完整路径的信息，而不仅仅是下一个跳点。这使得 BGP 能够支持高级功能，如路由策略和流量工程。

在 Linux 机器上使用 BGP 有多种方式，具体取决于您的使用场景和网络环境。

**BIRD** 是一款支持 BGP 和其他路由协议的 Linux 路由守护进程。你可以配置 BIRD，使其充当 BGP 发言人并与其他 BGP 路由器交换路由信息。BIRD 可以安装在大多数 Linux 发行版上，并可以通过简单的配置文件进行配置。

**Quagga** 是另一款支持 BGP、OSPF 和其他路由协议的 Linux 开源路由软件套件。你可以配置 Quagga 使其充当 BGP 发言人并与其他 BGP 路由器交换路由信息。Quagga 可以安装在大多数 Linux 发行版上，并可以通过命令行接口或配置文件进行配置。

**Free Range Routing**（**FRR**）是一款支持 BGP、OSPF 和其他路由协议的 Linux 路由软件套件；它是 Quagga 的一个分支。

你可以在以下的 Linux Journal 文章中阅读更多关于 BGP 和使用 BIRD 的内容：[`www.linuxjournal.com/content/linux-advanced-routing-tutorial`](https://www.linuxjournal.com/content/linux-advanced-routing-tutorial)。

# 总结

本章介绍了你在 DevOps 团队工作中可能会遇到的一些基本网络主题。这是一个起点和基础，帮助你理解处理容器内部运行的服务时与网络相关的主题。你可能还会想通过阅读关于 IPv6 协议的资料来扩展你的知识，IPv6 尚未完全取代 IPv4。

在下一章中，我们将重点介绍现代组织中主要使用的**版本控制系统**（**VCS**）：**Git**。
