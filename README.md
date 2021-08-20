# Syphre — a quantum-safe cryptographic protocol, custom-built for Syvita and the Web3 industry.

> THE SYPHRE SPEC’ IS A WORK IN PROGRESS AND COULD CHANGE AT ANY POINT

----

Syphre is a protocol designed to provide guidelines for Syvirean projects as well as the Web3 industry as a whole. It is being conceived to replace TLS for the Lightlane protocol since TLS has centralisation & post-quantum issues, and is also expanding to be a general protocol for securing data.

The main issue with TLS is that public keys need to be integrated in a signed certificate, which creates centralisation at the root Certificate Authorities. Syphre is designed to not require any such certificate (more on this in a bit), and also to be faster than TLS and quantum-safe by default.

----

Unlike TLS, with Syphre a certificate isn’t required to verify that the server the client is connecting to is indeed the correct one and isn’t being changed by something like a MITM attack. In TLS, a client assumes that a Certificate Authority (a trusted public key) has checked that the public key used for encryption is owned by the IP or domain owner. These public keys are traditionally hardcoded into every user device so it can verify the server public key.

This is unneeded in Syphre because we have a trusted public key already from the Atlas Peer Network and BNS. In the BNS zone file that stores the IP address of a BNS name or subdomain, Lightlane also stores approved public keys for that name/subdomain. A hash of this zone file is cryptographically signed by the Stacks address that owns the name and therefore we can trust the zone file to have originated from the owner. This means we don’t need to verify the authenticity of the public key.

This also removes the need for any handshake at all when using Syphre. Because we use a set version of protocols, not only is implementation easier for developers, there is also no need to handshake to get the public key or establish algorithms for the connection. We know them in advance, since they are stored on Atlas in the same zone file that gives Lightlane its DNS-like functionality. This is partly what makes Lightlane so insanely fast — as well as private — the first connection isn’t distinguishable from any other connection. All data looks like identical gibberish to any network watchers. All metadata is minimised as much as possible.

----

