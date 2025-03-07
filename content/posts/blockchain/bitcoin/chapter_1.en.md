---
layout: post
title: "The Technical Principles and Development Status of Bitcoin"
description: "Introduce the underlying technical principles of Bitcoin and understand its own and related technologies' current development status."
date: 2025-03-05T14:25:18+08:00
image: "/posts/blockchain/bitcoin/images/chapter_1-cover.jpg"
tags: ["Blockchain"]
---

## Monetary Systems Rely on Trust

When we conduct cash transactions, we usually exchange money for goods directly. The currency is issued by the state and backed by the state's credit, so everyone recognizes its value. There are also various anti-counterfeiting mechanisms to check the authenticity of the currency, which also ensures the security of the transaction. From this, it can be seen that the recognition of the currency's value and its security in transactions are the basic elements for the currency to be used in transactions.

However, when transactions occur in the digital world, the situation changes. For instance, when we make online purchases, the transaction is not just between the buyer and the seller; it also requires the bank to process the transaction, deducting funds from the buyer's account and adding funds to the seller's account.

<div class="content-image">
<img src="/posts/blockchain/bitcoin/images/chapter_1-cash-vs-electronic-cash.jpg" width="80%">
<p>Cash vs Electronic Cash</p>
</div>

Why can't online transactions be like offline ones, where the buyer directly gives the money to the seller? You might say it's because the form of currency has changed. Yes, the form of currency has changed from physical to digital. Physical currency is issued by the state and has effective anti-counterfeiting mechanisms to ensure its security, so individuals cannot create cash out of thin air. Both parties complete the transaction based on this consensus and trust. However, digital currency is just a piece of data and does not have effective anti-counterfeiting measures like physical currency. If everyone managed their digital accounts like they do with cash, it would mean they could freely modify account data to "issue" digital currency and forge their account balances. Then, citizens' trust in the digital currency system would collapse, and transactions would be impossible. Therefore, a trusted third-party institution is needed. Everyone hands over their accounts to it for management. When a transaction is needed, the institution handles it. Both parties complete the online transaction based on their trust in the third-party institution.

## Bitcoin: The Native Trustworthy Digital Currency

In 2008, Satoshi Nakamoto released the Bitcoin white paper titled <i>Bitcoin: A Peer-to-Peer Electronic Cash System</i>, with the aim of designing a digital currency system that does not rely on a third-party trusted institution to handle transactions. The actual purpose was to design a digital currency that could be traded directly between two parties, just like using cash.

We have already discussed that in transactions, the recognition of the value of currency and the security of currency in transactions are the basic elements for currency to be used in transactions. Bitcoin is different from traditional physical currency and digital currency. Traditional physical currency is usually issued by the state, while digital currency is exchanged for physical currency at par by a trusted institution. However, Bitcoin is issued by the Bitcoin system itself. Therefore, in order for Bitcoin to exert its monetary attributes, whether it is to recognize its value or its security in transactions, essentially, it is necessary for people to trust the value and security of the Bitcoin system itself.

At present, the value of Bitcoin is still linked to fiat currencies, and its monetary attributes are far lower than its asset attributes. From an economic perspective, Bitcoin's high volatility has attracted a large number of speculators, and its price is easily driven by market sentiment, institutional movements or macroeconomic events rather than long-term value logic. From a political perspective, the nature of Bitcoin is still unclear in many countries and regions, which leads to its value fluctuations being affected by policy expectations, and some countries restrict the trading of Bitcoin with fiat currencies. Therefore, to recognize the value of Bitcoin, it is necessary to go beyond speculation and establish a long-term consensus based on its practicality and anti-inflation characteristics.

The security of Bitcoin stems from blockchain technology, which is the most crucial aspect in establishing the value of Bitcoin as a currency. Now let's delve into the essence of blockchain technology and see how it eliminates the reliance on trusted third parties to ensure the security of Bitcoin transactions, even when the currency is issued within the Bitcoin system itself.

## Blockchain

