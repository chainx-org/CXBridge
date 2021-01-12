# 资产保险库

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
  > 如何使用？ 如果多名Vault被清算会发生什么？
- LiquidationVault
- Vaults
- VaultBtcAdresses
- Version
  版本号

## 交易

