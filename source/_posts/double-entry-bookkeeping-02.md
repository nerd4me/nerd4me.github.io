---
title: 程序员复式记账指南（下）
index_img: /img/beancount-bookkeeping.png
tags:
  - Beancount
  - Bookkeeping
abbrlink: 858cde56
date: 2023-01-17 21:24:12
---

文接上篇 [程序员复式记账指南（上）](https://nerd4me.github.io/posts/7a37cdfc.html)，上篇文章的最后，我们安装了 Beancount 基本环境，并通过官方提供示例账本，对其有了一个初步的认识。

## 一、基础

现在可以开始着手写我们的第一个账本了。怎么写？Beancount 作者提供了非常详尽的[使用文档](https://beancount.github.io/docs/index.html)

比如老王如果使用 Beancount 记账的话，他的账本将是这样的

```beancount
1970-01-01 open Income:Sales CNY
1970-01-01 open Assets:Bank:Saving:ICBC CNY
1970-01-01 open Assets:Fixed CNY ;固定资产
1970-01-01 open Liabilities:CreditCard:ICBC CNY
1970-01-01 open Expenses:Food CNY

2023-01-01 * "xx公司" "购买设备"
  Assets:Bank:Saving:ICBC      -1,000.00 CNY
  Assets:Fixed                  1,000.00 CNY

2023-01-01 * "采购食材"
  Liabilities:CreditCard:ICBC   -500.00 CNY
  Expenses:Food                  500.00 CNY

2023-01-02 * "煎饼销售收入"
  Income:Sales                -1,000.00 CNY
  Assets:Bank:Saving:ICBC      1,000.00 CNY

2023-01-02 * "质量问题食材退货"
  Expenses:Food                 -100.00 CNY
  Liabilities:CreditCard:ICBC    100.00 CNY
```

老王的账本里首先要设立账户。开户日期可随自己的喜好去定义，只需要比最早一笔涉及到该账户的交易早即可，这里偷了个懒，统一使用1970年1月1日作为开启日期，保证之后记录的所有都发生在这个日期之后。

然后就可以真正开始记账了，交易格式如上所示。其中日期后面的星号（*）代表这是一笔已确认的交易，如果换成（!）号的话，则代表这笔交易有疑问，后期对账时应注意。紧跟在对账标识后面的是「收款人」（Payee）和「交易备注」（Narration），需要使用双引号括起来。Payee 是可选的，像上面的示例中只有一串字符串的话，其代表的就是 Narration 了。

老王的账本已经写完了，纯手工书写，肉眼阅读也没啥障碍。那么 Beancount 有什么用？那当然是生成报表啦：

```bash
(BEANCOUNT) bean-report laowang.bean balances
Assets:Bank:Saving:ICBC
Assets:Fixed                   1000.00 CNY
Equity
Expenses:Food                   400.00 CNY
Income:Sales                  -1000.00 CNY
Liabilities:CreditCard:ICBC    -400.00 CNY
```

通过上面生成的报表，老王对自己的账务状况一目了然：有价值 1,000 元的固定资产，在食材上花了 400 元，一共收入了 1,000 元，信用卡欠款 400 元。bean-report 没有报错，说明账是平的（总和为0）。bean-report 命令还能生成很多报表，具体可以使用 `bean-report -h` 查看帮助。

当然，还可以用 fava 来启动一个 Web UI 服务，执行 `fava laowang.bean` 命令，然后在浏览器中打开 http://localhost:5000/ 即可。由于老王的数据还比较单薄，这里帖两张样例账本的示例图

{% asset_img fava_balance_sheet_sample.png 资产负债表 %}
{% asset_img fava_income_statement_sample.png 损益表 %}

## 二、进阶

这里将另举几个例子，展现下 Beancount 和「复式记账法」能处理多么复杂的交易。这些复杂的交易用单式记账法来记录是极其困难且容易出错的，但是在复式记账中却是如此的自然且流畅。在上一篇里说过，复式记账的「复」是指一笔交易会涉及到复数个账户。上面老王的例子都是两个账户间「一对一」交易。但实际上，生活中会遇到各种「一对多」或「多对多」的交易场景：购买大件物品时因银行支付限额而使用多张银行卡合并付款；和朋友出去聚餐，事后 AA 平摊等。以下几个例子是有意构造的涉及到两个以上账户的交易，让我们一起来看看小王的账本。

### 2.1 第一个例子

```beancount
2023-01-10 * "Some Company" "💰工资2022-12"
  Income:SomeCompany:Salary           -20,000.00 CNY ;应发工资
  Income:SomeCompany:Benefits            -500.00 CNY ;节日福利
  Income:SomeCompany:HousingFund       -2,000.00 CNY ;住房公积金单位扣除
  Assets:Government:HousingFund         4,000.00 CNY ;住房公积金缴纳
  Expenses:Government:SocialSecurity    1,500.00 CNY ;社保缴纳
  Expenses:Government:IncomeTax         3,000.00 CNY ;个税缴纳
  Assets:Bank:Saving:ICBC              14,000.00 CNY ;实发工资
```

这个例子展现了如何在 Beancount 里体现工资条上的内容。每个月工资条上总是会有各种各样的条目。小王在使用了 Beancount 之后，可以方便地把工资、福利、住房公积金、社保、个税等信息都记录进去，以后能很方便地统计每个月有多少工资是喂狗的。

本例中，Income:* 账户有三条记录，Expenses:* 账户有两条记录，Assets:* 账户有两条记录，共7条记录，总和为0。

### 2.2 第二个例子

```beancount
2023-01-11 * "XX 购物中心" "购物"
  Liabilities:CreditCard:ICBC     -1,000.00 CNY ;信用卡刷卡
  Expenses:Entertainment:Game        400.00 CNY ;游戏卡
  Assets:Receivables:Xiao-Ming       600.00 CNY ;帮小明付钱，记入应收账户
```
小王拿到工资的第二天就和小明去购物中心逛游戏店，买了一张新出的游戏卡，花费 400 元，小明没带卡，身上现金不够，于是让小王帮他垫付，小王使用信用卡消费 1,000 元。

本例子中小王的 Liabilities:* 账户减少了 1,000 元，Expenses:* 账户增加 400 元，Assets:* 账户增加 600 元。帮小明付的钱，算是小明欠小王的钱，所以算做资产。所有数字加起来和为0。

### 2.3 第三个例子

```beancount
2023-01-13 * "xx 饭店" "和小明吃饭"
  Assets:Cash:Wallet            -300.00 CNY ;钱包现金
  Assets:Receivables:Xiao-Ming  -200.00 CNY ;小明帮我付的现金
  Expenses:Food:DiningOut        250.00 CNY ;AA我的一半
  Assets:Receivables:Xiao-Ming   250.00 CNY ;AA小明的一半
```
过了几天小王和小明去一家饭店吃饭。两人一共消费了 500 元，还只能付现金。小王和小明各自掏空了钱包，凑齐了 500 元现金，其中小王付了 300 元，小明付了 200 元。这顿饭两人 AA。

### 2.4 第四个例子

```beancount
2023-01-17 * "在免税店买东西"
 Assets:Cash                                 -200.00 USD
 Liabilities:CreditCard:ICBC                 -650.00 CNY @@ 100.00 USD
 Expenses:Clothing:Pants                     +150.00 USD
 Expenses:Clothing:Shoes                     +150.00 USD
```
小王去国外出差了，回国前为了把兑换的美元现金花掉，忍不住又在免税店大肆购物，结果现金不够，于是 300 美元的商品用现金支付了 200 美元，用信用卡支付了 100 美元。小红的信用卡开通了外币消费人民币入账功能，刷美元也出人民币账单。

本例中，涉及到了合并付款和货币转换。小明的信用卡被扣掉了 650 人民币，这其实是由 100 美元转换而来。在 Beancount 中使用 @@ 即可连接两种互相转换的 commodity。在本次交易中，负数共 -200.00 USD + (-100.00 USD) = -300.00 USD，正数共 +150.00 USD + (+150.00 USD) = +300.00 USD，正负相加依然得到的是 0。

### 2.5 使用 bean-query 进行复杂查询

fava 提供的 Web UI 已经能展现很多有用的财务报表，满足大部分用户的需求，如果用户需要进行一些更复杂的数据统计，比如「我经常去哪里加油」，则可以使用 bean-query 工具用 SQL 语句进行查询，详见 Beancount 作者的文档：[Beancount – Query Language](https://beancount.github.io/docs/beancount_query_language.html)。这是一个用来统计我加油次数的例子：

{% asset_img bean_query_sample.png 加油次数 %}

## 三、最佳实践

笔者从2022年9月开始使用 Beancount 来管理个人财务，在使用过程中也总结了一些最佳实践。以下内容说说我个人是怎么使用 Beancount 的，希望对你有所帮助

### 3.1 编辑器支持

笔者目前是使用 `VSCode` 做为账本文件编辑器，加上 [Lencerf/vscode-beancount](https://github.com/Lencerf/vscode-beancount) 插件的配合，会自动为 *.bean 或者 *.beancount 文件加上语法高亮以及账户名自动补全，还可以实现金额数据自动对齐，具体配置方法请自己 Google 或者参考 Github 上作者提供的帮助文档。

### 3.2 账户开户日期的选择

Beancount 中账户的开户日期必须在该账户第一笔交易之前，小王为了省事将所有账户的开户日期设置为 1970-01-01 这个日期，其实可以有一些更有意义的选择：

- **Expenses** 账户可以使用自己的出生日期作为开户日期；
- **Income** 账户可以按来源分类，如 `Income:SomeCompany:Salary`、`Income:OtherCompany:Salary` 等，然后以入职公司的时间作为开户日期；
- **Assets** 和 **Liabilities** 账户中的借/贷记卡，可以以在银行的开户日期作为 Beancount 账户的开户日期。

不要害怕开账户。即使是一些短时间用的小账户（比如只用两个月的储值卡），也可以开账户，因为账户是可以关闭的。关闭后的账户不会出现在关闭后的报表里，所以并不会触发你的强迫症。

### 3.3 账本文件的分割

随着时间的积累，账本文件会越来越大，编辑起来不太方便。Beancount 有 include 语句，可以在一个账本文件里包含另一个账本文件。我的主账本文件里只有一些 option 条目，其他都是 include，各种打开/关闭账户的的条目放单独的文件里，然后每个月的账本是一个单独的文件，也 include 进来。

Beancount 会把所有交易都读到内存里后按日期重新排序，所以每条交易在文件里出现的顺序并不重要。如下是我的账本文件分割结构，或者可以给你提供参考：

```bash
.
├── 2022
│   ├── 09.bean
│   ├── 10.bean
│   ├── 11.bean
│   ├── 12.bean
│   ├── __index.bean
│   ├── creditcard.bean
│   ├── event.bean
│   ├── forecast.bean
│   ├── income.bean
│   ├── loan.bean
│   └── transfer.bean
├── 2023
│   ├── 01.bean
│   ├── __index.bean
│   ├── forecast.bean
│   ├── income.bean
│   ├── insurance.bean
│   ├── loan.bean
│   └── transfer.bean
├── account
│   ├── __index.bean
│   ├── assets.bean
│   ├── equity.bean
│   ├── expenses.bean
│   ├── income.bean
│   └── liabilities.bean
├── commodity
│   ├── __index.bean
│   └── fund.bean
├── doc
│   └── __index.bean
└── main.bean
```

### 3.4 定期断言

一个维护良好的账本应当定期做断言（assertion），即定期标记某个账户在某个日期的余额（balance），断言的示例如下：

```beancount
2023-01-01 balance Assets:Cash 500.00 CNY
2023-01-01 balance Assets:Cash 100.00 USD
2023-01-01 balance Liabilities:CreditCard:CMB1234 1,000.00 CNY
```

断言语句告诉 Beancount，这个账户在指定日期凌晨 00:00:00 这个时间点，余额为这个数字，上面的示例代表在笔者的账本里，截止2022年12月底，笔者有 500 人民币、100 美元的现金，同时招行尾号 1234 的信用卡里有 1,000 人民币欠款。

添加了断言之后，Beancount 便会检查那个账户的数字是否与断言的数字相等，如果不相等就会报错。人总是会犯错的，当你因为各种原因在账目上出现了错误，断言能帮助你缩小查错范围——你只需要检查最后一次成功的断言之后的发生的交易即可。

### 3.5 合理填充

Beancount 另一个有趣的功能是填充（padding），填充是配合断言一起用的，当 Beancount 解析到填充语句时，会自动在这条语句和下一条断言语句之间插入一条填充交易，使得断言成功。在填充语句所在日期和断言语句所在日期之间不能再有其他交易。请看示例：

```beancount
2022-12-31 pad Assets:Bank:Saving:CMB1234 Income:Trade:PnL
2023-01-01 balance Assets:Bank:Saving:CMB1234 12,043.00 CNY
```

笔者在2022年12月底做账目核对的时候，发现招商银行借记卡里的余额是 12,043 元，但是根据 11 月底的余额，以及 12 月的交易记录，卡里的余额应该没有 12,043 元这么多，仔细看了下，网银里的账单记录，发现之前买的一个活期理财产品每天都有些收益，因此这里使用填充（pad）功能来解决这个问题，在 12 月最后一笔交易和 2023-01-01 的断言之间插入一条 `pad` 语句，这样 Beancount 会自动插入一条交易，使 `Assets:Bank:Saving:CMB1234` 里的余额调整为 12,043 元，而因此产生的差额，则记录到 `Income:Trade:PnL` 账户里。

填充功能比手工写一笔交易有什么好呢？你不需要去计算两个数字之间的差额了—— Beancount 会自动算出差额并帮你插入交易。此外，这个差额是动态计算的。