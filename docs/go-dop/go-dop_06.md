# 第五章：使用常见的数据格式

DevOps 工程师需要的关键技能之一是能够跨各种存储介质操作数据。

在上一章中，我们与本地文件系统交互，读取和流式传输文件。这为我们在本章中将要学习的技能打下了基础。

本章将重点讲解如何操作工程师常用的常见数据格式。这些格式用于配置服务、结构化日志数据，以及导出度量数据，当然还有很多其他用途。

在本章中，你将学习如何使用`struct`字段标签来存储有关字段的元数据。此外，你还将学习如何在处理大量数据时高效地流式传输这些格式。

掌握这些技能将使你能够通过操作配置文件、查找可能包含日志或度量数据的记录，以及将数据导出到 Excel 中进行报告，从而与服务进行交互。

本章将涉及以下主题：

+   CSV 文件

+   流行的编码格式

在接下来的章节中，我们将深入探讨如何使用最古老的格式之一——CSV 来处理数据。

让我们开始吧！

# 技术要求

本章的代码文件可以从[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/5`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/5)下载

# CSV 文件

CSV 是 DevOps 工程师最常遇到的常见数据源之一。

这种简单的格式长期以来一直是企业界的主流，是将数据从系统中导出进行处理，并再导入数据存储的最简单方式之一。

许多大型云服务提供商的关键系统，如 Google 的 GCP 和 Microsoft 的 Azure，依赖 CSV 格式的关键数据源和系统。我们已经看到像网络建模和关键数据报告这样的系统存储在 CSV 中。

数据科学家喜欢 CSV，因为它易于搜索和流式处理。能够在软件中快速可视化数据的额外优点更是增加了它的吸引力。

和许多其他格式一样，它是人类可读的，这使得数据可以手动操作。

在本节中，我们将专注于使用以下方法导入和导出 CSV 数据：

+   `strings`包和`bytes`包

+   `encoding/csv`包

此外，我们还将探讨如何使用`excelize`包将数据导入和导出到流行的 Excel 电子表格格式。`excelize`是一个广泛使用的 Microsoft Excel 工具包。

现在，让我们讨论如何使用简单的字符串/字节操作包来读写 CSV 文件。

## 使用 strings 包进行基本的值分隔

Go 提供了几个在操作`string`和`[]byte`类型时非常有用的包：

+   `strings`

+   `bytes`

这些包提供了类似的功能，如以下内容：

+   分割数据的函数，如`strings.Split()`

+   合并带分隔符数据的函数，如`strings.Join()`

+   实现了`io`包接口的缓冲区类型，例如`bytes.Buffer`和`strings.Builder`

处理 CSV 文件时，开发人员可以选择流式读取数据或一次性读取整个文件。

许多开发人员更喜欢将整个文件读取到内存中，并将其从`[]byte`类型转换为`string`类型。字符串对开发人员来说更容易理解连接和拆分规则。

然而，这在转换过程中会创建一个副本，这可能会导致效率低下，因为你需要使用双倍的内存并且占用一些 CPU 进行复制。当出现这个问题时，开发人员通常会使用`bytes`和`bufio`包。虽然这些包稍微难以使用，但它们避免了不必要的转换开销。

让我们看看如何读取整个文件并将条目转换成结构化记录。

### 读取整个文件后的转换

在进行基本的 CSV 操作时，有时候更简单的做法是使用换行符拆分数据，然后根据逗号或其他分隔符来拆分每一行。假设我们有一个包含名字和姓氏的 CSV 文件，我们将这个 CSV 文件拆分为记录：

```
type record []string
func (r record) validate() error {
     if len(r) != 2 {
          return errors.New("data format is incorrect")
     }
     return nil
}
func (r record) first() string {
     return r[0]
}
func (r record) last() string {
     return r[1]
}
func readRecs() ([]record, error) {
     b, err := os.ReadFile("data.csv")
     if err != nil {
          return nil, err
     }
     content := string(b)
     lines := strings.Split(content, "\n") // Split by line
     var records []record
     for i, line := range lines {
          // Skip empty lines
          if strings.Trimspace(line) == "" {
               continue
          }
          var rec record = strings.Split(line, ",")  
          if err := rec.validate(); err != nil {
               return nil, fmt.Errorf("entry at line %d was invalid: %w", i, err)
          }
          records = append(records, rec)
     }
     return records, nil
}
```

上面的代码执行以下操作：

1.  它基于一个字符串切片`[]string`定义了一个`record`类型。

1.  我们可以通过调用`validate()`方法来检查一个`record`类型是否有效。

1.  可以使用`first()`方法获取记录的名字。

1.  可以使用`last()`方法获取记录的姓氏。

1.  它定义了一个`readRecs()`函数来读取名为`data.csv`的文件。

1.  它将整个文件读取到内存中，并转换为名为`content`的字符串。

1.  `content`通过换行符`\n`拆分，每个条目代表一行。

1.  它通过逗号（`,`）来拆分每一行。

1.  它将`Split`返回的每个结果（一个`[]string`类型）赋值给`record`类型。

1.  它将所有记录汇总到一个记录切片`[]record`中。

你可以在[`play.golang.org/p/CVgQZzScO8Z`](https://play.golang.org/p/CVgQZzScO8Z)查看此代码的运行情况。

### 按行转换

如果文件较大且我们希望提高效率，可以使用`bufio`和`bytes`包：

```
func readRecs() ([]record, error) { 
    file, err := os.Open("data.csv") 
    if err != nil { 
        return nil, err 
    } 
    defer file.Close() 
    scanner := bufio.NewScanner(fakeFile) 
    var records []record 
    lineNum := 0 
    for scanner.Scan() { 
        line := scanner.Text() 
        if strings.TrimSpace(line) == "" { 
            continue 
        } 
        var rec record = strings.Split(line, ",") 
        if err := rec.validate(); err != nil { 
            return nil, fmt.Errorf("entry at line %d was invalid: %w", lineNum, err) 
        } 
        records = append(records, rec) 
        lineNum++ 
    } 
return records, scanner.Err()
}
```

这与之前的代码不同，因为以下情况发生了：

+   我们逐行读取每一行，使用`bufio.Scanner`，而不是读取整个文件。

+   `scanner.Scan()`会读取下一组内容，直到遇到`\n`。

+   该内容可以通过`scanner.Text()`获取。

你可以在[`play.golang.org/p/2JPaNTchaKV`](https://play.golang.org/p/2JPaNTchaKV)查看此代码的运行情况。

在这个版本中，我们仍然对每一行进行`[]byte`转换为`string`类型。如果你对不做这种转换的版本感兴趣，请参考[`play.golang.org/p/RwsTHzM2dPC`](https://play.golang.org/p/RwsTHzM2dPC)。

### 写入记录

使用我们之前测试过的方法，写入 CSV 记录是相当简单的。如果在读取记录之后，我们希望对其进行排序并将其写回文件，可以使用以下代码实现：

```
func writeRecs(recs []record) error {
     file, err := os.OpenFile("data-sorted.csv", os.O_CREATE|os.O_TRUNC|os.O_WRONLY, 0644)
     if err != nil {
          return err
     }
     defer file.Close()
     // Sort by last name
     sort.Slice(
           recs, 
           func(i, j int) bool { 
               return recs[i].last() < recs[j].last()
           },
     )
     for _, rec := range recs {
          _, err := file.Write(rec.csv())
          if err != nil {
               return err
          }
     }
     return nil
}
```

我们还可以修改`record`类型，添加这个新方法：

```
// csv outputs the data in CSV format.
func (r record) csv() []byte {
     b := bytes.Buffer{}
     for _, field := range r {
          b.WriteString(field + ",")
     }
     b.WriteString("\n")
     return b.Bytes()
}
```

你可以在[`play.golang.org/p/qBCDAsOSgS6`](https://play.golang.org/p/qBCDAsOSgS6)看到这段代码的运行情况。

`writeRecs()`函数执行以下操作：

+   它打开`data-sorted.csv`进行写入。

+   它使用`sort`包中的`sort.Slice()`对记录进行排序。

+   它循环遍历记录并写出 CSV 文件，这是由新的`csv()`方法生成的。

`csv()`方法执行以下操作：

+   它创建了一个`bytes.Buffer`接口，类似于一个内存中的文件。

+   它循环遍历记录中的每个字段，并写入字段值，后跟逗号。

+   它在 CSV 行的内容后写入回车符。

+   它返回一个`[]bytes`类型的缓冲区，现在表示单行数据。

## 使用`encoding/csv`包

为了处理符合 RFC 4180 标准的 CSV 编码，[`www.rfc-editor.org/rfc/rfc4180.html`](https://www.rfc-editor.org/rfc/rfc4180.html)，标准库提供了`encoding/csv`包。

开发者应选择使用此包处理符合此规范的 CSV。

这个包提供了两种类型来处理 CSV：

+   `Reader`用于读取 CSV。

+   `Writer`用于写入 CSV。

在本节中，我们将解决与之前相同的问题，但我们将使用`Reader`和`Writer`类型。

### 一行一行读取

与之前一样，我们想要一次读取文件中的每个 CSV 条目，并将其处理为`record`类型：

```
func readRecs() ([]record, error) { 
    file, err := os.Open("data.csv") 
    if err != nil { 
        return nil, err 
    } 
    defer file.Close() 
    reader := csv.NewReader(file) 
    reader.FieldsPerRecord = 2 
    reader.TrimLeadingSpace = true 
    var recs []record 
    for { 
        data, err := reader.Read() 
        if err != nil { 
            if err == io.EOF{ 
                break 
            } 
            return nil, err 
        } 
        rec := record(data) 
        recs = append(recs, rec) 
    } 
    return recs, nil 
}
```

你可以在[`go.dev/play/p/Sf6A1AbbQAq`](https://go.dev/play/p/Sf6A1AbbQAq)查看这段代码的实际运行情况。

这个函数利用我们的 reader 执行以下操作：

+   将文件传递给我们的`NewReader()`构造函数。

+   设置 reader 要求每条记录有两个字段。

+   删除行首的空格。

+   读取每条记录并将其存储在`[]record`切片中。

`Reader`类型还有其他字段可以改变数据的读取方式。更多信息请参考[`pkg.go.dev/encoding/csv`](https://pkg.go.dev/encoding/csv)。

此外，`Reader`提供了一个`ReadAll()`方法，可以一次性读取所有记录。

### 一行一行写入

CSV 的`Reader`类型的伴侣，`Writer`，使得写入文件变得简单。让我们替换之前`writeRecs()`函数中的写入部分：

```
w := csv.NewWriter(file) 
defer w.Flush() 
for _, rec := range recs {
     if err := w.Write(rec); err != nil {
          return err
     }
}
return nil
```

这是可运行的代码：[`play.golang.org/p/7-dLDzI4b3M`](https://play.golang.org/p/7-dLDzI4b3M)

上面的代码执行以下操作：

+   它生成一个新的`Writer`类型，写入我们的文件。

+   它在函数退出时将内容刷新到文件。

+   它将每条记录写出为 CSV 文件，每行一条。

## 在处理 Excel 时使用 excelize

微软的 Excel 自 1980 年代以来一直是可视化数据的流行工具。尽管该程序的功能不断增强，但它的简易性帮助电子表格成为大多数企业中常见的工具。

虽然 Excel 不是 CSV 格式，但它可以导入和导出 CSV 数据。对于基本用法，你可以使用本章前面详细介绍的`encoding/csv`包。

然而，如果你的组织使用 Excel，使用其原生格式来写入数据并提供数据的可视化展示会更有帮助。`excelize` 是一个第三方 Go 包，可以帮助你完成这项工作。

包含该包的地址为 [`github.com/qax-os/excelize/tree/v2`](https://github.com/qax-os/excelize/tree/v2)。此外，官方文档可以在 [`xuri.me/excelize/`](https://xuri.me/excelize/) 查阅。

还有一个 Excel 的在线版本，作为微软 Office 365 的一部分。你可以直接在那儿操作电子表格；不过，我发现离线操作电子表格然后再导入会更方便。

如果你对 REST API 感兴趣，可以在 [`docs.microsoft.com/en-us/sharepoint/dev/general-development/excel-services-rest-api`](https://docs.microsoft.com/en-us/sharepoint/dev/general-development/excel-services-rest-api) 阅读相关内容。

### 创建一个 .xlsx 文件并添加一些数据

Excel 有一些特性，对于理解它非常有帮助：

+   一个 Excel 文件具有 `.xlsx` 扩展名。

+   每个 `.xlsx` 文件包含**工作表**。

+   每个工作表包括一组行和列。

+   `.xlsx` 文件有一个默认的工作表，称为 **Sheet1**。

+   一行和一列的交点称为**单元格**。

+   列从字母 A 开始。

+   行从数字 1 开始。

我们将添加一些代表虚构设备舰队的服务器数据。这些数据包括服务器名称、硬件代数、获取时间以及 CPU 厂商：

```
func main() {
    const sheet = "Sheet1"
    xlsx := excelize.NewFile()    
    xlsx.SetCellValue(sheet, "A1", "Server Name")
    xlsx.SetCellValue(sheet, "B1", "Generation")
    xlsx.SetCellValue(sheet, "C1", "Acquisition Date")
    xlsx.SetCellValue(sheet, "D1", "CPU Vendor")
    xlsx.SetCellValue(sheet, "A2", "svlaa01")
    xlsx.SetCellValue(sheet, "B2", 12)
    xlsx.SetCellValue(sheet, "C2", mustParse("10/27/2021"))
    xlsx.SetCellValue(sheet, "D2", "Intel")
    xlsx.SetCellValue(sheet, "A3", "svlac14")
    xlsx.SetCellValue(sheet, "B3", 13)
    xlsx.SetCellValue(sheet, "C3", mustParse("12/13/2021"))
    xlsx.SetCellValue(sheet, "D3", "AMD")
    if err := xlsx.SaveAs("./Book1.xlsx"); err != nil {
        panic(err)
    }
}
```

上面的代码执行了以下操作：

+   它创建了一个 Excel 电子表格。

+   它添加了列标签。

+   它添加了两个服务器，`slvaa01` 和 `slvac14`。

+   它保存了 Excel 文件。

有一个 `mustParse()` 函数（上面使用但未定义），它将表示日期的字符串转换为 `time.Time`。在 Go 中，当你看到函数名之前有 `must` 时，按惯例如果函数遇到错误，它会引发 panic。

你可以在 [`github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/5/excel/simple/excel.go`](https://github.com/PacktPublishing/Go-for-DevOps/blob/rev0/chapter/5/excel/simple/excel.go) 仓库中找到可运行的代码。

这个示例是向工作表添加数据的最简单方式。然而，它的可扩展性不强。让我们创建一个更具可扩展性的方法：

```
type serverSheet struct {
    mu sync.Mutex
    sheetName string
    xlsx *excelize.File
    nextRow int
}
func newServerSheet() (*serverSheet, error) {
    s := &serverSheet{
        sheetName: "Sheet1",
        xlsx: excelize.NewFile(),
        nextRow: 2,
    }
    s.xlsx.SetCellValue(s.sheetName, "A1", "Server Name")
    s.xlsx.SetCellValue(s.sheetName, "B1", "Generation")
    s.xlsx.SetCellValue(s.sheetName, "C1", "Acquisition")
    s.xlsx.SetCellValue(s.sheetName, "D1", "CPU Vendor")
    return s, nil
}
```

上面的代码执行了以下操作：

+   它为管理我们的 Excel 工作表创建了一个 `serverSheet` 类型。

+   它有一个构造函数来添加我们的列标签。

现在我们需要一些方法来添加数据：

```
func (s *serverSheet) add(name string, gen int, acquisition time.Time, vendor CPUVendor) error {
    s.mu.Lock()
    defer s.mu.Unlock()
    if name == "" {
            return errors.New("name cannot be blank")
    }
    if gen < 1 || gen > 13 {
            return errors.New("gen was not in range")
    }
    if acquisition.IsZero() {
            return errors.New("acquisition must be set")
    }
    if !validCPUVendors[vendor] {
            return errors.New("vendor is not valid )
    }
    s.xlsx.SetCellValue(s.sheetName, "A" +
strconv.Itoa(s.nextRow), name)
    s.xlsx.SetCellValue(s.sheetName, "B" + strconv.Itoa(s.nextRow), gen)
    s.xlsx.SetCellValue(s.sheetName, "C" + strconv.Itoa(s.nextRow), acquisition)
    s.xlsx.SetCellValue(s.sheetName, "D" + strconv.Itoa(s.nextRow), vendor)
    s.nextRow++
    return nil
}
```

这段代码执行了以下操作：

+   它使用锁来防止多次调用。

+   它执行非常基础的数据验证检查。

+   它添加了一行，并递增我们的内部 `nextRow` 计数器。

现在我们有了一个更具可扩展性的方法来向工作表添加数据。接下来，让我们讨论如何总结数据。

### 数据汇总

有两种方式可以总结添加的数据：

+   在我们的对象中跟踪汇总数据

+   Excel 数据透视表

对于我们的示例，我将使用第一种方法。这个方法有几个优点：

+   它更容易实现。

+   它执行更快的计算。

+   它从电子表格中删除了复杂的计算。

然而，它有一个显著的缺点：

+   数据变化不会影响汇总。

为了跟踪我们的数据汇总，让我们添加一个`struct`类型：

```
type summaries struct {
     cpuVendor cpuVendorSum
}
type cpuVendorSum struct {
     unknown, intel, amd int
}
```

让我们修改之前写的`add()`方法来总结我们的表格：

```
     ...
     s.xlsx.SetCellValue(s.sheetName, "D" + strconv.Itoa(s.nextRow), vendor)
     switch vendor {
     case Intel:
          s.sum.cpuVendor.intel++
     case AMD:
          s.sum.cpuVendor.amd++
     default:
          s.sum.cpuVndor.unknown++
     }
     s.nextRow++
     return nil
}
func (s *serverSheet) writeSummaries() {
    s.xlsx.SetCellValue(s.sheetName, "F1", "Vendor Summary")
    s.xlsx.SetCellValue(s.sheetName, "F2", "Vendor")
    s.xlsx.SetCellValue(s.sheetName, "G2", "Total")
    s.xlsx.SetCellValue(s.sheetName, "F3", Intel)
    s.xlsx.SetCellValue(s.sheetName, "G3", s.summaries.cpuVendor.intel)
    s.xlsx.SetCellValue(s.sheetName, "F4", AMD)
    s.xlsx.SetCellValue(s.sheetName, "G4", s.summaries.cpuVendor.amd)
}
```

上面的代码执行以下操作：

+   它查看我们的供应商并将其添加到我们的汇总计数器中。

+   它添加了一个方法，将我们的汇总写入工作表。

接下来，让我们讨论如何使用这些数据添加可视化。

### 添加可视化

使用 Excel 而不是 CSV 进行输出的原因之一是添加可视化元素。这使得你可以快速生成用户可以查看的报告，这些报告比 CSV 更具吸引力，而且比网页更容易编写。

添加图表是通过`AddChart()`方法完成的。`AddChart()`接受一个表示 JSON 的字符串，用于指示如何构建图表。在我们的示例中，你将看到一个名为`chart`的包，它提取了`excelize`中的私有类型，这些类型用于表示图表，并将其转换为公共类型。通过这种方式，我们可以使用一个类型化的数据结构，而不是已经转换成该结构的 JSON。这样也方便了发现你可能想要设置的值：

```
func (s *serverSheet) createCPUChart() error {
    c := chart.New()

    c.Type = "pie3D"
    c.Dimension = chart.FormatChartDimension{640, 480}
    c.Title = chart.FormatChartTitle{Name: "Server CPU Vendor Breakdown"}
    c.Format = chart.FormatPicture{
            FPrintsWithSheet: true,
            NoChangeAspect: false,
            FLocksWithSheet: false,
            OffsetX: 15,
            OffsetY: 10,
            XScale: 1.0,
            YScale: 1.0,
    }
    c.Legend = chart.FormatChartLegend{
            Position: "bottom",
            ShowLegendKey: true,
    }
    c.Plotarea.ShowBubbleSize = true
    c.Plotarea.ShowCatName = true
    c.Plotarea.ShowLeaderLines = false
    c.Plotarea.ShowPercent = true
    c.Plotarea.ShowSerName = true
    c.ShowBlanksAs = "zero"
    c.Series = append(
            c.Series,
            chart.FormatChartSeries{
                    Name: `%s!$F$1`,
                    Categories: fmt.Sprintf(`%s!$F$3:$F$4`, s.sheetName),
                    Values: fmt.Sprintf(`%s!$G$3:$G$4`, s.sheetName),
            },
    )
    b, err := json.Marshal(c)
    if err != nil {
            return err
    }
    if err := s.xlsx.AddChart(s.sheetName,  "I1", string(b)); err != nil {
            return err
    }
    return nil
}
```

这段代码执行以下操作：

+   它创建了一种新的 3D 饼图类型。

+   它设置了尺寸、标题和图例。

+   它应用了图表的值和类别。

+   它将图表的指令转化为 JSON 格式。

+   它调用`AddChart`将图表插入到工作表中。

你可以在以下代码库中找到可运行的代码：[`github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/5/excel/visualization`](https://github.com/PacktPublishing/Go-for-DevOps/tree/rev0/chapter/5/excel/visualization)

因此，我们已经涵盖了使用 Excel 输出报告的基本要求。还有许多其他选项，包括插入图片、数据透视表和高级格式化指令。尽管我们不推荐使用 Excel 作为系统的数据输入或数据存储格式，但它对于汇总和查看数据来说，是一个有用的数据输出系统。

# 流行的编码格式

CSV 是 DevOps 工程师会遇到的最基础的人类可读编码格式之一，但它绝不是唯一的。在过去的二十年里，出现了几种新格式，它们用于传输信息或为应用程序提供配置。

**JavaScript 对象表示法**（**JSON**）是一种数据序列化格式，旨在将 JavaScript 对象转换为文本表示形式，以便保存或传输。由于其简洁性和清晰性，这种标记法已经被几乎所有语言采纳，用于数据传输。

**另一种标记语言**（**YAML**）是一种数据序列化格式，常用于存储服务的配置信息。YAML 是 Kubernetes 集群的主要配置语言。

在这一部分，我们将讨论如何将数据从 Go 类型转换为这些格式，再从这些格式转换回 Go 类型的方式。

## Go 字段标签

Go 有一个叫做字段标签（field tags）的功能，允许开发人员向 `struct` 字段添加字符串标签。这使得 Go 程序在执行操作之前，可以查看有关字段的额外元数据。标签是键/值对：

```
type Record struct {
     Last string `json:"last_name"`
}
```

在前面的代码片段中，你可以看到一个 `struct` 类型，包含一个名为 `Last` 的字段，该字段具有字段标签。字段标签是一个内联原始字符串。原始字符串用反引号表示。这将生成一个键为 `"json"`、值为 `"last_name"` 的标签。

Go 包可以使用 `reflect` 包来读取这些标签。这些标签使得包能够根据标签数据更改操作的行为。在这个例子中，它告诉我们的 JSON 编码器包在写入 JSON 数据时使用 `last_name` 而不是 `Last`，反之亦然。

这个特性对处理数据序列化的包至关重要。

## JSON

在过去的十年中，JSON 格式已经成为数据编码到磁盘并通过 RPC 与服务通信的事实标准。在云领域，没有支持 JSON 的语言是无法成功的。

开发人员可能会遇到将 JSON 作为应用程序配置语言的情况，但由于以下原因，它并不适合这个任务：

+   缺乏多行字符串

+   无法添加注释

+   对标点符号的苛求（也就是说，机器适用，人类不适用）

对于数据交换，JSON 在一些小缺点的情况下仍然非常有用，以下是其中的一些：

+   无模式

+   非二进制格式

+   缺乏字节数组支持

架构（schema）是对消息内容的定义，它存在于代码之外。

无模式意味着没有严格的定义来说明一个消息包含什么内容。这意味着，对于每种受支持的语言，我们必须为该语言创建消息的定义。诸如协议缓冲区（protocol buffers）等格式已经进入这个领域，提供了一个可以用来为任何语言生成代码的架构。

JSON 也是一种人类可读的格式。这类格式在大小和速度上不如二进制格式高效。通常，当试图扩展大规模服务时，这一点很重要。然而，许多人更喜欢人类可读的格式，因为它们易于调试。

JSON 不支持字节数组也是一个缺陷。虽然 JSON 仍然可以传输原始字节，但它需要使用 `base64` 编码对字节进行编码和解码，并将其存储在 JSON 的 `string` 类型中。这需要额外的编码步骤，而这些步骤本不应存在。有几种 JSON 的超集（例如二进制 JSON，简称 BSON）包含字节数组类型，但它们并未广泛支持。

JSON 通过多种方式传递给用户：

+   作为一个可以包含子消息的单一消息

+   作为 JSON 消息的数组

+   作为 JSON 消息流

JSON 最初的起源是作为一种格式，用于简单地编码 JavaScript 对象以进行传输。然而，随着使用场景的增多，发送大消息或消息流的需求也成为了一个实际应用场景。

单个大消息可能很难解码。通常，JSON 解码器会读取整个消息到内存中，并验证消息的内容。

为了简化大量的消息集或流式内容，你可能会遇到一个被括号`[]`包围的消息集合，或者是由回车符分隔的单个消息。这些不是按预期的有效 JSON，但已经成为处理大量数据作为小的、单独的消息组成整个流的事实标准。

因为 JSON 是云生态系统中的标准部分，Go 语言在标准库的`encoding/json`包中提供了内置支持。在接下来的部分中，我们将详细介绍使用 JSON 包的最常见方法。

### 对`map`进行编码和解码

由于 JSON 没有固定的模式，因此在流或文件中可能存在不同类型的消息。这通常是不可取的，最好有一个顶级消息来包含这些不同类型的消息。

当你需要处理多种消息类型或对消息进行发现时，Go 允许你将消息解码成`map[string]interface{}`，其中`string`键表示字段名，`interface{}`表示值。

让我们来看一个将文件解码成`map`的示例：

```
b, err := os.ReadFile("data.json") 
if err != nil { 
    return "", 
    err
} 
data := map[string]interface{}{} 
if err := json.Unmarshal(b, &data); err != nil { 
    return "", err 
}
v, ok := data["user"]
if !ok {
     return "", errors.New("json does not contain key 'user'")
}
switch user := v.(type) {
case string:
     return user, nil
}
return "", fmt.Errorf("key 'user' is not a string, was %T", v)
```

上面的示例执行了以下操作：

+   它将`data.json`文件的内容读取到变量`b`中。

+   它创建一个名为`data`的`map`，用于存储我们的 JSON 内容。

+   它将原始字节解码为 JSON，存储到`data`中。

+   它查找`data`中的`user`键。

+   如果`user`不存在，我们返回一个错误。

+   如果确实存在，我们通过`type assert`来确定值的类型。

+   如果值是字符串，我们返回其内容。

+   如果值不是字符串，我们返回一个错误。

使用`map`，我们可以探索数据中的值以发现消息类型，`type` `assert` 将`interface{}`值断言为具体类型，然后使用该具体值。记住，类型断言将`interface`变量转换为另一个`interface`变量或具体类型，例如`string`或`int64`。

使用`map`是 JSON 数据解码中最复杂的方法。仅在 JSON 不可预测且无法控制数据提供者的情况下推荐使用这种方法。通常，最好是让数据提供者改变其行为，而不是以这种方式进行解码。

将`map`编码为 JSON 很简单：

```
if err := json.Marshal(data); err != nil {
     return err
}
```

`json.Marshal`会读取我们的`map`并输出有效的 JSON 内容。`[]byte`字段会自动以`base64`编码转换为 JSON 的`string`类型。

### 对结构体进行编码和解码

JSON 解码的首选方法是在 Go 的 `struct` 类型中进行，这个类型表示数据。以下是创建用户记录结构体的示例，我们将使用它来解码 JSON 流：

```
type Record struct {
     Name string `json:"user_name"`
     User string `json:"user"`
     ID int
     Age int `json:"-"`
}
func main() {
     rec := Record{
          Name: "John Doak",
          User: "jdoak",
          ID: 23,
     }
     b, err := json.Marshal(rec)
     if err != nil {
          panic(err)
     }
     fmt.Printf("%s\n", b)
}
```

上述代码输出 `{"user_name":"John Doak","user":"jdoak","ID":23}`。你可以在 [`play.golang.org/p/LzoUpOeEN9y`](https://play.golang.org/p/LzoUpOeEN9y) 找到可运行的代码。

这段代码做了以下操作：

+   它定义了一个 `Record` 类型。

+   它使用字段标签告诉 JSON 输出字段映射应为怎样。

+   它在 `Age` 字段上使用了 `-` 的字段标签，以便该字段不会被序列化。

+   它创建了一个名为 `rec` 的 `Record` 类型。

+   它将 `rec` 序列化为 JSON。

+   它打印出了 JSON。

请注意，`Name` 字段被转换为 `user_name`，`User` 转换为 `user`。`ID` 字段在输出中没有改变，因为我们没有使用字段标签。`Age` 字段没有输出，因为我们使用了 `-` 的字段标签。

由于以小写字母开头的字段是私有的，因此无法导出。这是因为 JSON 序列化器在另一个包中，无法看到当前包中的私有类型。

你可以在 `encoding/json` GoDoc 中阅读 JSON 支持的字段标签，位于 `Marshal()` 下 ([`pkg.go.dev/encoding/json#Marshal`](https://pkg.go.dev/encoding/json#Marshal))。

JSON 包还包括 `MarshalIndent()`，它可以用来输出更易读的 JSON，其中字段之间有行分隔符和缩进。

将数据解码为 `struct` 类型（例如前面的 `Record`）可以如下进行：

```
rec := Record{}
if err := json.Unmarshal(b, &rec); err != nil {
     return err
}
```

这将表示 JSON 的文本转换为存储在 `rec` 变量中的 `Record` 类型。你可以在 [`play.golang.org/p/DD8TrKgTUwE`](https://play.golang.org/p/DD8TrKgTUwE) 找到可运行的代码。

### 序列化和反序列化大型消息

有时，我们可能会收到包含 JSON 消息列表的 JSON 消息流或文件。

Go 提供了 `json.Decoder` 来处理一系列消息。以下是借自 GoDoc 的示例，其中每条消息由回车符分隔：

```
const jsonStream = `
     {"Name": "Ed", "Text": "Knock knock."}
     {"Name": "Sam", "Text": "Who's there?"}
`
type Message struct {
     Name, Text string
}
reader := strings.NewReader(jsonStream)
dec := json.NewDecoder(reader)
msgs := make(chan Message, 1)
errs := make(chan error, 1)
// Parse the messages concurrently with printing the message.
go func() {
     defer close(msgs)
     defer close(errs)
     for {
          var m Message
          if err := dec.Decode(&m); err == io.EOF {
               break
          } else if err != nil {
               errs <- err
               return
          }
          msgs <- m
     }
}()
// This will print the messages as we decode them.
for m := range msgs {
     fmt.Printf("%+v\n", m)
}
if err := <-errs; err != nil {
     fmt.Println("stream error: ", err)
}
```

你可以在 [`play.golang.org/p/kqmSvfdK4EG`](https://play.golang.org/p/kqmSvfdK4EG) 查看此运行中的代码。

这个例子做了以下操作：

+   它定义了一个 `Message` 结构体。

+   它通过 `strings.NewReader()` 将 `jsonStream` 原始输出包装在 `io.Reader` 中。

+   它启动了一个 goroutine，解码消息并将其放入通道中。

+   它读取所有发送的消息，直到输出通道被关闭。

+   它打印出遇到的任何错误。

有时，这种流格式会在消息周围加上括号 `[]`，并使用逗号作为条目之间的分隔符。

在这种情况下，我们可以利用解码器的另一个特性，`dec.Token()`，来安全地移除它们：

```
const jsonStream = `[
     {"Name": "Ed", "Text": "Knock knock."},
     {"Name": "Sam", "Text": "Who's there?"}
]`
dec := json.NewDecoder(reader)
_, err := dec.Token() // Reads [
if err != nil {
     return fmt.Errorf(`outer [ is missing`))
}
for dec.More() {
     var m Message
     // decode an array value (Message)
     err := dec.Decode(&m)
     if err != nil {
          return err
     }
     fmt.Printf("%+v\n", m)
}
_, err = dec.Token() // Reads ]
if err != nil {
     return fmt.Errorf(`final ] is missing`)
}
```

你可以在 [`play.golang.org/p/_PrUVUy4zRv`](https://play.golang.org/p/_PrUVUy4zRv) 查看此运行中的代码。

这段代码以相同的方式工作，只是它移除了外括号，并且要求使用逗号分隔的列表。

在流中编码数据与解码非常相似。我们可以将 JSON 消息写入 `io.Writer`，以输出到流。以下是一个示例：

```
func encodeMsgs(in chan Message, output io.Writer) chan error {
     errs := make(chan error, 1)
     go func() {
          defer close(errs)
          enc := json.NewEncoder(output)
          for msg := range in {
               if err := enc.Encode(msg); err != nil {
                    errs <- err
                    return
               }
          }
     }()
     return errs
}
```

你可以在 [`play.golang.org/p/ELICEC4lcax`](https://play.golang.org/p/ELICEC4lcax) 查看这段代码的运行情况。

这段代码的作用如下：

+   它从 `Message` 类型的 `channel` 中读取数据。

+   它写入 `io.Writer`。

+   它返回一个信号通道，表示编码器完成处理。

+   如果返回了错误，意味着编码器遇到了问题。

这将 JSON 输出为分隔的值，不带括号。

### JSON 的最终思考

`encoding/json` 包支持其他解码方法，这些方法在这里未涉及。你可以将 `map[string]interface{}` 混合到 `struct` 类型中，反之亦然，或者你可以逐个解码每个字段和数值。

然而，最佳的使用场景是那些直接的 `struct` 类型，作为单个值或值流。

这就是为什么 `encoding/json` 是我在编码或解码 JSON 值时的首选方法。它不是最快的，但它是最灵活的。

还有其他第三方库可以提高吞吐量，但会牺牲一些灵活性。这里列出了一些你可能想考虑的包：

+   [`github.com/francoispqt/gojay`](https://github.com/francoispqt/gojay)

+   [`github.com/goccy/go-json`](https://github.com/goccy/go-json)

+   [`pkg.go.dev/github.com/json-iterator/go`](https://pkg.go.dev/github.com/json-iterator/go)

+   [`pkg.go.dev/github.com/valyala/fastjson`](https://pkg.go.dev/github.com/valyala/fastjson)

## YAML 编码

YAML（另一个标记语言/YAML 不等于标记语言）是一种常用于编写配置的语言。

YAML 是 Kubernetes 等服务的默认语言，用于保存配置，作为 DevOps 工程师，你很可能会在各种应用中遇到它。

YAML 相对于 JSON 在配置使用中有一些优势：

+   支持注释

+   对人类更灵活，如不带引号的字符串和带引号的字符串

+   多行字符串

+   使用锚点和引用来避免重复相同的文本数据

YAML 经常被认为有以下缺点：

+   它是无模式的。

+   规范很庞大，某些功能可能令人困惑。

+   大文件可能会出现缩进错误而未被注意到。

+   某些语言中的实现可能会意外执行嵌入在 YAML 中的代码。这可能会导致软件项目中的一些安全补丁。

Go 标准库并不原生支持 YAML，但它有一个第三方库，已经成为 YAML 序列化的事实标准包，名为 `go-yaml` ([`github.com/go-yaml/yaml`](https://github.com/go-yaml/yaml))。

接下来，让我们讨论如何读取这些 YAML 文件来读取我们的配置。

### 对映射的序列化和反序列化

YAML 和 JSON 一样是无模式的，并且有相同的缺点。然而，与 JSON 不同的是，YAML 旨在表示配置，因此我们不需要像处理 JSON 那样去流式处理内容。

对于 YAML，一般的使用情况将涉及编码/解码为 `struct` 类型而不是 `map`。然而，如果您需要消息发现，YAML 可以像我们处理 JSON 一样处理 `map` 解码。

让我们看一个将文件解组为 `map` 的示例：

```
data := map[string]interface{}{}
if err := yaml.Unmarshal(yamlContent, &data); err != nil {
     return "", err
}
v, ok := data["user"]
if !ok {
     return "", errors.New("'user' key not found")
}
```

前面的示例做了以下几件事情：

+   它创建了一个名为 `data` 的 `map` 来存储我们的 YAML 内容。

+   它将表示 YAML 的原始字节解组为 `data`。

+   它查找 `data` 中的 `user` 键。

+   如果 `user` 不存在，则返回错误。

要查看更完整的示例，请参阅 [`play.golang.org/p/wkHkmu47e6V`](https://play.golang.org/p/wkHkmu47e6V)。

将 `map` 编组为 YAML 是简单的：

```
if err := yaml.Marshal(data); err != nil {
     return err
}
```

在这里，`yaml.Marshal()` 将读取我们的 `map` 并为其内容输出有效的 YAML。

### 编组和解组为结构体

`struct` 序列化是处理 YAML 的首选方式。由于 YAML 是一种配置语言，程序必须预先知道可用的字段以设置程序参数。

YAML 序列化的工作方式与 JSON 序列化类似，您会在大多数数据序列化包中找到这种相似性：

```
type Config struct {
     Jobs []Job
}
type Job struct {
     Name     string
     Interval time.Duration
     Cmd      string
}
func main() {
     c := Config{
          Jobs: []Job{
               {
                    Name:     "Clear tmp",
                    Interval: 24 * time.Hour,
                    Cmd:      "rm -rf " + os.TempDir(),
               },
          },
     }
     b, err := yaml.Marshal(c)
     if err != nil {
          panic(err)
     }
     fmt.Printf("%s\n", b)
}
```

您可以在 [`play.golang.org/p/SvJHLKBsdUP`](https://play.golang.org/p/SvJHLKBsdUP) 上看到这段正在运行的代码。

这将输出以下内容：

```
jobs:
- name: Clear tmp dir
  interval: 24h0m0s
  cmd: rm -rf /tmp
```

前面的代码做了以下几件事情：

+   它创建了一个名为 `Config` 的顶级配置。

+   它创建了一个名为 `Job` 的子消息列表。

+   它将示例编组为文本表示。

解组同样简单：

```
     data := []byte(`
jobs:
  - name: Clear tmp
    interval: 24h0m0s
    whatever: is not in the Job type
    cmd: rm -rf /tmp
`)
     c := Config{}
     if err := yaml.Unmarshal(data, &c); err != nil {
          panic(err)
     }
     for _, job := range c.Jobs {
          fmt.Println("Name: ", job.Name)
          fmt.Println("Interval: ", job.Interval)
     }

```

前面的代码做了以下几件事情：

+   它接受由数据表示的 YAML 配置。

+   它将其转换为 `Config` 类型。

+   它打印出包含的 `Job` 信息。

+   它忽略了 `whatever` 字段。

此代码将忽略未知的 `whatever` 字段。然而，在许多情况下，您不希望忽略可能拼写错误的字段。在这些情况下，我们可以使用 `UnmarshalStrict()`。

这将导致此代码失败，并显示以下消息：

```
line 5: field whaterver not found in type main.Job
```

使用 `UnmarshalStrict()` 时，您必须在将其添加到配置文件之前将新字段支持到您的程序中并部署它们，否则会导致旧的二进制文件失败。

### YAML 最终思考

`github.com/go-yaml/yaml` 包支持其他我们这里不会涵盖的序列化方法。其中最常用的一种是解码为 `yaml.Node` 对象，以保留注释，然后更改内容并重新写入配置。然而，这相对不常见。

在本节中，您已经学会如何使用 JSON 和 YAML 读取和写入它们各自的数据格式。在下一节中，我们将探讨如何与常用于存储数据的 SQL 数据源进行交互。

# 总结

这也标志着我们关于使用常见数据格式的章节结束。我们已经涵盖了如何读取和写入 CSV 文件以及 Excel 报告。此外，我们还学习了如何在 JSON 和 YAML 格式中进行数据编码和解码。本章节展示了如何在流中解码数据，同时强化了使用 goroutine 并发读取和使用数据的方法。

你刚学到的 JSON 技能将在我们下一章立即派上用场。在那一章中，我们将学习如何连接 SQL 数据库并与 RPC 服务进行交互。由于 REST RPC 服务和像 Postgres 这样的数据库可以使用 JSON，这项技能将非常有用。

那么，我们开始吧！
