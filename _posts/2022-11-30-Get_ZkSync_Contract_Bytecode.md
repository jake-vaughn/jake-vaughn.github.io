---
title: Get ZkSync Contract Bytecode
date: 2022-11-30 8:00:00 +/-1111
author: Jake Vaughn
categories: [Guides]
tags: [zksync, bytecode, guide, EIP712, API, RPC, customData]
---

# Overview:
This will be a short post on how to obtain a ZkSync 2.0 smart contract's bytecode from the contract creation transaction. This post will also detail some of the differences between deploying a contract to zksync vs a regular EVM chain.

## ZkSync Contract Creation Transaction
One of the biggest differences that ZkSync has vs Ethereum is how smart contracts are deployed to the chain. On ZkSync smart contracts are compiled with a zksync specific compiler. The bytecode produced is then used to call the create function of the [ContractDeployer](https://v2-docs.zksync.io/dev/developer-guides/contracts/system-contracts.html#contractdeployer) along with the hash of the contract to be published, as well as its constructor arguments. The transaction made to the ContractDeployer contract is in the form of a [EIP712](https://v2-docs.zksync.io/api/api.html#eip712) transaction. The bytecode is supplied in the `factory_deps` field of the transaction and constructor arguments are supplied in the data field.
> Note: If you want more detail on Contract Deployment visit the [zkSync docs](https://v2-docs.zksync.io/dev/developer-guides/contracts/contracts.html#solidity-vyper-support)

## Deployment Transaction Receipt
ZkSync contracts can only be deployed using EIP712 transactions. Because of this, it should be easy enough to grab the customData.factoryDeps field of a transaction receipt to get the bytecode. 
```json
"ergsPerPubdata": "1212",
"customSignature": "0x...",
"paymasterParams": {
  "paymaster": "0x...",
  "paymasterInput": "0x..."
},
"factory_deps": ["0x..."]
```
The problem with this is that the current version of the zkSync RPC API does not supply EIP712 customData in transaction receipts. We can see that a `CREATE` call is made to the [ContractDeployer](https://v2-docs.zksync.io/dev/developer-guides/contracts/system-contracts.html#contractdeployer) and the constructor argument supplied in the data field, but we can not directly get the bytecode from the receipt.

To get the bytecode of the deployed contract we will need a workaround. The simplest way is to get the bytecode directly from the created contract's address. The contracts address can be obtained from the `contractAddress` field of the `ContractDeployed` event emitted by the [ContractDeployer](https://v2-docs.zksync.io/dev/developer-guides/contracts/system-contracts.html#contractdeployer)

```solidity
event ContractDeployed(address indexed deployerAddress, bytes32 indexed bytecodeHash, address indexed contractAddress);
```

The event data can be obtained from the `Logs` of the contract creation transaction receipt. With the contract's address all that is needed is to use your favorite tool to get the bytecode from the contract. The example below uses ethers.js.

```typescript
const bytecode = ethers.provider.getCode("Contract Address Here")
```

## Conclusion
Although it requires a small workaround obtaining contract bytecode from a zkSync Create call transaction receipt is possible. With the bytecode, it is possible to redeploy the same contract using the bytecode in the factory_deps field. This is because the constructor arguments are not a part of the bytecode as they are on Ethereum EVM bytecode. The constructor arguments are instead supplied in the data field of the transaction after the `CREATE` function Selector.

I hope this post brought some insight into zkSync contract deployment!