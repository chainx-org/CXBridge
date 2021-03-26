---
sidebar: auto
---
# CXBridge文档

## 概述

ChainX是基于Substrate架构，实现跨链交易的项目，而CXBridge则是ChainX中的核心模块之一。CXBridge用于实现BTC网络和ChainX的跨链交易，实现比特币网络和ChainX之间的资产价值转移。CXBridge**通过分布式资产保险库的形式进行资产管理，并通过抵押品的方式进行风险控制。**这有别于ChainX1.0中信托多签模式的去中心化托管方案。

![](D:\Typora文档\区块链\ChainX\图\概念图.png)

### Question

**CXBridge究竟是什么？**

确切来说，CXBridge本质上就是ChainX的一种智能合约，被部署在ChainX的所有节点上，用于处理与BTC网络跨链交易的相关请求。

## 关键概念

### CXBridge参与角色

本部分将介绍CXBridge中的参与角色。CXBridge的参与者包括：

* 普通用户（User）
* 资产保险库（Vault）
* 监察者（Watcher）

#### 普通用户（User）

普通用户，即ChainX区块链网络中的用户，用户可提交充值和提现请求，并且可以通过抵押PCX的方式，申请成为资产保险库，这是因为资产保险库需要抵押品为其信用背书，以做清算（清算的具体内容可见核心功能部分），防止资产保险库无法进行BTC转账交易，或存在其他风险，所以一般情况下，**资产保险库的抵押资产价值需要高于账户名下的BTC资产价值。**

#### 资产保险库（Vault）

资产保险库是BTC网络和ChainX网络的中转站，是ChainX网络中的特殊用户，其根本的作用是通过抵押资产的方式，进行信用背书，以实现BTC从BTC网络到ChainX网络的价值转移，将BTC转换为XBTC。ChainX的任意用户可以通过锁定PCX作为抵押品，成为资产保险库。**资产保险库与XBTC之间是彼此隔离的，即通过某个资产保险库产生的XBTC不需要通过同一保险库进行提现。**资产保险库的收益主要来源于虚拟挖矿。通过该资产保险库所发行的代币所得的挖矿利润，会按一定的比例分给该资产保险库，例如某用户进行充值操作，资产保险库发行XBTC，该XBTC发行交易被打包上链之后所得的挖矿利润，将按一定比例支付给资产保险库。

#### 监察者（Watcher）

监察者负责监督整个ChainX网络中的资产保险库，每个资产保险库所关联的地址是受到监管的，当发生异常交易或资产保险库的风险评估结果较差时，CXBridge会强制清算该资产保险库。

### CXBridge核心功能

本部分将介绍CXBridge负责的核心功能。CXBridge的核心功能包括：

* 充值（Recharge）
* 提现（Redeem）
* 抵押（Collateral）
* 清算（Liquidation）

#### 充值（Recharge）

用户进行充值操作，即将比特币网络中的BTC转化为ChainX中的XBTC，这一过程将发行XBTC。在这个过程中，用户将在BTC网络中向选定的一个资产保险库转账，资产保险库将给该用户对应数量的XBTC。其完整流程如下：

![充值流程](D:\Typora文档\区块链\ChainX\图\充值流程.png)

* 当User提交一个Recharge请求时，请求中包括User要求充值的XBTC数量，以及少量的用于抵押的PCX。提交请求后首先需要确定一个Vault，并且要保证该Vault有足够的XBTC发行能力。
* 选择好Vault后，User将获取到Vault的BTC网络地址，接着User将向BTC网络提交转账交易，即从User绑定的BTC账户向Vault账户中进行转账。
* 转账完成后，ChainX将获取到BTC网络传回的转账交易证明，进行验证。
* BTC转账交易验证完成后，将发行XBTC给User。

**需要注意的是，在User请求Recharge时，需要保证Vault已按照Vault注册表锁定了抵押品，并且其XBTC有足够的能力发行用户请求的XBTC。另外，如果User没有在预定时间内完成Recharge操作，系统将取消此次请求，并且User在发起Recharge请求时抵押的PCX将归Vault所有，这是为了防止恶意的Recharge请求。**

#### 提现（Redeem）

用户进行提现操作，即将ChinaX中的XBTC价值转移到BTC网络中，这一过程用户将销毁要提现的XBTC，而Vault将在BTC网络中向User的BTC网络账户进行转账。其完整流程如下：

![提现流程](D:\Typora文档\区块链\ChainX\图\提现流程.png)

* 用户在提交Redeem请求时，将根据请求的提现数额，锁定自己账户中的等量XBTC。
* 在锁定好用于提现资产后，用户需要确定一个Vault，用于处理此次的提现交易请求，该Vault的BTC账户需要有足够的BTC资产进行转账操作。
* 选定好Vault后，Vault将从用户的请求中获取到用户的BTC地址，然后Vault将向BTC网络提交转账交易，即从Vault的BTC账户向User的BTC账户进行转账。
* 转账完成后，BTC网络将转账证明返回给ChainX网络，并对证明进行验证。
* 验证完成后，User将销毁锁定的用于提现的XBTC。

