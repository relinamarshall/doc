---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# 分析函数

## 分析函数剖析

分析函数具有3个基本组成部分:分区子句,排序子句以及开窗子句。分析函数的基本语法是:

{% code lineNumbers="true" %}
```sql
function1 (argument1,...) 
    over([partition-by-clause] [order-by-clause] [windowing-clause])
```
{% endcode %}

Function1是所调用的接收0个或多个参数的分析函数。分区子句按照分区列的值对数据行进 行分组。所有分区列的值相同的数据行被组合为一个数据分区。

从操作上来说，数据行按照分区列进行排序并被分为数据分区。例如，SQL子旬`partition by product，country`使用Product列和Country列的值进行分区。数据行按照Product和Country两列的值进行排序并按照产品和国别的组合来进行分区。

排序子句通过一列或一个表达式的值来对数据分区中的行进行排序。在一个分析型SQL语句中，一个数据行在数据分区中的位置是很重要的，并且是由排序子句来控制的。在一个数据分区内的数据行按照排序列的值进行排序。因为在分区子句中按照分区列的值来进行排序，你实际上最终得到的是按照分区子句和排序子句中指定的列来进行排序后的结果。

排序可以按照升序也可以按照降序。使用`NULLS FIRST`或`NULLS LAST`子句可以将空值放到数据分区的最上面或最下面。

开窗子句指定了分析函数进行运算的数据子集。这个窗口可以是动态的并且被很恰当地称为滑动窗口。你可以使用窗口说明子句来指定滑动窗口的上下边界条件。窗口说明子句的语法如下:

{% code lineNumbers="true" %}
```sql
[ROWS | RANGE] BETWEEN <Start expr> AND <End expr>
Whereas
<Start expr> is [UNBOUNDED PRECEDING | CURRENT ROW | n PRECEDING | n FOLLOWING]
<End expr> is [UNBOUNDED PRECEDING | CURRENT ROW | n PRECEDING | n FOLLOWING]
```
{% endcode %}

关键字`PRECEDING`指定了窗口的上边界条件，`FOLLOWING`或`CURRENT ROW`子句指定了窗口的下边界条件。滑动窗口提供了简便的复杂矩阵计算能力。例如，你可以使用子句`rows between unbounded preceding and current row`来对Sale列动态求和。在这个例子中，窗口最上面的一行是当前分区中的第一行而窗口最下面一行是当前数据行。

{% hint style="info" %}
并不是所有分析函数都支持开窗子句。
{% endhint %}

然后在视图之外分析函数不能进行嵌套。但可以通过将所包含的SQL语句放在内嵌视图中，使用分析函数来实现嵌套效果。分析函数也可以被用在多层嵌套内嵌视图中。

{% hint style="info" %}
默认的窗口子句是`rows between unbounded preceding and current row`。如果你没有显式声明窗口，就将会使用默认窗口。显式声明这个子句是避免模两可的好办法。
{% endhint %}

## 函数列表



<table><thead><tr><th width="197">函数</th><th>描述</th></tr></thead><tbody><tr><td>Lag</td><td>访问一个分区或结果集中之前的一行</td></tr><tr><td>Lead</td><td>访问一个分区或结果集中之后的一行</td></tr><tr><td>First_value</td><td>访问一个分区或结果集中第一行</td></tr><tr><td>Last_value</td><td>访问一个分区或结果集中最后一行</td></tr><tr><td>Nth_value</td><td>访问一个分区或结果集中的任意一行</td></tr><tr><td>Rank</td><td>将数据行值按照排序后的顺序进行排名,在有并列的情况下排名值将被跳过</td></tr><tr><td>Dense_rank</td><td>将数据行值按照排序后的顺序进行排名,在有并列的情况下也不跳过排名值</td></tr><tr><td>Row_number</td><td>对行进行排序并为每一行增加一个唯一编号。这是一个非确定性函数</td></tr><tr><td>Ratio_to_report</td><td>计算报告中值的比例</td></tr><tr><td>Percent_rank</td><td>将计算得到的排名值标准化为0到1之间的值</td></tr><tr><td>Percentile_cont</td><td>取出与指定的排名百分比相匹配的值。是<code>percent_rank</code>函数的反函数</td></tr><tr><td>Percentile_dist</td><td>取出与指定的排名百分比相匹配的值。采用谨慎分布模型</td></tr><tr><td>Ntile</td><td>将数据行分组为单元</td></tr><tr><td>Listagg</td><td>将来自不同行的列值转化为列表格式</td></tr></tbody></table>

