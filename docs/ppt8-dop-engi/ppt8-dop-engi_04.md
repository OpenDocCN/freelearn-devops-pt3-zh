# 4

# 变量和数据类型

本章将介绍 Puppet 如何处理变量，特别是 Puppet 在如何使用和声明变量方面与大多数声明性语言的不同之处。我们将查看用于定义变量值所能包含的内容以及如何与之交互的核心数据类型。然后，我们将探讨数据类型和变量如何允许我们在*第三章*中讨论的类接收外部数据并处理默认值。

数组和哈希将会详细讨论，包括如何声明它们、访问值以及如何使用操作对它们进行操作。`Sensitive` 数据类型将会展示，你可以使用它来保护日志和报告中的值，同时明确该数据类型的限制以及它无法保护的内容。我们还将涵盖抽象数据类型，并展示如何允许更复杂和灵活的变量与值的定义。本章的最后，我们将讨论变量的作用域和命名空间如何与变量一起工作。我们还会讨论变量的作用域，以及如何访问不同作用域的变量，以及哪些作用域可以访问哪些级别的数据。

本章我们将讨论以下主要内容：

+   变量

+   数据类型

+   数组和哈希

+   抽象数据类型，包括 `Sensitive`

+   作用域

# 技术要求

对于本章，你需要通过下载 [`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch04/params.json`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch04/params.json) 文件，然后在 `pecdm` 目录中使用以下命令，来准备一个包含 Windows 客户端和 Linux 客户端的 Puppet 服务器标准架构：

```
bolt --verbose plan run pecdm::provision –params @params.json
```

本章的各个部分将给出使用 `notify` 函数的示例，该函数输出到代理命令行。这些示例可以通过将所有代码放入清单文件中——例如，`example.pp`——并运行 `puppet apply example.pp` 在本地开发环境中执行。

另外，任何必需的变量都可以使用 `FACTER_variable_name` 的环境变量格式进行设置，并运行 `puppet apply –e '<example_code>'`。要运行其中一个子字符串示例，你可以运行以下代码：

```
export FACTER_example_string='substring'
puppet apply -e 'notify{ "${example_string[3]}": }'
```

# 变量

本节将讲解如何在 Puppet 中使用变量，以及这与其他过程性语言的区别。理解 Puppet 变量的关键是，它们在给定作用域内的编译过程中仅被赋值一次。在传统的过程性语言中，通常会在代码中广泛使用变量，在代码运行时收集当前状态，使用变量来追踪并在代码的不同阶段更新它，以便做出处理和决策。以下是一个简单的 PowerShell 脚本示例，它多次运行命令，并将输出添加到一个单一变量中。它通过使用`select-string`在用户代码目录中的`.sh`和`.pp`文件中查找包含`?`的文件来实现：

```
$Matches = Select-String -Path "$PSHOME\code\*.sh" -Pattern '\?
…
…
$Matches = $Matches + Select-String -Path "$PSHOME\code\*.pp" -Pattern '\?
```

由于所有的求值在目录开始时进行，并基于服务器发送过来的编译状态，Puppet 并不会执行这个状态检查。这样就提供了将服务器状态转化为所需状态所需的步骤。在 Puppet 中，我们为重复使用的内容（如文件路径）或条件逻辑（如`if`或`case`）分配变量。在这些情况下，必须选择一个值并进行赋值，具体取决于初始状态。

注意

我们在这里稍微简化了过程，因为现在有延迟函数可以在复杂性处理后运行。然而，这仍然不允许我们重新赋值。我们将在*第五章*中更详细地讲解这一点。

Puppet 变量的声明格式是：美元符号（`$`）后跟变量名，再加上等号（`=`）和要赋值的值。例如，一个名为`example_variable`，值为`'this is a value'`的变量应该像这样：

```
$example_variable = 'this is a value'
```

请注意，与资源不同，变量依赖于求值顺序，并且必须在代码中声明后才能调用。

注意

有几个被称为内置变量的变量，它们返回服务器信息。然而，由于这些更多涉及基础设施和环境，它们将在*第十章*和*第十一章*中详细讲解，在相关部分讨论。

## 命名

变量名是区分大小写的，可以包含大写字母、小写字母、数字和下划线，但必须以下划线（`_`）或小写字母开头。例外的是正则表达式捕获变量，这些变量仅使用数字命名，例如`$0`、`$1`等。我们将在*第七章*和*第十一章*中讨论它们，并将它们作为条件语句和节点定义的一部分使用。

注意

变量名以下划线开头将限制其仅限于局部作用域使用，具体细节将在本章末尾讨论。

## 保留的变量名

有一些内置变量是不能在代码中使用的，具体如下：

+   `$``facts`

+   `$``trusted`

+   `$``server_facts`

这些都是从 Facter 生成的内置变量，不能被使用或重新赋值。我们将在 *第五章*中详细讨论这些变量，以及它们提供的值。

正如我们在 *第三章*中所讲，`$title` 和 `$name` 变量是由类和定义的类型使用的，应该避免使用。

