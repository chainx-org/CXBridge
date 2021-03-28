# CXBridge文档

## 概述

ChainX是基于Substrate架构，实现跨链交易的项目，而CXBridge则是ChainX中的核心模块之一。CXBridge用于实现BTC网络和ChainX的跨链交易，实现比特币网络和ChainX之间的资产价值转移。CXBridge本质上就是ChainX上的一种智能合约，被部署在ChainX的所有节点上，用于处理ChainX与BTC跨链交易的相关请求。CXBridge**通过分布式资产保险库的形式进行资产管理，并通过抵押的方式进行风险控制。**这有别于ChainX1.0中信托多签模式的去中心化托管方案。

![](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/概念图.png)

## 关键概念

### 参与角色

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

### 核心功能

本部分将介绍CXBridge负责的核心功能。CXBridge的核心功能包括：

* 充值（Recharge）
* 提现（Redeem）
* 抵押（Collateral）
* 清算（Liquidation）

#### 充值（Recharge）

用户进行充值操作，即将比特币网络中的BTC转化为ChainX中的XBTC，这一过程将发行XBTC。在这个过程中，用户将在BTC网络中向选定的一个资产保险库转账，资产保险库将给该用户对应数量的XBTC。其完整流程如下：

![充值流程](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/充值流程.png)

* 当User提交一个Recharge请求时，请求中包括User要求充值的XBTC数量，以及少量的用于抵押的PCX。提交请求后首先需要确定一个Vault，并且要保证该Vault有足够的XBTC发行能力。
* 选择好Vault后，User将获取到Vault的BTC网络地址，接着User将向BTC网络提交转账交易，即从User绑定的BTC账户向Vault账户中进行转账。
* 转账完成后，ChainX将获取到BTC网络传回的转账交易证明，进行验证。
* BTC转账交易验证完成后，将发行XBTC给User。

**需要注意的是，在User请求Recharge时，需要保证Vault已按照Vault注册表锁定了抵押品，并且其XBTC有足够的能力发行用户请求的XBTC。另外，如果User没有在预定时间内完成Recharge操作，系统将取消此次请求，并且User在发起Recharge请求时抵押的PCX将归Vault所有，这是为了防止恶意的Recharge请求。**

#### 提现（Redeem）

用户进行提现操作，即将ChinaX中的XBTC价值转移到BTC网络中，这一过程用户将销毁要提现的XBTC，而Vault将在BTC网络中向User的BTC网络账户进行转账。其完整流程如下：

![提现流程](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/提现流程.png)

* 用户在提交Redeem请求时，将根据请求的提现数额，锁定自己账户中的等量XBTC。
* 在锁定好用于提现资产后，用户需要确定一个Vault，用于处理此次的提现交易请求，该Vault的BTC账户需要有足够的BTC资产进行转账操作。
* 选定好Vault后，Vault将从用户的请求中获取到用户的BTC地址，然后Vault将向BTC网络提交转账交易，即从Vault的BTC账户向User的BTC账户进行转账。
* 转账完成后，BTC网络将转账证明返回给ChainX网络，并对证明进行验证。
* 验证完成后，User将销毁锁定的用于提现的XBTC。

**需要注意的是，如果在给定的时间内，Vault未能完成Redeem操作，用户将向Vault收取等价值量的PCX以弥补在其BTC网络中的损失，或者用户也可以选择取消该次Redeem操作，再重新选择一个资产保险库执行Redeem操作。**

没有BTC账户的ChainX用户是可以进行Recharge而不可Redeem的。当一个没有BTC账户的ChainX用户进行Recharge操作时，由于没有用户BTC地址，Recharge将失败，而Recharge需要用户抵押一定的PCX，失败后，这些PCX将被转入处理的Vault账户。而Redeem操作本身就需要用户锁定XBTC，而无法进行Recharge的用户是没有XBTC的，所以无法进行Redeem操作。

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
**α**称为风险系数，当风险系数小于某一阈值时，我们认为该Vault的风险较高，会对该Vault进行清算。清算的方式是把该Vault所有抵押的PCX转给系统Vault，并且取消该Vault的资格。

## API介绍

### 总体结构

