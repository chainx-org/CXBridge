---
title: 金库
sidebar: auto
lang: zh-cn
---
# 概述

金库(Collateral)用于管理账户的抵押品(PCX)，包括用户充值请求的抵押和资产保险库的抵押。
金库本身不提供对外的交易，仅内部模块交互时使用，

## 储存
### TotalCollateral
```rust
u32
```
金库的总余额。

