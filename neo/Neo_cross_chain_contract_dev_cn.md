# Neo跨链合约开发

[English](Neo_cross_chain_contract_dev_en.md) | 中文

Neo多链生态是基于跨链合约构建的，所谓跨链合约就是合约的逻辑贯穿两条甚至多条链，例如：

在Neo上发布的某种NEP5资产可以和Ethereum上的某种ERC20资产相对应并互相流转。

## 跨链合约概述

跨链合约实际上是由多本合约构成的，例如dApp开发者需要在链A和链B上实现跨链业务，此时开发者需要分别在链A和链B上部署智能合约（智能合约A和智能合约B）。

跨链合约的开发总的来说可以分为两部分，业务部分和跨链部分：

业务部分是指在某条链中运行的逻辑代码，按照标准的智能合约开发方式开发，完成合约在该链中的业务。

跨链部分主要是指跨链管理合约，当需要跨链时，如链A的逻辑执行完，接下来需要执行链B的逻辑，那么就需要用到跨链管理合约中的跨链接口。

## 跨链接口

合约的跨链对开发者来说只需要关注一个跨链接口，也就是跨链管理合约的`CrossChain`接口，该接口将链A上已经执行的业务存入梅克尔树，会有矿工生成该跨链交易的梅克尔证明，并将其提交到链B的跨链管理合约中，该跨链管理合约会验证梅克尔证明，并按照参数调用智能合约B中对应的方法。

## 跨链合约开发示例（新发行的NEP5资产）

某开发者想在链A和链B上发行资产，但是希望链A和链B上的资产互通，也就是说其需要发行一种资产，这种资产能够在链A和链B上同时使用，并且可以在链A和链B上自由转移。

为了使资产可以在链A和链B之间互相转移，在NEP5标准接口的基础上，需要增加`Lock`和`Unlock`接口，用户在链A中调用`Lock`接口将资产锁定在智能合约A中，该接口同时调用跨链管理合约实现跨链调用智能合约B中的`Unlock`接口，并在链B中将智能合约B中的资产释放给该用户。反之亦然。

`Lock`接口实现：

```C#
public static bool Lock(byte[] fromAddress, BigInteger toChainId, byte[] toAddress, BigInteger amount)
{
    // check parameters
    if (!IsAddress(fromAddress))
    {
        Runtime.Notify("The parameter fromAddress SHOULD be a legal address.");
        return false;
    }
    if (toAddress.Length == 0)
    {
        Runtime.Notify("The parameter toAddress SHOULD not be empty.");
        return false;
    }
    if (amount < 0)
    {
        Runtime.Notify("The parameter amount SHOULD not be less than 0.");
        return false;
    }
    // more checks
    if (!Runtime.CheckWitness(fromAddress))
    {
        Runtime.Notify("Authorization failed.");
        return false;
    }
    if (IsPaused())
    {
        Runtime.Notify("The contract is paused.");
        return false;
    }

    // lock asset
    StorageMap asset = Storage.CurrentContext.CreateMap(nameof(asset));
    var balance = asset.Get(fromAddress).AsBigInteger();
    if (balance < amount)
    {
        Runtime.Notify("Not enough balance to lock.");
        return false;
    }
    StorageMap contract = Storage.CurrentContext.CreateMap(nameof(contract));
    var totalSupply = contract.Get("totalSupply").AsBigInteger();
    if (totalSupply < amount)
    {
        Runtime.Notify("Not enough supply to lock.");
        return false;
    }
    asset.Put(fromAddress, balance - amount);
    contract.Put("totalSupply", totalSupply - amount);

    // construct args for the corresponding asset contract on target chain
    var inputBytes = SerializeArgs(toAddress, amount);
    var toContract = GetContractAddress(toChainId);
    // constrct params for CCMC
    var param = new object[] { toChainId, toContract, "unlock", inputBytes };

    // dynamic call CCMC
    var ccmc = (DynCall)CCMCScriptHash.ToDelegate();
    var success = (bool)ccmc("CrossChain", param);
    if (!success)
    {
        Runtime.Notify("Failed to call CCMC.");
        return false;
    }

    LockEvent(toChainId, fromAddress, toAddress, amount);
    return true;
}
```