## 其他分析函数

### Lead和Lag

Lag和Lead函数提供了跨行引用的能力。Lag提供了访问结果集中前面的行的能力，Lead函数允许访问结果集中后面的行。

如前面所讨论的，分析型SQL中的数据是按照分区列的值来进行分区的。获取前一行的值是一个与位置相关的运算,数据分区中各行的顺序对于维护逻辑上的一致性是很重要的。在一个数据分区内部，数据行通过orderby子句来进行排序以控制某一行在结果集中的位置。Lag函数的语法如下:

```sql
lag(expression,offset,default) over(partition-clause order-by-clause)
```

Lead和Lag函数不支持开窗子句。这两个函数仅支持`partition by`子句和`order by`子句。

> 理解数据行的位移

通过指定不同的位移可以来访问一个数据分区中的所有行。Lag函数使用了位移量10来访问往前第10行的数据。假设显示了在`Year=2001`，`Week=52`那一行，Lag函数来访问结果集中往前第10行，也就是第40周的数据。**注意并不是访问将当前周的值52减去10得到的第42周的数据，而是访问分区中往前推10行的数据**。在这个例子中，往前第10行是Week列值为40的数据行。

这个问题是比较棘手的，因为通常在开发环境中不会检测到数据缺口。但是在生产环境中，就为一个缺陷(bug)，你可以为缺少的行填充虚拟值。

### First\_value和Last\_value

`first_value`和`last_value`函数在计算排过序的结果集中的最大值和最小值的时候是很有用的。`first_value`函数从一个数据行窗口中第一行获取列值,而`last_value`函数从该窗口中最后一行数据获取列值。从本质上来说，任何计算最大值和最小值的报表都可以使用`first_value`和`last_value`函数。

`first_value`和`last_value`函数的能力来自于对分区子句和开窗子句的支持。多层级聚合可以使用分区子句来精确地实现。使用开窗子句，你可以为这些函数的运算定义动态滑动窗口。这个窗口可以定义为包含几个之前以及/或之后的数据行或者是包含数据分区中的所有行。特别是，计算例如到目前为止最大销售额这样的矩阵可以使用这些函数来实现。

First value函数的语法是:&#x20;

```plsql
first_value(expression)over(partition-clause order-by-clause windowing-clause)
```

### Nth\_value

Oracle数据库11gR2中新引人了另一个分析函数:`nth_value`，这是`first_value`和`last_value`函数的通用化版本。使用`nth_value`函数，你可以获取排过序的结果集中的任意一行，而不仅是第一行或最后一行。`First_value`函数可以被写为nthvalue(`column_name`，1)。在统计分析中，在结果集的头部或尾部可能会出现异常值。在某些情况下，忽略`first_value`或经过排序的结果集中的`first_value`而从第二行开始取值是很重要的。结果集中的第2个值可以通过使用`nth_value`函数并将位移值设为2来取得。

`Nth_value`函数的语法如下:

```sql
NTH_VALUE(column, n)[FROM FIRST|FROM LAST] [RESPECT NULLS|IGNORE NULLS]
    OVER(partitioning-clause order-by-clause windowing-clause)
```

Nth\_value函数的第1个参数是列名，第2个参数为窗口位移量。对于`nth_value`函数，`FROM FIRST`和`RESPECT NULLS`子句是默认值。如果声明了`FROM FIRST`子句则`nth_value`函数从窗口的第1行开始寻找移位后的数据行。`RESPECT NULLS`子句表示如果在移位行中包含空值则将会返回空值。

具有了声明开窗子句的能力，`nth_value`函数就变得非常强大，可以访问结果集或分区中的任意行。

### Rank

`Rank`函数以数值形式返回一个数据行在排序后的结果集中的位置。如果数据行是按某一列进行排序的，则这一行在窗口中的位置就反映了该值在窗口内数据行中的排名。在排名并列的情况下，具有同样值的行将具有同样的排名而接下来的排名就会被跳过，从而在排名值上留下空隙。这意味着某两行可能具有同一排名，排名也不一定是连续的。

