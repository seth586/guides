[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

**[Intro]** - [ [Jail Creation](1_jail.md) ] - [ [Postgresql](2_postgresql.md) ] - [ [synapse](3_synapse.md) ] - [ [reverse proxy](4_nginx.md) ] - [ [token registration](5_registration.md) ] - [ [tor ](6_tor.md)] - [ [coturn](7_coturn.md) ] - [ [call](8_call.md) ] - [ [bridges](9_bridges.md) ]

## Guide to matrix-synapse server on TrueNAS ![BSDBTC60.png](images/matrix60.png)

### Intro

[Matrix](https://matrix.org/) is a protocol for secure, decentralised, real-time communication. [Synapse](https://github.com/matrix-org/synapse) is the reference server implementation of the matrix protocol and [Element](https://element.io/) is the reference client implementation of the matrix protocol. Other servers and clients exist, but are usually forks of the main projects.

### What makes the Matrix protocol so special?

Remember email? You can send an email from 1 email provider to another! So how come all modern chat apps lack this fundemental interoperability? Why cant people on telegram chat with people on wechat? The matrix protocol specifies a server software implementation that communicates not just with clients, but with other servers, too, allowing interoperability in communication between chat hosts.

### Can't servers just censor other servers?
Sure, gmail could in theory block all communication with hotmail. In the same way, one matrix server could block their users from connecting to another matrix server.

Censorship only works on centralized networks. Consider the following network topologies:

#### The centralized server: 
One monolithic sever - Twitter - decides who you can and can't talk to on their platform. Complete centralized control rules with tyrrany. Absolute power corrupts absolutely. The individual has no choice in moderation policies. 

#### The federated network:
gnu/social server mastodon.social server blocks the gnu/social gab.com server. Individual adminstrators decide who their members can and can't talk to. Power structure is moved closer to the individual. This increases the freedom for individuals as they can now choose their servers and moderators. 

#### The decentralized network:
Every individual runs their own [dendrite](https://github.com/matrix-org/dendrite) matrix server and chat client on the same device. Completely decentralized network. The individual decides who to block and who to voluntarily associate with. 

Eventually, with enough engineering, the matrix protocol can reach a completely decentralized state. Until then, the best we can do is run a federated network topology for like minded individuals.

### Goal
By the end of this guide, you will be running a matrix-synapse server in a TrueNAS / FreeBSD jail. 

You will be able to connect a chat client to your server. I personally like [Element Secure Messenger](https://element.io/get-started).

Your server will be able to federate with other matrix servers. Send me a message at [@seth586:nym.im](https://matrix.to/#/@seth586:nym.im) to test your federation, and let me know how it went!

Public signups will be disabled, however you will have a token generator to give permissioned signups to friends. 

You will have a stun/turn server running, to help 1-on-1 WebRTC voice and video calls connections find each other (stun) or act as a relay to punch through various NAT & network topologies (turn).

You will have a jitsi server running to serve group voice and video calls.

We will also allow clients to connect over tor, allowing them to stay anonymous. Unfortunately, the matrix-synapse  server can not federate with other servers over tor, but it is on the development roadmap.

Optionally, you can set up [bridges](https://matrix.org/bridges/), to communicate with other popular chat services.

### Target Versions

This guide was written and tested to work with FreeBSD 12.2 and Synapse 1.40. If installing a newer version (likely) catch up on the [upgrade notes](https://matrix-org.github.io/synapse/latest/upgrade) to ensure these instructions and configurations are still accurate. 

Next: [ [Jail Creation](1_jail.md) ]
