---
title: "Re-entrancy Vulnerabilities in Smart Contracts"
date: 2022-10-05T00:00:00+05:45
image: "images/blog/default.png"
author: "Aayushman Thapa Magar"
# type: "featured"
description: "This is meta description"
draft: false
---

{{< toc >}}

## Foreword

This is second in a series of articles on vulnerabilities that smart contracts are susceptible to. You can find my articles in this topic by following the links.

1. [Transaction Order Dependence](/blog/transaction-order-dependence-vulnerabilities-on-smart-contracts/)
2. [Re-entrancy](/blog/re-entrancy-vulnerabilities-in-smart-contracts/)
3. [Signature Malleability](/blog/signature-malleability-vulnerabilities-in-smart-contracts/)
4. [Arithmetic Vulnerabilities](/blog/arithmetic-vulnerabilities-in-smart-contracts/)

The primary purpose for these articles is **not** to educate others, but to test the depth of my own understanding. However, I do hope that my efforts provide even a little value to you dear readers. Please let me know if any questions arise or any mistakes are made so we can learn and grow together.

## Contents

1.  Re-entrancy Overview
2.  Security Risk
3.  Identification techniques
4.  Mitigation measures
5.  References

### Re-entrancy Overview
In computer science, re-entrancy refers to multiple invocation of a procedure, where it can be interrupted and called again without completing its previous execution. This issue is especially prevalent in single threaded systems, such as the EVM. Re-entrancy vulnerabilities occur when a function makes an external call to untrusted contracts which can recursively call the original function and hijack the control flow.

This often happens when funds are transferred and the fallback/receive functions on the receiving contract call the original transfer function.

{{< image src="/images/blog/smart-contract/contract.webp" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

### Security Risk
If the accounting mechanism executes **after** the transfer mechanism, re-entrancy can be used to siphon funds out of the vulnerable contract. This iterative calling of functions can last as long as there is remaining gas.

Such was the case with the infamous DAO hack which led to $60M worth of ether to be stolen. As a response, the Ethereum blockchain, was forked into Ethereum and Ethereum Classic. Below are some notable re-entrancy hacks.

*   Uniswap/Lendf.Me lost $25M (April 2020)
*   The BurgerSwap lost $7.2M (May 2021)
*   The SURGEBNB lost $4M (August 2021)
*   CREAM FINANCE lost $18.8M (August 2021)
*   Siren protocol lost $3.5M (September 2021)
*   Fei Protocol lost $80M (April 2022)

#### Example

Re-entrancy is perhaps the most popular web3 vulnerability due to its impact and prevalence. It can also be easy to miss during development as it is a type of logic flaw.

Lets take a look at the code snip below to better understand the vulnerability.

<script src="https://gist.github.com/AayushmanThapaMagar/0233892b0bdb13744f16428d71603804.js"></script>

The above contract is a simple one. Users can send some ether to the contract and withdraw it later. However, this contract is vulnerable to re-entrancy attack.

An attacker could write a malicious contract such as the following to exploit this weakness.

<script src="https://gist.github.com/AayushmanThapaMagar/57d0298b9fbfdf17f92075c387eea78a.js"></script>

Here, the attacker is depositing some amount of ether pass the check at line 12 in the vulnerable contract. The deposited amount is then immediately withdrawn. The **_withdraw_** function in the victim contract will send the amount to the attacker contract, where the receive function will be executed, which in turn withdraws more amount.

The accounting mechanism at line 15 in the victim contract will never get a chance to execute and reset the balance of the attacker as the two functions are called recursively until all the funds are exhausted.

## Identification techniques
There is always risk associated with performing external calls. To identify whether a contract is vulnerable to re-entrancy attacks, the following must be considered.

1.  Call to external untrusted contracts is made.
2.  A state change is made afterwards.

Although these are not always indicative of whether re-entrancy is present, it can be a good indicator. In our case, the answer was **yes** to both, hence we had the vulnerability.

## Mitigation measures
The simplest way to prevent re-entrancy vulnerabilities would be to complete all the internal accounting mechanisms before interacting with external functions. This is also known as the _check-effects-interactions_ pattern, where each statement is categorized as either a check, an effect (state change) or an interaction and arranged **strictly** accordingly.

In our example, it would look something like this;

<script src="https://gist.github.com/AayushmanThapaMagar/3869dec0ce2f4d55ca98e02248dd7ca3.js"></script>

Re-entrancy guards can also be used to prevent such vulnerabilities. Re-entrancy guards are function modifiers that prevents re-entrancy attacks. Open Zeppelin has a library that enables this feature, which can be found [here](https://github.com/OpenZeppelin/openzeppelin-contracts-upgradeable/blob/master/contracts/security/ReentrancyGuardUpgradeable.sol).

## References

1.  [https://consensys.github.io/smart-contract-best-practices/attacks/reentrancy/](https://consensys.github.io/smart-contract-best-practices/attacks/reentrancy/)
2.  [https://hacken.io/education/reentrancy-attacks/](https://hacken.io/education/reentrancy-attacks/)
3.  [https://blog.openzeppelin.com/reentrancy-after-istanbul/](https://blog.openzeppelin.com/reentrancy-after-istanbul/)
4.  [https://hackernoon.com/hack-solidity-reentrancy-attack](https://hackernoon.com/hack-solidity-reentrancy-attack)
5.  [https://quantstamp.com/blog/what-is-a-re-entrancy-attack](https://quantstamp.com/blog/what-is-a-re-entrancy-attack)
6.  [https://arxiv.org/pdf/2105.02881.pdf](https://arxiv.org/pdf/2105.02881.pdf)
7.  [https://www.youtube.com/watch?v=4Mm3BCyHtDY](https://www.youtube.com/watch?v=4Mm3BCyHtDY)