`Rank`函数对于计算最上面或最下面N行是非常有用的。例如，查找销售量在前10位的周就是零售业数据仓库中一个典型的查询。这样一个查询将会大大获益于rank函数的使用。如果你需要写查询来计算某个结果集中最上面或最下面N个元素的值，就可以使用rank或dense\_rank函数。Rank函数对于找出中间的N行数据也是很有用的。例如，如果目标是取出按销售额排序的21位到40位的数据，那么你可以在一个具有谓语between21 and 40的子查询中使用rank函数来筛选出中间的20行数据。

Rank函数的语法如下:&#x20;

```sql
rank() over(partition-clause order-by-clause)
```

### Dense\_rank

`Dense_rank`是`rank`函数的变体。`Rank`和`dense_rank`函数的区别在于当存在并列的时候`dense_rank`函数不会跳过排名值。如在上一节所讨论的，`dense_rank`函数对于查找结果集中顶部、底部或中间N行的数据是非常有用的。`Dense_rank`函数在排名值需要连续的时候是很有用的。例如，在一个班级的学生花名册上排名前10的学生就不能被跳过。另一方面，`rank`函数在排名值不需要连续的时候是很有用的。

`Dense_rank`函数中空值的排序位置可以通过`NULLS FIRST`或`NULLS LAST`子句来控制。对于升序排列来说，`NULLS LAST`是默认值，而对于降序排列来说，`NULLS FIRST`是默认值。降序排列，的`NULLS FIRST`子句起作用。在这种情况下空值的排名为1。

### Row\_number

`Row_number`函数为有序结果集中的每一行分配唯一的行编号。如果声明了分区子句，则为每一行分配一个基于其在该有序分区中位置的唯一编号。如果没有声明分区子句，则为结果集中的每一行分配唯一编号。&#x20;

`Row_number`函数对于获取顶部、底部或中间N行数据的查询也是很有用的，与rank和`dense_rank`函数类似。尽管`rank`、`dense_rank`和row\_number函数具有类似的功能，在它们之间还是有很微妙的区别的。其中一个区别就是`row_number`函数不支持开窗子句。

`Row_number`函数的语法如下:&#x20;

```sql
Row_number() over(partition-clause order-by-clause) 
```

`Row_number`函数是一个非确定性函数。如果一个数据分区中的两行具有同样的值,`row_number`函数的值是不确定的。例如四行值一样，`row_number`函数为各行返回的值分别为31、32、33、34。但结果也很可能会是34、31、32、33或32、34、31、33。事实上，执行这个查询你可能会得到不同的结果。相反地，`rank`和`dense_rank`函数是确定性函数，如果重复执行查询将会返回一致的数据。

### Ratio\_to\_report

分析函数`ratioto_report`计算数据分区中某个值与和值的比率。如果没有声明分区子句，这个函数将会计算一个值与整个结果集中和值的比率。这个分析函数对于在不同层级上计算比率是非常有用的，不需要进行自联结。

`ratio_to_report`在计算报表中某个值占总值的百分比的时候是很有用的。例如，考虑某个零售连锁店中某种产品的销售报表。每家门店都对该产品的销售总额做出了贡献，并且知道每家门店的销售额占总销售额的百分比对于市场趋势分析是非常有用的。`ratio_to_report`允许你很方便地计算百分比。从本质上来讲，数据可以被以多种不同的方式切块或切片来进行市场趋势分析。

`Ratio_to_report(sale) over(partition by product,country,region,year)`子句计算按照`Product、Country、Region`和`Year`列进行分区的数据分区中Sale列的值相对于这一列的和值的比率接下来的子句`ratio to_report(sale) over(partition by product,country,region)`的不同之处在于Year列没有包含在分区列中，因此这个比率是针对所有年份计算出来的。

如果函数中所指定的表达式或列返回空值，则`ratio_toreport`函数也将会返回空值。但数据分区中的其他空值将会被当做零值或空字符串来处理，与聚合函数类似。

### Percent\_rank

`Percent_rank`函数以0到1之间的分数形式返回某个值在数据分区中的排名。`Percent_rank`的计算公式为`(rank-1)/(N-1)`其中如果声明了分区子句N就是分区中的数据行数，如果没有声明分区子句N就是结果集中所有的数据行数。`Percent_rank`函数对于计算某个值在结果集中按百分比所处的相对位置是很有用的。