<div class="content-image">
<img src="/posts/blockchain/bitcoin/images/chapter_1-blockchain.jpg" width="50%">
<p>Blockchain</p>
</div>

We refer to the underlying technology of the Bitcoin system as blockchain technology. This is because in the Bitcoin system, transaction data is packaged into a data structure called a block and organized in the form of a linked list.

### How is The transaction Carried Out?

In traditional digital currency systems, the accounts of both transaction parties are uniformly managed by a trusted third-party institution. But in the absence of a centralized organization, how can we build and manage our own accounts? Bitcoin achieves account management through asymmetric encryption technology and the UTXO model.

#### Public Key And Private Key

Each account has a pair of public and private keys. The private key is the sole proof of a user's control over their Bitcoin, and all transactions must be digitally signed with the private key to be valid. Once the private key is lost, the funds in the account can never be retrieved. The public key is used to verify the legitimacy of transactions and ensure the authenticity of the transaction initiator's identity.

#### UTXO Transaction Model

Transactions in the Bitcoin system are divided into inputs and outputs. The system maintains a UTXO pool, where each UTXO represents an unspent transaction output. Inputs consume existing UTXOs as the source of transaction funds, while outputs define the new destination of the funds. The Bitcoin balance of each account is the sum of the UTXOs in the UTXO pool associated with that account. For example, in the figure below, Account A initiates a 0.6 BTC transfer to Account B. At this time, there are two UTXOs in the UTXO pool associated with Account A. Since no single UTXO is sufficient to cover the payment, these two UTXOs, totaling 0.7 BTC, are used. During the transfer, the 0.1 BTC change is returned to Account A.

<div class="content-image">
<img src="/posts/blockchain/bitcoin/images/chapter_1-transaction-process.jpg" width="50%">
<p>Transaction Process</p>
</div>

### How to Reach A Consensus on The Transaction?

On the island of Yap in Micronesia, stone carvings are used as currency. The largest one has a diameter of 3.6 meters. When its ownership changes, people do not move it, but everyone in the community knows it has a new owner.

Similarly, when Bitcoin is traded, only the change of ownership occurs. In traditional digital currency transactions, a centralized and trusted third-party institution handles them, and since everyone believes in this institution, they all recognize the validity of the transactions. However, in the Bitcoin system, there is no such third-party institution. Whether a transaction is legal and valid requires the consensus of all nodes in the entire Bitcoin system.

#### Transaction Broadcast

When a transaction is issued, it will be broadcasted in the Bitcoin system network. The Bitcoin network is a peer-to-peer network that adopts the Gossip protocol. When a transaction is broadcasted, it spreads throughout the network like gossip. Nodes that receive the transaction will verify its validity. If the transaction is valid, it will continue to be broadcasted; otherwise, it will not. Meanwhile, after confirming the transaction's validity, miner nodes will also store it in the transaction pool, waiting to be packed into a block.

<div class="content-image">
<img src="/posts/blockchain/bitcoin/images/chapter_1-broadcast-transaction.jpg" width="50%">
<p>Broadcast Transaction</p>
</div>

#### Proof of Work

The process by which miner nodes package blocks is called "mining", a term that stems from its resource-consuming and competitive nature, similar to traditional mineral extraction which requires physical labor or capital investment to obtain scarce resources.

Miners need to calculate to find a block hash value (Nonce) that meets the network difficulty requirements, which requires a large amount of computing resources and electricity for hash operations, similar to the energy input required for mining. All miners in the system compete simultaneously to solve the hash problem, and only the winner can obtain the block reward. This mechanism of obtaining rewards by consuming resources is called <i>Proof of Work (PoW)</i> mechanism. The total amount of Bitcoin is capped at 21 million, and the mining reward is halved every 210,000 blocks (approximately once every four years), simulating the finiteness of "minerals". When all Bitcoins have been "mined", the mining rewards of miners will come entirely from transaction fees.

