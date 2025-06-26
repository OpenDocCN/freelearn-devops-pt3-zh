# 4

# 使用 Shell 脚本自动化

在本章中，我们将通过 shell 脚本演示系统管理任务的自动化。我们将展示使用 Bash shell 处理脚本的几种方法。计划是创建一个脚本来自动化数据库转储的创建。这个任务虽然简单，但将演示如何处理可能出现的问题，并且如何应对这些情况。

在本章中，我们将涵盖以下主题：

+   备份数据库

+   理解脚本编写

+   理解 Bash 内建命令和语法

+   理解备份脚本——第一步

+   处理错误和调试

# 技术要求

对于本章内容，你将需要一个可以安装软件包的 Linux 系统，并且不害怕在过程中出错。为此，虚拟机是最理想的，通常是在一台可以从头开始重新安装的旧计算机上运行。我们不指望会出错，但在学习过程中，可能会发生这种情况。

# 备份数据库

对于最常见的数据库，如 MySQL 和 PostgreSQL，至少有两种不同的方式来备份数据库：

+   通过提取当前的所有数据和数据库架构来进行数据库转储

+   复制复制日志

在云环境中，你也可以对存储备份的磁盘数据库进行快照。

数据库转储也可以作为完整备份使用。复制日志并不是独立的数据库转储，因此你需要将它们与完整备份结合使用。这叫做增量备份。

进行完整备份可能会花费很长时间，尤其是对于大型数据库。在备份过程中，数据库会锁定其数据文件，因此不会将新数据保存到磁盘上；相反，所有数据都会存储在复制日志中，直到数据库锁被释放。对于大型数据库，这个过程可能需要数小时。鉴于此，我们将每周进行一次完整备份，并每小时复制一次所有复制日志。此外，我们还会每天创建一次 AWS 数据库实例的磁盘快照。

有了这些知识，我们可以开始创建最基本版本的脚本了。

# 理解脚本编写

Shell 脚本是一个简单的文本文件，里面包含了命令。与编译程序不同，shell 脚本在执行之前并不会被解析，而是在执行时解析。这使得开发过程非常快速——完全没有编译过程。但同时，执行速度稍慢。而且，编译器能够捕获的错误会在执行时显现，通常会导致脚本退出。

优点是，编写脚本时需要学习的东西并不多——远远少于用 C 或 Python 编写程序时。与系统命令的交互就像是直接输入命令的名称一样简单。

Bash 编程语言缺乏许多复杂性：它没有很多数据类型和结构，作用域控制非常基础，内存实现也不是为了在大规模下高效。

没有一个固定的经验法则来选择何时编写脚本，何时开发程序。然而，有一些要点需要考虑。适合用 Shell 脚本的一个好候选条件如下：

+   它不会很长。我们不能给你一个具体的规则，但当你的代码行数达到数百行时，考虑使用 Python 或将其拆分成多个脚本可能是个好主意。

+   它与系统命令进行交互，有时会进行很多交互。你可以将其视为一种自动化运行这些命令的方式。

+   它不做大量的数据处理。只有少数的数组、字符串和数字，仅此而已。

+   它不执行任何系统调用。Shell 脚本不是用来做系统调用的，也没有直接的方法来做到这一点。

Shell 脚本的基本结构如下：

```
#!/bin/bash
echo "Hello world"
```

第一行以所谓的 she-bang 开头。它用于告诉系统使用哪个解释器（在本例中是 Bash）来运行这个脚本。在很多网上的脚本中，she-bang 看起来像这样：

```
#!/usr/bin/env bash
```

使用 `env` 命令既有一个很大的优势，也有一个劣势。使用它的优点是它会使用当前用户 `PATH` 环境变量中排在最前面的 Bash 可执行文件。具体取决于目的。`env` 命令还不允许你传递任何参数给你选择的解释器。

上面脚本的第二行简单地显示了**Hello world**。它使用了一个内置命令 `echo`，它的作用就是显示你输入的任何文本。

现在，要执行这个脚本，我们需要将其保存到一个文件中。最好以 `.sh` 或 `.bash` 后缀来结束这个文件。执行这个新脚本有两种方式——通过调用解释器并传入脚本名，或者直接通过脚本名执行：

```
admin@myhome:~$ bash ./myscript.sh
Hello world
admin@myhome:~$
```

要直接执行脚本，我们需要更改其权限，使其可以被执行；否则，系统将无法识别它为可执行文件：

```
admin@myhome:~$ chmod +x myscript.sh
admin@myhome:~$ ./myscript.sh
Hello world
admin@myhome:~$
```

类似地，我们可以轻松地将 Python 或任何其他 Shell 设置为我们脚本的解释器。

