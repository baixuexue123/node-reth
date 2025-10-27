# Flashblocks RPC

This crate provides RPC methods for accessing Flashblocks state and subscribing to real-time flashblock updates.

## Features

- **Enhanced Ethereum RPC Methods**: Support for the `pending` tag in standard Ethereum RPC methods to query preconfirmed flashblocks state
- **Flashblocks Subscription**: Real-time WebSocket subscription to receive flashblock updates as they are processed

## Flashblocks Subscription

The flashblocks subscription API allows clients to receive real-time updates about new flashblocks as they are received and processed by the node.

### Usage Example

```javascript
// Using WebSocket
const ws = new WebSocket('ws://localhost:8545');

// Subscribe to flashblocks
ws.send(JSON.stringify({
  "jsonrpc": "2.0",
  "id": 1,
  "method": "flashblocks_subscribe",
  "params": []
}));

// Listen for flashblock notifications
ws.onmessage = (event) => {
  const data = JSON.parse(event.data);
  if (data.method === 'flashblocks_subscription') {
    const flashblock = data.params.result;
    console.log('New flashblock:', flashblock);
    console.log('Block number:', flashblock.metadata.block_number);
    console.log('Flashblock index:', flashblock.index);
  }
};
```

### Features

- **Real-time Updates**: Receive flashblocks as soon as they are processed by the node
- **Optimized Delivery**: Only the latest flashblock is sent to avoid flooding clients
- **Full Flashblock Data**: Includes transactions, receipts, state changes, and metadata

See [spec.md](./spec.md) for detailed API documentation.

## Testing

To include integration tests when testing, run:

```bash
# Must be run in the root of the repo
# Build the base-reth-node binary
cargo build --bin base-reth-node

# Must be run in the crate
# Run the unit tests + integration tests
cargo test --features "integration"
```