---
layout: post
title:  "The Obscure Chat"
date:   2020-1-14
categories: [products, architecture]
---

I had this idea a few months ago and am only now having the opportunity to put it out there.

## Issue

We have a bad habit, as consumers of digital social platforms, of relying on extreme scale and security through obscurity to protect our personal info. Take Facebook for example. It's no secret that Facebook holds a _lot_ of your digital information, but if I asked the "average" user, "where is your data, physically?" I can nearly guarantee that they wouldn't be able to tell you. I also can't tell you, but I can tell you that, if you wanted to see your various database entries in their system, the data owners would not be happy to share it with you. That's frankly pretty wild.

You have absolutely no idea about _so many factors_:

* Are my data transfers like requests and responses encrypted?
* Who can access my data?
* What of my data is stored in plaintext?
* Are my files encrypted?
* Who can access my files?
* What metadata is stored about me?
* Who can access my metadata?

If any of these questions concern you, you're not alone. The answers are certainly not readily available and, even if your data packets were end-to-end encrypted, even if nobody could access your data (definitely not the status quo), even if they cleared your data every day (it's common practice for many software companies to never clear data; it may be sent to a data warehouse, but that data is an amazing source of passive income!), even if all of those personal privacy protections were in place, you _still_ can't be sure what of your data is in plaintext. That means every time you send a photo to someone, or even a text message, all of that data and more is stored, usually in plaintext, on a server that you have _little to no access to_. That is a recipe for disastrous breaches of trust.

## Notes on GDPR

First, please note that I am not an expert on any sort of law or standards and this is not legal advice. The [GDPR](https://gdpr.eu/checklist/) (General Data Protection Regulation) put into place by the EU in 2018, is a regulation on how digital data should be handled and maintained. It lays out guidelines for E2E encryption, data anonymization, access and removal of personal data off of databases, and quite a few more minimal legal standards. The [right to access article](https://gdpr.eu/article-15-right-of-access/) is especially useful here because it allows the _European_ users of an application to access - with the proper paperwork - whatever data the company has on them. This is all fine and good except for a couple of key points:

1. The GDPR is a [European regulation](https://gdpr.eu/article-3-requirements-of-handling-personal-data-of-subjects-in-the-union/), which means that, if a company doesn't intend to offer its app or online service to citizens of the EU, even if there are users from the EU on the app, the company likely will not work very diligently to be GDPR-compliant.
2. Even large companies will make it a [pain](https://takeout.google.com/) to [access](https://www.facebook.com/your_information/) the [data](https://www.amazon.com/gp/help/customer/display.html?nodeId=202188270). This is for a couple of reasons, including:
    1. _It's very difficult to get all of a user's data,_ at least all in one place. Even requesting data from Google comes with a notice that it will likely take between hours and days to complete. When attempting to retrieve my data, the archive process even failed twice before I was able to receive my data.
    2. [Data is money](https://www.economist.com/leaders/2017/05/06/the-worlds-most-valuable-resource-is-no-longer-oil-but-data). Even if a few sources argue that the saying "data is the new oil" [isn't](https://www.forbes.com/sites/bernardmarr/2018/03/05/heres-why-data-is-not-the-new-oil/) [accurate](https://techcrunch.com/2018/03/27/data-is-not-the-new-oil/), there is no questioning that the value of big data is incredibly important to companies who thrive on ad revenue in markets for both personal data and metadata.

Overall, the GDPR tries its best, but laws and regulations like the GDPR in the age of the internet are usually treated more like suggestions than ultimatums. The only real solution is to own your own data security and to take the steps needed to maintain the level of data privacy that works for you.

## State of the Art

_I am not an expert in this field,_ but I have come across a few services that offer end-to-end encrypted chats with encrypted databases and sometimes even a feature that removes the chats (Snapchat-style) from access to the user. I'm careful to say "removes the chats from access to the user" because, like Snapchat, there isn't very much visibility into that process. Snapchat has a feature, likely in order to be GDPR-friendly, where you can [access the history of all your snaps](https://accounts.snapchat.com/accounts/downloadmydata). That means that a data owner can access your private data, and they do (usually _en mass_, so it's mostly metadata)! Having worked in various roles that had me auditing, managing, manipulating, and analyzing companies' data, I find it amazing how easily available users' data is for both technical and non-technical data owners. And trust me, a non-disclosure agreement doesn't stop most consultants from telling a story or two about outliers they see in the field.

## Idea

What if you could own your data, outright, 100%. What if you knew what data was stored on your social media, where it was stored, how it's stored, and who can access it. What if you knew everything that went through the pipes, plaintext, encrypted, files, and all. It's frankly not that hard, just ensure that you own the data pipeline, not Facebook, not Snapchat or Google, you.

### Implementation

1. The user accesses the _Obscure Chat_ protocol server at a domain like https://obsc.re. At that domain you are greeted with a static webpage, no preloading, no non-redirecting API calls, just a welcome page.
2. On the welcome page, `/`, you are given the option to enter a chat room - one of a pre-set number of rooms. Entering the chat room is equivalent to going to the page `/r/{room #}` (as short a URL as possible in order to allow more room for the room number, thus allowing more rooms).
3. Entering a "chat room" would follow the following process:
    1. Request `/r/{room #}`
    2. The response is, as always, a static webpage with no requests available back to the obsc.re server (other than to navigate back to home if necessary).
    3. The page now opens a peer-to-peer socket, to be referred to as the _Zeitgeist_ (to be discussed further in a separate _idea_), to the address.
    4. If the socket discovers a _Source of Truth_ member node (a peer which is responsible for owning the ledger of the session), it requests access to the ledger's member info to display to the user.
    5. If no _SOT_ member node is discovered, it labels itself as an _SOT_ member. It now owns the blank ledger and awaits future instruction from members that connect.
4. Once the initial connection to the Zeitgeist is established, the record, if any, of current and past members is handed to the newly established member.
5. The member checks for any known member decryption keys in the record, and then requests for all others. If on this request, any of the members deny the new member access, the messages in the message ledger that were encrypted with the denying member's key will remain undecrypted in the new member's view.
6. All members have the ability to rotate their keys (doing so results in the publishing of the intent to rotate). On key rotation, all other members request the decryption key and the rotated member may accept or deny requests as they see fit.
7. If a member is the SOT, they may transfer the SOT member status to another member who assumes responsibility of the SOT position. In this way, a member can assign a server as the SOT, this server acts as a non-participating member but will keep the ledger active for the remainder of its tenure as the SOT, even if all other members leave the chat.
8. SOT members may have various mechanisms of dismissal from the SOT position:
    1. If the SOT member leaves, the SOT position will be handed to the next available member of the Zeitgeist.
    2. If a member requests the dismissal of the SOT member and is either the only member available to do so, or is seconded, the SOT member status will be transferred to the _seconder_ (this is to allow a server SOT to request its own dismissal and then on seconding, the seconder is handed the SOT member status)
    3. Other instances are likely possible, I just haven't come up with any examples...
9. The SOT member may have a backup SOT member for safety, the value of this and the mechanism will be discussed further in future _ideas_.

### Intuition

The idea behind this implementation is that no server unless purposefully added, would have any way of accessing the data passed through any channel. It would be completely server-free, and therefore no company can own the data without the members' express permission and the members would have complete clarity on what was being transferred, not just a vague notion. In this way, the issue of data privacy is completely avoided.
