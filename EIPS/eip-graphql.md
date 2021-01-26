---
eip: <to be assigned>
title: Ethereum API Using GraphQL
author: Kris Shinn (@kshinn), Raul Kripalani (@raulk), Nick Johnson (@Arachnid)
discussions-to: <URL>
status: Draft
type: Interface
category: Interface
created: 2018-12-28
---


## Simple Summary
<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the EIP.-->
GraphQL provides Ethereum clients an efficient and future proof API for modern 
application developers to interface with. It simplifies querying the node, 
is self-documenting, and provides backward compatiblity for adding new features
to the Ethereum API.

## Abstract
<!--A short (~200 word) description of the technical issue being addressed.-->
Developer experience is a central concern to grow the adoption of Ethereum based DApps. 
Success of the ecosystem requires a vibrant and active developer community building 
on the platform. The Ethereum API interface should support developers with modern 
well-understood tooling. The GraphQL specification for client server communication 
provides a modern interface for developers to adopt and opens up an entire ecosystem 
of tooling that is not currently not available with the current JSON-RPC 2.0 implementation.

## Motivation
<!--The motivation is critical for EIPs that want to change the Ethereum protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the EIP solves. EIP submissions without sufficient motivation may be rejected outright.-->
Modern applications need to filter, page, group and join data before it can useful. 
The latency introduced by this processing requires the end user to stare at loaders 
while the application can assemble all of the data needed for display. For “web3” 
technologies to gain adoption, the technology stack needs to be as good or better 
than the centralized alternatives.  With GraphQL gaining adoption in the commercial
space, the Ethereum clients should keep pace and offer similar amenities to developers.