现在，让我们聚焦于 Bash，并看一些我们将要使用的 Bash 内置命令。

# 理解 Bash 内置命令和语法

在我们开始创建脚本之前，让我们回归基础。首先，我们将探讨 Bash 脚本语言的语法及其局限性。

内置命令是与 Bash 紧密相关的命令，是我们将要使用的主要脚本语法。Bash 会尝试执行它运行所在系统中的任何其他命令。

就像其他任何解释型语言（例如 Python 或 Ruby）一样，Bash 有其独特的语法和语法规则。让我们来看看。

Bash，类似于其他编程语言，从上到下、从左到右解释文件。每行通常包含一个或多个命令。你可以使用管道符号（`|`）或双管道符号（`||`）、分号（`;`）或双和符号（`&&`）将多个命令连接在同一行中。记住，双管道的作用与逻辑`OR`相同，而双和符号的作用与逻辑`AND`相同。这种方式可以让你按顺序运行命令，并根据前一个命令的结果执行下一个命令，而不需要使用更复杂的条件语句。这被称为命令列表或命令链：

```
commandA && commandB
```

在前面的命令中，你可以看到使用双和符号的示例。在这里，`commandB` 只有在 `commandA` 成功执行后才会执行。我们可以通过在命令链的末尾添加`||`来继续连接更多的命令。

```
commandA || commandB
```

另一方面，这个示例展示了如何使用双管道。在这里，`commandB` 只有在 `commandA` 执行失败时才会执行。

Bash（或 Linux 中的任何其他 shell）通过使用返回码来判断一个命令是否执行失败或成功退出。每个命令需要以一个正整数退出——零（`0`）是成功的代码，任何其他代码都是失败的。如果你使用`AND(&&)`或`OR`(`||`)将多个命令连接在一起，则整行的返回状态将由先前执行的命令的返回状态决定。

那么命令后面的单个和符号（`&`）呢？它有一个完全不同的功能——它会在后台执行命令，并且脚本会继续运行，而无需等待命令完成。这对于不需要其他部分程序完成的任务，或者同时运行多个相同命令的实例（例如同时进行多个数据库的完整备份以缩短执行时间）非常有用。

现在我们知道如何连接命令，接下来我们可以了解任何编程语言的另一个核心特性——变量。

## 变量

在 Bash 中，有两种类型的变量：全局变量和局部变量。*全局变量*在脚本运行期间可以访问，除非该变量被取消设置。*局部变量*仅在脚本的某个块中可访问（例如，定义的函数）。

每当脚本执行时，它会从当前运行的 shell 中获取一组现有的变量；这被称为*环境*。你可以使用`export`命令将新变量添加到环境中，使用`use unset`命令移除变量。你也可以使用`declare -x`命令将函数添加到环境中。

所有的参数，无论是局部的还是全局的，都以美元符号（`$`）为前缀。所以，如果你有一个名为 `CARS`（区分大小写）的变量，引用它时，需要在脚本中写成`$CARS`。

对于变量，单引号或双引号（或没有引号）是重要的。如果你将变量放在单引号中，它将不会被展开，而引号中的变量名将被当作字面字符串处理。双引号中的变量会被展开，并且这被认为是一种安全的引用变量的方式（以及连接它们，或者将它们拼接在一起），因为如果字符串中有空格或其他特殊字符，它们对脚本不会有实际意义 —— 也就是说，它们不会被执行。

有时，你需要将多个变量连接起来。你可以使用大括号（`{}`）来完成。例如，`"${VAR1}${VAR2}"` 将扩展为你设置的 `VAR1` 和 `VAR2` 的值。在这种情况下，你也可以使用大括号来截取或替换字符串的部分。这里有一个例子：

```
name="John"
welcome_str="Hello ${name}"
echo "${welcome_str%John}Jack"
```

上述代码将显示 `%` 操作符只会移除字符串末尾的字符。如果你想从字符串开头截取变量，你可以使用 `#` 操作符，方法相同。

如果你在没有引号的情况下引用变量，变量值中的空格可能会破坏脚本的流程并妨碍调试，因此我们强烈建议使用单引号或双引号。

## 参数

我们可以使用两种类型的参数，但它们都是特殊类型的变量，因此每个变量前都有一个美元符号（`$`）。

你需要注意的第一类参数是 *位置参数* —— 这些是传递给脚本或脚本内函数的参数。所有参数从 `1` 开始索引，一直到 `n`，其中 `n` 是最后一个参数。你可以用 `$1` 到 `$n` 引用每一个参数。你可能会好奇，如果使用 `$0` 会发生什么。

`$0` 包含脚本的名称，因此它对于在脚本中生成文档等操作非常有用。

