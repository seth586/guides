[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

**[Intro]** - [ [Jail Creation](1_jail_creation.md) ] - [ [murmur](2_murmur.md) ] - [ [SSL & Domain](3_ssl_domain.md) ] - [ [Basic ACL Config](4_acl.md) ]

## Guide to Mumble server (murmur) on FreeNAS/TrueNAS
### Intro

![JailMumble](images/mumble.jpg)

Discord is currently the most popular voice chat system among many communities, however they are a closed source platform, do not offer end to end encryption, and frequently engage in censorship. 

The mumble server, [called murmur](https://wiki.mumble.info/wiki/Running_Murmur) offers us complete, uncensorable control of our voice chat community. Mumble offers end to end encryption, and a very powerful Access Control List ruleset that allows you to give special permissions to your users. ACLs are a little hard to udnerstand at first, so at the end of the guide there is a basic ACL configuration example to get you started.

### Objective
By the end of this guide, we will have a mumble server configured to a domain name that you can hand out to friends and strangers alike. A basic access control list will be configured into a three tier privilege system of:

Anonymous User - upon connnecting to the server they will be in the root lobby with no privileges to talk, type, or move to a lobby.
Registered User - (for your friends) will have privileges to talk, type, and move themselves and anonymous users between lobbys.
Adminsitrator User - (for your trusted friends and family) will have privileges to promote a user to registered user, as well as kick, ban and unban privileges.

Next: [ [Jail Creation](1_jail_creation.md) ] >>
