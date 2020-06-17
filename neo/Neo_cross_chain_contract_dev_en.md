# Neo Cross Chain Contract Development

English | [中文](Neo_cross_chain_contract_dev_cn.md)

The Neo multi-chain ecosystem is based on cross chain contracts which have the capability to interact with two or more chains. For example: an NEP5 asset released on Neo can be transferred to Ethereum as a corresponding ERC20 asset, or a dApp contract which has some of its business logic taking place on Neo and other carried out on Ethereum.

## Cross chain contract overview

The cross chain system actually consists of multiple smart contracts. For example, if a dApp needs to achieve cross chain business logic on Chain A and Chain B, the dApp developers need to deploy smart contract A on Chain A and smart contract B on Chain B. Both smart contracts have part of the dApp business logic.

Cross chain contract development is composed of two parts: the business logic part and the cross chain supporting part.

The business logic part is nothing special and should be developed as a conventional smart contract application. It runs its business logic code on the chain where it is deployed.

The cross chain supporting part is mainly about the cross chain management contract (CCMC). When the business logic is finished on Chain A and the business logic on Chain B is required to be executed, then those interfaces in the CCMC are used.

## Cross chain interface

Cross chain contract developers only need to focus on one interface which is the `CrossChain` method in the CCMC. This method stores the business logic execution result on Chain A into a merkle tree. A miner then generates the merkle proof of this cross chain transaction and sends it to the CCMC on Chain B. This CCMC verifies the merkle proof and invokes the corresponding contract methods on Chain B based on the parameters passed by.

## Cross chain contract development sample (for new released NEP5 asset)

A developer intends to release a new asset on Chain A and Chain B and wants the asset on both chains are interchangeable which means the asset can be used and transferred between two chains conveniently.

In order to achieve the interoperability of the asset on both chains, two additional methods `Lock` and `Unlock` are added to the standard NEP5 contracts. The user invokes the `Lock` method on Chain A and the contract locks the cross chain amount of the asset. At the same time, the method then invokes the CCMC on Chain A which invokes the `Unlock` method of the contract on Chain B to release the corresponding amount of asset back to the user, and vice versa.

Sample `Lock` method implementation in C#：

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

**Parameters setup：**  

- fromAddress: the user account address starting the cross chain operation where mining fee is deducted
- toChainId: the target chain ID
- toAddress: the user address on target chain
- amount: cross chain asset amount

In the above code, the method first locks the specified amount of the user's asset on Chain A, reduces the asset total supply (since that amount of asset is locked and not circulating), and then invokes the `CrossChain` of the Neo CCMC which takes four parameters:

```C#
public static bool CrossChain(BigInteger toChainID, byte[] toChainAddress, byte[] functionName, byte[] args)
```

**Parameters setup：**  

- toChainId: the target chain ID
- toChainAddress: the target chain contract address whose `Unlock` method needs to be invoked
- functionName: the method name of the target chain contract, in this case it's "unlock"
- args: the serialized parameters for the target chain contract

In the above code of `Lock` method, the method name parameter passed to the CCMC is "unlock", so here is the `Unlock` method:

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

    // parse the args bytes constructed in the source chain proxy contract, passed by multi-chain
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

**Parameters setup：**  

- inputBytes: serialized parameters for business logic
- fromContract: the contract on the source chain which invokes the cross chain operation
- fromChainId: the source chain ID
- caller: the caller of this method, no need to pass by, the contract adds this parameter automatically

This method first checks if it's invoked by the CCMC, then checks the "from" contract parameter is same as what's stored in its storage, deserializes the parameters to run business logic and finally unlocks the user's asset on Chain B and increases the asset total supply.

## Cross chain contract development sample (for existing NEP5 asset)

Existing NEP5 assets cannot be modified and implement the `Lock` and `Unlock` methods. So an additional proxy contract is needed to add the cross chain functionality to those NEP5 contracts. Also, off the shelf proxy contracts are ready to be used to avoid duplicate development.

The proxy contract must implement the following methods:

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

**Parameters setup：**  

- toChainId: the target chain ID
- targetProxyHash: the proxy contract hash on target chain

This method binds the target chain proxy contract hash with chain ID, and acts as the intermerdiary before NEP5 assets are transferred to the token contract on the target chain which is deployed in advance.

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

**Parameters setup：**  

- fromAssetHash: the asset contract hash on the source chain
- toChainId: the target chain ID
- toAssetHash: the asset contract hash on the target chain
- initialAmount: initial locked amount

This method binds the NEP5 token contract hash with the corresponding asset hash on the target chain, such as the USDT contract hash, and sets the cross chain amount limit.

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

**Parameters setup：**  

- fromAssetHash: the NEP5 contract hash on the source chain
- fromAddress: the user account address starting the cross chain operation where mining fee is deducted
- toChainId: the target chain ID
- toAddress: the user account address on the target chain
- amount: the cross chain asset amount

This method locks the NEP5 asset in the contract address, and invokes the `CrossChain` method of the CCMC to start a cross chain operation.

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

**Parameters setup：**  

- inputBytes: serialized parameters for business logic
- fromProxyContract: the proxy hash starting the cross chain operation on the source chain
- fromChainId: the source chain ID
- caller: the caller of this method, no need to pass by, the contract adds this parameter automatically

This method is only called by the CCMC and returns the previously locked NEP5 asset to the user.

In conclusion, the proxy contract is mainly used to bind, lock and unlock the NEP5 asset. "lock" means users transfer their asset to the proxy contract address and "unlock" does the opposite. Developers can either deploy their own proxy contracts, or use the existing ones. Multiple NEP5 assets can share a proxy contract.