要引用从 1 开始的所有可用参数，你可以使用 `$@`（美元符号，at 符号）。

以下是一些其他常用的特殊参数：

+   `#`：位置参数的数量

+   `?`：最近执行的前台命令的退出代码

+   `$`：Shell 的进程 ID

## 循环

你可能熟悉其他编程语言中的不同类型的循环。在 Bash 中也有这些循环，但语法稍有不同。

最基本的 `for 循环` 看起来像这样：

```
for variable_name in other_variable; do some_commands; done
```

这个变种的 `for` 循环将 `other_variable` 中的每个元素设置为 `variable_name` 的值，并为每个找到的元素执行 `some_commands`。执行完成后，它将以最后执行的命令的状态退出循环。`in other_variable` 部分是可以省略的 —— 在这种情况下，`for` 循环会为每个位置参数执行一次 `some_commands`。使用该参数的示例如下：

```
for variable_name; do some_commands; done
```

上述 `for` 循环将根据你为函数（或此情况下的脚本）添加的输入变量执行多次。你可以使用 `$@` 引用所有位置参数。

以下是一个 C/C++风格的`for`循环：

```
for (( var1 ; var2 ; var3 )); do some_commands; done
```

下面是此语法的一个示例：

```
for ((i=1; i<=5; i++)); do echo $i; done
```

第一个表达式将`i`变量设置为`1`，第二个表达式是循环继续运行的条件，最后一个表达式将`i`变量的值增加 1。每次循环运行时，会显示分配给`i`变量的下一个值。

此命令的输出将如下所示：

```
1
2
3
4
5
```

另一个有用的循环是`while`循环，它会根据需要运行多次，直到满足条件（传递给它的命令成功退出——返回零）。与之对应的是`until`循环，它会一直运行，直到传递给它的命令返回非零状态：

```
while some_command; do some_command; done
until some_command; do some_command; done
```

你可以通过使用始终满足条件的命令来创建一个无限循环，对于`while`循环，条件可以简单地设为`true`。

最常用的命令块是条件语句，它们与`if`语句一起使用。让我们仔细看看。

## 条件执行——`if`语句

`if`语句的语法如下：

```
if test_command
then
  executed_if_test_was_true
fi
```

`test_command` 可以是你能想到的任何命令，但通常，测试是用双方括号或单方括号包围的。这两者之间的区别是前者是一个系统命令，叫做`test`（你可以通过执行`man test`来查看它的语法），而后者是 Bash 的内建命令，功能更强大。

放置变量在方括号中的经验法则是使用双引号，这样即使变量包含空格，也不会改变我们的测试意图：

```
if [[ -z "$SOME_VAR" ]]; then
    echo "Variable SOME_VAR is empty"
fi
```

`-z` 测试检查`$SOME_VAR`变量是否为空。如果变量为空，评估为`true`，否则为`false`。

以下是其他常用的测试：

+   `-a`：逻辑“与”

+   `-o`：逻辑“或”

+   `-eq`：等于

+   `-ne`：不等于

+   `-gt 或 >`：大于

+   `-ge 或 >=`：大于或等于

+   `-lt 或 <`：小于

+   `-le 或 <=`：小于或等于

+   `= 或 ==`：等于

+   `!=`：不等于

+   `-z`：字符串为空（其长度为零字符）

+   `-n`：字符串不为空

+   `-e`：文件存在（目录、符号链接、设备文件或文件系统中的任何其他文件）

+   `-f`：文件是常规文件（不是目录或设备文件）

+   `-d`：文件是目录

+   `-h` 或 `-L`：文件是符号链接

+   `-r`：文件具有读取权限（对于执行测试的用户）

+   `-w`：文件具有写权限（对于执行测试的用户）

+   `-x`：文件可以被执行脚本的用户执行

注意，使用系统测试（单方括号，`[...]`）时，测试可能与内建测试（双方括号，`[[...]]`）行为不同。

双等号比较运算符，在使用通配符匹配字符串时，将根据你是否对模式加引号来匹配模式或字面字符串。

以下是一个模式匹配的例子，假设字符串以`w`开头：

```
if [[ $variable == w* ]];
    echo "Starts with w"
fi
```

当使用系统测试（单个方括号）而不是内置测试时，测试将尝试查找`$`变量是否与本地目录中的任何文件名匹配（包括带有空格的文件）。这可能会导致一些不可预测的结果。

以下是模式匹配的示例，如果字符串为`w*`：

```
if [[ $variable == "w*" ]];
    echo "String is literally 'w*'"
fi
```

拥有这些知识后，我们已经准备好开始创建和运行脚本了。让我们直接开始吧！

# 理解备份脚本 – 第一步

