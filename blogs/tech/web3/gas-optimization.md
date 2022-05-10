---
title: NFT的gas优化终极指南
date: 2022-05-01
tags:
 - web3
 - gas优化
 - nft
 - 转载
categories:
 - 技术文章
---

在我们尝试着创造一个新的收藏品的时候，发现gas费比NFT本身还要贵！

本文旨在解决上面的问题。

接下来我们将看到的是，NFT智能合约的工程团队去寻找降低gas费用的方法时，会发生什么。

本指南是我们精心研究和实验的结果。

**在本文中，我们将通过不同的方法来提高铸造的成本效益：**

- 我们真的需要ERC721Enumerable吗?
- 使用映射而不是数组
- ERC721A标准
- 从代币 Id 1开始
- 白名单的Merkle树
- 打包变量
- 使用未经检查的
- 为什么第一个铸造的更贵，我们能做什么?
- 使用优化器
- 将' if语句'转换为单独的函数

本文中提到的所有代码都可以在如下的Github上找到：

https://github.com/WallStFam/gas-optimization

## 我们真的需要ERC721Enumerable吗?

在编写mint函数时，需要确保该函数使用了所需的最少代码。

有时，在合约中添加更多的函数以备将来使用，或者使链下查询更容易。问题是，我们添加的任何额外函数都会增加gas成本。

使用昂贵的Mint函数最常见的情况之一是让我们的合约继承自 ERC721Enumerable。

这个扩展的问题是，它为转账增加了大量的开销。

ERC721Enumerable使用4个映射和一个数组来跟踪每个用户的代币id。在每次转账中写入这些结构要花费大量的gas。

以下是两个智能合约制造一个代币的gas成本的比较。一个继承自ERC721Enumerable，另一个不继承:

|contract | Gas Used |
|---|---|
ERC721 | 73,549
ERC721Enumerable | 145,323

ERC721Enumerable的价格是普通ERC721的2倍！

如果我们看看排在第一次铸造之后的铸造，我们就会发现两者的差异更明显：

|contract | Gas Used(after first mint) |
|---|---|
ERC721 | 56,439
ERC721Enumerable | 150,923

ERC721Enumerable在第一次铸造后的成本几乎比 ERC721 高出 3 倍！

> 注意：在solidity中，将变量从0设置为非0比从非0设置为非0更昂贵。这就是为什么在ERC721中的第一个铸造会更昂贵，因为用户的余额从0变化到1。

但有趣的是，ERC721中的第一次铸造更贵，ERC721Enumerable中的第一次铸造更便宜。如果我们有兴趣并且想知道为什么会这样，请查看Open Zepplin的ERC721Enumerable的98行。

> 注解  _ownedTokens[to][length] = tokenId;

第一次铸造的是更昂贵的，因为_addTokenToOwnerEnumeration(address，uint256)在第一次铸造将映射中的值设置为零，并将值从0设置为0是没有成本。

因此，在添加ERC721Enumerable之前，请自问：“我的合约中真的需要这个函数吗?”

如果我们只打算从合约之外查询每个用户的代币id，那么有一些方法可以做到这一点，而无需使用ERC721Enumerable。

这里有两种方法：

为每个代币调用ownerOf(uint tokenId)。
查询来自ERC721的Transfer事件并处理它们以获得每个代币的所有者
可以在我们的Github库中找到这两种方法的脚本：