**参数设置：**  

- fromAddress: 跨链调用发起地址，矿工费从该地址扣除
- toChainId: 目标链的链ID
- toAddress: 目标链上的用户地址
- amount: 跨链资产数量

在上面的代码中可以看到该接口先冻结用户在链A上的需要跨链的数量的资产，并缩减该资产总量（因为该部分资产将锁定而不再流通），然后调用Neo跨链管理合约的`CrossChain`接口，该方法接受四个参数：

```C#
public static bool CrossChain(BigInteger toChainID, byte[] toChainAddress, byte[] functionName, byte[] args)
```

**参数设置：**  

- toChainId: 目标链的链ID
- toChainAddress: 跨链需要调用的目标链上的合约
- functionName: 跨链资产数量
- args: 序列化之后的目标合约输入参数

在`Lock`接口的代码中可以看到该方法调用了目标合约的`Unlock`接口，`Unlock`接口实现：

```C#
private static bool Unlock(byte[] inputBytes, byte[] fromContract, BigInteger fromChainId, byte[] caller)
{
    //only allowed to be called by CCMC
    if (caller.AsBigInteger() != CCMCScriptHash.AsBigInteger())
    {
        Runtime.Notify("Only allowed to be called by CCMC");
        return false;
    }

    byte[] storedContract = GetContractAddress(fromChainId);

    // check the fromContract is stored, so we can trust it
    if (fromContract.AsBigInteger() != storedContract.AsBigInteger())
    {
        Runtime.Notify(fromContract);
        Runtime.Notify(fromChainId);
        Runtime.Notify(storedContract);
        Runtime.Notify("From contract address not found.");
        return false;
    }

    // parse the args bytes constructed in source chain proxy contract, passed by multi-chain
    object[] results = DeserializeArgs(inputBytes);
    var toAddress = (byte[])results[0];
    var amount = (BigInteger)results[1];
    if (!IsAddress(toAddress))
    {
        Runtime.Notify("ToChain Account address SHOULD be a legal address.");
        return false;
    }
    if (amount < 0)
    {
        Runtime.Notify("ToChain Amount SHOULD not be less than 0.");
        return false;
    }

    // unlock asset
    StorageMap asset = Storage.CurrentContext.CreateMap(nameof(asset));
    var balance = asset.Get(toAddress).AsBigInteger();
    StorageMap contract = Storage.CurrentContext.CreateMap(nameof(contract));
    var totalSupply = contract.Get("totalSupply").AsBigInteger();
    asset.Put(toAddress, balance + amount);
    contract.Put("totalSupply", totalSupply + amount);

    UnlockEvent(toAddress, amount);
    return true;
}
```

**参数设置：**  

- inputBytes: 序列化之后的合约参数
- fromContract: 发起跨链的源链上的合约
- fromChainId: 源链ID
- caller: 该方法的调用者，不用传入，合约自动补充该参数

该接口会先校验调用是不是来自跨链管理合约，然后校验源链上的合约地址是否可信，接着反序列化出所需参数，最后解锁链B上的资产给用户及增加该资产总量。

## 跨链合约开发示例（已发行的NEP5资产）

对于已经存在的NEP5资产来说，其合约代码是不能修改并添加`Lock`、`Unlock`方法的，因此需要额外实现一个代理合约，相当于原先NEP5合约的补充功能，实现了跨链协议中的主要接口，也可以使用现有的代理合约，实现跨链，避免重复开发。

以下为代理合约必须实现的接口：

### BindProxyHash

