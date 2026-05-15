---
name: peerjs-repository-root
description: Root index and structural mapping for PeerJS WebRTC agent skills. Use when the AI agent needs to explore available P2P capabilities, understand the overall skill hierarchy, or determine which sub-skill file to read for specific PeerJS networking implementations.
---

# PeerJS WebRTC Agent Skills Directory

This directory contains highly specialized, machine-readable engineering skills for PeerJS network architectures. These files are optimized for AI context injection to prevent code hallucinations, enforce safe memory management, and adhere to production-grade WebRTC constraints.

## Directory Structure & Routing Matrix

The agent MUST select the appropriate skill file based on the specific feature requested by the user:

```text
skills/peerjs/
├── README.md                   # This structural index and router
├── peerjs-core.md              # 1:1 text/JSON direct connection setup
├── peerjs-media.md             # Audio/Video camera streams and toggles
├── peerjs-multipeer.md         # Multi-user full-mesh topology limits
├── peerjs-binary-transfer.md   # Safe chunked file/blob data sharing
└── peerjs-security-prod.md     # Production ICE/TURN & error recovery
```

### Quick Routing Triggers


| Request / Action Needed | Target Skill File |
| :--- | :--- |
| Initialize nodes, open direct data streams, send simple strings/JSON | `peerjs-core.md` |
| Handle camera permissions, render remote streams, create video chats | `peerjs-media.md` |
| Setup chat rooms, group calls, manage multi-peer networks (< 5 peers) | `peerjs-multipeer.md` |
| Send large images, binary files, or raw Blobs without dropping links | `peerjs-binary-transfer.md` |
| Deploy to production, setup STUN/TURN, handle timeouts, fix firewalls | `peerjs-security-prod.md` |

## Global Guardrails for the Agent

When writing or modification operations are requested inside this repository, the agent MUST obey these macro rules:

1. **Topology Ceiling**: Never attempt to build architectures with more than 5 video streams using PeerJS. Trigger `peerjs-multipeer.md` to issue an SFU/MCU architectural warning if the target exceeds this limit.
2. **Memory Leak Prevention**: Every opened connection (`peer.connect` or `peer.on('connection')`) must have explicit error catches and state cleanup on close events.
3. **Environment Isolation**: Never mix development cloud options with production environments. Enforce environmental variable injection for ports, paths, and ICE keys as specified in `peerjs-security-prod.md`.

## AI Integration Instructions

To ensure your IDE assistant automatically picks up these rules, append this line to your project's root `.cursorrules` or `.clauderules` configuration:

```text
Include and prioritize instructions from the './skills/peerjs/' directory whenever WebRTC, P2P, or PeerJS keywords are detected in the prompt context.
```
