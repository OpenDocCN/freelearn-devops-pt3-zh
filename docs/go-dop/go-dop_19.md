# 第十六章：为混乱而设计

编写在完美条件下工作的程序是容易的。如果你永远不需要担心网络延迟、服务超时、存储故障、应用程序行为异常、用户发送错误参数、安全问题，或者我们在现实生活中遇到的任何其他场景，那就好了。

根据我的经验，故障通常有以下三种方式：

+   立即

+   渐进地

+   壮观地

**立即**通常是应用代码发生变化，导致服务在启动时或接收到请求时崩溃的结果。大多数开发测试环境或金丝雀发布能够在生产中发生任何实际问题之前捕捉到这些。这种类型的问题通常很容易修复和预防。

**渐进性**通常是由于某种类型的内存泄漏、线程/协程泄漏，或忽视设计限制。这些问题随着时间的推移积累，开始引发问题，导致服务崩溃或延迟增长到无法接受的水平。很多时候，一旦问题被识别出来，这些问题可以在金丝雀发布过程中轻松解决。对于设计问题，修复可能需要几个月的密集工作来解决。某些罕见版本的这种问题，会出现我所称之为“悬崖故障”：渐进性增长遇到一个无法通过增加更多资源来克服的限制。这类问题属于下一个类别。

那种类别是**壮观的**。这就是你在生产环境中发现一个问题，导致大规模故障，而几分钟前一切都正常工作。手机到处响起警报，仪表盘变红，狗和猫开始一起生活——大规模的恐慌！这可能是一个有缺陷的服务上线，压垮了你的网络，依赖的缓存服务崩溃，或是某种查询导致你的服务崩溃。这些停机造成大规模恐慌，考验你在团队之间有效沟通的能力，且通常会出现在新闻报道中。

本章将重点讨论如何设计能够应对混乱的基础设施工具。大型云公司最壮观的故障往往是基础设施工具的结果，从**Google 站点可靠性工程**（**Google SRE**）擦除他们集群卫星上的所有磁盘，到**亚马逊云服务**（**AWS**）用基础设施工具**远程过程调用**（**RPCs**）压垮其网络。

本章将探讨**第一响应者**（**FRs**）如何停止自动化的安全方法，如何编写幂等的工作流工具，失败的 RPC 的增量回退包，推出时的节奏限制器等内容。

为此，我们将介绍一些概念和包，这些包将构建到一个通用的工作流系统中，供你进一步学习使用。该系统能够接受请求来执行某种工作，验证参数是否正确，按照一组策略验证请求，然后执行该工作。

在此模型中，客户端（可以是**命令行界面**（**CLI**）应用程序或服务）通过协议缓冲区详细描述要执行的工作，并将其发送到服务器。工作流系统执行所有实际工作。

本章将涵盖以下主要主题：

+   使用过载防护机制

+   使用速率限制器防止工作流失控

+   构建可重复且不会丢失的工作流

+   使用策略限制工具

+   构建具有紧急停止功能的系统

# 技术要求

