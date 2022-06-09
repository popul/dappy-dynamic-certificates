# Use Dappy as remote trusted certificate store

Today, managing certificates is a pain for many companies. When exposing a B2B portal, online SaaS or APIs are exposed to many security and downtime risks coming from corrupted certificates or simply downtimes. When a company discovers a security breach such as a Man-In-The-Middle TLS interception, it is often too late. Expired certificates also expose clients and partners to security warnings and frictions ultimately reducing their trust.

**Post incident analysis raises the following questions : Why are we are in this situation and can we do anything to remediate it ?**

## What are certificates and stores ?

Certificates are the angular stone of TLS protocol and allow two computers to communicate in a secure way. They bring data integrity, confidentiality and authentication. There are two types of certificates. The first ones are **server certificate**. They are delivered by companies and institutional servers and contain the server's identity. Server certificates are signed by a second type of certificate, **trusted certificates** (intermediate and root certificates). Trusted certificates are installed on client devices, such as computers and mobile phones. A server certificate is trusted if signed by a trusted certificate known by the end user, this is the concept of [chain of trust](https://en.wikipedia.org/wiki/Chain_of_trust).

In a public web context, trusted certificates are distributed over Web PKI root stores, authored by browser companies like [Mozilla](https://wiki.mozilla.org/CA), [Microsoft](https://docs.microsoft.com/en-us/security/trusted-root/program-requirements), [Google](https://www.chromium.org/Home/chromium-security/root-ca-policy/), [Apple](https://www.apple.com/certificateauthority/ca_program.html). Root stores contain hundreds of certificates owned by public [certificate authorities](https://en.wikipedia.org/wiki/Certificate_authority). The trusted certificates have to be installed on their local stores in order to be trusted, this is usually done during software updates.

## What's wrong with certificates stores ?

Firstly, the local list of trusted certificate is the point of reference, each client has one, it does not matter that a web server has a certificate signed by "Let's Encrypt" if the clients don't have the Sectigo SSL trusted certificate installed. Local trusted certificates may differ from trusted certificates listed by Web PKI root stores. Quite obviously, the larger this gap, the higher the security risk. Browsing the public web with an outdated local certificate store can be extremely insecure.

Secondly, certificates can’t be easily repudiated. The cryptographic proof (signature) is valid forever because a certificate has been signed by a trusted certificate (to be exact, it has been signed by its corresponding private key). OCSP and CRL protocols permit repudiation, the burden of checking the validity of certificates is on the softwares that interpret the metadata. These protocols are overlays, they can be implemented or interpreted differently by softwares, **ultimately, OCSP and CRL do not and cannot invalidate CA signatures**. Sadly, these protocols do not guarantee that clients or partners are browsing your website with only valid and non revoked certificates. Of course, setting an expiration date minimizes the attack surface, but that's only half the solution. Moreover, how would you install the new certificate before the expiry date in the local root store ? 

If you're following along so far, you understand that **your server identity can be impersonated or revoked by any certificate authority.** For example, on apple devices, [164 root certificates](https://support.apple.com/en-us/HT212140) are valid. Who can guarantee today that countries like United States, China or some private corporations are not conducting shady interceptions in relation with the trusted certificate stores ? War in Ukraine recently [illustrated](https://www.bleepingcomputer.com/news/security/russia-creates-its-own-tls-certificate-authority-to-bypass-sanctions/) that the chain of trust is easily corruptible. It takes only one trusted certificate authority to be compromised for your (or your client's) TLS communications to be exposed to Man-In-The-Middle attacks. MITM attacks are partly mitigated by [CT Logs](https://datatracker.ietf.org/doc/html/rfc6962), but still possible. This attack is called TLS interception and is widely used in company intranets or private networks to prevent data leaks. But few people realize that we are also exposed to this attack on the public web. 

**Relation with DNS and the public names**

[DNS](https://en.wikipedia.org/wiki/Domain_Name_System) is the authority for the management of public names. We now have a unique registry for public names, but on the other hand many root certificate authorities as root certificate authorities. Unlike DNS (domains, subdomains, TXT records), it is hard to know all the certificates linked to a public name, CAs are opaque. A public PKI system should offer visibility, that is not an easy task with today's infrastructure. If you are the owner of a domain name, no one should be able to impersonate you (through name, or cryptographic proof), but as argued earlier the PKI and root stores of the public web make this hard to achieve. What happens when a domain name has a new owner ? You guessed it … certificates are still valid. And conversely, why does a certificate become invalid despite the fact that name ownership hasn’t changed.

Many attempts were made to patch DNS and allow DNS zones to store TLS certificates, [RFC 4398](https://www.rfc-editor.org/rfc/rfc4398), or to save certificate fingerprints with [DANE](https://datatracker.ietf.org/doc/html/rfc6698). But DNS failed to be secured using [DNSSEC](https://datatracker.ietf.org/doc/html/rfc4033) in an end to end way.

As we discussed, many problems are related to local root certificate stores.  If a revoked certificates is still used by end users,  Also, in practice, anyone who wants to secure their public communications, **has** to provide a certificate signed by one these trusted certificates. A public web identity is in reality owned by certificate authorities, whom are all exposed to security issues. If just one authority is compromised, all public secured communications are compromised because a root authority can create a certificate for any domain **without the owner's permission**. Today, we implicitly accept this **single point of failure** because it's the only effectuve way to secure communications and transactions on Internet.

A public web identity is in reality borrowed from certificate authorities. If just one authority is compromised in any way, all secured public communications are compromised, a root authority can create a certificate for any domain **without the owner's permission**. We implicitly accept this **single point of failure** because it's the only effective way to secure communications and transactions on the internet.

At FABCO, we are convinced that too much trust is delegated to local stores and their trusted certificates. No system can guaratee that some revoked certificates are not used, this will be a gigantic issue as companies continue to digitize their operations, and deploy critical web applications on the public Internet.

## No more local trusted store ! 

People and companies just want to communicate in a private manner on Internet. For communications lacking significant value, the PKI of the public web is acceptable. Is it an acceptable solution for B2B portals that handle critical operations each day ? More and more value will be managed through APIs and web interfaces in the coming years, web cryptocurrency exchanges already handle daily volumes that are in the Billions, [home ownership can be digitalized using NFTs](https://www.forbes.com/sites/forbesbusinesscouncil/2022/02/16/nfts-and-the-future-of-commercial-real-estate/) and exchanged publicly. Not surprisingly, the number of cyber attacks has never been so high.


In order to secure a web/TLS communication we should just have to create a certificate, distribute it with a web server and then clients just have to trust it. But how ? **The dappy name system** is a public name system and PKI system that aims at solving the aforementioned issues related to certificate revokation, lack of transparency, poor monitoring capabilities and TLS interception possibilities.

---- BEGIN REPETITION (pour moi la partie suivante répète ce qui a été dit 2 ou 3 fois déjà)

On public web, server certificates are signed by trusted certificates that are preinstalled on clients. As argued earlier, these trusted certificates can be outdated or worse, compromised and revoked. How to garantee to our client to use latest trusted certificates ? Issuing signed certificates is not an easy task, specially for companies or teams missing security specialist. Maybe other ways exist to enabling needed trust.

---- END REPETITION

----- BEGIN COMMENTAIRE

Plutôt ok pour la suite mais je pense pas qu'on doive présenter rchain autant, plutôt les propriétés principales du dappy name system:
- Own a TLD
- Distribute a trusted certificate along side this TLD
- Clients do co-resolution with independant peers

----- END COMMENTAIRE

[**Dappy name system**](https://github.com/fabcotech/dappy-propositions/blob/master/01_co_resolution.MD) is a hierarchical name system like [DNS](https://en.wikipedia.org/wiki/Domain_Name_System), that allows anyone to own [TLD](https://en.wikipedia.org/wiki/Top-level_domain). Dappy name system based on [RChain](https://rchain.coop/), a fully distributed and decentralized public database technology, censorship resistent. RChain brings two main benefits to Dappy name system. It resolves the blockchain trilemma. Decentralization, security and scalability are key points for a public name system like Dappy. Secondly, RChain is enough scalable to persist complex data structures like certificates.

First of all, as **remote trusted certificate store**, Dappy name system distributes trusted certificates to end users. Applications built on top of dappy technologies, such as the [dappy browser](https://github.com/fabcotech/dappy), dappy agent (name and certificate client resolver), and librairies like the [dappy lookup](https://github.com/fabcotech/dappy-lookup) will fetch **certificates dynamically**, removing problems with revocated trusted certificates. Data integrity is resolved using [co-resolution](https://github.com/fabcotech/dappy-propositions/blob/master/01_co_resolution.MD#4-co-resolution), a trustless mecanism, used to query Dappy name system.

Trusted certificates are **always related to their domain** name and they are resolved at runtime by walking through the dappy name system from leaf to root, during name resolution. Companies and institutions don't have to trust and transfer their identity to a third party, like a public certificate authority. 

But **Dappy name system** wants to do more for companies or teams that didn't have their own PKI or don't want to use an external one. To enable clients to trust their server certificate, the idea is to save certificates fingerprints in Dappy name system, in TLSA records like [DANE](https://datatracker.ietf.org/doc/html/rfc7671) does. In that way, clients can verify the authenticity and integrity of server certificates without PKI. It give access 

## What's next ?

Interested by our security solutions ? We provide free assistance for companies that wish to try dappy, you can reach out to us by email contact[at]fabco.tech or through the [dappy.tech/hello](https://dappy.tech/hello) form.
