Learning Substrate here

# Useful Links
[Intro to Substrate](https://www.youtube.com/watch?v=-6BBIr-DmI4&t=5s)

[Substrate docs](https://docs.substrate.io/learn/)

# Blockchain basics

**A blockchain essentially a state machine**

- A blockchain is a decentralized ledger that records information in a sequence of blocks
- Nodes communicate with each other to form a decentralized P2P network
- In most cases, users interact with a blockchain by submitting a request that might result in a change in state

## Consensus
Consensus model/algorithm: The method that blockchain uses to batch transactions into blocks and t select which node can submit a block to the chain

## Smart Contract
A program that runs on a blockchain and executes transactions on behalf of users under specific conditions 

## Blockchains Node
- Database
- P2P Network
- Block Authoring
- Fork Choice Rule
- Transaction Handling
- State Transition Function(Known as Runtime)

Substrate as a FRAMEWORK, will provide you with the above basic infrastructure, so that you can freely expend and customize any aspects of logic on this basics:

- Database Layer
- Networking Layer
- Transaction Queue
- Consensus Engine
- Library of Runtime Modules

# Substrate
Substrate is a sdk that allows you to build application-specific blockchains that can run as standalone services or in parallel with other chains

## Three Level
- Node Template (High level)
- Substrate Frame
- Substrate Core (Low level)

## Pallets and Runtime
- pallets are specific modules that can be used in runtime
- you select and combine the pallets that suit your application to compose a custom runtime

## Substrate Consensus
- Who may **author** new blocks and when?
- When are blocks considered **final**?

Substrate make the answer to  this two questions **pluggable**

## Substrate Runtime
- The runtime in the **block execution logic** of the blockchain, it is composed of pallets
- Application logic that makes this chain special

## Substrate Node
