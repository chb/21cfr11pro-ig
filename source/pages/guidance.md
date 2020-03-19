---
title: Implementation Guidance
layout: default
active: guidance
f: http://build.fhir.org/
---

{:.no_toc}

<!-- TOC  the css styling for this is \pages\assets\css\project.css under 'markdown-toc'-->

* Do not remove this line (it will not be displayed)
{:toc}


### Introduction

The implementation guidance outlined in this page provides useful information for developers implementing the various PRO actors and their capabilities for 21 CFR 11 compliance. All resources and parameters used in this part of the Implementation Guide are examples and are expected to conform to the resources, profiles and other constraints outlined in this implementation guide's [CapabilityStatements](capstatements.html) and that of the PRO Implementation Guide.

This implementation guide is additive to the PRO IG and assumes the guidance of the PRO IG has been followed before reading this IG.


### Build a mechanism for managing client keys and certificates

**Step 1:** Build a mechanism for creating and securing a client secret key 

Clients require both a secret key and an appropriately certified identity certificate that can be used to verify trust in the signer of Provenance data. 

In an app, or multi-client environment the best practical mechanism for generating a private key is the clientself-generating a private key and keeping that secure. For a gateway-based system with few clients it can be practical for a system operator or administrator to generate and secure the private key. 

**Step 2:** Implement a mechanism for obtaining a certified identity certificate from a certificate authority

An implementor can decide on the most appropriate method of obtaining an identity certificate signed by a certifying authority trusted by the community of users and auditors to reliably establish identity and manage certificate status. This could either be a CA run as a managed PKI solution by the organisation implementing the PRO collection system, or a large public certificate authority with a good enough reputation for FDA and other auditors.

**Step 3:** Build a mechanism for updating the status of an identity certificate

Build or use a PKI infrastucture that maintains an OCSP or CRL repository. If a client accidentally discloses a private key, or if a client proves to have been impersonated, then the certificate should be invalidated. Expired certificates are still valid for the period in which resources were signed, so any revocation or status database should distinguish between expired and invalidated certificates.

**Step 4:** Share identity certificates with auditing tools and personnel

Make public identity keys available for storage by auditors for resource Provenance verification.


### Implementation Guidance for Provenance with resource signatures

The section outlines the implementation guidance for the various actors involved in the delivery of Provenance resources with external digital signatures for created and updated resources such as PRO QuestionnaireResponses.

Developers can follow these steps to implement a FHIR client that creates Provenance records with resource signatures.

**Step 1:** Support the FHIR API for POSTing `DocumentReference`, `QuestionnaireResponse` and `Provenance` resources.

The only formats is JSON to support signing of the canonical form. XML guidance may be forthcoming.

`POST [BaseURL]/QuestionnaireResponse/`

`POST [BaseURL]/Provenance/`

`POST [BaseURL]/DocumentReference/`

**Step 2:** Support private keys, identity certificates and digital signing

The FHIR client should generate or have access to a key pair. The secret key is required for signing, and the corresponding identity certificate is certified by a reputable certifying authority

**Step 3:** Send resources with the HTTP `Prefer: return=representation` header set

The `Prefer` header requests a full copy of the server-updated resource in the HTTP response. This is essential for creating an externalised signature for inclusion in a `Provenance` resource. The canonical version of the returned with the server metadata excluded can be used for ensuring the server's version of the resource is the same as the client's version, and an external signature generated for the resource that can be included in the `Provenance`.

**Step 4:** Implement an user or operator alert for system integrity failures

Raise an alarm to the user or operator if the server does not return the same resource content as was sent.

**Step 5:** Build and send a Provenance resource for properly persisted QuestionnaireResponse resources

