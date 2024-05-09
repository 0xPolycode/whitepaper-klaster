# Klaster Whitepaper

*Interchain Commitment Layer*

## Intro & rationale 
Blockchain, born out of the desire to create a global decentralized currency - Bitcoin, has evolved a lot over the fifteen years since it's been developed. The first evolution came in the form of smart contract blockchains. Recognizing the limitations of the "single-app" model, the "world-computer" model of Ethereum was born. This has allowed developers to build trustless & trust-minimized applications and has resulted in the birth of several new industries - most notably DeFi and NFTs. 

## Scalability challenges
However, while Ethereum has sucessfully solved for the "programability" aspect of blockchain, many challenges have remained to be solved. The most notable of those is - scalability. Ethereum blockchain is notoriously unscalable - with a maximum throughput of ~15 transactions per second. In times of market demand, this can raise the fees for a simple "money transfer" action to well over $50. For more complex transactions, these fees can reach into the $100 - $1000 range. This issue has made DeFi the playground of the well-off and has stood as a barrier towards decentralized services reaching mainstream adoption.

### Scaling solutions 
The problem of blockchain scalability has had many solutions, some more sucessfull than others. This document will not serve to iterate over all of them, but just mention a few meaningful examples. First example were the various alerantive blockchains, which operated with a few masternodes processing the majority of transactions. Some examples include Polygon PoS & Binance Smart Chain. The second category were blockchains operating on a different model from Ethereum - impelmenting alternate virtual machines, sharding and many other scaling techniques. Some examples in this category include NEAR Protocol & Solana. The third category, and the one this paper will focus on the most are so-called "rollups". Rollups are attached to a "parent" blockchain and use some form of cryptography to "inherit" a part of all of the security of the "parent" blockchain. Most rollups today can broadly be split into two categories - **Optimistic Rollups** and **Zero Knowledge Rollups**. Some notable examples of rollups include Optimism, Base, Arbitrum, zkSync, Polygon zkEVM, Scroll, etc... 

With the methods outlined in the text above, blockchain fees can be reduced to <$0.01 per transaction, which unlocks the potential of blockchain to a much wider audience. 

### Data availability
Coming back to rollups - in this paper we will focus on Ethereum-aligned rollups ("EVM Rollups"). In order to achieve the functionality promised by those rollups, a lot of changes needed to be done at the protocol level. 

Since neither Ethereum nor its associated virtual machine - the EVM - were originally built with rollups in mind, the developers started adapting the core blockchains and building novel solutions to support functionality of rollups. The most notable one of these solutions was the appearance of "Data Avalibility" layers - protocols which store the data required by rollups to ensure the security of validated transactions. Some of these protocols include Celestia and NEAR. Even Ethereum itself adapted to support rollups - with the ERC4844 improvement - called "proto danksharding". While technically different, all of these protocols functionally achieve the same thing - they provide a cheap but ephemeral storage for rollup data - a technical prerequisite for rollups to hit the <$0.01/tx goal. 

## Usability challenges
While scalability has been touted as the biggest challenge towards the mainstream adoption of blockchain technology, it is by no means the only one. The usability of many blockchain architectures is substandard. Again, this paper will explore the Ethereum (EVM) ecosystem, but similar conclusions can be applied to other architectures.

### Transaction Fees (Gas)
One of the core features of smart contract blockchain architectures is the concept of "gas". Gas is used to pay for transactions and is usually denominated in a coin which is native to the blockchain on which the transaction is being executed on. On Ethereum, this coin is Ether (ETH). Gas is a necessary requirement for smart contract blockchains. It prevents malicious users griefing the nodes which validate the transactions by running expensive computation or infinite loops. However, gas introduces complexity when people are interacting with tokens which are not native to the blockchain on which the transaction is being executed on. 

One example of such a token is _USDC_. This is a token, deployed on all major programmable blockchain networks which is tied 1:1 to the United Stated Dollar. Many users interact with these tokens, since they're looking to escape from the value volatility of crypto-only tokens, such as ETH or BTC. However, when a user is sending USDC to someone, they need to have not only USDC, but also the native currency. So in the case of sending ETH on Ethereum, the user must have USDC and ETH. 