## Specification
<!--The technical specification should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (go-ethereum, parity, cpp-ethereum, ethereumj, ethereumjs, and [others](https://github.com/ethereum/wiki/wiki/Clients)).-->
The GraphQL interface should be presented as an option on clients and should be able to
run along side the JSON-RPC. This would require a separate configuration parameters (e.g. interface port). An
example for enabling a GraphQL interface taken from the Geth implementation might be:

```
--graphql                   // enable the graphQL interface
--graphql.addr 127.0.0.1    // Provide a bind address 
--graphql.port 8765         // Provide the listener port
```

The GraphQL interface should support Query, Mutator, and Subscription formats in order
to fully support all functionality in the JSON-RPC specification.

The Query format is defined as the following:
```graphql
extend type Query {
  "Selects an account."
  account(address: Address!): Account

  "Selects a block based on either a number, hash or a tag."
  block(number: BlockNumber, hash: Bytes32, tag: BlockTag): Block

  "Selects a block based on a reference block and an offset from it."
  blockOffset(number: BlockNumber, hash: Bytes32, tag: BlockTag, offset: Int!): Block

  "Selects an arbitrary set of blocks based on their numbers or hashes."
  blocks(numbers: [BlockNumber], hashes: [Bytes32]): [Block]

  "Selects a range of blocks."
  blocksRange(numberRange: [BlockNumber], hashRange: [Bytes32]): [Block]

  "Selects a transaction by hash."
  transaction(hash: Bytes32): Transaction

  "Returns the health of the server."
  health: String!
}

"""
An Ethereum Block.
"""
type Block {
  "The block number."
  number: BlockNumber!

  "The block hash."
  hash: Bytes32!

  "The parent block."
  parent: Block

  "The block nonce."
  nonce: String!

  "The block's transactions trie root."
  transactionsRoot: Bytes32!

  "The number of transactions in the block."
  transactionCount: Int!

  "The block's state trie root."
  stateRoot: Bytes32!

  "The receipt trie root."
  receiptsRoot: Bytes32!

  "The miner's account."
  miner: Account!

  "Any extra data attached to the block."
  extraData: String

  "The cumulative gas limit of all transactions in this block."
  gasLimit: Long

  "The cumulative gas used of all transactions in this block."
  gasUsed: Long

  "The timestamp when block was mined, in seconds after epoch."
  timestamp: String

  "The bloom filter for the logs contained in this block."
  logsBloom: String

  "The mix hash for this block."
  mixHash: Bytes32

  "The difficulty of this block."
  difficulty: Long

  "The total difficulty of the canonical chain this block is part of."
  totalDifficulty: Long

  "The ommer blocks (also known as 'uncles')."
  ommers: [Block]

  "Gets a single transaction from this block, addressed by its position in the block."
  transactionAt(index: Int!): Transaction

  "Gets all transactions from this block. If a filter is passed, only the transactions matching the filter will be returned."
  transactions(filter: TransactionFilter): [Transaction]

  """
  Gets all transactions from this block as long as they involve any of the addresses specified.
  If a filter is passed, only the transactions matching the filter will be returned.
  """
  transactionsInvolving(participants: [Address]!, filter: TransactionFilter): [Transaction]

  """
  Gets all transactions from this block where the provided addresses take the indicated roles.
  If a filter is passed, only the transactions matching the filter will be returned.
  """
  transactionsRoles(from: Address, to: Address, filter: TransactionFilter): [Transaction]
}

"""
Named block identifiers.
"""
enum BlockTag {
  EARLIEST
  LATEST
  PENDING
}

"""
Input object to select a block by offset.
"""
input BlockOffset {
  number: Long
  hash: Bytes32
  offset: Int
}

"""
The storage of this contract.

Provides fluent accessors for solidity contract storage. Four data types are supported:
Dynamic arrays, fixed arrays, maps with string keys, and maps with numeric keys.

The algorithm to calculate the storage key varies for each data type.
"""
type Storage {
  "Gets the value at this storage slot."
  value(at: Int!): String

  """
  Steps into a map at this storage slot.
  keyType can be address, number, or string.
  """
  solidityMap(at: Int!, keyType: KeyType!): SolidityMap

  "Steps into a fixed array at this storage slot."
  solidityFixedArray(at: Int!): SolidityFixedArray

  "Steps into a dynamic array at this storage slot."
  solidityDynamicArray(at: Int!): SolidityDynamicArray
}

"""
A solidity map.
"""
type SolidityMap {
  "Gets the value returned by this key."
  value(at: String!): String

  """
  Steps into the map returned by this key.
  keyType can be address, number, or string.
  """
  solidityMap(at: String!, keyType: KeyType!): SolidityMap

  "Steps into the fixed array returned by this key."
  solidityFixedArray(at: String!): SolidityFixedArray

  "Steps into the dynamic array returned by this key."
  solidityDynamicArray(at: String!): SolidityDynamicArray
}

"""
A fixed solidity array.
"""
type SolidityFixedArray {
  "Gets value at this index."
  value(at: Int!): String

  """
  Steps into the map at this index.
  keyType can be address, number, or string.
  """
  solidityMap(at: Int!, keyType: KeyType!): SolidityMap

  "Steps into the fixed array at this index."
  solidityFixedArray(at: Int!): SolidityFixedArray

  "Steps into the dynamic array at this index."
  solidityDynamicArray(at: Int!): SolidityDynamicArray
}

"""
A dynamic solidity array.
"""
type SolidityDynamicArray {
  "Gets value at this index."
  value(at: Int!): String

  """
  Steps into the map at this index.
  keyType can be address, string, or number.
  """
  solidityMap(at: Int!, keyType: KeyType!): SolidityMap

  "Steps into the fixed array at this index."
  solidityFixedArray(at: Int!): SolidityFixedArray

  "Steps into the dynamic array at this index."
  solidityDynamicArray(at: Int!): SolidityDynamicArray
}

"""
An Ethereum account.
"""
type Account {
  "The address of this account"
  address: Address

  "The balance of this account"
  balance(unit: Unit): Long

  "The code behind this account"
  code: String

  "The type of this account"
  type: AccountType

  "The number of transactions this account has sent"
  transactionCount: Int

  "The storage of this account"
  storage: Storage
}

"""
A filter for logs.
"""
input LogFilter {
  """
  Only selects logs that are published under the given topics.

  Items within an inner list are combined with an AND, and items on the outer list are combined with OR.

  For example: { topics: [["0x1234...", "0xabcd..."], ["0xbcde..."]] } will return all logs published under
  topics "0x1234..." AND "0xabcd...", as well as those published under topic "0xbcde..."
  """
  topics: [[Bytes32]]
}

"""
An Ethereum transaction.
"""
type Transaction {
  "Transaction hash"
  hash: Bytes32!

  "Transaction nonce"
  nonce: Long!

  "The index of the transaction within the block"
  index: Int!

  "Sender of this transaction"
  from: Account

  "Recipient of this transaction"
  to: Account

  "Value of the transaction"
  value(unit: Unit): Float!

  "Price set for each gas unit"
  gasPrice(unit: Unit): Float!

  "The amount of gas expended in the transaction"
  gas: Long!

  "The input data to the transaction"
  inputData: String

  "The status of the transaction"
  status: TransactionStatus

  "The block the transaction is contained in"
  block: Block!

  "The logs emitted by this transaction."
  logs(filter: LogFilter): [Log]

  """
  The decoded transaction.

  This is a best-effort interpretation of the transaction data.  There may be cases where a transaction cannot be unambiguously decoded.
  For example, because some standards share function signatures, a single transaction may appear to match several different standards.
  """
  decoded: DecodedTransaction

  "Contract created by this transaction"
  createdContract: Account
}

"""
An Ethereum log.
-- I think that this should be left up to the clients since the log is dependent on the ABI definition
"""
type Log {
  "The index of this log in the block's logs array."
  index: Int!

  "The account that emitted this log."
  account: Account!

  "The topics under which this log was published."
  topics: [String]

  "The data within this log statement."
  data: String

  "The block this log was contained in."
  block: Block!

  "The transaction that emitted this log."
  transaction: Transaction!

  """
  The decoded log.

  This is a best-effort interpretation of the log data.  There may be cases where a log cannot be unambiguously decoded.
  For example, because some standards share event signatures, a single log may appear to match several different standards.
  """
  decoded: DecodedLog
}

"""
The status or outcome of the transaction.
"""
enum TransactionStatus {
  "Transaction failed"
  FAILED

  "Transaction was successful"
  SUCCESS

  "Transaction has not been mined yet"
  PENDING
}

enum KeyType {
  address
  number
  string
}

"""
A filter for transactions. Setting multiple criteria is equivalent to combining them with an AND operator.
"""
input TransactionFilter {
  "Only selects transactions that emit logs."
  withLogs: Boolean

  "Only selects transactions that received input data."
  withInput: Boolean

  "Only selects transactions that created a contract."
  contractCreation: Boolean
}

"""
Entities are a way to group related standards into one functional concept, e.g. ERC20, ERC777 refer to the 'token' entity,
the ENS standard refers to the 'domain' entity.
-- This section is probably best left up to the clients
"""
enum Entity {
  token
}

interface DecodedTransaction {
  "The entity this transaction refers to. See documentation on the Entity type."
  entity: Entity

  "The ERC standard (or official name) this transaction appears to comply with."
  standard: String

  "The name of the function invoked in this transaction."
  operation: String
}

interface DecodedLog {
  "The entity this log refers to. See documentation on the Entity type."
  entity: Entity

  "The ERC standard (or official name) this log appears to comply with."
  standard: String

  "The name of the event associated with this log (i.e. first log topic)."
  event: String
}

"""
Type of account.
"""
enum AccountType {
  CONTRACT
  EXTERNALLY_OWNED
}

enum Unit {
  "base unit"
  wei
  "1 kwei == 1_000 wei"
  kwei
  "1 babbage == 1_000 wei"
  babbage
  "1 femtoether == 1_000 wei"
  femtoether
  "1 mwei == 1_000_000 wei"
  mwei
  "1 lovelace == 1_000_000 wei"
  lovelace
  "1 picoether == 1_000_000 wei"
  picoether
  "1 gwei == 1_000_000_000 wei"
  gwei
  "1 shannon == 1_000_000_000 wei"
  shannon
  "1 nanoether == 1_000_000_000 wei"
  nanoether
  "1 nano == 1_000_000_000 wei"
  nano
  "1 szabo == 1_000_000_000_000 wei"
  szabo
  "1 microether == 1_000_000_000_000 wei"
  microether
  "1 micro == 1_000_000_000_000 wei"
  micro
  "1 finney == 1_000_000_000_000_000 wei"
  finney
  "1 milliether == 1_000_000_000_000_000 wei"
  milliether
  "1 milli == 1_000_000_000_000_000 wei"
  milli
  "1 ether == 1_000_000_000_000_000_000 wei"
  ether
  "1 kether == 1_000_000_000_000_000_000_000 wei == 1_000 ether"
  kether
  "1 grand == 1_000_000_000_000_000_000_000 wei == 1_000 ether"
  grand
  "1 mether == 1_000_000_000_000_000_000_000_000 wei == 1_000_000 ether"
  mether
  "1 gether == 1_000_000_000_000_000_000_000_000_000 wei == 1_000_000_000 ether"
  gether
  "1 tether == 1_000_000_000_000_000_000_000_000_000_000 wei == 1_000_000_000_000 ether"
  tether
}
```

## Rationale
<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->
The existing JSON-RPC as well as other ReST based abstractions that layer over it suffer 
from the problem of over-fetching or under-fetching data. In cases of over fetching, 
the client may only be interested in a subset of fields on a particular block, but must 
parse through the entire response to pull out the one or two needed fields. In the case 
of under-fetching, a client may need data from a large number of resources (say, multiple 
blocks, or calls to blocks as well as calls to transactions). The worst of both cases is 
where a client only uses one or two fields, but needs to make multiple calls to get all 
of the data. A new standard for data fetching needs to be specified at a fundamental level 
similar to the JSON-RPC spec to benefit from direct access of data and drive developer 
adoption to build applications.

GraphQL provides the following improvements:
* Efficient querying of data through explicity selection of the fields the application is interested in.
* Provides a way to evolve the API while providing backward compatibility
* Existing tooling provides self documenting, client generation, and developer playgrounds
* Provides a stronger typed system than existing interfaces
* Provides additional features such as stateless filtering, paging, scanning, and subscriptions
* More efficient wire transmission by omitting data not needed by the applications. 

## Backwards Compatibility
<!--All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EIP must explain how the author proposes to deal with these incompatibilities. EIP submissions without a sufficient backwards compatibility treatise may be rejected outright.-->
This specification is fully backward compatible with the existing JSON-RPC api. It does not 
call for the deprecation of the JSON-RPC specification.  GraphQL can exist along side of a 
JSON-RPC style interface to provide backward compatibility.

## Implementation
<!--The implementations must be completed before any EIP is given status "Final", but it need not be completed before the EIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

## Copyright
Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).
