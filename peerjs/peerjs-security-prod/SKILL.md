---
name: peerjs-security-prod
description: Hardens PeerJS configurations for production environments using custom signaling, secure SSL infrastructure, ICE options, and STUN/TURN servers. Use when the user asks to "prepare PeerJS for production", "configure TURN server", "add STUN options", "fix corporate firewall issues", "handle PeerJS connection errors", or "deploy custom signaling server".
---

# Skill: PeerJS Production & Security Configurations

## Configuration Standards

### 1. Production Peer Options Blueprint
```javascript
const productionPeerConfig = {
  host: process.env.PEER_SERVER_HOST || '://yourdomain.com',
  port: parseInt(process.env.PEER_SERVER_PORT || '443', 10),
  path: process.env.PEER_SERVER_PATH || '/myapp',
  secure: true,
  config: {
    iceServers: [
      { urls: 'stun:://google.com' },
      { urls: 'stun:://google.com' },
      {
        urls: process.env.TURN_SERVER_URL || 'turn:yourturnserver.com:3478',
        username: process.env.TURN_SERVER_USERNAME || 'secure_user',
        credential: process.env.TURN_SERVER_PASSWORD || 'secure_password'
      }
    ],
    iceTransportPolicy: 'all'
  }
};
```

## Error Recovery Matrices
```javascript
peer.on('error', (err) => {
  switch(err.type) {
    case 'browser-incompatible':
      alert('Your browser does not support WebRTC P2P communication.');
      break;
    case 'disconnected':
      console.warn('Signaling server disconnected. Attempting reconnect...');
      peer.reconnect();
      break;
    case 'invalid-id':
      console.error('The provided Peer ID contains illegal characters.');
      break;
    case 'peer-not-found':
      console.error('The target remote Peer ID does not exist on the signaling network.');
      break;
    case 'unavailable-id':
      console.error('The requested Peer ID is already taken by another node.');
      break;
    case 'network':
    case 'server-error':
      console.error('Signaling infrastructure unreachable.');
      break;
    default:
      console.error('Unhandled PeerJS Exception:', err);
  }
});
```

## Guardrails & Constraints
- Absolute Rule: Completely forbid using the free, default cloud server (`new Peer()` with no parameters) in any code block marked for production.
- Always ensure sensitive TURN server credentials are read from environment variables, never hardcoded string literals.
- Always explicitly specify `secure: true` when running on `https://` production environments to avoid mixed-content blocking.
