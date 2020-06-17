# Neo跨链设计

[English](README.md) | 中文

## 概述

所谓跨链，即是信息在两条甚至多条链之间交互的过程，这里的信息可以是一个dApp的业务逻辑，也可以是某一条链上的一种资产，等等。目前在Neo上有两种方式可以实现跨链操作：一种是通过节点如Neo-CLI和Neo-GUI等调用合约发送跨链交易，另一种是通过SDK构造跨链交易。

## 跨链的实现

以资产跨链为例，用户把资产在原链上锁定，之后在目标链上发行映射资产，同时可以在目标链上申请提现，最后在原链上解锁的过程。要实现该过程，则需要目标链可以验证原链上发生的行为，即验证原链上确实锁定了一定数量的原资产。

非资产类的信息跨链也是如此，需要目标链验证原链上的信息变化，从而做出相应的变化。

这种验证过程目前都是通过merkle证明的形式实现的，即原链将其上发生的行为存储下来，并构造一棵merkle tree，然后将merkle tree的树根和该行为的merkle proof提交给目标链。目标链根据提交的merkle root验证proof的合法性，从而确定原链上发生的行为。

## 架构

<div align=center><img width="290" height="458" src="resources/CrossChainStructure.png"/></div>

上图显示了Neo跨链生态的架构，从上到下分别是Neo链、Neo链的Relayer、中继链Poly、目标链的Relayer和目标链。简单来说，用户在Neo链上发出的跨链交易的证明会经由Relayer传递到Poly，再由目标链的Relayer传递到目标链，目标链验证Neo链上的交易证明并执行相应的交易。反之亦然。

生态中的角色定义如下：

- [**中继链 Poly**](../poly/README_CN.md)：中继链是整个生态中的重要部分，每个节点由不同的个人或组织运行，有自己独特的治理模式和信任机制，它负责将各个链连接到一起和跨链信息的传递。
- [**Relayer**](https://github.com/polynetworks/docs/blob/master/neo/How_to_become_neo_relayer_cn.md)：每条链都有自己的Relayer，它们负责把交易等跨链信息搬运到中继链或从中继链搬运到源链，并且它们会在这个过程中获取收益。
- [**应用**](https://github.com/polynetworks/docs/blob/master/neo/Neo_cross_chain_contract_dev_cn.md)：应用是指开发跨链业务的人或组织，任何人都可以部署跨链合约来构建跨链应用，然后把应用公开出去招揽用户。
- [**用户**](https://github.com/polynetworks/docs/blob/master/neo/How_to_cross_NEP5_asset_cn.md)：对跨链生态来说，最重要的就是用户，通过调用具有跨链功能的应用，实现Neo到以太坊等链的跨链业务。

## Neo链和中继链之间的区块头同步

<div align=center><img src="resources/HeaderSync.png"/></div>

上图展示了Neo链和中继链之间同步区块头的具体流程：

一方面，Neo链的共识节点由Neo持有者实时投票选举产生，理论上每个区块的验证者集合都可能变化。若验证者集合保持不变，则不需要同步该区块头，只需要同步关键区块头（即包含切换了验证者集合后的区块头），这样可以大大减少需要同步的区块头数量。同时由于Neo的merkle root不包含在区块头中，而是由共识节点独立签名并广播，所以当有跨链交易发生时，Relayer会把这些交易所在区块的merkle root和proof以及区块高度转发给中继链，中继链仅更新一下当前已同步的Neo链的高度并转发merkle root和proof。

另一方面，中继链区块头由Relayer同步到Neo链，Neo链的跨链管理合约会验证区块头的合法性，并存储区块头，该链的其它任何合约都可以从该合约中读取同步的区块头。

## Neo链和中继链之间的跨链交易

<div align=center><img src="resources/Neo2Relay.png"/></div>

从Neo链到目标链的跨链流程如下：

1. 用户在Neo链上的业务合约中发起跨链交易；
2. 业务合约调用跨链管理合约的跨链接口，跨链管理合约处理跨链请求，分配唯一自增ID，发出跨链事件；
3. 共识节点会生成该跨链交易所在区块的merkle root和该交易的merkle proof；
4. Neo的Relayer捕获跨链事件后通过RPC调用获取上述merkle root和merkle proof以及区块高度并提交到中继链；
5. 中继链验证merkle proof的合法性，将跨链交易的信息以事件的形式广播，目标链的Relayer会监听这些事件并把发往自己链的交易捕获到，然后转发到对应目标链；
6. 目标链的跨链管理合约验证中继链merkle proof的合法性，验证通过则说明原链上的跨链信息合法，目标链的跨链管理合约会调用相应的业务合约，执行目标链上的业务逻辑。

<div align=center><img src="resources/Relay2Neo.png"/></div>

从目标链返回到Neo链的跨链流程如下：

1. 用户在目标链上的业务合约中发起跨链交易；
2. 目标链的Relayer将跨链交易信息转发到中继链，调用中继链的跨链管理合约；
3. 跨链管理合约验证交易信息后，将验证后的信息存下来，并构造新的merkle tree，将树根写入中继链的区块头中，并生成跨链信息的merkle proof；
4. Neo的Relayer会监听中继链，将区块头和proof提交到Neo链；
5. Neo链的跨链管理合约会读取中继链区块头，验证中继链的proof，然后调用业务合约执行对应的逻辑。
