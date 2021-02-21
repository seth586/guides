[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

**[Intro]** - [ [Jail Creation](freenas_1_jail_creation.md) ] - [ [Bitcoin](freenas_2_bitcoin.md) ] - [ [Tor](freenas_3_tor.md) ] - [ [Electrum](freenas_4_electrum.md) ] - [ [lnd](freenas_5_lnd.md) ] - [ [loopd ](freenas_5a_loopd.md)] - [ [RTL](freenas_6_rtl.md) ] - [ [mempool](freenas_8_mempool.md) ] - [ [Extras](extras.md) ]

## Guide to matrix-synapse server on TrueNAS ![BSDBTC60.png](images/BSDBTC60.png)

### Intro

[Matrix](https://matrix.org/) is a protocol for secure, decentralised, real-time communication. [Synapse](https://github.com/matrix-org/synapse) is the reference implementation of the matrix protocol.

### What makes the Matrix protocol so special?

Remember email? You can send an email from 1 email provider to another! So how come all modern chat apps lack this fundemental interoperability? Why cant people on telegram chat with people on wechat? The matrix protocol specifies a server software implementation that communicates not just with clients, but with other servers, too, allowing interoperability in communication between chat hosts.

### Can't servers just censor other servers?
Sure, gmail could in theory block all communication with hotmail. In the same way, one matrix server could block their users from connecting to another matrix server.

Censorship only works on centralized networks. Consider the following network topologies:

#### The centralized server: 
Twitter decides who you can and can't talk to on their platform. Complete centralized control rules with tyrrany. Absolute power corrupts absolutely. The individual has no choice in moderation policies.

#### The federated network:
gnu/social server mastodon.social server blocks the gnu/social gab.com server. Individual adminstrators decide who their members can and can't talk to. Power structure is moved closer to the individual. This increases the freedom for individuals as they can now choose their own moderators. 

#### The decentralized network:
Every individual determines who they want to block. 



This guide 

Next: [ [Jail Creation](freenas_1_jail_creation.md) ]
