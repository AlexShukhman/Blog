---
layout: post
title:  "The Zeitgeist Model"
date:   2020-1-14
categories: [architecture]
---

RPC or _Remote Procedure Call_ protocol is a form of protocol where the client connects with the server over a form of websocket which allows the client to send procedure requests to the server and receive the output of the procedure as a response. This differs from the comparable to HTTP in that it requires knowledge of the available procedures on the server and a trusted connection with the server to avoid malicious behavior. Where in HTTP the client "requests" a "response" from the server by sending a message to the server with a variety of pre-determined, globally-accepted headers, a body and other metadata (making for a very large packet size), the RPC protocol requires a very specific call-style with much smaller packet size usually only including the procedure call and relevant parameter values. The intention of this _idea_ is to explore the potential for an RPC-style protocol for decentralized, P2P, massively distributed procedure calls.

## Issue

The _modus operandi_ of the most popular internet frameworks and ecosystems is to rely on the client-server model. This means that nearly all the information being sent between clients in the form of anything from strings to maps to blobs are generally sent first to a communal server - usually operated by what I will refer to as a "legislating body" or the owner/operator of the server(s) - then to the intended client. This means that the requestor's private information is continually sent through the legislating body's machine and it is the onus of the legislating body to ensure that the data is protected. The issue is that legislating bodies see no inherent value to protecting the data. The onus, therefore, is placed on the user.

## State of the Art

## Idea

The idea is to make a system that allow the fair, safe, and efficient transfer of data and procedure calls to one or more users in the same network.

### Vocabulary

**Member**: A user or client of the protocol

**Ledger**: The most complete record of transactions between members

**Source of Truth (SOT)**: The member whose responsibility it is to save and process new additions to the ledger

**Model**: The definition of available members, verbs, and adjectives

**Zeitgeist**: The abstract implementation of the _SOVA_ protocol in order to update the ledger

**Subject**: A pointer to the originating member of the procedure call

**Object**: A pointer to the SOT member intended to own the procedure call

**Verb**: The procedure call and its accompanying parameter values

**Adjective**: Any environment characteristics assumed for the procedure call that are not specific to the parameters

**SOVA Protocol**: The structure, saved in the ledger, that is shared in the zeitgeist that describes the subject, the object, the verb, and the adjective of the procedure call in a globally pre-determined markup.

### Implementation

#### Basic Infrastructure

For clarity, the intended host of a "member" can be any stateful machine or system capable of handling the infrastructure of the zeitgeist, accessing and writing to its own local memory, and maintaining a copy of its own information, the zeitgeist, and the model. For contributing members and members intending to reach the procedure calls, the members must also have the capability to encrypt and decrypt using the pre-determined encryption protocol.

While this model likely can be implemented with multiple types of Internet Protocol infrastructures, my intention is to focus on the websocket infrastructure. The ledger is to be updated by a member over a publicly-accessible websocket by publishing the SOVA markup to a channel available to all members.

#### The Zeitgeist Model

The zeitgeist is a generalizing term for all that goes on between the members. When any member contributes to the ledger, they are updating the zeitgeist. The zeitgeist differs from the ledger in that the ledger is part of the zeitgeist.

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

Members are not invited to the zeitgeist, they are accepted. That means that it is the onus of the member to engage themself with the zeitgeist. The procedure of engagement is as follows:

1. The engager connects with the socket hosting the protocol (see [above](#basic-infrastructure) for infrastructure details)
2. The engager publishes an introductory message (just the subject portion of the SOVA)
3. If the engager is accepted (acceptance protocol has not yet been defined), the SOT will publish a plaintext version of the model they have been maintaining, omitting only the pubKeys of the contributing members.

### Intuition
