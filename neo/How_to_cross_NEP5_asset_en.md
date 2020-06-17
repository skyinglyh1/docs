# How to Transfer NEP5 Assets to Other Chains

English | [中文](./How_to_cross_NEP5_asset_cn.md)

## Collect the corresponding contract addresses for the asset on different chains

For example, an NEP5 asset has the following contract addresses:

-Neo chain: 4da43995ee75be3931fd3abae06a3e6447d2febf (little endian)

-Ethereum chain: 0x86B7Be11D77043E5aDe3858f2F694E4f89d4a941

## Start cross chain operation

### NEP5 assets with cross chain features

If the asset has embedded the "Lock" and "Unlock" API methods, cross chain transactions can be carried out by invoking the "Lock" method directly:

```C#
public static bool Lock(byte[] fromAddress, BigInteger toChainId, byte[] toAddress, BigInteger amount)
```

**Parameters setup：**  

- fromAddress: 5392ebf880d9a7119b17b2f1adaebc5f71c28e75 (script hash of a Neo chain address APPmjituYcgfNxjuQDy9vP73R2PmhFsYJR in little endian)  
- toChainId: 2 (Ethereum chain id)  
- toAddress: 0x0B24aBDd39185055311aaa27082F9dEb294A7255 (an address on Ethereum chain)  
- amount: 10000 (cross chain amount)

### NEP5 assets without cross chain features

If the asset doesn't have the "Lock" and "Unlock" API methods natively, then the "Lock" method of its corresponding proxy contract is invoked to create cross chain transactions:

```C#
public static bool Lock(byte[] fromAssetHash, byte[] fromAddress, BigInteger toChainId, byte[] toAddress, BigInteger amount)
```

**Parameters setup：**  

- fromAssetHash: 4da43995ee75be3931fd3abae06a3e6447d2febf (script hash of the NEP5 asset in little endian)  
- fromAddress: 5392ebf880d9a7119b17b2f1adaebc5f71c28e75 (script hash of a Neo chain address  APPmjituYcgfNxjuQDy9vP73R2PmhFsYJR in little endian)  
- toChainId: 2 (Ethereum chain id)  
- toAddress: 0x0B24aBDd39185055311aaa27082F9dEb294A7255 (an address on Ethereum chain)  
- amount: 10000 (cross chain amount)