`percent_rank() over(partition by product, country, region, year order by sale desc)`子句在由分区列`Product`、`Country`、`Region`和`Year`所定义的数据分区上计算Sale列值的百分比。数据行按照Sale列的降序排列。为得出百分比，计算的结果可乘以100。

### Percentile\_cont

`Percentile_cont`函数对于计算内插值(例如每个地区或城市中等收入家庭的收入)是非常有用的。`Percentile_cont`函数接收一个0到1之间的几率值并返回与声明了排序的`percent_rank`函数计算值相等的内插值百分比。事实上，`percentile_cont`函数是`percent_rank`函数的反函数，与`percent rank`函数的输出结合起来看可以更容易地来理解`percentile_cont`的数。

`Percentile_cont`函数取与参数的`percent_rank`相匹配(或内插的)的列值。例如，`percentile_cont`(0.25)子句获取`percent_rank`为0.25的值，假设这两个函数的排序顺序相匹配。另一个例子是计算一个城市或地区中等收入家庭的收入值。由于定义是中等收入家庭，中位值的`percent_rank`为0.5。`percentile_cont(0.5)`子句将会返回中位值，因为`percentile_cont`函数计算的值`percent_rank`为0.5。实际上，`median`函数是`percentile_cont`函数的一个默认值为0.5的特例。

空值将被这个函数忽略。同时这个的数也不支持开窗子句。`Percentile_cont`函数的语法是:&#x20;

```sql
Percentile_cont(expr) within group(sort-clause)
    over(partition-clause order-by-clause)
```

`Percentile_cont`函数的语法与到目前为止所讨论的分析函数的语法稍有不同。一个新的子句`within group(order by sale desc)`取代了之前的`order-by`子句，但在功能上与声明一个`order-by`子句是一样的。`percentile_cont(0.5) within group(order by sale desc) over( partition by product,country,region,year)`子句调用了`percentile_cont`函数并传入了几率值0.5。排列顺序通过`within group(order by sale desc)`子句来定义。`Partition-by`子句`over( partition by product，country，region，year)`用来声明分区列。

### Percentile\_disc

`Percentile_disc`函数在功能上类似于`percentile_cont`函数，只是`percentile_cont`函数使用了连续分布模型，而`percentile_disc`函数使用了离散分布模型。如上一节所讨论的，当没有值与指定的`percent_rank`精确匹配的时候,`percentile_cont(0.5)`会计算两个离得最近的值的平均值相反,在升序排列的情况下,`percentile_disc`函数只取比所传递的参数`percent_rank`值更大的值。在隆序排列的时候，`percentile_disc`函数只取比所传递的参数`percent_rank`值更小的值。

### NTILE

`NTILE`函数对一个数据分区中的有序结果集进行划分，将其分组为各个桶，并为每个小组分配一个唯一的组编号。这个函数在统计分析中是很有用的。例如，如果想移除异常值(正常值以外的值)，你可以将它们分组到顶部或底部的桶中，然后在统计分析的时候将这些值排除。Oracle数据库统计信息收集包也使用NTILE函数来计算直方图信息边界。在统计学术语中，NTILE函数创建等宽直方图信息。

桶的数目作为一个参数传递给该分析函数。例如，`ntile(100)`将会将数据行分组为100个桶并为每个桶分配唯一编号。但是，这个的数不支持开窗子句。

使用`ntile(10)`子句来将一个数据分区划分为10个桶。数据行按照Sale列的降序排列。`NTILE`函数将数据行划分为多个桶，每个桶中行的数目相等。由于数据行是按照Sale列降序排列的，组编号较小的数据行Sale列的值更大。通过这一技术可以很容易地来剔除异常数据。 **如果数据行不能被等分，各桶之间的行数最多相差1行**。

### Stddv

`Stddev`函数可以用来在一个数据分区中的某些数据行上，或者如果没有声明分区子句的话在整个结果集上计算标准偏差。这个函数为分区子句所指定的数据分区计算标准偏差，定义为方差的平方根。如果没有声明分区子句，就将在结果集中的所有数据行上计算标准偏差。

### Listagg

Oracle数据库11gR2版本中引入了另一个分析函数，在进行字符串处理时很有用的`listagg`函数。这个分析函数提供了将来自多个行中的列值转化为列表格式的能力。例如，如果你要把一个部门中所有员工的名字连起来，那么你可以使用这个函数将所有名字放到一个列表中。

这个函数的语法格式如下:&#x20;

