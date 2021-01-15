---
name: Vault
lang: zh-cn
sidebar: auto
---

# 资产保险库

资产保险库用于保管用户充值进来的BTC。每个资产保险库在注册的时候需要质押一笔PCX， 质押的PCX的多寡决定了该资产保险库最多可发行的代币。

## 存储

- VaultMinimumCollateral  
  注册成为资产保险库的PCX最小抵押数量
- PunishmentDelay  
  资产保险库惩罚时间。如果超时未处理转移或者体现请求， 那么该保险库在一段时间内被禁用。
- SecureCollateralThreshold  
  资产保险库抵押率的安全阈值
- PremiumRedeemThreshold  
  资产保险库需要超额赎回的阈值
- LiquidationCollateralThreshold  
  资产保险库被清算的阈值
- LiquidationVaultAccountId  
  在genesis中声明的账户，当资产保险库被清算时，他的X-BTC和抵押的PCX将被转移到此账户。  
  ::: details Question
  多名Vault被清算会使所有被清算的资产保险库的数据合并到此账户下。包括已发行的代币，抵押物，待发行代币和待体现代币。
  :::
- LiquidationVault  
  同上    
- Vaults  
  资产保险库的列表。
- VaultBtcAdresses  
  资产保险库的BTC地址， 地址需要唯一， 但是可以对应到同一个资产保险库。
- Version  
  版本号

## 交易

### 注册
```rust
fn register_vault(origin, collateral: PCX, btc_address: BtcAddress) -> _ {
  let sender = ensure_signed!(origin)?
  ensure_unique([sender, btc_address])?;
  ext::collateral::lock(sender, collateral)?; // Error: if collateral < minimum_collateral or collateral is not sufficiant.
  insert_vault_to_storage(Vault::new(...))
  deposit_event(...)
}
```
提供初始抵押和btc地址，保证申请人和比特币地址唯一。
```rust
Vault {
  id,
  wallet: Wallet::(BtcAddress),
  to_be_issued_tokens,
  issued_tokens,
  to_be_redeem_tokens,
  banned_until: Option<BlockHeight>,
  status: VaultStatus {
    Active,
    Liquidated,
    CommitedTheft
  }
}
```
### 增加抵押品
```rust
fn lock_additional_collateral(origin, amount: PCX) -> _ {
  let sender = ...;
  ensure_vault_exist!(sender);
  ext::collateral::lock(sender, amount)?;
}
```

在抵押率低的时候， 允许资产保险库增加抵押品。

### 抵押品提现
```rust
fn withdraw_collateral(orgin, amount: PCX) -> _ {
    ...
}
```
从金库中解锁抵押品，需满足提现之后的余额仍然高于最小抵押额度和安全阈值。安全阈值通过其已发行的代币和汇率换算得来。

### 更新比特币地址
```rust
fn update_btc_address(origin, address) -> _ {
    ...
  }
```
新增比特币地址, 插入到存储中，一个资产保险库可以有多个比特币地址，在地址之间的转账是允许的。
