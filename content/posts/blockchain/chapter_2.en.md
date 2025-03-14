---
layout: post
title: "A Brief Overview of All Common Consensus Algorithms"
description: "Introduce all common consensus algorithms, including PoW, PoS, DPoS, etc."
date: 2025-03-12T13:15:05+08:00
image: "/posts/blockchain/images/chapter_2-cover.jpg"
tags: ["Blockchain"]
---
## What's A Consensus Algorithm?

A consensus algorithm is a rule and protocol in a distributed system by which all nodes reach an agreement on a certain data state or transaction sequence.

In 1982, Leslie Lamport proposed the "<a href="https://en.wikipedia.org/wiki/Byzantine_fault">Byzantine Generals Problem</a>", which studies how distributed systems can tolerate faults. The problem is set against the backdrop of a group of generals communicating through messengers to vote on whether to attack or retreat. Due to the possibility of traitors among the generals who may vote maliciously or impersonate other generals, or because messages may be lost or tampered with during transmission, the army ultimately fails to coordinate its actions.

<div class="content-image">
<img src="/posts/blockchain/images/chapter_2-byzantine-generals.jpg" width="80%">
<p>Byzantine Generals</p>
</div>

In a computer system, the general is the network node and the messenger is the communication system. How to ensure that all honest nodes can still synchronize and recognize the same "correct" data or transaction status even when the network is unreliable, there is a delay or some nodes are malicious, this problem involves information security, but relying solely on digital signatures cannot solve it, because digital signatures can only verify the integrity and source of the message, but cannot identify when malicious nodes actively send incorrect information. For this reason, we need consensus algorithms.

## Consensus Algorithms in Blockchain

Consensus algorithms are tools that enable a group of mutually distrustful individuals to reach an agreement in a distributed network, ensuring the secure and fair operation of the blockchain without centralized control. They determine who has the right to generate new blocks (the right to bookkeeping) and how the validity of blocks is verified.

### PoW（Proof of Work）

Nodes compete for the right to create blocks by solving complex mathematical problems. The node that solves the problem first gains the right to create the block. Its characteristics are low efficiency, high energy consumption, and high centralization. Security depends on computing power. It is suitable for public chains and value storage.

### PoS（Proof of Stake）

Block production rights are allocated based on the quantity of tokens held by nodes and the duration of holding (stake). Unlike Proof of Work (PoW), PoS does not require nodes to perform complex computations but instead randomly selects block-producing nodes based on the tokens they hold. However, this may lead to the rich getting richer. Its characteristics include medium efficiency, low energy consumption, and medium centralization. Security depends on the amount of tokens held and it is suitable for public chains, financial applications, and community governance scenarios.

### DPoS（Delegated Proof of Stake）​

The aim is to elect block-producing nodes (referred to as "witnesses" or "delegates") through voting by token holders, with these nodes responsible for verifying transactions and generating blocks. DPoS is an improvement on the traditional Proof of Stake (PoS), enhancing the network's efficiency and decentralization through the introduction of a delegation mechanism. It is characterized by high efficiency, low energy consumption, and a relatively high degree of centralization. Its security relies on the credibility of the witnesses and it is suitable for public chain and community governance scenarios.

### PoA（Proof of Authority）​

Block generation is handled by pre-authorized nodes, and transactions are verified and blocks are generated through pre-authorized nodes (referred to as "authoritative nodes" or "validators"). Unlike PoW and PoS, the core idea of PoA is to trust the authorized nodes rather than competing for block generation rights through computing power or token holdings. Its characteristics are high efficiency, low energy consumption, and high centralization. Security depends on the credibility of the validators, making it suitable for consortium chains, private chains, and test networks.

### PoET（Proof of Elapsed Time）

PoET aims to determine the block-producing nodes through a fair random selection process while reducing energy consumption. Proposed by Intel, PoET is based on Trusted Execution Environment (TEE) technology, ensuring that nodes obtain the right to produce blocks after waiting for a random period of time. Its characteristics include high efficiency, low energy consumption, and moderate centralization. Its security relies on TEE technology and is suitable for consortium chains, private chains, and the Internet of Things.

### PoH（Proof of History）

