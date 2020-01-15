---
layout: post
title:  "The Obscure Chat"
date:   2020-1-14
categories: [products, architecture]
---

I had this idea a few months ago and am only now having the opportunity to put it out there.

## Issue

We have a bad habit, as consumers of digital social platforms, of relying on extreme scale and security through obscurity to protect our personal info. Take Facebook for example. It's no secret that Facebook holds a _lot_ of your digital information, but if I asked the "average" user, "where is your data, physically?" I can nearly guarantee that they wouldn't be able to tell you. I also can't tell you, but I can tell you that, if you wanted to see your various database entries in their system, the SRE's and database admins would not be happy to share it with you. That's frankly pretty wild. You have absolutely no idea about _so many factors_:

* Is my data transfers like requests and responses encrypted?
* Who can access my data?
* What of my data is stored in plaintext?
* Are my files encrypted?
* Who can access my files?
* What metadata is stored about me?
* Who can access my metadata?

If any of these questions concern you, you're not alone. The answers are certainly not readily available and, even if your data is end-to-end encrypted (possible, but not very likely IMO), even if nobody can access your data (definitely not the reality), even if they clear your data every day (it's common practice for many software companies to never clear data, it may be sent to a data warehouse, but that data is an amazing source of passive income!), even if all of those personal privacy protections are in place, you _still_ can't be sure what of your data is in plain text. That means every time you send a tasteful nude photo to someone _who requested you do so_, or discuss the benefits and disadvantages of various illicit substances, all of that data and more is stored, usually in plain text, on a server you have _no access to_. That, I find, is generally a recipe for disasterous breeches of trust.

## State of the Art

_I do not pretend to be an expert in this field,_ but I have come across a few services that offer end-to-end encrypted chats with encrypted databases and sometimes even a feature that removes the chats (Snapchat-style) from access to the user. I'm careful to say "removes the chats from access to the user" because, like Snapchat, there isn't very much visibility into that process. Snapchat has said (looking for the reference still!) that they never delete any image off their database. That means that, while you may now be able to access it, a database admin or SRE or the likes can, and they do! Having had worked in various management consulting roles, I find it amazing how easily available users' data is for both technical and non-technical users. And trust me, a non-disclosure agreement doesn't stop most consultants from telling a story or two about outliers they see on the job.

## Idea

What if you could own your data, outright, 100%. What if you knew what data was stored on your social media, where it was stored, how it's stored, and who can access it. What if you knew everything that went through the pipes, plaintext, encrypted, files, and all. What would you do with the security of ownership of that information. It's frankly not that hard, just make sure that you own the data pipeline, not Facebook, not Snapchat or Google, you.

## Implementation

1. Access the _Obscure_ chat protocol server at a domain like https://obsc.re. At that domain you are greeted with a static webpage, no preloading, no nothing, just a welcome page.
2. On the welcome page, `/`, you are given the option to enter a chat room, one of some pre-set number of servers. Entering the chat room is as simple as going to the page `/r/{room #}` (as short a url as possible in order to allow more room for the room #, thusly allowing more rooms).
3. Entering a "chat room" would follow the following process:
    1. Request `/r/{room #}`
    2. Response is, as always, a static webpage with no requests available back to the obsc.re server (other than to navigate back to home I guess?).
    3. The page now opens a peer-to-peer socket I will refer to as the _Zeitgeist_ (to be discussed further in a separate _idea_) to the address.
    4. If the socket discovers a _Source of Truth_ member node (a peer which is responsible for owning the ledger of the session), it requests access to the ledger's member info to display to the user.
    5. If no _SOT_ member node is discovered, it labels itself as an _SOT_ member. It now owns the blank ledger and awaits future instruction from members that connect.
4. Once the initial connection to the 