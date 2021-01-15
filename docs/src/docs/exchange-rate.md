---
title: Exchange Rate
lang: zh-cn
sidebar: auto
---

# 汇率

## 储存
### ExchangeRate
```rust
u128
```
### LastExchangeRateTime
```rust
Moment
```

### MaxDelay
```rust
Moment
```
更新汇率的最长间隔

### AuthorizedOracles
```rust
map(AccountId) => Vec<u8>
```
允许更新汇率的账户

## 交易

### 设置汇率
```rust
fn set_exchange_rate(origin, btc_dot: u128) -> _ {
  let signer = ensure_authorized(origin)?;
  ...
}
```

### 设置BTC每字节的交易费
```rust
fn set_btc_tx_fees_per_bytes(origin, fast, half, hour) -> _ {
  ensure_not_shutdown()?;
  ...
}
```
