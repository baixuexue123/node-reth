# Flashblocks RPC Methods Specification

This document specifies the JSON-RPC methods implemented by the Flashblocks RPC provider.

Note: This document serves as a reference for the current implementation and may differ from the full Flashblocks specification in terms of scope and methods implemented.

## Type Definitions

All types used in these RPC methods are identical to the standard OP Stack RPC types. No modifications have been made to the existing type definitions.

## Modified Ethereum JSON-RPC Methods

The following standard Ethereum JSON-RPC methods are enhanced to support the `pending` tag for querying preconfirmed state.

## eth_getBlockByNumber

Returns block information for the specified block number.

### Parameters
- `blockNumber`: `String` - Block number or tag (`"pending"` for preconfirmed state)
- `fullTransactions`: `Boolean` - If true, returns full transaction objects; if false, returns transaction hashes

### Returns
`Object` - Block object

### Example
```json
// Request
{
  "method": "eth_getBlockByNumber",
  "params": ["pending", false],
  "id": 1,
  "jsonrpc": "2.0"
}

// Response
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": {
    "hash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "parentHash": "0x...",
    "stateRoot": "0x...",
    "transactionsRoot": "0x...",
    "receiptsRoot": "0x...",
    "number": "0x123",
    "gasUsed": "0x5208",
    "gasLimit": "0x1c9c380",
    "timestamp": "0x...",
    "extraData": "0x",
    "mixHash": "0x...",
    "nonce": "0x0",
    "transactions": ["0x..."]
  }
}
```

### Fields
- `hash`: Block hash calculated from the current flashblock header
- `parentHash`: Hash of the parent block
- `stateRoot`: Current state root from latest flashblock
- `transactionsRoot`: Transactions trie root
- `receiptsRoot`: Receipts trie root
- `number`: Block number being built
- `gasUsed`: Cumulative gas used by all transactions
- `gasLimit`: Block gas limit
- `timestamp`: Block timestamp
- `extraData`: Extra data bytes
- `mixHash`: Mix hash value
- `nonce`: Block nonce value
- `transactions`: Array of transaction hashes or objects

### `eth_getTransactionReceipt`

Returns the receipt for a transaction.

**Parameters:**
- `transactionHash`: `String` - Hash of the transaction

**Returns:** `Object` - Transaction receipt or `null`

**Example:**
```json
// Request
{
  "method": "eth_getTransactionReceipt",
  "params": ["0x..."],
  "id": 1,
  "jsonrpc": "2.0"
}

// Response
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": {
    "transactionHash": "0x...",
    "blockHash": "0x0000000000000000000000000000000000000000000000000000000000000000",
    "blockNumber": "0x123",
    "transactionIndex": "0x0",
    "from": "0x...",
    "to": "0x...",
    "gasUsed": "0x5208",
    "cumulativeGasUsed": "0x5208",
    "effectiveGasPrice": "0x...",
    "status": "0x1",
    "contractAddress": null,
    "logs": [],
    "logsBloom": "0x..."
  }
}
```

**Fields:**
- `transactionHash`: Hash of the transaction
- `blockHash`: zero hash (`0x000...000`) for preconfirmed transactions
- `blockNumber`: Block number containing the transaction
- `transactionIndex`: Index of transaction in block
- `from`: Sender address
- `to`: Recipient address
- `gasUsed`: Gas used by this transaction
- `cumulativeGasUsed`: Total gas used up to this transaction
- `effectiveGasPrice`: Effective gas price paid
- `status`: `0x1` for success, `0x0` for failure
- `contractAddress`: Address of created contract (for contract creation)
- `logs`: Array of log objects
- `logsBloom`: Bloom filter for logs

### `eth_getBalance`

Returns the balance of an account.

**Parameters:**
- `address`: `String` - Address to query
- `blockNumber`: `String` - Block number or tag (`"pending"` for preconfirmed state)

**Returns:** `String` - Account balance in wei (hex-encoded)

**Example:**
```json
// Request
{
  "method": "eth_getBalance",
  "params": ["0x...", "pending"],
  "id": 1,
  "jsonrpc": "2.0"
}

// Response
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": "0x1bc16d674ec80000"
}
```