This means that users must continually maintain the balance of ETH and "seed" every new address that they create with an initial ETH balance. For new users, this is another friction point which requires a learning curve and disincetivizes adoption. 

This problem has been explored and approached through many standards. On Ethereum, it's mainly solved by Account Abstraction teams working on standards such as ERC4337 & ERC3074. Later in this paper, we will present a novel approach to this problem.

### Elliptic curves
While interacting with smart contract blockchains, users must commit signed transactions to the blockchain. These transactions are based on asymetric cryptography, notably - elliptic curve cryptography. Each blockchain can choose their own curve, but the most popular one is `secp256k` used by Bitcoin, Ethereum and all EVM rollups. For example, Solana uses `curve25519` and thus validates the cryptographic commitments differently.

Elliptic curve cryptography isn't specific to blockchains only, and many interesting solutions exist which implement different elliptic curves. One such example is _Passkey Authentication_, which uses `ES256` as its authetntication curve. In native blockchain implementations, these alternate methods are not supported - limiting the usability of blockchains.

### Account recovery & protections
Handling crypto in self-custody is off-putting to many users since they must take care of preserving their own accounts and protecting themselves from losses. If the users lose access to their private key, they lose access to their funds. While "crypto natives" have adjusted to this way of thinking - it remains a large hurdle towards adoption of crypto self-custody. 

### Smart contract accounts
Similar to solving the gas fee problem, the solutions to the usage of elliptic curves and account recovery are _Smart Contract Accounts_ (SCAs). These accounts can attach arbitrary execution logic to any transaction and as such they can improve the usability of blockchain systems substantially. Some examples of SCAs include:

* Gas abstraction & sponsorship: Allowing the users to cover on-chain transcation fees with non-native assets _or_ allowing blockcahin applications to sponsor the gas costs for their users.

* Account recovery: The ability for users to recover funds if they loose access to their private key. This can be done through a trusted recovery provider or through methods such as social recovery.

* Custom elliptic curves: The ability for users to use passkeys or other cryptography which is not native to the core blockchain. 

* Custom execution logic: The ability for users to customize the execution logic of their blockchain accounts. This includes setting spending limits, authorizing multiple signers, requiring a form of two-factor authentication, etc...

## Interchain Commitment Layer

In this paper, we are presenting a model for creating a P2P network, which is able to operate on top of _multichain smart accounts_. It combines the scalability benefits of a rollup-centric future, with the usability benefits of smart account systems, while remaining backwards-compatible with existing smart contracts and dApps.

Klaster enables users to use a single signature to execute one or more transactions across multiple independent blockchains. The network is also _conditional_ - it can add execution logic to the transactions.

### Decoupling transaction commitment and execution

When the architecture of blockchain networks was designed, the design considerations assumed a monolithic blockchain environment, where - even if multiple blockchains existed - their intercommunication would be limited. This is easy to notice from initial Ethereum 2.0 design goals - implementing sharding on top of a single blockchain.

The industry, however, has moved in a direction of many, sometimes app-specific, blockchains which need frequent interchain communication. This has opened up the space to transfer value between them and many bridge and cross-chain swap providers have entered the market.

However, one area remains neglected - transaction signing & commitment flows. In the current model, the user needs to sign a new transaction every time they're interacting with a new blockchain. The transactions themselves, even in account abstraction models, are single-chain by default. This design is inadequate for the multichain future. 

Klaster has built a system - which can _decouple_ signing & commiting actions to the blockchain from the execution of the transactions themselves. Users are able to sequence multiple transactions, across multiple blockchains and have them executed by signing just a single signature.

Beyond this, Klaster enables cryptoeconomically protected transaction scheduling - having nodes commit to execute certain transactions after some amount of time has elapsed. 


This _Interchain Commitment Layer_ enables dApp developers to encode multichain actions as signle signatures, abstracts away gas payments and paves the way for a future where users interact with apps across fully independent blockchains, without even realizing it.

This enables many interesting use-cases for dApp and wallet developers:

- Execute a transaction on one chain, pay gas on another. 
- Open multichain bridge + execute flows. These enable users to efortleslly onboard to new rollups, without ever explicitly "bridging" their assets.
- Asset / balance abstraction. Apps can access funds that users have on blockchains which are not the same as the one the app is deployed on. The app would simply encode a transaction "bridge from chain A and execute on chain B". Unlike existing (bridge) solutions, this can all be done off-chain (in the SDK, or on frontend) - without the need to upgrade existing smart contracts.
- Full chain abstraction. Even though more work needs to be done on this front, Klaster can serve as a base layer upon which developers build completely chain abstracted apps. For example - a version of AAVE where you can deposit USDC from multiple blockchains onto a signle one, with a single transaction. Once multichain tokens start to gain market adoption, Klaster can enable true chain-abstracted experiences - where users don't see and don't care - which blockchain they're interacting with.



### High level overview

![Klaster Diagram](https://github.com/0xPolycode/whitepaper-klaster/blob/master/assets/network-diagram-lnfn.png?raw=true)

The Klaster network works by allowing `Klaster Nodes` which stake tokens on the `StakingManager` contracts on the supported blockchains to _commit_ signed `Transaction Bundles` by users. These transaction bundles can be multichain, multiaccount and can include scheduling information (execute after certain block height). The nodes are incentivized to execute the bundles by taking transcation fees. The nodes must cryptographically commit to execute the entire transaction bundle. If they fail to execute after commiting, they get slashed. 


#### Transaction execution flow
1) Dapp/wallet creates a `Transaction Bundle`. The bundle contains all the required information for the nodes to execute a sequence of tranactions across multiple blockchains. It contains:
    1) A list of transactions bundled as ERC4337 `UserOp` objects, with an extra information. The extra information on top of `UserOp` object is:
        * `ChainID` on which they want the transaction to be executed.  
        * `Salt` parameter, which determines on _which_ multichain smart account the transaction needs to be executed on.
        * `Minimum Block Height` stating that the node will not execute the transaction before some lower-bound block height.  
    2) Payment infomation. The user communicates a `PaymentInfo` object in which they specify how they would like to cover the cost of the transaction. This can include native gas payment, payment in a token (ERC20 or otherwise) or even an off-chain payment structure (pay by credit card, prepay trancations, subsidised transactions, ...)

2) The dApp/wallet is connected to a _light node_ and sends the transaction bundle to that node. 

3) The light node picks up the request and forwards it to the _full nodes_. 

4) The full nodes respond with a `Transaction Quote`, which contains:
    1) Information on the transactions being executed
    2) Information on how much the node will charge the user to execute the entire bundle.
    3) Maximum block height for every `UserOp` the node has received. This sets the block height for every commited transaction, after which - if the node hasn't executed the transaction - it will get slashed.

5) The Light Node gathers all of the quotes and uses some strategy to select the best quote. It can be based on price, reputation, expiry time, etc... It forwards this quote to the dApp/Wallet

6) dApp/Wallet gets the quote from the light node and if it's acceptable it connectes directly to the full node.

7) The full node receives the notice and uses the transactions in the `Transaction Quote` object to generate a Merkle Tree of transactions. It then signs the root hash of the Merkle Tree with its private key. This singed Merkle Tree root hash can later be used to slash the node, if it doesn't execute transactions under conditions laid out in the `Transaction Quote` it has signed.

8) dApp/Wallet takes the signed Merkle Tree root hash and signs it again with the user private key. It then sends the double-signed Merkle Tree root hash to:
    1) The full node which has commited to the transaction. This enables that node to start executing the `Transaction Bundle`. 
    2) The light node. The light node will then send the signed transcation to other full nodes, which at that point act as _Witness_ nodes, confirming that the user has indeed sent the signed transaction to the Klaster network. This is required due to slashing conditions. 

9) The node executes the transactions as it has commited to _or_ it fails to execute the transactions and gets slashed. 

![Transaction Flow](https://github.com/0xPolycode/whitepaper-klaster/blob/master/assets/process.png?raw=true)

### On-chain execution validation

To enable nodes to participate in the transaction flow, each supported blockchain must have a `KlasterEntryPoint` singleton smart contract deployed(Note: This is a different contract than the EIP4337 `EntryPoint` contract). 

The `EntryPoint` contract validates the condition for the transaction execution of `UserOp` objects contained within a `TransactionBundle`.

#### Deriving wallets
The Klaster protocol works by deriving a _Smart Contract Account_ (SCA) from an address generated by some private key. Alternatively, SCAs can be derived from some other cryptographic proving scheme, but this will remain outside of the scope of this paper - as it's the same model that existing 4337 protocols use (Klaster uses Biconomy SCAs in the background).

The core of this assumption is the deterministic contract creation with the `CREATE2` opcode, which allows Klaster to assume that it'll be able to deploy a SCA on all supported chains - even if the user never interacted with that chain before (and thus doesn't have a deployed account there).