```sql
Listagg(string,separator)within group(order-by-clause)
    over(partition-by-clause)
```

`Listagg`函数的语法中使用`within group(order-by-clause)`子句来声明排序顺序。这个子句与其他分析函数中的order-by子句是类似的。这个函数的第1个参数是需要进行连接的字符串或列名。第2个参数是值的分隔符。

## 性能调优

分析函数对于复杂SQL语句的性能调优是非常有用的。跨行引用、在不同层级上进行聚合以及对第n行的访问是分析函数所提供的几个重要的特性。例如，典型的SQL查询如果要同时获取聚合的和非聚合的数据行必须要进行自联结。在数据仓库环境中,由于其中所包含表的绝对大小， 这种自联结从成本上来说是禁止的。

分析函数所提供的高效率通常使得它们成为重写性能不佳的查询的有效工具。但是,相应地你有时候也需要对分析函数进行调试。对于这一点来说，关于分析函数和执行计划、分析和谓语以及索引策略方面你需要知道一些很有用的事实。

### 执行计划

分析函数在SQL的执行计划中几乎没有引入新的运算。关键字`WINDOW SORT`的出现表明SQL语句使用了一个分析函数。在本节中，我将回顾分析函数执行的结构。

下面给出了一个典型的SQL语句执行计划。这个计划从第4步开始执行,直到第1步。&#x20;

* (4)SALES FACT表使用全表扫描访问路径进行访问。
* (3)Product、Country、Region和Year列上的筛选谓语被用来筛选出所需的数据行。
* (2)在第3步中所筛选出来的数据行上应用分析函数。
* (1)在这些分析函数执行完后应用Week列上的谓语。

<figure><img src="../.gitbook/assets/image (29).png" alt=""><figcaption></figcaption></figure>

{% hint style="info" %}
基于成本的优化器不为分析函数分配或计算成本(从11gR2开始)。计算出的SQL语句成本并未考虑分析函数成本。
{% endhint %}

### 谓语

谓语应该尽可能早地应用于表上来减小结果集以获得更好的性能。数据行必须尽早进行筛选，从而可以在相对较少的数据行上应用分析函数。在执行分析函数时谓语安全性是需要考虑的很重要的一方面，因为并不是所有的谓语都能够在分析函数之前应用。

在下列代码清单定义了一个max\_5\_weeks\_vw视图并通过一个含有Country、Product、Region、Year和Week列上谓语的SQL语句来访问该视图。

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption></figcaption></figure>

然而，谓语`"WEEK"<14`并没有在第3步中应用，而只是在第1步中应用了，表明这个谓语是在第2步的窗口排序步骤中执行完分析函数以后应用的。除了这个Week列上的谓语以外，其他谓语都被推进了视图中。那些谓语的筛选也都在分析函数执行之前进行。分区列上的谓语在执行分析函数之前应用，因为一般来说，分区列上的谓语可以很安全地推人到视图中。但分析函数语法中order-by子句中的列不能被安全地前推，因为跨行引用需要访问同一分区中的其他数据行，即使这些数据行并不在最终的结果集中返回。

### 索引

好的索引选择策略是与表访问步骤中的谓语相匹配的。如上一节中所讨论的，分区列上的谓语被前推到视图中，并且这些谓语在执行分析函数之前应用。因此，如果SQL语句使用这些谓语的话，可能对分区列进行索引是更好的方法。

下列代码中，在`Country`和`Product`列上增加了一个新索引。执行计划第4步显示使用了基于索引的访问。谓语信息部分显示所有4个分区列上的谓语都在执行分析函数之前的第4步和第3步中应用。但`Week`列上的谓语直到执行计划中的第1步才进行了应用。因此，在这个例子中，将Week列加入索引是没有用的，因为直到分析函数执行完成之后才会应用这一列上的谓语。

<figure><img src="../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

## 高级话题

关于分析函数的几个高级话题是值得讨论的。我将讨论诸如动态分析语句、分析函数的嵌套并行以及PGA大小等几个话题。

### 动态SQL

关于分析SQL语句的一个普遍问题是是否可以在分区或排序列上使用绑定变量。不可以。如果想要灵活地动态修改分区或排序列，你需要使用动态SQL语句。静态分析SQL语句不能改变分 区列或排序列。