### `eth_getTransactionCount`

Returns the number of transactions sent from an address (nonce).

**Parameters:**
- `address`: `String` - Address to query
- `blockNumber`: `String` - Block number or tag (`"pending"` for preconfirmed state)

**Returns:** `String` - Transaction count (hex-encoded)

**Example:**
```json
// Request
{
  "method": "eth_getTransactionCount",
  "params": ["0x...", "pending"],
  "id": 1,
  "jsonrpc": "2.0"
}

// Response
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": "0x5"
}
```

## Behavior Notes

### Pending Tag Usage
- When `"pending"` is used, the method queries preconfirmed state from the flashblocks cache
- If no preconfirmed data is available, falls back to latest confirmed state
- For non-pending queries, behaves identically to standard Ethereum JSON-RPC

### Error Handling
- Returns standard JSON-RPC error responses for invalid requests
- Returns `null` for non-existent transactions or blocks
- Falls back to standard behavior when flashblocks are disabled or unavailable


## Flashblocks Subscription Methods

### `flashblocks_subscribe`

Subscribe to new flashblocks as they are received and processed by the node.

**Parameters:**
None

**Returns:**
- `subscriptionId`: `String` - Subscription ID for managing the subscription

**Example:**
```json
// Request (via WebSocket)
{
  "method": "flashblocks_subscribe",
  "params": [],
  "id": 1,
  "jsonrpc": "2.0"
}

// Response
{
  "id": 1,
  "jsonrpc": "2.0",
  "result": "0x1234567890abcdef"
}

// Notification (sent whenever a new flashblock is received)
{
  "jsonrpc": "2.0",
  "method": "flashblocks_subscription",
  "params": {
    "subscription": "0x1234567890abcdef",
    "result": {
      "payload_id": "0x0000000000000000",
      "index": 1,
      "base": null,
      "diff": {
        "state_root": "0x...",
        "receipts_root": "0x...",
        "gas_used": "0x5208",
        "block_hash": "0x...",
        "transactions": ["0x..."],
        "withdrawals": [],
        "logs_bloom": "0x...",
        "withdrawals_root": "0x..."
      },
      "metadata": {
        "block_number": 123,
        "receipts": {},
        "new_account_balances": {}
      }
    }
  }
}
```

**Fields:**
- `payload_id`: Unique identifier for the payload
- `index`: Sequential index of the flashblock within the block
- `base`: Base execution payload (present only for index 0)
- `diff`: Delta changes in this flashblock
    - `state_root`: Current state root
    - `receipts_root`: Receipts root
    - `gas_used`: Cumulative gas used
    - `block_hash`: Current block hash
    - `transactions`: List of transactions in this flashblock
    - `withdrawals`: List of withdrawals
    - `logs_bloom`: Bloom filter for logs
    - `withdrawals_root`: Withdrawals root
- `metadata`: Additional metadata
    - `block_number`: Block number
    - `receipts`: Transaction receipts by hash
    - `new_account_balances`: Updated account balances

**Behavior:**
- The subscription delivers flashblocks in real-time as they are received from the upstream source
- Each notification contains the latest flashblock processed by the node
- The subscription remains active until explicitly unsubscribed
- If multiple flashblocks are processed simultaneously, only the latest one is sent to avoid flooding the client

### Usage Example

```javascript
// Using ethers.js or similar WebSocket-based library
const provider = new ethers.providers.WebSocketProvider('ws://localhost:8545');

// Subscribe to flashblocks
const subscriptionId = await provider.send('flashblocks_subscribe', []);

// Listen for flashblock notifications
provider.on('flashblocks_subscription', (flashblock) => {
  console.log('New flashblock received:', flashblock);
  console.log('Block number:', flashblock.metadata.block_number);
  console.log('Flashblock index:', flashblock.index);
  console.log('Transactions:', flashblock.diff.transactions.length);
});

// Unsubscribe when done
await provider.send('eth_unsubscribe', [subscriptionId]);
```