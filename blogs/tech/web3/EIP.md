---
title: 什么是以太坊升级
date: 2022-05-01
tags:
 - web3
 - 以太坊
 - 小旱獭的以太坊笔记
 - 以太坊基本概念
categories:
 - 技术文章
---

### 什么是以太坊升级

每个软件都会有新的版本，以太坊也不例外，从以太坊创立到今天，已经经历了很多次大大小小的升级，而每次升级，都是为了完成一个或者多个提案。

#### 以太坊改进提案 Ethereum Improvement Proposals (EIPs)

EIP描述了以太坊平台的标准，包括核心协议规范、客户端 API 和合约标准。常见的有著名的ERC20 代币标准，它就是基于EIP20进行实现的。

#### 提案阶段

每个提案的需要经历以下几个过程:

- **Idea**- pre-draft的想法。它不会在EIP存储库中进行跟踪。
- **Draft** - EIP 开发中的第一个正式跟踪阶段。在格式正确后，EIP 编辑器将会把这个EIP合并到EIP存储库中。
- **Review**- EIP 作者将 EIP 标记为准备接受同行评审，并请求进行同行评审。
- **Last Call** - 这是 EIP 在进入 FINAL 之前的最终审核阶段。EIP 编辑器将分配 `Last Call` 状态并设置审核结束日期(`last-call-deadline`)，通常为 14 天后。如果此期间存在必要的规范性更改，它将回退到Review状态。
- **Final** - 此 EIP 代表最终标准。`Final EIP` 处于最终确定状态，仅限更新勘误表或添加非规范性的说明。
- **Stagnant**- 任何处于草案或审查中的 EIP 如果在 6 个月或更长时间内处于非活动状态，则将被移至停滞。作者或 EIP 编辑可以通过将 该EIP 撤回到`Draft`状态来恢复该EIP提案。
- **Withdrawn**- EIP 作者已撤回提议的 EIP。此状态具有最终性，无法再使用此 EIP 编号复活。如果这个想法在以后需要被继续探讨，它被认为是一个新的提议。
- **Living** - EIP 的特殊状态，旨在不断更新且不会达到最终状态。这其中包括最值得注意的 EIP-1。



#### EIP类型

EIP 分为多种类型

##### Standard Track

描述影响大多数/所有以太坊实现的任何更改，例如网络协议的更改、块/交易有效性规则的更改、提议的应用程序标准/约定，或影响使用以太坊的应用程序交互操作性的任何更改或添加。此外，标准 EIP 可以分为以下几类

- **Core**

需要共识分叉的改进（例如[EIP-5](https://eips.ethereum.org/EIPS/eip-5)、[EIP-101](https://eips.ethereum.org/EIPS/eip-101)），以及不一定对共识至关重要但可能与“核心开发”讨论相关的更改（例如，矿工/节点策略更改 2、3、和[EIP-86](https://eips.ethereum.org/EIPS/eip-86)的 4 个）

- Networking

包括围绕 devp2p ( [EIP-8](https://eips.ethereum.org/EIPS/eip-8) ) 和 Light Ethereum Subprotocol 的改进，以及对网络协议规范的 Whisper 和 Swarm 提出的改进。

- Interface

包括围绕客户端 API/RPC 规范和标准的改进，以及某些语言级别的标准，如方法名称 ( [EIP-6](https://eips.ethereum.org/EIPS/eip-6) ) 和合约 ABI。标签·`Interface`与`interface`存储库一致，讨论应该主要发生在该存储库中，然后再将 EIP 提交到 EIP 存储库。

- ERC

应用程序级标准和约定，包括合约标准，例如代币标准 ( [ERC-20](https://eips.ethereum.org/EIPS/eip-20) )、名称注册 ( [ERC-137](https://eips.ethereum.org/EIPS/eip-137) )、URI 方案 ( [ERC-681](https://eips.ethereum.org/EIPS/eip-681) )、库/包格式 ( [EIP190](https://eips.ethereum.org/EIPS/eip-190) ) 和钱包格式 ( [EIP- 85](https://github.com/ethereum/EIPs/issues/85) )

##### Meta

描述围绕以太坊的流程或提议，对其中的流程/事件进行更改。流程 EIP 类似于Standard Track EIP，但适用于以太坊协议本身以外的领域。他们可能会提出一个不会针对以太坊的代码库的实施方案；这一般取决于社区共识；与Informational EIP 不同，它们不仅仅是建议，一般来讲用户不能随意忽略它们。Meta EIP的内容一般包括程序、指南、决策过程的更改以及以太坊开发中使用的工具或环境的更改。任何 Meta EIP 也被视为Process EIP。

##### Informational

描述以太坊设计问题，或向以太坊社区提供一般指南或信息，但未提出新功能。Informational EIP 不一定代表以太坊社区的共识或建议，因此用户和实施者可以选择忽略Informational EIP 或听从他们的建议。



>  refs:

> [eth eips](https://eips.ethereum.org/)



