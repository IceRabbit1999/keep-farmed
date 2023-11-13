# Core Definition
## Account
- In Solana, Everything is an Account
```rust
pub struct Account {
    /// 账号余额
    pub lamports: u64,
    #[serde(with="serde_bytes")]
    pub data: Vec<u8>,
    pub  owner: Pubkey,
    pub executable: bool,
    pub rent_epoch: Epoch
}
```
# Signature: Ed25519
计算快、安全性高、生成的签名内容小的不对称加密算法

# Transaction
Every transaction contains:
- an array of all accounts it intends to read or write
- one or more instruction
- a recent blockhash
- one or more signatures

Basic Flow
1. Applications send transactions to an RPC client
2. RPC client forwards transactions to the network
3. Validators execute transactions, which invoke programs
4. Programs update account state

```rust
pub struct Message {
    pub header: MessageHeader,
    pub account_keys: Vec<Pubkey>,
    pub recent_blockhash: Hash,
    #[serde(with = "short_vec")]
    pub instructions: Vec<CompiledInstruction>,
    #[serde(with = "short_vec")]
    pub address_table_lookups: Vec<MessageAddressTableLookup>
}

pub enum VersionedMessage {
    Legacy(LegacyMessage),
    V0(v0::Message)
}

pub struct VersionedTransaction {
    pub signatures: Vec<Signature>,
    pub message: VersionedMessage
}
```
# Instruction
Every instruction is made up of 3 components:
- the intended programs'ID
- an array of all accounts's involved
- a byte buffer of instruction data

```rust
pub struct CompiledInstruction {
    pub program_id_index: u8,
    pub accounts: Vec<u8>,
    pub data: Vec<u8>
}
```

# PDA(Program Derived Address)
- Program store data in PDAs
- PDAs have no secret key
- think of PDAs as kv store: k for address, v for the data inside the account

# SPL-Token
represent all non-native tokens on the Solana Network(Both fungible and non-fungible tokens)

SPL is for Solana Program Library

# Program
- Programs on solana are a particular type of account that stores and executes instruction logic
- Solana Programs have a single entry point
- A program processes an instruction using the `program_id`, list of `accounts`, and `instruction_data` 
# Contract
## System(Native) Program
系统合约是由节点在部署的时候生成的，普通用户无法更新，他们像普通合约一样，可以被其他合约或者RPC进行调用
- System Program
- BPF Loader Program
- Vote Program
## On Chain Program
普通合约是由用户开发并部署，Solana官方也有 一些官方开发的合约，如Token、ATA账号等合约。

Solana上的合约是可以被更新的，也可以被销毁。并且当销毁的时候，用于存储 代码的账号所消耗的资源也会归还给部署者。
# 租约
租金与交易费用不同。 支付租金（或保存在账户中）以将数据存储在 Solana 区块链上。 而交易费用是为了处理网络上的指令而支付的。