既然我们知道脚本的样子了，就可以开始编写脚本了。你可以使用你喜欢的控制台编辑器或 IDE 来做这件事。让我们创建一个名为`run_backups.sh`的空文件，并更改其权限，使其可执行：

```
admin@myhome:~$ touch run_backups.sh && chmod +x run_backups.sh
admin@myhome:~$ ls -l run_backups.sh
-rwxr-xr-x  1 admin  admin  0 Dec  1 15:56 run_backups.sh
```

这是一个空文件，因此我们需要添加一个基本的数据库备份命令，并从那里继续。我们不会讨论如何授予该脚本访问数据库的权限。我们将备份一个 PostgreSQL 数据库，并使用`pg_dump`工具来实现。

让我们在基本脚本中输入一个 shebang 行并调用`pg_dump`命令：

```
#!/usr/bin/env bash
pg_dump mydatabase > mydatabase.sql
```

要执行此脚本，我们需要启动以下命令：

```
admin@myhome:~$ ./run_backups.sh
```

点号和斜杠表示我们要执行当前目录中的某个文件，文件名为`run_backups.sh`。如果没有最初的点斜杠对，当前运行的 shell（这里是`bash`）将查找`PATH`环境变量，并尝试在其中列出的目录中查找我们的脚本：

```
admin@myhome:~$ echo $PATH
/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

如你所见，这是一个由冒号分隔的目录列表。

现在，让我们看看执行时我们的 Bash 脚本做了什么：

```
admin@myhome:~$ ./run_backups.sh
./run_backups.sh: line 3: pg_dump: command not found
```

除非你系统中已安装`pg_dump`，否则你将看到此错误。这意味着 Bash 没有找到我们打算运行的命令。它还会显示错误发生的行。此外，一个空的`mydatabase.sql`文件被创建了。

通常，我们会创建一个包含所有所需工具的 Docker 镜像，另一个镜像运行 PostgreSQL 数据库。但由于这一部分将在*第八章*中讲解，我们就直接继续安装所有所需的软件包到本地机器上吧。假设你使用的是 Ubuntu 或 Debian Linux 系统，你可以运行以下命令：

```
admin@myhome:~$ sudo apt-get update
Get:1 http://archive.ubuntu.com/ubuntu jammy InRelease [270 kB]
[Condensed for brevity]
Get:18 http://archive.ubuntu.com/ubuntu jammy-backports/main amd64 Packages [3520 B]
Fetched 24.9 MB in 6s (4016 kB/s)
Reading package lists... Done
admin@myhome:~$ sudo apt-get install postgresql
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following additional packages will be installed:
  cron libbsd0 libcommon-sense-perl libedit2 libgdbm-compat4 libgdbm6 libicu70 libjson-perl libjson-xs-perl libldap-2.5-0 libldap-common libllvm14 libmd0 libperl5.34 libpopt0 libpq5 libreadline8
[Condensed for brevity]
Suggested packages:
  anacron checksecurity default-mta | mail-transport-agent gdbm-l10n libsasl2-modules-gssapi-mit | libsasl2-modules-gssapi-heimdal libsasl2-modules-ldap libsasl2-modules-otp libsasl2-modules-sql
[Condensed for brevity]
0 upgraded, 42 newly installed, 0 to remove and 2 not upgraded.
Need to get 68.8 MB of archives.
After this operation, 274 MB of additional disk space will be used.
Do you want to continue? [Y/n] y
```

经用户确认后，数据库将被安装、配置，并在后台启动。为了可读性，我们已经截断了后续输出。

安装完成后，可能还需要对数据库做一些额外的配置更改，以便你能够使用另一个名为`psql`的工具连接到数据库。`psql`是一个用于连接 PostgreSQL 的控制台命令。在`/etc/postgresql/14/main/pg_hba.conf`文件中，我们定义了信任关系以及谁可以使用多种机制连接到数据库。

查找以下行：

```
local   all  postgres peer
```

将其更改为以下内容：

```
local   all  all  trust
```

做出此修改后，你可以使用以下命令重启数据库：

```
admin@myhome:~$ sudo systemctl restart postgresql
* Restarting PostgreSQL 14 database server
```

现在，你应该能够登录到数据库并列出所有可用的数据库：

```
admin@myhome:~$ psql -U postgres postgres
psql (14.5 (Ubuntu 14.5-0ubuntu0.22.04.1))
Type "help" for help.
postgres=# \l
                              List of databases
   Name    |  Owner   | Encoding | Collate |  Ctype  |   Access privileges