Proposed by the Solana project, Proof of History (PoH) aims to enhance consensus efficiency and network scalability by proving the sequence of events through a timestamp sequence. PoH is not an independent consensus algorithm but is used in conjunction with Proof of Stake (PoS), where PoS is responsible for selecting block-producing nodes and PoH is responsible for verifying the sequence of events, providing an efficient time ordering and verification mechanism for the blockchain. Despite its reliance on time synchronization and high technical complexity, PoH's efficiency and security make it an important innovation in the blockchain field. Its features include high efficiency, low energy consumption, and low centralization, with security relying on the timestamp sequence, making it suitable for public chains and financial applications.

### PoSpace（Proof of Space）

Block production rights are competed for based on the storage space provided by nodes. Unlike Proof of Work (PoW), which relies on computing power, PoSpace utilizes the storage resources of nodes to ensure the security and decentralization of the network. The goal of PoSpace is to reduce energy consumption while providing an efficient consensus mechanism. It is characterized by moderate efficiency, low energy consumption, and moderate centralization. Its security depends on storage space and is suitable for public chains, storage networks, and the Internet of Things.

### PoB（Proof of Burn）

Nodes prove their willingness to participate in consensus by burning tokens and thereby obtain the right to create blocks. The core idea of Proof of Burn (PoB) is to "burn" tokens (i.e., send them to an inaccessible address) to demonstrate the node's commitment and investment in the network. PoB aims to provide a fair and decentralized consensus mechanism while avoiding the high energy consumption problem of Proof of Work (PoW). Despite its relatively high token-burning and centralization risks, its fairness and decentralization features make it an important innovation in the blockchain field. It is characterized by medium efficiency, low energy consumption, and medium decentralization, with security relying on token burning. It is suitable for public chains and community governance.

### PoC（Proof of Capacity）

Block production rights are competed for based on the storage capacity provided by nodes. Unlike Proof of Work (PoW), which relies on computing power, Proof of Capacity (PoC) utilizes the storage resources of nodes to ensure the security and decentralization of the network. The goal of PoC is to reduce energy consumption while providing an efficient consensus mechanism. It is characterized by medium efficiency, low energy consumption, medium decentralization, and security that depends on storage capacity.

### Paxos

Paxos was proposed by Leslie Lamport in 1990, aiming to solve the problem of multiple nodes reaching consensus on a certain value in distributed systems. Paxos is one of the important algorithms in the field of distributed computing and is widely applied in distributed databases, distributed storage systems, and blockchain, among other areas.

The Paxos algorithm consists of two phases: the prepare phase and the accept phase.

* Prepare Phase
    * The Proposer sends a Prepare Request to the Acceptors, which includes a Proposal Number.
    * Upon receiving the Prepare Request, if the Proposal Number is greater than any previously received Proposal Number, the Acceptor promises not to accept any smaller Proposal Numbers and returns the value it has already accepted (if any).
    * The Proposer collects responses from the majority of Acceptors. If an accepted value is found, it selects that value as the proposed value.
* Accept Phase
    * The proposer sends an Accept Request to the acceptors, which includes the proposal number and the proposal value.
    * Upon receiving the Accept Request, if the acceptor's committed minimum proposal number is less than or equal to the proposal number in the request, it accepts the proposal value.
    * The proposer collects the Accept Responses from the majority of the acceptors. If a majority is reached, the proposal value is selected.

### Raft

Raft was proposed by Diego Ongaro and John Ousterhout in 2014, aiming to simplify the process by which multiple nodes in a distributed system reach consensus on a certain value. Raft is an alternative to Paxos. By decomposing the consensus process into more understandable modules, it reduces the complexity of the algorithm while maintaining high fault tolerance and consistency.

The Raft algorithm is divided into three main modules: leader election, log replication, and safety.

* Leader Election
    * When a node starts up, it is in the Follower state and waits for heartbeat messages from the leader.
    * If a Follower does not receive a heartbeat message within the timeout period, it transitions to the Candidate state and initiates an election.
    * A Candidate sends RequestVote RPCs to other nodes. If it receives a majority of votes, it becomes the Leader.
    * The Leader periodically sends heartbeat messages to other nodes to maintain its leadership.
