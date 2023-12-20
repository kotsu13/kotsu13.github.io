---
layout: post
title:  "Some thoughts about Keybase"
description: "Random musings about Keybase"
date: 2018-06-18
img: # Add image post (optional)
fig-caption: # Add figcaption (optional)
tags: [keybase]
---

I recently came to know about Keybase through a friend, and thought it was a pretty interesting product.

For one: sometimes identities on the internet are hard to prove, and supposing you know someone's Keybase profile (maybe through meeting them in real life), you have a way to verify that they own a particular social media account, because of cryptographic proofs.

The other reason is a bit more on the idealistic side - I was wondering if I could glean some ideas on whether its possible to make e2e emails more user friendly.

Having attempted to send PGP-encrypted emails a few times, I realised the process does have some frustrating portions at the start. In terms of email client choices, all I had was Thunderbird for both Windows/Linux since I didn't want to consider browser-based plugins. You can then generate a PGP keypair via command line or through the UI on Thunderbird. One of the main things I considered was how to port about my PGP key so I could still decrypt my emails if I used another PC. Do I really want to put that key onto a thumbdrive...? 

To that end, I bought a cheap <a href="https://www.sparkfun.com/products/retired/11280">Maple Mini</a> clone off Aliexpress for only USD$2, soldered the pins on, and installed <a href="https://github.com/me21/gnuk">GnuK</a> on it. This was great if I was using 2 different computers and wanted to use the same PGP key from my maple mini. But I later wondered how I would easily encrypt my email if I sent it from a phone. In the end, I gave up entirely as I didn't need to send encrypted emails very often anyway.

The other issue was that it often takes multiple steps to import someone's PGP public key, either from them sending an introduction email with their key on the first send, or downloading it from other sources (eg. their website), then registering it with Thunderbird.


Eventually, after reading the <a href="https://book.keybase.io/docs/chat/crypto">Keybase crypto docs</a>, I realised several things.

1. The email eco-system is not homogeneous compared to chat platforms. There are multiple email providers available, such as gmail, yahoo, hotmail, etc. In Keybase, when you start a new chat with a user, Keybase can automatically settle the key exchange with a protocol as it is their own platform. Every time a new device is added to your account, the per-device keys are used to transfer the 32-bit symmetric key for every chat you have from an existing device to the new device. On email however, the concept of generating or importing keys may be a bit confusing for a non-technical user, not to mention that you cannot send an e2e email without first knowing your recipient's public key. Would there ever be a protocol for such key exchange for emails that will be widely adopted? 

2. While desktop apps for emails exist, it is also pretty common for people to just use email on the browser. When used on the browser, there needs to be a way to access the PGP private key in order to decrypt emails. There are some possibilities here: you can upload this private key to the server (do you trust your mail server?) or even use browser plugins like Mailvelope. However, not all mobile browsers support browser plugins yet.
