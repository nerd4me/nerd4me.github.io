---
title: 写给程序员的记账指南（上篇）
tags:
  - Bookkeeping
  - Beancount
hide: true
abbrlink: fef7416f
date: 2022-10-22 12:30:00
---


## 复式记账 vs 图论

### 账户 = 节点，交易 = 边

假设你在便利店，使用信用卡消费5元买了一盒饼干。然后又去逛了逛IKEA商场，花费500元买了一张单人沙发，使用储蓄卡支付。一共两笔交易，每一笔交易代表图中的一条边，每条边标记上具体的交易金额。

到这里，你可能会有一个疑问：在一个图里边，一条边总是连接着两个节点，那么这些节点是什么？好吧，这里你可以随意的定义它们（尽管会有一些约束）。暂时如下：

{% mermaid %}
graph LR;
A([储蓄卡]):::black -->|"¥500"| B([IKEA]):::black 
C([信用卡]):::black -->|"¥5"| D([饼干]):::black
classDef black fill:#FFF,stroke:#000;
{% endmermaid %}

让我们加入更多的细节。你用储蓄账户还了5元的信用卡账单。想一想我们储蓄账户里的钱最初是从哪里来的？啊，我明白了，里边的5000元是从你以往的积蓄里来的，现在我们的图如下所示：

{% mermaid %}
graph LR;
H([权益]):::black -->|"¥5,000"| A
A([储蓄卡]):::black -->|"¥500"| B([IKEA]):::black 
C([信用卡]):::black -->|"¥5"| D([饼干]):::black
A -->|"¥5"| C
classDef black fill:#FFF,stroke:#000;
{% endmermaid %}

{% note primary %}
等下，上图中引入了一个叫**权益**的账户，那它是怎么来的？仔细想想，我们不是一个刚出生的婴儿，开始记账之前的收入、负债、资产、支出怎么算呢？答案就是放到**权益**里。当我们决定开始使用复式记账法的时候，我们从「权益」账户里拿一些钱放到其他账户里（或者从其他账户里拿一些钱放到权益账户里），将其他账户的余额调节成符合当前的实际情况。如上面的例子里，当我们开始记账时，储蓄账户里有5000元的存款。
{% endnote %}

从上面的图中，很容易看出，我们的资金沿着箭头的方向在各个账户之间流动。

逛着逛着，又饿了，你去金拱门花8元买了一个鸡肉卷，使用信用卡付款。现在我们应该为**鸡肉卷**创建另外一个节点，事情到这里变得不可控起来 -- 我们并不真正关心自己在**饼干**上花了多少钱，在**鸡肉卷**上又花了多少钱。这里我们统一把两者归类为**食物**。此外，**IKEA**这名称对我的记账没有什么益处，我已经快忘了那500元是做什么用的。我们就叫它**家具**好了。

{% mermaid %}
graph LR;
H([权益]):::black -->|"¥5,000"| A
A([储蓄卡]):::black -->|"¥500"| B([家具]):::black 
C([信用卡]):::black -->|"¥5"| D([食物]):::black
C -->|"¥8"| D
A -->|"¥5"| C
classDef black fill:#FFF,stroke:#000;
{% endmermaid %}

到目前为止，完全没有问题，我们有代表实际银行账户或信用卡的节点，其他代表「食物」或者「家具」等抽象类别的节点。

### 账户余额

在图中的每个节点在会计学中叫做账户/科目，每个账户都有余额。余额就是一个数字，它完全由进出账户的交易决定：

1. 在最开始时，每个节点的值都为0.
2. 对于每个节点，加上所有指向该节点的边上标记金额、再减去所有指向其他节点的金额。

经过简单的计算之后，每个节点的值代表着当前账户的余额。

{% mermaid %}
graph LR;
H(["权益<br>¥-5,000"]):::black -->|"¥5,000"| A
A(["储蓄卡<br>¥4,495"]):::black -->|"¥500"| B(["家具<br>¥500"]):::black 
C(["信用卡<br>¥-8"]):::black -->|"¥13"| D(["食物<br>¥13"]):::black
A -->|"¥5"| C
classDef black fill:#FFF,stroke:#000;
{% endmermaid %}

注意，这里的账户余额有两个非常棒的特性：

1. 因为每笔交易都会作用于复数个账户，其中某些账户的余额增加，余下的账户余额减少，且金额相等，所以所有账户的余额加起来，永远是0。
2. 将图中的节点集合分成任意两个不相交的子集，把两个子集中的节点余额各自加起来，计算出来两个值互为相反数（毕竟所有节点的和为0）。

