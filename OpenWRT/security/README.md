[ [<< Back to Main Menu](https://github.com/seth586/guides/blob/master/README.md) ]

**[Intro]** - [ [OpenSSH Client](1_install_client.md) ] - [ [OpenSSH on OpenWRT](2_install_openssh.md) ] - [ [Generate Keys](3_keys.md) ] - [ [Bastion](4_bastion.md) ]

## SSH Bastion Security Hardening Guide
### Intro
Password based authentication for your SSH sessions and web user interfaces are not secure. This guide will configure your home router and freenas server to utilize public key cryptography for authentication.

### Threat Model - Passwords
Up to this point you have probably been authenticating your SSH sessions for your router and freenas with a password. Lets go over the threat models you currently face with this configuration:

1. Passwords can be brute forced. 

2. Passwords can be keylogged. 

Does your daily driver laptop have malware? Did you ever copy/paste a password on your phone? Guess what, apps on your phone can monitor the clipboard! 

3. Passwords can be leaked. 

Are you using unique passwords for every single password based service you use? If not, database breaches will dump used passwords onto the internet, giving brute force attackers a much smaller range of passwords to try. If an attacker is smart, they can target your router and freenas server with passwords credited to your use on other services.

4. Every device behind your firewall is a liability.

*"But wait!"* You say, *"None of this matters because my router blocks port 22 from the public internet!"* This is true, no one from **outside** your router's firewall can attempt to enter a password for your SSH session (unless you are a complete fool and publically forward port 22). But can we truly trust every device inside our firewall? What about the numerous internet of things inside your home? By default, every internet connected device has LAN access to your home server and router. Your amazon echo or smart light bulb or internet connected coffee machine could be attempting brute force attacks on your router right now!

### Mitigation with Public Key Cryptography

1. Public Key cryptography can not be brute forced.

2. Keylogging the public key is useless, since authentication is based on proving that you know the private key without ever revealing the private key.

3. Leaking or posting your public keys are safe! Post them on twitter for the world to see! 

4. If configured correctly, you could forward port 22 safely utilizing public key authentication.

### Basic overview on how it works

Lets define the three devices that you are going to use in a common home server management scenario:

Server #1: Your home router

This guide will refer to all console commands on your home router with a proceeding `root@OpenWrt:~#`.

Server #2: Your FreeNAS server

This guide will refer to all console commands on your home FreeNAS server with a proceeding `root@freenas:~#`

Client: Your home desktop, laptop, cell phone, tablet, etc.

This guide will refer to all console commands on your client with a proceeding `$`

There are two files you must manage when utilizing public key cryptography: a **Public Key** and a **Private Key**. Your servers only need the public key on them. Your clients only needs the Private key. You can recreate a public key from a private key, but you can not create a private key from a public key. Hence it is important to not lose your private key! **If you lose your private key, you lose all ability to SSH into your devices!** 

Brain exercise: You can theoretically give your bank your public key, and authenticate using your private key on your client device. If your bank experiences a data breach and your public key is now revealed to the whole internet, your accounts are still secure! Logging in proves you own the private key without ever revealing the private key to your bank! Pretty cool, huh? 

This is called 'passwordless login', and this is how FIDO/U2F devices (such as the yubikey) work: They give your bank, email, social media accounts a public key, and the device safely stores the private key to create signatures to authenticate without ever revealing the private key.

**It is paramount that we secure this private key**, so lets look at the threat model in this scenario:

### Threat Model - Stolen Private Key

A private key can be stolen since its just a computer file. Perhaps your laptop was stolen and you have your private key on the laptop. Lets discuss how we might mitigate this security threat:

1. Have multiple private keys

We can configure our servers with multiple public keys corresponding to multiple private keys. If our laptop is stolen, we can use a different private key to authenticate, then remove the public key corresponding to our stolen private key. But what if we don't know the private key is stolen? Perhaps malware scanned our hard drive and phoned home our private key. There is a better way to secure a private key:

2. Derive a private key with multiple inputs

We can derive a private key requiring more input to fully decrypt it, such as also requiring a password and/or a FIDO/U2F device! By combining something we **HAVE** (private key file) with something we **KNOW** (password) and something **PHYSICAL** (FIDO/U2F or yubikey), we encrypt the private keys with multiple layers of security. If our private key is stolen, its still secure because its encrypted with a password and/or a FIDO/U2F device.

So now that we know how we can secure the private key, what threat models remain?

### Threat Model - Losing a private key 

Make multiple backups on multiple devices, such as USB thumb drives. Use an offline QR code generator, print out the private key, and store it in a fireproof box. Use an old-school printer, modern wifi or network-enabled printers are extremely unsecure! Always have a private key backed up in a geographically seperated location!

If you are adding a third layer of protection with a FIDO/U2F device, buy two or three of them, and create private public key pairs for each of them. If you lose or break your FIDO/U2F device, the other two will save your butt!

### Threat Model - the Web User Interface
Securing SSH is half the battle. Your OpenWRT and FreeNAS web user interfaces are also privileged entrances to your system. They may not have complete file system access, but they control the permissions on who gets access! Are you running the [bitcoin & lightning guide](https://github.com/seth586/guides/blob/master/FreeNAS/bitcoin/README.md)? RTL's web ui gives unrestricted access to your funds! Imagine a smart light bulb trying to brute force RTL's password authentication inside your home noetwork, scary and plausible scenario if we don't upgrade to public key authentication!

In the guide we wil redirect web based interfaces to only work over an authenticated SSH session. Hopefully in the near future these web-UIs can integrate second factor authentication like TOTP or FIDO/U2F so we don't have to worry about this step.

### Summary
We briefly discussed the threats we face using passwords and why public key cryptography is more secure. We also touched on the importance of not losing your private key, and how to adequately secure your private key. So lets move on and create a private key to access our SSH servers securely!


Next: [ [OpenSSH Client](1_install_client.md) ] >>

