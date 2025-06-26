

# 模板化、迭代和条件语句

本章将介绍 Puppet 语言中的高级结构，包括允许在模板文件中插入变量的模板。Puppet 中有两种格式可用：**嵌入式 Ruby**（**ERB**）模板，它基于原生 Ruby 模板化，和**嵌入式 Puppet**（**EPP**）模板，它是基于现代 Puppet 语言的模板。本章将讨论这两种格式，重点介绍它们之间的区别，以及使用 EPP 而非 ERB 的核心优势。

此外，本章还将深入探讨 Puppet 中的迭代和循环，展示如何使用 Puppet 中的迭代函数和代码块（lambdas），而不是其他语言中常见的`loop`关键词。最后，本章将讨论 Puppet 中可用的不同类型的条件语句，包括`if`、`case`和`unless`语句，这些语句是任何编程语言中常见的，以及 Puppet 特有的选择器，它允许根据事实或变量选择键或变量上的值。本章还将详细探讨在条件语句中使用正则表达式的情况。

本章将涵盖以下主要内容：

+   Puppet 中的模板格式 – EPP 和 ERB

+   迭代和循环

+   条件语句

# 技术要求

本节中的所有代码都可以在本地开发服务器上进行测试。

# Puppet 中的模板格式 – EPP 和 ERB