```C#
public static bool BindProxyHash(BigInteger toChainId, byte[] targetProxyHash)
{
    if (!Runtime.CheckWitness(Operator)) return false;
    StorageMap proxyHash = Storage.CurrentContext.CreateMap(nameof(proxyHash));
    proxyHash.Put(toChainId.AsByteArray(), targetProxyHash);
    return true;
}
```

**参数设置：**  

- toChainId: 目标链的链ID
- targetProxyHash: 跨链需要调用的目标链上的代理合约

该接口绑定目标链上的代理合约hash，或者实现了跨链接口的合约hash，在NEP5朝其他链跨链的时候会将该目标链代理合约作为中转站，进而将NEP5转到目标链的映射合约，当然在跨链开始前应该预先部署好映射合约。

### BindAssetHash

```C#
public static bool BindAssetHash(byte[] fromAssetHash, BigInteger toChainId, byte[] toAssetHash, BigInteger initialAmount)
{
    if (!Runtime.CheckWitness(Operator)) return false;
    StorageMap assetHash = Storage.CurrentContext.CreateMap(nameof(assetHash));
    assetHash.Put(fromAssetHash.Concat(toChainId.AsByteArray()), toAssetHash);

    if (GetAssetBalance(fromAssetHash) != initialAmount)
    {
        Runtime.Notify("Initial amount incorrect.");
        return false;
    }

    StorageMap lockedAmount = Storage.CurrentContext.CreateMap(nameof(lockedAmount));
    lockedAmount.Put(fromAssetHash, initialAmount);
    BindAssetHashEvent(fromAssetHash, toChainId, toAssetHash, initialAmount);
    return true;
}
```

**参数设置：**  

- fromAssetHash: 源链上的资产合约hash
- toChainId: 目标链的链ID
- toAssetHash: 目标链上的资产合约hash
- initialAmount: 该资产初始锁定数量

该接口绑定NEP5代币合约地址和目标链映射合约，比如USDT地址，并设置初始锁定数量。

### Lock

```C#
public static bool Lock(byte[] fromAssetHash, byte[] fromAddress, BigInteger toChainId, byte[] toAddress, BigInteger amount)
{
    // check parameters
    if (fromAssetHash.Length != 20)
    {
        Runtime.Notify("The parameter fromAssetHash SHOULD be 20-byte long.");
        return false;
    }
    if (fromAddress.Length != 20)
    {
        Runtime.Notify("The parameter fromAddress SHOULD be 20-byte long.");
        return false;
    }
    if (toAddress.Length == 0)
    {
        Runtime.Notify("The parameter toAddress SHOULD not be empty.");
        return false;
    }
    if (amount < 0)
    {
        Runtime.Notify("The parameter amount SHOULD not be less than 0.");
        return false;
    }

    // get the corresbonding asset on target chain
    var toAssetHash = GetAssetHash(fromAssetHash, toChainId);
    if (toAssetHash.Length == 0)
    {
        Runtime.Notify("Target chain asset hash not found.");
        return false;
    }

    // get the proxy contract on target chain
    var toContract = GetProxyHash(toChainId);
    if (toContract.Length == 0)
    {
        Runtime.Notify("Target chain proxy contract not found.");
        return false;
    }

    // transfer asset from fromAddress to proxy contract address, use dynamic call to call nep5 token's contract "transfer"
    byte[] currentHash = ExecutionEngine.ExecutingScriptHash; // this proxy contract hash
    var nep5Contract = (DynCall)fromAssetHash.ToDelegate();
    bool success = (bool)nep5Contract("transfer", new object[] { fromAddress, currentHash, amount });
    if (!success)
    {
        Runtime.Notify("Failed to transfer NEP5 token to proxy contract.");
        return false;
    }

    // construct args for proxy contract on target chain
    var inputBytes = SerializeArgs(toAssetHash, toAddress, amount);
    // constrct params for CCMC 
    var param = new object[] { toChainId, toContract, "unlock", inputBytes };
    // dynamic call CCMC
    var ccmc = (DynCall)CCMCScriptHash.ToDelegate();
    success = (bool)ccmc("CrossChain", param);
    if (!success)
    {
        Runtime.Notify("Failed to call CCMC.");
        return false;
    }

    // update locked amount
    StorageMap lockedAmount = Storage.CurrentContext.CreateMap(nameof(lockedAmount));
    BigInteger old = lockedAmount.Get(fromAssetHash).ToBigInteger();
    lockedAmount.Put(fromAssetHash, old + amount);

    LockEvent(fromAssetHash, toChainId, toAssetHash, fromAddress, toAddress, amount);

    return true;
}
```

