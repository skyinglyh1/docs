# 如何将NEP5资产跨到其他链上

[English](How_to_cross_NEP5_asset_en.md) | 中文

## 收集资产在链上对应的合约地址

以某个NEP5资产为例，这种资产在链上的合约地址为：

-Neo链: 4da43995ee75be3931fd3abae06a3e6447d2febf (小端序)

-Ethereum链: 0x86B7Be11D77043E5aDe3858f2F694E4f89d4a941

## 执行跨链操作

### 集成跨链功能的NEP5资产

如果该资产集成了跨链的Lock和Unlock接口，则可以直接调用该资产的Lock接口进行跨链资产转移：

```C#
public static bool Lock(byte[] fromAddress, BigInteger toChainId, byte[] toAddress, BigInteger amount)
```

**参数设置：**  

- fromAddress: 5392ebf880d9a7119b17b2f1adaebc5f71c28e75 (Neo链地址APPmjituYcgfNxjuQDy9vP73R2PmhFsYJR的script hash，小端序)  
- toChainId: 2 (Ethereum链的编码)  
- toAddress: 0x0B24aBDd39185055311aaa27082F9dEb294A7255 (Ethereum链地址)  
- amount: 10000 (跨链数量)

### 未集成跨链功能的NEP5资产

如果该资产未集成跨链的Lock和Unlock接口，则需要调用该资产对应的代理合约的Lock接口：

```C#
public static bool Lock(byte[] fromAssetHash, byte[] fromAddress, BigInteger toChainId, byte[] toAddress, BigInteger amount)
```

**参数设置：**  

- fromAssetHash: 4da43995ee75be3931fd3abae06a3e6447d2febf (NEP5资产的script hash，小端序)  
- fromAddress: 5392ebf880d9a7119b17b2f1adaebc5f71c28e75 (Neo链地址APPmjituYcgfNxjuQDy9vP73R2PmhFsYJR的script hash，小端序)  
- toChainId: 2 (Ethereum链的编码)  
- toAddress: 0x0B24aBDd39185055311aaa27082F9dEb294A7255 (Ethereum链地址)  
- amount: 10000 (跨链数量)
