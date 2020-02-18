---
title: "Cryptography and Cryptocurrencies"
date: 2018-06-01
permalink: /notes/2018/06/01/crypto-notes
--- 

## BitCoin, Cryptocurrencies, and Cryptography
### Cryptographic Hash Function - mathematical function
  - Input: any string
  - Output: fixed size output
  - Efficiently computable
  - Security Properties:
    - **Collision-free** - no x and y such that H(x)=H(y) (collisions do exist, but they cannot be found) => allows us to use Hash as *message digest*
    - **Hiding** - given H(x), no way of finding x
        - We achieve this by concatenating a random string r so that given H(r|x), it is difficult to find x => allows us to achieve *commitment*
    - **Puzzle-friendly** - for every possible output y, one cannot find H(k | x) = y

#### Bitcoin's Hash Function: SHA-256
```
1) Cut message into chunks of 512 bits (with some padding at end of the form 10*)
2) Take IV (256 bit number) and 1st chunk through compression algorithm
3) Take result and 2nd chunk through compression algorithm
4) Take result and 3rd chunk through compression algorithm
...
5) Output 256 bit hash
```

### Hash Pointers and Data Structures
  **Hash pointer** - a pointer to where some info is stored and crypto hash of the info
    - can get info back
    - can verify it hasn't change
    - we can use hash pointers to create data structures

**Blockchain** - linked list where we use hash pointers instead of normal pointers => use case is a tamper-evident log

**Merkle Tree** - binary tree with hash pointers => use case is tamper-evident tree
  - can verify membership in O(log n) time/space

Generally, we can use hash pointers with any data structure using pointers that is acyclic

### Digital Signatures
  - Only you can sign, but anyone can verify
  - Tied to a particular document
API for digital signatures
```
(sk, pk) = generateKeys(keysize) //randomized algo
// sk is secret key, pk is public key
sig = sign(sk, message) //randomized algo
// sign the message using secret key
isValid = verify(pk, message, sig)
// verify the message using the signature and public key
```

**EDCSA** - Bitcoin's Digital Signature Algorithm requires good randomness

### Decentralized Identity Management
Another useful trick is to think of public keys as identities.
So `sk` can be used to 'speak' for the identity and `pk` is the 'name' for the identity (in BitCoin, these are called *addresses*).

=> anybody can make an identity whenever they want and they can make multiple identities with no centralized control

### A Simple Cryptocurrency
### Goofy Coin
  - Goofy can create new coins:

  ```
  signed by Goofy's pk
  CreateCoint(uniqueCoinID)
  ```

  - A coin's owner can pass it on:

  ```
  signed by Goofy's pk
  pay to Alice's public key: H(pointer to Goofy's coin data structure)

  ```

Rules:
  - Goffy can create new coins by signing a statement that he's making a new coin with a unique coin ID
  - Whoever owns a coin can pass it on by signing a statement saying to pass on to person X
  - Verify validity of a coin by following the chain and all verifying all signatures along the way

**Double Spending Attack** - a problem with Goofy coin is that if Alice passes coin onto Bob AND Chuck, both have a right to the coin!


### Scrooge Coin
  - Scrooge publishes a history of all transactions with a coin via a block chain signed by Scrooge
      - Each transaction corresponds to a node in linked list
      - Solves double-spending problem since we have entire history of a coin

Two types of transactions:
  - CreateCoin creates new coin
    - Each call has a transaction ID, a list of coins created with values and public keys of recipients
    - coinID<transID>(num) points to coin
  - PayCoins consumes and destroys some coins, and creates new coins with same total value
    - Each call has a transaction ID, a list of consumed coin IDs, and a list of new coins created (with values, recipients), and a list of digital signatures of all owners of consumed coins
    - Valid iff: consumed coins valid, not already consumed, total value in = total value out, signed by all owners of consumed coins
Note that this introduces the concept of *immutable* coins.

**Decentralization Problem** - a problem with Scrooge Coin is everything is centralized around Scrooge! Can we decentralize Scrooge coin?


### Decentralization in BitCoin
**Distributed Consensus Protocol** - fixed number of *n* nodes with an input value
  - Protocol terminates with all correct nodes decide on same value
  - Value must be one of values proposed by correct nodes

In BitCoin, the distributed consensus protocol is achieved by broadcasting a transaction to all the nodes in the peer-to-peer network. Then, the nodes want to reach a consensus on
what transactions were made and in what order they were made.