* Log Replication
    * The Leader receives client requests and appends them to its local log.
    * The Leader sends log entries to other nodes via AppendEntries RPCs, asking them to replicate the entries.
    * Other nodes append the log entries to their local logs and send an acknowledgment message to the Leader.
    * Once a log entry has been replicated by a majority of nodes, the Leader applies it to its local state machine and notifies the client that the operation was successful.
* Safety
    * Raft ensures safety through leader election restrictions and log matching properties.
    * Leader election restriction: Only the node with the most up-to-date log can become the Leader.
    * Log matching property: If two nodes have the same log entry at a certain index, all entries before that index are also the same.

### The BFT Family

Byzantine Fault Tolerance (BFT) is a consensus mechanism in distributed systems that ensures consistency even in the presence of Byzantine faults. A Byzantine fault refers to the possibility of nodes malfunctioning in any way, including sending incorrect information, not responding, or engaging in malicious behavior. The goal of BFT is to ensure that the system can still reach consensus in the presence of malicious nodes, and it can typically tolerate no more than one-third of the total number of nodes being malicious.

The BFT algorithm typically includes the following steps: 

* Proposal Phase:
    A node (the proposer) broadcasts a proposed value to other nodes.
* Pre-Vote Phase:
    Other nodes pre-vote on the proposed value, indicating whether they support it.
* Vote Phase:
    If the proposed value receives sufficient pre-vote support, nodes vote on it formally.
* Commit Phase:
    If the proposed value receives sufficient formal vote support, nodes commit it to their local state machines.
* Learn Phase:
    Nodes learn the committed value and broadcast it throughout the system.

#### PBFT

PBFT (Practical Byzantine Fault Tolerance) is a practical BFT algorithm proposed by Miguel Castro and Barbara Liskov in 1999. It is the first practical algorithm capable of achieving Byzantine fault tolerance in an asynchronous network, and is applicable to distributed systems to reach consensus even in the presence of malicious nodes. 

The PBFT algorithm achieves consensus through multiple rounds of message passing and voting mechanisms. Its core process includes the following stages: 

* Request Phase:
    The client sends a request to the primary node to execute a certain operation.
* Pre-Prepare Phase:
    After receiving the request, the primary node broadcasts a pre-prepare message to all replica nodes. The pre-prepare message contains detailed information about the request and a sequence number.
* Prepare Phase:
    Upon receiving the pre-prepare message, each replica node verifies its validity. If the verification is successful, the replica node broadcasts a prepare message to other replica nodes. The prepare message indicates that the node supports the primary node's proposal.
* Commit Phase:
    When a replica node receives a sufficient number of prepare messages (typically 2f + 1, where f is the number of malicious nodes), it broadcasts a commit message to other replica nodes. The commit message indicates that the node is ready to execute the request.
* Reply Phase:
    When a replica node receives a sufficient number of commit messages (typically 2f + 1), it executes the request and sends the result to the client. The client confirms that the request is completed after receiving the same result from a sufficient number of nodes.

#### Tendermint

Tendermint is an efficient and secure BFT algorithm and blockchain framework proposed by Jae Kwon in 2014. It combines Proof of Stake (PoS) and BFT mechanisms, aiming to address the scalability, security, and decentralization issues of traditional blockchains. Tendermint serves as the core consensus engine of the Cosmos network and is widely used in blockchain development and cross-chain interoperability. 

The core idea is to quickly reach network consensus through multiple rounds of voting and signing. The consensus process of Tendermint includes the following stages: 

* Propose Phase:
    The Proposer, a rotating main node, proposes a new block and broadcasts it to all Validator nodes.
* Pre-Vote Phase:
    Upon receiving the proposal, the Validator nodes verify its validity and broadcast a Pre-Vote Message to other nodes. The Pre-Vote Message indicates that the node supports the Proposer's proposal.
* Pre-Commit Phase:
    When a Validator node receives a sufficient number of Pre-Vote Messages (typically 2f + 1, where f is the number of malicious nodes), it broadcasts a Pre-Commit Message to other nodes. The Pre-Commit Message indicates that the node is ready to commit the proposal.
