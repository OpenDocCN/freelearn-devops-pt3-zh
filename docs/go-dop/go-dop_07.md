# 第六章：与远程数据源交互

在上一章中，我们讨论了如何处理常见的数据格式，并展示了如何读取和写入这些格式的数据。但在那一章中，我们仅仅处理了可以通过文件系统访问的数据。

虽然文件系统实际上可能通过**网络文件系统**（**NFS**）或**服务器消息块**（**SMB**）等服务，拥有存储在远程设备上的文件，但也存在其他远程数据源。

在本章中，我们将讨论一些常见的方式，用于在远程数据源中发送和接收数据。本章将重点介绍如何使用**结构化查询语言**（**SQL**）、**表述性状态转移**（**REST**）和**Google 远程过程调用**（**gRPC**）来访问远程系统中的数据。你将学习如何访问常见的 SQL 数据存储，重点是 PostgreSQL。我们还将探讨如何使用 REST 和 gRPC 风格的 RPC 方法创建和查询**远程过程调用**（**RPC**）服务。

通过在这里学到的技能，你将能够连接并查询 SQL 数据库中的数据，向数据库添加新条目，向服务请求远程操作，并从远程服务获取信息。

本章我们将讨论以下主题：

+   访问 SQL 数据库

+   开发 REST 服务和客户端

+   开发 gRPC 服务和客户端

在下一节中，我们将深入探讨使用最古老的数据格式之一——**逗号分隔值**（**CSV**）。

让我们开始吧！

# 技术要求