-----------+----------+----------+---------+---------+-----------------------
 postgres  | postgres | UTF8     | C.UTF-8 | C.UTF-8 |
 template0 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         | postgres=CTc/postgres
 template1 | postgres | UTF8     | C.UTF-8 | C.UTF-8 | =c/postgres          +
           |          |          |         |         |
postgres=CTc/postgres
(3 rows)
postgres=# \q
admin@myhome:~$
```

登录后，我们使用 `\l`（反斜杠，小写 L）列出所有可用的数据库，并使用 `\q`（反斜杠，小写 Q）退出 `psql` shell。一旦设置完成，我们可以回到脚本并再次尝试运行它：

```
admin@myhome:~$ ./run_backups.sh
pg_dump: error: connection to server on socket "/var/run/postgresql/.s.PGSQL.5432" failed: FATAL:  role "root" does not exist
```

PostgreSQL 中没有 root 角色，这是此时预期的错误。我们需要使用不同的角色连接到数据库。默认角色是 `postgres`，传递给 `pg_dump` 的选项是 `-U`，这与我们在 `psql` 中使用的相同。更新后，我们的脚本将如下所示：

```
#!/usr/bin/env bash
pg_dump -U postgres mydatabase > mydatabase.sql
```

最后一步是创建一个数据库并生成一些实际数据，以确保输出的 `sql` 文件不会为空。以下脚本将创建一个名为 `mydatabase` 的数据库，并创建两个包含随机数据的表：

```
CREATE DATABASE mydatabase;
\c mydatabase
CREATE TABLE random_data AS SELECT data_series, md5(random()::text) from generate_series(1,100000) data_series;
CREATE TABLE another_random AS SELECT data_series, md5(random()::text) from generate_series(1,100000) data_series;
```

`CREATE DATABASE` 这一行正在创建一个名为 `mydatabase` 的数据库。第二行表示我们正在连接到这个新数据库。接下来的两行以 `CREATE TABLE` 开头，分别创建表并使用内置的 PostgreSQL 函数填充数据。让我们将其分解为两个独立的查询——`SELECT` 和 `CREATE`：

```
SELECT data_series, md5(random()::text) from generate_series(1,100000) data_series;
```

这里发生了几件事：

+   `generate_series()` 函数正在创建一个从 1 到 100,000 的整数序列——这将生成表中的所有记录

+   `data_series` 关键字位于分号之前，命名了来自 `generate_series()` 函数的输出，因此它是我们打算创建的表中的实际字段名。

+   `random()` 函数生成一个介于 0 和 1 之间的值——即大于或等于 0 小于 1

+   `random()` 函数后的 `::text` 将此函数的输出转换为文本格式

+   `md5()` 函数将 `random()::text` 的输出进行哈希处理，使用 `md5` 算法，确保我们获得一个唯一的字符串，并运行的次数与 `generate_series()` 函数的输出数量一致（这里是从 1 到 100,000）

+   最后，`SELECT data_series, md5()` 正在生成一个包含两列（`data_series` 和 `md5`）的表，这两列的数据由两个函数生成

现在，回到 `CREATE TABLE`，有一部分叫做 `another_random AS` —— 这将从 `SELECT` 获取输出，并为我们创建一个表。

有了这些知识，我们可以创建一个 `sql` 脚本并使用 `psql` 执行它：

```
admin@myhome:~$ psql -U postgres < create_db.sql
CREATE DATABASE
You are now connected to database "mydatabase" as user "postgres".
```

要检查我们是否创建了某些内容并查看我们创建的数据，我们再次需要使用 `psql` 和对新数据库的 `SELECT` 查询：

```
admin@myhome:~$ psql (14.1)
Type "help" for help.
postgres=# \c mydatabase
You are now connected to database "mydatabase" as user "postgres".
mydatabase=# \dt
             List of relations
 Schema |      Name      | Type  |  Owner
--------+----------------+-------+----------
 public | another_random | table | postgres
 public | random_data    | table | postgres
(2 rows)
mydatabase=# select * from random_data ;
 data_series |               md5
-------------+----------------------------------
           1 | 4c250205e8f6d5396167ec69e3436d21
           2 | a5d562ccd600b3c4c70149361d3ab307
           3 | 7d363fac3c83d35733566672c765317f
           4 | 2fd7d594e6d972698038f88d790e9a35
