---
layout: post
title:  "The Zeitgeist Model"
date:   2020-6-14
categories: [architecture, zeitns, deprecated]
---

# _DEPRECATION NOTICE: As of Aug 24, 2020, this is no longer a valid description of the product._

The most difficult portion of defining a secure, invisible, non-discoverable network is, frankly discovery. The Zeitgeist Model is a description of network architecture that attempts to describe such a potential P2P (peer to peer) network. A significant portion of this document discusses implementation of such a network and uses a fairly modern web-based protocol called WebRTC to handle message brokering. This can also be acheived leveraging other technology, WebRTC just happens to be a simple JavaScript-based solution for real-time peer-to-peer message brokering.

## Issue

The _modus operandi_ of the most popular internet frameworks and ecosystems is to rely on the client-server model. This means that nearly all the information being sent between clients in the form of anything from strings to maps to blobs are generally sent first to a central server cloud - usually operated by what I will refer to as a "legislating body" or the owner/operator of the server(s) - then to the intended reciever. This means that the requestor's private information is continually sent through the legislating body's machine and it is the onus of the legislating body to ensure that the data is protected. The issue is that legislating bodies see no inherent value to protecting the data. The onus, therefore, should be placed on the user.

## State of the Art

There exist a few peer-to-peer network architectures and systems which rely on various protocols to describe messages send between nodes. 

[This article](https://www.mobileprocessing.org/p2p.html) outlines some basic bluetooth implementation ideas and obviously [BitTorrent](https://www.bittorrent.com/) and [WebRTC](https://webrtc.org/) are good contenders for generic solutions. I do not intend for this to be a replacement for these protocols as obviously they have their values. Even [blockchain technology](https://en.wikipedia.org/wiki/Blockchain) implements a variety of fairly robust and complex peer-to-peer protocols. 

The Zeitgeist Model should be treated as more of an alternative for simpler, lighter-weight applications. Its fairly general description can be implemented in a variety of ways from extremely complex and secure (relying on client-side encryption and signatures with salts and hashing), to extremely lightweight and fragile (relying on a more robust re-connection method to handle spotty peer service availability). Where other actors in the space focus on reliability of connection and integrity of data, the Zeitgeist Model's rotating SOT system can be used as a solution to situations where lost data recovery could be more necessary.

## Idea

The idea is to make a system that allow the egalitarian, safe, and efficient transfer of data of any type to one or more nodes in the same network.

### Vocabulary

**Node/Member**: A user or client of the protocol

**Ledger**: The most complete record of transactions between members

**Source of Truth (SOT)**: The member whose responsibility it is to save and process new additions to the ledger

**Candidate SOT**: A member whose responsibility it is to save new additions to the ledger in the event that the SOT becomes unresponsive. All member nodes can be candidates, but limiting the number of candidate SOT's limits the number of discoverable nodes in the network. This lowers the potential for doxxing most members and it limits the weight on the network.

**Model**: The definition of available members, verbs, and adjectives

**Zeitgeist**: The abstract implementation of the _SOVA_ protocol in order to update the ledger

**Subject**: A pointer to the originating member of the procedure call

**Object**: A pointer to the SOT member intended to own the procedure call

**Verb**: The procedure call and its accompanying parameter values

**Adjective**: Any environment characteristics assumed for the procedure call that are not specific to the parameters

**SOVA Protocol**: The structure, saved in the ledger, that is shared in the zeitgeist that describes the subject, the object, the verb, and the adjective of the procedure call in a globally pre-determined markup.

### Implementation

#### Basic Infrastructure

For clarity, the intended host of a "member" can be any stateful client capable of handling the infrastructure of the zeitgeist, accessing and writing to its own local memory, and maintaining a copy of its own information, the zeitgeist, and the model. For contributing members and members intending to reach the procedure calls, the members must also have the capability to encrypt and decrypt using the pre-determined encryption protocol.

While this model likely can be implemented with multiple types of Internet Protocol infrastructures, my intention is to focus on the websocket infrastructure. The ledger is to be updated by a member over a publicly-accessible websocket by publishing the SOVA markup (encrypted and signed appropriately) to a channel available to all members.

#### The Zeitgeist Model

The zeitgeist is a general term of art for all that goes on between the members. When any member contributes to the ledger, they are updating the zeitgeist. The zeitgeist differs from the ledger in that the ledger is part of the zeitgeist.

In order to send a procedure call to more than one individual at the same time while retaining a structured call stack, a source of truth must be determined. All members look to the source of truth to evaluate the diff on their local ledgers. The source of truth is a member responsible for maintaining the primary ledger (although all other members maintain their own ledgers as well), updating the primary ledger based on local updates from members, and maintaining the model and its updates.

#### Member Basics

A member is defined by three strings:

1. The member's _handle_ or **UID** (for simple identification)
2. The member's _publishable key_ or **pubKey** (for decrypting the member's ledger contributions)
3. The member's _secret key_ or **secKey** (for encrypting the member's ledger contributions)