**需要注意的是，如果在给定的时间内，Vault未能完成Redeem操作，用户将向Vault收取等价值量的PCX以弥补在其BTC网络中的损失，或者用户也可以选择取消该次Redeem操作，再重新选择一个资产保险库执行Redeem操作。**

#### 抵押（Collateral）

在CXBridge中，抵押用以保证User、Vault可信任，其抵押的基本单位为PCX。在以下情况下，会出现抵押操作：

* User注册成为Vault
* Vault需要增加抵押品
* 用户发起Recharge请求时

User注册成为Vault时，User需要抵押PCX以进行信用背书，抵押的PCX数量取决于User的需求，但有最低限制，这是为了防止过多的注册Vault请求，减轻系统压力，并且User抵押的PCX价值也决定了成为Vault后，其应对Recharge和Redeem请求的能力。

如果某个Vault希望提升自己应对Recharge和Redeem请求的能力，那么它将需要增加自己的抵押品价值。

当User发起Recharge请求时，User需要抵押一定量的PCX，这是为了确定User确实存在Recharge的需求，而非恶意请求，当User发起的Recharge未能完成时，User抵押的PCX将被转入进行Recharge处理的Vault账户下，以弥补Vault在这段时间内的损失。

#### 清算（Liquidation）

清算是Watcher对Vault监督的一种结果，若某Vault发生异常交易时，或者当一个Vault的风险评估结果较差时，CXBridge将对该Vault进行清算。其中Vault的风险评估的方式为
$$
\alpha = \frac{抵押的PCX价值}{获取的BTC价值}
$$
α称为风险系数，当风险系数小于某一阈值时，我们认为该Vault的风险较高，会对该Vault进行清算。清算的方式是把该Vault所有抵押的PCX转给系统Vault，并且取消该Vault的资格。

### Question

**对于未在BTC网络中拥有账户的用户，是否可以进行Recharge、Redeem操作？**

没有BTC账户的ChainX用户是可以进行Recharge而不可Redeem的。当一个没有BTC账户的ChainX用户进行Recharge操作时，由于没有用户BTC地址，Recharge将失败，而Recharge需要用户抵押一定的PCX，失败后，这些PCX将被转入处理的Vault账户。而Redeem操作本身就需要用户锁定XBTC，而无法进行Recharge的用户是没有XBTC的，所以无法进行Redeem操作。

**如何将ChainX网络中用户的账户与BTC网络中的账户进行绑定？**



**User如何选择Vault，如何确定Vault已经锁定了抵押品？**



**Vault如何获得BTC网络中的交易证明？**



## API介绍

### 充值

**简单描述**

User申请Recharge请求以及处理Recharge请求的函数。

**接口描述**

* request_issue

  * 参数列表

    | 参数名              | 参数类型 | 参数描述           |
    | ------------------- | -------- | ------------------ |
    | origin              |          | 申请Recharge的User |
    | vault_id            |          | User选定的Vault    |
    | btc_amount          |          | Recharge的BTC数量  |
    | griefing_collateral |          | 抵押品（PCX）数量  |

  * 程序流程

    ![充值程序流程](D:\Typora文档\区块链\ChainX\图\充值程序流程.png)

* execute_issue

  * 参数列表

    | 参数名        | 参数类型 | 参数描述            |
    | ------------- | -------- | ------------------- |
    | origin        |          | 发起Recharge的User  |
    | request_id    |          | 请求ID              |
    | _tx_id        |          | BTC网络中的交易ID   |
    | _merkle_proof |          | BTC交易的merkle证明 |
    | _raw_tx       |          | 原Recharge交易      |

  * 程序流程

    ![执行充值程序流程](D:\Typora文档\区块链\ChainX\图\执行充值程序流程.png)

* cancel_issue

  * 参数列表

    | 参数名        | 参数类型 | 参数描述            |
    | ------------- | -------- | ------------------- |
    | origin        |          | 发起Recharge的User  |
    | request_id    |          | 请求ID              |

  * 程序流程

    ![取消充值程序流程](D:\Typora文档\区块链\ChainX\图\取消充值程序流程.png)

### 提现

**简单描述**

User申请Redeem请求以及处理Redeem请求的接口。

**接口描述**

* request_redeem

  * 参数列表

    | 参数名        | 参数类型 | 参数描述         |
    | ------------- | -------- | ---------------- |
    | origin        |          | 发起Redeem的User |
    | vault_id      |          |                  |
    | redeem_amount |          |                  |
    | btc_address   |          |                  |

  * 程序流程

    

* execute_redeem

  * 参数列表

    

  * 程序流程

    

* cancel_redeem

  * 参数列表

    

  * 程序流程

    

### 抵押

**简单描述**



**方法描述**



### 清算

**简单描述**



**方法描述**



### 注册

**简单描述**



**方法描述**



### 资产管理

**简单描述**



**方法描述**