This allows the nodes to work efficiently across multiple blockchains, being assured that there exists an account which they can effectively execute transactions against.

#### Validating the UserOp
When one or more `UserOp` objects are commited to the `KlasterEntryPoint` contract, they pass through a series of validations. These are more-or-less inherited from the EIP4337 model, with few additions specific to the Klaster Protocol model. The `EntryPoint` contract will check:

1) Does the node executing the transaction have enough stake to be allowed to execute the transaction.
2) `ecrecover` the Merkle Tree root hash and verify that the user has signed and approved this `UserOp`
3) Derive the intended wallet
4) Call the `execute` function on the derived wallet

### Slashing conditions

In order to achieve the desierd effects, the Klaster network uses cryptoeconomic security guarantees in the form of _staking_ and _slashing_. In order to keep the nodes compliant to the tasks they're entrusted to do, the network requires them to post a *stake* in the $TBD token. The node operators post the token to the `StakingManager` smart contracts on the chains on which they wish to execute transactions. The EntryPoint smart contracts will not accept executions by nodes which don't have a stake in the contract. 

The nodes are slashed for commiting to a transaction and then not executing it. One of the big benefits of the Klaster Protocol is the fact that the nodes never take custody of user funds nor do they have the ability to execute anything that the user hasn't explicitly signed. This achieves all major goals the protocol aimed to solve, while making the node implementation and slashing conditions surprisingly elegant.

**Condition**: The node has commited itself to execute a `UserOp` on some chain with a maximum block height of `BH`. The chain has reached `BH`, but the node hasn't executed the `UserOp`. This has triggered a `challenge period` in which the node can be slashed.

**Considerations**: The node not executing the commited transaction can be caused by the node itself beign malicious _or_ if the user has not returned the signed commitment hash to the node. The slashing mechanism protects both the user from the node and the node from the user.

#### Slashing flow
1) The _Witness_ nodes track onchain information and realize that the transaction hasn't been executed. 

2) The _Witness_ nodes take the dual-signed Merkle Root hash and commit it to the blockchain `StakingManager` signleton contract.

3) For every dual-signed hash the `StakingManager` receives it increments the `total slashing commitment` variable by the amount of stake the _Witness_ node had in the `StakingManager`. Once the `slashing commitment` variable reaches some `threshold` value set by the Klaster protocol, it slashes the offending node and distributes its slashed stake to the _Witness_ nodes which participated in the slashing.

## Security assumptions

In this part, we'll present a short, non-exhaustive, overview of the security assumptions for the protocol and slashing:

* **Malicious Slashing** - A fair node can be slashed if certain (probabilistically unlikely) conditions are met:
    * The user and _Witness_ nodes could collude to slash the node. This process would work in the following way:
        1. The user requests a quote from the full node
        2. The user accepts the quote
        3. The full node signs the node commitment
        4. The user never signs the quote and waits for the `max_block_height` that the full node has been reached. 
        5. After that, the user sends the signed hash to the _Witness_ nodes
        6. The _Witness_ nodes commit enough stake to slash the fair node.

        The assumption, in this case, is that more than a set `threshold` amount of stake in the network will be malicious nodes. Further examination must be used to determine which `threshold` values are good in this case. Assuming `threshold` is set to some high amount - e.g. 51% - this should not be a problem.

* **Low stake chains** - Since the stake is split across mutliple blockchains, a situation may arise where certain chains have a very low stake commited to them. This would increase the probability of a malicious actor taking over the `threshold` level of stake, creating useless `UserOp` requests and using the malicious staking mechanism metioned earlier to slash the fair nodes on that chain. 

    One solution to this would be to have a unified stake, but this would depend on some form of state proof validation to communicate the fact that a certain action was not commited on `ChainA` to a unified `StakingManager` contract on e.g. Ethereum.  