So in BitCoin, consensus could be achieved by all nodes at any given time:
  - All nodes have a sequence of block of transactions they've already reached consensus on
  - Each node has a set of outstanding transactions they've not reached consensus on
But we don't do it this way because:
  - Nodes may crash/be malicious
  - Network imperfections (e.g. not all nodes connected, latency, etc.)

BitCoin uses some assumptions/additives to achieve distributed consensus:
  - Incentives
  - Embraces randomness - consensus does not have specific start/end, occurs over long period of time

### Consensus without identity via the block chain
**Why doesn't BitCoin use identities?**
  - *Sybil Attack* - one adversary creates multiple nodes to create appearance of many participants
  - Pseudonymity is a goal of BitCoin

Assume we can pick a random node by giving all nodes a token and choosing a single token.

**Implicit Consensus**
  - In each round, a random node is picked
  - random node will propose next block
  - other nodes implicity accept (by extending it) or reject (by ignoring the block)

**BitCoin's Consensus Algorithm**
```
1) New transactions broadcast to all nodes
2) Each node collects new transactions into a block
3) In each round, random node gets to broadcast its block
4) Other nodes accept block iff all transactions in block are valid
5) Nodes express acceptance by including its hash in next block they create
```
- Protection against invalid transactions is cryptographic, but enforced by consensus
- Protection against double-spending is purely consensus
  - Never 100% a transaction ends on consensus chain => but we have exponential probability guarantee (good after ~6 transactions)

### Incentives and proof of work
Incentivize honest nodes by paying them in BitCoins
  - Incentive 1: Block reward
    - Creator of block gets to include special coin-creation transaction in block and choose recipient of next transaction
    - Fixed at 25 BitCoins currently, halves every 4 years
      - This is the only way BitCoins can be created! => puts a cap on number of BitCoins to be 21 million, running out by 2040
  - Incentrive 2: Transaction fees
    - Creator of transaction can choose output value less than input value and remainder goes to block creator (like a tip)

**Proof of Work** - Approximate selection of random node in proportion to a unmonopolizable resource (e.g. computing power or ownership). We achieve this concretely by forcing nodes to
find a *nonce* such that H(nonce | prev_hash | t1 | ... | tn) is small (nodes simply guess nonces randomly). 

So all nodes are competing at once to find a nonce that succeeds. Then, the first node that succeeds in their quest gets to propose the next block!
  - 1) Difficult to compete
    - In reality there are 10^40 nodes/block so P(finding nonce) = 1/10^40
    - Hence, only some blocks try to solve Hash Puzzle - BitCoin Miners
  - 2) Parameterizable cost
      - As computing power increases, scale size of Hash every 2 weeks
      - Enforce that average time between proposals ~10 min (computed via Poisson process)
  - 3) Trivial to verify
    - Other miners simply check that H(nonce | prev_hash | t1 |...| tn) = target

### Mining Economics
One should mine if:

mining reward (block reward+tx fees) > hardware+electricity costs

## Mechanics of BitCoin

### Bitcoin Transactions
#### Option 1: An Account Based Ledger
```
| Create 25 coins and assign to Alice
| Transfer 17 coins from Alice to Bob
| Transfer 8 coins from Bob to Charlie
| ...
```
Disadvantage: To check if Alice can transfer 15 coins to David, we need to look backwards to the beginning of time.

#### Bitcoin's transaction based ledger
```
| 1: Input: None, Output: 25->Alice // transaction has an ID, Input, Output
| 2: Input: 1[0], Output: 17->Bob, 8->Alice
| ...
```
Now we can just check a finite number of steps backwards. We implement id's using Hash Pointers.

Low-level transaction:
  - Metadata: housekeeping, hash of transaction, lock time (cannot publish until time x)
  - Inputs: an array of inputs
    - Each has: previous transaction and signatures
  - Outputs: an array of outputs
    - Each has: value and scriptPubKey

### Bitcoin Scripts
`scriptPubKey` - Output addresses are **scripts** in Bitcoin.

`scriptSig` - Input addresses are also scripts in Bitcoin.

Input + output addresses combined create a valid script that runs successfully to claim a bitcoin.

#### Script Properties
  - Simple, compact
  - Support for cryptography
  - Stack based
  - Limits on time/memory
  - No looping
An example script

```
<sig>
<pubKey>
OP_DUP // take value at top of stack and duplicate it
OP_HASH160 // take top value and compute cryptographic hash
<pubKeyHash?> // hash key sent by person wanting to claim coins
OP_EQUALVERIFY // are top two values equal?
OP_CHECKSIG // check signature is valid
```

