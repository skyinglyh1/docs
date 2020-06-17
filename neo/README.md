# Design of Neo Cross Chain

English | [中文](README_CN.md)

## Overview

Cross chain is the procedure of how information interacts between two or more block chains. The information can be the business logic of an dApp, or an asset released on one chain and etc. Currently there are two ways to achieve cross chain operation on Neo: one is to use full nodes like Neo-CLI and Neo-GUI to invoke contracts and send out cross chain transactions, the other is to construct cross chain transactions with SDK.

## Cross chain implementation

Take asset cross chain as an example: users should be able to lock their assets on the source chain and release corresponding assets on the target chain, they also can request to withdraw their assets on the target chain and finally unlock their assets from the source chain. To achieve this goal, it is required that what has happened on the source chain must be verifiable by the target chain, in this case, which means the target chain can verify that the cross chain transactions do exist and certain amount of assets are locked on the source chain.

It is same for other cross chain information. The target chain needs to verify the information change on the source chain and make corresponding actions.

Currently such verification procedures are carried out in the form of merkle proof. The source chain stores cross chain information and constrct a merkle tree, and then send the merkle tree root and the merkle proof of the information to the target chain. The target chain verify the proof using the merkle root and confirm the validity of the cross chain information.

## Cross chain structure

<div align=center><img width="290" height="458" src="resources/CrossChainStructure.png"/></div>

The above picture shows the structure of Neo cross chain ecosystem which consists of Neo chain, neo-relayer, the relay chain Poly, relayer of the target chain and the target chain. Briefly speaking, the merkle proof of the cross chain transactions created by on Neo chain is delivered to Poly by neo-relayer, and then delivered to the target chain by its relayers. The target chain verifies the proof and make corresponding actions and vice versa.

The entities involved in the ecosystem are defined as below:

- [**Poly**](../poly/README.md)：The relay chain is one of the crucial components of the cross chain ecosystem. Each node of it is deployed and maintained by different individuals or organizations. The relay chain has its unique governance and trust mechanism and is responsible for connecting different chains and relay cross chain information.
- [**Relayer**](https://github.com/polynetworks/docs/blob/master/neo/How_to_become_neo_relayer_en.md)：Every chain has its own relays which basically transmit the cross chain transaction information from or to the relay chain, thus connecting the relay chain with the outside world. Relayers collect small incentives during this procedure.
- [**Applications**](https://github.com/polynetworks/docs/blob/master/neo/Neo_cross_chain_contract_dev_en.md)：Applications refer to people or organizations who develop and implement cross chain business. Anyone can deploy cross chain contracts to start cross chain business.
- [**Users**](https://github.com/polynetworks/docs/blob/master/neo/How_to_cross_NEP5_asset_en.md)：Users are those who use cross chain applications to launch their business across multiple chains.

## Block header synchronization between Neo chain and the relay chain

<div align=center><img src="resources/HeaderSync.png"/></div>

The above picture shows the procedure of Block header synchronization between Neo chain and the relay chain:

On one hand, the consensus nodes of Neo chain are voted and elected by NEO holders for each block, so the block validators can be changed in any future block. If the block validators are not changed, then there's no need to sync this block header. Only key headers (which changes validators, are also called epoch changing blocks) are required to sync. This greatly reduces the amount of headers needed to sync. And the merkle root is not included in Neo's block headers, instead is signed and broadcast by the consensus nodes independently. So if a cross chain transaction occurs, neo-relayer sends the merkle root, merkle proof and block height to the relay chain, and relay chain updates the current synced Neo block height and relays the root and proof to the target chain.

On the other hand, the relay chain block headers are synced to Neo chain by neo-relayers. Then the cross chain management contract of Neo chain validates and stores these block headers. Any contracts on Neo chain can read these stored headers.

## Cross chain transactions between Neo chain and the relay chain

<div align=center><img src="resources/Neo2Relay.png"/></div>

The cross chain procedure from Neo chain to the target chain:

1. Users create cross chain transactions in business contracts on Neo chain;
2. Business contracts invoke the cross chain method of the cross chain management contract, which handles these transactions and assigns a unique ID for each transaction;
3. The consensus nodes then generates the merkle root of the block containing the cross chain transaction and merkle proof of that transaction;
4. The neo-relayers get the merkle root and proof through RPC and send them to the relay chain;
5. The relay chain verifies the validity of the merkle proof and broadcasts the cross chain information as events. The relayers of the target chain monitors these events, catches those are related to their chain and sends the information to the target chain;
6. The cross chain management contract of the target chain verifies the validity of the merkle proof of the relay chain. Successful verification means the cross chain information from the source chain is valid. The cross chain management contract of the target chain invokes corresponding business contract.

<div align=center><img src="resources/Relay2Neo.png"/></div>

The cross chain procedure from the target chain back to Neo chain:

1. Users create cross chain transactions in business contracts on the target chain;
2. The relayers of the target chain send the cross chain information to the relay chain and invoke the cross chain management contract of the relay chain;
3. The cross chain management contract verifies and stores the cross chain information, and builds a new merkle tree, adds the root into the block header of the relay chain, and generates the merkle proof for the cross chain information;
4. The relayers of Neo monitors the relay chain events, and sends the related block header and proof to Neo chain;
5. The cross chain management contract of Neo chain verifies the merkle proof and invoke corresponding business contract to finish the business logic.
