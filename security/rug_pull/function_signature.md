## 函数签名欺诈合约
* https://www.bilibili.com/video/BV1Th4y1y7gg/

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract FakeFuncSig is ERC20 {
    // 构造函数
    constructor() ERC20("MyCoin", "MC") {
        _mint(address(this), 200 * 10**18);
    }

    // ？？？
    function remedy(address addr) external returns (bool) {
        bytes memory data = abi.encodeWithSelector(0xa9059cbb, addr, balanceOf(address(this)));
        (bool success, ) = address(this).call(data);
        return success;
    }

}
```
在这个合约的构造函数中，合约给自己铸造了 200 个币。
remedy 函数使用 ABI 编码了一段数据作为参数，使用底层调用 call
（什么是ABI：https://zhuanlan.zhihu.com/p/540502709）
这段代码中，其实就是使用了 `transfer` 方法的签名，将合约中的代币全部转走的。
这个 0xa9 打头的数据实际上就是 `transfer` 的函数签名。
它通过对 `transfer` 函数签名，再拼接了接收地址 `addr`，以及合约中的所有余额，打包起来。
然后做为一个参数，传递给底层调用 `call`
执行成功后，合约中的代币被清空，全部转给了 `addr`
所以，这个 remedy 函数改写一下，其实就等价于这种写法:
```solidity
function remedy(address addr) external returns(bool) {
    bool success = transfer(addr, balanceOf(address(this)));
    return success;
}
```
但这种写法，欺骗手段太明显了，很多资深投资者一眼就能看穿。欺诈合约的欺诈者为了迷惑投资者，于是就使用了函数签名这种欺诈手法

## 什么是函数签名

在 `Solidity` 中，函数签名是指通过对函数名称和参数类型进行哈希运算，生成的唯一标识符。
在编译合约时，`Solidity` 编译器会根据函数的名称和参数类型，来计算函数签名，并将其嵌入到合约字节码中。
在已经编译好的合约的字节码中，函数签名是用来决定函数入口点的。

* 生成函数签名一共需要 3 步：

第一步：将函数的名称和参数类型按照规定的格式进行拼接;\
第二步：将拼接后的字符串进行 Keccak-256 哈希运算，得到 256 位的哈希值;\
第三步：取这个 256 位哈希值的前 4 个字节，这个值就是函数的签名。

* 举个实际的例子，按照以上步骤具体算一下。

比如，ERC20 标准合约中 transfer 函数的原型是这样的：
```solidity
function transfer(address to, uint256 amount) public returns (bool)
```
我们来计算它的函数签名。
```solidity
第一步，将函数名称和参数类型拼接成一个字符串，得到一个结果：
transfer(address,uint256)

第二步，将拼接后的字符串执行哈希运算，得到 256 位哈希值：
0xa9059cbb2ab09eb219583f4a59a5d0623ade346d962bcd4e46b11da047c9049b

第三步，取哈希值的前4个字节，得到最终结果：
0xa9059cbb

这 4 个字节就是函数 transfer 的签名。
```
把这个计算过程写成了一个合约，可以把它复制到 Remix 里自己试一下。
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

import "@openzeppelin/contracts/token/ERC20/ERC20.sol";

contract CalcFuncSig {
    // 生成函数签名
    function sig(string calldata str) external pure returns(bytes4){
        return bytes4(keccak256(bytes(str)));
    }
}
```
## 如何使用函数签名

* 看一个合约的源码：
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.0;

contract MyCoin {
    // 函数签名为 0x9d61d234
    function transfer(address to, uint256 amount) public returns (bool) {
    	 // ......
        return true;
    }

    // 函数签名为 0x8b069f2a
    function approve(address spender, uint256 amount) public returns (bool) {
    	// ......
        return true;
    }
}
```
这个合约里有两个函数 `transfer` 和 `approve`
这个合约经过编译后变成字节码后，会变成这样的一种结构：

外部调用合约方法时，都是会执行这同一份代码。只不过调用不同的方法时，EVM 会根据`调用请求中的 msg.sig 里的函数签名`，决定执行哪一段代码。
#### 外部调用的时候，请求数据就会填充到 msg 这个数据结构中，所以说，函数签名是用来决定函数入口点的。
由于函数签名是函数名称和参数类型拼接在一起，再进行哈希，截取部分哈希值生成的，所以不同的函数声明，它的函数签名是不同的。而且经过这么处理后，函数签名的数据长度统一，并且简短，易于处理。
这也是编译后合约在内部使用函数签名的原因。