This step contains a series of sub-steps that cover the generation of data for inclusion in the client-generated Provenance resource.

  * Convert the QuestionnaireResponse JSON to [canonical form](http://wiki.laptop.org/go/Canonical_JSON)
  * Use the client's secret key and the canoncal JSON to generate a signature using SHA256 with RSA key
  * Create a Provenance resource and populate it with:
    * An external resource signature with coding data type:
      * System: "urn:iso-astm:E1762-95:2013"
      * Code OID: "1.2.840.10065.1.12.1.14"
      * Display: "SHA-256 Source Signature"
      * Signature: the base64 encoded version of the generated signature from above
    * An agent key with:
      * Agent Coding: 
        * System: http://dicom.nema.org/resources/ontology/DCM
        * Code: 110150 (Application Audit participant role ID of software application)
      * Agent Thumbprint Coding
        * System: "urn:pki:thumbprint"
        * Thumbprint: The client's current identity certificate thumbprint
    * A resource target referencing the server resource instance

Then `POST [BaseURL]/Provenance/` this Provenance resource to the server.

**Step 6:** Raise an alarm to the user or operator if the server does not return the same _Provenance_ resource content as was sent.

### Implementation Guidance for tamper-proof transaction journal

The section outlines the implementation guidance for FHIR servers to ensure that resource changes are tracked in a journal that is able to provide tamper-evidence even in the case of a resource being deleted. Journals may be mirrored to an isolated location that is additive-only and write-only, or a cryptographic journal can be used.

**Step 1:** Implement the FHIR server to maintain a transaction journal 

The server should use the journal to store FHIR REST verbs, non-PHI metadata like resource ID, and a cryptographic hash of the changed resource's state.

The transaction journal should be protected from tampering through remote transaction mirroring to a serperately secured security domain, or through cumulative cryptographic hashing. The implementation should be able to detect resource deletion and protect against replay attacks.

An example of an off-the-shelf tool that can provide this kind of assurance is Amazon Web Services' Quantum Ledger Database (QLDB).

**Step 2:** Implement a system integrity check

The server should provide operators or auditors with a mechanism that compares the system state against the journal on request. The integrity checker should report the resources and time that system integrity was compromised.

### Automating Identity Certificate Management

This section describes the use of a subset of [ACME protocol](https://tools.ietf.org/html/rfc8555)
operations to implement a server API for creating Client Identity Certificates that can be used to 
provide digitally-signed provenance for client-contributed PRO data and related resources. Automated creation of identity certificates provides an automatic persistent method of assessing provenance 
based on digital cryptography and leveraging the security of the study enrolment process that provides 
strongly-identified patients with access to provide PROs.

The ACME server can either operate as a local CA, or as a delegate from a Certifying Authority managed PKI

**Precondition:** Implement an OAuth 2.0 Authentication/Authorization Scheme per [FHIR Security Specification](http://hl7.org/fhir/security.html#http)

**Step 1:** Implement the newOrder, revokeCert and keyChange ACME services

Add support for these services on the FHIR server. See the [RFC](https://tools.ietf.org/html/rfc8555) for protocol details.

_Do not_ require a challenge to prove identity before accepting a signing request if the client has a valid OAuth 2.0 or OpenID Connect session that demonstrates system authentication and authorization.

**Step 2:** Restrict certificate order/signing requests to the client's identity

Restrict the client to sign only a client's own identity certificate signing request. This would restrict certificates to client identity passed from OAuth 2.0 or OIDC.

**Step 3:** Build a FHIR client that orders identity certificates as required

The FHIR client should order new certificates when:

  1. The FHIR client (or device) does not have an existing identity certificate
  2. The client's existing identity certificate has expired
  3. The client's identity certificate status is revoked or the client's secret key is changed

**Step 4:** Restrict access to the server's revokeCert  method to the owning client and administrators

Clients should permitted only to revoke their own certificates. Administrators can revoke any certificates.

**Step 5:** Journal all certificate operations

It is particularly important to record a timestamp of when certificates were revoked for system audit.

**Step 6:** Implement a client that POSTs new identity certificates to the server.

`POST [BaseURL]/DocumentReference/`

Certificates are already signed by the server, so generating a `Provenance` resource to accompany the `DocumentReference` resource is optional, but provides a consistent auditing approach for all client-POSTed data.  



