---
title: Staked-slayer
lang: zh-cn
sidebar: auto
---

# 监察者

## 储存

### ActiveStakedRelayers  
```rust
map(AccountId) => ActiveStakedRelayer<PCX>;
```
### ActiveStakedSlayersCount  
```rust
u64;
```
### InactiveStakedRelayers  
```rust
map(AccountId) => InactiveStakedRelayer;
```
### ActiveStatusUpdates
```rust
map(u64) => StatusUpdate;
```
### InactiveStatusUpdates
```rust
map(u64) => StatusUpdate;
```
### StatusCounter
```rust
u64;
```
### TheftReports
```rust
map(BtcTx) => bset(AccountId);
```
### GovernanceId
```rust
AccountId;
```

## 交易

### 注册

```rust
fn register_staked_relayer(origin, stake) -> _ {
  let signer = ensure_signed(origin)?;
  ensure_not_register(origin)?;
  ensure_stake_exceed_minimum_amount()?;
  lock_collateral(signer, stake);
  add_inactive_relayer(signer, stake, period)
}
```

### 注销
```rust
fn deregister(origin) -> _ {
  let signer = ...;
  let staked_relayer = get(signer)?; // Error if not exists;
  ensure_is_not_active(staked_relayer)?;
  release_collateral(signer, staked_relayer.stake);
  remove_active_relayer(signer);
}
```

::: tip
如果有signer名下有活跃的状态更新请求，会被禁止注销。
:::

### 激活

```rust
fn activated(origin) {
  let signer = ...;
  ensure_inactive(signer)?;
  remove_from_inactive(signer);
  insert_to_active(signer);
}
```

### 冻结
```rust
fn deactivated(origin) -> _ {
  let signer = ...;
  ensure_active(signer)?;
  remove_from_active(signer);
  insert_to_active(signer);
}
```

### 提案
```rust
fn suggest_status_update(origin, deposit: PCX, error_code: ErrorCode, add_error: Option<ErrorCode>, remove_error: Option<ErrorCode>) -> _ {
  let signer = ...;
  if let ref ErrorCode::Shutdown = error_code {
    ensure_is_governance()?;
  }
  ensure(deposit > MinimumDeposit)
  lock_collateral（signer, deposit)?;
  let tally = ...;
  tally.aya.insert(signer);
  insert_active_status_update(StatusUpdate {
    proposer: signer,
    tally: tally,
    message,
    add_error,
    remove_error,
    new_status_code,
    old_status_code,
    start
    end: start + VoltingPeriod,
    proposal_status: Pending,
  })
}
```

### 投票
```rust
fn vote_status_update(origin, status_update_id, approve: bool) -> _ {
  let signer = ...;
  ensure_exists(status_update_id);
  let status_update = get(status_update_id);
  vote_once!(status_update.tally.vote(signer, approve));
  ActiveStatusUpdates.insert(status_update_id, status_update);
}
```

### 强制更新
```rust
fn force_status_update(origin, status_code, add_error, remove_error) -> _ {
  ensure_is_governance();
  set_parachain_status_code(status_code);
  insert_error(add_error);
  remove_error(remove_error);
}
```

### 作恶惩罚
```rust
fn slash_staked_relayer(origin, staked_relayer_id) -> _ {
  let signer = ...;
  ensure_is_governance();
  slash_staked_relayer(staked_relayer_id);
  remove_active_relayer(staked_relayer_id);

}
```

### 举报资产保险库
```rust
fn report_vault_theft(origin, vault_id, tx_id, merkle_proof, raw_tx) -> _ {
  let signer = ...;
  ensure_not_reported_yet(vault_id)?;  
  tx_invalid(except([Merge, Redeem, Replace]))?;
  liquidate_theft(vault_id);
  set_parachain_status_code(Error);
  set_error_code(Liquidation);
}
```
