## 访问控制漏洞

* https://www.bilibili.com/video/BV1pa411R7ve/
* http://www.codebaoku.com/smartcontract/smartcontract-access.html

```
攻击者进入不该被访问的函数/变量

solidity访问控制
1.代码层面的可见性:public, private, external, internal
2.逻辑层面权限约束 
```

* 代码层面的可见性

```solidity
public – 公共状态变量可以在内部访问，也可以从外部访问。对于公共状态变量，将自动生成一个 getter 函数。
private – 私有状态变量只能从当前合约内部访问，派生合约内不能访问。
external - 外部状态变量只能在合约之外调用，不能被合约内的其他函数调用。（合约内调用通过this.func方式）
internal – 内部状态变量只能从当前合约或其派生合约内访问。
```

#### 真实案例2022-04-11 Creat Future - Lost $1.9 million

```solidity
https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/past/2022/README.md#20220411-creat-future

演示代码:
contract CFToken{
	mapping(address=>uint) private accounts;
	function _transfer(address from, address to, uint amount) public{
		//transfer amount from A to B
		// should to set this function as 'private'
	}
	function deposit() external payable{
		//deposit
	}
	function getMyBalance public view returns(uint){
		return accounts[msg.sender];
	}
}
```

#### 真实案例2022-12-25 - Robic - Lost $1.5 million 

```solidity
https://github.com/SunWeb3Sec/DeFiHackLabs/blob/main/past/2022/README.md#20221225---rubic---arbitrary-external-call-vulnerability

演示代码:
contract Rubic{
	uint private balance = 0;
	uint private collectedFees = 0;
	uint private feePercent = 10;
	uint private pyramidMultiplier = 300;
	uint private payoutOrder = 0;
	address private creator;
	//sets creator
	function DynamicPyramid(){
		creator = msg.sender;
	}
}	//这个合约2016年的代码，没有声明作用域，默认是public; 
	//本来合约叫做DynamicPyramid，则这个函数是constructor，
	//后来改了合约名称，但是忘记改这个函数名称
```