Puppet 中的模板化允许通过替换变量并使用条件逻辑来定制内容，从而生成标准格式的内容。Puppet 支持两种模板格式：ERB，它是一个原生 Ruby 模板格式（[`github.com/ruby/erb`](https://github.com/ruby/erb)），并且在所有版本的 Puppet 中都可用；EPP 模板，它基于 Puppet 语言，在 Puppet 4 中引入，并且在启用未来解析器的 Puppet 3 版本中也可以使用。

模板提供的灵活性超过了字符串，但比使用`file_line`、`augeas`或`concat`等资源控制单个或一组设置的灵活性要低。因此，在决定使用模板还是资源时，需要在复杂性之间找到平衡。

对于相对较短的`heredoc`文件或简单字符串，使用带变量插值的模板可能已足够。然而，对于更复杂的文件，尤其是多个模块可能管理不同设置或接受手动编辑的文件，使用资源来处理每个设置或部分会更加简单且易于管理。

在旧代码中，可能会发现模板使用过度，这反映了在 Puppet 早期版本中缺乏资源类型（如 `file_line`）。因此，审查尝试实现的目标状态非常重要，并确保通过使用模板控制所有内容设置时，整个文件不会被不必要地强制执行，避免其中包含已变得冗余的设置，因为与配置文件相关的底层应用程序已经更新并更改了其配置设置。

虽然新代码中没有必要使用 ERB，但许多 forge 模块和遗留代码库可能包含 ERB，因此本节将涵盖这两种格式，以帮助理解。在展示两种格式的语法后，将讨论使用 EPP 的优点以及将 ERB 转换为 EPP 的理由。

模板可以通过使用模板文件中的内容或通过字符串（称为内联模板）生成。对于模板文件，ERB 使用 `template` 函数，EPP 使用 `epp` 函数。对于内联模板，EPP 使用 `inline_epp` 函数，ERB 使用 `inline_template` 函数。

## EPP 模板

EPP 模板文件是一个文本文件，包含文本和 Puppet 语言表达式的混合，表达式被标签包围。这些标签指示 Puppet 表达式应如何评估，并可以修改模板中的文本，从而基于 Puppet 语言特性（如变量插值、逻辑语句和函数）创建文件。

*表 7.1* 显示了可以使用的标签类型：

| **标签名称** | **起始标签（****带修剪）** | **结束标签（****带修剪）** | **目的** |
| --- | --- | --- | --- |
| 参数 | `<% &#124;` (`<%- &#124;`) | `&#124;%>` (`&#124; - %>`) | 声明模板接受的参数 |
| 非打印表达式 | `<%` (`<%-`) | `%>` (`<%-`) | 评估 Puppet 代码但不打印 |
| 表达式打印 | `<%=` | `%>` (`-%>`) | 评估代码并打印其值 |
| 注释 | `<%#` (`<%#-`) | `%>` (`-%>`) | 允许仅为模板文件本身添加注释行 |

表 7.1 – EPP 模板标签

当模板被评估时，它在遇到起始标签时会切换到 Puppet 模式，遇到结束标签时返回到文本模式。在文本模式下，它将文本输出为内容，当找到标签时，起始标签和结束标签之间的 Puppet 代码会被评估，具体取决于起始标签的类型。

如*表 7.1*所示，一些标签可以通过使用短横线（`-`）来修剪适当的空格和换行。

参数标签是可选的，如果使用，必须是模板文件中的第一个内容，注释标签除外，且注释标签必须使用闭合连字符。其行为类似于 Puppet 类的参数声明方式，如在 *第八章* 中所示。参数遵循与类相同的模式，因此它们可以选择性地在开头包含类型。接着必须有一个美元符号（`$`），后跟变量名，后可选择一个等号（`=`）和默认值，最后必须以逗号结束。

例如，若要使一个选项参数包含一个默认值为空字符串的字符串，一个 `application_mode` 参数（可以包含 full、partial 或 none 字符串，并且默认值为 `node`），以及一个 `cluster_enabled` 参数（布尔值），以下代码将开始我们的模板：

```
<%- |
String $options = '',
Enum[full,partial,none] $application_mode = 'none',
Boolean $cluster_enabled,
|-%>
```

当参数传递给 EPP 模板时，它们变为局部作用域，并且可以直接按名称调用，但来自调用类的变量必须使用完整的命名空间名称；这类似于定义类型。任何没有默认值的参数，例如前述示例中的 `cluster_enabled`，都是必填的，必须传入。

注意

推荐始终在参数前使用连字符，以避免模板开头的任何意外空格。

如果没有使用参数，可以直接使用类作用域访问类变量，例如 `$example_module::example_param`。

参数使得模板在多个不同位置使用时更加灵活，确保数据更为明确且与需求紧密绑定，并且一目了然地展示了消耗的是什么数据。当需要使用大量变量时，使用变量可能比使用参数更合适，因为参数无法扩展。当未在参数列表中定义的参数被传递时，将导致语法错误，尽管如果不使用参数，可以向模板传递任何参数。本节后面将展示如何在引用 `epp_template` 函数时传递哈希值。

Puppet 模板中的 `comment` 标签允许在模板文件内添加注释。这些注释在模板被评估并生成内容时不会出现在输出中。以下是 Puppet 模板中注释的示例：

```
<%#- An example comment. -%>
```

注意

`<%#-` 连字符修剪功能自 Puppet 6.0.0 版本起提供。在此之前，修剪行为是默认假定的。

打印表达式标签将 Puppet 表达式的返回值输出到结果中。这可以是一个变量或事实，函数的输出，或者运算符的输出。最终的输出是一个字符串，并会在必要时自动转换。在最简单的情况下，这可以用来打印事实作为值。例如，以下行将读取 `application = exampleapp`，如果 `application_name` 事实包含 `exampleapp` 值：

```
application = <%= $facts[application_name] -%>
```

本示例也是变量在这个上下文中首次展示，但它们的访问方式与常规 Puppet 代码中的变量相同。

非打印标签包含迭代和条件逻辑。这与其他标签不同，因为标签的效果可以跨越多行，直到另一个标签关闭迭代器或条件逻辑。例如，`if` 语句（将在*条件语句*部分中讨论）以花括号 `{` 开始，以花括号 `}` 结束。在下面的示例中，我们可以确保如果应用程序名称从 `getvar` 函数返回 `undef`，则不会输出 `application =`，就像我们之前的例子中所做的那样。相反，如果变量未定义，则会忽略该行：

```
<% if getvar(facts.application_name) { -%>
 application = <%= $facts[application_name] -%>
<% } -%>
```

可以使用多级非打印标签来创建适当的嵌套 `if` 或 `case` 语句。

有一些语法错误需要小心处理。如果非打印表达式标签包含注释，则基本上会注释到行尾，并且需要在下一行上关闭标签，如本例所示：

```
<%-# I don't finish commenting here  -%> but on the next line
-%>
```

这种错误显然可能会发生在误键入的 `comment` 标签中，因此必须小心，并且任何标签，而不仅仅是注释关闭标签，都会被忽略直到新行。

要在模板输出中包含文字 `<%` 或 `%>` 字符而不让它们作为 EPP 标签进行评估，您可以使用额外的 `%` 字符来进行转义。例如，要将 `<% Puppet expression example %>` 输出为文本，您应该写成 `<%% Puppet expression example %%%>`。请注意，转义仅适用于遇到的第一个 `<%` 或 `%>`，因此如果您需要在一行中仅转义其中一个，您可以使用一次转义，然后正常使用另一个符号。

可以使用 `puppet epp validate <template_name.epp>` 命令验证 EPP 模板，并且在*第八章*中，将看到 **Puppet 开发工具包** (**PDK**) 将在其验证过程中运行此命令。

要测试模板的渲染，可以使用 `render` 命令，并根据需要使用值哈希：`puppet epp render <template_name.epp> --values '{key1 => value1, key2 =>` `value2}'。

注意

EPP 模板的完整规范可以在线查看 [`github.com/puppetlabs/puppet-specifications/blob/master/language/templates.md`](https://github.com/puppetlabs/puppet-specifications/blob/master/language/templates.md)。

在审查 EPP 模板文件的语法之后，让我们看看如何在 Puppet 资源中使用 `epp` 函数。可以通过将其传递给 `content` 属性，将 `epp` 函数用于诸如 `file` 等资源。此外，可以提供键值哈希以指定参数，正如在关于 `parameter` 标签的前一节中讨论的那样：

```
file { '/etc/exampleapp.conf':
  ensure => file,
  content => epp('exampleapp/exampleapp.conf.epp', {'version' => '1', 'clustered' => false}),
}
```

注意

如果你希望在开发环境中尝试前面的示例，请在系统上创建一个合适的模板文件，并将`exampleapp`模块名更改为包含模板的绝对路径，例如`/var/tmp`或`C:\Users\David Sandilands`。

`epp`中使用的命名空间假设它要么形成一个模块路径`<modulename/templatename.epp>`，该路径转换为`modulepath/modulename/templates/templatename.epp`，要么是磁盘上的绝对路径。在*第八章*中，将详细介绍模块的结构。

内联模板类似于常规模板，但它们不使用单独的模板文件，而是需要传递一个字符串或变量给它们。它们通常用于解决方法或在使用 heredoc 比使用模板文件更简单的情况下。

一个解决方法的例子是在使用*第五章*中讨论的 Vault 模块时，通过延迟函数来检索密钥。Vault 模块返回一个键值对，但我们可能只想访问密码的值。由于该值是延迟的，因此不能像字符串一样操作。使用`inline_epp`函数，如以下示例所示，可以在代理运行时解包字符串并将其应用到文件中：

```
$vault_keypair = { 'password' => Deferred('vault_lookup::lookup', ["secret/examleapp", 'https://vault:8200']), }
file { '/etc/exampleapp_secret.conf':
  ensure => file,
  content => Deferred('inline_epp', ['PASSWORD=<%= $password.unwrap %>', $vault_keypair]),
}
```

在介绍了用于管理遗留代码的 EPP 模板后，我们将回顾 ERB 的不同之处。

## ERB 模板

ERB 模板与 EPP 模板类似，但有一些值得注意的区别。ERB 模板是包含文本和 Ruby 语言表达式的混合文本文件，表达式被标签包围。ERB 使用与 EPP 相同的标签，但它没有参数标签，并且无法传递参数。

在 ERB 中，模板具有局部作用域和父作用域，后者是评估模板的类或定义的类型。当前作用域中的变量可以使用`@`符号访问，这是 Ruby 通常访问变量的方式。要访问作用域外的变量，可以使用`scope`对象或较旧的`scope.lookup`函数，后者是在引入哈希格式之前在 Puppet 中使用的。

为了给出一些简单的 Ruby 示例，可以使用`if`语句检查`exampleapp_extras`变量是否不包含`NONE`，并在模板中输出`extras <exampleapp_version>`字符串。还可以使用`unless`语句检查`exampleapp_key`变量是否为`nil`，如果它有定义值，则输出`key <exampleapp_nill>`字符串：

```
<% if @exampleapp_extras!= "NONE" %>extras<%= @exampleapp_version%><% end %>
<% unless @exampleapp_key.nil? -%>
key <%= @exampleapp_%>
<% end -%>
```

Ruby 中的迭代类似，`each`函数也可用。以下示例显示了通过迭代将一个变量中的设置数组逐个输出到模板内容中：

```
<% @array_of_settings.each do |setting| -%>
<%= val %>
<% end -%>
```

Puppet 变量的数据将从 Puppet 类型转换为等效的 Ruby 类型（更多信息请参见官方 Puppet 文档：[`www.puppet.com/docs/puppet/latest/lang_template_erb.html#erb_variables-puppet-data-types-ruby`](https://www.puppet.com/docs/puppet/latest/lang_template_erb.html#erb_variables-puppet-data-types-ruby)）。然而，如何将这些数据转换为 Ruby 类型已经超出了本书的讨论范围。

还可以使用 `<%scope.function_name(<函数名称>, <参数数组>)%>` 语法在 ERB 模板中调用 Puppet 函数。

例如，要对 `example_variable` 变量使用 `downcase` 函数并将结果输出到模板，可以使用以下代码：

```
<%= scope.call_function('downcase', [@example_variable]) %>
```

验证 ERB 模板的语法可以通过运行 `erb` 命令来完成：`erb -P -x -T '-' example.erb | ruby -c`。与 EPP 一样，PDK 在运行验证时会检查两种模板。遗憾的是，无法呈现 ERB 模板。

在文件中使用 ERB 模板文件的内容看起来与 EPP 非常相似，但如前所述，它没有参数，并且使用 `template()` 函数。转换 EPP 示例的方式如下：

```
file { '/etc/exampleapp.conf':
  ensure => file,
  content => template('exampleapp/exampleapp.conf.erb'),
}
```

可以传递并评估多个模板文件，这些文件将会合并在一起。例如，更新如下内容将合并两个模板：

```
content => [ template('exampleapp/exampleapp.conf.erb'), template('exampleapp/exampleapp2.conf.erb')]
```

内联 ERB 仅使用与内联 EPP 相同系统中的`inline_template`函数，并且过去经常用来通过 Ruby 代码提供一种解决方案，弥补早期版本的 Puppet 缺乏迭代/循环功能，并执行数据转换。

现在既然已经讨论了 ERB，是时候突出 EPP 相较于 ERB 更受青睐的原因以及考虑转换遗留代码的理由了。

## EPP 与 ERB 比较

在审查了两种语法模板后，很明显 EPP 相较于 ERB 具有多个优势。首先，EPP 的性能明显优于 ERB。每次评估模板时，ERB 会为所有事实和顶层变量创建一个作用域对象，而 EPP 仅使用与模板相关的事实和变量。在拥有大量事实的环境中，这可能会对性能产生显著影响。

此外，EPP 提供了更高的安全性，因为模板可以提供一个有限的数据范围供使用，并在使用前验证所有数据是否存在。而 ERB 则没有内建的验证，且不存在的变量会被简单地忽略。例如，如果类中的某个变量在模板使用前未被评估，ERB 将无法捕捉到这一点。

EPP 也被认为更易于使用，因为它采用 Puppet DSL 风格，并且不需要任何 Ruby 知识。这使得编码更加容易，特别是能够使用`puppet epp render`和`validate`命令。此外，EPP 正在积极开发中，最近的功能（例如，6.20 及以后版本中能够自动解开敏感变量的模板）仅在 EPP 中可用。

# 迭代和循环

Puppet 的迭代和循环方法受其变量不可变性的影响，这意味着一旦设置，变量就不能更改。这使得许多正常使用`loop`或`do`关键字来转换数据的方法变得不可能。在语言的早期版本中，通过传递数组给定义类型来解决这个问题，如*第三章*所述，或者使用带有 Ruby 代码的内联 ERB 模板来操作数组和哈希。

然而，使用定义类型方法的问题在于，执行工作的代码被抽象化了，不可见。而且，每次需要不同类型的迭代时，都需要其自定义的定义类型，这使得代码膨胀。因此，回顾旧代码并重构这些模式为接下来将讨论的方法非常重要。

在现代 Puppet 中，采取的方法是使用迭代函数将数据从数组和哈希传递给 lambda。lambda 是一个没有名字的函数，因此除非通过某个函数调用，否则无法在其他地方调用。lambda 可以附加到任何函数调用中，包括自定义函数。*表 7.2*提供了涉及迭代和 lambda 的完整函数列表。虽然一些函数可能不被认为是迭代器，但它们有类似的行为。还应注意，其他函数可以与这些示例组合/链接，例如`unique`：

| **函数名称** | **目的** | **返回类型** | **参数** |
| --- | --- | --- | --- |
| `all` | 遍历所有元素，直到 lambda 返回`false`或完成并返回`true`。 | `true`或`false` | 1 或 2 |
| `any` | 遍历所有元素，直到 lambda 返回`true`或完成并返回`false`。 | `true`或`false` | 1 或 2 |
| `break` | 用于 lambda 内部，停止迭代。 | 无 | 无 |
| `each`, `reverse_each`, `tree_each` | 依次传递哈希或数组的每个元素供 lambda 处理（逆序或递归变体）。 | 无 | 1 或 2 |
| `filter` | 遍历所有元素并与 lambda 代码匹配，返回匹配的元素数组。 | 数组 | 1 或 2 |
| `index` | 遍历所有元素，并在 lambda 代码中首次匹配时，返回匹配元素的索引。 | 整数 | 1 或 2 |
| `lest` | 该函数接受一个参数；如果该值未定义，它将运行一个 lambda 并返回结果。如果参数未定义，它将返回该参数。 | 任意有效类型 | 0 |
| `map` | 遍历所有元素并对该元素应用 lambda 代码。返回应用 lambda 后的元素数组。 | 数组 | 1 或 2 |
| `next` | 在 lambda 中使用，用于改变迭代中下一个元素的值。 | n/a | n/a |
| `reduce` | 遍历所有元素并应用 lambda 代码，将结果传递给每次迭代。 | 数组 | 2 |
| `return` | 用于使 lambda 返回（不能在顶层作用域中使用）。 | n/a | n/a |
| `slice` | 按切片大小遍历元素，例如每次迭代三个元素。 | 数组 | 1 或切片大小 |
| `step` | 链接到另一个可迭代函数，传递一个从起始元素到结束元素按步长递增的元素序列。 | 可迭代 | n/a |
| `then` | 接受一个参数，如果该参数不是 `undefined`，则调用一个 lambda 并传递该参数。否则，返回 `undefined`。 | 任何有效类型 | 1 |
| `with` | 接受一个参数，并无条件将其传递给 lambda 并使用该参数运行。返回 lambda 的结果。 | 任何有效类型 | 1 |

表 7.2 – 迭代和 lambda 函数

在 Puppet 中使用 lambda 的迭代函数的基本语法结构如下：

```
<function acting on data> | <parameter(s)> | { lambda of Puppet code }
```

例如，考虑使用 `each` 函数对一个数组进行迭代，使用一个参数（可选类型），并打印输出：

```
['first', 'second', 'third'].each | String $x | { notice $x }
```

这将导致 `notice` 函数为数组中的每个字符串打印输出，类似于大多数语言中的 `for` 循环和 `print`/`echo` 命令。

`each` 函数也可以使用两个参数，第一个参数为索引，第二个参数为该索引对应的内容。以下代码将在第二次迭代时打印 `index 2 contains second`：

```
['first', 'second', 'third'].each | $index $value | { notice "index $index contains $value" }
```

注意

要在开发者桌面上测试这些示例，只需运行 `puppet apply -e '<``example code>'`。

为了明确，当在哈希上使用 `each` 函数并且仅使用一个参数时，每个键值对将作为一个数组传递给 lambda。例如，运行代码 `[{ key1 => 'val1', key2 => 'val2' }].each | $key_pair | { notice $key_pair }` 将输出两个数组，每个键值对对应一个数组：`['key1', 'val1']` 和 `['``key2', 'val2']`。

如果 lambda 使用两个参数，第一个参数代表键，第二个参数代表值。例如，运行代码 `[{ key1 => 'val1', key2 => 'val2' }].each | $key, $value | { notice "$key contains $value" }` 将输出两个字符串，每个键值对对应一个字符串：`key1 contains val1` 和 `key2 contains val2`。

还值得注意的是，其他数据类型（例如字符串）可以自动转换为数组，其中字符串中的每个字符都被视为一个元素。此外，还可以使用 `Integer` 类型声明数字范围；例如，运行代码 `Integer[100, 150].each | Integer $number | { notice $number }` 将输出从 `100` 到 `150` 的所有整数。

最后，迭代可以嵌套；例如，为了处理具有数组值的哈希，可以在 lambda 内使用迭代函数。运行以下代码将输出 `key1` 键值对数组中的每个值——`'value1'` 和 `'value2'`：

```
[{ key1 => ['value1', 'value2'], key2 => 'val2' }].each | $key, $value_array | {
  $value_array.each | $value | {
    notice $value
  }
}
```

总体而言，本节概述了 Puppet 中最常用的函数，但若想了解更深入的描述，用户可以参考官方文档：[`www.puppet.com/docs/puppet/latest/function.html`](https://www.puppet.com/docs/puppet/latest/function.html)。

## 迭代循环

到目前为止审查的主要函数是 `each`，还有几个函数执行元素的循环或操作循环。`reverse_each` 函数简单地按名称所示取元素的反向顺序。`tree_each` 允许返回嵌套数组/哈希中的值，并根据提供的标志采用不同的行为。它相对复杂且较为冷门。`slice` 函数允许我们在每次迭代中获取特定数量的元素。例如，以下代码会每次传递三个数字的数组给 lambda：

```
Integer[100, 151].slice(3) | Array $numbers | { notice $numbers }
```

在最后一次迭代时，它将提供剩余的元素；在此示例中，即一个仅包含 `[151]` 的数组。也可以使用多个参数，但参数的数量必须与 `slice` 的大小相同：

```
Integer[100, 151].slice(3) | Integer $first, Integer $second, Integer $third  | { notice $numbers }
```

`step` 函数允许我们选择要传递的可迭代元素。在这个代码示例中，它将从第一个元素开始，然后是第四个、第七个，以此类推：

```
Integer[100, 150].step(3) | Integer $numbers | { notice $numbers }
```

这在链式调用到另一个可迭代函数时非常有用。下一个函数类型是用于匹配模式。这是一种不同风格的迭代函数，在这种方式下，迭代函数定义了 lambda 如何返回，而不是仅将元素传递给 lambda 执行某些操作。例如，`all` 寻找所有元素是否都满足 lambda 中的检查条件以返回 `true`。如果任何一个 lambda 返回 `false`，该函数将返回 `false`。例如，以下代码将输出 `true`，因为所有元素都大于 `99`：

```
Integer[100, 151].all | Integer $number | { $number > 99 }.notice()
```

`any` 函数与 `all` 相反，如果没有匹配项，则返回 `false`，如果在任何迭代中 lambda 返回 `true`，则返回 `true`。`index` 函数类似于 `any`，但它不是返回 `true` 或 `false`，而是返回匹配元素的索引号，若没有匹配项则返回 `undef`。例如，以下代码会输出 `20`，因为 `number` 会匹配到第 20 个元素：

```
Integer[100, 151].index | Integer $number | { $number == 120 }.notice()
```

所有这些函数都可以在数组或哈希上使用两个参数，如 `each` 函数的示例所示。

## 数据转换

数据转换是迭代器用于遍历元素并在返回之前进行调整的另一种方式。这也是迭代器与 lambda 模式开发的主要原因之一，因为 Puppet 无法重新赋值变量。例如，`map`函数会遍历每个元素，并应用一个 lambda，其结果会存储在一个数组中。例如，下面的代码会将每个元素除以`1024`，并返回一个数组`[2, 3, 1]`：

```
[2048, 3096, 1024].map | $size | {  $size / 1024 } .notice()
```

`filter`函数在每次迭代中处理每个元素，并应用 lambda 中的代码。如果 lambda 返回`true`，则该元素将被添加到数组中并返回。否则，如果返回`false`，则会继续到下一次迭代。例如，下面的 filter 会遍历每个数组，检查其大小是否大于`0`，结果会输出`[[1, 2, 3], ['a', 'b', 'c']]`，并移除第二个元素中的空数组：

```
[[1,2,3], [], [a,b,c]].filter | $array | { $array.size > 0   } .notice()
```

`filter`和`map`都可以处理一个或两个参数，如在数组和哈希上的`each`函数所示。

`reduce`函数允许在 lambda 中进行累积操作。它与其他函数不同，需要两个参数：第一个参数在迭代过程中保持其值，而第二个参数是元素。此外，可以通过传递一个值给`reduce`函数来选择第一个参数的初始值。在这个例子中，`total`参数的初始值为`1`，并且在每一轮中将元素加到它的总和上，最终返回并打印`15`：

```
[2, 4, 8].reduce(1) | $total, $number | {  $total + $number } .notice()
```

在 lambda 中，也可以改变迭代的流程。`next`函数可以改变下一个元素是什么：如果没有传递任何值给`next`函数，则为`undef`，否则为提供给`next`函数的值。`break`函数会在代码的这一点停止迭代器，并返回到迭代函数，从而有效地结束迭代器的执行。相比之下，`return`函数会从迭代函数中返回，因此不会继续执行，并返回到包含类、函数或定义的类型。

为了演示这一流程的变化，第一个使用`map`的例子会遍历一系列数字，运行`next`函数，当元素等于`101`时，会将下一个元素`102`替换为`1984`，然后当元素大于`104`时会运行`break`函数。因此，最终会打印的输出将是一个数组`[100, 1984, 102]`：

```
Integer[100, 151].map | Integer $number | { if $number == 101 { next(1984) } if $number > 103 { break() } $number }.notice()
```

为了突出用`return`函数替换`break`函数后的不同行为，下面的例子将不会打印任何内容：

```
class example {
  Integer[100, 151].map | Integer $number | { if $number == 101 { next(1984) } if $number > 103 { return() } $number }.notice()
} 
```

这就是在这个例子中我们将函数放入类中的原因，因为`return`只能在类、函数或定义类型中调用，不能在顶层作用域调用。

## 嵌套数据

最后一类函数在处理嵌套数据或处理未定义值时非常有用。`then` 会在 lambda 后链接，如果它从该 lambda 输出 `undef`，则返回 `undef`；否则，它会将该值传递给另一个 lambda。所以，以下示例将使用 `dig` 函数来尝试访问数组中第二个哈希表中不存在的 `c` 元素，由于 `then` 将接收到 `undef`，它因此会返回 `undef`：

```
$example = {first => { second => [{a => 10, b => 20}, {d => 30, e => 40}]}}
$example.dig(first, second, 1, c ).then |$x| { $x / 10 }.notice()
```

为了澄清前面的语句，如果将 `dig(first, second, 1, d)` 改为 `dig(first, second, 2, d)`，它将把 `30` 传递给 lambda，该 lambda 将除以 `10` 并打印出 `3`。

`lest` 是 `then` 的反义词，并返回其定义的值；否则，它会将 `undef` 传递给 lambda，这样就可以采取诸如设置默认值等操作。这在与 `then` 一起使用时非常有用。以先前的示例为例，添加 `lest` 将允许在值为 `0` 时返回 `undef`：

```
$example.dig(first, second, 1, c ).then |$x| { $x / 10 }.lest() || { 0 } .notice()
```

`with` 函数有些特殊，因为它用于通过 lambda 传递值，如果我们的 lambda 能处理 `undef` 或已定义的值。

因此，经过对各种函数的审查，以及数据转换和数据探索的可能性，值得再次强调，与过去使用定义类型的方式不同，当我们需要创建多个资源时，应该使用另一种方法。所以，例如，要为请求的多个实例创建目录，可以使用以下代码：

```
Integer[1,$instance_number].each |Integer $id | {
  file {"/opt/app/exampleapp/instance${id}":
    ensure => directory,
  }
}
```

在讲解了 Puppet 如何执行循环和迭代后，接下来将回顾条件语句。

# 条件语句

Puppet 具有你在任何语言中都能预期的条件语句，`if`、`unless` 和 `case` 允许根据事实或来自外部源的数据等不同因素使代码行为有所不同。此外，Puppet 还使用选择器，这类似于 `case` 语句，但返回一个值，而不是在结果上执行代码。

## `if` 和 `unless` 语句

`if` 语句遵循特定的语法，其中包括 `if` 关键字，后跟条件，一个左花括号（`{`），当条件为 `true` 时要执行的 Puppet 代码，最后是一个右花括号（`}`）。

以下示例是对 `example_bool` 中布尔值的简单检查，如果它包含 `true`，则打印一条通知：

```
if $example_bool {
  notice 'It was true'
}
```

这可以通过在`if`语句的闭括号（`}`）后可选地添加 `else` 关键字，并使用左花括号（`{`）结合 Puppet 代码来执行，当条件为 `false` 时。然后再用右花括号（`}`）结束。如果要在 `example_bool` 为 `false` 时也打印，则代码更新如下：

```
if $example_bool {
  notice 'It was true'
}else{
  notice 'It was false'
}
```

类似地，要一起执行多个`if`检查，可以在`if`语句的闭括号（`}`）之后使用`elsif`关键字，允许再次使用相同的`if`语法。可以根据需要进行嵌套和重复。举个例子，在布尔值检查后，使用`elsif`，我们可以添加第二个检查，查看值变量是否大于`2`，如果是，则打印一条通知，并使用`else`语句打印两个条件都是`false`的情况：

```
if $example_bool {
  notice 'It was true'
}elsif $value > 2{
  notice 'It was false and value is greater than 2'
}else {
  notice 'It was false and the value was 2 or less'
}
```

`unless`语句只是`if`语句的反义语句。它允许你避免对条件进行取反，并且它也可以与`if`语句结合使用。然而，它没有与`elsif`相对应的部分，如果使用将导致编译失败。以之前的`if`示例为例，可以改用`unless`语句来检查`example_bool`是否为`false`，如果是，则打印一条通知：

```
unless $example_bool {
  notice 'It was false'
}else{
  notice 'It was true'
}
```

条件中使用的任何非布尔值将根据数据类型规则转换为布尔值，如在*第四章*中所述。

Puppet 风格指南建议，在`if`和`unless`关键字之后的代码行应该缩进两个空格，并与示例中显示的对齐。

## `case`语句

`case`语句通过匹配控制表达式输出的值来工作。通常这是事实或变量的内容，但它也可以是一个表达式或函数。该格式以`case`关键字开始，后跟一个控制表达式，解析为一个值，并用大括号括起来。每一行以匹配的 case 或逗号分隔的 case 列表开始，后跟冒号，然后是要应用于匹配 case 的 Puppet 代码，代码用大括号括起来。`case`语句最后以大括号结束。

例如，要测试`hardwareisa`事实的值并根据使用的处理器架构类型包含一个配置文件，可以使用以下代码。它包括`sparc`或`powerpc`值的 Unix 配置文件，`i686`和`i386`值的 Linux 32 位配置文件，任何以`64`结尾的值的 64 位配置文件，以及任何无法匹配到某个情况的值的默认配置文件：

```
case $facts[' hardwareisa'] {
  'sparc', 'powerpc': { include profile::unix}
  'i686', 'i386': { include profile::linux::32bit}
  /(*64)/: { include profile::linux::64bit}
  default: { include profile::default }
}
```

Puppet 风格指南建议始终使用`default` case，它可以是一个失败的情况，甚至只是一个空的大括号。

## 选择器

选择器类似于`case`语句，但它们不会应用 Puppet 代码，而是返回一个值。选择器可以在期望值的任何地方使用，如变量赋值、资源属性和函数参数。Puppet 风格指南建议仅在变量赋值中使用选择器以提高可读性，但在遗留代码中，它也可以在资源属性中看到，因为它曾是一个流行的模式。

选择器的语法是一个控制表达式，解析为一个值，一个问号（`?`）和一个左花括号（`{`）。然后它有多个匹配案例，从一个单独的案例或`default`关键字开始，后跟一个哈希火箭符号（`=>`），返回的值，最后是一个闭合的逗号。选择器以一个右花括号（`}`）闭合。

以下示例展示了如何根据`os.family`事实的输出分配`apache_package_name`变量，使用`httpd`作为 Red Hat 的名称，`apache2`用于 Debian 或 Ubuntu，`apache-httpd`用于 Windows，如果没有匹配，则默认为`httpd`。然后，包资源可以使用这个名称来安装相关包：

```
$apache_package_name = $facts['os']['family'] ? {
  'RedHat' => 'httpd',
  /(Debian|Ubuntu)/ => ' apache2 ',
  'Windows' => 'apache-httpd',
  default => 'httpd',
}
package { $apache_package_name }
```

和`case`语句一样，Puppet 风格指南建议选择器始终使用`default`案例，这可以是一个失败，甚至只是一个空的花括号。

## 捕获变量

如果使用的`case`语句是正则表达式，则所称的捕获变量将在相关代码中作为数字变量（如`$1`和`$2`）可用，整个匹配项则可通过`$0`访问。

要修改*Case 语句*部分中的示例，如果匹配的是`amd64`，这将包括`profile::linux::amd64`：

```
/(*64)/: { include profile::linux::${1} }
```

在回顾了模板、条件语句、迭代和循环的各个方面后，我们将通过一个实验来总结并结合这些概念。

# 实验 – 创建并测试包含循环和条件的模板

在这个实验中，我们将把你在本章中看到的所有内容结合起来，测试和验证一些示例模板，并创建一个包含逻辑和迭代的模板：

1.  在[`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/tree/main/ch07/templates_to_check`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/tree/main/ch07/templates_to_check)下载模板文件，并验证和解析它们以确保模板生成以下内容（`<>`显示需要插值的部分）：

    +   **“此模板在具有<#你机器的 cpu 数量>个 cpu 的机器上运行”**

    +   **“自定义事实包.lab 已被<设置为事实的内容或字符串‘未设置’>”**

提示

你会想使用`getvar`函数来测试事实，并通过传入哈希来测试它，在解析时设置它以进行测试。

+   **“系统运行时间是<仅显示天、小时和分钟，如果它们非零>”**

+   **“此机器是<不是/是>虚拟的<并且运行在<$virtual>上”**

1.  创建一个模板，打印以下内容

**“这是一个<os family>机器，运行版本<os release full>”** **“以下目录在路径中<列出每个路径>”**

提示

使用 `split` 函数 ([`www.puppet.com/docs/puppet/latest/function.html#split`](https://www.puppet.com/docs/puppet/latest/function.html#split)) 将路径事实字符串分割成可以迭代的数组，并记住，Windows 路径是通过 `;` 分隔，而基于 Unix/Linux 的路径则通过 `:` 分隔。参见 [`github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch07/2_sample.epp`](https://github.com/PacktPublishing/Puppet-8-for-DevOps-Engineers/blob/main/ch07/2_sample.epp) 中的答案。

# 总结

本章审视了在 Puppet 中使用模板。展示了 Puppet 的两种模板类型——EPP 和 ERB——它们以类似的方式工作，通过将纯文本和代码周围的标签混合，允许 Puppet/Ruby 应用逻辑和变量，并在评估时创建更复杂的内容。文中警告，在使用模板代替诸如 `file_line` 等函数或单独控制资源之前，应仔细考虑复杂性的层级。此外，由于缺乏用于单行或文件设置控制的函数，模板已被过度使用，遗留代码应仔细审查，以确保模板是正确的复杂性级别。

显示了 EPP 是推荐的模板生成方式，因为它是用 Puppet 语言编写的，且对 Puppet 开发者来说更容易学习。它也更安全，因为可以通过参数限制其作用域，并且由于它仅为所需的变量和事实创建作用域，因此性能也更好；而 ERB 每次使用模板时，都会为所有事实生成一个作用域。此外，还提到所有 Puppet 的未来开发工作都将集中在 EPP 上，例如 EPP 文件中自动解包敏感变量的功能，且渲染模板的能力仅在 EPP 中可用。

`epp` 和 `template` 函数显示可以通过文件引用和评估模板，其中多个文件可以组合在一起。同时也展示了通过 `inline-template` 或 `inline-epp` 函数可以实现内联模板，这样文本可以直接传递给函数，而不需要存储在文件中。

然后展示了迭代和循环，强调了 Puppet 之前由于 Puppet 变量的不可变性，使得更传统的`loop`关键字变得不切实际。相反，Puppet 被展示为使用对数组和哈希的迭代函数，将值作为参数传递给 lambda 函数。这些没有命名的 lambda 函数只能通过其他函数调用，允许创建完全局部于 lambda 函数的作用域，因此允许变量名被重用。迭代函数选择如何将值传递给 lambda，例如每次传递一个值或键值对，或者使用`Reduce`，它允许在每个 lambda 函数中传递一个参数，连同每个值和键值对一起传递，并且可以用于执行累积转换。

然后讨论了 Puppet 的条件逻辑，显示其与大多数其他语言相似。`if` 检查会将检查评估为布尔值语句/比较，如果为`true`，则使用 Puppet 代码进行操作。`else` 关键字允许在布尔值为`false`时执行某个操作，`elsif` 关键字允许将多个检查链式连接在一起。`unless` 被显示为 `if` 的反义操作，当检查为负时进行操作，并允许 `else` 在检查为 `true` 时执行，尽管它没有 `elsif` 的等价项。

然后讨论了 `case` 关键字。我们展示了它是通过获取值并将其与某个匹配项进行匹配，基于匹配结果运行 Puppet 代码，或者如果没有找到匹配值，则执行默认操作。`selector` 关键字的行为类似于 `case` 语句，但它用于分配一个值，而不是运行 Puppet 代码。需要强调的是，尽管过去在资源中使用选择器是一个常见模式，但这已不再被认为是最佳实践。最后，捕获变量作为可用于条件语句的变量被展示，它们通过正则表达式显示匹配结果。

现在已经回顾了 Puppet 核心语言，是时候学习如何构建我们迄今为止使用的 manifest 文件和类了。下一章将展示模块如何提供必要的结构来容纳我们所研究的 manifest、类以及其他配置和实现文件。还将探讨角色和配置文件模式，它提供了一种额外的抽象，用于表示客户的技术堆栈和业务需求。此外，将演示 Puppet Forge 作为一个模块来源，可以通过它减少开发需求，并与 Puppet 社区协作以增强现有代码。
