# neo-relayer

English | [中文](./How_to_become_neo_relayer_CN.md)

## Structure

On one hand, neo-relayer monitors Neo network continuously, scans each Neo block and synchronizes those key block headers which contains different "NextConsensus" field to the relay chain. At the same time, neo-relayer can recognize cross-chain transactions, and provides the "StateRoot" of the block containing those transactions together with the "MerkleProof" of those transactions to the relay chain.  
On the other hand, neo-relayer also monitors the relay chain, scans each relay chain block and synchronizes those key block headers which has changed next consensus addresses to Neo chain. Moreover, the relay chain creates cross-chain transactions to Neo chain when it receives cross-chain transactions from other chains. Then neo-relayer relays the "StateRoot" of the block containing those transactions together with the "MerkleProof" of those transactions to Neo chain.  
The entire Neo cross-chain ecosystem can continue to function normally even with just one active neo-relayer.

## Setup

1. Create a relay chain wallet  
Create a wallet to be used on the relay chain. If you have an existing Ontology wallet it can be used on the relay chain. The two wallets are mutually interchangeable.

2. Register the neo-relayer  
Register your address as the **relayer** on the relay chain.

## Start Up

1. Download the executable file of the relayer and extract it to a specific directory.

2. Modify the configuration settings. Here is a sample configuration:

```json
{
  "RelayJsonRpcUrl": "http://138.91.6.125:40336", // the rpc address of the relay chain
  "RelayChainID": 0, // the relay chain id
  "WalletFile": "./wallet.dat", // the wallet used on the relay chain
  "NeoWalletFile": "TBD", // to be determined, future versions may replace WIF with wallet file of Neo network
  "NeoWalletWIF": "L3Hab7wL43SbWLnkfnVCp6dT99xzfB4qLZxeR9dFgcWWPirwKyXp", // private key in WIF format
  "NeoJsonRpcUrl": "http://47.89.240.111:11332", // the rpc address of the Neo chain
  "NeoChainID": 4, // Neo chain id
  "NeoCCMC": "b0d4f20da68a6007d4fb7eac374b5566a5b0e229", // cross chain management contract of Neo chain
  "ScanInterval": 2, // the time interval to scan the relay chain, in seconds
  "GasPrice": 0,
  "GasLimit": 200000
}
```

3. Use the following command with the configuration file path to enable the relayer:

```bash
./main --loglevel 0 --cliconfig "path"
```