When a block is packed, it will be broadcasted in the system just like a transaction. Nodes that receive the block will verify its validity. If it is valid, the block will be added to the local blockchain and continue to be broadcasted. If it is invalid, the broadcast will be stopped. If the node is a miner node, it will also check whether the block already contains the transaction it is currently packing. If it does, the miner node will discard the block it is currently packing and select transactions from the transaction pool to start packing again.

<div class="content-image">
<img src="/posts/blockchain/bitcoin/images/chapter_1-broadcast-block.jpg" width="80%">
<p>Broadcast Block</p>
</div>

#### Block And Chain Structure

The data structure of a block can be divided into two parts: the block header and the block body. The block header includes the hash of the previous block, the difficulty target, the timestamp, the version number, the Merkle root, and the nonce; the block body contains a set of transactions.

<div class="content-image">
<img src="/posts/blockchain/bitcoin/images/chapter_1-the-structure-of-block.jpg" width="100%">
<p>The Structure of Block</p>
</div>

* Previous Hash：The hash value pointing to the previous block in the blockchain forms a chain structure, ensuring that blocks can only be appended to the end of the chain and preventing the tampering of historical records.
* Difficulty Target：The difficulty target requires miners to calculate and find a nonce so that the hash value of the block header is less than the target value. The system automatically adjusts the difficulty based on the total network computing power to ensure that a block is produced on average every 10 minutes.
* Timestamp：Record the Unix timestamp of the block creation, accurate to the second.
* Version：Indicate the protocol version used in the block.
* Merkle Root：Merkle root hash, the double hash of all transaction hash values, generated through the Merkle tree, represents the fingerprint of all transactions in that block.
* Nonce：Miners search for the Nonce value through brute force, making the Block Hash less than the Difficulty Target.

Such a chain structure ensures the immutability of block data. If one wants to tamper with a certain transaction, they must reconstruct the block where the record is located and all subsequent blocks. Moreover, the generation of each block requires computing power and also needs to be recognized by the majority of nodes in the network. Therefore, if one wants to tamper with the data, they need to possess more than 50% of the system's computing power, which is almost impossible.

#### Chain Forks And Double Spending

Please consider such a situation: when there are two or more miner nodes in the system that almost simultaneously package blocks and broadcast them in the system. Since both of these blocks are legal, a fork occurs in the blockchain. However, as subsequent blocks are created, one of the forks will become the longer chain, and at this point, the other forks will be eliminated. This characteristic is called the principle of the longest valid chain.

##### Example

* Miner A's block was first accepted by some nodes, while Miner B's block was accepted by another part of the nodes.
* When miner C generates a new block on miner A's chain, the length of that chain becomes n + 1, while miner B's chain remains at n.
* All nodes on the network automatically switch to the chain of miner A, and the block of miner B is discarded.

<div class="content-image">
<img src="/posts/blockchain/bitcoin/images/chapter_1-the-longest-valid-chain.jpg" width="80%">
<p>The Longest Valid Chain</p>
</div>

The longest valid chain principle can be used to avoid the problem of double spending. When an account conducts multiple transactions in a short period of time, due to network latency, some UTXOs may be used as inputs for multiple transactions and broadcasted across the network. Due to network latency, these transactions reach different nodes in different orders, so different nodes consider different transactions as valid. This results in the blocks packaged containing different transactions, causing a fork in the blockchain. However, as subsequent blocks are added, one of the chains will eventually become the longest chain, and at that point, other versions of the chain will become invalid.

## Summary

* The recognition of the value of currency and the security of currency in transactions are the basic elements for currency to be used in transactions. Traditional digital currency systems rely on the centralized management of a trusted third party to achieve these two points, but Bitcoin has realized a decentralized digital currency system through blockchain technology.

* The Bitcoin system employs asymmetric encryption technology and the UTXO transaction model to achieve digital currency transactions without relying on third-party institutions. Coupled with a peer-to-peer network, the PoW mechanism, blocks, chain locking, and the principle of the longest valid chain, it resolves the double-spending problem and ensures that the data on the chain cannot be tampered with.