[getAllOwners_ownerOf](https://github.com/WallStFam/gas-optimization/blob/master/scripts/getAllOwners/getAllOwners_ownerOf.js)

[getAllOwners_Transfer_event](https://github.com/WallStFam/gas-optimization/blob/master/scripts/getAllOwners/getAllOwners_Transfer_event.js)


这些脚本查询区块链以获得ERC721合约的每个代币的所有者。

我们在这些脚本中使用了Wall Street Dads合约作为示例，但是我们可以自由地将该代码用于任何其他合约。我们只需要替换abi和合约地址。

## 使用映射而不是数组

有时可以用映射替代数组的函数。映射的优点是，我们可以访问任何值，而不必像通常使用数组那样进行迭代。

例如，在NFT集合中使用白名单是很常见的。被添加到白名单的用户有优先权，通常可以得到比公开销售更低的价格。

我们可以做一个白名单使用数组如下:

```js solidity
address[] whitelistedUsers;
function mintPublicSale() external payable {
    require(msg.value >= 0.2 ether, "Not enough ether");
    _mint(msg.sender, currTokenId++);
}
function mintWhitelist() external payable {
    require(isWhitelisted(msg.sender), "You are not whitelisted");
    require(msg.value >= 0.1 ether, "Not enough ether");
    _mint(msg.sender, currTokenId++);
}
function addToWhitelist(address user) external onlyOwner {
    require(!isWhitelisted(user), "User is already whitelisted");
    whitelistedUsers.push(user);
}
/// @dev 是否是白名单
function isWhitelisted(address _user) public view returns (bool) {
    for(uint i=0; i<whitelistedUsers.length; i++){
        if(whitelistedUsers[i] == _user){
            return true;
        }
    }
    return false;
}
```

尽管这段代码有用，但它有一个大问题：随着越来越多的用户被添加到whitelistedUsers数组，调用mintWhitelist()的变得越来越昂贵。这是因为数组越大，需要迭代的次数就越多，以确定是否添加了用户。

通常情况下，Solidity中的循环可能不是正确的解决方案。在某些情况下使用数组是可以的，但要确保循环是有界的。这意味着循环具有已知的最大迭代次数，并且迭代次数相对较少。在这个例子中，循环没有边界，这就是问题所在。

如果循环是无界的，则需要尝试不同的方法。也许可以将东西移出链或使用不同的代码结构。

让我们用映射而不是数组来重写代码:

```js solidity
mapping(address => bool) whitelistedUsers;
function mintPublicSale() external payable {
    require(msg.value >= 0.2 ether, "Not enough ether");
    _mint(msg.sender, currTokenId++);
}
function mintWhitelist() external payable {
    require(isWhitelisted(msg.sender), "You are not whitelisted");
    require(msg.value >= 0.1 ether, "Not enough ether");
    _mint(msg.sender, currTokenId++);
}
function addToWhitelist(address _user) external onlyOwner {
    whitelistedUsers[_user] = true;
}
function isWhitelisted(address _user) public view returns (bool) {
    return whitelistedUsers[_user];
}
```
使用whitelistedUsers映射允许智能合约在一条指令中检查一个用户是否被列入白名单，而不是通过循环迭代。

这使得代码的执行成本更低，并且当添加更多用户到白名单时，它不会变得更昂贵(白名单用户的成本是不变的)。

这是针对不同数量的用户调用mintWhitelist()的平均成本：

|  | WhiteListMapping(Avg) | WhiteListArray(Avg) |
|---|---|---|
MintWhiteList | 58,598 | 434,427

这些值是模拟位于白名单不同位置的350个用户来计算的。

使用WhitelistArray获得的值将取决于用户(msg.sender)在数组中的位置。

如果它在开头，对mintWhitelist()的调用将比在末尾调用要便宜得多。

对于WhitelistMapping调用mintWhitelist()的gas成本是相同的(min=max=avg)，与用户数无关。

对于WhitelistArray，本例中的最小值和最大值范围为60.884(对于白名单开头的用户)到990.516(对于列表末尾的用户)。

智能合约的完整源代码可以在这里找到：

[WhitelistArray](https://github.com/WallStFam/gas-optimization/blob/master/contracts/WhitelistArray721.sol)

[WhitelistMapping](https://github.com/WallStFam/gas-optimization/blob/master/contracts/WhitelistMapping721.sol)

下面是用于计算每种方法的gas成本的脚本：

[mintWhitelisted](https://github.com/WallStFam/gas-optimization/blob/master/scripts/mintWhitelisted.js)

## ERC721A标准

来自[Azuki NFT](https://www.azuki.com/)的团队发布了ERC721的新标准ERC721A。

这个新标准允许用户制造多个代币，其成本接近制造一个代币的成本。

Azuki团队在`https://www.erc721a.org/`上分享了ERC721A的一个很好的解释。

我们将重新审视这个新标准，让它更容易被理解，并展示如何将他们提出的解决方案应用到智能合约的其他方面。

值得一提的是，铸造过程中节省的一些成本，之后会用于交易。

这取决于我们希望我们的用户在哪里节省gas。不过，ERC721A的一个好处是，铸造过程中节省的gas可能比用户需要为交易支付的额外gas多得多。

出于营销目的，Azuki团队比较了使用他们的标准铸造和使用ERC721Enumerable铸造的gas成本。

但公平地说，他们应该比较ERC721A和ERC721。当然，将ERC721A与ERC721进行比较的影响不像与ERC721Enumerable进行比较那样大，正如我们在本文中已经看到的那样，它增加了大量的开销。

但幸运的是，即使与ERC721相比，如果我们铸造多个代币，ERC721A也会使铸造成本更低，如下图所示:

| Mint Amount | ERC721 | ERC721A | Overhead |
|---|---|---|---|
Mint 1 | 56,037 | 56,372 | -335
Mint 2 | 112,074 | **58,336** | 53,738
Mint 5 | 280,185 | **64,228** | 215,957
Mint 10 | 560,370 | **74,048** | 486,322
Mint 100 | 5,603,700 | **250,808** | 5,352,892

那么，他们是如何实现如此低的gas费的呢?

他们的做法很简单：

当用户生成多个代币时，ERC721A只更新一次铸造者的余额，并且还将这批代币的所有者设置为一个整体，而不是每个代币。

为批次而不是每个代币设置变量，如果我们希望生成许多代币，则生成多个代币的成本会低得多。

正如我们前面提到的，ERC721A的问题是，由于这种铸造优化，当用户想要转移代币时，将产生更多的gas成本。

下面是通过模拟20个用户以随机顺序铸造和转移不同数量的代币100次而生成的图表:

||ERC721|ERC721A|
|---|---|---|
Avg|45,451|70,441|
Min|40,531|49,215|
Max|62,431|105,742|

在平均转账中，使用ERC721A的代币要贵55%。

要决定是否使用ERC721A，请考虑转移代币的额外成本，并考虑用户是否会铸造大量代币。

如果我们想自己检查这些值，或者模拟更高数量的用户或转账，下面是用来计算这些值的脚本：

[vs721A transfer](https://github.com/WallStFam/gas-optimization/blob/master/scripts/vs721A_transfer.js)

## 从代币 ID 1开始

许多合约的代币id从0开始(例如Azuki, BAYC等)。

如果我们，作为创造者，要做第一个铸造，这是可以的。因为正如我们之前在“我们真的需要ERC721Enumerable吗?”一节中提到的那样:在Solidity中，将变量从0设置为非0是很昂贵的。

将代币id从1开始是一个很好的技巧。这样我们的第一个铸造就会便宜得多。

下面是使用初始化为 0 和 1 的 代币Id 的 ERC721A 合约的第一个铸币厂的比较：

||ERC721A(tokenld=0)|ERC721A(tokenId=1)|Overhead|
|---|---|---|---
First mint | 90,572 | 73,472 | 17,100

因此，如果我们的某个用户打算要铸造第一个，那么通过将代币初始化为1来降低他们的成本。

## 白名单的Merkle树

在前一节“使用映射而不是数组”中，我们展示了实现白名单的合约的示例。

这些示例使用数组或映射来存储白名单地址。尽管映射比使用数组便宜，但如果我们计划有1000个(或更多)白名单用户，那么映射仍然是一个非常昂贵的解决方案。

下面是使用数组和映射将用户列入白名单的成本：



|Function|WhitelistArray|WhitelistMapping 
|---|---|---
AddToWhitelist 10 | 645,651 | 461,260
AddToWhitelist 100 | 20,393,142 | 4,612,552
AddToWhitelist 500 | 486,715,698 | 23,062,604

使用数组的代价非常高，主要是因为每次向白名单添加新用户时，都需要检查该用户是否还未被添加，当更多用户已经被添加到白名单时，检查将越来越昂贵。

注意：我们可以对WhitelistArray使用另一种方法，而不检查用户是否已经在白名单中，但这仍然会很昂贵，因为它至少与WhitelistMapping一样昂贵，后者也非常昂贵。

将500、1000甚至2000或3000用户列入白名单并不罕见。

按照目前的gas价格，白名单上的500名用户相当于要花费5648美元！

考虑到我们可能不想花那么多钱将用户添加到我们的白名单中，解决方案和最便宜的方法是使用Merkle树。

注：我们将解释Merkle树的工作原理，但这里有一个视频可以很好地解释它

[video](https://www.youtube.com/watch?v=YIc6MNfv5iQ)

Merkle树是一种存储哈希的二叉树。树中的每个叶节点都是一个哈希，父节点是子节点的哈希。

在本例中，我们将使用它来存储白名单地址，因此树的每个叶节点都是一个地址的哈希值。

一个包含 4 个地址的简单树看起来像这样:

![imgage]()

哈希H7是Merkle树的根。为列入白名单用户而使用的Merkle树的优点是，我们需要写入智能合约的唯一数据是Merkle树的根。

因此，不需要在我们的智能合约中写入数千个地址，我们只需要写入一个只有32字节的哈希。

当然，这使得向智能合约编写白名单的成本变低，而且它与白名单的大小无关(不管白名单的大小是10或10,000，成本将是相同的)。

但也有一些缺点:

使用Merkle树使白名单的Mint函数更复杂，这导致成本会高一些(进一步我们将看到多少)
从客户端调用白名单Mint函数需要做更多的工作
要检查地址是否在Merkle树中，我们需要提供所谓的Merkle证明。

Merkle证明是一个哈希数组，我们的智能合约将使用它来检查用户的地址是否在Merkle树中。它用于从叶子到根的迭代哈希，然后检查存储在智能合约中的根是否与使用证明计算的根相匹配。

如果我们回到这个例子，想象一下智能合约已经存储了根(H7)，并且地址为4的用户调用了白名单Mint函数。

前端将计算证明并将其传递给mint函数。在本例中，证明将是一个包含元素H3和H5的数组。这是因为它首先会计算H4 = Hash(地址4)，然后计算H6 = Hash(H3 + H4)，最后计算H7 = Hash(H5 + H6)。

所以它只需要 H3 和 H5 从叶子到根计算每一步的哈希值。

在下表中，我们比较了一个使用映射的合约和另一个使用Merkle树的合约的白名单Mint函数:

|Function|WhitelistMapping|WhitelistMerkle
|---|---|---
MintWhitelist|58,107|67,212


幸运的是，使用Merkle树生成白名单的开销非常小(大约多15%的gas)。这是因为在solidity中计算哈希的成本很低：

> Calculating a hash in solidity costs 30 gas + 6 gas per byte. 
> 
> For a tree of height 10, a total of 10 hashes need to be calculated: (30 + 6 * 4) * 10 = 540 gas

为了决定我们是否应该在我们的智能合约中使用Merkle树，确保我们了解利弊。

gas的成本并不高，但我们需要设置前端，以便它能够创建Merkle树的实例，并使用它来计算证明。

下面是用于计算gas成本的脚本(我们还将看到如何使用地址列表创建Merkle树，以及如何计算证明并传递给智能合约)：

[vsMerkle](https://github.com/WallStFam/gas-optimization/blob/master/scripts/vsMerkle.js)

我们在脚本中使用了以下智能合约(WhitelistMerkle721.sol)，它实现了使用Merkle树生成白名单的函数:

[WhitelistMerkle](https://github.com/WallStFam/gas-optimization/blob/master/contracts/WhitelistMerkle721.sol)

## 打包变量

Solidity将变量排列在32字节的槽中。

一些变量，如uint256占用一个完整的槽，但其他像uint8, uint16, bool等，只占用槽的一部分。

这意味着要读取一个uint8，例如，我们需要读取在同一槽的其他变量。

我们可以使用这个函数来优化gas成本：如果我们有一个函数，它使用的变量都在同一个槽中，那么读取和写入变量的成本会更低，因为我们只需要加载一次槽。

让我们看看打包3个变量的两种方法：

**A)**
```js sol
uint8 var1 = 1;
uint256 var2 = 1;
uint8 var3 = 1;
```
**B)**
```js sol
uint256 var1 = 1;
uint8 var2 = 1;
uint8 var3 = 1;
```
由于变量是按照它们在智能合约中输入的顺序打包的，在A)的情况下，这3个变量使用了3个槽，因为var2使用了一个完整的槽，使得其他变量各使用一个槽。

在B中，3个变量使用2个槽，因为var2和var3可以打包在一起。

当我们以这种方式打包变量时，我们可以节省部署合约和调用使用它们的函数的成本。

让我们来看看调用以下函数的gas成本:

```js sol
function foo() public {
    var1 = 2;
    var2 = 3;
    var3 = 4;
}
```

|Function |Gas Cost(A)|Gas Cost(B)
|---|---|---
foo() | 36,356 | 31,606

正如我们所看到的，调用foo()的gas成本增加了近5000个gas单位，这仅仅是因为变量的打包方式!

现在考虑一下在智能合约中调用和分配变量的所有不同位置。所有那些对未打包或未正确打包的变量的调用将累加。

另一件需要注意的事情是，选择将哪些变量打包在一起也很重要，因为如果变量需要同时加载，我们更愿意将它们打包在一起。

如果一个函数将被调用多次，请确保可以将函数使用的所有变量放入尽可能小的槽中。

> 其实就是创建结构体变量，有点像c语言的结构体定义

## 使用未经检查的区块

算术操作可以打包在未检查的区块中，这样编译器就不会包含额外的操作代码来检查下溢/溢出。这可以提高代码的成本效率。

让我们来看一个例子：

```js sol
uint a = 1;
uint b = 2;
uint c = 10;
function unchecked_() public {
    unchecked {
        a = a *5;
        c += a;
        b += c + a * 2;
    }
}
function checked() public {
    a = a *5;
    c += a;
    b += c + a * 2;
}

```
checked和unchecked_函数执行相同的算术操作，但它们使用的gas量不同.

节省的空间并不大，但如果我们有很多不同的算术操作，例如，在for循环中修改迭代器的值，那么我们可以使用未经检查的区块为用户节省一些gas。

## 为什么第一个铸造更贵，我们能做什么?

第一次铸造通常更贵，因为有一些变量会从零变为非零，这在 Solidity 中非常昂贵。

给定一个变量，这是将其设置为“0到非0”、“非0到非0”和“从非0到0”的成本：

| |Gas Cost|
|---|---|
From zero to non-zero | 43,300
From non-zero to non-zero | 26,222
From non-zero to zero | 21,444

设置一个从0到非0的变量的成本几乎是设置一个从非0到非0的变量的两倍。

所以，我们应该注意到这一点，如果可以将变量初始化为非0，而不是0，那么我们可以为用户节省一些gas。

可以查看智能合约和用于计算gas成本的脚本:

- https://github.com/WallStFam/gas-optimization/blob/master/contracts/SetVariables.sol
- https://github.com/WallStFam/gas-optimization/blob/master/scripts/setVariables.js

## 使用优化器

Solidity 编译器带有一个集成的优化器。

优化器应该能够降低部署和函数调用的gas成本。我们要测试一下这是不是真的！

为了使用优化器，我们需要启用它并设置“运行次数”。

从文档中可以看出:“运行的次数(--optimize-runs)大致指定了在合约的生命周期内，部署代码的每个操作码执行的频率”。

通俗地说，这意味着如果我们有需要多次调用的函数，那么我们应该设置较高的运行次数，以提示编译器如何优化我们的代码。

> 注意：默认行为取决于每个平台。例如，在Hardhat中，优化器被禁用，默认运行200次。

优化器有许多不同类型的优化。关于它们的详细解释以及优化器是如何工作的，请参考Solidity团队的[AMA (Solidity Optimizer 部分)](https://blog.soliditylang.org/2020/11/04/solidity-ama-1-recap/)。

为了评估优化器的有效性，我们测试了不同的mint函数，将run参数设置为1、200和5000。我们还在关闭优化器的情况下测试了代码：

铸造1个代币：

Optimizer | ERC721 | Enumerable721 | ERC721A | Merkle721
---|---|---|---|---|
Runs 1 | 56,127 | 150,372 | 56,600 | 67,340
Runs 200| 56,037| 150,260| 56,372| 67,180| 
Runs 5000| 56,019| 150,242| 56,345| 67,127| 
Off| 56,440| 150,924| 57,949| 69,015

铸造10个代币：

Optimizer | ERC721 | Enumerable721 | ERC721A | Merkle721
---|---|---|---|---|
| Runs 1| 561,270| 1,503,720| 74,573| 673,400| 
Runs 200| 560,370| 1,502,600| 74,048| 671,800| 
Runs 5000| 560,190| 1,502,420| 74,021| 671,270| 
Off| 564,400| 1,509,240| 75,616| 690,150

铸造100个代币：

Optimizer | ERC721 | Enumerable721 | ERC721A | Merkle721
---|---|---|---|---|
| Runs 1| 5,612,700| 15,037,200| 254,303| 6,734,000| 
Runs 200| 5,603,700| 15,026,000| 250,808| 6,718,000| 
Runs 5000| 5,601,900| 15,024,200| 250,781| 6,712,700| 
Off| 5,644,000| 15,092,400| 252,286| 6,901,500

我们注意到的第一件事是，关闭优化器总是会导致mint函数的成本更高。但不幸的是，使用优化器并没有太多好处。

gas的价格是较低，但只是少了一点点。此外，尽管运行次数越多，效果越好，但对于所有mint类型和不同的运行次数，结果几乎是相同的。

仅仅看这些结果，我们可能会得出这样的结论:“也许它没有那么好，但至少是更好了!”所以我可能会打开它，并设置运行的数量非常高。”

在得出早期结论之前，我们需要再看一下优化器的另一个方面。

优化器的工作方式是，做出一些牺牲，可能会增加生成的字节码的大小，这将增加部署智能合约的成本。

让我们看看通过改变运行量和设置优化器来部署每个合约需要多少成本：

Optimizer | ERC721 | Enumerable721 | ERC721A | Merkle721
---|---|---|---|---|
| Runs 1| 1,247,060| 1,487,799| 1,182,422| 1,550,044| 
Runs 200| 1,272,592| 1,513,331| 1,197,502| 1,579,927| 
Runs 5000| 1,465,382| 1,696,103| 1,443,689| 1,719,947| 
Off| 2,329,618| 2,780,945| 2,226,648| 2,864,429

第一个结论：最好设置优化器。将关闭优化器将使部署和调用函数的成本更高。

真正令人惊讶的是，当关闭优化器时，部署的成本会高得多，几乎是原来的两倍。

第二个结论：“运行次数”对部署成本的影响最大。运行设置为5000时，部署成本比运行设置为1或200时高出20%左右。

希望这能让我们大致了解，在将优化器应用于代码时，它会带来什么。

但请务必理解，我们给出的值只适用于ERC721的不同变体的mint函数，如果我们的合约执行完全不同的代码，我们可能会发现不同的结果。

因此，请确保以类似于我们介绍的方式测试代码，以找到针对特定情况的最便宜配置。

如果我们想自己测试这些函数或查看我们是如何计算这些值的，请参阅存储库中的scripts/testOptimizer.js文件。

我们可以将hardhat.config.js中的优化器运行参数更改为我们想要测试的任何值。我们还可以在那里启用和禁用优化器。在进行任何更改后，请确保重新编译代码。

## 将' if语句'转换为单独的函数

Wall St Moms的智能合约使用了我们称为“铸造阶段”的东西。

有3个阶段：经典，现代和元，每个阶段都有不同的要求和铸造限制。

最初，我们认为在所有阶段使用一个Mint函数是一个好主意。在内部，该函数将使用“if语句”检查当前阶段，并相应地进行操作。

这样，智能合约接口将只是一个函数，前端客户端将不需要知道合约当前处于哪个阶段。

这种方法可能适用于其他环境，但对于智能合约，它不是被推荐的模式。问题在于，检查智能合约中的阶段增加了复杂性，降低了可读性，并增加了gas成本。

让我们看一个简单的例子:

```js sol
function mint() external payable {
    require(msg.value >= 0.1 ether, "Not enough ether");
    _mint(msg.sender, tokenId);
    tokenId++;
}
function mintPhases(uint a) external payable {
    require(msg.value >= 0.1 ether, "Not enough ether");
    if(a == 1){
        _mint(msg.sender, tokenId);
        tokenId++;
    } else if(a == 2){
        _mint(msg.sender, 1000 + tokenId);
        tokenId++;
    }
}
```
如果仔细观察这两个函数，就会发现调用mint()与调用mintPhases(1)的效果完全相同。这是两个通信的gas费用:

|Function | Gas Cost |
|---|---|
mint()|56,037
mintPhases(1) | 56,325

使用if语句添加300个gas。

注：这段代码只是一个示例，用于说明这个概念。

mintphase函数可以被分成两个不同的函数来节省gas成本:

```js sol

function mintPhases_1(uint a) external payable {
    require(msg.value >= 0.1 ether, "Not enough ether");
    _mint(msg.sender, tokenId);
    tokenId++;
}
function mintPhases_2(uint a) external payable {
    require(msg.value >= 0.1 ether, "Not enough ether");
    _mint(msg.sender, 1000 + tokenId);
    tokenId++;
}
```

为每个阶段使用单独的函数将' if语句'的成本传递给前端。

现在，我们可能认为300美元不是什么大问题(实际上在当前的价格中大约是5美分)，但要意识到，我们在这里只是提出了一个非常简单的例子，同样的模式可以应用到更复杂的情况。

此外，所有的gas成本都加起来了，在这种情况下，使用该模式没有缺点，我们只需要在前端添加一些逻辑，以读取智能合约处于哪个阶段，并调用相应的mintPhases_x函数。

## 自己做测试

推动区块链技术向前发展的最佳方法之一是为最终用户创建更好的用户体验。

降低gas成本是创造更好用户体验的好方法。

在本文中，我们探索了可以应用于智能合约的不同通用技术。

但是如何处理自己的自定义代码呢?

在Solidity中优化代码的最佳方法是测试函数的gas成本。

这个想法很简单，我们计算一个函数消耗了多少gas，对代码进行更改，然后再次计算，看看是否减少了gas成本。

我们可以这样计算任何函数的gas成本:

```js sol
let tx = await contract.foo();
tx = await tx.wait(); // Wait until the transaction is mined
const gasUsed = tx.gasUsed.toNumber(); // gasUsed is a BigNumber, you can cast it to number if you need
console.log(gasUsed);

这是一个简短的版本:
const tx = await (await contract.foo()).wait();
console.log(tx.gasUsed.toNumber());
```

## 常见流行NFT合约的gas成本
在本文的最后，我们想看看流行的NFT集合的智能合约的gas成本。

我们选择了BAYC, Doodles和Cool Cats。

让我们看看每个合约的 mint 函数成本是多少：

NFT Collection | Mint 1 | Mint 5
| --- | --- | --- |
Bored Ape Yatch Club | 173,576 | 636,912
Doodles | 152,599 | 610,131
Cool Cats | 149,778 | 608,730

令人惊讶的是，这三份合约的铸造成本都非常相似，而且成本很高。

使用本文介绍的技术，我们可以将铸造1个代币的成本设定为60,000个gas左右，铸造5个代币的成本设定为70,000个gas左右(如果我们使用ERC721A来利用多个代币铸造)。

这3个合约之所以如此昂贵，是因为它们都使用了ERC721Enumerable，正如我们在“我们真的需要ERC721Enumerable吗?”所说，大多数时候是可以避免的。

铸造 5 个代币的成本非常高（因为它可能会低近 10 倍），如果他们实施 ERC721A 或类似的解决方案，他们会让他们的用户大受青睐。

## 总结
我们仍处于Web3生态系统的早期阶段。不利的一面意味着我们都将面对糟糕的用户体验和昂贵的运营。

好处是有机会排除所有可能的解决方法，以确保获得良好的体验。

----

## 小旱獭总结时间：

|方法|优点|缺点|
|---|---|---|
ERC721Enumerable | 第一次锻造成本低，方便从合约外查询每个用户的代币id | 开销几乎是ERC721的三倍 |
使用白名单映射 | 以太坊的hash运算比数组遍历好太多| 无 |
ERC721A标准 | 节省非常多的锻造成本 | 提高了转移代币时候的gas花费
从代币 Id 1开始 | 第一次锻造便宜很多 | 可能有人就是喜欢0
白名单Merkle树 | 让gas花费和白名单数量无关 | 代码更复杂
打包变量 | 字节维度节省空间 | 无
使用未经检查的区块 | 编译器就不会包含额外的操作代码来检查溢出 | 节省的空间并不大, 没有溢出检查
使用优化器 | 最好设置优化器。关闭优化器将使部署和调用函数的成本更高 | 确定运行次数
拆分判断逻辑 | 减少if判断带来了gas的优化 | 降低了代码可读性


[Source](https://medium.com/@WallStFam/the-ultimate-guide-to-nft-gas-optimization-7e9289e2d88f)