[Bitcoin Q&A_ Migrating to Post-Quantum Cryptography.mp4](https://res.craft.do/user/full/99e68972-962a-3ee2-6069-ddaf255ca93d/doc/59590CC0-B0A1-4A21-AF4A-F41B69C4C62A/4F2E9BEF-10C1-4EB8-865F-ABF10081A2FA_2/Bitcoin%20QA_%20Migrating%20to%20Post-Quantum%20Cryptography.mp4)

A video by aantonop detailing how Bitcoin can move to quantum-safe algorithms. Once this happens (if not prior), Stacks will also migrate to the same quantum-safe algorithms. This is limits Syphre’s security currently, as it relies on the public keys in the BNS/Atlas zone file to be trusted. Currently, Stacks uses ECDSA for its digital signature system. This is vulnerable to quantum-based attacks, unlike the signature system that Syphre uses — FALCON.

----

## Specification

We chose the algorithms for Syphre based on NIST’s PQC project which you can check out [here](https://csrc.nist.gov/projects/post-quantum-cryptography). Pre-quantum combinations (such as used in TLS) first used RSA, then moved to elliptic curve based cryptography to improve speed, security, and efficiency. Both RSA & elliptics are considered secure in regards to classical computing, but are not secure in a quantum world. This means elliptic curve based tech that we currently use will have to be replaced with quantum-safe alternatives.

Some of the most prominent new primitives are lattice-based. So far, some lattice-based constructions appear to be resistant to attack by both classical and quantum computers. Furthermore, many lattice-based constructions are considered to be secure under the assumption that certain well-studied computational lattice problems cannot be solved efficiently.

Syphre is entirely based on lattice-based cryptography for its digital signatures, key encapsulation, and homomorphic encryption (where lattices are required). Hashing algorithms (like BLAKE3, SHA256) as well as ciphers (like AES, ChaCha20) appear to be secure from quantum-based attacks and so don’t need to be replaced. We combine XChaCha20 with BLAKE3 for extremely fast symmetric transport encryption in Syphre.

Here’s a list of the functions that Syphre defines, with their shortened terms in brackets.

----

Digital signatures (DSA) — FALCON-512
Key encapsulation (KEM) — LightSABER 
Symmetric encryption (STE) — XChaCha20 & BLAKE3 
Homomorphic shared storage encryption (HSSE) — ZamaTFHE 
Slugged hashing (sHash) — SHA-256d 
Volant hashing (vHash) — BLAKE3 
Key derivation (KDF) — BLAKE3 
Pseudorandom generation (PRF) — BLAKE3 
Message authentication (MAC) — BLAKE3

----

### Detailed specification for each function

#### DSA: FALCON-512

Out of the 3 choices we had out of NIST’s Round 3 finalists, we chose FALCON.

This was a pretty simple choice. We needed small public keys to fit into Atlas zone files (which are restricted to 40 kilobytes) as well as very fast signing time. Signature size wasn’t much of a consideration, as long as it wasn’t above 1 kilobyte.

Rainbow was instantly taken out of consideration — even with its extremely small signatures compared to other PQDSAs, its public keys are huge — 157.8 kilobytes for its lowest security level, and increasing to ~1.9 megabytes for its top level!

Dilithium is also then taken out the equation, being considerably slower than FALCON-512 in its Dilithium3 variant, and its public keys and signature sizes being a lot more than FALCON’s too. Which ends us up with FALCON-512!

#### KEM: LightSABER

We wanted to use a Key Encapsulation Mechanism instead of a Public-key Encryption scheme, so it was between CRYSTALS-KYBER and SABER. Both algorithms have very similar speeds and sizes, so we could’ve picked either based on our speed requirements.

However, in contrast to KYBER, the running time of SABER is even constant-time over various different public keys as there is no rejection sampling. Due to the power-of-two moduli, this means the communication (public key, cipher text) looks like uniformly random bits and contains no structure. This makes SABER well-designed for anonymous communication, which is why we chose SABER over KYBER. Syphre was primarily built to be Lightlane’s cryptographic framework, where the whole point is anonymous communication by default!

We also chose the LightSABER variant as it gives a post-quantum security level similar to AES-128, which is considered secure. This gives us a high-speed and secure KEM, which happens to also factor privacy in too.

#### STE: XChaCha20 with BLAKE3

We chose XChaCha20 specifically because it needed to be super-fast. We wanted to use BLAKE3 for its speed, which meant using it for both authenticating the cipher and creating a random nonce too. XChaCha20 allows us to use these random nonces generated through BLAKE3 securely, while also being very, very fast and lightweight.

#### HSSE: ZamaTFHE

TFHE’s full scheme name is *Fast Fully Homomorphic Encryption over the Torus*

We are also adding support for fully homomorphic encryption in Syphre (though most likely not used in Lightlane). Homomorphic encryption allows you to perform operations on its encrypted data without first decrypting it. These resulting computations are left in an encrypted form which, when decrypted, result in an identical output to that produced had the operations been performed on the unencrypted data.

Homomorphic encryption is game-changing since it can enable new applications. For example, predictive analytics in health care can be hard to apply via a third party service provider due to medical data privacy concerns, but if the predictive analytics service provider can operate on encrypted data instead, these privacy concerns are diminished.

We can create a healthcare system where user-owned data is shared with healthcare providers using homomorphic encryption, so the provider can operate on the data to perform calculations, while the actual content of the data is securely hidden from the provider and is also entirely user-owned. But that’s a whole different topic that I’ll cover in another post.

Homomorphic Shared Storage Encryption (HSSE)

We’re calling this HSSE by adding in “shared storage” to “homomorphic encryption”. This is really important since this is the main use of TFHE for us — securely and privately sharing data from a user to a service provider. Our thinking is that a user would share their data using a Syphre-based HSSE protocol from their Gaia hub (for example) to a service provider like a health care system which doesn’t have any access to the plain data. They can use the data without storing it through the user’s Gaia hub, while not having access to the data itself. The possibilities are endless and HSSE is something we’re really excited for.

It’s worth saying FHE (*fully* homomorphic encryption) appears highly experimental as of now and is still in development. We are monitoring its progress in detail and will post updates if we manage to make any POCs! FHE is integrated in Syphre using Zama’s variant of the TFHE algorithm. We chose Zama's variant it implements programmable bootstrapping or PBS. Data flows (such as neural networks) are represented as efficient linear combinations of univariate functions instead of expensive boolean circuits. This makes it a lot more efficient when data needs to be passed through data flows like neural networks, on top of all the benefits TFHE brings by itself. Read the whitepaper below!

[ZamaTFHE.pdf](https://res.craft.do/user/full/99e68972-962a-3ee2-6069-ddaf255ca93d/doc/59590CC0-B0A1-4A21-AF4A-F41B69C4C62A/358DA424-0627-44D5-9ACC-784D1DA29044_2/whitepaper.zama.ai.pdf)

You can use this through [Concrete](https://concrete.zama.ai/) — an alpha Rust library that lets you use Zama’s variant of TFHE in real apps.

FHE also happens to be based on lattice-based cryptography. Coincidence? We think not! LBC is the next big thing in cryptography, and Syphre’s going to be leading that edge for mainstream development.

#### sHash: SHA-256d

The slugged hashing algorithm we chose for Syphre is SHA-256 from NIST's SHA-2 series. SHA-256 remains quantum-safe and has been around for a long time. Bitcoin uses it, Stacks uses it and it has native implementations in Clarity. We use SHA-256d (aka, `SHA256(SHA256(x))`) to protect against length-extension attacks.

For a slugged hashing algorithm that’s used when hashes are needed but speed is not, SHA-256d is perfect for Syphre.

#### vHash: BLAKE3

The vHash function, or volant hash function, is designed to rapidly hash data. It shouldn’t be used to hash secrets, since it’s optimised to be fast. It has streaming capabilities and is very efficient.

BLAKE3 is based on an optimised instance of the established hash function BLAKE2 and on the original Bao tree mode. The specifications and design rationale are available in the BLAKE3 paper. The default output size is 256 bits. The current version of Bao implements verified streaming with BLAKE3.

![upscaled-removebg.png](https://res.craft.do/user/full/99e68972-962a-3ee2-6069-ddaf255ca93d/doc/59590CC0-B0A1-4A21-AF4A-F41B69C4C62A/0C0DB43E-3F8F-427A-B89A-0BEB3CE699C8_2/upscaled-removebg.png)

#### BLAKE3 is a cryptographic hash function that is:

- **Much** **faster** than MD5, SHA-1, SHA-2, SHA-3, and BLAKE2.
- **Secure**, unlike MD5 and SHA-1. And secure against length extension, unlike SHA-2.
- **Highly parallelizable** across any number of threads and SIMD lanes, because it's a Merkle tree on the inside.
- Capable of **verified streaming** and **incremental updates**, again because it's a Merkle tree.
- A **PRF, MAC, KDF, and XOF**, as well as a regular hash.
- **One algorithm with no variants**, which is fast on x86-64 and also on smaller architectures.

#### KDF: BLAKE3

We use BLAKE3 for the vHash, KDF, PRF & MAC in Syphre because it is so efficient. Here’s some info:

BLAKE3 is based on an optimised instance of the established hash function BLAKE2 and on the original Bao tree mode. The specifications and design rationale are available in the BLAKE3 paper. The default output size is 256 bits. The current version of Bao implements verified streaming with BLAKE3.

![upscaled-removebg.png](https://res.craft.do/user/full/99e68972-962a-3ee2-6069-ddaf255ca93d/doc/59590CC0-B0A1-4A21-AF4A-F41B69C4C62A/0C0DB43E-3F8F-427A-B89A-0BEB3CE699C8_2/upscaled-removebg.png)

#### BLAKE3 is a cryptographic hash function that is:

- **Much** **faster** than MD5, SHA-1, SHA-2, SHA-3, and BLAKE2.
- **Secure**, unlike MD5 and SHA-1. And secure against length extension, unlike SHA-2.
- **Highly parallelizable** across any number of threads and SIMD lanes, because it's a Merkle tree on the inside.
- Capable of **verified streaming** and **incremental updates**, again because it's a Merkle tree.
- A **PRF, MAC, KDF, and XOF**, as well as a regular hash.
- **One algorithm with no variants**, which is fast on x86-64 and also on smaller architectures.

#### PRF: BLAKE3

We use BLAKE3 for the vHash, KDF, PRF & MAC in Syphre because it is so efficient. Here’s some info:

BLAKE3 is based on an optimised instance of the established hash function BLAKE2 and on the original Bao tree mode. The specifications and design rationale are available in the BLAKE3 paper. The default output size is 256 bits. The current version of Bao implements verified streaming with BLAKE3.

![upscaled-removebg.png](https://res.craft.do/user/full/99e68972-962a-3ee2-6069-ddaf255ca93d/doc/59590CC0-B0A1-4A21-AF4A-F41B69C4C62A/0C0DB43E-3F8F-427A-B89A-0BEB3CE699C8_2/upscaled-removebg.png)

#### BLAKE3 is a cryptographic hash function that is:

- **Much** **faster** than MD5, SHA-1, SHA-2, SHA-3, and BLAKE2.
- **Secure**, unlike MD5 and SHA-1. And secure against length extension, unlike SHA-2.
- **Highly parallelizable** across any number of threads and SIMD lanes, because it's a Merkle tree on the inside.
- Capable of **verified streaming** and **incremental updates**, again because it's a Merkle tree.
- A **PRF, MAC, KDF, and XOF**, as well as a regular hash.
- **One algorithm with no variants**, which is fast on x86-64 and also on smaller architectures.

#### MAC: BLAKE3

We use BLAKE3 for the vHash, KDF, PRF & MAC in Syphre because it is so efficient. Here’s some info:

BLAKE3 is based on an optimised instance of the established hash function BLAKE2 and on the original Bao tree mode. The specifications and design rationale are available in the BLAKE3 paper. The default output size is 256 bits. The current version of Bao implements verified streaming with BLAKE3.

![upscaled-removebg.png](https://res.craft.do/user/full/99e68972-962a-3ee2-6069-ddaf255ca93d/doc/59590CC0-B0A1-4A21-AF4A-F41B69C4C62A/0C0DB43E-3F8F-427A-B89A-0BEB3CE699C8_2/upscaled-removebg.png)

#### BLAKE3 is a cryptographic hash function that is:

- **Much** **faster** than MD5, SHA-1, SHA-2, SHA-3, and BLAKE2.
- **Secure**, unlike MD5 and SHA-1. And secure against length extension, unlike SHA-2.
- **Highly parallelizable** across any number of threads and SIMD lanes, because it's a Merkle tree on the inside.
- Capable of **verified streaming** and **incremental updates**, again because it's a Merkle tree.
- A **PRF, MAC, KDF, and XOF**, as well as a regular hash.
- **One algorithm with no variants**, which is fast on x86-64 and also on smaller architectures.

----

## For Developers

This section showcases some common use cases where cryptography is required, and the best implementations for each. If your use case isn’t on here and you want to know how you should uase Syphre in your Syvirean or Web3 project, come ask the team in the Discord for advice!

**F E A T U R E D**

#### Secure communication between two BNS identities across insecure channels over Lightlane

**Use-cases:** E2EE messaging, voice/video chat, file transfers, most other communication

If you’re building something something like this, the best idea to piggyback your app/protocol over Lightlane. You can use custom application protocols over Lightlane where Lightlane handles data security, routing and the rest. All you have to deal with is how your data is structured, and Lightlane will handle all else.

Learn [how Lightlane secures communication between BNS identities](craftdocs://open?blockId=73A70A81-6BDD-4E20-BE12-F59C484C2428&spaceId=99e68972-962a-3ee2-6069-ddaf255ca93d), or [how it privately routes this traffic over the Lanal Fabric](craftdocs://open?blockId=97BD7EAA-EE0E-41C5-B844-890F44D543C7&spaceId=99e68972-962a-3ee2-6069-ddaf255ca93d) — a peer-to-peer anonymisation network that has a built-in CDN & more.

#### Access & utilise user-owned data stored on Gaia using HSSE, without knowing the data itself

**Use-cases:** Healthcare services, machine learning as-a-service (MLaaS), or any 3rd-party service that needs access to user data on a device outside of the user’s full control.

> **WARNING:** Homomorphic Shared Storage Encryption is highly experimental technology and you should use Syphre’s implementation at your own risk.

**A L L**

#### Hide secrets effectively using sHash

**Use-cases:** Healthcare services, machine learning as-a-service (MLaaS), or any 3rd-party service that needs access to user data on a device outside of the user’s full control.

> **WARNING:** Homomorphic Shared Storage Encryption is highly experimental technology and you should use Syphre’s implementation at your own risk.

```typescript
import { STE, PRF } from 'syphre'

let data = "hello"
let key = PRF.generate(32)
let nonce = PRF.generate(8)

let encrypted = STE.encrypt(data, key, nonce)
console.log(encrypted.toString())
```

#### Calculate data hashes rapidly through vHash

**Use-cases:** Healthcare services, machine learning as-a-service (MLaaS), or any 3rd-party service that needs access to user data on a device outside of the user’s full control.

> **WARNING:** Homomorphic Shared Storage Encryption is highly experimental technology and you should use Syphre’s implementation at your own risk.

#### Digitally sign & verify data through DSA

**Use-cases:** Healthcare services, machine learning as-a-service (MLaaS), or any 3rd-party service that needs access to user data on a device outside of the user’s full control.

> **WARNING:** Homomorphic Shared Storage Encryption is highly experimental technology and you should use Syphre’s implementation at your own risk.

#### Generate cryptographically-sound pseudorandom data with PRF

**Use-cases:** Healthcare services, machine learning as-a-service (MLaaS), or any 3rd-party service that needs access to user data on a device outside of the user’s full control.

> **WARNING:** Homomorphic Shared Storage Encryption is highly experimental technology and you should use Syphre’s implementation at your own risk.

#### Create secure private keys with PRF & KDF

**Use-cases:** Healthcare services, machine learning as-a-service (MLaaS), or any 3rd-party service that needs access to user data on a device outside of the user’s full control.

> **WARNING:** Homomorphic Shared Storage Encryption is highly experimental technology and you should use Syphre’s implementation at your own risk.

#### Share a symmetric key or multiple over an insecure channel with KEM

**Use-cases:** Healthcare services, machine learning as-a-service (MLaaS), or any 3rd-party service that needs access to user data on a device outside of the user’s full control.

> **WARNING:** Homomorphic Shared Storage Encryption is highly experimental technology and you should use Syphre’s implementation at your own risk.

----

## Deep dives

Below is a quick dive into the technical details of lattice-based cryptography that Syphre builds upon. Quite technical, so if you want you can skip this one!

#### Lattice-based cryptography

*Modified from* [*Infiniti's Newsletter*](https://infinitiventures.substack.com/p/fully-homomorphic-encryption-data)

First, what are lattices? A lattice can basically be thought of as any regularly spaced grid of points stretching out to infinity i.e infinite set(discrete) of points in n-dimensional Euclidean space with a periodic structure. Although lattices, in theory, can tend towards infinity, computers are limited with finite memory. To represent different points on a lattice, a basis is used. A basis is a set of vectors that can be used to reproduce any point in the grid that forms the lattice. There are decade-old problems based on LBC that are extremely difficult to solve.

![https---bucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com-public-images-0d111fa5-62a7-441e-bf73-564349674a99_832x776.png](https://res.craft.do/user/full/99e68972-962a-3ee2-6069-ddaf255ca93d/2DCAC614-2314-478C-8464-48A2DE61F57D_2/https---bucketeer-e05bbc84-baa3-437e-9518-adb32be77984.s3.amazonaws.com-public-images-0d111fa5-62a7-441e-bf73-564349674a99_832x776.png)

A couple of popular hard problems based on lattice-based cryptography are as follows:

- The shortest vector problem (SVP): given a basis B, find the shortest non-zero vector in the lattice generated by B.
- The closest vector problem (CVP): given a basis B and a target vector t, find a lattice vector generated by B that is closest to t.

These may sound very simple to solve in 2D. But what if the lattice were 3D, 100D or 50,000D? In other words, a lattice with 50,000 coordinates, not 2. Finding the shortest or closest vector across 50,000 generated coordinates, turns out, is extremely hard that no known quantum algorithm on a quantum computer can solve it, let alone a conventional one. It is conjectured that there is no probabilistic polynomial-time algorithm that can solve the above lattice-based problems, even approximately, by breaking it down into polynomial factors.

### Learning with Error:

One of the most widely used lattice-based algorithms in cryptography is learning with errors(LWE). The security of LWE relies upon (provably reducible) solving the Shortest-Vector-Problem(SVP) in a lattice. Introduced by Oded Regev in 2005, LWE involves the difficulty of finding the values which solve B = A*s +e where A and B are known, and s is the secret message. Introducing a random error e (called noise) makes the equation hard to solve.