The member is responsible for maintaining the generation of all three of these strings. The `[UID, pubKey]` pair is used as the member's entry in the `members` portion of the model. The member's secKey is stored member-side and _should only be accessible to the member who uses it.

Members have the ability to request access to:

* The ledger
* Certain portions of the model, IE:
  * The `verbs` node
  * The `adjectives` node
  * The UIDs of the `members` node and the pubKeys of non-contributing members
* (From any member) the pubKeys of a member

Members have the responsibility to maintain local copies of:

* Their three strings
* The ledger
* The model that is as complete as possible

Members may also maintain a local copy of `[UID, pubKey]` pairs in order to expedite the engagement process with future zeitgeists.

#### Member Engagement Process

Members are not invited to the zeitgeist, they are accepted. That means that it is the onus of the member to engage themself with the zeitgeist. The procedure of engagement is as follows (implementation uses WebRTC but can use any socket-style protocol):

1. The new member instantiates a connection with the SOT node. The SOVA for this would be the following:
    ```js
    {
      S: "{member's address}",
      O: "{SOT's address}",
      V: "SAVE_CRED",
      A: null,
    }
    ```
    1. The new member must have access to the SOT's location (for example via NAT traversal protocols). This must be handled outside the zeitgeist.
    2. One solution to handing data like that is a dead-drop-style, encrypted bucket of information which regularly cleans itself.
2. The SOT node then send a message to the new member with the following body:
    ```js
    [{
      S: "{some candidate SOT's address}",
      O: "{member's address}",
      V: "SAVE_CRED",
      A: null,
    }, ...] // continue for all connected members who contributed to ledger and members accepted by SOT
    ```
3. For every node address, a location as well as decryption key (pubKey) should be included.
4. The new member would then instantiate their communication with all the nodes' info handed to them (including the SOT):
    ```js
    {
      S: "{member's address}",
      O: "{candidate SOT's address}",
      V: "SAVE_CRED",
      A: null,
    }
    ```
5. The resultant responses should be the following:
    ```js
    {
      S: "{current ledger}",
      O: "{member's address}",
      V: "SAVE_TOTAL_[SOT_]LEDGER", // SOT only when from SOT 
      A: "{ledger latest transaction timestamp}",
    }
    ```
