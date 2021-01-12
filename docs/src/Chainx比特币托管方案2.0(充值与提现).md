## 概述：

现实中运行的公链与ChinX资产交互流通的四种方式

     1.轻节点验证
     
     2.多签控制
     
     3.POA（Proof of authority）
     
     4.抵押发行 （本文档所采用的方案）
     

--------------------

## 流程：

#### 1.发行XBTC（即充值）


    用户向信托热地址充值BTC附带OP_RETURN(ChainX账户信息)
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

#### 2.赎回BTC（即提现）

           用户向ChainX发起申请提现X-BTC
                      |
                      V
            ChainX锁定该用户的X-BTC
                      |
                      V
             信托获取到该提现请求
                      |
                      V
               其他信托给该请求签名
                      |
                      V
        ChainX中继链提交该提现请求至比特币网络中
                      |
                      V
         ChainX中继链得到比特币网络中提现确定
                      |
                      V
               ChainX销毁锁定的X-BTC
             
             
             
             
             
             
             
             
             
             


