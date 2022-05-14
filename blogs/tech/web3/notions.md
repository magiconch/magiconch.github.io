---
title: Web3的一些零碎知识点
date: 2022-05-02
tags:
 - web3
 - 以太坊
 - 智能合约
 - solidity
categories:
 - 技术文章
---

## SPDX-License-Identifier 是什么

在6.0以后需要指定开源协议，类似于github上面的，如果不想开源，请使用UNLICENSED.

## 8.0引入的safeMath有什么作用

```sol
function testUnderflow() public pure returns (uint) {
    uint x = 0;
    x--;
    return x;
}

function testUncheckUnderflow() public pure returns (uint) {
    uint x = 0;
    unchecked {
        x--;
    }
    return x;
}
```

第一个函数会直接报错，第二个函数会返回uint256的最大值

## 简述一下solidity的错误处理

一般我们用revert来抛出错误，但是revert有一个问题就是他的
但是在8.0之后引入了自定义错误的新特性`error YourErrorName (address pram)`

## 合约之外的函数

## create2 函数 部署合约

## 变量

- 状态变量

在合约内部定义，永远在链上，修改需要花费gas

- 局部变量

在函数内部定义

- 全局变量

```sol
address msg.sender; // 调用函数的地址，有可能是一个人，也有可能是上个调用的合约
uint block.timestamp; // 区块的时间戳 只读的，当前的时间
uint block.number; // 区块编号

blockhash(uint blockNumber) returns (bytes32) // 指定区块的区块哈希——仅可用于最新的 256 个区块且不包括当前区块
block.chainid (uint) // 当前链 id
block.coinbase ( address ) // 挖出当前区块的矿工地址
block.difficulty ( uint ) // 当前区块难度
block.gaslimit ( uint ) // 当前区块 gas 限额
block.number ( uint ) // 当前区块号
block.timestamp ( uint) // 自 unix epoch 起始当前区块以秒计的时间戳
gasleft() returns (uint256) // 剩余的 gas
msg.data ( bytes ) // 完整的 calldata
msg.sender ( address ) // 消息发送者（当前调用）
msg.sig ( bytes4 ) // calldata 的前 4 字节（也就是函数标识符）
msg.value ( uint ) // 随消息发送的 wei 的数量
tx.gasprice (uint) // 交易的 gas 价格
tx.origin (address payable) // 交易发起者（完全的调用链）
```

## view 和 pure 的区别

区别在于是否读取链上的信息

## 常量消耗的gas值怎么理解

当我们定义常量的时候使用 constant 来定义常量，这样可以减少gas的消耗

ps：这里gas的消耗指的是通过访问函数调用变量的消耗

## 错误控制

require revert assert