* Commit Phase:
    When a Validator node receives a sufficient number of Pre-Commit Messages (typically 2f + 1), it broadcasts a Commit Message to other nodes. The Commit Message indicates that the node has committed the proposal.
* Decide Phase:
    When a Validator node receives a sufficient number of Commit Messages (typically 2f + 1), it executes the proposal and applies the result to its local state.

#### ​Algorand

Algorand is an efficient and decentralized blockchain protocol that adopts Pure Proof of Stake (PPoS), cryptographic sortition, and BFT to ensure decentralization and efficiency. It was proposed by Silvio Micali, a professor at the Massachusetts Institute of Technology, in 2017. Algorand aims to address the performance, scalability, and decentralization issues in traditional blockchain technologies. Through its unique consensus mechanism and cryptographic techniques, it achieves a fast, secure, and permissionless blockchain network. 

The consensus process of Algorand is divided into the following stages: 

* Encrypted lottery:
    Each token holder is randomly selected to become a Proposer or Verifier based on the number of tokens they hold through an encrypted lottery algorithm. The lottery process is probabilistic, and the more tokens one holds, the higher the probability of being selected.
* Block proposal:
    The selected Proposer is responsible for proposing a new block and broadcasting it to other nodes in the network.
* Soft Vote:
    Verifiers validate the proposed block and vote for the most effective one. Through multiple rounds of voting, the network reaches a preliminary consensus on the block.
* Certify Vote:
    Verifiers make a final confirmation of the block that has reached a preliminary consensus to ensure its validity and security. Once the block receives a sufficient number of votes, it will be added to the blockchain.
* Fast finality:
    Algorand's blocks are immediately confirmed upon generation, without the need to wait for multiple block confirmations, ensuring the fast finality of transactions.

#### Avalanche

Avalanche is an innovative blockchain protocol proposed by Team Rocket in 2018 and officially released in 2020. It aims to address the scalability, decentralization, and security issues of traditional blockchains. Through a unique consensus mechanism and architectural design, Avalanche achieves a high-performance, low-latency blockchain network. 

Avalanche adopts the Avalanche Consensus protocol, which is a consensus mechanism based on metastability, random sampling, and BFT algorithm. Its core idea is to quickly reach network consensus through multiple rounds of random sampling and voting. 

The features of the Avalanche protocol include: 

* Metastable state:
    In the network, nodes gradually converge to a stable consensus state through multiple rounds of random sampling and voting.
* Random sampling:
    Each node randomly selects a group of other nodes for voting, ensuring the efficiency and decentralization of consensus.
* Fast finality:
    The avalanche protocol can confirm transactions within 1-2 seconds, providing a near real-time transaction experience.

#### HotStuff

HotStuff is an efficient BFT algorithm proposed by the VMware Research team in 2019. It is designed to address the complexity and performance bottlenecks of traditional BFT algorithms, such as PBFT, and is particularly suitable for large-scale distributed systems. The core idea of HotStuff is to significantly improve consensus efficiency and scalability through a chaining structure and pipelining design. HotStuff is the core consensus algorithm of Facebook's blockchain project Libra (now Diem). 

The consensus process of HotStuff is based on a chain structure and pipeline design. Its core idea is to divide the consensus process into multiple stages and connect these stages in a chain structure. The consensus process of HotStuff includes the following stages: 

* Propose Phase:
    The leader node proposes a new block and broadcasts it to all replica nodes.
* Prepare Phase:
    Upon receiving the proposal, the replica nodes verify its validity and broadcast a prepare message to other nodes. The prepare message indicates that the node supports the leader's proposal.
* Pre-Commit Phase:
    When a node receives a sufficient number of prepare messages (typically 2f + 1, where f is the number of malicious nodes), it broadcasts a pre-commit message to other nodes. The pre-commit message indicates that the node is ready to commit the proposal.
* Commit Phase:
    When a node receives a sufficient number of pre-commit messages (typically 2f + 1), it broadcasts a commit message to other nodes. The commit message indicates that the node has committed the proposal.
* Decide Phase:
    When a node receives a sufficient number of commit messages (typically 2f + 1), it executes the proposal and applies the result to its local state.