--More--
```

前面的输出末尾的 `--More--` 表示还有更多记录要显示。你可以按空格键查看更多数据，或按 *Q* 键退出。

一旦你创建了一个数据库并填充了一些数据，你可以尝试再次运行我们的备份脚本：

```
admin@myhome:~$ ./run_backup.sh
admin@myhome:~$
```

没有错误，所以我们可能已经成功创建了完整的数据库转储：

```
admin@myhome:~$ ls -l mydatabase.sql
-rw-r--r--    1 root     root      39978060 Dec 15 10:30 mydatabase.sql
```

输出文件不是空的；让我们看看里面有什么：

```
admin@myhome:~$ head mydatabase.sql
--
-- PostgreSQL database dump
--
-- Dumped from database version 14.1
-- Dumped by pg_dump version 14.1
SET statement_timeout = 0;
SET lock_timeout = 0;
SET idle_in_transaction_session_timeout = 0;
```

在一些 `SET` 语句之后，你还应该找到 `CREATE TABLE` 和 `INSERT` 语句。由于输出内容较多，我没有提供完整的输出。

在本节中，我们学习了如何为脚本设置测试环境，并使得脚本能够创建数据库转储。在下一节中，我们将更多地关注错误处理，检查备份是否成功。

# 错误处理和调试

在运行备份脚本时，我们可能会遇到几种错误：数据库访问可能被阻止，`pg_dump` 进程可能被杀死，磁盘空间可能不足，或者任何其他错误导致我们无法完成完整的数据库转储。

在任何这些情况下，我们需要捕获错误并优雅地处理它。

此外，我们可能需要重构脚本，使其具有配置功能，使用函数，并进行调试。调试将非常有用，特别是在处理较大的脚本时。

让我们深入了解并开始添加一个函数：

```
#!/usr/bin/env bash
function run_dump() {
  database_name=$1
  pg_dump -U postgres $database_name > $database_name.sql
}
run_dump mydatabase
```

我们添加了一个 `run_dump` 函数，它接受一个参数，并将该参数的内容设置为一个名为 `database_name` 的局部变量。然后它使用这个局部变量将选项传递给 `pg_dump` 命令。

这将立即允许我们通过使用 `for` 循环来备份多个数据库，代码如下：

```
for dbname in mydatabase mydatabase2 mydatabase3; do
    run_dump $dbname
done
```

这个循环将创建数据库 `mydatabase`、`mydatabase2` 和 `mydatabase3` 的完整转储。备份将逐个完成，通过此函数。我们现在可以将数据库列表放入变量中，以使其更具配置性。当前脚本将如下所示：

```
#!/usr/bin/env bash
databases="mydatabase"
function run_dump() {
  database_name=$1
  pg_dump -U postgres $database_name > $database_name.sql
}
for database in $databases; do
  run_dump "$database"
done
```

现在，这个备份脚本变得更加复杂。我们需要注意接下来会发生的一些事情：

+   如果任何备份失败，脚本将继续运行

+   如果备份由于 `pg_dump` 无法访问数据库而失败，我们将覆盖之前的数据库转储

+   我们将在每次运行时覆盖转储文件

在脚本中，几项默认设置被认为是良好的实践需要覆盖。我们提到的第一个问题可以通过在任何命令返回一个与零（或 `true`）不同的值时终止运行来减轻。这意味着该命令执行时出现了错误。此选项名为 `errexit`，我们可以通过 `set` 命令覆盖它，`set` 是 Bash 内建命令。我们可以通过两种方式来实现这一点：

```
set -o errexit
set -e
```

这里是我们推荐使用的一些其他选项：

+   `set -u`：这将把我们在脚本中尝试使用的任何未设置的变量视为错误

+   `set -o pipefail`：当使用管道链式命令时，这个管道的退出状态将是最后一个命令的状态，如果该命令以非零状态结束，或者如果所有命令都成功执行（即退出状态为零），则为零（成功）。

+   `set -C` 或 `set -o noclobber`：如果启用，Bash 将不会使用重定向命令（例如我们在脚本中使用的 `>`）覆盖任何现有文件

一个非常有用的附加选项是 `set -x` 或 `set -o xtrace`，它使得 Bash 在执行每个命令之前打印该命令。

让我们看看一个简单 Bash 脚本的工作原理：

```
#!/usr/bin/env bash
set -x
echo "Hello!"
```

这是执行该脚本后的输出：

```
admin@myhome:~$ ./simple_script.sh
+ echo 'Hello!'
Hello!
```

让我们用推荐的 Bash 设置更新我们的备份脚本：

```
#!/usr/bin/env bash
set -u
set -o pipefail
set -C
databases="mydatabase"
function run_dump() {
  database_name=$1
  pg_dump -U postgres $database_name > $database_name.sql
}
for database in $databases; do
  run_dump "$database"
