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

### Implementation Guidance for Provenance with resource signatures

The section outlines the implementation guidance for the various actors involved in the delivery of Provenance resources with external digital signatures for created and updated resources such as PRO QuestionnaireResponses.

Developers can follow these steps to implement a FHIR client that creates Provenance records with resource signatures.

**Step 1:** Support the FHIR API for POSTing `QuestionnaireResponse`s and `Provenance` resources.

The only two formats supported are JSON and XML bacause of their ability to be structured in a canonical form.

`POST [BaseURL]/QuestionnaireResponse/`

`POST [BaseURL]/Provenance/`


**Step 2:** Support private keys, identity certificates and digital signing



**Step 3:** Send resources with the HTTP `Prefer: return=representation` header set


**Step 4:** Implement an user or operator alert for system integrity failures

Raise an alarm if the server does not return the same resource content as was sent.

**Step 5:** Build and send a Provenance resource for properly persisted QuestionnaireResponse resources



**Step 6:** Build a mechanism for updating the status of identity certificates

Build or use a PKI infrastucture that maintains an OCSP or CRL repository. 
Make public keys available for storage by auditors for resource Provenance verification.


### Implementation Guidance for tamper-proof transaction journal


The section outlines the implementation guidance for FHIR servers to ensure that resource changes are tracked in a journal that is able to provide tamper-evidence even in the case of a resource being deleted. Journals may be mirrored to an isolated location that is additive-only and write-only, or a cryptographic journal can be used.

**Step 1:** Implement the FHIR server to maintain a transaction journal 

The server should use the journal to store FHIR REST verbs, non-PHI metadata like resource ID, and a cryptographic hash of the changed resource's state.

The transaction journal should be protected from tampering through remote transaction mirroring to a serperately secured security domain, or through cumulative cryptographic hashing. The implementation should be able to detect resource deletion and protect against replay attacks.

An example of an off-the-shelf tool that can provide this kind of assurance is Amazon Web Services' Quantum Ledger Database (QLDB).

**Step 2:** Implement a system integrity check

The server should provide operators or auditors with a mechanism that compares the system state against the journal on request. The integrity checker should report the resources and time that system integrity was compromised.