Not much creativity in what people use - this is pretty much the only script used!

**Proof of Burn** - script that can never be redeemed

**Pay-to-script hash** - receiver of coins tells sender to pay to a certain hash of a script rather than multiple public keys

#### Applications of Bitcoin Scripts
- Escrow Transactions
  - By Pay x to 2 of 3 of two participants and a judge
- Green Addresses
  - If receiver is offline, sender can send to a green address (like a bank)
- Efficient Micropayments
  - Spender signed minor payments, but receiver only signs last one
i
### Bitcoin Blocks
Consists of a Hash Chain of blocks where each node has a pointer to the previous block and a hash to a Merkle Tree of transactions.
Specifically:
  - Block Header: hash, prev_block, time, mining puzzle information, etc.
  - Merkle Tree of Hashes
i
### Bitcoin Network
Bitcoin's P2P Network
  - Ad-hoc protocol (runs on TCP port 8333)
  - Random topology
  - All nodes equal
  - New nodes can join at any time

Joining the Network - New node sends `getaddr()` message to seed node and repeat recursively

Transaction propagation (flooding algorithm) - each node sends to all it's neighbors unless it already has seen the hash. Hence, we only relay if:
  - 1) Transaction valid with current block chain
  - 2) Script matches a whitelist to avoid unusual script
  - 3) Haven't seen the hash before
  - 4) Doesn't look like a double-spend
Race Conditions can exist! Transactions/blocks can conflict! In this case, accept what you hear first (so network position matters).

Block propagation is nearly identical, except:
  - 1) Block must have *all* transactions validated
  - 2) Block builds on longest current chain

Storage Requirements
  - A fully validating node must always be on the network and maintain a set of unspent transaction outputs => 20GB of RAM
  - Lightweight nodes (SPV nodes) aren't fully validating. They store block headers only and trust fully-validating nodes.

### Limitations and Improvements
  - Slow throughput
  - Fixed values for coins
  - Cryptography primitives fixed

Updating Software of Bitcoin
  - *Hard-forking* changes in Bitcoin results in separation of block chains where old nodes never catch up
  - *Soft-forking* - adding features making validation rules stricter
    - Examples: new signature schemes, extra per-block metadata

## How to Store and Use Bitcoins
To spend a Bitcoin, we need:
  - Some info about public blockchain
  - How to store and manage secret signing key - 3 goals: availability, security, convenience

*Hot Storage* - online money (money kept on you) vs. *Cold Storage* - offline money (money in bank)
  - Hot Storage implemented by keeping secret keys on a phone/computer


## Bitcoin Overview
Properties:
  - Decentralized, P2P
  - Cryptocurrency

### Cryptographic Hash Function
Transforms arbitrary message into a fixed-length digest so that:
  - Consistency - same output for one input 
  - Collision resistant - hard to find 2 distinct inputs mapping to one output
  - Hide information about input given output

### Digital Signature
Alice wants to 'sign' a document (give her stamp of approval):
  - Generate signing key (sk) and verification key (vk)
    - sk is private, vk is public

Signing Process:

Alice uses sk to sign message M generating signature sM

Verification Process:

Anyone can use message M, signature sM, and vk to output yes/no

### Transaction Records
Alice (vk_a, sk_a) wants to transfer 50 bitcoins to Bob (vk_b, sk_b)
  - Input: Sender (vk_a) includes previous transactions' digests 
      - Ex: Alice includes D_x, D_y, D_z digests indicating she has 65 bitcoins
  - Output: 
    - Recipient (vk_b) and number of bitcoins (50)
    - Sender (vk_a) and number of bitcoins in change (14)
    - Use sk_a to sign inputs and outputs and append to end of transaction record
  - Transaction is broadcast to everyone in P2P network
    - Transaction fee (1 coin) goes to bitcoin miner for broader validation to prevent double spending

### Proof of Work
Proof that someone has performed serious computational effort that can be easily verified.
  - Challenge string c combined with prover's response p
  - Fed into cryptographic hash function to generate output starting with 0's ending normally
  - => Computationally expensive (~10 min) to find response p

### Transaction Block Chain
  - Broadcasted transactions coalesced into a transaction block (multiple transactions) by pairwise cryptographically hashing to a digest
  - New digest D added to global transaction block chain containing all previous digests
  - Concatenation of D and global block chain turned into challenge string C
  - Bitcoin miner provides proof of work with response p and broadcasts to network
      - Miner adds coin-based generation award + transaction fees for all transactions in block
      - In case of multiple miners solving, network will adhere to chain with most work associated with it

