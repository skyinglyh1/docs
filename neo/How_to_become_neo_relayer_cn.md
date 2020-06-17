# neo-relayer

[English](./How_to_become_neo_relayer_en.md) | 中文

## 架构

一方面，neo-relayer实现了对Neo网络的监听，会持续扫描每个Neo区块并将切换了下一轮共识地址的关键区块头同步到中继链上，同时能识别并转发跨链交易，向中继链提交区块的状态根和交易的梅克尔证明。  
另一方面，neo-relayer也实现了对中继链的监听，会持续扫描每个中继链区块并将切换了共识地址的关键区块头同步到Neo链上，中继链收到其他链的跨链信息会构造到Neo链的返回交易，neo-relayer会将这些交易所在的中继链区块头同步到Neo链上因为其中包含了状态根，同时转发这些交易的梅克尔证明及跨链信息。  
只要存在一个neo-relayer在工作，那么整个Neo跨链生态就可以持续运作。

## 准备

1. 申请中继链钱包  
申请一个中继链的钱包，如果有本体的钱包可以直接使用，二者钱包是通用的。

2. 注册Relayer  
然后向中继链注册自己的地址为Relayer。

## 启动Relayer

1. 下载neo-relayer的可执行文件，解压至特定位置
2. 更改配置信息，例如：

```json
{
  "RelayJsonRpcUrl": "http://138.91.6.125:40336", // 中继链rpc地址
  "RelayChainID": 0, // 中继链编码
  "WalletFile": "./wallet.dat", // 中继链钱包
  "NeoWalletFile": "TBD", // 待定，后续版本可能采用钱包文件以替代钱包wif
  "NeoWalletWIF": "L3Hab7wL43SbWLnkfnVCp6dT99xzfB4qLZxeR9dFgcWWPirwKyXp", // Neo钱包wif
  "NeoJsonRpcUrl": "http://47.89.240.111:11332", // Neo链rpc地址
  "NeoChainID": 4, // Neo链编码
  "NeoCCMC": "b0d4f20da68a6007d4fb7eac374b5566a5b0e229", // Neo链的跨链管理合约
  "ScanInterval": 2, // 扫描中继链的时间间隔，单位为秒
  "GasPrice": 0,
  "GasLimit": 200000
}
```

3. 输入./main --loglevel 0 --cliconfig “path” 执行Relayer可执行文件以运行neo-relayer
