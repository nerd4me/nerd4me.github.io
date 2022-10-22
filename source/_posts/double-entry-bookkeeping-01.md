---
title: 程序员复式记账指南（上）- Why?
tags:
  - Beancount
  - Bookkeeping
abbrlink: 7a37cdfc
date: 2022-10-22 17:52:12
---

本系列文章将分为上下两篇分别介绍 **复式记账法[^1]** 的基本概念以及如何使用 **Beancount[^2]** 记账。

## 一、为什么要记账

关于「为什么要记账？」，大多数人的回答无非如下几点：

1. 掌控自己的收支情况，以便更好的规划自己的理财计划；
2. 单纯的作为一种记录，帮助自己保存记忆；
3. 希望通过记账来改变自己的消费习惯；
4. 觉得自己穷，希望通过记账来寻找可削减的开支；

然而，在我看来，记账的功效远不止于上面所说的。记账能让我们更清晰宏观的了解自身的财务状况，并通过合理的财富资源配置（用大白话讲就是：让钱去该去的地方），帮助我们更好的应对风险，规划自己的投资行为。

一个有着良好维护的账本能够生成很多有用的账务报表，其中最有用的两个：

- **损益表（Income Statement）**: 通过周期性的审阅这个报表，我们能够清楚的知道这段时间内资金的流向（用大白话讲就是：钱从哪里来？钱到哪里去了？）,以及我们的盈利/亏损（Profit and Loss）情况（也就是收入减去支出）。
- **资产负债表（Balance Sheet）**：体现我们有多少钱，并且这些钱都分布在哪里。

为了拥有一个维护良好的账本，我们需要一种更科学的记账方式。

## 二、复式记账法 vs 图论[^3]

