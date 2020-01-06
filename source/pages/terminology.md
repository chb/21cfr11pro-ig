---
title: terminology used in this Implementation Guide
layout: default
active: terminology
---

### Public Key Cryptography

(Also asymmetric cryptography)

    [...] is a cryptographic system that uses pairs of keys: public keys which may be disseminated widely, and private keys which are known only to the owner. The generation of such keys depends on cryptographic algorithms based on mathematical problems to produce one-way functions. Effective security only requires keeping the private key private; the public key can be openly distributed without compromising security.

  [-- wikipedia](https://en.wikipedia.org/wiki/Public-key_cryptography)

#### Identity Certificate

This is a certificate containing at least a public key generated from a subject's private key. ID certificate generation is initiated when a client uses their private key to generate a signing request. The signing request is then sent to a Certificate Authority (CA) who verifies the identity of the client and the veracity of provided attributes before signing the certificate. An ID certificate signed this way provides anyone with the ID certificate with a means of:

  * Verifying that the certificate was signed by the Certificate Authority as containing accurate identifying information
  * Verifying that information was signed by the holder of the certificate's corresponding private key
Without exposing the client's private key (which would allow impersonation).

#### Private Key

This is a secret known only to a client. This is used by the client to sign and encrypt data. If this is disclosed then it should be discarded and any corresponding public certificates should be revoked or marked invalid.

#### Key Pair

This is a private key and the corresponding ID certificate. This can be a misnomer when multiple certificates are issued for a key, but the term is used for both the private and public (distributable) material required for Public Key Cryptography.

#### Fingerprint 

(Also Thumbprint) This is a unique identifier for a certificate (Usually an MD5 hash of the certificate content). Fingerprints are used by certificate revocation (CRL) and status verification (OCSP) services as a unique identifier for certificates for efficient lookup.

### Non-Repudiation

Non-repudiation is a characteristic of PKI systems where content that is digitally signed using a private key can be matched to a public certificate to give a high confidence that the resource was signed by an individual or system with sole possession of the private key. This characteristic usually relies on a mechanism or contractual undertaking by the signer to keep their private key safe and only access it for purposes of proving provenance or assent.