6. At this point the member can start writing the ledgers to memory until the ledger from the SOT is recieved.
7. On recieving a ledger from the SOT, the member should go through a ledger audit:
    1. Unencrypt all ledgers as extensively as deemed necessary (see [note](#further-notes) on ledger size pruning)
    2. Using the SOT ledger as a guide, compile an array of ledger entries that are either missing or extraneous.
    3. The member should now return the compiled list of potentially erroneous entries to the SOT:
        ```js
        {
          S: ["{erroneous entry}", ...],
          O: "{SOT's address}",
          V: "CHECK",
          A: [
            "{return address}", 
            "{expected completedBy timestamp}", 
            "{request GUID}", // newly provisioned
          ],
        }
        ```
    4. The SOT node would then respond with a heartbeat to the member and validate each erroneous entry:
        ```js
        [{
          S: "{request GUID}",
          O: "{member's address}",
          V: "ACK_MESSAGE_RECIEVED",
          A: null,
        }, {
          S: "{erroneous entry}",
          O: "{entry signer's address}",
          V: "CHECK",
          A: [
            "{return address}",
            "{operation timeout timestamp}",
            "{request GUID}", // newly provisioned per entry
          ],
        }, ..., // continue on for all entries
        {
          S: "{request GUID}",
          O: "{member's address}",
          V: "ACK_REQUEST_SENT",
          A: null,
        }]
        ```
    5. Each node checking their entry should respond with either of the following:
        ```js
        {
          S: "{request GUID}",
          O: "{SOT's address}",
          V: "ACK_REQUEST_VALID",
          A: null,
        }
        ```
        ```js
        {
          S: "{request GUID}",
          O: "{SOT's address}",
          V: "NACK_REQUEST_INVALID",
          A: null,
        }
        ```
    6. These responses, on total compilation or request timeout, shoud be relayed back to the member (with a map, the SOT can answer multiple requests from one node). The results should also be used to update the SOT's ledger as well as that of all the candidate SOT's:
        ```js
        {
          S: {
            ["{request GUID}"]: ["{valid entry}", ...]
          },
          O: "{member's address}",
          V: "ACK_REQUEST_VALID",
          A: null,
        }
        ```
    7. The validated entries can now be added back to the member's ledger as well.
    8. The member is now able to bring their local env up to speed with the zeitgeist.
8. At this point, the member should be able to send and recieve new messages from the SOT.

#### Member Upgrade Process

While this process is not yet complete, it is fair to say the upgrade process from member to candidate SOT to SOT would likely be simliar to a data-locking mechanism where all members bid and the last member to bid in the timeframe is awarded the upgrade.

#### Member Downgrade Process

While this process is not yet considered, it is fair to say that it probably shouldn't be. The only time a member, candidate SOT, or SOT should be downgraded would likely be in the event of unresponsiveness in which case a replacement for their position would be determined by the [upgrade process](#member-upgrade-process).

#### Sending and Recieving Messages

1. The member should send the message in the following format to the SOT:
    ```js
    {
      S: "{message}",
      O: ["{recieving node address"}, ...],
      V: "{message verb}",
      A: ["{message adjective}", ...],
    }
    ```
2. Adjectives should contain any metadata required by the Zeitgeist itself. These standard adjective requirements can be set by the SOT and the candidate SOTs only. (Method by which these standards are set will be described in another _idea_)
3. On recieving the message, the SOT will distribute it to the candidate SOTs as well as assigning the candidate SOTs each an array of recieving node addresses to pass it on to:
    ```js
    [{
      S: "{message}",
      O: "{candidate SOT address}",
      V: "{message verb}",
      A: [
        "{message adjective}", ..., 
        "{recieving node address", ...,
      ],
    }, ..., {
      S: "{message GUID if relevant}",
      O: "{origin address}",
      V: "ACK_REQUEST_SENT",
      A: null,
    }]
    ```
4. On receiving a non-ack message, all recievers must ACK as is demonstrated in the last entry sent. They must also add the message to their local ledger.
5. On reciving an ack message, the reciever can relay the ack back to the origin of the message:
    ```js
    {
      S: "{message GUID if relevant}",
      O: "{origin address}",
      V: "ACK_REQUEST_RECIEVED",
      A: null,
    }
    ```
6. If any operation times out (nack), the node should be considered "dead" and the nack should be relayed back to the origin of the message. The original sender may then choose to resend the message to the nacked user. A map can be used to relay multiple nacks for multiple requests (for less load on the network)
    ```js
    {
      S: {["{message GUID}"]: ["{nacked address}", ...]},
      O: "{origin address}",
      V: "NACK_REQUEST_FAILED",
      A: null
    }
    ```
7. The message is now sent.

#### Leaving the Zeitgeist

Leaving is simple and should be treated as commonplace. The zeitgeist is left by not responding to requests. This response failure should be handled as gracefully as possible.

### Intuition

The intuition here is that any computer, connected device or message endpoint (even a physical mailbox!) can act as a point of communication. The path by which communication happens shouldn't have to be dictated by a company or governing body like online chats, email, mail and phone services.

An ultra-simplified network-agnostic protocol could allow even entry-level developers to create networks that don't rely on internet service providers, large tech giants, or massive blockchains.

### Further Notes

* In light of recent events (like the 2020 Black Lives Matter protests), it's become more important than ever to protect the sanctity of free and unadulterated communication. This protocol could be used to provide a valuable life-saving service to protestors in case of dangerous anti-communication measures like blocking internet and cell tower connections.
* Great precaution should be taken in sending the initial member-SOT connection information. It is vital that this intantiation of connection be safeguarded against DDoS, MiM and other fraudulent or malicious attacks.
* Given the potential size of the ledger, the logs therefrom should be rotated (the ledger should be pruned) based on age and relevance. This pruning mechanism can be discussed in a separate _idea_.
