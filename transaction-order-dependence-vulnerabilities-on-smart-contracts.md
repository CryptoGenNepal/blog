---
title: "Transaction Order Dependence Vulnerabilities on Smart Contracts"
date: 2022-10-04T00:00:00+05:45
image: "images/blog/default.png"
author: "Aayushman Thapa Magar"
type: "regular"
description: "This is meta description"
draft: false
---

{{< toc >}}

## Foreword

This work is first in the series of articles on vulnerabilities that smart contracts are susceptible to. You can find my articles in this topic in the following links:

1. [Transaction Order Dependence](/blog/transaction-order-dependence-vulnerabilities-on-smart-contracts/)
2. [Re-entrancy](/blog/re-entrancy-vulnerabilities-in-smart-contracts/)
3. [Signature Malleability](/blog/signature-malleability-vulnerabilities-in-smart-contracts/)
4. [Arithmetic Vulnerabilities](/blog/arithmetic-vulnerabilities-in-smart-contracts/)

The primary purpose for these articles is **not** to educate others, but to test the depth of my own understanding. However, I do hope that my efforts provide even a little value to you dear readers. Please let me know if any questions arise or any mistakes are made so we can learn and grow together.

## Table of Contents

1. Transaction order dependence overview
2. Security Risk
3. Identification techniques
4. Mitigation measures
5. References

## Transaction Order Dependence Overview

### Introduction

Transaction order dependence (TOD) vulnerabilities are a type of race condition flaw that can be present in smart contracts. These are also known as _Front-Running_ vulnerabilities. Such vulnerabilities can be present because of how transactions are handled on the Ethereum blockchain.

### Ethereum mempool

When a user creates a transaction, it is **not** added to the ledger immediately. The transaction is broadcasted throughout the network and stored in the mempool of nodes, where it is held temporarily. Mempool (portmanteau of memory and pool) can be thought of as a waiting area for unprocessed/pending transactions i.e. transactions that have not been added to a block yet. Mempool is a necessary mechanism in Ethereum as once user transactions are added to the ledger, it cannot be changed.

**How mempool works:**

1. A user creates a transaction (via DAPP or a wallet), and is sent to node(s) that it is connected to.
2. The node adds the transaction to its own mempool and performs checks to validate the transaction.
3. The transaction is broadcasted to the rest of the network and added to the receiving node’s mempool.
4. The transaction is also received by mining nodes, which add them to blocks.
5. After mining a block successfully, it is added to the blockchain and the block is broadcasted throughout the network.
6. If a receiving node contains a included transaction in its mempool, it is removed.

Generally, nodes designate roughly 300 MB of space for mempool. During high network activity, this storage can reach its limit and transactions may be dropped to clear some memory. To ensure that a transaction is not dropped, users can pay a higher gas fee to ensure higher priority. Higher priority transactions are also placed higher in the queue for processing. This can pose a security risk.

## Security Risk

Due to the open and decentralized nature of the blockchain, anyone (with a full node client) can track transactions and explore the mempool for malicious propose. Transactions order can also be manipulated by providing higher gas fees. Miners are also capable of rearranging the order of transaction in the block they mined.

If critical logic is dependent the order of transaction, it can lead to unexpected output and subversion of result.

### Example

Lets take a look at the following smart contract to understand the vulnerability better.

<script src="https://gist.github.com/AayushmanThapaMagar/1bbd1500b7c201dae830a0cf54f40d54.js"></script>

It is a simple smart contract in which if a player guesses the number correctly, they will be deemed the winner. However, this contract is vulnerable to TOD. The attack narrative could be the following:

1. A player will guess the number correctly and submit it as a transaction.
2. The pending transaction will be stored in a node’s mempool.
3. A cheater will be monitoring the mempool for transactions to our smart contract.
4. The cheater will see the correct guess and submit it with higher gas fee.
5. The cheater’s transaction will be processed first and be deemed the winner.