**参数设置：**  

- fromAssetHash: 源链上的NEP5资产合约hash
- fromAddress: 跨链调用发起地址，矿工费从该地址扣除
- toChainId: 目标链的链ID
- toAddress: 目标链上的用户地址
- amount: 跨链资产数量

该接口会把NEP5资产锁定到代理合约账户中，并调用跨链管理合约的`CrossChain`接口从而发出跨链请求。

### Unlock

```C#
private static bool Unlock(byte[] inputBytes, byte[] fromProxyContract, BigInteger fromChainId, byte[] caller)
{
    //only allowed to be called by CCMC
    if (caller.AsBigInteger() != CCMCScriptHash.AsBigInteger())
    {
        Runtime.Notify("Only allowed to be called by CCMC");
        return false;
    }

    byte[] storedProxy = GetProxyHash(fromChainId);

    // check the fromContract is stored, so we can trust it
    if (fromProxyContract.AsBigInteger() != storedProxy.AsBigInteger())
    {
        Runtime.Notify(fromProxyContract);
        Runtime.Notify(fromChainId);
        Runtime.Notify(storedProxy);
        Runtime.Notify("From proxy contract not found.");
        return false;
    }

    // parse the args bytes constructed in source chain proxy contract, passed by multi-chain
    object[] results = DeserializeArgs(inputBytes);
    var assetHash = (byte[])results[0];
    var toAddress = (byte[])results[1];
    var amount = (BigInteger)results[2];
    if (assetHash.Length != 20)
    {
        Runtime.Notify("ToChain Asset script hash SHOULD be 20-byte long.");
        return false;
    }
    if (toAddress.Length != 20)
    {
        Runtime.Notify("ToChain Account address SHOULD be 20-byte long.");
        return false;
    }
    if (amount < 0)
    {
        Runtime.Notify("ToChain Amount SHOULD not be less than 0.");
        return false;
    }

    // transfer asset from proxy contract to toAddress
    byte[] currentHash = ExecutionEngine.ExecutingScriptHash; // this proxy contract hash
    var nep5Contract = (DynCall)assetHash.ToDelegate();
    bool success = (bool)nep5Contract("transfer", new object[] { currentHash, toAddress, amount });
    if (!success)
    {
        Runtime.Notify("Failed to transfer NEP5 token to toAddress.");
        return false;
    }

    // update locked amount
    StorageMap lockedAmount = Storage.CurrentContext.CreateMap(nameof(lockedAmount));
    BigInteger old = lockedAmount.Get(assetHash).ToBigInteger();
    lockedAmount.Put(assetHash, old - amount);

    UnlockEvent(assetHash, toAddress, amount);

    return true;
}
```

**参数设置：**  

- inputBytes: 序列化之后的合约参数
- fromProxyContract: 发起跨链的源链上的代理合约hash
- fromChainId: 源链ID
- caller: 该方法的调用者，不用传入，合约自动补充该参数

该接口会被跨链管理合约调用，为用户释放原先锁定的NEP5资产，即从其他链返回的NEP5代币。

所以代理合约主要实现的是NEP5的注册、锁定与解锁，锁定就是用户将NEP5转到合约中，解锁只有跨链管理合约可以调用。代理合约可以自己部署，也可以使用现有的合约，多个NEP5资产可以使用一本代理合约。
