---
name: peerjs-binary-transfer
description: Slices, transmits, and reassembles large binary files safely over WebRTC Data Channels using sequential chunk buffering. Use when the user asks to "send a file P2P", "transfer large binary data", "share images via PeerJS", "send ArrayBuffer or Blob", "implement a P2P file drop", or "fix connection dropping during file transfer".
---

# Skill: PeerJS Large Binary & File Chunking

## Core Mechanics
- WebRTC Data Channels have maximum message size limits (typically 16KB to 64KB for cross-browser safety).
- Large files must be sliced into sequential chunks, transmitted as ArrayBuffers, and reassembled on the receiving peer.

## Implementation Patterns

### 1. High-Performance Chunked File Sender
```javascript
export async function sendFileInChunks(dataConnection, file, onProgress) {
  const CHUNK_SIZE = 16384; // 16KB safe buffer slice
  const fileId = crypto.randomUUID();
  let offset = 0;

  dataConnection.send({
    type: 'FILE_META',
    payload: { fileId, name: file.name, size: file.size, type: file.type }
  });

  while (offset < file.size) {
    const slice = file.slice(offset, offset + CHUNK_SIZE);
    const buffer = await slice.arrayBuffer();

    dataConnection.send({
      type: 'FILE_CHUNK',
      payload: {
        fileId,
        sequence: offset / CHUNK_SIZE,
        chunk: buffer,
        isLast: (offset + CHUNK_SIZE) >= file.size
      }
    });

    offset += CHUNK_SIZE;
    if (onProgress) onProgress((offset / file.size) * 100);
  }
}
```

### 2. Stream Reassembler (Receiver)
```javascript
export class FileReceiver {
  constructor() {
    this.buffers = new Map();
  }

  handleIncomingChunk(packet) {
    const { fileId, chunk, isLast, sequence } = packet.payload;

    if (!this.buffers.has(fileId)) {
      this.buffers.set(fileId, []);
    }

    this.buffers.get(fileId)[sequence] = chunk;

    if (isLast) {
      const allChunks = this.buffers.get(fileId);
      const fileBlob = new Blob(allChunks);
      this.buffers.delete(fileId);
      return fileBlob;
    }
    return null;
  }
}
```

## Guardrails
- Absolute Rule: Forbid the AI from writing `conn.send(fileBlob)` directly for files larger than 1MB. It will cause silent connection drops on mobile or older web browsers.
- Always clear out chunk arrays from RAM (`this.buffers.delete(fileId)`) right after reassembly to prevent critical browser tab memory exhaustion.