done
```

现在，让我们回到控制台测试它是否仍然按预期工作：

```
admin@myhome:~$ ./run_backups.sh
./run_backups.sh: line 12: mydatabase.sql: cannot overwrite existing file
```

我们已启用 `noclobber` 选项，它已防止我们覆盖先前创建的备份。我们需要重命名或删除旧文件才能继续。现在，我们还将启用 `xtrace` 选项，以查看正在执行的命令脚本：

```
admin@myhome:~$ rm mydatabase.sql
admin@myhome:~$ bash -x ./run_backups.sh
+ set -u
+ set -o pipefail
+ set -C
+ databases=mydatabase
+ for database in $databases
+ run_dump mydatabase
+ database_name=mydatabase
+ pg_dump -U postgres mydatabase
```

为了避免覆盖现有文件错误，我们可以采取以下三种方法之一：

+   在尝试运行备份之前删除旧文件，这将销毁以前的备份。

+   重命名上一个备份文件并添加当前日期后缀。

+   确保每次运行脚本时，转储文件都有一个不同的名称，例如当前日期。这将确保我们保留以前的备份，以防需要恢复到比上次完整备份更早的版本。

在这种情况下，最常见的解决方案是我们提出的最后一个——每次备份运行时生成不同的备份文件名。首先，让我们尝试获取一个带有本地日期和时间的时间戳，格式为 `YYYYMMDD_HHMM`，我们有以下选项：

+   `YYYY`：当前年份的四位数字格式

+   `MM`：当前月份的两位数字格式

+   `DD`：两位数格式的日期

+   `HH`：当前小时

+   `MM`：当前分钟

我们可以通过使用`date`命令来实现这一点。默认情况下，它将返回当前的日期、星期几和时区：

```
admin@myhome:~$ date
Fri Dec 16 14:51:34 UTC 2022
```

为了更改该命令的默认输出，我们需要使用格式字符传递一个日期格式字符串。

`date` 命令的最常见格式字符如下：

+   `%Y`：年份（例如，2022）

+   `%m`：月份（01-12）

+   `%B`：长月份名称（例如，January）

+   `%b`：短月份名称（例如，Jan）

+   `%d`：日期（例如，01-31，取决于某个月的天数）

+   `%j`：年份中的天数（例如，001-366）

+   `%u`：星期几（1-7）

+   `%A`：完整的星期几名称（例如，Friday）

+   `%a`：短星期几名称（例如，Fri）

+   `%H`：小时（00-23）

+   `%I`：小时（01-12）

+   `%M`：分钟（00-59）

+   `%S`：秒（00-59）

+   `%D`：以 mm/dd/yy 格式显示日期

要按照我们希望的格式格式化日期，我们需要使用格式字符 `%Y%m%d_%H%M`，并将其传递给 `date` 命令进行解释：

```
admin@myhome:~$ date +"%Y%m%d_%H%M"
20221216_1504
```

为了将输出字符串传递给脚本中的变量，我们需要在子 shell 中运行 `date`（由我们的 Bash 脚本执行的 Bash 进程）：

```
timestamp=$(date +"%Y%m%d_%H%M")
```

让我们将其放入脚本中，并使用`timestamp`变量来生成输出文件名：

```
#!/usr/bin/env bash
set -u
set -o pipefail
set -C
timestamp=$(date +"%Yum'd_%H%M")
databases="mydatabase"
function run_dump() {
  database_name="$1"
  pg_dump -U postgres "$database_name" > "${database_name}_${timestamp}".sql
}
for database in $databases; do
  run_dump "$database"
