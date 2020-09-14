# MainNet | [TestNet](README_TestNet.md) | [DevNet](README_DevNet.md) 

This's cross chain asset contract info ONLY in <strong>MAINNET</strong> mode, it's used to bind asset mapping in different chain, if the name ends with (s) on behalf of the asset is mapping.


## Decentralized Application MainNet Nodes
Chain | IP | Rpc Port
---|---|---
Poly | ```http://seed1.poly.network``` | 20336
Neo | ```http://seed9.ngd.network``` | 11332



## Btc


## Ethereum

Type | Contract Hash | Desc
---|---|---
CCMP | 0x5a51e2ebf8d136926b9ca7b59b60464e7c44d2eb | Cross Chain Manager Proxy contract hash 
ECCD | 0xcf2afe102057ba5c16f899271045a0a37fcb10f2 | Ethereum Cross Chain Data contract hash
ECCM | 0x838bf9e95cb12dd76a54c9f9d2e3082eaf928270 | Ethereum Cross Chain Manager contract hash
Lock Proxy| 0x250e76987d838a75310c34bf422ea9f1AC4Cc906 | Ethereum Asset Proxy contract hash

## Ontology

Type | Contract Hash | Desc
---|---|---
Lock Proxy | B: 0x86b4ab5d99037113867247a1e68f70e348c07597 </br> L: 9775c048e3708fe6a1477286137103995dabb486 | ONT with 9 decimal contract hash 
pUSDC | B: 0x061a07cd393aac289b8ecfda2c3784b637a2fb33 </br> L: 33fba237b684372cdacf8e9b28ac3a39cd071a06 | Ontology wrapped USDT (Ethereum) Cross Chain asset contract hash
pWBTC | B: 0x8037dd7161401417d3571b92b86846d34309129a </br> L: 9a120943d34668b8921b57d31714406171dd3780 | Ontology wrapped WBTC (Ethereum) Cross Chain asset contract hash
prenBTC | B: 0x46c3051c553aaeb3724ea69336ec483f39fa91b1 </br> L: b191fa393f48ec3693a64e72b3ae3a551c05c346 | Ontology wrapped WBTC (Ethereum) Cross Chain asset contract hash


## Neo
Type | Contract Hash | Desc
---|---|---
ETHx | B: 0x17c76859c11bc14da5b3e9c88fa695513442c606 </br> L: 06c642345195a68fc8e9b3a54dc11bc15968c717 | Eth asset hash in Neo chain
ONTx | B: 0x271e1e4616158c7440ffd1d5ca51c0c12c792833 </br> L: 3328792cc1c051cad5d1ff40748c1516461e1e27 | ONT asset hash in Neo chain
CCMC | B: 0x82a3401fb9a60db42c6fa2ea2b6d62e872d6257f </br> L: 7f25d672e8626d2beaa26f2cb40da6b91f40a382 | Cross Chain Manager Contract
Lock Proxy | B: 0x82e749792af617d66593ef14eb34331ed58a0fcb </br> L: cb0f8ad51e3334eb14ef9365d617f62a7949e782 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx

## Note 
`B` means big-endian, we can search the contract transaction history in corresponding explorer.

`L` means little-endian, we usually use it as the asset hash input when we do binding asset hash operation.

## Switcheo

Type | Denom (coin name) | Asset/Contract Hash | Desc
:-:|:-:|:-:|:-:


## Router And ChainId
Type | Router Number | ChainId
:-:|:-:|:-:
Bitcoin | 1 | 1
Ethereum | 2 | 2
Ontology | 3 | 3
NEO | 4 | 4
Switcheo | 5 | 5