本章的代码文件可以从[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/6/grpc`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/6/grpc)下载。

# 访问 SQL 数据库

DevOps 工程师通常需要访问存储在数据库系统中的数据。**SQL** 是与数据库系统通信的标准，是 DevOps 工程师在日常工作中会遇到的内容。

Go 提供了一个标准库，用于与基于 SQL 的系统交互，称为`database/sql`。该包提供的接口，结合数据库驱动程序，允许用户与多种不同的 SQL 数据库进行操作。

在本节中，我们将学习如何使用 Go 访问 Postgres 数据库，以执行基本的 SQL 操作。

重要提示

本节中的示例将要求你设置一个 Postgres 数据库。此内容超出了本书的范围。本书并不讲解 SQL。需要具备一些基本的 SQL 知识。

你可以在[`www.postgresql.org/download/`](https://www.postgresql.org/download/)找到关于如何为你的操作系统安装 Postgres 的信息。如果你更倾向于在本地 Docker 容器中运行 Postgres，可以在[`hub.docker.com/_/postgres`](https://hub.docker.com/_/postgres)找到相关信息。

## 连接到 Postgres 数据库

要连接到 Postgres 数据库，需要使用 Postgres 的数据库驱动程序。目前推荐的第三方包是`github.com/jackc/pgx`。该包实现了一个适用于`database/sql`的 SQL 驱动程序，并提供了自己的方法/类型来支持 Postgres 特定功能。

使用`database/sql`或 Postgres 特定类型的选择取决于是否需要确保不同数据库之间的兼容性。使用`database/sql`允许你编写适用于任何 SQL 数据库的函数，而使用 Postgres 特定功能则移除了兼容性，使迁移到其他数据库变得更加困难。我们将讨论如何使用这两种方法执行我们的示例。

下面是如何使用标准 SQL 包连接而不使用额外的 Postgres 功能：

```
/* 
dbURL might look like:
"postgres://username:password@localhost:5432/database_name"
*/
conn, err := sql.Open("pgx", dbURL)
if err != nil {
     return fmt.Errorf("connect to db error: %s\n", err)
}
defer conn.Close()
ctx, cancel := context.WithTimeout(
     context.Background(), 
     2 * time.Second
)
if err := conn.PingContext(ctx); err != nil {
  return err
}
cancel()
```

在这里，我们使用`pgx`驱动程序打开一个 Postgres 连接，该驱动程序将在导入以下包时注册：

```
_ "github.com/jackc/pgx/v4/stdlib"
```

这是一个匿名导入，意味着我们没有直接使用`stdlib`。这是在我们想要产生*副作用*时使用的，例如在使用`database/sql`包注册驱动程序时。

`Open()`调用不会测试我们的连接。你会看到`conn.PingContext()`来测试我们是否能够向数据库发起请求。

当你想要使用`pgx-specific`类型来操作 Postgres 时，设置略有不同，从不同的包导入开始：

```
"github.com/jackc/pgx/v4/pgxpool"
```

要创建该连接，请键入以下内容：

```
conn, err := pgxpool.Connect(ctx, dbURL)
if err != nil {
     return fmt.Errorf("connect to db error: %s\n", err)
}
defer conn.Close(ctx)
```

这使用连接池连接到数据库，以提高性能。你会注意到我们没有`PingContext()`调用，因为本地连接会在`Connect()`过程中自动测试连接。

现在你知道如何连接到 Postgres 了，接下来我们来看看如何进行查询。

## 查询 Postgres 数据库

让我们考虑一下如何向你的 SQL 数据库发起请求，获取存储在表中的用户信息。

使用标准库，键入以下内容：

```
type UserRec struct {
     User string
     DisplayName string
     ID int
}
func GetUser(ctx context.Context, conn *sql.DB, id int) (UserRec, error) {
     const query = `SELECT "User","DisplayName" FROM users WHERE "ID" = $1`
     u := UserRec{ID: id}
     err := conn.QueryRowContext(ctx, query, id).Scan(&u)
     return u, err
}
```

这个示例执行了以下操作：

+   创建`UserRec`以存储用户的 SQL 数据

+   创建一个名为`query`的查询语句

+   查询我们的数据库，获取请求的 ID 对应的用户

+   返回`UserRec`和一个错误（如果有的话）

我们可以通过在对象中使用预准备语句，而不是仅仅使用函数来提高此示例的效率：

```
type Storage struct {
     conn *sql.DB
     getUserStmt *sql.Stmt
}
func NewStorage(ctx context.Context, conn *sql.DB) *Storage
{
     return &Storage{
          getUserStmt: conn.PrepareContext(
               ctx,
               `SELECT "User","DisplayName" FROM users WHERE "ID" = $1`,
          )
     }
}
func (s *Storage) GetUser(ctx context.Context, id int) (UserRec, error) {
     u := UserRec{ID: id}
     err := s.getUserStmt.QueryRow(id).Scan(&u)
     return u, err
}
```

这个示例执行了以下操作：

+   创建一个可重用的对象

+   存储`*sql.Stmt`，这可以提高重复查询时的效率

+   定义一个`NewStorage`构造函数来创建我们的对象

由于使用标准库的通用性，在这些示例中，任何`*sql.DB`的实现都可以使用。只要 MariaDB 有相同的表名和格式，切换 Postgres 为 MariaDB 也是可行的。

如果我们使用 Postgres 特定的库，代码将如下所示：

```
err = conn.QueryRow(ctx, query).Scan(&u)
return u, err
```

这种实现方式看起来和工作方式与标准库类似。但这里的 `conn` 对象是一个不同的非接口类型 `pgxpool.Conn`，而不是 `sql.Conn`。尽管功能相似，`pgxpool.Conn` 对象支持 Postgres 特有的类型和语法，例如 `jsonb`，而 `sql.Conn` 不支持。

在使用 Postgres 特有的调用时，不需要为非事务操作使用预处理语句。调用信息会自动缓存。

上面的示例比较简单，我们只是拉取了一个特定的条目。如果我们想要一个方法来检索所有 ID 在两个数字之间的用户呢？我们可以使用标准库来定义：

```
/* 
stmt contains `SELECT "User","DisplayName","ID" FROM users 
WHERE "ID" >= $1 AND "ID" < $2`
*/
func (s *Storage) UsersBetween(ctx context.Context, start, end int) ([]UserRec, error) {
     recs := []UserRec{}
     rows, err := s.usersBetweenStmt(ctx, start, end)
     defer rows.Close()
     for rows.Next() {
          rec := UserRec{}
          if err := rows.Scan(&rec); err != nil {
               return nil, err
          }
          recs = append(recs, rec)
     }
     return recs, nil
}
```

Postgres 特有的语法保持不变，只是将 `s.usersBetweenStmt()` 替换成了 `conn.QueryRow()`。

## 空值

SQL 有一个空值概念，适用于布尔值、字符串和 int32 等基本类型。Go 没有这个约定；相反，它为这些类型提供了零值。

当 SQL 允许某列具有空值时，标准库提供了在 `database/sql` 中的特殊空类型：

+   `sql.NullBool`

+   `sql.NullByte`

+   `sql.NullFloat64`

+   `sql.NullInt16`

+   `sql.NullInt32`

+   `sql.NullInt64`

+   `sql.NullString`

+   `sql.NullTime`

当设计你的模式时，最好使用零值而不是空值。但有时，你需要区分一个值是否已设置与零值。在这种情况下，你可以使用这些特殊类型代替标准类型。

例如，如果我们的 `UserRec` 可能有一个空的 `DisplayName`，我们可以将 `string` 类型更改为 `sql.NullString`：

```
type UserRec struct {
     User string
     DisplayName sql.NullString
     ID int
}
```

你可以在这里查看服务器如何根据 `DisplayName` 列的值设置这些值的示例：[`go.dev/play/p/KOkYdhcjhdf`](https://go.dev/play/p/KOkYdhcjhdf)。

## 向 Postgres 写入数据

向数据库写入数据很简单，但需要考虑语法。用户写入数据时最常见的两个操作如下：

+   更新现有条目

+   插入新条目

在标准 SQL 中，你不能执行 *如果存在则更新条目；如果不存在则插入*。由于这是一个常见操作，每个数据库都有自己独特的语法来完成此操作。在使用标准库时，你必须在执行更新或插入之间做出选择。如果你不知道条目是否存在，你将需要使用事务，稍后我们会详细说明。

执行更新或插入只是使用不同的 SQL 语法和 `ExecContext()` 调用：

```
func (s *Storage) AddUser(ctx context.Context, u UserRec) error {
     _, err := s.addUserStmt.ExecContext(
          ctx,
          u.User,
          u.DisplayName,
          u.ID,
     )
     return err
}
func (s *Storage) UpdateDisplayName(ctx context.Context, id int, name string) error {
     _, err := s.updateDisplayName.ExecContext(
          ctx,
          name,
          id,
     )
     return err
}
```

在这个示例中，我们添加了两个方法：

+   `AddUser()` 将新用户添加到系统中。

+   `UpdateDisplayName()` 更新具有特定 ID 的用户的显示名称。

+   两者都使用 `sql.Stmt` 类型，它将作为对象中的一个字段，类似于 `getUserStmt`。

使用 Postgres 原生包实现时的主要区别在于调用的方法名称，以及没有预处理语句。实现 `AddUser()` 的代码如下：

```
func (s *Storage) AddUser(ctx context.Context, u UserRec) error {
     const stmt = `INSERT INTO users (User,DisplayName,ID)
     VALUES ($1, $2, $3)`
     _, err := s.conn.Exec(
          ctx, 
          stmt, 
          u.User, 
          u.DisplayName, 
          u.ID,
     )
    return err 
}
```

有时候，仅仅对数据库进行读写操作是不够的。有时，我们需要原子性地执行多个操作，并将它们视为一个整体。因此，在下一节中，我们将讨论如何使用事务来实现这一点。

## 事务

事务提供了一系列在服务器上作为一个整体执行的 SQL 操作。它通常用于提供某种类型的原子操作，其中需要执行读和写，或者在执行写操作之前先读取数据。

在 Go 中，事务很容易创建。让我们创建一个 `AddOrUpdateUser()` 调用，在添加或更新数据之前检查用户是否存在：

```
func (s *Storage) AddOrUpdateUser(ctx context.Context, u UserRec) (err error) {
     const (
          getStmt = `SELECT "ID" FROM users WHERE "User" = $1`
          insertStmt = `INSERT INTO users (User,DisplayName,ID)
          VALUES ($1, $2, $3)`
          updateStmt = `UPDATE "users" SET "User" = $1,
          "DisplayName" = $2 WHERE "ID" = 3`
     )
     tx, err := s.conn.BeginTx(ctx, &sql.TxOptions{Isolation: sql.LevelSerializable})
     if err != nil {
           return err
     }
     defer func() {
          if err != nil {
               tx.Rollback()
               return
          }
          err = tx.Commit()
     }()
     _, err := tx.QueryRowContext(ctx, getStmt, u.User)
     if err != nil {
          if err == sql.ErrNoRows {
               _, err = tx.ExecContext(ctx, insertStmt, u.User, u.DisplayName, u.ID)
               if err != nil {
                    return err
               }
          }
          return err
     }
     _, err = tx.ExecContext(ctx, updateStmt, u.User, u.DisplayName, u.ID))
     return err
}
```

这段代码执行了以下操作：

+   创建一个隔离级别为 `LevelSerializable` 的事务。

+   使用 `defer` 语句来判断是否发生了错误：

    +   如果找到了用户，我们会回滚整个事务。

    +   如果没有找到用户，我们尝试提交事务。

+   查询用于查找用户是否存在：

    +   它通过检查错误类型来确定这一点。

    +   如果错误是 `sql.ErrNoRows`，说明我们没有找到该用户。

    +   如果错误是其他类型，则表示系统错误。

+   如果我们没有找到该用户，则执行插入语句。

+   如果我们找到了该用户，则执行更新语句。

事务的关键要素如下：

+   `conn.BeginTx`，用于开始事务

+   `tx.Commit()`，用于提交我们的更改

+   `tx.Rollback()`，用于回滚我们的更改

`defer` 语句是一种很好的方式来处理创建事务后执行 `Commit()` 或 `Rollback()`。它确保在函数结束时，无论如何都会执行其中一个操作。

隔离级别对事务很重要，因为它会影响系统的性能和可靠性。Go 提供了多个隔离级别；然而，并非所有数据库系统都支持所有的隔离级别。

你可以在这里了解更多关于隔离级别的内容：[`en.wikipedia.org/wiki/Isolation_(database_systems)#Isolation_levels`](https://en.wikipedia.org/wiki/Isolation_(database_systems)#Isolation_levels)。