这些特性对于检查你的账目正确性非常有用，如果它们被违反了，说明「你出错了！！！」

### 收入

通过上面「家具」和「食物」的例子，相信你已经对整个记账的流程有了比较好了解，但做为一个完整的例子，怎么能少得了收入这一环节。

假设你供职于「A公司」，给你的月薪是10,000元/月，同时公司接了一个大项目，你有幸参与其中，完成之后，公司承诺给项目组核心成员10,000元/人的奖金，目前项目接近尾声，公司打算同当月薪水一起，先给发一半的项目奖金。

所以你一共收到 10,000元 + 5,000元，直接转到你的银行储蓄卡，当然，这里实际情况还会涉及到个税、社保等其他项目，这里为了示例简单明晰，直接省去。

{% mermaid %}
graph LR;
E(["薪资<br>¥-10,000"]):::black -->|"¥10,000"| A
F(["项目奖金<br>¥-5,000"]):::black -->|"¥5,000"| A
H(["权益<br>¥-5,000"]):::black -->|"¥5,000"| A
A(["储蓄卡<br>¥19,495"]):::black -->|"¥500"| B(["家具<br>¥500"]):::black 
C(["信用卡<br>¥-8"]):::black -->|"¥13"| D(["食物<br>¥13"]):::black
A -->|"¥5"| C
classDef black fill:#FFF,stroke:#000;
{% endmermaid %}

但这里有点不对，公司承诺的项目奖金是10,000元，但实际上，我的银行账户里只收到5,000元，怎么在图中把剩下的5,000元给体现出来呢？

这里的解决方案是将项目奖金的发放拆成两笔交易，5,000元直接转入储蓄卡，另外5,000元转入一个应收账户，待项目完成，剩下的项目奖金到账时，再将应收账户里的5,000元转到储蓄卡账户里。此时，我们的图应该是长这样：

{% mermaid %}
graph LR;
H(["权益<br>¥-5,000"]):::black -->|"¥5,000"| A
E(["薪资<br>¥-10,000"]):::black -->|"¥10,000"| A
F(["项目奖金<br>¥-10,000"]):::black -->|"¥5,000"| A
F-->|"¥5,000"| G(["奖金应收<br>¥5,000"]):::black
A(["储蓄卡<br>¥19,495"]):::black -->|"¥500"| B(["家具<br>¥500"]):::black 
C(["信用卡<br>¥-8"]):::black -->|"¥13"| D(["食物<br>¥13"]):::black
A -->|"¥5"| C
classDef black fill:#FFF,stroke:#000;
{% endmermaid %}

看到没？我这里只是添加了一个新的节点，名叫「奖金应收」，同时从「项目奖金」里转入5,000元到这个新的节点。这并没有改变我们银行账户的余额。

「薪资」&「项目资金」的余额代表你通过付出时间、体力、脑力而获得的酬劳，但它们的余额都是复数，这看起来有点奇怪，这里其实可以想想挣钱的本质是什么？不就是靠消耗我们的时间、体力还有脑力来换取 `Money` 么，这些账户里的余额正代表我们所消耗的东西，只是转换成金额给体现了出来。

### 完善我们的示例

为了进一步完善它，让我在故事里加入更多的事件（= 在我们的图中加入更多的边）

此时，你的新家正在装修，在建材市场买装修材料花费了16,000元，使用储蓄卡付款

你把上面的图给你从事会计行业的朋友看了，然后他对你讲了很多的专业术语。出于某种奇怪的原因，他坚持认为你针对「单人沙发」的记账方式不对，他觉得你应该分四年去折旧它，也就是说，它在4年时间内，价值逐渐减少至0，虽然不是完全理解，但你还是听从了朋友的意见，以每年125元的价值去折旧你的沙发，此时我们的图变成：

{% mermaid %}
graph LR;
H(["权益<br>¥-5,000"]):::black -->|"¥5,000"| A
E(["薪资<br>¥-10,000"]):::black -->|"¥10,000"| A
F(["项目奖金<br>¥-10,000"]):::black -->|"¥5,000"| A
F-->|"¥5,000"| G(["奖金应收<br>¥5,000"]):::black
A(["储蓄卡<br>¥3,495"]):::black -->|"¥500"| B(["家具<br>¥375"]):::black 
A -->|"¥16,000"| J(["装修<br>¥16,000"]):::black
C(["信用卡<br>¥-8"]):::black -->|"¥13"| D(["食物<br>¥13"]):::black
A -->|"¥5"| C
B -->|"¥125"| I(["折旧<br>¥125"]):::black
classDef black fill:#FFF,stroke:#000;
{% endmermaid %}

