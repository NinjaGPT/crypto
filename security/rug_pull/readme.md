## 欺诈手段汇总
#### 诈骗合约主要的欺诈手段，包括:
* 使用代理合约
* 预留自毁合约
* 函数签名欺诈
* 调用外部合约
* 貔貅代币
* 限制代币卖出
* 存在暂停交易功能
* 存在交易冷却功能
* 存在黑名单
* 存在白名单
* 不合理交易税
* 针对特定地址改税
* 内置防巨鲸功能
* 欺骗巨鲸追随者
* 套路机器人
* 存在增发功能
* 可重获所有权
* Owner可改余额
* 批量Meme骗局
* 资金盘骗局
* 倾销砸盘
* 移除流动性

----

#### 欺诈合约常用模版：
https://github.com/binschoolapp/rug-pull-contract/
----

## ERC20合约模版
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract MyToken is ERC20 {
    constructor() ERC20("MyCoin", "MC") {
        _mint(msg.sender, 100*10**18);
    }
}
```