![程序结构图](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/程序结构图.png)

* Pallet为智能合约部分 ，其中Config是合约的配置，Call为对外开发的接口，Internal部分是内部私有接口，用以Call调用。
* Genesis为创世区块配置，可设置默认配置，Build用以创建创世区块。
* Event和Error分别为事件类型和错误类型。

### Pallet Config

| Config选项名              | 选项描述                                                     |
| ------------------------- | ------------------------------------------------------------ |
| Event                     | 事件类型                                                     |
| TargetAssetId             | 目标资产ID，指BTC资产ID                                      |
| DustCollateral            | Vault抵押品的下限                                            |
| SecureThreshold           | 安全阈值，即风险系数高于该值时，认为Vault安全                |
| PremiumThreshold          | 溢价阈值，当Vault风险系数小于该值时，执行Redeem需要支付额外费用给User |
| LiquidationThreshold      | 清算阈值，当Vault风险系数小于该值时，将清算该Vault           |
| IssueRequestExpiredTime   | Recharge请求过期时间，当Recharge请求时延超过该值，则请求将被取消 |
| RedeemRequestExpiredTime  | Redeem请求过期时间，当Redeem请求时延超过该值，则请求将被取消 |
| ExchangeRateExpiredPeriod | 汇率过期时间，每个一段时间更新汇率                           |
| RedeemBtcDustValue        | 提现BTC的最小额度，若提现额小于该值，则取消该Redeem请求      |

### 充值

**简单描述**

User申请Recharge请求以及处理Recharge请求的接口。

| 方法名        | 方法描述                 |
| ------------- | ------------------------ |
| request_issue | 请求充值，即请求发行XBTC |
| execute_issue | 执行充值，即发行XBTC     |
| cancel_issue  | 取消充值，即取消发行XBTC |

**接口描述**

* request_issue

  请求充值，即请求发行XBTC。

  * 参数列表

    | 参数名              | 参数描述           |
    | ------------------- | ------------------ |
    | origin              | 发起Recharge的User |
    | vault_id            | User选定的Vault    |
    | btc_amount          | Recharge的BTC数量  |
    | griefing_collateral | 抵押品数量         |

  * 返回值类型

    DispatchResultWithPostInfo

  * 程序流程

    ![请求充值程序流程](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/请求充值程序流程.png)

* execute_issue

  执行充值，即发行XBTC。

  * 参数列表

    | 参数名        | 参数描述            |
    | ------------- | ------------------- |
    | origin        | 发起Recharge的User  |
    | request_id    | Recharge请求ID      |
    | _tx_id        | BTC交易ID           |
    | _merkle_proof | BTC交易的merkle证明 |
    | _raw_tx       | 原Recharge交易      |

  * 返回值类型

    DispatchResultWithPostInfo

  * 程序流程

    ![执行充值程序流程](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/执行充值程序流程.png)

* cancel_issue

  取消充值，即取消发行XBTC。

  * 参数列表

    | 参数名     | 参数描述           |
    | ---------- | ------------------ |
    | origin     | 发起Recharge的User |
    | request_id | 请求ID             |

  * 返回值类型

    DispatchResultWithPostInfo

  * 程序流程

    ![取消充值程序流程](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/取消充值程序流程.png)

### 提现

**简单描述**

User申请Redeem请求以及处理Redeem请求的接口。

| 方法名         | 方法描述                 |
| -------------- | ------------------------ |
| request_redeem | 请求提现，即请求销毁XBTC |
| execute_redeem | 执行提现，即销毁XBTC     |
| cancel_redeem  | 取消提现，即取消销毁XBTC |

**接口描述**

* request_redeem

  请求提现，即请求销毁XBTC。

  * 参数列表

    | 参数名        | 参数描述         |
    | ------------- | ---------------- |
    | origin        | 发起Redeem的User |
    | vault_id      | User选定的Vault  |
    | redeem_amount | 提现的数额       |
    | btc_address   | User的BTC地址    |

  * 返回值类型

    DispatchResultWithPostInfo

  * 程序流程

    ![请求提现程序流程](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/请求提现程序流程.png)