### 损益表(The Profit and Loss Statement)

到这里，我们已经完整的完成了图的构建，现在我将向你展示该图如何映射到公司财报中最常用的两个财务报表：「**损益表**」和「**资产负债表**」，这两张表对于我们管理个人/家庭财务也有着非常重要有意义。

为了生成上面说的两张报表，我们需要对图中的各个节点进行着色处理：

{% mermaid %}
graph LR;
H(["权益<br>¥-5,000"]):::pink -->|"¥5,000"| A
E(["薪资<br>¥-10,000"]):::blue -->|"¥10,000"| A
F(["项目奖金<br>¥-10,000"]):::blue -->|"¥5,000"| A
F-->|"¥5,000"| G(["奖金应收<br>¥5,000"]):::green
A(["储蓄卡<br>¥3,495"]):::green -->|"¥500"| B(["家具<br>¥375"]):::green 
A -->|"¥16,000"| J(["装修<br>¥16,000"]):::blue
C(["信用卡<br>¥-8"]):::green -->|"¥13"| D(["食物<br>¥13"]):::blue
A -->|"¥5"| C
B -->|"¥125"| I(["折旧<br>¥125"]):::blue

classDef blue fill:#8AD0FC,stroke:#000;
classDef pink fill:#FCB6C0,stroke:#000;
classDef green fill:#A2FE97,stroke:#000;
{% endmermaid %}

各种颜色所代表的含义（我会将对应的会计术语放到括号中，因为这些你在今后的记账中可能会遇到）：

- <span style="color:#A2FE97">**绿色**</span> 代表 **你所拥有的东西**（资产-Assets），例如：银行存款、现金，或者你已经买了且将来有可能折旧卖出的东西，比如说图中的「家具」。同时还代表公司/个人 **欠你的钱**（债务人-Debtors，比如说公司欠你的项目奖金），and **你欠的钱**（负债-Liabilities，比如说你的信用卡账单）。
- <span style="color:#8AD0FC">**蓝色**</span> 代表 **你的工资/奖金**（收入-Income）和 **你花出去的**且永远不会再回来的钱（花费-Expenses）。「个人沙发」是绿色，因为你可能再将它卖出，但「食物」是蓝色，因为你一旦买了（吃了），它将永远不会再回来
- <span style="color:#FCB6C0">**粉色**</span> 代表 **你开始记账前所拥有的**（权益-Equity）。（如果你有银行贷款，它不是粉色，而是绿色，因为你欠银行的钱是需要还的）

通过这些颜色表示，「**损益表**」就是这些 **蓝色节点的集合**，「**净利润-P&L**」则是通过将这些蓝色节点的余额相加得到。负数代表「**盈利-Profit**」，正数代表「**亏损-Loss**」，这样看起来很让人迷惑，所以通常我们需要翻转下对应的符号

转换成标准的格式，我们的「**损益表**」应该是长这样：

{% raw %}
<table style="margin: 1.5em 0">
  <tbody><tr>
    <th rowspan="3" style="font-variant: small-caps;">INCOME</th>
    <td style="border-bottom: 1px solid #888;">薪资</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">¥10,000</td>
  </tr>
  <tr>
    <td style="border-bottom: 1px solid #888;">项目奖金</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">¥10,000</td>
  </tr>
  <tr>
    <th style="border-bottom: 1px solid #888;">Total income</th>
    <th style="border-bottom: 1px solid #888; text-align: right;">¥20,000</th>
  </tr>
  <tr style="height: 0.7em"></tr>
  <tr style="margin-top: 1em">
    <th rowspan="4" style="font-variant: small-caps;">Expenses</th>
    <td>折旧</td>
    <td style="text-align: right;">¥125</td>
  </tr>
  <tr>
    <td style="border-bottom: 1px solid #888;">装修</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">¥16,000</td>
  </tr>
  <tr>
    <td style="border-bottom: 1px solid #888;">食物</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">¥13</td>
  </tr>
  <tr>
    <th style="border-bottom: 1px solid #888;">Total expenses</th>
    <th style="border-bottom: 1px solid #888; text-align: right;">¥16,138</th>
  </tr>
  <tr style="height: 0.7em"></tr>
  <tr style="margin-top: 1em">
    <th style="padding-top: 1em; font-variant: small-caps;">Total</th>
    <th style="border-bottom: 1px solid #888; font-weight: bold;">Profit/Loss</th>
    <th style="border-bottom: 1px solid #888; font-weight: bold; text-align: right;">¥3,862</th>
    <td>(= total income - total expenses)</td>
  </tr>
