---
layout: docs
title: Protocol specification
mathjax: true
---

At its heart, Private Data Donor relies on a cryptographic protocol, first presented in the paper [Efficient Private Statistics with Succinct Sketches](https://arxiv.org/abs/1508.06110) (by L. Melis, G. Danezis and E. De Cristofaro).
We present on this page its formal definition, and how it is actually used in the Private Data Donor platform.

* TOC
{:toc}

## Mathematical definition

We use $$N$$ to denote the numbers of clients in the system, and $$L$$ the number of monitored queries.
$$C_i, i \in [1,N]$$ denotes the $$i$$-th client, and $$W_j, j \in [1,L]$$ denotes the $$j$$-th monitored query.
Let $$\mathbb{G}$$ be a cyclic group of prime order $$q$$ for which the Computational Diffie-Hellman problem is hard, and $$g$$ be the generator of the same group.
Also, we use a cryptographic hash function, $$H: {0,1}^* \rightarrow \mathbb{Z}_q$$, mapping strings of arbitrary length to integers.
Let "$$||$$" denote the string concatenation operator, and $$a \in_r A$$ indicates that $$a$$ is sampled at random for $$A$$.
The Private Data Donor protocol involves three steps.

**Initialization.**
Each client $$C_i$$ generates a private key $$x_i \in_r \mathbb{G}$$ and a public key $$y_i = g^{x_i} \bmod q$$.
The client then sends its public key along with its identifier $$i$$ to the server to be registered.

**Encryption.**
At each round $$s$$, client $$C_i$$ holds a counts vector $$V_{si} = (v_j \in \mathbb{N}, j \in [1,L])$$, where $$v_j$$ represents the number of times the query $$W_j$$ was searched for.
To encrypt this vector, client $$C_i$$ first generates a blinding factors vector:

$$
K_{si} = \Big( \sum_{\substack{j=1, j \neq i}}^N H(y_j^{x_i} || \ell || s ) \times (-1)^{i > j} \bmod q, \ell \in [1, L] \Big)
$$

where $$(-1)^{i >j} = -1$$ for $$i > j$$ and $$1$$ otherwise.
Then, each $$C_i$$ encrypts its counts vector $$V_{si}$$ into an encrypted vector $$\hat{V}_{si} = V_{si} + K_{si}$$
which is then to be sent to the server.

**Decryption.**
Once the server has received the encrypted vectors for all clients at round $$s$$, it can decrypt it by summing them:

$$
V_s = \sum_{i=1}^N \hat{V}_{si} = \sum_{i=1}^N V_{si} + \sum_{i=1}^N K_{si} = \sum_{i=1}^N V_{si}
$$

Indeed, the sum of all $$K_{si}$$ is a zero vector:

$$
\forall \ell \in [1,L], \sum_{i=1}^N K_{si_\ell} = \sum_{i=1}^N \sum_{\substack{j=1 \\ j \neq i}}^N H(y_j^{x_i} || \ell || s ) \times (-1)^{i > j} = 0
$$

## Practical mitigations

This protocol has one main drawback, which is that if a single client does not send its encrypted vector, it is impossible to decrypt the data for all other clients for the current round (as the sum of all $$K_{si}$$ will not equal to zero anymore).
If not handled properly, this may severely decrease the utility of the platform, as clients may join or quit the data collection process, and thus leave data encrypted forever.
We introduce to measures to mitigate that risk.

First, users are organised into groups of a fixed size (between 10 and 100 users), and the protocol is played independently inside each group.
At the end, the decrypted vectors for all groups are trivially summed.
Consequently, if a single user never sends its encrypted data, it will prevent the decryption of one group of a controlled size, while still allowing to decrypt the vectors of the other groups.

Second, some delay $$\delta$$ is introduced before the decrypted data is published, allowing clients that may not have been online on day $$d$$ to send their data on any day $$d+1, d+2, ..., d+\delta$$.
This will not allow to recover if the client definitively left the data collection, but will still allow to recover from a client that does not come online every day.
[As Mozilla pointed out in a study](https://blog.mozilla.org/data/2017/09/19/two-days-or-how-long-until-the-data-is-in/), most users have a frequent usage of their browser, and used it most days.

## Actors in Private Data Donor

The following sequence diagram summarises the interactions between the clients and the server.
In Private Data Donor, clients have the form of a browser extension deployed inside the volunteers' browser, while the server is managed by the institution collecting data (i.e., University College London for the official instance).

![Sequence diagram](/assets/images/sequence.png)

Upon installation, clients generate a pair of cryptographic keys and contact the server to register themselves.
The server replies with a *client name*, which is to be used by the client as a mean of authentication against the server for further calls.
Then, once a day, the client contacts the server with a so-called *ping*, which returns some relevant information to the client, among which a list of commands to perform.
Each command represents an encrypted vector (also called sketch internally) that is to be computed and sent to the server.
A command contains a period of time, a set of queries of interest and the public keys of the members of the group.
Client $$C_i$$ uses these information to extract from the local browser history the searches that match the command and generate a counts vector $$V_{si}$$.
It then generates blindings factors $$K_{si}$$ by using the public keys contained in the command, and uses it to encrypt to counts vector.
Lastly, each encrypted count vector is submitted as a sketch to the server.

It may happen that, when pinging the server, a client receives a response indicating that it is not known to the server.
The reason for this is usually that the client has been somehow evicted from the data collection process, e.g., because it has been inactive for a long time.
In that situation, the client first has to re-register itself to the server, before pinging it again.

Please note that clients may receive multiple commands to be computed at each ping.
Indeed, the server allows to create multiple data collection campaigns, each one of them monitoring its own set of queries of interest.
In that case, every active campaign will be materialised as a different command.
Clients may also receive multiple commands if they "missed" one or several pings in the past days because they did not have any connectivity.
In this case, they may receive additional commands to catch up with days inactivity, up to the maximum delay $$\delta$$ configured in the campaign.

## Implementation details in Private Data Donor

The current implementation uses [Elliptic-curve cryptography](https://en.wikipedia.org/wiki/Elliptic-curve_cryptography) to implement the cryptographic operations, more specifically, the [Ed25519](https://en.wikipedia.org/wiki/EdDSA#Ed25519) scheme.
The hash function $$H$$ is the [SHA-256](https://en.wikipedia.org/wiki/SHA-2) function.
Finally, we use the regular string concatenation operator "$$||$$", as provided by the implementation language.

The encrypted numbers generated by this protocol should be manipulated with case, as they are very likely to exceed the limits permitted for even long integers.
Most languages have libraries to handle big integers.

The round number $$s$$ described in the protocol is provided by the server as part of the command.
Even though in practice, it should map to a day index (since the campaign started), it should be treated as an opaque value.