done
```

如果你在`pg_dump`命令行中看到变量之间有大括号，可能会想知道为什么我们需要它们。我们使用大括号确保变量名称在扩展为字符串时是正确的。在我们的案例中，我们防止了 Bash 尝试搜索一个不存在的变量名`$database_name_`。

现在，每次运行备份脚本时，它都会尝试创建一个带有当前日期和备份开始时间的文件。如果我们每天运行这个脚本，文件数量会随时间增加，最终填满我们的磁盘空间。所以，我们还需要让脚本删除旧备份——比如，删除 14 天及以上的备份。

我们可以通过使用`find`命令来实现这一点。让我们找到所有以数据库名开头，后跟下划线，且以`.sql`结尾的文件，这些文件的修改时间超过 14 天：

```
admin@myhome:~$ find . -name "mydatabase_*.sql" -type f -mtime +14
./mydatabase_20221107.sql
```

`find`命令有一种独特的语法，与其他命令行工具略有不同，所以我们来描述一下每个选项的含义：

+   `.`（一个点）：这是我们希望搜索文件的目录。点代表*当前目录*。

+   `-name`：此选项可以接受完整字符串或通配符，如`*`或`?`，用于查找文件名。它是区分大小写的。如果我们不确定正在查找的文件或目录是大写还是小写，可以使用`-iname`选项代替。

+   `-type f`：这表示我们在寻找一个常规文件。其他选项如下：

    +   `d`：目录

    +   `l`：符号链接

    +   `s`：套接字文件

    +   `p`：FIFO 文件

    +   `b`：块设备

    +   `c`：字符设备

+   `-mtime +14`：此文件的修改时间应早于 14 天。此选项还可以接受其他单位（秒，`s`；周，`w`；小时，`h`；天，`d`——如果未提供单位，则默认为天）。

要删除找到的文件，我们至少有两种选择：使用`-delete`选项或通过`-exec find`选项使用`rm`命令。让我们看看在这两种情况下的表现：

```
admin@myhome:~$ find . -name "mydatabase_*.sql" -type f -mtime +14 -delete
admin@myhome:~$ find . -name "mydatabase_*.sql" -type f -mtime +14 -exec rm -- {} \;
```

+   在这种情况下，更安全的选择是使用`-execdir`而不是`-exec`。它们的区别微妙但重要：`-exec`不会在找到的文件所在的同一目录中执行，而`-execdir`会，这使得它在边缘情况下更安全。

让我们解析一下`-exec`选项之后的内容：

+   `rm`：这是一个 CLI 工具，用于删除文件或目录。

+   `--`（双破折号）：这表示将从`stdin`获取参数，或`find`命令的输出。

+   `{}`：这将替代我们找到的文件名。

+   `\;`（反斜杠，分号）：这将允许多个命令由`-exec`执行。反斜杠是一个转义字符，可以防止该分号被解释为下一个命令的分隔符。`find`工具使用`;`或`+`来终止 Shell 命令，因此我们可以将其标记为`";"`、`\+`或`+`（不带引号）。

+   `-delete`选项用于删除文件，但它总是返回`true`，因此如果例如我们的脚本没有删除文件的权限，它会悄无声息地失败。在我们的脚本中使用它相对安全，所以我们会继续使用它。

现在，让我们将这个嵌入到我们的脚本中，看看它的最终版本：

```
#!/usr/bin/env bash
set -u
set -o pipefail
set -C
timestamp=$(date +"%Y%m%d_%H%M")
databases="mydatabase"
function cleanup_old_backups() {
  database_name="$1"
  find . -type f -name "${database_name}_*.sql" -mtime +14 -delete
}
function run_dump() {
  database_name="$1"
  pg_dump -U postgres "$database_name" > "${database_name}_${timestamp}".sql
}
for database in $databases; do
  cleanup_old_backups "$database"
  run_dump "$database"
done
```

在这里，我们添加了一个名为`cleanup_old_backups`的函数，它会在创建新转储之前运行，以释放一些空间来存放新的文件。我们在`run_dump`之前的 for 循环中调用了这个函数。这个脚本可以通过`cron 守护进程`或`systemd cron`服务自动运行；我们将在*第五章*，*在 Linux 中管理服务*中更详细地讨论这一点。

在本节中，我们了解了在 Shell 脚本中建议启用的选项以及如何启用调试选项。我们现在知道如何创建函数和循环。此外，我们也部分了解了 PostgreSQL 以及如何创建测试数据库。

下一章将带领我们深入了解各种 Linux 服务，以及如何使用`init`和`systemd`来管理它们。

# 总结

Shell 脚本是一种在 Linux 系统中自动化定期执行任务的常见方式。有时，它会演变成一个更大的系统，通过多个 Bash 脚本和 Python 程序连接起来，完成复杂任务，同时利用多个小任务以非常可靠的方式同时做一件事。

在现代系统中，你可能会看到和 Bash 一样多的 Python 脚本。

在这一章中，我们学习了如何创建可执行脚本，并且如何创建一个简单的备份脚本，处理错误并在每次运行时生成一个新的文件名。我们还添加了一个函数来删除旧的备份，以避免填满磁盘空间。此外，作为附带效果，我们学习了如何创建一个新的 PostgreSQL 数据库，并允许本地系统访问它。

在下一章，我们将学习如何创建 Linux 服务以及如何管理它们。

# 练习

尝试以下练习，以测试你对本章内容的理解：

1.  编写一个函数，列出所有数据库，并将该列表传递给我们创建的 for 循环。

1.  将日期时间戳转换为您选择的另一种格式。

1.  捕捉`find`函数可能返回的任何错误（例如，它无法删除文件）。

# 第二部分：日常 DevOps 工具

在第二部分中，我们将学习 Linux 内部机制。从管理服务和网络开始，我们将继续了解最常见的工具，如 Git 和 Docker。

本部分包含以下章节：

+   *第五章*，*在 Linux 中管理服务*

+   *第六章*，*Linux 中的网络*

+   *第七章*，*Git，通向 DevOps 的大门*

+   *第八章*，*Docker 基础*

+   *第九章*，*深入探讨 Docker*