* execute_redeem

  执行提现，即销毁XBTC。

  * 参数列表

    | 参数名        | 参数描述            |
    | ------------- | ------------------- |
    | origin        | 发起Redeem的User    |
    | request_id    | Redeem请求ID        |
    | _tx_id        | BTC交易ID           |
    | _merkle_proof | BTC交易的merkle证明 |
    | _raw_tx       | 原Redeem交易        |

  * 返回值类型

    DispatchResultWithPostInfo

  * 程序流程

    ![执行提现程序流程](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/执行提现程序流程.png)

* cancel_redeem

  取消提现，即取消销毁XBTC。

  * 参数列表

    | 参数名     | 参数描述          |
    | ---------- | ----------------- |
    | origin     | 发起Redeem的User  |
    | request_id | Redeem请求ID      |
    | reimburse  | 是否需要Vault补偿 |

  * 返回值类型

    DispatchResultWithPostInfo

  * 程序流程

    ![取消提现程序流程](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/取消提现程序流程.png)

### Vault管理

**简单描述**

User通过抵押PCX注册成为Vault的方法。

| 方法名         | 方法描述          |
| -------------- | ----------------- |
| register_vault | User注册成为Vault |

**方法描述**

* register_vault

  * 参数列表

    | 参数名      | 参数描述          |
    | ----------- | ----------------- |
    | origin      | 注册的User        |
    | collateral  | 抵押品            |
    | btc_address | User在BTC中的地址 |

  * 返回值类型

    DispatchResultWithPostInfo

  * 程序流程

    ![Vault注册程序流程](https://github.com/Black-Block/CXbridge-Document/blob/main/picture/Vault注册程序流程.png)

### 抵押相关

**简单描述**

提供对抵押品的操作，获取抵押品信息的相关方法。

| 方法名                        | 方法描述                      |
| ----------------------------- | ----------------------------- |
| collateral_ratio_of           | 获取Vault的抵押比，即风险系数 |
| lock_collateral               | 锁定抵押品                    |
| unlock_collateral             | 解锁抵押品                    |
| slash_collateral              | 转移抵押品                    |
| calculate_collateral_ratio    | 计算抵押比                    |
| calculate_required_collateral | 计算满足需求的抵押品阈值      |
| calculate_slashed_collateral  | 计算已转移的抵押品            |

### 清算相关

**简单描述**

对Vault进行清算相关操作的相关方法。

| 方法名                   | 方法描述          |
| ------------------------ | ----------------- |
| recover_from_liquidating | 回复被清算的Vault |
| liquidate_vault          | 清算Vault         |
| _check_vault_liquidated  | 检查被清算的Vault |

### 代币转换

**简单描述**

用于不同资产之间的价值转换的相关方法。

| 方法名         | 方法描述       |
| -------------- | -------------- |
| convert_to_pcx | 将BTC转换为PCX |
| convert_to_btc | 将PCX转换为BTC |

### XBTC管理

**简单描述**

对ChainX中的XBTC进行管理操作的相关方法。

| 方法名                                | 方法描述             |
| ------------------------------------- | -------------------- |
| move_xbtc                             | 转移XBTC             |
| reserve_xbtc_to_withdrawal            | 锁定User的XBTC       |
| release_xbtc_from_reserved_withdrawal | 解锁锁定的User的XBTC |
| burn_xbtc                             | 销毁XBTC             |
| usable_xbtc_of                        | 获取User可用的XBTC   |

### 请求管理

**简单描述**

获取请求(request)的信息的相关操作。

| 方法名                      | 方法描述                       |
| --------------------------- | ------------------------------ |
| get_next_issue_id           | 获取下一个Recharge请求的ID     |
| get_next_redeem_id          | 获取下一个Redeem请求的ID       |
| get_issue_request_by_id     | 通过request ID获取Recharge请求 |
| get_redeem_request_duration | 获取Redeem请求时延             |
| get_issue_request_duration  | 获取Recharge请求时延           |

### 其他

**简单描述**

其他方法。

| 方法名                     | 方法描述              |
| -------------------------- | --------------------- |
| update_exchange_rate       | 更新交易率            |
| add_extra_collateral       | Vault增加额外的抵押品 |
| force_update_exchange_rate | 强制更新汇率          |
| force_update_oracles       | 强制更新预言机        |
| update_issue_griefing_fee  | 更新充值手续费        |

### Genesis Config

创世区块的配置项。

| Config选项名       | 选项描述        |
| ------------------ | --------------- |
| exchange_rate      | 汇率            |
| oracle_accounts    | 语言机账户      |
| liquidator_id      | 清算者/监察者ID |
| issue_griefing_fee | Recharge手续费  |
| redeem_fee         | Redem手续费     |

### Event Type

系统事件类型

| 事件类型                              | 事件描述               |
| ------------------------------------- | ---------------------- |
| ExchangeRateUpdated                   | 汇率更新               |
| ExchangeRateForceUpdated              | 汇率强制更新           |
| OracleForceUpdated                    | 预言机强制更新         |
| CollateralSlashed                     | 抵押品转让             |
| BridgeCollateralReleased              | 抵押品发放（to User）  |
| ExchangeRateExpiredPeriodForceUpdated | 汇率过期时间强制更新   |
| VaultRegistered                       | Vault注册              |
| ExtraCollateralAdded                  | 抵押品增加             |
| CollateralReleased                    | 抵押品释放（to Vault） |
| NewIssueRequest                       | 新Recharge请求         |
| IssueRequestExecuted                  | 执行Recharge           |
| IssueRequestCancelled                 | 取消Recharge           |
| NewRedeemRequest                      | 新Redeem请求           |
| RedeemExecuted                        | 执行Redeem             |
| RedeemCancelled                       | 取消Redeem             |
| GriefingFeeUpdated                    | 提现手续费更新         |

### Error Type

系统错误类型。

| 错误类型                       | 错误描述                                   |
| ------------------------------ | ------------------------------------------ |
| NotOracle                      | 非预言机，拒绝访问                         |
| InsufficientFunds              | 请求者没有足够的PCX作为抵押品              |
| ArithmeticError                | 计算下溢或溢出                             |
| InsufficientCollateral         | 账户没有足够的抵押品来转让                 |
| BridgeNotRunning               | CXBridge关闭或出错                         |
| NoIssuedTokens                 | 尝试在没有发行XBTC时计算抵押率（风险系数） |
| CollateralAmountTooSmall       | 抵押数额不足最小抵押额度                   |
| InsufficientVaultCollateral    | 抵押数额在转移后不足最小抵押额度           |
| VaultAlreadyRegistered         | Vault已注册                                |
| BtcAddressOccupied             | BTC地址被占用                              |
| VaultNotFound                  | 没有找到Vault                              |
| VaultInactive                  | Vault不活跃                                |
| InsufficientGriefingCollateral | User充值抵押品不足                         |
| IssueRequestNotFound           | 没有找到该Recharge请求                     |
| IssueRequestNotExpired         | Recharge请求在未过期时被取消               |
| InvalidConfigValue             | 无效的配置值                               |
| IssueRequestExpired            | Recharge请求过期                           |
| InsecureVault                  | 不安全的Vault，Vault抵押率低于安全阈值     |
| RequestDealt                   | 请求已被执行或取消                         |
| RedeemRequestNotFound          | 没有找到该Redeem请求                       |
| RedeemRequestNotExpired        | Redeem请求在未过期时被取消                 |
| RedeemRequestExpired           | Redeem请求过期                             |
| VaultLiquidated                | Vualt已被清算                              |
| InvalidRequester               | 无效的请求，行动者不是请求的所有者         |
| AmountBelowDustAmount          | 提现额过低                                 |
| InsufficiantAssetsFunds        | 提现额度不正确                             |
| RedeemRequestProcessing        | 正在提现中                                 |
| RedeemRequestAlreadyCompleted  | Redeem请求已完成                           |
| RedeemRequestAlreadyCancled    | Redeem请求已被取消                         |
| BridgeStatusError              | CXbBridge状态不正确                        |
| InvalidBtcAddress              | BTC地址无效                                |
| RedeemAmountTooLarge           | 提现数额超过Vault库存                      |
| AssetError                     | 从xpallet_assets传递的错误                 |
