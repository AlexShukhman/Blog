---
layout: post
title:  "The Zeitgeist Model (v1.0.0)"
date:   2020-8-25
categories: [zeitns, architecture, products]
---

_This is the most recently updated version of this product's description_

The "Zeitgeist Model" encompasses the various technical descriptions required to make a peering connection using the SOVA protocol. The model is still in progress and is not feature-complete.

## History

The Zeitgeist Model was first described as a peer-to-peer (P2P) protocol in a heavier, less-efficient format in June of 2020. [This "idea"](https://ideas.alexshukhman.com/deprecated/2020/06/14/zeitgeist-model.html) by Alex Shukhman describes how it was intended to work as a connection similar to WebRTC, but as requirements were added, it became more clear that the model worked best as a lighter-weight ethernet-like model.

## Intention

This iteration of the Zeitgeist Model is intended to more clearly state the value of the model as a highly-extensible system for cross-protocol communication over strictly-enforceable networks like local or wide-area networks. The SOVA protocol is loosely based on the "Ethernet Frame" protocol as it, too contains a semblance of an address, data block, and metadata. It is not intended to be used without end-to-end encryption and is not yet a viable solution for large, constantly-changing networks (like ISP networks). Efficiency will come with routing optimizations planned for future versions.

## SOVA

SOVA stands for:

*   Subject (the receiver of the data)
*   Object (the transmitter of the data)
*   Verb (the data packet -- as a command)
*   Adjective (the metadata around the packet)

It can be simplified greatly and therein lies the flexibility of the infrastructure. For example, in a small network, the subject (either described by a ZeitToken or a network location -- see below for nameserver applications) can be described with as little as one "bit pair" or two bits of information. The object can as well. The verbs, if strictly defined, can be limited to a small instruction set for embedded protocols like BlueTooth, Z-Wave, and Zigby, but can also be described in more flexible instruction sets like in the case of IP communications. Adjectives may contain structured data as well and can be ignored in the case of ultra-lightweight communication for NFC implementations.

### Subject/Object

The subject and object of a SOVA packet can be described in two ways: using a ZeitToken (see below for more details on token-to-address conversion options), or using an address notation. The address notation is a series of bit pairs describing the node's (in this case, either the source client's or the destination client's) relation to the presiding nameserver. This can be optimized in future editions of the Zeitgeist Model. 

All parent nodes may have only two child nodes (this is why the pseudo-node concept is valuable) so the left node will be described as [0,0] or 0 in base-4 whereas the right node will be described as [0,1] or 1 in base-4. The parent node will be described as [1,0] or 2 in base-4 and the [1,1] description may be used for clock mechanics. These pairs are valuable because the communication of SOVA can now be easily parallelized over traditional ethernet (Cat5+) cables. This address management solution makes next-hop calculations trivial_ but may require more work for optimizing pseudo-nodes_.

### Verb

The verb consists of a bit-pair-encoded command (from a predetermined list of commands that are node-specific) and a pre-encrypted, binary encoded (then demuxed) payload (non-standard data related to the verb like asset descriptions or update values). Multi-threading-style standards can also be considered in the muxing and demuxing step.

### Adjective

The adjective can consist of various standardized metadata points. For example, if connecting to an IP device, the adjective can include a checksum, publishable/public key, or other security-related data. There exists a lot of potential for adjectives related to next-hop alternatives as well.

## Multiplexing (Muxing)

A _hardware_ solution (transistor logic) should be used to handle the serialization and parallization (muxing and demuxing respectively) of the data. Except in the case of non-standard SOVA elements (like addresses and commands), the data is expected to be interpreted initially as a serial bitstream. This serial bitstream should use optimized muxing transistor logic (this should be standard or should be described by shared standards definitions) to allow for two parallel outputs/inputs to simultaneously transmit and recieve data (respectively) ingestable by the clients' systems.

## Nameservers/ZeitNS

To aid in node discovery, nameservers should be omnipresent in a Zeitgeist. A nameserver contains the ZeitToken-to-Address lookup for the sub-Geist (sub-network of a Zeitgeist). These lookups can be published to the parent nameserver or available only as a directed (unpublished) lookup in the child nameserver. Nameserver lookups should be described in the following ways:

*Lookup All*:

```yaml
S: 10 (parent)
O: {client address}
V: {LOOKUP}:{ALL}
A: null (can include a key to allow if required)
```

*Lookup One*:

```yaml
S: 10 (parent)
O: {client address}
V: {LOOKUP}:{ZeitToken}
A: null (can include a key to allow if required)
```

**Lookup responses from nameservers can take one of four formats:**

*Success*: 

```yaml
S: {client address}
O: 10
V: 11(Success):{Address Location}
A: null
```

*Failures*:

```yaml
S: {client address}
O: 10
V: 00 (Fail -- Server) || 01 (Fail -- Client)
A: null || {failure description} (respectively)
```

*Not Changed:*

```yaml
S: {client address}
O: 10
V: 10 (No Change)
A: null
```

**On connecting to a Zeitgeist, the client must pair with the closest nameserver:**

*Set ZeitToken*:

```yaml
S: 10 (parent)
O: {client address}
V: {SET}:{ZeitToken}
A: null (can include a key to allow if required)
```

**Nameservers will send a success, no change, or failure in response:**

*Success*:

```yaml
S: {client address}
O: 10
V: 11 (Success)
A: null
```

*Failures*:

```yaml
S: {client address}
O: 10
V: 00 (Fail -- Server) || 01 (Fail -- Client)
A: null || {failure description} (respectively)
```

*Not Changed*:

```yaml
S: {client address}
O: 10
V: 10 (No Change)
A: null
```

These responses are not required for all protocols, for protocols that do not rely on handshakes, if a response is required, the no-change response should be used. Client failures should be expected to result in retries (progressive rollback preferred) and should include failure descriptions (limited to a minimal description option set).

## ZeitTokens

The concept of a ZeitToken is not yet well-defined. The options may include, but are not limited to, the following:

*   A random set-length bit array
*   A base-4 encoded "nickname" that is client-specified and then validated by the presiding nameserver
*   An encoding of the various possible connection information for the device (eg IP address + Mac Address + SDP + BlueTooth Address)
*   A combination or permutation of any of the above

## Routing/Pseudo-Nodes

Routing should use binary tree logic. That is, a parent node (like a switch) should have at most two child nodes. The child nodes may be either real connections to actual hardware devices or virtualized _pseudo-nodes; each_ of these pseudo-nodes should have their own address locations and should have no functionality other than as pass-through nodes to their parents. 

Pseudo-nodes, like nodes, can have at most two child nodes. The denotation of "left" versus "right" nodes can be arbitrary until a better, more efficient system is in place. As most hardware connections are to be considered unstable, node and pseudo-node discovery and allocation (respectively) should be as lightweight as possible, favoring speed of allocation over the long-run efficiency.

## Security

As has been noted in previous versions, Zeitgeist does not have its own transport layer security. It is expected that all connections over SOVA are to be publicly accessible by all nodes and all communications publicly readable by all nodes. This is in order to improve performance (relying on tighter, ideally lighter-weight, and less opaque end-to-end encryption strategies than IP's TLS). It, therefore, also promotes the innovation of lighter-weight E2E encryption strategies for embedded devices and the Internet of Things which can potentially cut out the need for certificate authorities.

## Hardware Considerations

As this is a natively base-4 encoded protocol system, hardware requirements include connections with at least 7 pins (Receive_L, Receive_R, Receive_COM, Transmit_L, Transmit_R, Transmit_COM, and CLOCK if operating the clock at a comparable voltage to the receive-type data pins) or 5 data inputs in the case of fiber (Receive_L, Receive_R, Transmit_L, Transmit_R, and CLOCK). This is extremely well-suited to modern ethernet cables (with 8 pins, one may be used for shielding), and can be optimized for paired, multi-modal, or multi-core fiber cables as well. Hardware considerations for wireless connections have not yet been made but would presumably rely on either dual-band connections or on simple muxing and de-muxing circuits edge-side.

## Data Structures and Code Efficiency

The variable-length but extremely-standardized location definitions may cause data allocation issues. This can be mitigated using various language-specific optimizations but should be considered when planning networks (specifically with regards to planning sub-Geist sizes and nameserver creation). It is likely that many of these optimizations will take into account the recursive nature of the structures and can, therefore, be confidently optimized to create IPv6-style address shortening strategies.
