---
name: peerjs-multipeer
description: Orchestrates multi-user P2P group networks using a full-mesh topology while applying strict participant scalability limits. Use when the user asks to "create a P2P chat room", "build a multi-user mesh", "connect multiple peers together", "handle group video call", "broadcast to all peers", or "manage group room state".
---

# Skill: PeerJS Multi-Peer & Mesh Topologies

## Architectural Constraints (Strict)
- Mesh Limits: PeerJS uses a pure Mesh network topology (direct connections between every single user).
- Data Channels: Limit to a maximum of 15-20 concurrent peers per client.
- Media Streams (AV): Limit strictly to a maximum of 4-5 concurrent peers per client.
- Scale-Up Rule: If the user requests a video call architecture exceeding 5 participants, the agent must refuse to use raw PeerJS for the media engine and recommend a WebRTC Media Server (e.g., Mediasoup, Janus, or an SFU topology).

## Implementation Patterns

### 1. Decentralized Full-Mesh Room Tracker
```javascript
export class P2PMeshRoom {
  constructor(peerInstance, roomCode) {
    this.peer = peerInstance;
    this.roomCode = roomCode;
    this.peersInRoom = new Set();
    this.connections = new Map();
  }

  broadcast(type, payload) {
    const packet = { type, payload, from: this.peer.id, timestamp: Date.now() };
    this.connections.forEach((conn) => {
      if (conn.open) {
        conn.send(packet);
      }
    });
  }

  connectToNewPeer(newPeerId) {
    if (this.peersInRoom.has(newPeerId) || newPeerId === this.peer.id) return;

    const conn = this.peer.connect(newPeerId, { reliable: true });
    this.peersInRoom.add(newPeerId);

    conn.on('open', () => {
      this.connections.set(newPeerId, conn);
      this.broadcast('USER_JOINED_MESH', { peerId: this.peer.id });
    });

    conn.on('close', () => {
      this.connections.delete(newPeerId);
      this.peersInRoom.delete(newPeerId);
    });
  }
}
```

## Guardrails
- Never let the AI create a nested loop that re-triggers `.connect()` symmetrically, which causes infinite loop handshake collisions between two peers.
