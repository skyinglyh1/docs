# [MainNet](README.md) | TestNet | [DevNet](README_DevNet.md) 

This's cross chain asset contract info on <strong>TESTNET</strong>, it's used to bind asset mapping in different chain, if the name ends with (s) on behalf of the asset is mapping.

## Decentralized Application TestNet Nodes
Chain | IP | Rpc Port
---|---|---
Poly | ```http://beta1.poly.network``` | 21336
Neo | ```http://seed9.ngd.network``` | 20332



## Ethereum

Type | Contract Hash | Desc
---|---|---
CCMP | 0xb600c8a2e8852832B75DB9Da1A3A1c173eAb28d8 | Cross Chain Manager Proxy contract hash 
ECCD | 0xA38366d552672556CE82426Da5031E2Ae0598dcD | Ethereum Cross Chain Data contract hash
ECCM | 0xcF9b45217192F70c1df42Ba460Fd644FDB248eA5 | Ethereum Cross Chain Manager contract hash
Lock Proxy | 0xE44dDbAb7a9Aa11b9D0cA8b686F9E0299DB735d1 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20 | 0x33fb970bE14548309bbbdcbFe6327865C6d71b70 | ERC20 template contract hash in Ethereum chain
OEP4x | 0xde5dBad6ab0FD0EBDEBf6B6cf4eB93D51207dc8f | OEP4x template contract hash in Ethereum chain
ONGx | 0x0A03605ad4E8381855cF7b650f9482002267ad33 | ONGx contract hash in Ethereum chain
ONTx | 0x8cE92808118Bc43c2B8635d3eb6b1b67cD2fB9a0 | ONTx contract hash in Ethereum chain
ETH | 0x0000000000000000000000000000000000000000 | The asset hash that we treat as the Ether asset
BTCx | 0x92705a16815A3d1AEC3cE9Cc273C5aa302961FcC | Btcx contract hash corresponding with unique btc redeem script
NEOx |  |

## Ontology
#### Please donot send from or to Ontology network during upgrade of Ontology testnet

Type | Contract Hash | Desc
---|---|---
Lock Proxy | B: 34a593c5ccfb8590e9a4bc7018529016aa4a55ad </br> L: ad554aaa1690521870bca4e99085fbccc593a534 | The bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx
ERC20x | B: e930755b130dccb25dc3cfee2b2e30d9370c1a75  </br> L: 751a0c37d9302e2beecfc35db2cc0d135b7530e9  | ERC20 template contract hash in Ontology chain
OEP4 | B: 969850e009b5e2a061694f3479ec8e44bc68bcd3 </br> L: d3bc68bc448eec79344f6961a0e2b509e0509896 | OEP4 template contract hash in Ontology chain
ONG | B: 0200000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000002 | ONG asset hash in Ontology chain
ONT | B: 0100000000000000000000000000000000000000 </br> L: 0000000000000000000000000000000000000002 | ONT asset hash in Ontology chain
ETHx | B: ec2e25c4c12371ca37b129477a23a152e62ff103 </br> L: 03f12fe652a1237a4729b137ca7123c1c4252eec | Ethx asset hash in Ontology chain
BTCx | B: 814d32455c21bfc25c33b75ccbfc34fe8e79bff1 </br> L: f1bf798efe34fccb5cb7335cc2bf215c45324d81 |  Btcx contract hash in Ontology chain
CNEOx | B:  </br> L:  |  CNEOx contract hash in Ontology chain
CGASx | B:  </br> L:  | CGasx contract hash in Ontology chain

## Neo

Type | Contract Hash | Desc
---|---|---
CNEO | B: 0xc074a05e9dcf0141cbe6b4b3475dd67baf4dcb60 </br> L: 60cb4daf7bd65d47b3b4e6cb4101cf9d5ea074c0 | 
CGAS | B: 0x74f2dc36a68fdc4682034178eb2220729231db76 </br> L: 76db3192722022eb7841038246dc8fa636dcf274 |
ETHx | B: 0xd7b32de37ad906df80805c2419ff5560d20f9cbf </br> L: bf9c0fd26055ff19245c8080df06d97ae32db3d7 | Eth asset hash in Neo chain
BTCx | B: 0x3ee29d5cc82771e91383f9ba09c6f5c5878f3f24 </br> L: 243f8f87c5f5c609baf98313e97127c85c9de23e | BTC asset hash in Neo chain
ONTx | B: 0x5a9222225f1bdb135123b74354c7248200c440aa </br> L: aa40c4008224c75443b7235113db1b5f2222925a | ONT asset hash in Neo chain
ATOMx | B: 0xb25b51d684f7945f7aab43496cb0e87138abdb35 </br> L: 35dbab3871e8b06c4943ab7a5f94f784d6515bb2 | ATOM asset hash in Neo chain
CCMC | B: 0x80cd0c6fb005da87b78c54dd03c65ef1447195fa </br> L: fa957144f15ec603dd548cb787da05b06f0ccd80 | Cross Chain Manager Contract
Lock Proxy | B: 0xd9b91c5a23e902f39d80593ab5233dfa16a6365c </br> L: 5c36a616fa3d23b53a59809df302e9235a1cb9d9 | The lock proxy bridge contract hash for asset not implementing the "lock" and "unlock" logic to do cross chain tx


## Note 
`B` means big-endian, we can search the contract transaction history in corresponding explorer.

`L` means little-endian, we usually use it as the asset hash input when we do binding asset hash operation.


## Cosmos

Type | Denom (coin name) | Asset/Contract Hash | Desc
:-:|:-:|:-:|:-:
Lock Proxy | | f71b55ef55cedc91fd007f7a9ba386ec978f3aa8 |
ERC20 | erc20x | 657263323078 |
OEP4x | oep4x | 6f65703478 |
ONGx | ongx | 6f6e6778 |
ONTx | ontx | 6f6e7478 |
ETHx | ethx | 65746878 |
BTCx | btcx | 62746378 |
NEOx | neox | 6e656f78 | not including currently
GASx | gasx | 67617378 | not including currently
ATOM | stake | 7374616b65 | not including currently

## Router And ChainId
Type | Router Number | ChainId
:-:|:-:|:-:
Bitcoin | 1 | 1
Ethereum | 2 | 2
Ontology | 3 | 3
NEO | 4 | 4
Cosmos-gaia | 5 | 5
Switcheo | 5 | release soon