本章有与前几章相同的要求，只是增加了访问以下 GitHub 仓库的需求：[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/16/workflow`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/16/workflow)。

话虽如此，让我们进入第一章，讨论如何使用过载防护机制，在出现问题时保持网络和服务的健康。

# 使用过载防护机制

当你拥有一小组服务时，应用程序异常通常只会造成小问题。这是因为数据中心内通常有过剩的网络容量来吸收异常行为的应用程序，并且在服务数量较少的情况下，通常可以直观地找出问题的根源。

当你运行大量应用程序时，通常网络和机器会出现超载现象。**超载**意味着你的网络和系统无法处理所有应用程序在 100% 运行的情况。超载在网络或集群中非常常见，用来控制成本。之所以可行，是因为在任何给定时刻，大多数应用程序的流量、**中央处理单元**（**CPU**）和内存都会随着网络流量的波动而波动。

如果应用程序突然遇到某种类型的错误，可能会进入**重试循环**，迅速使服务崩溃。此外，如果发生了某种灾难性事件，导致服务下线，尝试将应用程序重新上线可能会因为所有客户端的请求排队而导致服务崩溃。

更糟糕的是网络可能发生的情况。如果网络被压垮，或者云设备的**每秒查询数**（**QPS**）被超出，其他应用程序的流量可能会受到负面影响。这可能掩盖问题的真正原因。

防止这些类型问题的方式有几种，最常见的两种方法如下：

+   电路断路器

+   回退实现

这些预防机制的思路是相同的：当发生故障时，防止重试请求压垮服务。

基础设施服务通常是这些防护机制的一个被忽视的应用场景。我们很多时候关注的是公共服务，但基础设施服务同样重要。如果该服务是关键服务且被压垮，恢复它可能非常困难，除非手动调整其他服务以减轻负载。

让我们来看一下其中一种更流行的方法：**断路器**。

## 案例研究 – AWS 客户请求压垮了网络

当一个行为不当的应用开始在客户网络与其核心网络之间的网络边界上发送过多流量时，AWS 发生了全球性故障，影响了全球的 AWS 客户。虽然这次故障仅限于其`us-east-1`区域，但多个地点的客户都受到了影响。

问题有两个方面，包含以下因素：

+   一个行为不当的应用发送了过多的请求。

+   它们的客户端在故障时没有退避。

正是第二个问题导致了长时间的故障。AWS 在使用标准客户端进行 RPC 时做得是正确的，当请求失败时，会进行递增的退避。然而，由于某种原因，在这个案例中，客户端库没有按预期表现。

这意味着，当终端被压垮时，负载并没有自我减少，而是进入了某种类型的无限循环，持续增加受影响系统的负载并压垮了它们的网络交叉连接。这种交叉连接的压垮禁用了它们的监控，并使得他们无法看到问题的根源。结果是，他们不得不通过缩减应用流量来尝试减少网络负载，同时尽量不影响仍在正常工作的客户服务——这是一项我不愿意面对的任务。

这个案例突显了在故障发生时防止应用重试的重要性。如需阅读更多关于此方面的内容，请访问以下网页：[`aws.amazon.com/message/12721/`](https://aws.amazon.com/message/12721/)。

## 使用断路器

断路器的工作原理是将 RPC 调用包装在一个客户端中，一旦达到阈值，任何尝试都会自动失败。然后，所有的调用都会返回失败，而不会实际尝试，持续一段时间。

断路器有三种模式，如下：

+   闭合

+   打开

+   半开

当一切正常时，断路器处于**闭合**状态。这是正常状态。

当一些故障导致断路器跳闸时，断路器处于**打开**状态。在此状态下，所有请求都会自动失败，而无需尝试发送消息。此状态持续一段时间。建议这段时间设置为一定的时长，并加入一些随机性，以防止自发的同步。

断路器在处于打开状态一段时间后，会进入**半开**状态。一旦进入半开状态，部分请求会被实际尝试。如果超过某个成功阈值，断路器会重新进入**闭合**状态。如果没有，断路器会再次进入**打开**状态。

你可以找到几种不同的 Go 语言断路器实现，但其中一个最受欢迎的是索尼开发的，叫做**gobreaker**（[`github.com/sony/gobreaker`](https://github.com/sony/gobreaker)）。

让我们来看一下如何使用它来限制**HTTP**查询的重试，如下所示：

```
type HTTP struct {
     client *http.Client
     cb     *gobreaker.CircuitBreaker
}
func New(client *http.Client) *HTTP {
     return &HTTP{
          client: client,
          cb: gobreaker.NewCircuitBreaker(
               gobreaker.Settings{
                    MaxRequests: 1,
                    Interval:    30 * time.Second,
                    Timeout:     10 * time.Second,
                    ReadyToTrip: func(c gobreaker.Counts) bool {
                         return c.ConsecutiveFailures > 5
                    },
               },
          ),
     }
}
func (h *HTTP) Get(req *http.Request) (*http.Response, error) {
     if _, ok := req.Context().Deadline(); !ok {
          return nil, fmt.Errorf("all requests must have a Context deadline set")
     }
     r, err := h.cb.Execute(
          func() (interface{}, error) {
               resp, err := h.client.Do(req)
               if resp.StatusCode != 200 {
                    return nil, fmt.Errorf("non-200 response code")
               }
               return resp, err
          },
     )
     if err != nil {
          return nil, err
     }
     return r.(*http.Response), nil
}
```

上面的代码定义了以下内容：

+   一种包含这两者的 HTTP 类型：

    +   用于发送 HTTP 请求的`http.Client`

    +   一个用于 HTTP 请求的断路器

+   为我们的`HTTP`类型创建一个`New()`构造函数。它创建一个断路器，带有强制执行以下内容的设置：

    +   在半开放状态时每次允许一个请求

    +   在关闭状态后，我们将进入一个 30 秒的半开放状态

    +   有一个持续 10 秒的关闭状态

    +   如果连续五次失败，则进入关闭状态

    +   `HTTP`上的`Get()`方法执行以下操作：

    +   检查`*http.Request`是否定义了超时

    +   调用我们`client.Do()`方法上的断路器

    +   将返回的`interface{}`转换为底层的`*http.Response`

这段代码给我们提供了一个强大的 HTTP 客户端，包装了一个断路器。这个更好的版本可能会将设置传递给构造函数，但我希望它为示例打包得更加简洁。

如果你想看到断路器实际运行的演示，可以在这里看到：

[`go.dev/play/p/qpG_l3OE-bu`](https://go.dev/play/p/qpG_l3OE-bu)

## 使用回退实现

**回退实现**包装了 RPC 客户端，客户端将在尝试之间进行重试，并且每次重试之间都会有一段暂停时间。这些暂停时间会越来越长，直到达到某个最大值。

回退实现可以有多种计算时间段的方法。在本章中，我们将集中讨论指数回退。

指数回退简单地在每次尝试中增加延迟，这些延迟会随着失败次数的增加而指数增长。与断路器一样，有许多包提供回退实现。在这个例子中，我们将使用[`pkg.go.dev/github.com/cenk/backoff`](https://pkg.go.dev/github.com/cenk/backoff)，这是谷歌 HTTP 回退库的一个实现，适用于 Java。

这个回退实现提供了许多谷歌在多年研究服务失败中发现有用的重要特性。库中最重要的特性之一是向重试之间的睡眠时间添加随机值，这可以防止多个客户端同步它们的重试操作。

其他重要特性包括能够尊重上下文取消操作并提供最大重试次数。

让我们来看一下如何使用它来限制 HTTP 查询的重试，如下所示：

```
type HTTP struct {
     client *http.Client
}
func New(client *http.Client) *HTTP {
     return &HTTP{
          client: client,
     }
}
func (h *HTTP) Get(req *http.Request) (*http.Response, error) {
     if _, ok := req.Context().Deadline(); !ok {
          return nil, fmt.Errorf("all requests must have a Context deadline set")
     }
     var resp *http.Response
     op := func() error {
          var err error
          resp, err = h.client.Do(req)
          if err != nil {
               return err
          }
          if resp.StatusCode != 200 {
               return fmt.Errorf("non-200 response code")
          }
          return nil
     }
     err := backoff.Retry(
          op, 
          backoff.WithContext(
               backoff.NewExponentialBackOff(),
               req.Context(),
          ),
     )
     if err != nil {
          return nil, err
     }
     return resp, nil
}
```

上面的代码定义了以下内容：

+   一种包含这两者的 HTTP 类型：

    +   用于发送 HTTP 请求的`http.Client`

    +   一个用于 HTTP 请求的指数回退

+   为我们的`HTTP`类型创建一个`New()`构造函数

+   `HTTP`上的`Get()`方法

+   它还做了以下事情：

    +   创建一个`func()`错误，尝试我们的请求，名为`op`

    +   以重试和指数延迟的方式运行`op`

    +   创建一个具有默认值的指数回退

    +   将该回退包装在`BackOffContext`中，以尊重我们的上下文截止时间

对于`ExponentialBackoff`的默认值列表，请参见以下网页：

[`pkg.go.dev/github.com/cenkalti/backoff?utm_source=godoc#ExponentialBackOff`](https://pkg.go.dev/github.com/cenkalti/backoff?utm_source=godoc#ExponentialBackOff)

如果你想看到这个退避机制的实际演示，你可以在这里查看：

[`go.dev/play/p/30tetefu9t0`](https://go.dev/play/p/30tetefu9t0)

## 将电路断路器与退避机制结合

在选择预防实现时，另一种选择是将电路断路器与退避机制结合起来，以实现更强大的实现。

退避实现可以设置最大重试时间。将其封装在电路断路器内，使一组失败的尝试触发我们的电路断路器，不仅可以通过减缓请求来潜在地减少我们的负载，还可以通过电路断路器停止这些尝试。

如果你想看到一个结合这两者的实现，你可以访问以下网页：

[`go.dev/play/p/gERsR7fvDck`](https://go.dev/play/p/gERsR7fvDck)

在本节中，我们讨论了防止网络和服务过载的机制的必要性。我们还讨论了一个 AWS 宕机事件，部分原因是由于这些机制的失败。你已了解了电路断路器和退避机制，以防止此类故障的发生。最后，我们展示了两个常用的包来实现这些机制，并附带了示例。

在我们的工作流引擎中，我们将为**Google RPC**（**gRPC**）客户端实现这些预防机制，以防止与服务器通信时出现问题。你可以在这里看到：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/client/client.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/client/client.go)

在我们的下一节中，我们将研究如何使用限流器防止工作流执行得太快。为工作流的操作强制执行节奏，并防止同一类型的工作流同时执行过多，这是非常重要的。

# 使用限流器来防止失控的工作流

**DevOps 工程师**可能负责由数十个微服务组成的服务。这些微服务可能会在全球的数据中心中运行成千上万个实例。一旦一个服务包含多个实例，就需要某种形式的速率控制，以防止不良的发布或配置更改导致大规模破坏。

一种**限流器**，用于带有强制暂停间隔的工作，对于防止失控的基础设施变更至关重要。

限流容易实现，但限流器的作用范围将取决于你的工作流所做的事情。对于服务，你可能希望一次只发生一种类型的变化，或者一次只影响一些实例。

第一种速率限制方式是防止同一类型的工作流同时运行。例如，你可能希望每次只能进行一个卫星磁盘擦除操作。

第二种方式是限制能够同时受影响的设备、服务等的数量。例如，你可能只希望允许同一区域中的两个路由器进行固件升级。

为了使速率限制器有效，拥有一个执行一组服务操作的单一系统可以大大简化这些工作。这使得可以集中执行速率限制等政策。

让我们来看看 Go 中使用通道实现的最简单速率限制器。

## 案例研究——谷歌卫星磁盘擦除

在早期，谷歌并不拥有如今所有的数据中心空间——我们租用了大量空间并使用了大量机器。然而在一些地方，这样的成本非常高。为了加速这些地方的连接速度，我们会租用小型空间，这些地方可以放置缓存机器，终止 HTTP 连接并将流量回传到数据中心。我们称这些地方为**卫星**。

谷歌有一个自动化的机器退役流程，其中一部分就是磁盘擦除，机器的磁盘会被清空。

该软件是用来获取卫星机器列表并过滤掉其他机器的。不幸的是，如果你在一个卫星上运行它两次，过滤器将不会生效，你的机器列表将会包含每个卫星中的所有机器。

磁盘擦除非常高效，在所有卫星中的所有机器都被同时加入磁盘擦除队列，直到操作完成。

如果你需要更详细的分析，可以阅读[`sre.google/workbook/postmortem-culture/`](https://sre.google/workbook/postmortem-culture/)，在那里，几位**站点可靠性工程师**(**SREs**)提供了更多关于事后分析的细节。

我们可以查看代码中的过滤部分并讨论糟糕的设计，但总会有编写不良的工具和错误的输入。即使你当前有一个良好的代码审查文化，也总会有疏漏。在工程师快速增长的时期，这类问题可能会露出丑陋的面目。

一些在少数经验丰富的工程师手中已知的危险工具，在新工程师手中可能会很安全使用，但没有经验的工程师或缺乏适当警觉的工程师，可能会迅速摧毁你的基础设施。

在这种情况以及许多其他情况下，集中执行并配合速率限制和其他强制性安全机制，可以让新手编写可能危险但影响范围有限的工具。

## 基于通道的速率限制器

**基于通道的速率限制器**在一个程序处理自动化任务时非常有用。在这种情况下，你可以创建一个基于通道大小的限制器。让我们来实现一个只允许在同一时间处理固定数量项目的限制器，如下所示：

```
limit := make(chan struct{}, 3)
```

我们现在有了一个可以限制可处理项目数量的工具。

让我们定义一个简单的类型，表示要执行的某些操作，如下所示：

```
type Job interface {
     Validate(job *pb.Job) error
     Run(ctx context.Context, job *pb.Job) error
}
```

这定义了一个可以执行以下操作的`Job`：

+   验证传递给我们的`pb.Job`定义

+   使用该定义运行任务

这是一个非常简单的示例，展示了如何执行一组包含在名为“块”的容器中的任务，块只是一个`Job`切片的容器：

```
go
wg := sync.WaitGroup{}
for _, block := range work.Blocks {
     limit := make(chan struct{}, req.Limit)
     for _, job := range block.Jobs {
          job := job
          limit <- struct{}{}
          wg.Add()
          go func() {
               defer wg.Done()
               defer func() {
                    <-limit
               }()
               job()
          }()
     }
}
wg.Wait()
```

在上面的代码片段中，发生了以下事情：

+   我们循环遍历`work.Blocks`变量中的`Block`切片。

+   我们循环遍历`block.Jobs`变量中的`Jobs`切片。

+   如果我们已经有`req.limit`个项目在运行，`limit <- struct{}{}`将会阻塞。

+   它并发执行我们的任务。

+   当我们的 goroutine 结束时，我们从`workLimit`队列中移除一个项目。

+   我们等待所有 goroutine 结束。

这段代码防止同时处理超过`req.limit`个项目。如果这是一个服务器，你可以将`limit`设为所有用户共享的变量，并防止系统中同时发生超过三个项目的工作。或者，你可以为不同类别的工作设置不同的限制器。

关于`job := job`部分的说明。它正在创建一个`job`的遮蔽变量。这可以防止`job`变量在 goroutine 内被更改，避免在循环和 goroutine 并行运行时修改原变量，而是将变量的副本放在 goroutine 相同作用域内。这是 Go 新手常见的并发错误，通常被称为**for 循环陷阱**。这是一个你可以使用的游乐场，用来理解为什么这是必要的：[`go.dev/play/p/O9DcUIKuGBv`](https://go.dev/play/p/O9DcUIKuGBv)。

我们已在游乐场完成了以下示例，您可以在其中操作以探索这些概念：

[`go.dev/play/p/aYoCTEFvRBI`](https://go.dev/play/p/aYoCTEFvRBI)

你可以在`runJobs()`方法中的工作流服务中看到基于通道的速率限制器的实际应用：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/service/executor/executor.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/service/executor/executor.go)

## 令牌桶速率限制器

**令牌桶**通常用于为服务提供可突发的流量管理。令牌桶有几种类型，最常见的是标准令牌桶和泄漏令牌桶。

这些通常不用于基础设施工具的部署，因为客户端通常是内部的，且比面向外部的服务更可预测，但一种有用的令牌桶类型可以用于提供速率控制。标准令牌桶只是保存一些固定的令牌集，这些令牌会在某个时间间隔后重新填充。

这是一个示例：

```
type bucket struct {
     tokens chan struct{}
}
func newbucket(size, incr int, interval time.Duration) (*bucket, error) {
     b := bucket{tokens: make(chan struct{}, size)}
     go func() {
          for _ = range time.Tick(interval) {
               for i := 0; i < incr; i++ {
                    select{
                    case <-b.tokens:
                         continue
                    default:
                    }
                    break
               }
          }
     }()
     return &b, nil
}
func (b *bucket) token(ctx context.Context) error {
     select {
     case <-ctx.Done():
          return ctx.Err()
     case b.tokens <-struct{}{}:
     }
     return nil
}
```

上面的代码片段执行了以下操作：

+   定义一个`bucket`类型来保存我们的令牌

+   有`newBucket()`，它创建一个带有以下属性的新`bucket`实例：

+   `size`，即可以存储的令牌总数

    +   `incr`，即每次添加的令牌数量

    +   `interval`，即向桶中添加的时间间隔

它还执行以下操作：

+   启动一个 goroutine，以间隔填充桶

+   只会填充到最大`size`值

+   定义了`token()`，它用来获取一个令牌：

    +   如果没有可用的令牌，我们将等待一个。

    +   如果`Context`被取消，我们将返回一个错误。

这是一个相当稳健的标准令牌桶实现。你可能能使用`atomic`包实现一个更快速的版本，但这样做会更复杂。

一个带有输入检查并且能够停止通过`newBucket()`创建的 goroutine 的实现可以在这里找到：

[`go.dev/play/p/6Dihz2lUH-P`](https://go.dev/play/p/6Dihz2lUH-P)

如果需要，我们可以使用令牌桶来限制执行的速率，只允许按照我们定义的速率执行。这可以用于限制某个操作的执行速度，或者在某段时间内仅允许一定数量的工作流实例发生。我们将在下一节中使用它来限制某个特定工作流的执行时机。

我们的通用工作流系统在这里有一个令牌桶包：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/token/token.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/token/token.go)

在这一节中，我们探讨了如何使用速率限制器来防止工作流失控。我们以谷歌的卫星磁盘擦除为案例研究，讨论了这一类事件。我们展示了如何实现基于通道的速率限制器，以控制并发操作。我们还讨论了如何使用令牌桶来限制一定时间内的执行次数。

本节还为我们构建的工作流系统示例中，定义为作业的操作执行奠定了基础。

现在我们有了一些关于如何限制操作速率的思路，接下来我们来看看如何开发可重复且不会被客户端丢失的工作流。

# 构建可重复且永不丢失的工作流

作为 DevOps 工程师，我们经常编写工具。在小型公司中，很多时候这些工具是脚本集合。而在大型公司中，这些工具通常是复杂的系统。

正如你从前言中可能已经看出来的那样，我认为工具的执行应该始终发生在一个集中式的服务中，无论规模大小。一个基础服务很容易编写，你可以随着新需求的出现进行扩展和替换。

但是，要使工作流服务正常工作，你创建的工作流必须满足以下两个关键概念：

+   它们必须是可重复的。

+   它们不能丢失。

第一个概念是，在相同基础设施上运行工作流多次应该产生相同的结果。我们称之为**幂等性**，借用了计算机科学中的术语。

第二点是，工作流不能丢失。如果一个工具创建了一个要由系统执行的工作流，而该工具崩溃了，那么该工具必须能够知道工作流正在运行并恢复监控。

## 构建幂等工作流

**幂等性**是一个概念，如果你使用相同的参数多次调用，你将得到相同的结果。这是编写某些类型的软件时非常重要的概念。

在基础设施中，我们稍微修改了这个定义：幂等操作是指如果使用相同的参数重复执行，并且在此调用之外的基础设施没有变化，它将返回相同的结果。

幂等性是使工作流在工作流系统崩溃时仍能恢复的关键。简单的工作流系统可以直接重复整个工作流。更复杂的系统可以从中断的位置重新启动。

许多时候，开发人员没有深入考虑幂等性。例如，让我们来看一个简单的操作，将一些内容复制到一个文件。这里是一个简单的实现：

```
func CopyToFile(content []byte, p string) error {
     return io.WriteFile(p, content)
}
```

上述代码包含以下内容：

+   一个表示文件内容的`content`参数

+   一个`p`参数，表示文件的路径

它还执行以下操作：

+   将`content`写入路径为`p`的文件

这看起来最初是幂等的。如果我们的工作流在调用`CopyToFile()`之后但在调用`io.WriteFile()`之前被杀死，我们可以重复这个操作，最初看起来如果我们调用两次，我们仍然会得到相同的结果。

但是如果文件不存在，我们创建了它，但是没有权限编辑现有文件呢？如果我们的程序在记录`io.WriteFile()`的结果之前崩溃了，但在更改已经发生之后，重复此操作会报告错误，因为基础设施没有发生变化，因此该操作不是幂等的。

让我们修改这个代码，使其具备幂等性，如下所示：

```
func CopyToFile(content []byte, p string) error {
     if _, ok := os.Stat(p); ok {
          f, err := os.Open(p)
          if err != nil {
               return err
          }
          h0 := sha256.New()
          io.Copy(h0, f)
          h1 := sha256.New()
          h1.Write(content)
          if h0.Sum(nil) == h1.Sum(nil) {
               return nil
          }
     }
     return io.WriteFile(p, content)
}
```

这段代码检查文件是否存在，然后执行以下操作：

+   如果文件已存在并且已有内容，它不做任何操作。

+   如果没有，它会写入内容。

这使用标准库的`sha256`包来计算校验和哈希值，以验证内容是否相同。

提供幂等性的关键通常只是检查工作是否已经完成。

这引出了一个叫做三次握手的概念。这个概念可以在需要通过 RPC 与其他系统交互时提供幂等性。我们将讨论如何在执行工作流时使用这一概念，但它也可以用于与其他服务交互的幂等操作。

## 使用三次握手防止工作流丢失

当我们编写一个与工作流服务交互的应用程序时，重要的是该应用程序永远不能失去对我们服务上运行的工作流的追踪。

三次握手是我从**传输控制协议**（**TCP**）借用的名称。TCP 有一个握手过程，用来在两台机器之间建立一个套接字。它包括以下内容：

+   **SYNchonize**（**SYN**），请求打开连接

+   **ACKnowledge**（**ACK**），对请求的确认

+   SYN-ACK，对 ACK 的确认

当客户端发送请求执行工作流时，我们不希望工作流服务执行一个客户端因为崩溃而不知道存在的工作流。

这种情况可能发生在客户端程序崩溃或客户端运行的机器发生故障时。如果我们发送了一个工作流，并且服务在一个单一 RPC 后开始执行，客户端可能在发送 RPC 后但在收到工作流**标识符**（**ID**）之前崩溃。

这将导致一种情况，当客户端重启时，它不知道工作流服务已经在运行该工作流，并且可能会发送另一个工作流，执行相同的操作。

为了避免这种情况，工作流应有一个三次握手，而不是单个 RPC 来执行工作流，三次握手的过程包括以下内容：

+   将工作流发送到服务

+   接收工作流 ID

+   向服务发送请求，执行带有 ID 的工作流

这允许客户端在执行之前记录工作流的 ID。如果客户端在记录 ID 之前崩溃，服务将只拥有一个未运行的工作流记录。如果客户端在服务开始执行后崩溃，当客户端重启时，它可以检查工作流的状态。如果正在运行，它可以简单地监控。如果没有运行，它可以请求再次执行。

对于我们的工作流服务，让我们创建一个支持三次握手的服务定义，使用 gRPC，如下所示：

```
service Workflow {
     rpc Submit(WorkReq) returns (WorkResp) {};
     rpc Exec(ExecReq) returns (ExecResp) {};
     rpc Status(StatusReq) returns (StatusResp) {};
}
```

这定义了一个包含以下调用的服务：

+   `Submit`提交一个描述待处理工作的`WorkReq`消息。

+   `Exec`执行之前通过`Submit`发送到服务器的`WorkReq`。

+   `Status`检索`WorkReq`的状态。

这些服务调用的消息内容将在下一节中详细讨论，但关键是，在`Submit()`时，`WorkResp`将返回一个 ID，但工作流不会执行。当调用`Exec()`时，我们将发送从`Submit()`调用中收到的 ID，而`Status()`调用让我们能够检查任何工作流的状态。

我们现在有了工作流服务的基本定义，包括一个三次握手，防止我们的客户端丢失任何工作流。

在本节中，我们已经涵盖了不可丢失的可重复工作流的基础知识。我们讲解了幂等性以及它如何导致可重复的工作流。我们还展示了三次握手如何帮助我们防止正在运行的工作流变得*丢失*。

我们还定义了将在我们构建的工作流系统中使用的服务调用。

现在，我们希望了解工具如何理解正在执行的**工作范围**（**SOW**），以提供防止工具失控的保护。为此，让我们探索构建一个策略引擎。

# 使用策略来限制工具

限速对于防止一个坏的工具运行摧毁一个服务很有效，尤其是当所有工作项相等时。但并非所有工作项都是相等的，因为一些机器服务比其他服务更为重要和脆弱（例如，服务的数据库系统）。此外，机器或服务可能需要按逻辑分组，只能在某些有限的数量中进行。可以按站点、地理区域等进行划分。

该逻辑通常是特定于某一组工作项的。这种打包，我们称之为 SOW，可能会非常复杂。

要安全地执行工作，必须理解你的工作范围。这可能是如何安全地更新特定服务的数据库架构，或一个网络区域中一次可以修改多少个路由反射器。

为了在 SOW（工作说明书）中实现安全性，我们将引入策略的概念。策略将用于检查进入系统的一组工作是否符合合规性要求。如果不符合，它将被拒绝。

例如，我们将查看类似于谷歌磁盘擦除案例的磁盘擦除处理。这里是我们将添加的一些保护措施：

+   每小时只允许进行一次卫星磁盘擦除

+   限速，以便我们一次只能擦除五台机器

+   每执行五台机器的擦除后，必须暂停 1 分钟

为了能够构建一个策略引擎，我们必须有一种通用的方式来定义将要执行的工作类型、执行顺序以及并发度。

我们还希望工具工程师仅定义要执行的工作，并将其提交给一个单独的服务来执行。这样可以实现控制的集中化。

让我们定义一个可以在 gRPC 中执行的服务。

## 定义 gRPC 工作流服务

在前面的章节中，我们讨论了一个定义三次握手的服务定义。让我们看看这些调用的参数，以了解我们的客户端将向工作流服务发送什么，如下所示：

```
message WorkReq {
     string name = 1;
     string desc = 2;
     repeated Block blocks = 3;
}
message WorkResp {
     string id = 1;
}
message Block {
     string desc = 1;
     int32 rate_limit = 2;
     repeated Job jobs = 3;
}
message Job {
     string name = 1;
     map<string, string> args = 2;
}
```

这些消息用于定义客户端希望服务器执行的工作，并包含以下属性：

+   `WorkReq`消息包含工作名称和组成工作流的所有`Block`消息。

+   `Block`消息描述工作流中的一项工作；每个`Block`一次执行一个，并具有以下属性：

    +   有一组`Job`消息，描述要执行的工作

    +   执行`Job`消息描述的工作的并发度

    +   `Job`消息描述服务器上的工作类型，调用时使用的参数。

+   `WorkResp`消息返回与该`WorkReq`相关的 ID：

    +   使用`UUIDv1` ID，封装时间信息到 ID 中，以便我们知道它何时提交到系统

    +   使用时间机制防止在某个过期时间之前没有调用`Exec()` `RPC`时执行

`Exec`消息提供你要执行的 ID，如下所示：

```
message ExecReq {
     string id = 1;
}
message ExecResp {}
```

有更多的消息和`enums`，以允许进行`Status`调用。你可以在这里找到完整的协议缓冲区定义：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/proto/diskerase.proto`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/proto/diskerase.proto)

现在我们有了描述待处理工作的消息，让我们来看一下如何创建策略引擎。

## 创建策略引擎

策略会检查我们的工作，以确保某些参数是被允许的。在我们的案例中，这些参数位于`pb.WorkReq`实例内部。我们希望策略是通用的，这样它们就可以在多个由`pb.WorkReq`描述的工作类型中重用。定义完成后，我们将有一个`policy.json`文件，定义哪些策略应用于特定命名的`pb.WorkReq`。

为了实现这一点，每个策略都需要接收应应用于特定工作流的策略设置。让我们定义两个接口来描述策略及其设置，如下所示：

```
type Settings interface{
     Validate() error
}
type Policy interface {
     Run(ctx context.Context, name string, req *pb.WorkReq, settings Settings) error
}
```

`Settings`将始终作为某种结构体实现。它的`Validate()`方法将用于验证该结构体的字段是否设置为有效值。

`Policy`根据提供的设置运行我们的实现，作用于`pb.WorkReq`。

提交的每个`WorkReq`都将有一个要应用的策略列表。这个列表定义如下：

```
type PolicyArgs struct {
     Name string
     Settings Settings
}
```

`Name`是要调用的策略的名称。`Settings`是该调用的设置。

配置文件将详细列出一组`PolicyArgs`参数以供执行。每个策略都需要在系统中注册。我们将跳过策略注册方法的部分，但这就是策略注册的位置：

```
var policies = map[string]registration{}
type registration struct {
     Policy Policy
     Settings Settings
}
```

当`pb.WorkReq`进入系统时，我们希望同时对该`pb.WorkReq`调用这些策略。让我们看看这是如何工作的：

```
func Run(ctx context.Context, req *pb.WorkReq, args ...PolicyArgs) error {
     if len(args) == 0 {
          return nil
     }
     var cancel context.CancelFunc
     ctx, cancel = context.WithCancel(ctx)
     defer cancel()
     // Make a deep clone so that no policy is able to make changes.
     creq := proto.Clone(req).(*pb.WorkReq)
     runners := make([]func() error, 0, len(args))
     for _, arg := range args {
          r, ok := policies[arg.Name]
          if !ok {
               return fmt.Errorf("policy(%s) does not exist", arg.Name)
          }
          runners = append(
               runners,
               func() error {
                    return r.Policy.Run(ctx, arg.Name, creq, arg.Settings)
               },
          )
     }
     wg := sync.WaitGroup{}
     ch := make(chan error, 1)
     wg.Add(len(runners))
     for _, r := range runners {
          r := r
          go func() {
               defer wg.Done()
               if err := r(); err != nil {
                    select {
                    case ch <- err:
                         cancel()
                    default:
                    }
                    return
               }
          }()
     }
     wg.Wait()
     select {
     case err := <-ch:
          return err
     default:
     }
     if !proto.Equal(req, creq) {
          return fmt.Errorf("a policy tried to modify a request: this is not allowed as it is a security violation")
     }
     return nil
}
```

上述代码定义了以下内容：

+   如果`pb.WorkReq`的配置没有策略，则返回。

+   创建一个`Context`对象，以便在出现错误时取消正在运行的策略。

+   克隆我们的`pb.WorkReq`，使其无法被`Policy`更改。

+   确保每个被命名的`Policy`实际存在。

+   使用我们所提供的设置运行所有策略。

+   如果其中任何一个出现错误，记录该错误并取消所有正在运行的策略。

+   确保`pb.WorkReq`的副本与提交的内容相同。

我们现在已经具备了策略引擎的主要部分。完整的引擎可以在这里找到：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/policy/policy.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/policy/policy.go)

用于读取我们`policy.json`文件的`Reader`类型，在这里进行了详细描述：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/policy/config/config.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/policy/config/config.go)

让我们来看一下编写一个策略，以便我们引擎使用。

## 编写策略

你可以在工作流中定义的最基本策略之一是限制该工作流中允许的作业类型。

这可以防止在工作流中引入某些新类型的工作，而这些工作没有考虑到需要应用于该`Job`的策略。

对于我们的第一个`Policy`实现，让我们编写一个检查`pb.WorkReq`的策略，只允许我们在策略配置中定义的`Job`类型。如果接收到一个未预料的`Job`，我们将拒绝该`pb.WorkReq`。

让我们为我们的`Policy`定义设置，具体如下：

```
type Settings struct { 
    AllowedJobs []string 
}
func (s Settings) Validate() error { 
    for _, n := range s.AllowedJobs { 
        _, err := jobs.GetJob(n) 
        if err != nil { 
            return fmt.Errorf("allowed job(%s) is not defined in the proto") 
        } 
    }
    return nil 
}
func (s Settings) allowed(name string) bool {
     for _, jn := range s.AllowedJobs {
          if jn == name {
               return true
          }
     }
     return false
}
```

以上代码包含以下内容：

+   我们特定的`Settings`，实现了`policy.Settings`

+   `AllowedJobs`，即我们允许的作业名称

+   一个`Validate()`方法，用于验证列出的`Jobs`是否存在

+   一个`allowed()`方法，用于检查给定的名称是否符合我们允许的内容

+   它还使用我们的`jobs`包来执行这些检查

通过这些设置，用户可以在我们的配置文件中为任何工作流定义一个策略，指定允许哪些`Job`类型。

让我们定义一个实现`Policy`接口的类型，具体如下：

```
type Policy struct{}
func New() (Policy, error) {
     return Policy{}, nil
}
func (p Policy) Run(ctx context.Context, name string, req *pb.WorkReq, settings policy.Settings) error {
     const errMsg = "policy(%s): block(%d)/job(%d) is a type(%s) that is not allowed"
     s, ok := settings.(Settings)
     if !ok {
          return fmt.Errorf("settings were not valid")
     }
     for blockNum, block := range req.Blocks {
          for jobNum, job := range block.Jobs {
               if ctx.Err() != nil {
                    return ctx.Err()
               }
               if !s.allowed(job.Name) {
                    return fmt.Errorf(errMsg, blockNum, jobNum, job.name)
               }
          }
     }
     return nil
}
```

以上代码执行以下操作：

+   定义我们的策略，实施`policy.Policy`接口

+   定义了一个`New()`构造函数

+   实现了`policy.Policy.Run()`方法

+   验证传入的`policy.Settings`值是否是此`Policy`的`Settings`

+   遍历所有`req.Blocks`并获取我们的`Job`实例

+   检查每个`Job`是否具有允许的名称

我们现在有一个可以应用的策略，限制`pb.WorkReq`中的`Job`类型。我们可以在配置文件中将其应用于执行卫星磁盘擦除的工作流，如下所示：

```
{
     "Name": "SatelliteDiskErase",
     "Policies": [
          {
               "Name": "restrictJobTypes",
               "Settings": {
                    "AllowedJobs": [
                            "validateDecom",
                            "diskErase",
                            "sleep",
                            "getTokenFromBucket"
                    ]
               }
          }
     ]
}
```

该策略具有以下属性：

+   仅应用于名为`"SatelliteDiskErase"`的工作流

+   应用了一条单一策略，`"restrictJobTypes"`，这是我们定义的

+   只允许以下名称之一的`Job`类型：

    +   `"validateDecom"`

    +   `"diskErase"`

    +   `"sleep"`

    +   `"getTokenFromBucket"`

你可以在此查看完整的`Policy`实现：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/policy/register/restrictjobtypes/restrictjobtypes.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/policy/register/restrictjobtypes/restrictjobtypes.go)

你可以在以下目录中找到我们定义的其他策略：

[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/16/workflow/internal/policy/register`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/16/workflow/internal/policy/register)

你可以在此处查看当前定义的策略配置：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/policy/config/config.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/policy/config/config.go)

## 关于策略引擎的警告

在继续之前，我想提醒一句警告。

简单性是可持续软件的关键。我定义的可持续软件具有以下特征：

+   容易调试

+   用户最多可以在几个小时内理解如何使用它

策略引擎在防止重大问题方面可能非常有效，充当对一组操作的理智性进行的二次检查。与安全性一样，它应该在仅引入少量负担的同时，提供显著的好处。

策略引擎容易被过度开发，带着 100%保护的崇高目标，同时引入大量的复杂性和负担。通常，我会看到那些没有紧密结合单一工作流系统的策略引擎。相反，工程师们会设计一个通用系统，试图应对多个工具系统。

如果你的策略语句开始像编程语言（`if`语句、循环、函数）那样，说明你正朝着复杂性迈进。随着策略引擎变得通用，它们变得更复杂，难以处理。如果你在多个地方需要策略强制执行，这也是另一个警告信号。

并非所有工作流都能通过通用策略实现安全。当你拥有复杂的工作流时，可以自由设计一个为单个工作流做深度检查的策略。将你的`if`语句、循环和函数放在代码中，而不是配置中。

我见过工程师编写大量过于复杂的安全系统。专注于提供易于编写和更新的保护机制，覆盖 80%的情况，而不是 100%的情况。通过将创建执行操作的程序和验证这些操作是否符合策略的服务分离，你将不太可能在未来发生*磁盘擦除*类型的事件，更重要的是，你将能够保持开发速度。

在本节中，我们讨论了什么是 SOW。为了让我们的工作流服务理解 SOW 并强制执行它，我们设计了一个策略引擎，并创建了第一个可以应用于提交给我们系统的工作流的策略。

即使有策略，还是会出错。这可能只是一些事件的巧合，导致一个通常安全的操作变得不安全。为了能够快速响应这些类型的事件，让我们来看看如何引入紧急停止功能。

# 构建具有紧急停止功能的系统

系统将会失控。这是你在基础设施工具开发初期就需要接受的一个简单事实。

当你是一个小公司时，通常只有一小部分人非常了解系统，并监督任何更改以处理问题。如果这些人足够优秀，他们可以迅速响应问题。通常，这些人是软件的开发者。

随着公司规模的增长，工作开始变得更加专业化。公司越大，工作越专业化。当这种情况发生时，处理重大问题的第一响应者通常没有足够的访问权限或知识来处理这些问题。

这可能会在识别到重大问题和阻止问题恶化之间创建一个关键的时间差。

这就是允许紧急响应人员停止更改的功能所在。我们称之为紧急停止功能。

## 理解紧急停止

构建紧急停止系统有多种方式，但基本原理相同。软件将检查一个包含正在执行的工作流名称以及紧急停止状态的数据存储。

紧急停止系统的最简单版本有两种模式，如下所示：

+   `Go`

+   `Stop`

执行任何工作类型的软件需要定期引用该系统。如果它找不到自己列出，或者系统表明处于`Stop`状态，则软件终止，或者如果是执行系统，则终止该工作流。

更复杂的版本可能包含站点信息，以便停止在站点上运行的所有工具，或者它可能包括其他状态，如`Pause`。这些实现起来更复杂，因此我们在这里将坚持使用这种简单形式。

让我们看看实现可能是什么样子。

## 构建一个紧急停止包

我们首先需要定义数据格式的样子。对于这个练习，我们将使用`etcd`。虽然我这里使用的是 JSON，但它也可以是数据库中的一个表格或协议缓冲区。

让我们定义工作流可能具有的状态，如下所示：

```
// Status indicates the emergency stop status.
type Status string
const (
     Unknown Status = ""
     Go Status = "go"
     Stop Status = "stop"
)
```

这定义了几个状态，如下所示：

+   `Unknown`，表示状态未设置

+   `Go`，表示工作流可以执行

+   `Stop`，表示工作流应停止

关键是要知道，任何不是`Go`的状态都被视为`Stop`。

现在，让我们定义一个可以转换为 JSON 并从 JSON 转换的紧急停止入口，如下所示：

```
type Info struct {
     // Name is the workflow name.
     Name string
     // Status is the emergency stop status.
     Status Status
}
```

它包含以下字段：

+   `Name`，表示工作流的唯一名称

+   `Status`，表示此工作流的紧急停止状态

紧急停止包的另一个关键点是，每个工作流必须有一个入口。如果检查到一个没有命名的入口，它会被视为设置为`Stop`。

现在，我们需要验证一个入口。以下是处理方法：

```
func (i Info) validate() error {
     i.Name = strings.TrimSpace(i.Name)
     if i.Name == "" {
          return fmt.Errorf(“es.json: rule with empty name”)
     }
     switch i.Status {
     case Go, Stop:
     default:
          return fmt.Errorf("es.json: rule(%s) has invalid Status(%s), ignored", i.Name, i.Status)
     }
     return nil
}
```

上述代码执行以下操作：

+   移除工作流名称周围的任何空格。

+   如果`Name`值为空，则表示错误。

+   如果`Status`值既不是`Go`也不是`Stop`，则表示错误。

我们将这些错误视为规则不存在的错误。如果规则不存在，则工作流被认为处于`Stop`状态。

我们现在需要某些东西，以便定期读取此紧急停止文件或接收变化的通知。如果服务在短时间内无法访问保存我们紧急停止信息的数据存储，它应该报告`Stop`状态。

让我们定义一个`Reader`类型，用于访问我们的紧急停止数据，如下所示：

```
var Data *Reader
func init() {
     r, err := newReader()
     if err != nil {
          panic(fmt.Sprintf("es error: %s", err))
     }
     Data = r
}
type Reader struct {
     entries atomic.Value // map[string]Info
     mu          sync.Mutex
     subscribers map[string][]chan Status
}
func newReader() (*Reader, error) {...}
func (r *Reader) Subscribe(name string) (chan Status, Cancel){...}
func (r *Reader) Status(name string) Status {...}
```

上述代码执行以下操作：

+   提供一个`Data`变量，这是我们`Reader`类型的唯一访问点

+   提供一个`init()`函数，在程序启动时访问我们的紧急停止数据。

+   提供了一个`Reader`类型，允许我们读取紧急停止状态。

+   提供了一个`Subscribe()`函数，返回工作流的状态变化，以及一个`Cancel()`函数，当你不再想订阅时调用。

+   提供一个`Status()`函数，返回一次性状态。

+   提供了`newReader`，这是我们的`Reader`构造函数。

这里没有提供完整代码，但可以在以下链接找到：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/es/es.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/es/es.go)

我们只允许通过`Data`访问紧急停止信息，它充当单例模式。这防止了多个实例同时轮询相同的数据。我更喜欢通过变量访问单例，以清楚表明只有一个实例存在。

我们现在有一个包，可以告诉我们紧急停止状态。让我们看看如何使用它来取消某些操作。

## 使用紧急停止包。

现在我们有了一个可以读取紧急停止数据的包，让我们展示如何使用它，如下所示：

```
type Job interface{
     Run(ctx context.Context)
}
type Work struct {
     name string
     jobs []Job
}
func (w *work) Exec(ctx context.Context) error{
     esCh, cancelES := es.Data.Subscribe(w.name)
     defer cancelES() // Stop subscribing
     if <-esCh != es.Go { // The initial state
          return fmt.Errorf("es in Stop state")
     }
     var cancel context.CancelFunc
     ctx, cancel = context.WithCancel(ctx)
     defer cancel()
     // If we get an emergency stop, cancel our context.
     // If the context gets cancelled, then just exit.
     go func() {
          select {
          case <-ctx.Done():
               return
          case <-esCh:
               cancel()
          }
     }()
     for _, job := range w.jobs {
          if err := job(ctx); err != nil {
               return err
          }
     }
     return nil
}
```

上述代码执行了以下操作：

+   创建一个`Job`，执行我们想要执行的某个操作。

+   创建了一个`Work`类型，执行一组`Jobs`。

+   定义了`Exec()`，它执行所有的`Jobs`。

+   使用给定的工作流名称订阅紧急停止。

+   如果我们没有从`Go`状态开始，它会返回一个错误。

+   执行一个 goroutine，如果我们收到`Stop Status`类型，它会调用`cancel()`。

+   执行保存在工作`.jobs`中的 Job 实例。

这是一个简单的示例，使用`context.Context`对象来停止在调用`cancel()`时正在执行的任何`Job`。如果我们收到紧急停止状态变化（始终为`Stop`），我们会调用`cancel()`。

使用`es`包的更完整示例可以在这两个文件中找到：

+   [`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/service/service.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/service/service.go)

+   [`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/service/executor/executor.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/service/executor/executor.go)

一个示例的`es.json`文件，存储了紧急停止数据，可以在这里找到：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/configs/es.json`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/configs/es.json)

你可以在以下链接看到它作为我们`Work.Run()`方法的一部分，集成到我们的工作流系统中：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/service/executor/executor.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/internal/service/executor/executor.go)

## 案例研究 – 谷歌的网络骨干紧急停止

在一次网络工具问题的早期事后分析中，发现负责处理重大事件的值班工程师需要一种停止自动化操作的方式。当时，我们有许多小工具可能会在任何时候与网络执行操作。值班工程师发现问题时，并没有合适的方式来阻止其他工程师执行工作或停止一个失控的程序。

第一个紧急停止包是通过这次事后分析创建的，并集成到现有的工具中。其工作原理是获取工具的订阅者名称，并将其与紧急停止文件中包含的**正则表达式**（**regexes**）进行匹配。每当文件发生更改或工具执行开始时，都会进行此检查。

这个方法曾被用来停止几项自动化任务，避免了问题的蔓延。然而，这种实现方式对于我们这种快速增长的组织来说存在缺陷。

首先，它要求每个工具开发人员都集成紧急停止包。当更多的团队在初始核心团队之外开发工具时，他们可能并不知道这是一个必需的要求。这导致了不受控的工具开发。而随着谷歌开发自己的网络设备，工具开发跨越了多个部门，这些部门在许多方面并没有协调。这意味着许多工具从未集成紧急停止，或者是在一个独立的系统中完成的。

即使在工具中集成了紧急停止，有时这种实现也存在缺陷，无法正常工作。每次集成都依赖工程师正确执行操作。

最终，紧急停止系统有一个默认的`Go`状态。因此，如果没有规则与您的订阅者 ID 匹配，则假定其处于`Go`状态。这意味着许多时候，您只能停止所有操作，或者必须翻阅代码找出订阅者 ID，以便重新启用除了问题工具之外的所有内容。

为了解决我们网络骨干中的这些问题，我们将骨干工作的执行集中到一个中央系统中。这为我们提供了一个单一且经过充分测试的紧急停止实现，经过长时间的审计后，我们将紧急停止包切换为停止任何不符合规则的操作。

这为我们的应急响应人员提供了在发生重大问题时停止骨干自动化和工具的能力。如果我们发现有问题的工具，我们可以允许其他所有工具继续运行，直到对该工具进行适当修复。

在这一部分中，你已经学习了什么是紧急停止系统，为什么它很重要，如何实现一个基础的系统，以及如何将紧急停止包集成到工具中。

# 总结

本章提供了如何编写在面对混乱时能够提供安全保障的工具的基本理解。我们展示了如何通过断路器或指数退避技术，在发生意外问题时避免网络和服务的过载。我们展示了如何通过速率限制自动化，在响应者还未作出反应前防止工作流失控。你已经了解了通过集中式策略引擎进行工具作用域管理，如何在不加重开发者负担的情况下提供第二层安全保障。我们学习了幂等性工作流的重要性，以便实现工作流的恢复。最后，我们介绍了如何通过紧急停机系统，帮助首批响应者在调查问题时，快速限制自动化系统的损害。

此时，如果你还没有玩过我们一直在开发的工作流系统，应该去探索代码并尝试示例。`README.md` 文件将帮助你入门。你可以通过以下链接找到它：

[`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/README.md`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/16/workflow/README.md)