=> The proposal mechanism in Bitcoin is by taking a block, turning it into digest D, combining with all previous digests into challenge string. Miner searches for response p to concatenate with challenge string to produce 0000...xxxx.

### Bitcoin Money Supply
Controlled by 2 mechanisms: Calibration of coin-based generation reward and Calibration of proof of work

Bitcoin money supply is hard-capped at 21,000,000 => coin based generation will run out!
  - Coin-based generation halves every time 210,000 coins generated
  - In January 2009, reward was 50 bitcoins, currently 25 bitcoins

=> Transaction fees become more and more important over time (i.e. larger transaction fees will be added to incentivize miners to include in a block)

Calibrating Proof of Work
  - For every 2,016 blocks generated should take ~2 weeks to generate

### Security of Transaction Block Chain
If there is a conflict between global block chains => everyone works off longest chain (i.e. chain with most work associated with it as defined by sum of difficulty scores)

If Alice attempts to double-spend (give 5 coins to Bob AND Carol)
  - Alice creates fork in chain: 2 transaction blocks (one w/ transaction with Bob, another with Carol) and broadcasts to network
  - Alice must solve proof of work multiple times to outperform previously created chain

## Ethereum
Ethereum allows people to write decentralized applications using blockchain technology.

In Bitcoin's case, the blockchain is a ledger of transactions and predefined operations. Ethereum is a programmable blockchain allowing users to create their own operations.

How does Ethereum achieve this? Via the **Ethereum Virtual Machine** - a large decentralized computer with millions of *accounts* that can 1) maintain an internal database 2) execute code 3) talk to each other.

Accounts:
  - **Externally owned accounts (EOAs)**: accounts associated with a private key => if you have private key you can send messages/ether from it
  - **Contract** - an account with its own code and controlled by its own code (owned by no one)

An EOA can send a transaction to another EOA or Contract. 
  - recipient EOA can receive ether
  - recipient Contract simply runs its code

Contracts can do 4 things:
  - 1) data storage (e.g. a currency or membership in organization)
  - 2) *forwarding contract* - an EOA with more complicated access policy (e.g. multiple private keys, requiring certain messages, etc.)
  - 3) manage an ongoing contract between users
  - 4) provide functions to other contracts (i.e. a software library)

There are 2 types of transactions:
  - 1) Sending transaction
    - Receiving Address
    - Ether amount
    - Data bytearray
    - Other params
    - Digital signature associated with pk of sender
  - 2) Contract creating transaction
    - like a sending transaction without a receiving address

**State Machine** - the EVM uses a stack-based byte code to create states and execute instructions. We can use higher-level languages such as Solidity, LLL, Serprent, or Mutan which gets compiled and runs on the EVM.

**Gas** - 
Every operation executed in the EVM is executed by ALL nodes => computational steps expensive => every computational step has a *gas* (price)

**Ethereum's Blockchain**

Each block in the blockchain has a `prevhash`, `stateroot`, `timestamp`, `number`, etc. The `stateroot` is the root of a *Patricia tree* containing all accounts in the EVM. Each node in the tree consists of `[account_nonce, ether_balance, code_hash, storage_root]`
  - `account_nonce`: number of transactions sent from the account
  - `ether_balance`: ether balance of account
  - `code_hash`: hash of the code if the account is a contract and "" otherwise
  - `storage_root`: root of yet another Patricia tree which stores the storage data

Every minute, a miner produces a new block containing a list of transactions since previous block and root hash of new Patricia tree representing state. The miner receives an ether reward for creating the new block.


## Casper & Smart Contract Consensus

### Proof of Work vs. Proof of Stake

**Proof of Work** - a piece of data that is compuatationally expensive to find => rewards miners who solve difficult mathemtical problems that determines next block

Bitcoin and most Altcoins use Proof of Work, where miners vote on a blockchain via hash power/computational capability

**Proof of Stake** - creator of new block is deterministically chosen dependent on wealth/stake

Ethereum plans to pivot towards a Proof of Stake protocol where validators are given the ability to vote on a blockchain where their power is proportional to the amount of Eth they have

PoS is better than PoW because it is 
  - a) faster 
  - b) reduces electricity usage 
  - c) scalable

**Casper's PoS** 
  - *Accountability* - a validator who violates a rule can be found and penalized 
  - *Dynamic Validators* - ability to change validator set
  - *Defense* - defend against 1/3 of validators dropping out
  - *Modular overlay* - easy to overlay upon existing PoW architecture

