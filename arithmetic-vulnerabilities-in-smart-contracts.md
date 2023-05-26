---
title: "Arithmetic Vulnerabilities in Smart Contracts"
date: 2022-11-04T00:00:00+05:45
image: "images/blog/default.png"
author: "Aayushman Thapa Magar"
# type: "featured"
description: "This is meta description"
draft: false
---

## Forword
This is fourth in a series of articles on vulnerabilities that smart contracts are susceptible to. You can find the rest of the articles on the following links:

1. [Transaction Order Dependence](/blog/transaction-order-dependence-vulnerabilities-on-smart-contracts/)
2. [Re-entrancy](/blog/re-entrancy-vulnerabilities-in-smart-contracts/)
3. [Signature Malleability](/blog/signature-malleability-vulnerabilities-in-smart-contracts/)
4. [Arithmetic Vulnerabilities](/blog/arithmetic-vulnerabilities-in-smart-contracts/)

The previous article was hefty, so for this week, I wanted to keep things light. As always, the purpose of these articles is **not** to educate others, but to test the depth of my own understanding. If I can explain it properly, means I understand it properly. Hope my efforts are even a little useful to someone else and aid them in this journey with me.


## Contents
1.  Overflows and Underflows Overview
2.  Security Risk
3.  Mitigation measures
4.  References

## Overflows and Underflows Overview
Overflows and Underflows are a classic vulnerability in “normal” infosec world. Such flaws have been abused through the history leading to severe consequences — millions of devices being infected by spyware. In fact, Common Weakness Enumeration (CWE) placed overflows and underflows as 12th most common flaw in 2021.

That’s cool and all, but what _is_ overflow and underflow (CWE-190 and CWE-191 respectively)? Allow me to answer that question with a story about a young lad named Bob and his magic car.

Bob got an old Maruti Suzuki 800 as his first car on his 18th birthday. Bob fakes his enthusiasm but is still grateful to his parents, but its obvious the car has been around longer than he has been alive!

{{< image src="/images/blog/smart-contract/bobs-car.webp" caption="(not) Bob’s car" alt="(not) Bob’s car" height="" width="" position="center" command="fit" option="" class="img-fluid" title="(not) Bob’s car" >}}

The odometer reads that the car has been driven for a total of 99997 kilometers — which seems impossible and slightly interesting. Out of curiosity Bob decides to meet Alice at her house, which is 3 kilometers away. While driving, the odometer reads 99998 and finally 99999. After the final kilometer, the odometer shows 00000.

The car has now driven a total of 0 kilometers, making it brand new! A fairy hopped out of the glove compartment and cast a magic spell. The door handle is no longer rusty, the cracks in his windscreen is fixed, the suspicious stain on the passenger seat vanished and the seatbelt no longer smells like perimeter of a Chipotle restroom. The car even changed color from a depressing green to a vivid red! Well, not really, the odometer simply overflowed. Sorry Bob… :(

This happened because the odometer is unable to display more than 5 digits. So when the first 6 digit number appears (100000), it can only display the last 5. And the opposite is called integer underflow.

You may think overflow and underflow like Pacman appearing on the right side of the screen after passing through the left side and vice versa.

{{< image src="/images/blog/smart-contract/pac-man.webp" caption="" alt="" height="" width="" position="center" command="fit" option="" class="img-fluid" title="" >}}

### Arithmetic flaws in solidity

Lets take a closer look in the context of computer science to understand how exactly such vulnerabilities occur.

Every action we do on our computers; watching cat memes on YouTube, playing unhealthy amount of Valorant, reading an article on medium by an author who thinks he’s funny — everything, boils down to zeroes and ones being manipulated in very precise, mathematical ways. The same is true for the EVM. These zeroes and ones are called byte strings.

Arithmetic flaws often arise when byte strings exceed their allocated space (byte size). This can cause the program to act in unexpected ways, which hackers can use to their advantages.

Exactly how much space is allocated is dependent on the data type used. In the case of solidity;

```javascript
uint8 ranges from 0 to 2⁸ - 1,  
uint16 ranges from 0 to 2¹⁶ - 1
...
uint256 ranges from 0 to 2²⁵⁶ - 1.
```


Alternately, we can use the following to get the min and max value of an integer type (x).

```javascript
type(x).min
type(x).max
```

## Security Risk 
Solidity compiler before versions 0.8.0 does not check for overflows and underflows. This means that if the byte size is reached for a given integer type, it will assume the first viable value. This can cause minor issues from disrupting the execution of the smart contract to serious issues that could lead to gaining an absurd amount of ERC-20 tokens.

### Proof of concept

Lets take a look at the following code snip.

<script src="https://gist.github.com/AayushmanThapaMagar/aa9bc692153ac3b1a473b2aff5667840.js"></script>

Here, the contract is very simple. The _underflow_ and _overflow_ variables are of type _uint8_ and have the values 0 and 255 respectively. The _\_underflow()_ and _\_overflow()_ functions will subtract 1 from underflow and \_overflow will add 1 to overflow variables.

I will be using the brownie framework (because I am more familiar with python.)

{{< image src="/images/blog/smart-contract/compile.webp" caption="Compiling the contract using brownie" alt="Compiling the contract using brownie" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Compiling the contract using brownie" >}}

{{< image src="/images/blog/smart-contract/console.webp" caption="Deploying the contract locally" alt="Deploying the contract locally" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Deploying the contract locally" >}}

```bash
$ brownie compile
$ brownie console
>>> acct = accounts[0]
>>> arith = arithmetic.deploy({'from': acct}
```

Now, we have access to the deployed contract’s ABI. We can access the public variables via their getter functions.

{{< image src="/images/blog/smart-contract/airth.webp" caption="Public variable getter functions" alt="Public variable getter functions" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Public variable getter functions" >}}


Lets call the \_overflow() and \_underflow() functions and check if our smart contract is vulnerable.

{{< image src="/images/blog/smart-contract/airthOverflow.webp" caption="Calling the functions" alt="Calling the functions" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Calling the functions" >}}

Calling the getter functions once more to check their states.

{{< image src="/images/blog/smart-contract/airthOverflow-1.webp" caption="Overflow and underflow occurred" alt="Overflow and underflow occurred" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Overflow and underflow occurred" >}}

Here, we can see that the value of overflow (255) has been changed to 0, instead of 256, because it exceeded the byte size of uint8.

The value for underflow (0) has been changed to 255 instead of -1 because, unit cannot go below 0 and instead, it defaults to the maximum value for uint8.

## Mitigation measures
The easiest and the quickest fix would be to simply use solidity compiler versions newer or equal than 0.8.0. In these versions, the contract will throw an error if such vulnerabilities occur.

{{< image src="/images/blog/smart-contract/airthOverflow-2.webp" caption="Overflow and underflow occurred" alt="Overflow and underflow occurred" height="" width="" position="center" command="fit" option="" class="img-fluid" title="Overflow and underflow occurred" >}}

Another method to prevent such vulnerabilities would be to use the [safemath.sol](https://github.com/OpenZeppelin/openzeppelin-contracts/blob/master/contracts/utils/math/SafeMath.sol) library by OpenZeppelin.

## References
1.  [https://www.comparitech.com/blog/information-security/integer-overflow-attack/](https://www.comparitech.com/blog/information-security/integer-overflow-attack/)
2.  [https://hackernoon.com/hack-solidity-integer-overflow-and-underflow](https://hackernoon.com/hack-solidity-integer-overflow-and-underflow)
3.  [https://solidity-by-example.org/hacks/overflow/](https://solidity-by-example.org/hacks/overflow/)