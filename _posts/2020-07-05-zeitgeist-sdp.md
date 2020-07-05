---
layout: post
title:  "WebRTC’s SDP Implementation for Zeitgeist Model"
date:   2020-7-5
categories: [architecture]
---

Session Description Protocol or [SDP](https://webrtchacks.com/sdp-anatomy/) is a standardized format for describing a client session for peer-to-peer networks. It is the backbone for WebRTC’s connection handshake and can be Base64-encoded with no loss of functionality (this is useful for cross-platform “dead drops” of session descriptions). This article will attempt to apply SDP as a description of [zeitgeist](https://ideas.alexshukhman.com/architecture/2020/06/14/zeitgeist-model.html) member sessions.

## Issue

Establishing a connection with a peer in a P2P network is tricky. Conveniently, there are plenty of pre-existing patterns for doing so and even the standard JavaScript API [WebRTC](https://w3c.github.io/webrtc-pc/) to help handle connections and complex media sockets. There is no real point to re-invent this wheel at the current time, so WebRTC will do.

## State of the Art

SDP is used in a variety of commonplace technologies including WebRTC, RTP, and RTSP. Common users of these technologies include some streaming services (though some services like Netflix do not -- they send packets of data along with a manifest and populate a buffer from that), video conferencing services like [Zoom](https://github.com/zoom/zoom-e2e-whitepaper/blob/master/zoom_e2e.pdf), and video surveillance services like the one in [Ring](https://support.ring.com/hc/en-us/articles/205385394-The-Protocols-and-Ports-Used-by-Ring-Devices).

## Idea

SDP can and be used as a unique user identifier. In the Zeitgeist model, member “addresses” are described loosely at best as the location of the member. This location can be described over IP by SDP.

## Implementation

In plain English, the following must be done:

1. A member is handed the SDP of the SOT node through a third party of the Zeitgeist. This should be done carefully over a trusted end-to-end encrypted signaling server.
2. The member initializes a connection
    1. The member describes themselves as a [WebRTC PeerConnection](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection) object.
    2. The member sets their PeerConnection’s [remoteDescription](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/setRemoteDescription).
    3. The member creates an [answer](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/createAnswer).
    4. The member sets their PeerConnection’s [localDescription](https://developer.mozilla.org/en-US/docs/Web/API/RTCPeerConnection/setLocalDescription) to the answer and signals this answer back to the SOT node.
3. Once the connection with the SOT node is established, this connection is used as the signaling server for establishing connections with the candidate SOT nodes.

Signals can still follow SOVA, but the initial connection with the SOT does require a signaling server. This signaling server has the following minimum requirements to remain safe and useful:

*   It must send messages over an encrypted protocol.
*   It must be discoverable by the SOT and the member.
*   It must implement a validation pattern for the signals.
*   The signaling server should be considered untrustworthy until proven otherwise.

## Intuition

The intention of the SOVA protocol is to minimize infrastructure by making the protocol more opinionated. Because it’s so much more simplified, it could make for a lower-weight alternative to the higher-level protocols like HTTP and can even act as a _lingua franca _protocol across connection types like Bluetooth, IP, radio, and Wifi tethering.