如果你的目的是动态调整分区列，那么考虑创建一个存储过程包来获取存储过程中的逻辑。在下列代码清单中，存储过程`analytic_dynamic_prc`接收一个字符串作为分区列。使用传进去的参数构造了一个SQL语句，并使用`Execute_immediate`语法来动态执行。分析语句的结果被放到一个数组中并调用`dbms_output`包来进行打印。

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption></figcaption></figure>

在第1个调用中，`analytic_dynamic_prc`传递字符串`product`、`country`及`region`作为第一个参数,这个列表中的列就被用做分区列。对存储过程的第2次调用使用字符串`product`、`country`、`region`及`year`来为分区子句指定不同的列的列表。

注意这个存储过程是作为一个例子给出的，因此可能并不是适用于生产环境中的代码。

### 嵌套分析函数

分析函数不能进行嵌套，但可以通过使用子查询来实现嵌套的效果。例如，`lag(first_value(column,1),1)`子句的语法就是错误的。下面你将会看到，子查询可以用来产生嵌套效果。

假设你的目标是要在同一行中列出今年和去年Sale列的最大值，如果是这样，那么可以在子查询中使用`lag`和`first_value`分析函数来写出SQL语句。在下列代码清单中，内层子查询给出出现Sale列最大值的Year和Week列的值，外层查询中的lag函数取出去年的Sale列最大值。

注意`lag`和`first_value`函数的分区子句是不同的。分析函数`first_value`用来在由分区列`product`、`country`、`region`及`year`所指定的分区上计算Sale列的最大值，而`lag`是取仅声明了`order by year desc`排序子句的前一年Sale列的第一行。 通过多层级分析函数嵌套，可以使用分析函数来准确实现复杂目标。

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption></figcaption></figure>

### 并行

通过在SQL语句中声明`parallel`提示或在对象级设置并行度，分析函数也可以是并行的。如果你有大量的数据需要通过分析函数进行处理，并行是一个很好的选择。使用多层级嵌套的SQL语 句也可以从并行中获益。

下列代码给出了上面代码中的查询使用并行的执行计划。在这个执行计划中有两个`WINDOW`运算，因为SQL语句中嵌套了`lag`和`first_value`分析函数。在并行活动(`PQ slaves`)之间数据行的最优分布对于维护功能的正确性是很关键的，这由Oracle数据库自动进行处理。

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption></figcaption></figure>

### PGA大小

大多数与分析函数相关的运算都是在进程的程序共享区(`Program Global Area，PGA`)中来进行的。因此，为了得到最优的性能，有一个足够大的内存区域以便程序能够不必使用硬盘来执行分析函数是很重要的。对于排序运算也是这样的。如果排序运算由于较低的内存大小而使用了硬盘，那么排序运算的性能就不是最优的。类似地，如果运算外溢到硬盘中，分析函数的执行性能也将降低。

数据库初始化参数`PGA_AGGREGATE_TARGET(PGAT)`控制着PGA的最大大小。默认地，一个串行进程最大可以分配到的PGA为PGAT值的5%。对于并行进程，最大限制为PGAT的30%。将PGAT保持在一个较高的值对于提高分析函数的性能是很重要的。

## 组织行为

关于分析函数最困难的就是来自组织中的对改变的阻力。开发人员和数据库管理员都喜欢使用传统的语法来写SQL语句。使用分析语法并不是那么轻而易举。但是这些开发人员和数据库管理员也需要欣然接受变化。另外一点:使用分析函数强迫使用基于集合的思维方式。

Oracle公司在每一次Oracle数据库的主发布中都会发布新特性。需要运用这些新特性来写出更高效和简洁的SQL语句。对这些新特性的适当培训也是很必要的。如所期望的那样，本章深入讲解了分析函数。

当你开始使用分析函数来写SQL语句的时候，你需要从较简单的SQL语句开始。然后一点点增加复杂度来实现目标。

## 小结

可以使用分析函数来很简洁地写出复杂SQL语句。从分析的角度来说，对分析函数的理解为你提供了一种全新的思维方式。与分区和开窗子句相结合来引用另一行的能力使得你可以对复杂SQL进行简化。可以通过使用分析函数改写SQL语句来解决很多性能问题。刚开始使用分析函数的时候你可能会遇到来自开发人员和数据库管理员之类的阻力，但只要给他们展示出使用分析函数得到的性能上的提升，这个阻力就很容易克服。
