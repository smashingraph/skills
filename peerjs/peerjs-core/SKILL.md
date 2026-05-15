---
name: peerjs-core
description: Initializes fundamental PeerJS instances, configures simple P2P DataChannels, and manages unique peer identifiers. Use when the user asks to "set up PeerJS", "initialize a peer", "connect two browsers", "send data P2P", "create a P2P data connection", "implement direct text messaging", or "send JSON over WebRTC".
---

# Skill: PeerJS Core Network Operations

## Core API Syntax Rules
- Always import or initialize PeerJS safely for the target environment (Browser vs Node.js/Bun).
- Connection Initialization: `new Peer([id], [options])`.
- Data Connection: `peer.connect(targetId, [options])`.
- Data Event Binding: Always handle `.on('open')`, `.on('data')`, `.on('close')`, and `.on('error')`.

## Implementation Patterns

### 1. Robust Connection Wrapper (JSON Payload Optimized)
```javascript
import { Peer } from 'peerjs';

export class P2PDataNode {
  constructor(customId = null, config = {}) {
    this.peer = new Peer(customId, config);
    this.activeConnections = new Map();
    this.initListeners();
  }

  initListeners() {
    this.peer.on('connection', (conn) => {
      this.setupConnectionListeners(conn);
    });
    this.peer.on('error', (err) => this.handleError(err));
  }

  connectToPair(targetId) {
    const conn = this.peer.connect(targetId, {
      reliable: true,
      serialization: 'json'
    });
    this.setupConnectionListeners(conn);
    return conn;
  }

  setupConnectionListeners(conn) {
    conn.on('open', () => {
      this.activeConnections.set(conn.peer, conn);
    });

    conn.on('data', (data) => {
      this.routeIncomingData(conn.peer, data);
    });

    conn.on('close', () => {
      this.activeConnections.delete(conn.peer);
    });
  }

  sendPayload(targetId, type, payload) {
    const conn = this.activeConnections.get(targetId);
    if (conn && conn.open) {
      conn.send({ type, payload, timestamp: Date.now() });
    }
  }

  routeIncomingData(senderId, packet) {
    // Override in implementation
  }

  handleError(err) {
    // Critical error mapping handled in security skill
  }
}
```

## Guardrails & Constraints
- Never hardcode Peer IDs in code examples; always generate them dynamically or pass them via runtime arguments.
- Never omit the `.on('data')` listener when a connection opens. Unhandled data channels lead to silent memory leaks.