</tbody></table> 
{% endraw %}

上面的表看起来很直观。你靠自己的辛勤劳动获得了20,000元的收入，花费了16,138元，所以你的净利润是3,862元。

「**损益表**」通常是在一段时间内会计算一次（周期通常是以一个月、一个季度或者一年），如果要计算一段时间内的损益表，你需要将发生在这段时间内的交易给筛选出来，余额也仅是这段时间内的交易的累加

还有一个你需要注意的是：盈利并不能说明你的银行账户就增加了这么多余额。银行账户是一个绿色节点，但我们这里只看蓝色节点。在这个例子中，尽管你最初银行账户的余额是5,000元，但最终却是减少了，只有3,496元，因为你还有5,000的项目奖金没有真实到账。这也就是为什么有些公司的财报上看到是盈利的，但公司最终还是因资金不足而运行不下去。

### 资产负债表(The Balance Sheet)

资产负债表虽然不如损益表直观，但它是一个非常强大的工具。你可以一目了然地看到自己有多少资产、然后这些资产分别在哪些账户里、有多少负债、是对哪些银行/机构的负债。

还记得我们之前说过的将图中节点集合分成任意两个不相交的集合，将其各自的余额相加之后的值为0？这正是资产负债表的由来。我们取图中的所有节点，对其进行分类，绿色节点在一类，另一类是蓝色和粉色节点。所以蓝色和紫色节点的余额之和与所有绿色节点的余额之和加起来为0.

为了方便查看，我们将蓝色和粉色节点的余额做了求绝对值处理，这样在我们的资产负债表里，上下的两个值相等的：

{% raw %}
<table style="margin: 1.5em 0">
  <tbody><tr>
    <th rowspan="4" style="font-variant: small-caps;">Assets</th>
    <td>储蓄卡</td>
    <td style="text-align: right;">¥3,495</td>
  </tr>
  <tr>
    <td>奖金应收</td>
    <td style="text-align: right;">¥5,000</td>
  </tr>
  <tr>
    <td style="border-bottom: 1px solid #888;">家具</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">¥375</td>
  </tr>
  <tr>
    <th style="border-bottom: 1px solid #888;">Total assets</th>
    <th style="border-bottom: 1px solid #888; text-align: right;">¥8,870</th>
  </tr>
  <tr style="height: 0.7em"></tr>
  <tr style="margin-top: 1em">
    <th rowspan="2" style="font-variant: small-caps;">Liabilities</th>
    <td style="border-bottom: 1px solid #888;">信用卡</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">¥8</td>
  </tr>
  <tr>
    <th style="border-bottom: 1px solid #888;">Total liabilities</th>
    <th style="border-bottom: 1px solid #888; text-align: right;">¥8</th>
  </tr>
  <tr style="height: 0.7em"></tr>
  <tr>
    <th colspan="2" style="border-bottom: 1px solid #888; font-weight: bold;">Total assets - total liabilities</th>
    <th style="border-bottom: 1px solid #888; font-weight: bold; text-align: right;">¥8,862</th>
  </tr>
  <tr style="height: 0.7em"></tr>
  <tr style="margin-top: 1em">
    <th rowspan="3" style="padding-top: 1em; font-variant: small-caps;">Equity</th>
    <td>Profit/Loss</td>
    <td style="text-align: right;">¥3,862</td>
  </tr>
  <tr>
    <td style="border-bottom: 1px solid #888;">Equity</td>
    <td style="border-bottom: 1px solid #888; text-align: right;">¥5,000</td>
  </tr>
  <tr>
    <th style="border-bottom: 1px solid #888; font-weight: bold;">Total equity</th>
    <th style="border-bottom: 1px solid #888; font-weight: bold; text-align: right;">¥8,862</th>
  </tr>
</tbody></table>
{% endraw %}

上面的部分(Assets and Liabilities)代表图中的绿色节点，下面部分包含了粉色节点为和所有蓝色节点之和。我们已经在上面的损益表里展示了所有蓝色节点的余额，在资产负债表里，我们只需要将其计算出来的值填到「Profit/Loss」这一项即可。


## 附录
[^1]: Martin Kleppmann 是剑桥大学「分布式」系统研究人员，著有 [«Designing Data-Intensive Applications»](https://book.douban.com/subject/26197294/)一书
[^2]: [Accounting for Computer Scientists](https://martin.kleppmann.com/2011/03/07/accounting-for-computer-scientists.html)，这篇文章将复式记账与计算机图论结合起来，一路从单笔记账写到账务报表，本篇文章大部分内容都是借鉴了此文的理论