In our example, there was no significant damage. The impact of TOD is dependent on the logic implemented in the smart contract, and is usually classified as low or medium. However, it can have critical impact if there is potential for financial loss.

Technically speaking, this contract is vulnerable to **displacement** TOD. Displacement TOD vulnerabilities don't require any interaction between the victim and the malicious user. Other _flavors_ of TOD vulnerability include **Insertion** and **Suppression**.

In Insertion TOD, like the name suggests, inserts the malicious actor’s transaction in between the contract and the user’s transaction. For example, if Alice places an order to buy an NFT for _a best price,_ Bob could place his transaction before Alice and buy the NFT and immediately sell it for a higher price for profit.

Finally, Suppression TOD simply means flooding the contract with many transactions in order to delay other user’s access to the functions. This is also known as _block-stuffing attack._

Real world example includes **Bancor (BNT)**, which is an ICO that raised over $150M over a few minutes. Security researches discovered that BNT was vulnerable to TOD. Luckily, the vulnerability has not been exploited and the team at Bancor claims it has been fixed.

## Identification Techniques

Automated tools such as **Mythril, Oyente, and Securify** are able to discover such vulnerabilities, among many more. To discover TODs manually during the auditing process, the following should be considered:

1. Determine if a function changes the state of the smart contract.
2. Determine if such function’s output is depended on the order of inputs (transactions).

In the case of our example contract, the answer is **yes** for both criteria, thus we had the vulnerability.

## Mitigation Measures

TOD vulnerabilities can be difficult to prevent and fix, as they require significant changes to the logic of the code.

One solution would be to use **commit reveal hash scheme.** Instead of transmitting the answer directly, a **hash** of _salt, address, and answer_ to the contract is transmitted. Here salt is a random number chosen by the user. This hash is stored by the contract along with the user’s address. Now, when the user wants to submit their answer, they will create a transaction with _answer and salt._ The contract will recalculate the hash and if it matches with the previous hash, the answer is released.

The following modification to our example contract is made to remedy the vulnerability.

<script src="https://gist.github.com/AayushmanThapaMagar/b2f475d5067a7cdd1204773e3ccaf598.js"></script>

Here, the code the mapping stores a hash associated with a particular address. The **_storeHash_** function is used to do this. After that, the player will call the **_guess_** function and pass in the real answer along with the salt.

Although, the cheater is still able to intercept these messages and forward them to the contract before the player, it would be useless since the check would fail at line 13. Attempting to capture the answer and calculating the hash to submit them both wouldn't work either as it would be too late (a time based locking mechanism could also be implemented to enforce this).

## References

1. [https://swcregistry.io/docs/SWC-114](https://swcregistry.io/docs/SWC-114)
2. [https://consensys.github.io/smart-contract-best-practices/attacks/frontrunning/](https://consensys.github.io/smart-contract-best-practices/attacks/frontrunning/)
3. [https://www.getsecureworld.com/blog/transaction-order-dependence-attack-in-smart-contract/](https://www.getsecureworld.com/blog/transaction-order-dependence-attack-in-smart-contract/)
4. [https://www.blocknative.com/blog/mempool-intro](https://www.blocknative.com/blog/mempool-intro)
5. [https://www.quicknode.com/guides/web3-sdks/how-to-access-ethereum-mempool](https://www.quicknode.com/guides/web3-sdks/how-to-access-ethereum-mempool)
6. [https://www.immunebytes.com/blog/transaction-ordering-dependency-how-can-it-hamper-a-smart-contract/](https://www.immunebytes.com/blog/transaction-ordering-dependency-how-can-it-hamper-a-smart-contract/)
7. [https://www.youtube.com/watch?v=LDOzDQ44dM4](https://www.youtube.com/watch?v=LDOzDQ44dM4)
8. [https://www.youtube.com/watch?v=MN55R440twQ](https://www.youtube.com/watch?v=MN55R440twQ)
