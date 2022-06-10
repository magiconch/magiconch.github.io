---
title: 如何判断一个地址是否为合约地址
date: 2022-06-10
tags:
 - web3
 - 以太坊
 - 小旱獭的以太坊笔记
categories:
 - 技术文章
---

首先我们都知道地址分为两种，分别是合约地址和外部地址，大家平时使用的小狐狸钱包上的地址就是外部地址，具体两个地址的差异可以去看这篇文章，今天我们集中来讲一下如何来判断地址。



### 为什么需要判断一个地址为合约地址

我们可以在很多合约的代码中（尤其是NFT）看到禁止合约账户进行访问，我们都知道是为了防止科学家，但是科学家到底做了什么事情，让各大开发者都这么害怕合约用户的访问呢？

这里有一个通过MEV进行攻击的例子

- [Sevens NFT](https://dappradar.com/blog/drama-at-launch-the-sevens-nft-collection-suffers-minting-exploit)

- [什么是MEV](https://ethereum.org/zh/developers/docs/mev/)

### 0x00 使用Address提供的api

`OpenZeppelin`有一个叫做Address的库，它提供了一种简便的方法来识别，

```solidity
function isContract(address account) internal view returns (bool) {
	return account.code.length > 0;
}
```

这段代码通过合约账户hashcode不为0的特性来区分合约账户。



### 0x01 通过汇编

solidity是一门追求高效的语言，代码越高效，花费的真金白银就越少，因此还存在一种汇编写法：

```solidity
function isContract(address addr) returns (bool) {
  uint size;
  assembly { size := extcodesize(addr) }
  return size > 0;
}
```

这里通过assembly引入汇编代码，根本逻辑还是和Address没有区别



### 0x02 （目前）最安全的写法

其实前两种写法会存在一些问题，我们来看这样一个case：

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.13;

contract Target {
    function isContract(address account) public view returns (bool) {
        // This method relies on extcodesize, which returns 0 for contracts in
        // construction, since the code is only stored at the end of the
        // constructor execution.
        uint size;
        assembly {
            size := extcodesize(account)
        }
        return size > 0;
    }

    bool public pwned = false;

    function protected() external {
        require(!isContract(msg.sender), "no contract allowed");
        pwned = true;
    }
}

contract FailedAttack {
    // Attempting to call Target.protected will fail,
    // Target block calls from contract
    function pwn(address _target) external {
        // This will fail
        Target(_target).protected();
    }
}

contract Hack {
    bool public isContract;
    address public addr;

    // 当合约处于正在创建阶段，code为0，可以绕开IsContract的判断
    // This will bypass the isContract() check
    constructor(address _target) {
        isContract = Target(_target).isContract(address(this));
        addr = address(this);
        // This will work
        Target(_target).protected();
    }
}

```

这里我们可以看到，如果攻击者使用合约部署时的构造函数来访问合约，就会绕过前面两者合约的判断，目前常用的合约判断方式是使用

```solidity
tx.origin == msg.address
```

这里`tx.origin`是Solidity的一个全局变量，它遍历整个调用栈并返回最初发送调用（或事务）的帐户的地址。但是我们说这是目前的最优解，因为这个api可能会在未来不被支持。事实上，这个`tx.origin`也不是什么白莲花，也存在一些可以被攻击的地方，我们不应该使用它进行身份验证。



#### 加餐： tx.origin的安全问题

```solidity
contract Wallet {
    address public owner;

    constructor() payable {
        owner = msg.sender;
    }

    function transfer(address payable _to, uint _amount) public {
        require(tx.origin == owner);

        (bool sent, ) = _to.call{value: _amount}("");
        require(sent, "Failed to send Ether");
    }
}

contract Attack {
    address payable public owner;
    Wallet wallet;

    constructor(Wallet _wallet) {
        wallet = Wallet(_wallet);
        owner = payable(msg.sender);
    }

    function attack() public {
        wallet.transfer(owner, address(wallet).balance);
    }
}
    
```





> refs:

- https://solidity-by-example.org/hacks/contract-size