保留字的完整列表可以在 Puppet 文档中查看，链接为 [`www.puppet.com/docs/puppet/8/lang_reserved.html`](https://www.puppet.com/docs/puppet/8/lang_reserved.html)。

## 插值

Puppet 变量在调用时可以被评估并解析为其赋值，前提是没有使用引号，或者作为变量与我们数据中的双引号部分混合时使用。lint 检查强制执行的样式指南将确保在赋值中不包含变量时使用单引号，而仅包含变量时不使用引号。值和变量的混合可以通过双引号来书写。样式指南中会指出何时使用这种混合写法。linter 会检查确保它使用了大括号 `{}`。这可以通过下面的示例看到，在使用双引号进行插值时，`mixed_variable` 被赋值给变量声明：

```
$database_id = $dbname
$base_directory = '/opt'
$database_directory = "${base_directory}/database/${database_id}"
```

正如我们在上一章中所描述的，`notify` 函数可以用来检查变量的值：

```
notify{'debug variable':
  message => "The database directory is ${database_directory}"
}
```

本节讨论了 Puppet 变量如何与其他编程语言的变量不同，特别是在有状态性方面，以及它如何声明、访问和命名变量。

# 数据类型

Puppet 中的每个值都有一个数据类型；例如，在上一节中，变量被赋值为 `String` 类型。数据类型，当作为大写的不带引号的字符串使用时，如 `Integer`，可以用来指定类、定义的类型或 Lambda 中应该包含的参数，从而实现数据验证：

```
class example (
  String example_string = 'hello world',
  Integer example_integer = 1
) {
}
```

数据类型还可以用来比较变量的值、条件检查值，并根据结果执行不同的操作。例如，为了确认一个变量是否包含整数，可以使用以下匹配表达式。在这里，我们确认 `example_integer` 变量包含一个整数：

```
$example_integer =~ Integer
```

条件语句和比较将在 *第七章*中详细介绍。

下一部分将介绍最常用的 Puppet 核心数据类型。不幸的是，Puppet 没有类似于 `puppet describe` 命令的数据类型，所以所有参考必须来自于网络和 GitHub 文档，具体请见[`www.puppet.com/docs/puppet/8/lang_data_type.html`](https://www.puppet.com/docs/puppet/8/lang_data_type.html)。如果您使用的是来自 Forge 模块提供的类型，这将在*第八章*中详细介绍，文档应该在模块的参考页面中。各种函数可与数据类型一起使用，但这里不会讨论此内容。我们将在*第五章*中详细讨论函数。

## 字符串

字符串是 Puppet 中最常用的数据类型，正如在*第一章*中讨论的那样，最早的 Puppet 中仅使用字符串类型。字符串是任何长度的非结构化文本，以 UTF-8 编码。有四种方式可以在 Puppet 中声明字符串：

+   未加引号

+   单引号

+   双引号

+   Heredoc

### 未加引号的字符串

未加引号的字符串是以小写字母开头的单词，只包含字母、数字、连字符 (`-`) 和下划线 (`_`)，且不能是保留字。保留字通常是诸如 class 之类的关键字或其他语言函数；完整列表可以在[`www.puppet.com/docs/puppet/8/lang_reserved.html#lang_reserved_words`](https://www.puppet.com/docs/puppet/8/lang_reserved.html#lang_reserved_words)查看。

未加引号的字符串用于接受有限单词集的资源属性，例如 Puppet 服务资源类型，它的 `ensure` 属性只能接受 `running` 或 `stopped`：

```
service { 'defragsvc':
  ensure => stopped
}
```

### 单引号字符串

单引号字符串可以包含多个单词，但如前所述，不能进行变量插值。然而，它们可以包含换行符和使用反斜杠 (`\`)、转义反斜杠或单引号 (`'`) 的转义序列。这允许在字符串本身内使用单引号，并在字符串末尾使用反斜杠。此外，可以通过按下 *Enter* 键来实现换行符。

以下示例展示了 `sed_command` 变量，其中包含作为 `sed` 命令一部分所需的单引号，以在单引号字符串中进行转义，然后是 `install_dir` 变量，表示带有结尾反斜杠的 Windows 文件路径：

```
$sed_command = '/usr/bin/sed -i \'s/old/new/g\''
$intall_dir = 'c:\Program Files(x86)\exampleapp\\'
```

### 双引号

双引号字符串可以完全插入变量，并且支持更多可用的转义字符。除了单引号、反斜杠和转义字符，双引号还可以解释以下内容：

+   `\``n`: 换行符

+   `\r`: 回车

+   `\``s`: 空格

+   `\``t`: 制表符

+   `\$`: 美元符号，用于防止变量插值

+   `\uXXXX`: 其中 `xxxx` 是表示 Unicode 字符的四位十六进制数字

+   `\u{X}`: 其中 `X` 是两个到六位数字之间的十六进制数字，位于大括号 `{}` 内

+   `\"`: 双引号

+   `\r\n`: Windows 换行符

和单引号一样，换行符也可以在文本中使用（即只需按下 *Enter* 键）。

注意

如果在双引号中使用反斜杠而没有进行转义，且后面没有有效的转义字符，它将继续处理并将其视为普通字符，但会在日志中显示以下信息：*警告：未识别的转义序列*。以下是一个示例：

`警告：未识别的转义序列 '\T'` `位于 C:/Users/david/code/test.pp:1:50`

这通常影响使用双引号的 Windows 用户路径。

一个简单的双引号字符串示例，使用换行符和制表符（这些在 Makefile 内容的语法中非常重要），如下所示。这会创建一个字符串，然后可以在文件内容中使用：

```
$make_file_content = "hello:\n\techo \"hello world\""
file '/home/david/makefile' : {
  content => $make_file_content
}
```

### Heredoc

Puppet 对 heredoc 的实现涉及使用标签来指示 heredoc 文件内容的开始和结束。起始标签通常由以下元素组成：

+   `'@('`

+   一个字符串，称为结束文本，可以用双引号括起来以启用插值

+   一个可选的转义开关（或多个开关），以斜杠（/）开头，用于在文本中启用转义开关

+   一个可选的冒号（`:`），后跟语法名称检查

+   `')'`

要在 Puppet 中使用 heredoc，内容应在起始标签后的行中输入，保持所需的精确格式。heredoc 的结束由结束标签表示，结束标签应包括以下元素：

+   一个可选的竖线（`|`），表示应该从文本行中删除多少缩进

+   一个可选的短横线（`-`），它会从 heredoc 中移除最后的换行符

+   与起始标签中使用的结束文本标签相同

Puppet heredoc 中的结束文本是一个字符串，可以由大小写字母、数字和空格组成，但不能包含换行符、斜杠、冒号或圆括号。默认情况下，heredoc 的内容不会解析转义字符，因此如果需要转义开关，必须声明它们。以下转义开关可用，并且与双引号字符串的转义序列相同，但不需要双引号（因为它们在 heredoc 中没有特殊含义）：

+   `n`: 换行符

+   `r`: 回车符

+   `t`: 制表符

+   `s`: 空格

+   `$`: 美元符号，用于防止在双引号文本中进行插值

+   `u`: Unicode 字符

+   `L`: 换行符或回车符

+   `\:`: 所有之前提到的转义序列均可用

当选择任何转义序列时，可以使用 `\\` 来转义反斜杠。

默认情况下禁用变量插值，因此，如前所述，结束文本应在需要时用双引号括起来。

语法检查适用于各种内容，例如通过 `pp` 的 Puppet 清单或通过 `ruby` 的 Ruby 文件：

```
@(END:pp)
@(END:ruby)
```

语法检查仅在没有启用变量插值的情况下运行；如果输入了 Puppet 不支持的类型，它将被忽略。有关可用语法检查器的详细信息，可以在 Puppet 规范中找到，该规范还包含有关创建自定义语法检查器的详细内容。

Heredoc 声明可以放置在任何可以声明字符串的地方，因此，举个例子，`exec` 命令中的长命令可以如下声明：

```
exec { 'create databases':
  command => @("Database Commands"/L)
    sudo -u postgres psql \
    -c "CREATE DATABASE ${database1} ENCODING 'utf8' LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8'" \
    -c "CREATE DATABASE ${database2} ENCODING 'utf8' LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8'" \
    -c "CREATE DATABASE ${database3} ENCODING 'utf8' LC_COLLATE 'en_US.UTF-8' LC_CTYPE 'en_US.UTF-8'"
    |-"Database Commands"
```

本书建议谨慎使用 heredocs。在 `exec` 中执行长命令时，如前面示例所示，这种做法可能合适，但特别是对于文件内容，通常会使代码变得杂乱并且容易混淆，因此最好将其放在模板中，如*第七章*所讲，或作为模块中的文件，如*第八章*所述。关于如何最好地存储数据的问题将在*第九章*中讨论。

### 访问变量中的子字符串

在 Puppet 中调用字符串，最简单的方法是使用 `$` 符号，后跟变量名。但是，如果变量名包含无效字符，如空格，Puppet 将认为变量名已结束。因此，为了确保在字符串中正确地插入变量，最好将变量名括在花括号 `{}` 中。

要访问字符串中的特定字符或子字符串，Puppet 允许你使用 `[<start index>, <stop position>]` 指定一个索引范围，这也支持负数索引，从字符串末尾倒数或改变返回字符的顺序。例如，下面的代码将一个名为 `'example_string'` 的变量设置为 `'``substring'` 字符串：

```
$example_string = 'substring'
```

可以使用各种组合；例如，可以通过从开头取索引（例如 `3`）来调用单个字符，从而返回 `s`（我们从 `0` 开始）。

要从字符串变量中提取单个字符，在 Puppet 中，你可以指定从 `0` 开始的字符索引。例如，要提取字符串变量的第三个字符，你可以使用索引 `3`（因为索引从 `0` 开始）。在 Puppet 中，这可以如下表达：

```
notify { "${example_string[3]}" :}
```

这将返回索引 `3` 处的字符，在本例中为 `'s'`。

负数索引可以从字符串末尾开始，使用 `-6` 返回相同的 `s` 字符：

```
notify { "${example_string[-6]}" :}
```

要在 Puppet 中提取字符串变量的特定部分，可以使用方括号表示子字符串的起始索引和结束位置。例如，如果你有一个名为 `'example_string'` 的字符串变量，其值为 `'substring'`，并且你想提取一个从第三个字符开始并包含接下来的五个字符的子字符串，你可以在 Puppet 中使用以下语法：

```
notify { "${example_string[3,6]}" :}
```

这将返回从索引 `3` 开始（对应于 `'substring'` 中的字母 `'s'`）并包含接下来的五个字符，在本例中为 `'string'`。

要提取从负索引位置开始的子字符串，可以为停止位置指定负值。例如，要提取从倒数第四个索引位置开始并包含接下来的三个字符，可以使用以下语法：

```
notify { "${example_string[-4,-1]}" :}
```

这将返回从倒数第四个字符开始的子字符串（它对应于`'substring'`中的字母`'t'`），并包括接下来的三个字符，在这种情况下是`'tri'`。

最后，要提取一个从负索引位置开始并包括一定数量正整数的子字符串，可以使用以下语法：

```
notify { "${example_string[-4,4]}" :}
```

这将返回从倒数第四个字符开始的子字符串（它对应于`'substring'`中的字母`'t'`），并包括接下来的四个字符，在这种情况下是`'ring'`。

这种子字符串操作在需要将包名、应用程序版本或其他一致的名称字符串拆分为不同变量时非常有用。作为一个更实际的例子，一个组织有主机名，它们以位置代码开始，并包含角色、环境和服务器 ID：

```
$hostname = flkoracprd00034
$location = $hostname[0,3]
$role =$hostname[3,3]
$environment = $hostname[6,3]
$id = $hostname[-5,5]
```

### 字符串数据类型参数

当将参数的类型设置为字符串时，使用大写关键字`String`，并可选地指定字符串的最小和最大长度：

```
String[<Minimum length>, <Maximum Length>] $variable_name
```

最小值默认为 0，最大值默认为无限。如果要隐式使用默认值，可以使用默认的未加引号字符串关键字。

让我们来看一个名为`database`的类，它接受一个四个字符的数据库 ID 字符串，一个长度在六到八个字符之间的用户名（如果未提供，默认值为`dbuser`），以及一个长度不限的描述：

```
class 'database': {
  String[4,4] database_id,
  String[6,8] username = 'dbuser' ,
  String description,
}:
```

## 数字

本节将介绍 Puppet 用于数字的两种类型：整数和浮动点数。我们还将看看可以对它们执行哪些算术操作，数字如何与字符串之间转换，以及这些类型的变化。

这两种类型的数字都没有使用引号进行声明。在这里，字母的大小写不重要。以下模式是可用的：

+   `–`（假设为正数）

+   数字字符（八进制数字以`0`开始）

+   `–`（假设为正数）* `0x`或`0X`（大小写不重要）* 数字字符与大写或小写字母的混合* `–`（假设为正数）* 数字字符（使用-1 到 1 之间的数字时需要`0`）* 小数点* 数字字符* 可选的`e`或`E`，后面跟着数字（用于科学浮点数）

以下是一些简单且恰当命名的例子，涵盖了上述类型中的每一项：

```
$integer = 42
$negative_integer = -84
$float = 32.3333
$scientific float = 3e5
$octal = 0678
$hex = 0x
```

需要注意的是，八进制或十六进制数字不能表示为浮点数，如果尝试这样做，将会导致错误，因为它不是有效的八进制或十六进制数字。

### 算术运算符

我们无法对变量执行重新赋值操作，但可以基于已赋值变量之间的运算来赋予新变量。以下表达式可以用于变量之间：

+   `+`: 加法

+   `-`: 减法

+   `/`: 除法

+   `*`: 乘法

+   `%`: 取余，表示左边除以右边的余数

+   `<<`: 左移

+   `>>`: 右移

左移和右移运算较为陌生，需要进一步解释。左移是将第一个变量乘以 2 的第二个变量次幂。例如，`5 << 3` 等同于 5 * 23，结果为 40。

右移是将第一个变量除以第二个变量的 2 的幂。例如，`32 >> 2` 等同于 32 / 22，结果为 8。

注意

对于左移和右移，浮点数将向下舍入为整数。

此外，负号（`-`）可以用作前缀来取反变量，并且括号可以用来管理运算的优先级，在这里 **括号、次序（指数/幂或根号）、除法、乘法、加法和减法** (**BODMAS**) 规则适用。移位运算在这个优先级中本质上被当作乘法和取余运算来处理。

整数与浮点数之间的任何运算都会得到浮点数，且对整数的运算会导致浮点数向下舍入为整数。

以下是使用这些运算符的一些示例：

```
$a = 5
$b = 3
$addition = $a + $b
$subtraction = $a - $b
$division = $a / $b
$multiplication = $a * $b
$modulo = $a % $b
$shift_left = $a << $b
$shift_right = $a >> $b
$negate = -$a
```

为了进一步展示括号如何强制执行 BODMAS 规则，以下示例将等于负 40：

```
$bodmas_example = ($a + $b) * -$a
```

### 字符串到数值的转换

如果字符串用于数值运算，它会自动转换，但在其他任何上下文中则不会发生此情况。要将字符串转换为数字，可以声明对象为整数、浮点数或数值（我们将在 *抽象数据类型，包括敏感数据* 部分介绍数值对象）。转换的示例是将字符串 `1` 转换为整数，字符串 `1.1` 转换为浮点数：

```
$string_integer='1'
$string_float='1.1'
$converted_integer=Integer($string_integer)
$converted_float=Float($string_float)
```

### 数值到字符串的转换

数值类型在与字符串插值时会自动转换为字符串；这种自动转换使用的是十进制表示法。`String` 对象声明也可以用于转换，示例如下：

```
$string_from_integer = String(342)
```

### 整数数据类型

当将参数类型设置为整数时，使用大写的 `Integer` 关键字，并可以指定整数的最小值和最大值：

```
Integer[<Minimum Value>, <Maximum Value>]
```

默认值在技术上是负无穷大和正无穷大，但由于 Puppet 使用 64 位带符号整数，这个范围大约是从 −9,223,372,036,854,775,808 到 9,223,372,036,854,775,807。

### 浮点数据类型

当将参数类型设置为浮点数时，使用大写的 `Float` 关键字，并可以指定整数的最小值和最大值：

```
Float[<Minimum Value>, <Maximum Value>]
```

默认值在技术上是正无穷大和负无穷大，但在实际操作中，这个范围是 Ruby 实现中双精度浮点数的 -1.7E+308 到 +1.7E+308。

例如，考虑以下代码块，它定义了`Class application::filesystem`，用于在已知的`100`到`10000`的范围内为一个卷组分配百分比：

```
Class application::filesystem (
Float[0.1, 99.9] percentage_application,
Integer[100, 10000] volume_group_size
) {
}
```

## `undef`

`undef`在 Ruby 中被视为等同于 nil，表示没有给变量赋值。默认情况下，`strict_variables`设置为`false`，这意味着未声明的变量默认值为`undef`。在*第十章*中，我们将看到可以在`puppet.conf`配置文件中设置此项。

作为一个简单的例子，以下代码会通知`Print`，`test1`尚未声明：

```
  notify {"Print $test1":}
```

`undef`数据类型的唯一值是未加引号的`undef`，并且它本身不用于参数数据类型。这是因为强制要求值的缺失没有意义。

在本章的*抽象数据类型，包括敏感类型*部分，我们将看到如何接受`undef`值作为参数的一部分，作为可行选项的选择。

当插入到字符串中时，`undef`会被转换为空字符串(`''`)。

调用

在*第五章*和*第八章*中，我们将学习一些函数，如`delete_undef_values`和`filter`，它们可以用于修剪数组和哈希中的`undef`值。

## 布尔值

Puppet 中的布尔值代表`true`或`false`，在*第七章*中，当我们查看`if`/`case`语句时，你会发现所有 Puppet 的比较都会返回布尔类型。布尔变量应该仅包含一个未加引号的`true`或`false`值。因此，这使得数据类型非常简单，没有参数——只有大写的`Boolean`关键字。

举个例子，下面的代码是一个`exampleapp`类，它有一个管理用户的参数，默认设置为`true`，并且有一些硬编码的变量：

```
Class exampleapp (
  Boolean manage_users = true
) {
  $install_ssh = true
  $install_telnet = false
}
```

### 转换

在大多数情况下，除非显式指定了数据类型，否则会自动将其转换为布尔值。例如，在`if`语句中，变量可以像布尔值一样使用，只需写`$variable_name`。然而，自动转换可能会让人困惑，因为只有`undef`会被转换为`false`。这意味着`'false'`字符串、空字符串(`''`)、整数`0`和浮点数`0.0`都会被转换为`true`。

在使用布尔声明时，空字符串无法转换，`undef`也一样，而`'false'`字符串、整数`0`或浮点数`0.0`会转换为`false`。

由于这可能会让人困惑，因此最好使用来自`puppetlabs-stdlib`模块的`num2bool`和`str2bool`函数，这将在*第八章*中讲解。

## 正则表达式

`regexp`类型不同于我们迄今为止看到的类型。它表示 Puppet 中的有效正则表达式，正则表达式由基于 Ruby 正则表达式实现的正斜杠之间的表达式表示：[`ruby-doc.org/core/Regexp.html`](http://ruby-doc.org/core/Regexp.html)。

正则表达式的使用将在*第七章*中更详细地介绍，届时它将得到更实际的应用。不过，值得注意的是，本章稍后会介绍几个包含多个类型的抽象类型，包括`regexp`。

## 实验室

在上一章中，创建了一个合并的`all_grafana`清单，并提供了一个解决方案，详见[`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch04/all_grafana.pp`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch04/all_grafana.pp)。调整此文件，使其包含在一个名为`all_grafana`的类中，并且不再使用 Facter，而是使用参数。

这些参数应包括以下内容：

+   **源下载**：一个默认指向[`dl.grafana.com/enterprise/release/grafana-enterprise-8.4.3-1.x86_64.rpm`](https://dl.grafana.com/enterprise/release/grafana-enterprise-8.4.3-1.x86_64.rpm)的字符串变量

+   **端口**：服务监听的端口，类型为整数

+   **服务启用**：通过布尔值

要实现类的赋值，编写一个类声明，分配变量以确保该类包含在目录运行中。当你在清单上运行`bolt`时，它将确保你已经包含了变量。解决方案可参考[`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch04/all_grafana.pp`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch04/all_grafana.pp)。

# 数组和哈希

本节将涵盖 Puppet 中的两种核心数据集合：数组和哈希。你将学习如何创建、访问以及执行操作，将值操作到一个新的变量中。

## 分配数组

Puppet 数组是通过用方括号括起来的以逗号分隔的值列表来创建的。最后一个元素后可以添加可选的逗号，但本书不推荐这样做，以保持样式一致。例如，名为`example_array`的数组，包含`first`、`second`和`third`字符串，可以如下声明：

```
$example_array = ['first','second','third']
```

数组可以包含任何数据类型，也可以包含不同类型的混合数据。Puppet 变量不能在单独的值上重新赋值，也不能通过添加或移除值等任何其他操作进行修改。以下代码演示了如何将`mixed_example_array`数组赋值为整数`1`、来自`example_boolean`变量的布尔值`false`，以及`example`字符串：

```
$example_boolean = false
$mixed_example_array = [ 1, $example_boolean , 'example']
```

数组也可以为空，方括号中没有任何内容，`[]`。它们不会被识别为 `undef`，而是一个空数组。通常不会直接声明一个空数组；这通常是由于插值变量和运算符导致数组变为空。

## 访问数组索引

要访问数组变量的元素，可以通过索引指定特定的元素，这将返回该元素。例如，要获取 `example_array` 中第二个索引位置的第二个字符串并将其赋值给变量，可以使用以下代码：

```
$example_array = ['first','second','third']
$second_index = $example_arrary [1]
```

以下代码展示了一个 `notify` 资源，它输出一个字符串，插入了从 `example_array` 中倒数第三个元素的负数：

```
notify{ "The first element is ${example_array[-1]}"
```

访问不存在的元素将返回 `undef`。你不能在方括号和变量名之间放置任何空格；否则，它会被解释为一个变量，方括号会被分开。

## 访问数组的子集

访问数组子集时，使用第二个数字表示停止点。这与处理子字符串的方式不同。对于数组，正数表示要返回的元素数量。例如，使用计数位置 `1` 将返回一个包含单一元素的数组。要从 `example_array` 中提取一个只包含 `'second'` 元素的子数组，你可以使用以下代码：

```
$sub_array =  example_array[1,1]
```

这将把 `['second']` 子数组赋值给 `$``sub_array` 变量。

选择超过数组长度的长度将仅返回可用的元素。

负长度将从数组的末尾开始倒数。重要的是，与访问子字符串不同，不能通过越过起始索引来反转顺序；这将仅返回一个空数组。在下面的示例中，`negative_sub` 数组将返回整个数组，因为它的起始位置是 `0`，结束位置是数组末尾的第一个元素。`empty_sub_array` 变量将被赋值为空数组，因为结束位置会在起始位置之前。`second_element_array` 变量将被赋值为一个包含第二个元素的数组：

```
$negative_sub_array = example_array[0, -1]
$empty_sub_array = example_array[1, -3]
$second_element_array = example_array[1, -2]
```

## 嵌套数组

可以通过在数组内插入数组值来声明嵌套数组，插入的次数根据需要决定。然后，可以通过使用多个方括号来访问所需的级别。例如，如果创建一个嵌套数组，第一项是一个字符串，第二项是一个包含三个字符串的数组，第三项是一个字符串，尝试将第一项作为嵌套元素访问时，将返回字符串 `i`，因为索引为 `0` 的元素返回的是字符串。然后，在 `first` 字符串上使用第二组方括号。

`nest_second` 变量返回 `nest_second` 字符串，因为它返回了索引为 `1` 的嵌套数组；然后，使用第二组方括号访问第二个元素：

```
$nested_array= ['first',['nest_first','nest_second','nest_third'],'third']
$sub_string = $nested_array[0][1]
$nest_second = $nested_array[1][2]
```

要在数组中插入嵌套变量，必须用大括号包围变量名和方括号。例如，以下 `notify` 资源将打印 `nested_array` 的第一个元素：

```
notify {"Print ${nested_array[1][0]}":}
```

可以在嵌套括号内使用子集方法，但这可能会产生令人困惑且难以跟踪的访问方式，因此本书中不推荐这种风格。

## 数组操作符

一旦数组被赋值，就不能再直接操作它，但操作符可以操作数组的内容，以便分配新的数组变量。以下是可用的操作符：

+   `<<`：追加

+   `+`：连接

+   `-`：删除

+   `*`：扩展（Splat）

### 追加（Append）

追加（Append）接受任何类型的值，并将其作为新元素添加到数组的末尾。这包括将一个数组作为嵌套数组添加。要合并两个数组，必须使用连接操作符（`+`）。为了演示这一点，我们来看一个包含整数 `1` 和 `2` 的数组，它追加 `3` 形成一个新数组。这将产生一个新数组 `[1,2,'three']`；将 `[3,4]` 数组追加到 `example_array` 会形成一个新的嵌套数组 `[1,2,[3,4]]`：

```
$example_array=[1,2]
$new_array=$example_array << 'three'
$append_nest=$example_array << [3,4]
```

### 连接（Concatenate）

`Concatenate`（连接）接收一个数组，并本质上将其内容与另一个数组结合。如果第一个值不是数组，编译器会假定这是一个数值操作符。对于数字、字符串、布尔值和正则表达式，它的工作方式基本与追加相同，都会将值添加到数组的末尾。为了实现嵌套数组的条目，你必须提供一个嵌套数组。因此，举几个例子，`combined_1` 将变成一个数组 `[1,2,1]`，`combined_2` 会被赋值为 `[1,2,1,2]`，而 `combined_3` 会得到一个嵌套数组 `[1,2,[1]]`：

```
$combined_1 = $example_array + 1
$combined_2 = $example_array + [1,2]
$combined_3 = $example_array + [[1,2]]
```

如果需要连接一个哈希，它会被转换成数组，除非它已经变成只有一个哈希元素的数组。例如，在下面的代码中，转换后的变量将被赋值为一个嵌套数组，元素为 `test` 和 `value`，并赋值为 `[1,2,[test,'value']]`，而 `nested_hash` 变量则会添加一个嵌套哈希，赋值为 `[1,2,{test => '``value'}]`：

```
$converts = $example_array + {test => 'value'}
$nested_hash =$example_array + [{test => 'value'}]
```

### 删除（Remove）

删除操作会在从源数组中删除所有匹配元素后分配一个新数组。第一个变量必须是一个数组；否则，它将被视为数值操作符。对于第二个变量，如果是数字、字符串、布尔值或正则表达式，它会检查第一个数组中的每个元素，并在找到匹配时删除它。例如，从 `another_example_array` 中删除字符串 `one` 会匹配第一个元素和第三个元素并将其删除，但不会删除嵌套数组中的第一个元素，结果会将 `['two','three','four','three',['one','three','four']]` 赋值给 `remove_string` 变量：

```
$another_example_array = ['one','two','one','three','four','three',['one','three','four']]
$remove_string = $another_example_array – 'one'
```

当第二个变量是数组时，它会遍历数组中的每个元素，像直接呈现一样删除它们，就像我们之前的例子。在这个例子中，它将按照之前的例子移除`one`，然后继续搜索匹配的`three`和`four`字符串，移除第四、第五和第六个元素，同时将`['two',['one','three','four']]`赋值给`remove_array`变量：

```
$another_example_array = ['one','two','one','three','four','three',['one','three','four']]
$remove_array = $another_example_array – ['one','three','four']
```

当嵌套数组作为第二个变量时，它会匹配与相同数组的所有元素并将其移除。所以，在这个例子中，`remove_nested_array`变量将被赋值为`['one','two','one','three','four','three']`：

```
$remove_nested_array = $another_example_array – [['one','three','four']]
```

与连接操作一样，哈希必须放置在数组中；否则，它们将删除翻译后的嵌套数组中的任何匹配元素。

### 展开符

展开符与其他运算符不同，它们用于将数组作为函数调用的参数，提供以逗号分隔的列表。这在条件语句和选择器语句中都适用。使用数组展开符将在*第五章*和*第七章*中详细讲解。

## 数组数据类型

当设置参数的数据类型为数组时，必须使用大写的`Array`关键字，并指定数组元素的数据类型、数组的最小大小和最大大小：

```
Array[<Data Type>, <Minimum Size>, <Maximum Size>]
```

数据类型的默认值是数据，这将在本章的*抽象数据类型，包括敏感数据*部分中讲解，但这意味着数字（包括整数和浮点数）、字符串、布尔值和正则表达式，以及这些类型的数组和哈希都适用。如果选择更具体的数据类型，例如`String`，则期望数组中的每个元素都包含一个字符串。在*抽象数据类型，包括敏感数据*部分中，还将讲解其他混合类型，这些类型提供了更多的灵活性。

最小大小为 0，最大大小为无限。

例如，`database`类可以接受一个`db_uids`变量，其中至少期望一个元素，但最多可以包含六个元素。`user_names`变量可以是一个空数组或最多五个元素，但大多数情况下只包含字符串。最后，`extra_flags`变量是一个具有默认值的数组，因此它可以是一个空数组，大小可无限，且内容与数据类型匹配：

```
class 'database': {
  Array[default,1,6] db_uids,
  Array[string,0,5] user_names,
  Array extra_flags,
}
```

## 赋值哈希

哈希作为以逗号分隔的键值对书写，键值对之间用`=>`分隔，列表被大括号`{ }`包围。最后一对键值后可以加上尾随逗号，但本书不推荐这种样式。例如，以下哈希对可以用来为`make`键赋值为`skoda`字符串，`model`键赋值为`rapid`字符串，`year`键赋值为`2014`整数：

```
$my_car = { make => 'skoda', model => 'rapid', year => 2014 }
```

为了格式化风格，通常每个键都会占用一行，以确保键的开始对齐，箭头也能对齐。本书推荐在编写数组时，使用一个新的空行来结束大括号，并将其与起始大括号对齐：

```
$my_car = { make  => 'skoda',
            model => 'rapid',
            year  => 2014
          }
```

哈希的键和值可以是任何类型，但键通常应该是字符串类型，其他类型很少有实际意义。就像数组一样，哈希在 Puppet 中是变量，且只能赋值一次，除非重新分配一个新的哈希，否则无法对其进行修改。

注意

Puppet 只能将字符串类型的哈希键序列化到目录中。因此，无法将非字符串键的哈希分配给资源属性或类参数。

## 访问哈希值

类似于数组，哈希值可以通过方括号和键值来访问。举个例子，下面的代码将打印出`rapid`值：

```
notify {"Print ${my_car[model]}":}
```

## 嵌套哈希

与数组一样，通过在哈希中声明一个哈希，可以创建一个嵌套哈希，这样就可以通过链式键来访问。以下示例展示了一个包含`packages`和`services`键的变量包列表。`packages`键包含`httpd`键，其值为字符串`latest`，以及`cowsay`键，其值为浮动值`4.0`。`services`键包含`httpd`键，其值为字符串`running`，以及`nginx`键，其值为字符串`stopped`：

```
$package_list = { packages  => { httpd  => 'latest',
                          cowsay => 4.0
                        }
                  services => { httpd => 'running',
                                nginx => 'stopped'
                              }
                 }
```

为了打印出嵌套的两个`httpd`键，可以声明一个`notify`资源，如下所示：

```
notify {"Print ${package_list[packages][httpd]} ${package_list[services][httpd]}":}
```

## 哈希运算符

哈希有两个运算符——合并（`+`）和移除（`-`）。合并可以通过将键值对添加到现有的哈希中来分配一个新的哈希，而移除则通过从现有哈希中删除键值对来分配一个新的哈希。

### 合并

合并通过将一个哈希变量、一个`+`符号和一个具有偶数个值的哈希或数组来完成。请注意，合并时如果要添加的键已经存在，它将不会被重复添加。在下面的示例中，将一个包含`database`键和字符串`oracle`、`version`键和整数`11`的哈希与包含`web_server`键和字符串`httpd`、`version`键和`12`值的`app_web`哈希合并，最终会得到`combined_app`变量，包含`database`键值对和`web_server`键值对。然而，`app_web`中的`version`键将被忽略，因为`app_db`中已经存在该键：

```
$app_db    = { database => 'oracle', version = > 11}
$app_web = { web_server => 'httpd', version => 12 }
$combined_app = $app_db + $app_web
```

### 移除

删除操作符接受一个哈希变量，一个`–`符号，以及一个哈希、一个键的数组或一个单一的字符串键。如果提供哈希，则哈希中的值不重要，因为删除操作只是删除匹配的键。在以下示例中，可以看到一个`software_versions`哈希，其中包含`oracle`键和整数`11`，`httpd`键和值`12`，以及`cowsay`键和值`9`。当删除一个键以创建`no_cowsay`变量时，`cowsay`和`9`的键值对被删除。当`only_cowsay`被赋值时，要删除的哈希中`oracle`和`httpd`的值不重要，它将简单地删除键值对。而对于`only_oracle`变量，删除一个数组将使删除操作符遍历每个匹配的键并删除匹配项：

```
$software_versions = { oracle => 11, httpd => 12, cowsay => 9}
$no_cowsay = $software_versions – cowsay
$only_cowsay = $software_versions – { oracle => 'anything' , httpd => 'anything' }
$only_oracle = $software_versions – [httpd,cowsay]
```

## 哈希数据类型

哈希数据类型接受可选的键类型和值类型；如果指定了键类型，则必须指定值类型。可以为键对的数量指定最小和最大大小：

```
Hash[<Key type>, <Value type>, <Minimum size>, <Maximum size>]
```

例如，以下类具有一个`tunables`参数，它必须包含一个包含`1`到`10`个键值对（字符串和整数）的哈希：

```
Class kernel_overrides (
  Hash[String,integer,1,10] tunables
)
```

## 混合哈希和数组

由于哈希键值或数组值可以是任何数据类型，因此可以进行嵌套操作。但应注意不要让结构过于复杂。

以下示例显示了包含`nfs_share_servers`哈希的`server_cmdb`哈希，其中`prod`和`dev`键包含字符串数组：

```
$server_cmdb = {
  'nfs_share_servers => {
     prod =>  ['prdnfs01','prdnfs02','prdnfs02']
     dev => [ 'devnfs01','devnfs02,'devnfs03']
  }
}
```

要访问第一个`prod`数组的第三个值`prdnfs02`，可以执行以下调用：

```
$server_cmdb[nfs_share_servers][prod][2]
```

## 实验

为了练习我们所讲解的内容，编写一个类，该类接受一个软件包数组，并使用哈希定义的标准参数来安装提供者和版本的包。记得根据之前的实验声明类中的变量。例如，你可以安装最新版本的 RubyGems，包括 webrick、puma 和 sinatra。建议的解决方案可以在[`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch04/packages_array_hash_paramters.pp`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch04/packages_array_hash_paramters.pp)中找到。

# 抽象数据类型，包括敏感数据

抽象数据类型为你提供了灵活性，可以混合核心数据类型进行参数强制和特定模式的实现，还能在参数检查方面提供一些更高级的功能。抽象数据类型有很多种，所以下面的部分将覆盖最常用的几种。其他类型可以在[`github.com/puppetlabs/puppet-specifications/blob/master/language/types_values_variables.md`](https://github.com/puppetlabs/puppet-specifications/blob/master/language/types_values_variables.md)和[`www.puppet.com/docs/puppet/8/lang_data_abstract.html#variant-data-type`](https://www.puppet.com/docs/puppet/8/lang_data_abstract.html#variant-data-type)找到。

## 前缀

尽管这不是 Puppet 的术语，我们将回顾的类型将被描述为前缀，其中一个类型在另一个类型前加上前缀，且没有其他选项。

### 敏感

`Sensitive` 数据类型用于标记字符串为敏感值，这意味着该值将在代码和目录中以明文显示，但不会出现在任何 Puppet 报告或日志中。通过在参数和赋值前加上 `Sensitive` 关键字，这些字符串的内容会被标记为敏感。此操作影响字符串类型以及可以包含字符串或可以转换为字符串的资源。在以下示例中，我们展示了一个字符串、一个字符串数组和一个可以分配的数组。输出将打印 `[value redacted]`，表示已标记为敏感的部分：

```
$secret_string = Sensitive('password')
notify {"Print ${secret_string}":}
$single_sensitive_array = [Sensitive('password'),'password']
notify {"Print ${single_sensitive_array}":}
$secret_array = Sensitive(['password','password'])
notify {"Print ${secret_array}":}
```

当值需要在代码中使用时，`unwrap` 函数允许我们查看敏感值。此示例展示了如何解开该值并使用 `notify` 资源打印：

```
notify {"Print ${secret_string.unwrap}":}
```

这仅仅是一个示例，且会违背隐藏日志和报告中值的目的；更可能的是，它会传递给另一个资源。像 `password` 这样的属性，其用户识别的敏感值不需要解包，但像 `exec` 这样的资源则不能插值，因此值必须解包。为了避免泄露数据，像 `exec` 这样的资源不能插值，你可以将其包装为 `Sensitive`，以确保在日志中不会暴露任何部分。以下示例展示了将敏感字符串传递给 `user`，并将敏感字符串作为密码传递给 `curl` 命令：

```
user { 'max'
  id => 7
  password => $secret_string
}
exec {'secure curl':
  command => Sensitive("C:\\Windows\\System32\\curl.exe -u david:${secret_string.unwrap} http://example.com")
}
```

如果在使用 `debug` 运行 Puppet 时仅执行解包操作，命令和密码将完全可见。

在 *第七章* 中，我们将讨论模板，包括如何使用敏感值。然而，从 Puppet 7.0 和 6.20 开始，你不再需要在模板中使用敏感值之前将其解包。

注意

完整的端到端数据保护将在 *第九章* 中讨论。

`Enum` 和更高级的模式数据类型模式将在下一节中介绍，这些模式与 `Sensitive` 不兼容，应避免使用。在此，你应仅使用基本类型，如 `string`。

### 可选

`Optional` 数据类型允许 `undef` 作为数据类型的可接受输入，除此之外，还可以使用它所前缀的其他类型：

```
Optional <type> <variable name>
```

例如，要允许将 `Integer` 参数或 `undef` 分配给 `oracle_uid` 变量，只需在 `Integer` 类型前添加 `Optional` 关键字：

```
class oracle (
  Optional Integer orace_uid
)
```

`notundef` 类型有相反的效果，但用途更为有限且是例外情况。

## 模式

模式类型允许对属性应用类型组合，例如正则表达式或特定的字符串选择。

### 枚举

`Enum` 数据类型允许你列举字符串，使得多个选项可以在 `class` 参数中使用。以下代码声明了 `Enum`，后面跟着一个字符串数组，作为选项，最少包含一个或更多的字符串：

```
Enum[<string>,*<string>]
```

以下示例展示了如何在名为 `regional` 的类中使用这个，类的参数 `uk_region` 接受一个可用的英国区域：

```
class regional (
  Enum['Scotland,'England','Wales','Northern Ireland'] uk_region
)
```

### 变种

`Variant` 数据类型允许你将其他任何数据类型组合成一个数组。以下代码使用 `Variant` 关键字，并声明了参数允许的类型列表：

```
Variant[<type>,*<type>]
```

例如，下面的类接受 `true` 和 `false` 的布尔值，或者 `true` 和 `false` 字符串作为 `create_user_home` 变量的值。它还将接受一个字符串或一个字符串数组作为 `user_names` 变量的值：

```
class user_accounts(
  Variant[Boolean, Enum['true', 'false']] create_user_home
  Variant[String,Array[String]] user_names
)
```

### 模式

`Pattern` 数据类型类似于 `Variant`，但是它提供了一种方式来提供一组正则表达式，参数可以匹配其中任何一个。其语法如下：

```
Pattern[<regexcp>*<regexcp>]
```

在这里，我们使用 `Pattern` 关键字进行声明，后面跟着一个 `regexp` 类型的数组。例如，以下定义的类型 `server_access`，要求主机名是以 `edi`、`gla` 或 `abe` 开头的字符串：

```
Define server_access (
  Pattern[/^edi/,/^gla/,/^abe/] hostname
)
```

## 数组和哈希

在这一部分，我们将涵盖各种数组和哈希类型。

### 元组

在上一节中，我们讨论了数组类型可以声明一个类型来包含其所有内容。`Tuple` 允许在数组的特定索引位置使用任意数量的类型，并支持可选的最小值和最大值。最小值如果小于已分配的类型数量，则这些类型变为可选，而最大值则允许最后一个类型在最大值大于已声明类型数量时重复。最大值需要声明一个最小值：

```
Tuple[ <type>, *<type>,  <minimum size>, <maximum size>]
```

为了提供一个例子，假设有三个变量：`user_declaration`、`calculation` 和 `file_download`。`user_declaration` 变量需要一个字符串表示用户名，一个整数表示 UID，以及至少一个长度不超过八个字符的字符串，表示用户可以被分配到的组。`calculation` 变量需要一个整数，一个浮动数和一个整数。`file_download` 变量需要一个 URI 和一个字符串，另外，整数是可选的，并不是必须的：

```
class exampleapp (
Tuple [ string, integer, string, 3 , 10 ] user_declaration
Tuple [ integer, float, integer] calculation
Tuple [ uri, string , integer, 2] file_dowload
)
```

### 结构体

`Struct` 提供了一种类似于 `Tuple` 的类型来处理哈希。在 `Hash` 数据类型中，单个键类型和值类型被声明，而结构体允许按特定顺序声明字符串键，并且键的值可以选择性地为 `optional` 或 `undef`，同时允许声明值类型。与 `Tuple` 不同，没有最小或最大大小的限制：

```
Hash[<*optional *undef String name>, <Value type>, *(<*optional *undef String name >,<value type>)
```

为了说明可选键和值如何影响变量赋值，让我们考虑三个例子：`config_file`、`application_binary` 和 `application_startup`。`config_file` 变量需要键值对，包括具有字符串值的 `mode` 键，其值可以是 `file` 或 `link`，以及具有字符串值的 `path` 键。`application_binary` 变量与 `config_file` 相似，但它允许可选的 `owner` 键，值为字符串。如果存在，`owner` 键必须有字符串值。`application_startup` 变量要求一个 `owner` 键，可以是未定义值或字符串。此外，每个键的值必须匹配预期的数据类型：

```
class skeleton (
Struct[{mode => Enum[file, link],
        path => String config_file
Struct[{mode            => Enum[file, link],
        path            => String,
        Optional[owner] => String}] application_binary
Struct[{mode            => Enum[file, link],
        path            => String,
        owner           => Optional[String]}] application_startup
)
```

## 父数据类型

以下数据类型允许将多种数据类型组合为单一参数。直接使用它们可以使代码更简洁、更清晰：

+   `任意`：`任意`类型匹配任何 Puppet 数据类型，当确切的数据类型未知或不重要时非常有用。

+   `集合`：`集合`类型匹配任何数组或哈希数据类型，当数组或哈希可以包含多种数据类型时非常有用。

+   `标量`：`标量`数据类型匹配字符串、布尔值、正则表达式和数字。当需要包含这些数据类型的单个值时非常有用。

+   `数据`：`数据`类型匹配标量、未定义值、包含匹配数据的数组，以及具有匹配标量的键和匹配数据的值的哈希。当需要复杂数据结构时非常有用。

+   `数字`：`数字`类型匹配浮动和整数数据类型，当需要数值时非常有用。

## 实验室

继续我们的 `all_grafana` 类的工作，创建一个 `all_grafana_data_types` 类，并将其添加，使其接受一个 `file_options` 参数。此参数必须有一个名称，但可以选择性地具有模式、用户和作为哈希的组。确保这些资源的每个数据类型都是受限制的。添加一个 Grafana 用户和一个传递给该用户的敏感参数密码。

要实现类赋值，请在赋值类之前编写类声明。当你在清单上运行 `bolt` 时，它将包含你的变量。解决方案可以在 [`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch04/all_grafana_data_types.pp`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch04/all_grafana_data_types.pp) 上找到。

# 范围

在 Puppet 中，作用域是具有对变量和资源默认设置有限访问权限的代码级别。作用域有三个级别：顶级作用域、节点作用域和本地作用域。顶级作用域变量反映的是全局声明的变量，最常见的是在 `site.pp` 清单文件中声明。节点作用域变量是在节点定义中分配的，通常也在 `site.pp` 中声明，或通过 Puppet 环境中的 `site.pp` 清单文件使其对所有节点全局可用。或者，可以在 `site.pp` 中的节点定义或 ENC 中声明变量，以便在节点级别为特定服务器或服务器组提供变量。`site.pp` 是 Puppet 中的一个特殊清单文件，包含 Puppet 环境的主要配置。资源默认值是资源的默认设置，可以在更具体的作用域中覆盖，例如节点作用域或本地作用域。`site.pp`、ENC 和节点定义的完整使用将在 *第十章* 中详细解释。

在访问变量时，默认情况下，服务器将首先访问最低级别的变量，并且本质上会覆盖更高级别中同名的变量。其他本地作用域可以通过使用命名空间进行访问，但不能赋值。

这是一个示例，展示这些概念如何在单个 Puppet 清单文件中一起工作。我们可以定义一个名为 `'global'` 的全局变量，其值为字符串 `'world'`，并定义一个节点定义，该定义默认为所有节点分配一个名为 `'node'` 的变量，值为字符串 `'mynode'`。该节点定义包括两个类，`'local'` 和 `'also_local'`。在 `'local'` 类中，我们将一个名为 `'global'` 的变量赋值为字符串 `'override'`，它具有本地作用域并覆盖全局值。我们将使用两个通知资源来演示变量作用域是如何工作的。第一个通知资源打印 `'Print override'`，显示 `'global'` 本地变量已覆盖全局值。第二个通知资源使用 `::` 语法引用全局变量，因此它打印 `'Print world'`。第三个通知资源打印 `'Print node'`，因为没有具有该名称的本地变量。在 `'also_local'` 类中，我们定义了一个新变量 `'another_global'`，其值为字符串 `'another world'`。该类中的第一个 `notify` 资源使用直接访问的变量打印 `'Print another override'`。第二个 `notify` 资源使用 `::` 语法引用全局变量并打印 `'Print another world'`，因为没有声明名为 `'global'` 的本地变量。`notify` 资源是 Puppet 的一种资源类型，简单地将消息记录到控制台或系统日志中，通常用于调试或信息目的。

```
$global = 'world'
node default {
  $node = 'mynode'
  include local
  include also_local
}
class local
{
$global = 'override'
  notify {"Print ${global}":}
  notify {"Print ${::global}":}
  notify {"Print ${node}":}
}
class also_local {
  notify {"Print another ${local::global}":}
  notify {"Print another ${global}":}
}
```

资源标题或对资源的引用不受作用域限制，因为它们必须在整个目录中唯一。如前面示例所示，在 `also_local` 类中使用的 `notify` 资源的标题被调整为包含 `another`。这有助于我们避免在变量插值时出现资源标题冲突。否则，`local` 和 `also_local` 类都会包含名为 `Print override` 和 `Print world` 的 `notify` 资源，且会因重复资源而无法编译。

如前所述，`also_local` 类可以从 `local` 类中调用 `global` 变量，但不能将其赋值给该本地作用域。

# 总结

在本章中，我们学习了 Puppet 变量与普通过程语言中的变量不同，因为它们只能被赋值一次。我们看到某些词是保留的，不能用作变量命名。我们还看到，Puppet 变量可以进行插值，这取决于字符串的放置方式和位置。

我们介绍了各种核心数据类型及其如何用于限制参数和赋值变量。我们还探讨了 `undef` 和布尔值，它们在转换值时需要小心管理，以获得预期的结果。

接下来，我们研究了数组和哈希以及如何为它们赋值。尽管它们不能被更改，但我们了解了运算符如何将它们转化为新的赋值。我们还讲解了数组和哈希如何嵌套以及它们如何混合为数组的哈希或哈希的数组。

接着，我们研究了抽象数据类型及其如何通过 `Sensitive` 类型更加灵活地限制参数，它为日志和报告提供了作用域保护。

之后，我们回顾了如何在不同的作用域中声明 Puppet 变量，以及如何在不同的作用域中共享/查看变量。

在下一章中，我们将介绍事实和函数。我们将查看系统配置工具 Facter，它收集的信息，以及如何定制以收集用户特定的系统配置数据。函数提供 Ruby 代码插件，允许在编译时运行代码，可以执行数据操作或影响目录的运行。我们将讨论内置函数以及来自标准 `lib` 模块的函数，这些函数可以用于将数据类型转化为我们在本章中讨论的变量。