## 特定于 Postgres 的类型

到目前为止，我们已经向你展示了如何使用标准库和特定于 Postgres 的对象与 Postgres 交互。但我们还没有真正展示使用 Postgres 对象的充分理由。

当你需要使用不是 SQL 标准的一部分的类型或功能时，Postgres 对象非常有用。让我们重写我们的事务示例，但这次我们不通过标准列来存储数据，而是让我们的 Postgres 数据库只包含两个列：

+   `int` 类型的 ID

+   `jsonb` 类型的数据

`jsonb` 不是 SQL 标准的一部分，不能使用标准 SQL 库实现。`jsonb` 可以极大简化你的工作，因为它允许你在查询时使用 JSON 字段来存储 JSON 数据：

```
func (s *Storage) AddOrUpdateUser(ctx context.Context, u UserRec) (err error) {
     const (
          getStmt = `SELECT "ID" FROM "users" WHERE "ID" = $1`
          updateStmt = `UPDATE "users" SET "Data" = $1 WHERE "ID" = $2`
          addStmt = `INSERT INTO "users" (ID,Data) VALUES ($1, $2)`
     )
     tx, err := conn.BeginTx(
          ctx , 
          pgx.TxOptions{
               IsoLevel: pgx.Serializable,
               AccessMode: pgx.ReadWrite,
               DeferableMode: pgx.NotDeferrable,
          },
     )
     defer func() {
          if err != nil {
               tx.Rollback()
               return
          }
          err = tx.Commit()
     }()
     _, err := tx.QueryRow(ctx, getUserStmt, u.ID)
     if err != nil {
          if err == sql.ErrNoRows {
               _, err = tx.ExecContext(ctx, insertStmt, u.ID, u)
               if err != nil {
                    return err
               }
          }
          return err
     }
     _, err = tx.Exec(ctx, updateStmt, u.ID, u)
     return err
}
```

这个示例在几个方面有所不同：

+   它有额外的 `AccessMode` 和 `DeferableMode` 参数。

+   我们可以将我们的对象 `UserRec` 作为 `Data` `jsonb` 列传递。

访问模式和可延迟模式增加了额外的约束，这些约束在标准库中无法直接使用。

使用`jsonb`是一个福音。现在，我们可以在我们的表上使用`WHERE`子句进行搜索，这些子句可以过滤`jsonb`字段的值。

你还会注意到，`pgx`足够聪明，能够识别我们的列类型，并自动将我们的`UserRec`转换为 JSON。

