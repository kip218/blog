---
layout: post
title: "Ethereum Smart Contract"
excerpt: "Coding an Escrow Contract Using Solidity"
date: "2022-12-02"
---

For one of my elective courses for Computer Science, I am currently taking an introductory course on blockchains and distributed ledger technology. One of the assignments was to code a smart contract on the Ethereum Goerli Testnet using Solidity.

&nbsp;
## The Task

The assignment required me to create a smart contract on the `Ethereum` Testnet. More specifically, the smart contract had to be an escrow contract. 

What is an escrow contract?

An escrow contract is a smart contract which can be deposited to and withdrawn from as part of a transaction between two parties. As the name "escrow" suggests, it is a transaction arrangement that is mediated by the smart contract. 

<p align="center">
	<img src="/assets/images/Escrow_Contract/escrow_diagram.png"
		 width=400 />
</p>

During the transaction arrangement, the buyer deposits the payment amount into the escrow contract. The escrow contract holds the payment funds until the transaction conditions are satisfied. Once the transaction conditions are met, the funds are released and the selling party is allowed to withdraw from the escrow contract, which completes the transaction.

For this specific assignment, the transaction condition was a time limit of 1 day. If 1 day has passed after the buyer deposits to the contract, then the seller can withdraw from the contract. If not, the contract rejects the withdrawal request.

&nbsp;
## Coding in Solidity

Ethereum smart contracts are coded in a programming language called Solidity. To do this, we used a web-based IDE specifically designed for smart contracts called [Remix IDE](https://remix-project.org/){:target="_blank"}. The IDE was linked to my MetaMask wallet on the Ethereum Goerli Testnet.

&nbsp;
### What is Goerli?

The Ethereum Mainnet is where the actual ETH cryptocurrency is traded on. If web3 developers wanted to test their blockchain applications before fully deploying them, it would be a bad idea to do so on the Mainnet immediately. This is where the Goerli Testnet comes in. It is essentially a simulation network where developers can test their blockchain applications with fake ETH tokens. To get Goerli Testnet tokens, developers have to mine the tokens from the [Goerli Faucet](https://goerli-faucet.pk910.de/){:target="_blank"}. Afterwards, they are free to test their blockchain application using the Goerli tokens.

<p align="center">
	<img src="/assets/images/Escrow_Contract/goerli_faucet.png"
		 width=300 />
</p>

&nbsp;
## The Code

Since the requirements for this assignment were fairly simple, the Solidity code for the escrow contract was not very complicated. Here is the code I used for the contract:

```java
//SPDX-License-Identifier: MIT
pragma solidity 0.8.14;

// Escrow contract
// - only Alice can deposit to contract
// - amount stored is only withdrawable by Bob
// - Bob's address: 0xECf74C19215C8DD2BAF16AD3a6eC1A25386d813c

contract Escrow {
    address public Alice;
    address payable public Bob;
    uint withdrawTime;

    enum State {AWAITING_DEPOSIT, WITHDRAWABLE, COMPLETE}
    State public currentState;

    constructor(address _Alice, address payable _Bob) public {
        Alice = _Alice;
        Bob = _Bob;
    }

    function deposit(uint256 amount) public payable {
        require(msg.sender == Alice, "Only Alice can deposit to contract");
        require(currentState == State.AWAITING_DEPOSIT, "Already deposited by Alice");
        currentState = State.WITHDRAWABLE;
        withdrawTime = block.timestamp;
    }

    function withdraw() public payable {
       require(msg.sender == Bob, "Only Bob can withdraw from contract");
       require(withdrawTime + 1 days < block.timestamp, "Not withdrawable yet");
       require(currentState == State.WITHDRAWABLE, "Already withdrawn by Bob");
       Bob.transfer(address(this).balance);
    }

}
```

Here is what it looks like in the Remix IDE:

<p align="center">
	<img src="/assets/images/Escrow_Contract/escrow_code.png"
		 width=700 />
</p>

On the left side of the IDE, you can see that we are using my MetaMask wallet address for the Goerli Testnet to test this contract. Once deployed, the Remix IDE provides a convenient GUI for all the functions implemented into the deployed contract.

As for the code, it is a simple hard-coded escrow contract that allows only "Alice" to deposit into the contract and only "Bob" to withdraw from it. These addresses must be provided to the contract when deployed, as you can see in the constructor function. The contract only has 2 functions: Deposit() and Withdraw(). I made sure that the smart contract is time-aware by keeping track of the timestamp of when the payment is first deposited. If "Bob" tries to call the Withdraw() function, it checks to see if the time condition of 1 day is satisfied before approving the withdrawal attempt.

After deploying, I tested the Deposit() function by using the GUI provided by Remix IDE. I deposited a huge amount of WEI, as you can see in the screenshot below. It seems like a large amount of WEI, but it converts to 0.2 ETH. WEI is the smallest denomination of ETH, sort of like how a Cent is equal to 0.01 of a Dollar.

<p align="center">
	<img src="/assets/images/Escrow_Contract/after_deposit.png"
		 width=300 />
</p>

&nbsp;
## Checking the Transaction on Etherscan

The great thing about currencies based on blockchain technology is that every transaction is public. Therefore, the deposit we just made as "Alice" to our escrow contract should be visible as part of the Goerli Blockchain Testnet. We can check this using Etherscan, which is a block explorer for Ethereum blockchains. Etherscan allows the public to search and inspect any and all transactions made on the blockchain network.

This is the Etherscan link for the contract we just created: [https://goerli.etherscan.io/address/0x579f6c2bbf188013ec92730aaa26cd4e909cd833](https://goerli.etherscan.io/address/0x579f6c2bbf188013ec92730aaa26cd4e909cd833){:target="_blank"}.

<p align="center">
	<img src="/assets/images/Escrow_Contract/etherscan.png"
		 width=800 />
</p>

As you can see, there are two events recorded on the blockchain for our contract. The first one is when the escrow contract was initially created. The second one is the deposit we made as "Alice" to the escrow contract. Now all that is left for the contract is waiting for "Bob" with the correct address to call the Withdraw() function after a day has passed. For this assignment, I specified Bob's address to be the one given to us by the our Professor and TA.

&nbsp;
## Conclusion

The past several years have been full of talk and debate about cryptocurrencies and blockchain technology. We've all heard so much about how this new technology is going to change the world and start a huge tech revolution. I kept hearing terms like "Smart Contract", but I never really understood what a smart contract might look like in terms of code. While I still am a complete beginner to web3 and blockchain technology, this was a simple yet interesting introductory assignment to expose myself to newer technologies in the industry.