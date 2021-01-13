## 概述：

现实中运行的公链与ChinX资产交互流通的四种方式

     1.轻节点验证
     
     2.多签控制
     
     3.POA（Proof of authority）
     
     4.抵押发行 （本文档所采用的方案）
     

--------------------

## 流程：

#### 1.发行XBTC（即充值）

         用户输入必要信息（目标账户、发行的数量）
                       |
                       V
                 系统匹配资金保险库
                       |
                       V
             用户输确认充值并进行签名
                       |
                       V
    用户向资产保险库热地址充值BTC附带OP_RETURN(ChainX账户信息)
                       |
                       V
      从已确认的BTC区块中获取与ChainX用户相关的交易
                       |
                       V
      中继链发现这笔交易并提交交易至转接桥中
                       |
                       V
             ChainX对用户X-BTC进行更新
                       |
                       V
              用户观看到自己的X-BTC增多
              
  充值接口：
  ```rust
     fn apply_issue(origin, to_user, xbtc_count) -> _ {
       let sender = ensure_signed!(origin);
       let valt = assign_valt(sender,to_user,xbtc_count);
       sign_to_transaction(sender);
       validate_BTC(sender,xbtc_count);
       addxbtc_to_user(xbtc_count);
     }
   ```

#### 2.赎回BTC（即提现）

           用户向ChainX发起申请提现X-BTC
                      |
                      V
            ChainX锁定该用户的X-BTC
                      |
                      V
             资产保险库获取到该提现请求
                      |
                      V
              其他资产保险库给该请求签名
                      |
                      V
        ChainX中继链提交该提现请求至比特币网络中
                      |
                      V
         ChainX中继链得到比特币网络中提现确定
                      |
                      V
               ChainX销毁锁定的X-BTC
    
   提现接口：
   ```rust
     fn apply_redeem(origin, xbtc_count, to_btc_address) -> _ {
       let sender = ensure_signed!(origin)?
       let find_valt = valt_find_transaction(sender,xbtc_count,to_btc_address);
       valt_lock_xbtc(xbtc_count);
       other_valt_sign();
       tell_btc_unlock(btc_count,to_btc_address);
       destroy(xbtc_count);
     }
   ```
             
    
             
             
             
             
             