如果你想了解更多关于 Postgres 值类型的信息，可以访问[`www.postgresql.org/docs/9.5/datatype.html`](https://www.postgresql.org/docs/9.5/datatype.html)。

如果你想了解更多关于`jsonb`以及访问其值的函数，可以访问[`www.postgresql.org/docs/9.5/functions-json.html`](https://www.postgresql.org/docs/9.5/functions-json.html)。

## 其他选项

除了标准库和特定数据库的包之外，**对象关系映射**（**ORMs**）也是常见的工具。ORM 是管理服务和数据存储之间数据的流行模式。

Go 最流行的 ORM 叫做**GORM**，你可以在这里找到：[`gorm.io/index.html`](https://gorm.io/index.html)。

另一个流行的框架是 Beego，它也包括对 REST 和 Web 服务的支持，你可以在这里找到：[`github.com/beego/beego`](https://github.com/beego/beego)。

## 存储抽象

许多开发者倾向于直接在代码中使用存储系统，传递一个数据库连接。这并不是最优的做法，因为它可能会导致以下问题：

+   在存储访问之前添加缓存层。

+   为你的服务迁移到一个新的存储系统。

抽象存储在一个内部**应用程序编程接口**（**API**）接口的背后将允许你通过简单地实现接口并使用新的后端，来以后更换存储层。你可以随时插入新的后端。

这里有一个简单的例子，可能是为获取用户数据添加一个接口：

```
type UserStorage interface {
     User(ctx context.Context, id string) (UserRec, error)
     AddUser(ctx context.Context, u UserRec) error
     UpdateDisplayName(ctx context.Context, id string, name string) error
}
```

这个接口允许你使用 Postgres、本地文件、SQLite、Azure Cosmos DB、内存数据结构或任何其他存储介质来实现你的存储后端。

这样做的好处是能够通过插入一个新的实现来实现从一个存储介质到另一个存储介质的迁移。作为附加好处，你可以将测试与数据库解耦。大多数测试可以使用内存数据结构，这样可以测试你的功能，而无需启动和拆除基础设施，这对于真实数据库是必要的。

添加缓存层变成了一个简单的练习，只需要编写一个`UserStorage`实现，它在读取时调用缓存，当缓存中未找到时，调用你的数据存储实现。你可以替换原有实现，一切保持正常工作。

请注意，这里描述的所有关于通过接口进行抽象的内容适用于对服务数据的访问。SQL API 应该仅用于您的应用程序存储和读取数据。其他服务应使用稳定的 RPC 接口。这提供了相同类型的抽象，使您可以在不迁移用户的情况下更改数据后端。

## 案例研究——一个编排系统的数据迁移——Google

在我任职于 Google 期间，我参与的一个系统是用于自动化网络变更的编排系统。该系统接收自动化指令并执行这些指令，针对各种目标进行操作。这些操作可能包括通过**安全文件传输协议**（**SFTP**）推送文件、与网络路由器交互、更新权威数据存储或运行状态验证。

在操作中，确保表示工作流状态的数据始终是最新的至关重要。这不仅包括当前正在运行的工作流，还包括用于创建新工作流的先前工作流的状态。

为了减轻我们的操作负担，我们希望将工作流的存储系统从 Bigtable 迁移到 Spanner。Bigtable 需要更复杂的设置来处理发生问题时的故障切换到备份单元，而 Spanner 的设计就包括了这一功能。这使我们在单元出现问题时无需干预。

存储层被隐藏在存储接口背后。存储在我们的`main()`函数中初始化，并传递给其他需要它的模块。这意味着我们可以用新的实现替换存储层。

我们实现了一个新的存储接口，将数据同时写入 Bigtable 和 Spanner，并从它们两个中读取数据，使用最新的数据戳，并在需要时更新记录。

这使得我们在历史数据传输期间可以同时使用两个数据存储。一旦同步完成，我们将二进制文件迁移到仅包含 Spanner 实现的版本。我们的迁移完成了，且在成千上万的关键操作运行时没有发生服务停机。

到目前为止，在本章中，我们已经学习了如何使用`database/sql`访问通用数据存储，并特别讨论了 Postgres。我们学习了如何读取和写入 Postgres，并实现事务。我们还讨论了使用`database/sql`与使用数据库特定库（如`pgx`）之间的优劣。最后，我们展示了如何通过将实现隐藏在接口抽象背后，使您能够更轻松地更改存储后端，并对依赖于存储的代码进行封闭测试。

接下来，我们将研究如何使用 REST 或 gRPC 访问 RPC 服务。

# 开发 REST 服务和客户端

在互联网和现如今充斥云空间的分布式系统之前，系统间通信的标准并未广泛应用。这种通信通常称为 RPC。简单来说，这意味着一台机器上的程序调用运行在另一台机器上的函数并接收输出。

单体应用曾是常态，服务器通常要么按应用程序进行隔离并垂直扩展，要么作为作业运行在更大、更专业的硬件上，这些硬件来自 IBM、Sun、SGI 或 Cray 等公司。当系统需要相互通信时，它们往往使用自己的定制协议格式，例如你在 Microsoft SQL Server 中看到的格式。

随着 2000 年代互联网的定义，单体大系统无法以合理的成本为像 Google 搜索或 Facebook 这样的服务提供计算能力。为了为这些服务提供支持，企业需要将大量标准 PC 视为一个整体系统。单一系统可以使用 Unix 套接字或共享内存调用在进程之间进行通信，但企业需要在不同机器上运行的进程之间有共同且安全的通信方式。

随着 HTTP 成为系统间通信的事实标准，当今的 RPC 机制使用某种形式的 HTTP 来进行数据传输。这使得 RPC 可以更轻松地穿越系统，如负载均衡器，并轻松利用安全标准，如**传输层安全性**（**TLS**）。这还意味着，随着 HTTP 传输的升级，这些 RPC 框架可以借助数百甚至成千上万名工程师的努力。

在这一部分，我们将讨论最流行的 RPC 机制之一——REST。REST 使用 HTTP 调用和你选择的任何消息格式，尽管大多数情况下使用 JSON 作为消息格式。

## 用于 RPC 的 REST

在 Go 中编写 REST 客户端相当简单。如果你在过去的 10 年里一直在开发应用程序，那么你要么已经使用过 REST 客户端，要么已经编写过一个。像 Google Cloud Platform 的 Cloud Spanner、Microsoft 的 Azure Data Explorer 或 Amazon DynamoDB 等云服务的 API 都使用 REST 通过它们的客户端库与服务进行通信。

REST 客户端可以执行以下操作：

+   使用 `GET`、`POST`、`PATCH` 或任何其他类型的 HTTP 方法。

+   支持任何序列化格式（尽管通常是 JSON）。

+   允许数据流传输。

+   支持查询变量。

+   使用 URL 标准支持多个版本的 API。

Go 中的 REST 也不需要任何框架即可在服务器端实现。所需的一切都包含在标准库中。

### 编写一个 REST 客户端

让我们编写一个简单的 REST 客户端，访问服务器并接收一个 `POST` 请求 – `/v1/qotd`。

首先，让我们定义需要发送到服务器的消息：

```
type getReq struct {
     Author string `json:"author"`
}
type getResp struct {
     Quote string `json:"quote"`
     Error *Error `json:"error"`
}
```

让我们来讨论一下这些操作的作用：

+   `getReq` 详细说明了服务器 `/v1/qotd` 函数调用的参数。

+   `getResp`是我们期望从服务器函数调用中返回的内容。

我们使用字段标签来允许将小写键转换为我们的公共变量（这些变量首字母大写）。为了让`encoding/json`包能看到这些值并进行序列化，它们必须是公共的。私有字段将无法被序列化：

```
type Error struct {
     Code ErrCode
     Msg string
}
func (e *Error) Error() string {
     return fmt.Errorf("(code %v): %s", e.Code, e.Msg)
}
```

这定义了一个自定义错误类型。通过这种方式，我们可以存储错误代码并返回给用户。这个代码在我们的响应对象旁边定义，但直到稍后的代码中才会使用。

现在，让我们定义一个 QOTD 客户端和一个构造函数，进行一些基本的地址检查并创建一个 HTTP 客户端，以便我们能够向服务器发送数据：

```
type QOTD struct {
     addr string
     client *http.Client
}
func New(addr string) (*QOTD, error) {
     if _, _, err := net.SplitHostPort(addr); err != nil {
          return nil, err
     }
     return &QOTD{addr: addr, client: &http.Client{}}
}
```

下一步是创建一个通用函数，用于进行 REST 调用。由于 REST 非常开放，难以编写一个可以处理所有类型 REST 调用的函数。编写 REST 服务器时的最佳实践是只支持`POST`方法；永远不要使用查询变量和简单的 URL。然而，在实际应用中，如果你不控制服务，你将处理各种各样的 REST 调用类型：

```
func (q *QOTD) restCall(ctx context.Context, endpoint string, req, resp interface{}) error {
     if _, ok := ctx.Deadline(); !ok {
          var cancel context.CancelFunc
          ctx, cancel = context.WithDeadline(ctx, 2 * time.Second)
          defer cancel()
     }
     b, err := json.Marshal(req)
     if err != nil {
          return err
     }
     hReq, err := http.NewRequestWithContext(
          ctx, 
          http.POST, 
          endpoint,
          bytes.NewBuffer(b), 
     )
     if err != nil {
          return err
     }
     resp, err := q.client.Do(hReq)
     if err != nil {
          return err
     }
     b, err := io.ReadAll(resp.Body)
     if err != nil {
          return err
     }
     return json.Unmarshal(b, resp)
}
```

这段代码执行了以下操作：

+   检查我们的上下文是否有截止时间：

    +   如果它有值，则会被尊重。

    +   如果没有设置，则会设置一个默认值。

    +   在调用完成后，调用`cancel()`。

+   将请求转换为 JSON。

+   创建一个新的`*http.Request`，执行以下操作：

    +   使用`POST`方法。

    +   与端点进行通信。

    +   拥有一个`io.Reader`，用于存储 JSON 请求。

+   使用客户端发送请求并获取响应。

+   从`http.Response`的正文中获取响应。

+   将 JSON 反序列化为响应对象。

你会注意到`req`和`resp`都是`interface{}`类型。这使得我们可以将这个例程与任何表示 JSON 请求或响应的结构体一起使用。

现在，我们将在一个方法中使用它，通过作者获取 QOTD（每日名言）：

```
func (q *QOTD) Get(ctx context.Context, author string) (string, error) {
     const endpoint = `/v1/qotd`
     resp := getResp{}
     err := q.restCall(ctx, path.Join(q.addr, endpoint), getReq{Author: author}), &resp)
     switch {
     case err != nil:
          return "", err
     case resp.Error != nil:
          return "", resp.Error
     }
     return resp.Quote, nil
}
```

这段代码执行了以下操作：

+   为我们的`get`函数定义一个端点。

+   调用我们的`restCall()`方法，执行以下操作：

    +   使用`path.Join()`将我们的服务器地址和 URL 端点连接起来。

    +   创建一个`getReq`对象作为`restCall()`的`req`参数。

    +   将响应读取到我们的`resp`响应对象中。

    +   如果`*http.Client`返回一个错误，我们就返回那个错误。

    +   如果`resp.Error`被设置，我们返回它。

+   返回响应中的引用内容。

要查看它的运行效果，你可以访问这里：[`play.golang.org/p/Th0PxpglnXw`](https://play.golang.org/p/Th0PxpglnXw)。

我们在这里展示了如何使用 HTTP `POST`请求和 JSON 来创建一个基础的 REST 客户端。然而，我们仅仅触及了创建 REST 客户端的表面。你可能需要向头部添加认证信息，使用**JSON Web Token**（**JWT**）。这使用了 HTTP 而不是 HTTPS，因此没有传输安全性。我们没有尝试使用压缩，如 Deflate 或 Gzip。

虽然使用`http.Client`很容易，但你可能需要一个更智能的封装器，它为你处理这些功能。值得一看的一个库是`resty`，可以在这里找到：[`github.com/go-resty/resty`](https://github.com/go-resty/resty)。

### 编写一个 REST 服务

现在我们已经写好了客户端，接下来我们写一个 REST 服务端点来接收请求并将输出发送给用户：

```
type server struct {
     serv *http.Server
     quotes map[string][]string
}
```

这段代码执行了以下操作：

+   创建了服务器`struct`，`which`将充当我们的服务器

+   使用`*http.Server`来提供 HTTP 内容

+   有`quotes`，它存储作者作为键，值是一个名言切片

现在，我们需要一个构造函数：

```
func newServer(port int) (*server, error) {
     s := &server{
          serv: &http.Server{
               Addr: ":" + strconv.Itoa(port),
          },
          quotes: map[string][]string{
               // Add quotes here
          },
     }
     mux := http.NewServeMux()
     mux.HandleFunc(`/qotd/v1/get`, s.qotdGet)
     // The muxer implements http.Handler 
     // and we assign it for our server’s URL handling.
     s.serv.Handler = mux
     return s, nil
}
func (s *server) start() error {
     return s.serv.ListenAndServe()
}
```

这段代码执行了以下操作：

+   创建了一个`newServer`构造函数：

    +   这有一个`port`参数，指定运行服务器的端口。

+   创建一个`server`实例：

    +   创建一个`*http.Server`实例，运行在`:[port]`端口上

    +   填充我们的`quotes map`

+   添加`*http.ServeMux`来将 URL 映射到方法。

    注意

    我们稍后将创建`qotdGet`方法。

+   创建一个名为`start()`的方法，用于启动我们的 HTTP 服务器。

`*http.ServeMux`实现了`http.Handler`接口，`*http.Server`使用该接口。`ServeMux`通过模式匹配来决定哪个方法与哪个 URL 匹配。你可以在这里了解模式匹配的语法：[`pkg.go.dev/net/http#ServeMux`](https://pkg.go.dev/net/http#ServeMux)。

现在，让我们创建一个方法来回答我们的 REST 端点请求：

```
func (s *server) qotdGet(w http.ResponseWriter, r *http.Request) {
     req := getReq{}
     if err := req.fromReader(r.Body); err != nil {
          http.Error(w, err.Error(), http.StatusBadRequest)
          return
     }
     var quotes []string
     if req.Author == "" {
          // Map access is random, this will randomly choose a            	          // set of quotes from an author.
          for _, quotes = range s.quotes {
               break
          }
     } else {
          var ok bool
          quotes, ok = s.quotes[req.Author]
          if !ok {
               b, err := json.Marshal(
                    getResp{
                         Error: &Error{
                              Code: UnknownAuthor,
                              Msg:  fmt.Sprintf("Author %q was not found", req.Author),
                        },
                    },
               )
               if err != nil {
                    http.Error(w, err.Error(), http.StatusBadRequest)
                    return
               }
               w.Write(b)
               return
          }
     }
     i := rand.Intn(len(quotes))
     b, err := json.Marshal(getResp{Quote: quotes[i]})
     if err != nil {
          http.Error(w, err.Error(), http.StatusBadRequest)
          return
     }
     w.Write(b)
     return
```

这段代码执行了以下操作：

+   实现了`http.Handler`接口。

+   读取 HTTP 请求体并将其序列化到我们的`getReq`中：

    +   如果请求有问题，这段代码会使用`http.Error()`返回 HTTP 错误代码。

+   如果请求中没有包含“author”字段，则随机选择一个作者的名言。

+   否则，查找作者并获取他们的名言：

    +   如果该作者不存在，则响应`getResp`并包含一个错误信息。

+   随机选择一句名言并返回给客户端。

现在，我们有了一个 REST 端点，它能够回答客户端的 RPC 请求。你可以在这里看到这段代码的运行：[`play.golang.org/p/Th0PxpglnXw`](https://play.golang.org/p/Th0PxpglnXw)。

这只是构建 REST 服务的一个简单示例。你可以在此基础上构建认证、压缩、性能追踪等功能。

为了帮助快速启动并减少一些样板代码，以下是一些可能有用的第三方包：

+   Gin: [`github.com/gin-gonic/gin`](https://github.com/gin-gonic/gin)：

    +   一个 REST 示例：[`golang.org/doc/tutorial/web-service-gin`](https://golang.org/doc/tutorial/web-service-gin)

+   Revel: [`revel.github.io`](https://revel.github.io)

既然我们已经讨论过使用 REST 进行 RPC，那么让我们来看看被全球大公司广泛采用的更快速的替代方案——gRPC。

# 开发 gRPC 服务和客户端

gRPC 提供了一个基于 HTTP 的 RPC 框架，并使用 Google 的协议缓冲区格式，这是一种二进制格式，可以转换为 JSON，但提供了架构，并且在许多情况下比 JSON 提供了 10 倍的性能提升。

这个领域还有其他格式，例如 Apache 的 Thrift、Cap'n Proto 和 Google 的 FlatBuffers。然而，这些格式并不如协议缓冲区流行且得到广泛支持，或者只满足某些特定领域的需求，而且也较难使用。

gRPC 和 REST 一样，是一种客户端/服务器框架，用于发起 RPC 调用。gRPC 的不同之处在于，它更倾向于使用一种叫做**协议缓冲区**（简称**proto**）的二进制消息格式。

该格式有一个存储在 `.proto` 文件中的架构，用于通过编译器生成客户端、服务器和消息，以适应你选择的语言的本地库。当 proto 消息被序列化用于在网络上传输时，二进制表示在所有语言中都是一样的。

让我们深入了解协议缓冲区，gRPC 选择的消息格式。

## 协议缓冲区

协议缓冲区在一个位置定义 RPC 消息和服务，并可以使用 proto 编译器为每种语言生成一个库。协议缓冲区具有以下优点：

+   它们编写一次，生成适用于每种语言的代码。

+   消息可以转换为 JSON 以及二进制格式。

+   gRPC 可以使用反向代理来提供 REST 端点，这对于 Web 应用程序来说非常好。

+   二进制协议缓冲区更小，且编码/解码速度是 JSON 的 10 倍。

然而，协议缓冲区也有一些缺点：

+   你必须在对 `.proto` 文件进行任何更改后重新生成消息，才能获得更改。

+   Google 的标准 proto 编译器使用起来既痛苦又令人困惑。

+   JavaScript 原生不支持 gRPC，尽管它支持协议缓冲区。

工具可以帮助解决一些负面问题，我们将使用新的**Buf**工具，[`buf.build`](https://buf.build)，来帮助生成 proto 文件。

让我们看看一个用于 QOTD 服务的协议缓冲区 `.proto` 文件长什么样：

```
syntax = "proto3";
package qotd;
option go_package = "github.com/[repo]/proto/qotd";
message GetReq {
        string author = 1;
}
message GetResp {
        string author = 1;
        string quote = 2;
}
service QOTD {
   rpc GetQOTD(GetReq) returns (GetResp) {};
}
```

`syntax` 关键字定义了我们正在使用的 proto 语言的版本。最常见的版本是 `proto3`，它是该语言的第三个版本。三者的 wire 格式相同，但具有不同的特性集，且生成不同的语言包。

`package` 定义了 proto 包名，这使得该协议缓冲区可以被其他包导入。我们用 `[repo]` 作为占位符来表示 GitHub 仓库。

`go_package` 在生成 Go 文件时专门定义了包名。尽管这被标记为 `option`，但在为 Go 编译时它是必需的。

`message`定义了一种新的消息类型，在 Go 中被生成为`struct`。`message`中的条目详细说明了字段。`string author = 1`在`struct` `GetReq`中创建了一个名为`Author`的`string`类型字段。`1`是 proto 中的字段位置。你不能在消息中有重复的字段号，字段号永远不应该更改，字段也不应该被删除（尽管可以弃用）。

`service`定义了一个 gRPC 服务，包含一个 RPC 端点`GetQOTD`。这个调用接收`GetReq`并返回`GetResp`。

既然我们已经定义了这个协议缓冲文件，我们可以使用 proto 编译器为我们感兴趣的语言生成相应的包。这将包含我们所有的消息以及使用 gRPC 客户端和服务器所需的代码。

让我们来看一下如何从协议缓冲文件生成 Go 包。

## 说明先决条件

在本教程中使用协议缓冲时，您需要安装以下内容：

+   协议缓冲编译器：[`grpc.io/docs/protoc-installation/`](https://grpc.io/docs/protoc-installation/)

+   Go 编译器插件：[`grpc.io/docs/languages/go/quickstart/`](https://grpc.io/docs/languages/go/quickstart/)

+   Buf 工具：[`docs.buf.build/installation`](https://docs.buf.build/installation)

安装了这些之后，您将能够为 C++和 Go 生成代码。其他语言需要额外的插件。

## 生成您的包

我们需要创建的第一个文件是`buf.yaml`文件。我们可以通过进入`proto`目录并执行以下命令来生成`buf.yaml`文件：

```
buf config init
```

这应该生成一个包含以下内容的文件：

```
version: v1
lint:
  use:
    - DEFAULT
breaking:
  use:
    - FILE
```

接下来，我们需要一个文件来告诉我们生成什么输出。创建一个名为`buf.gen.yaml`的文件，并给它以下内容：

```
version: v1
plugins:
  - name: go
    out: ./
    opt:
      - paths=source_relative
  - name: go-grpc
    out: ./
    opt:
      - paths=source_relative
```

这表示我们应该在与`.proto`文件相同的目录中生成`go`和`go-grpc`文件。

现在，我们应该测试我们的 proto 文件是否能成功构建。我们可以通过执行以下命令来做到这一点：

```
buf build
```

如果没有输出，那么我们的 proto 文件应该能够成功编译。否则，我们将得到一个错误列表，需要我们去修复。

最后，让我们生成我们的 proto 文件：

```
buf generate
```

如果你将 proto 文件命名为`qotd.proto`，这将生成以下内容：

+   `qotd.pb.go`，它将包含你所有的消息

+   `qotd_grpc.pb.go`，它将包含所有的 gRPC 存根

现在我们有了 proto 包，让我们构建一个客户端。

## 编写一个 gRPC 客户端

在您仓库的根目录中，让我们创建两个目录：

+   `client/`，它将包含我们的客户端代码

+   `internal/server/`，它将包含我们的服务器代码

现在，让我们创建一个`client/client.go`文件，内容如下：

```
package client
import (
        "context"
        "time"
        "google.golang.org/grpc"
        pb "[repo]/grpc/proto"
)
type Client struct {
        client pb.QOTDClient
        conn   *grpc.ClientConn
}
func New(addr string) (*Client, error) {
        conn, err := grpc.Dial(addr, grpc.WithInsecure())
        if err != nil {
                return nil, err
        }
        return &Client{
                client: pb.NewQOTDClient(conn),
                conn: conn,
        }, nil
}
func (c *Client) QOTD(ctx context.Context, wantAuthor string) (author, quote string, err error) {
        if _, ok := ctx.Deadline(); !ok {
                var cancel context.CancelFunc
                ctx, cancel = context.WithTimeout(ctx, 2 * time.Second)
                defer cancel()
        }
        resp, err := c.client.GetQOTD(ctx, &pb.GetReq{Author: wantAuthor})
        if err != nil {
                return "", "", err
        }
        return resp.Author, resp.Quote, nil
}
```

这是围绕生成的客户端的一个简单包装，我们通过`New()`构造函数建立了与服务器的连接：

+   `grpc.Dial()`连接到服务器的地址：

    +   `grpc.WithInsecure()`允许我们不使用 TLS。（在实际服务中，您需要使用 TLS！）

+   `pb.NewQOTDClient()`接受一个 gRPC 连接并返回我们生成的客户端。

+   `QOTD()`使用客户端调用我们在`GetQOTD()`原型中定义的接口：

    +   如果没有定义超时，这里会定义一个超时。服务器接收这个超时设置。

    +   这会使用生成的客户端来调用服务器。

创建一个包装器作为客户端并非严格必要。许多开发者更喜欢让用户直接使用生成的客户端与服务进行交互。

在我们看来，这对于简单的客户端来说是没问题的。更复杂的客户端通常应通过将逻辑移动到服务器或使用更符合语言习惯的自定义客户端包装器来减轻负担。

现在我们已经定义了客户端，让我们创建我们的服务器包。

## 编写一个 gRPC 服务器

让我们在`internal/server/server.go`创建一个服务器文件。

现在，让我们添加以下内容：

```
package server
import (
        "context"
        "fmt"
        "math/rand"
        "net"
        "sync"
        "google.golang.org/grpc"
        "google.golang.org/grpc/codes"
        "google.golang.org/grpc/status"

        pb "[repo]/grpc/proto"
)
type API struct {
     pb.UnimplementedQOTDServer
     addr string
     quotes map[string][]string
     mu sync.Mutex
     grpcServer *grpc.Server
}
func New(addr string) (*API, error) {
     var opts []grpc.ServerOption
     a := &API{
          addr: addr,
          quotes: map[string][]string{
               // Insert your quote mappings here
          },
          grpcServer: grpc.NewServer(opts...),
     }
     a.grpcServer.RegisterService(&pb.QOTD_ServiceDesc, a)
     return a, nil
}
```

这段代码执行以下操作：

+   定义我们的 API 服务器：

    +   `pb.UnimplementedQOTDServer`是一个生成的接口，包含我们服务器必须实现的所有方法。这是必需的。

    +   `addr`是我们服务器将要运行的地址。

    +   `quotes`包含服务器存储的引用。

+   定义一个`New()`构造函数：

    +   这将创建一个我们`API`服务器的实例。

    +   这将实例注册到我们的`grpcServer`中。

现在，让我们添加启动和停止`API`服务器的方法：

```
func (a *API) Start() error {
     a.mu.Lock()
     defer a.mu.Unlock()
     lis, err := net.Listen("tcp", a.addr)
     if err != nil {
          return err
     }
     return a.grpcServer.Serve(lis)
}
func (a *API) Stop() {
     a.mu.Lock()
     defer a.mu.Unlock()
     a.grpcServer.Stop()
}
```

这段代码执行以下操作：

+   定义`Start()`方法来启动服务器，该方法执行以下操作：

    +   使用`Mutex`来防止同时停止和启动

    +   在`New()`中传递的地址上创建一个 TCP 监听器

    +   使用我们的监听器启动 gRPC 服务器

+   定义`Stop()`方法来停止服务器，该方法执行以下操作：

    +   使用`Mutex`来防止同时停止和启动

    +   告诉 gRPC 服务器优雅地停止

现在，让我们实现`GetQOTD()`方法：

```
func (a *API) GetQOTD(ctx context.Context, req *pb.GetReq) (*pb.GetResp, error) {
     var (
          author string
          quotes []string
     )
     if req.Author == "" {
          for author, quotes = range s.quotes {
               break
          }
     } else {
          author = req.Author
          var ok bool
          quotes, ok = s.quotes[req.Author]
          if !ok {
               return nil, status.Error(
                    codes.NotFound, 
                    fmt.Sprintf("author %q not found", req.author),
               )
          }
     }
     return &pb.GetResp{
          Author: author, 
          Quote: quotes[rand.Intn(len(quotes))],
     }, nil
}
```

这段代码执行以下操作：

+   定义客户端将调用的`GetQOTD()`方法

+   包含与我们的 REST 服务器类似的逻辑

+   使用 gRPC 的错误类型，该类型在`google.golang.org/grpc/status`包中定义，用于返回 gRPC 错误代码

现在我们已经有了客户端和服务器包，让我们创建一个二进制文件来运行我们的服务。

## 创建服务器二进制文件

创建一个名为`qotd.go`的文件，保存我们服务器的`main()`函数：

```
package main
import (
     "flag"
     "log"
     "github.com/[repo]/internal/server"
     pb "[repo]/proto"
)
var addr = flag.String("addr", "127.0.0.1:80", "The address to run on.")
func main() {
     flag.Parse()
     s, err := server.New(*addr)
     if err != nil {
          panic(err)
     }
     done := make(chan error, 1)
     log.Println("Starting server at: ", *addr)
     go func() {
          defer close(done)
          done <-s.Start()
     }()
     err <- done
     log.Println("Server exited with error: ", err)
}
```

这段代码执行以下操作：

+   创建一个标志`addr`，调用者传递此标志来设置服务器运行的地址。

+   创建我们的服务器实例。

+   写明我们正在启动服务器。

+   启动服务器。

+   如果服务器已存在，错误会被打印到屏幕上：

    +   这可能是某些提示，说明端口已被占用。

你可以使用以下命令运行这个二进制文件：

```
go run qotd.go --addr="127.0.0.1:2562"
```

如果你没有传递`--addr`标志，它将默认使用`127.0.0.1:80`。

你应该在屏幕上看到以下内容：

```
Starting server at: 127.0.0.1:2562
```

现在，让我们创建一个二进制文件，使用客户端获取 QOTD。

## 创建客户端二进制文件

创建一个名为`client/bin/qotd.go`的文件，然后添加以下内容：

```
package main
import (
        "context"
        "flag"
        "fmt"
        "github.com/devopsforgo/book/book/code/1/4/grpc/client"
)
var (
        addr   = flag.String("addr", "127.0.0.1:80", "The address of the server.")
        author = flag.String("author", "", "The author whose quote to get")
)
func main() {
        flag.Parse()
        c, err := client.New(*addr)
        if err != nil {
                panic(err)
        }
        a, q, err := c.QOTD(context.Background(), *author)
        if err != nil {
                panic(err)
        }
        fmt.Println("Author: ", a)
        fmt.Printf("Quote of the Day: %q\n", q)
}
```

这段代码执行以下操作：

+   设置一个标志用于服务器的地址

+   设置一个标志用于引用你想要的名言的作者

+   创建一个新的`client.QOTD`实例

+   使用`QOTD()`客户端方法调用服务器

+   将结果或错误打印到终端

你可以使用以下命令运行这个二进制文件：

```
go run qotd.go --addr="127.0.0.1:2562"
```

这将联系运行在此地址的服务器。如果你在其他地址运行服务器，你需要更改此地址以匹配。

如果没有传递`--author`标志，将随机选择一个作者。

你应该在屏幕上看到以下内容：

```
Author: [some author]
Quote: [some quote]
```

现在我们已经看到了如何使用 gRPC 创建一个简单的客户端和服务器应用程序。但这只是 gRPC 功能的开始。

### 我们仅仅是刚刚触及表面

gRPC 是云技术（如 Kubernetes）的关键基础设施组件。它是在多年的 Stubby 经验后构建的，Stubby 是谷歌的内部前身。我们仅仅触及了 gRPC 可以做的冰山一角。这里有一些额外的功能：

+   运行 gRPC 网关以导出 REST 端点

+   提供能够处理安全性和其他需求的拦截器

+   提供流式数据

+   支持 TLS

+   用于附加信息的元数据和尾部信息

+   客户端服务器负载均衡

以下是已经进行切换的一些大型公司：

+   Square

+   Netflix

+   IBM

+   CoreOS

+   Docker

+   CockroachDB

+   Cisco

+   Juniper Networks

+   Spotify

+   Zalando

+   Dropbox

让我们来聊聊如何在公司内部最佳地提供 REST 或 gRPC 服务。

## 公司标准的 RPC 客户端和服务器

谷歌技术栈成功的关键之一是围绕技术的整合。尽管技术中确实存在大量重复，谷歌在某些软件和基础设施组件上进行了标准化。在谷歌内部，很少看到不使用 Stubby（谷歌的内部 gRPC）的客户端/服务器。

工程师用于 RPC 的库被编写成在每种语言中都能一致工作。近年来，**站点可靠性工程**（**SRE**）组织推动围绕 Stubby 构建的包装器，提供一系列功能和最佳实践，以防止每个团队重新发明轮子。这些功能包括：

+   身份验证

+   压缩处理

+   分布式服务速率限制

+   带回退的重试（或断路器）

通过让客户端在没有回退的情况下重试，这消除了许多基础设施的威胁，去除了团队自行设计安全模型的成本，并允许专家对这些项目进行修复。对这些库的修改使每个人受益，降低了发现已经构建好的服务的成本。

作为一个可能携带寻呼机的 DevOps 工程师或 SRE，推动 RPC 层的标准化可以带来无数的好处，例如避免接到寻呼机！

尽管选择常常被视为好事，有限的选择可以让开发团队和运维人员继续专注于他们的产品，而不是基础设施，这对于打造健壮的产品至关重要。

如果你决定提供一个 REST 框架，以下是一些推荐的实践：

+   仅使用`POST`。

+   不要使用查询变量。

+   仅使用 JSON。

+   确保所有的参数都在你的请求中。

这将大大减少你在框架中需要编写的代码量。

在本节中，我们学习了什么是 RPC 服务以及如何使用两种流行的方法，REST 和 gRPC，来编写客户端。你还了解了 REST 有一套较为宽松的指南，而 gRPC 更倾向于使用 schema 类型，并自动生成使用系统所需的组件。

# 总结

本章结束了我们关于与远程数据源交互的内容。我们讨论了如何通过使用 Postgres 的示例连接到 SQL 数据库，了解了 RPC 是什么，并讨论了两种最流行的 RPC 服务类型，REST 和 gRPC。最后，我们为这两种框架编写了服务器和客户端。

本章使你能够连接到最流行的数据库和云服务，以获取和检索数据。现在你可以编写自己的 RPC 服务来开发云应用。

在下一章中，我们将利用这些知识构建工具，控制远程机器上的任务。

所以，不再多说，让我们直接进入如何编写命令行工具。