复式记账法保证了每一条帐目，都会有至少两个账户和至少一条交易，而图这种数据结构是节点和边的集合，如果我们把节点当成账户，把交易当成一条有向边，有向边的方向表示资金的流动方向。这就是 Martin Kleppmann[^4] 大神在他的 [Accounting for Computer Scientists](https://martin.kleppmann.com/2011/03/07/accounting-for-computer-scientists.html) 这篇文章中提出的理论，下面的章节将围绕这个理论来揭开复式记账法的神秘面纱。

### 账户 = 节点，交易 = 有向边

假设老王的老婆给了老王5000元的启动资金（转入银行卡），开了一个煎饼摊，买设备花了1000元（银行转账），然后还买了些做煎饼的原材料，总共花费500元（信用卡付款），最后还使用银行卡还了250元的信用卡账单。

图数据结构中的边永远是从一个节点指向其他的节点。这些节点的名称暂且随意定义（虽然在会计学中，账户的名称需要遵循一定的规则）

{% mermaid %}
graph LR;
A([启动资金]):::black -->|5,000元| B([银行卡]):::black
B -->|1,000元| C([煎饼设备]):::black
B -->|250元| D([信用卡]):::black
D -->|500元| E([食材]):::black
classDef black fill:#FFF,stroke:#000;
{% endmermaid %}

### 账户余额（Balance）

图数据结构中的节点在会计学中叫做账户，每一个账户都有余额，余额完全由进出账户的交易决定

{% mermaid %}
graph LR;
A([启动资金<br>-5,000元]):::black -->|5,000元| B([银行卡<br>3,750元]):::black
B -->|1,000元| C([煎饼设备<br>1,000元]):::black
B -->|250元| D([信用卡<br>-250元]):::black
D -->|500元| E([食材<br>500元]):::black
classDef black fill:#FFF,stroke:#000;
{% endmermaid %}

如上图数据结构中的账户余额有两个特别有用的特性：

1. 因为每笔交易同时出现在两个不同账户，图中所有账户余额之和为0；
2. 将所有节点的集合分成不相交的子集，分别计算两个子集的余额之和互为相反数；

这两个特性对于检查账目的准确性非常有用，如果其中任何一个被违反了，就说明账目有问题。

### 经营煎饼摊

老王的煎饼摊生意很不错，目前靠卖煎饼总共获得了5,000元的收入，而且他发明的改良版本煎饼设备获得了专利，煎饼设备生产厂家打算先生产500台试试水，并以每台设备10元的价格购买他的专利使用权（总共5,000元），厂家先预付了2,500元。

老王把上面的图给他做会计的老婆看了下，她老婆给他讲了一堆的会计专业术语，并坚持认为「煎饼设备」的记账方式不对，她觉得应该分4年去折旧这些设备，因为在4年之内，老王完全有可能再把它们给转让出去，而且「启动资金」那里应该改成「实收资本」。

老王的朋友觉得煎饼摊的生意不错，注资了25,000元，他终于可以给自己发工资了。

{% mermaid %}
graph LR;
F([主营业务收入<br>-5,000元]):::black -->|5,000元| B
G([专利收入<br>-5,000元]):::black -->|5,000元| H([工厂<br>2,500元]):::black
H -->|2,500元| B
A([实收资本<br>-30,000元]):::black -->|30,000元| B([银行卡<br>28,250元]):::black
B -->|1,000元| C([煎饼设备<br>750元]):::black
B -->|8,000元| I([佣金支出<br>8,000元]):::black
B -->|250元| D([信用卡<br>-250元]):::black
D -->|500元| E([食材<br>500元]):::black
C -->|250元| J([设备折旧<br>250元]):::black
classDef black fill:#FFF,stroke:#000;
{% endmermaid %}

### 财务报表

至此，我们已经完成了整个图数据结构的构建，现在将向你展示如何通过上面的图数据结构，生成公司财报中最常用的两个报表。

先对图数据结构中的节点进行分类着色处理：

{% mermaid %}
graph LR;
F([主营业务收入<br>-5,000元]):::blue -->|5,000元| B
G([专利收入<br>-5,000元]):::blue -->|5,000元| H([工厂<br>2,500元]):::green
H -->|2,500元| B
A([实收资本<br>-30,000元]):::pink -->|30,000元| B([银行卡<br>28,250元]):::green
B -->|1,000元| C([煎饼设备<br>750元]):::green
B -->|8,000元| I([佣金支出<br>8,000元]):::blue
B -->|250元| D([信用卡<br>-250元]):::green
D -->|500元| E([食材<br>500元]):::blue
C -->|250元| J([设备折旧<br>250元]):::blue

classDef blue fill:#8AD0FC,stroke:#000;
classDef pink fill:#FCB6C0,stroke:#000;
classDef green fill:#A2FE97,stroke:#000;
{% endmermaid %}

各种颜色所代表的含义（我会将对应的会计术语放到括号中，因为这些你在今后的记账中可能会遇到）：

- <span style="color:#A2FE97">**绿色**</span> 代表 **你所拥有的东西**（资产-Assets），例如：银行存款、现金，或者你已经买了且将来有可能折旧卖出的东西，比如说图中的「煎饼设备」。同时还代表公司/个人 **欠你的钱**（债务人-Debtors，工厂欠你的钱），and **你欠的钱**（负债-Liabilities，比如说你的信用卡账单）。
- <span style="color:#8AD0FC">**蓝色**</span> 代表 **销售你的产品/知识产权转让**（收入-Income）和 **你花出去的**且永远不会再回来的钱（花费-Expenses）。「煎饼设备」是绿色，因为你可能再将它卖出，但「食材」是蓝色，因为你一旦买了，它将永远不会再回来
- <span style="color:#FCB6C0">**粉色**</span> 代表 **投资者给的钱**（权益-Equity），通过出售煎饼摊的所有者权益。（如果你有银行贷款，它不是粉色，而是绿色，因为你欠银行的钱是需要还的）

### 损益表（Income Statement）

通过将图数据结构中所有的蓝色节点的余额相加，我们最终得到一个值，这个值是负数代表「盈利(Profit)」，其绝对值代表这段时间内老王煎饼摊的「净利润(Net Profit)」，正数代表「亏损(Loss)」，其值代表这段时间内老王煎饼摊的「净亏损(Net Loss)」

转换成会计学中标准的格式（这里的报表为了易于理解，针对收入余额做了取绝对值处理）：

{% raw %}
<table style="margin: 1.5em 0">
  <tbody><tr>
    <th rowspan="3" style="font-variant: small-caps;">Income</th>
    <td style="border-bottom: 1px solid #888;">主营业务收入</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">5,000元</td>
  </tr>
  <tr>
    <td style="border-bottom: 1px solid #888;">专利收入</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">5,000元</td>
  </tr>
  <tr>
    <th style="border-bottom: 1px solid #888;">Total income</th>
    <th style="border-bottom: 1px solid #888; text-align: right;">10,000元</th>
  </tr>
  <tr style="height: 0.7em"></tr>
  <tr style="margin-top: 1em">
    <th rowspan="4" style="font-variant: small-caps;">Expenses</th>
    <td>佣金</td>
    <td style="text-align: right;">8,000元</td>
  </tr>
  <tr>
    <td>设备折旧</td>
    <td style="text-align: right;">250元</td>
  </tr>
  <tr>
    <td style="border-bottom: 1px solid #888;">食材</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">500元</td>
  </tr>
  <tr>
    <th style="border-bottom: 1px solid #888;">Total expenses</th>
    <th style="border-bottom: 1px solid #888; text-align: right;">8,750元</th>
  </tr>
  <tr style="height: 0.7em"></tr>
  <tr style="margin-top: 1em">
    <th style="padding-top: 1em; font-variant: small-caps;">Total</th>
    <th style="border-bottom: 1px solid #888; font-weight: bold;">Profit/Loss</th>
    <th style="border-bottom: 1px solid #888; font-weight: bold; text-align: right;">1,250元</th>
    <td>(= total income - total expenses)</td>
  </tr>
</tbody></table>
{% endraw %}

上面的表看起来很直观。老王靠销售煎饼和专利转让获得了10,000元的收入，产生的费用8,750元，所以老王煎饼摊的净利润是1,250元。

「**损益表**」通常是在一段时间内会计算一次（周期通常是以一个月、一个季度或者一年），如果要计算一段时间内的损益表，你需要将发生在这段时间内的交易给筛选出来，余额也仅是这段时间内的交易的累加。

{% note warning %}
这里有个点需要注意一下：盈利并不能代表银行账户余额的增加，银行账户是一个绿色节点，但我们这里只统计了蓝色节点。在这个例子中，「实收资本」账户有30,000元流向「银行卡」账户，但「银行卡」账户实际余额只有28,250元，看起来银行账户的钱是减少了，因为「煎饼设备生产厂」还欠我们2,500元没有付。这也就是为什么有些公司从财务报表上看是盈利的，但还是会因为公司账上没钱而进行不下去。
{% endnote %}

### 资产负债表（Balance Sheet）

资产负债表虽然不如损益表直观，但它是一个非常强大的工具。你可以一目了然地看到自己有多少资产、然后这些资产分别在哪些账户里、有多少负债、是对哪些银行/机构的负债。

还记得我们之前说过的将图中节点集合分成任意两个不相交的集合，将其各自的余额相加之后的值为0？这正是资产负债表的由来。我们取图中的所有节点，对其进行分类，绿色节点在一类，另一类是蓝色和粉色节点。所以蓝色和紫色节点的余额之和与所有绿色节点的余额之和加起来为0.

为了方便查看，我们将蓝色和粉色节点的余额做了求绝对值处理，这样在我们的资产负债表里，上下的两个值相等的：

{% raw %}
<table style="margin: 1.5em 0">
  <tbody><tr>
    <th rowspan="4" style="font-variant: small-caps;">Assets</th>
    <td>银行卡</td>
    <td style="text-align: right;">28,250元</td>
  </tr>
  <tr>
    <td>债务人</td>
    <td style="text-align: right;">2,500元</td>
  </tr>
  <tr>
    <td style="border-bottom: 1px solid #888;">煎饼设备</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">750元</td>
  </tr>
  <tr>
    <th style="border-bottom: 1px solid #888;">Total assets</th>
    <th style="border-bottom: 1px solid #888; text-align: right;">31,500元</th>
  </tr>
  <tr style="height: 0.7em"></tr>
  <tr style="margin-top: 1em">
    <th rowspan="2" style="font-variant: small-caps;">Liabilities</th>
    <td style="border-bottom: 1px solid #888;">信用卡</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">250元</td>
  </tr>
  <tr>
    <th style="border-bottom: 1px solid #888;">Total liabilities</th>
    <th style="border-bottom: 1px solid #888; text-align: right;">250元</th>
  </tr>
  <tr style="height: 0.7em"></tr>
  <tr>
    <th colspan="2" style="border-bottom: 1px solid #888; font-weight: bold;">Total assets less total liabilities</th>
    <th style="border-bottom: 1px solid #888; font-weight: bold; text-align: right;">31,250元</th>
  </tr>
  <tr style="height: 0.7em"></tr>
  <tr style="margin-top: 1em">
    <th rowspan="3" style="padding-top: 1em; font-variant: small-caps;">Equity</th>
    <td>Profit/Loss</td>
    <td style="text-align: right;">1,250元</td>
  </tr>
  <tr>
    <td style="border-bottom: 1px solid #888;">实收资本</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">30,000元</td>
  </tr>
  <tr>
    <th style="border-bottom: 1px solid #888; font-weight: bold;">Total equity</th>
    <th style="border-bottom: 1px solid #888; font-weight: bold; text-align: right;">31,250元</th>
  </tr>
</tbody></table>
{% endraw %}

上面的部分(Assets and Liabilities)代表图中的绿色节点，下面部分包含了粉色节点为和所有蓝色节点之和。我们已经在上面的损益表里展示了所有蓝色节点的余额，在资产负债表里，我们只需要将其计算出来的值填到「Profit/Loss」这一项即可。

## 三、Beancount

### Ledger-like 和 Beancount

Beancount 是一个 Ledger-like 软件。Ledger 是这一类复式记账软件的开创者。他们共有的特点是：

- 采用改进的复式簿记方案（使用正负号而不是「借」和「贷」来表示账户之间的变化）；
- 使用纯文本文件作为账本，用户用文本编辑器即可记账；
- 账本既是用户输入的文件，同时也是软件的「数据库」；
- 软件读取账本并生成报表，账本本身也可供人类直接阅读。

市面上的复式簿记软件不少（如 GnuCash），但是大部分都是提供一个 GUI，用户在一堆文本框里输入各种数字和文字，软件接受输入然后存储到自己的数据库里（SQLite、MySQL 等）。用户无法直接看到或操作他们的数据，必须通过软件来操作；一旦软件停止更新，用户的数据就危在旦夕：难以导出，难以复用，很难跨平台或跨设备同步。

而 Ledger-like 软件则直接使用文本文件作为账本，用户直接用最喜爱的编辑器打开账本即可记账。软件只是读取你的账本并生成报表，即使软件停止更新，用户依然可以直接阅读账本。你可以方便地在各在平台上记账，甚至跨设备问题也可以用 Dropbox 等同步工具，或是 Git 等版本管理工具轻松解决。

Beancount 是 Ledger-like 软件中优秀的一员，相比用 C++ 写成的 Ledger，用 Python 写成的 Beancount 更轻便，更方便增加插件和二次开发，也增加了很多功能，如灵活强大的多「货币」支持。这里为加上引号是因为，Beancount 其实并不知道什么是「货币」，它记录的只是「通货」（commodity）的变化，所有的 commodity 皆由用户自己定义，因此 Beancount 可以用来记录包括货币在内任何东西的变化，比如年假天数、股票、航空里程、信用卡积分，当然了，还可以用来数豆子。这也是 Beancount 名字的来源。

### 快速上手

首先需要安装 `Python3` 环境，然后安装对应的包

```bash
pip install beancount fava
```

生成一个官方提供的示例

```bash
bean-example > example.beancount
```

通过 `fava` 命令运行 WebUI

```bash
fava -H 0.0.0.0 example.beancount
```

默认情况下Web UI会运行在 http://localhost:5000

这样就有一个基本环境了。 下一篇会进一步介绍。

## 四、参考

[^1]: 在会计学中，复式记账法（又称为复式簿记 - Bookkeeping）是商业及其他组织上记录金融交易的标准系统。该系统之所以称为复式簿记，是因为每笔交易都至少记录在两个不同的账户当中。每笔交易的结果至少被记录在一个借方和一个贷方的账户，且该笔交易的借贷双方总额相等，即“有借必有贷，借贷必相等”。
[^2]: [Beancount](https://github.com/beancount/beancount) 是一个开源的基于纯文本的复式记账软件。它为我们提供了一系列开箱即用的命令行工具，以及一套简洁实用且美观的 WebUI：[Fava](https://github.com/beancount/fava)
[^3]: 图论是研究事物之间关系的科学，万物之间都是有千丝万缕联系的，任何有联系（联接）的东西都可以抽象成图这种数据结构。抽象成图之后，更好做分析，比如分析哪个节点影响力最大，哪条路径最为关键，预测哪个方向会出现更多节点等等
[^4]: Martin Kleppmann 是剑桥大学「分布式」系统研究人员，著有 [«Designing Data-Intensive Applications»](https://book.douban.com/subject/26197294/)一书