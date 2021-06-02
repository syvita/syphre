# Syphre

Syphre is Syvita’s custom-built cryptographic protocol. It attempts to standardise the cryptographic algorithms used in Syvirean projects. 

It replaces TLS (Transport Layer Security) for Syvirean projects and defines the algorithms used for cryptographic functions. It is natively used in the [HYPRDRIIV protocol](https://hyprdriiv.com) to secure its transport mechanism. 

## **What does Syphre use?**

Syphre defines the following algorithms & methods for each use.

| Function | Algorithm |
| :--- | :--- |
| Volant hashing \(hashes for checksums of files, message validation etc\) | BLAKE3 |
| Slugged hashing \(hashes that need to be defended from brute-force attacks, eg secrets etc\) | Keccak256 |
| Key derivation \(KDF\) | BLAKE3 |
| Pseudorandom generator \(PRF\) | BLAKE3 |
| Message authentication codes \(MAC\) | BLAKE3 |
| Extendible output function \(XOF\) | BLAKE3 |
| Key exchange \(for generating a shared secret\) | DHKE |
| Symmetric encryption | XChaCha20 |
| Digital signatures | Ed25519 |

### Reasoning for algorithm selection

#### Volant \(fast\) hashing

We chose BLAKE3 for a number of reasons. Here are some of them. 

* **Much faster** than MD5, SHA-1, SHA-2, SHA-3.
* **Secure**, unlike MD5 and SHA-1. And secure against length extension, unlike SHA-2.
* **Highly parallelizable** across any number of threads and SIMD lanes because it's a Merkle tree on the inside, making it fast for the client and development. 
* **Capable of verified streaming and incremental updates**, which is useful for hot changes in development environments. This also gives HYPRDRIIV the unique ability to be able to stream data, such as audio & video, over its transport protocol. 
* A **PRF, MAC, KDF, and XOF, not just a regular hash**. Of which the MAC, KDF & PRF are used within the Transport component which replaces HTTPS in the Cypher component. 
* **One algorithm with no variants**, which is fast on x86-64 and future ARM-based devices. In a world full of IoT and mobile devices, being fast on all architectures is imperative. 

BLAKE3 is a relatively new algorithm, only being released in 2020. It's based on an optimized instance of the established hash function [BLAKE2](https://blake2.net/), which is in turn based upon the ChaCha stream cypher, of which Syphre's chosen symmetric encryption algorithm - XChaCha20 - is also based upon. 

The specifications and design rationale are available in the [BLAKE3 paper](https://github.com/BLAKE3-team/BLAKE3-specs/blob/master/blake3.pdf). The default output size is 256 bits. 

But how fast is it really? Take a look. 

![An example benchmark of 16 KiB inputs on modern server hardware \(a Cascade Lake-SP 8275CL processor\)](../.gitbook/assets/speed.svg)

#### Slugged \(slow\) hashing



