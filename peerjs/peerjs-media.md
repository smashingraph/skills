---
name: peerjs-media
description: Captures local user media streams and establishes real-time WebRTC audio or video calls between peers. Use when the user asks to "start a video call", "implement voice chat", "stream webcam P2P", "answer a PeerJS call", "handle localStream", "toggle camera or microphone", or "create an audio/video interface".
---

# Skill: PeerJS Media Stream Management

## Core API Syntax Rules
- Call Initiation: `peer.call(targetId, localStream, [options])`.
- Call Answering: `call.answer(localStream)`.
- Media Event Binding: Always handle `.on('stream')`, `.on('close')`, and `.on('error')`.

## Implementation Patterns

### 1. Video Call Handler with Hardware Toggle
```javascript
export class P2PMediaHandler {
  constructor(peerInstance) {
    this.peer = peerInstance;
    this.currentCall = null;
  }

  async getLocalStream(video = true, audio = true) {
    try {
      return await navigator.mediaDevices.getUserMedia({
        video: video ? { facingMode: 'user', width: { ideal: 1280 }, height: { ideal: 720 } } : false,
        audio: audio
      });
    } catch (err) {
      throw new Error(`Media Capture Failed: ${err.message}`);
    }
  }

  startCall(targetId, localStream, remoteVideoElement) {
    this.currentCall = this.peer.call(targetId, localStream);
    this.bindCallEvents(this.currentCall, remoteVideoElement);
  }

  answerIncomingCall(call, localStream, remoteVideoElement) {
    this.currentCall = call;
    call.answer(localStream);
    this.bindCallEvents(call, remoteVideoElement);
  }

  bindCallEvents(call, remoteVideoElement) {
    call.on('stream', (remoteStream) => {
      if (remoteVideoElement) {
        remoteVideoElement.srcObject = remoteStream;
        remoteVideoElement.play().catch(console.error);
      }
    });

    call.on('close', () => {
      if (remoteVideoElement) remoteVideoElement.srcObject = null;
      this.currentCall = null;
    });
  }

  endCurrentCall() {
    if (this.currentCall) {
      this.currentCall.close();
    }
  }
}
```

## Guardrails & Constraints
- Always ensure modern async/await syntax is preferred over older promise chaining for `getUserMedia`.
- Always clean up DOM elements (set `srcObject = null`) when a call closes to free system resources and turn off camera indicators.
- Never request video without specifying explicit fallback constraints or error handling for devices without webcams.
