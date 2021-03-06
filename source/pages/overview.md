---
title: 21 CFR 11 PRO Overview
layout: default
active: overview
---


<!-- TOC  the css styling for this is \pages\assets\css\project.css under 'markdown-toc'-->

* Do not remove this line (it will not be displayed)
{:toc}

<!-- end TOC -->

### CFR Title 21 Part 11 applied to PRO collection systems

[21 CFR 11](https://www.accessdata.fda.gov/scripts/cdrh/cfdocs/cfCFR/CFRSearch.cfm?CFRPart=11&showFR=1) section 11.10 lists a set of requirements related to the controls for closed systems that are addressed in this implementation guide.

There are requirements in 21 CFR 11 that can't be met with strict adherence to the FHIR PRO IG and recommended Security Profiles because of an emphasis on non-repudiation and capturing all data state changes including those of a system operator. The system operator can be an "insider threat" where duly-authorized personnel affect the integrity of collected data. The non-repudiation places an emphasis on ensuring that the contributor of PRO data is a trusted participant and that the PRO data has not changed since it was originally entered.

The below sections describe the relevant 21 CFR 11 requirements and their applicability to this extension of the PRO Implementation Guide.

#### 21 CFR 11.10 Controls for closed systems.

```
Persons who use closed systems to create, modify, maintain, or transmit electronic records shall employ procedures and controls designed to ensure the authenticity, integrity, and, when appropriate, the confidentiality of electronic records, and to ensure that the signer cannot readily repudiate the signed record as not genuine.[...]
```

This section is a summary of hte following requirements and notably mentions the signing of the electronic records and non-repudiation requirements. Also notible is that there is a stated requirement for a system to ensure the _integrity_ of electronic records.

Non-repudiation can be implemented with a managed public key infrastructure and identity verification controls. 

The integrity requirement can be met with a capability to independently verify the totality of the state of the stored records (cryptographic journaling) and the individual records themselves (client digital signing).

#### 21 CFR 11.10(a)

```
(a) Validation of systems to ensure accuracy, reliability, consistent intended performance, and the ability to discern invalid or altered records.
```

The significant part of this paragraph is the ability to discern invalid or altered records. This is a requirement that is not met with adherence to the FHIR Security profile alone. The FHIR Secirity profile requirements are necessary to meet all of the 21 CFR 11 requirements, but the threat model of that profile is mostly external actors. The 21 CFR 11 standard imagines both external and internal threats and requirements like this require a mechanism to repliably provide tamper-evidence. 

This implementation guide recommends the use of a cryptographically-secured journal with a copy of that journal data stored apart from the collected PRO data.

#### 21 CFR 11.10(e)

```
(e) Use of secure, computer-generated, time-stamped audit trails to independently record the date and time of operator entries and actions that create, modify, or delete electronic records. Record changes shall not obscure previously recorded information. Such audit trail documentation shall be retained for a period at least as long as that required for the subject electronic records and shall be available for agency review and copying.
```

The requirement for an audit trail is a conventional method for allowing auditing and forensic analysis of compuer system operation in event of a breach. The significant section is the requirement to provide an audit trail for deletions. If we sign all FHIR transactions individually, then an insider attacker can still delete the resource and the audit log entry. Cryptographic journals capture delete events and individual deletions can not be elided without complete recreation of the audit log with the participation of the resource signers.


### The PRO Implementation Guide and FHIR implementation patterns

A “pure FHIR” PRO collection mechanism adheres to the PRO Working Group’s Implementation Guide with PROs being captured using questionnaires delivered based on Questionnaire resources and the PRO data stored and exchanged using QuestionnaireResponse resources. These two resources are two of the three foundational resource types in the FHIR standard’s workflow pattern. 

{% include img.html img="pro-ig-scope.png" caption="PRO Implementation adherence to the FHIR workflow pattern " %}

21 CFR 11 compliance does not change this pattern, but it introduces some complementary Resources and 
constrains operations to provide a secure, auditable store with non-repudiable and tamper-evident transactions.


###  Background and Threat Model

When designing a system to meet the requirements of 21 CFR Part 11 more requirements than are described in FHIR Security and FHIR PRO Implementation Guides are required. FHIR standards generally assume that all attackers are outside the FHIR system’s perimeter trying to penetrate the defences and exfiltrate that data. 

System perimeter security and general security hygiene is described, but no method of ensuring that an internal attacker’s actions can be detected or curtailed.
The two main architectural approaches proposed in U24 are:

  1.  Digital signing of Provenance-sensitive resources by FHIR clients. While FHIR doesn’t have a fully transactional method for achieving this the FHIR system provides a mechanism to ensure that the provenance of data can be digitally verified when a FHIR client implements an additional operation to verify, sign and provide a Provenance resource for critical data.

  2.  Write-only, cryptographically-signed journaling. A 21 CFR Part 11-compliant FHIR server can implement a write-only, cumulatively signed journal of system changes to allow system tampering and corruption to be detected. The combination of resource provenance for patient-provided resources and the system journal satisfy 21 CFR Part 11 requirements for auditability, protection of records and secure, computer-generated audit records for all create, modify and delete activities for electronic records.



{% include img.html img="signing-and-journaling-flow.png" caption="Cryptographic provenance and transaction journaling flow overview and context" %}


The figure above shows a FHIR client which could be embedded in an app or a gateway to an Electronic Data Capture system like Medidata RAVE or EPIC. The client sends PRO and any other data as FHIR resources to a FHIR server which stores the resource and provides the “fixed up” FHIR resource back to the client as per the FHIR standard. The points below explain the sequence:


1. *Send Resource* This is a POST, PUT or DELETE with a HTTP `Prefer: return=representation` header set as per the [FHIR HTTP standards](https://www.hl7.org/fhir/http.html#ops). This ensures that the HTTP response contains the a full copy of the resource after any FHIR server mutations. The client can use the returned resource to compare with the expected values and sign that to provide a Provenance resource. A FHIR client can't sign the original resource because the server will change the ID, version and some other metadata before persisting the resource, rendering any signature invalid. The client has to verify the server's copy of the resource and sign that for a signature to be valid and able to be used to detect changes from the time the provenance was asserted. 
2. *Server stores resource & assigns ID* This is the FHIR server making the client's request change (adding, changing or deleting a resource) and fixing up any versions or unique IDs according to the server's internal state and implentation. 
3. *Server stores read-only action and Resource in journal* The server sends a copy of the persisted resource to a cryptographic journal.
4. *Response with server's fixed-up Resource* The HTTP response from the server returns a full copy of the resource after the sever has made any server-required changes to the Resource to add versions, IDs or any other metadata.
5. *Client verifies and generates signature from a [canonical JSON version](https://gibson042.github.io/canonicaljson-spec/) of the server response* The client receives the Resource from the HTTP response and compares a canonical version of the response with the value sent in the original FHIR transaction. To do this the client would need to "canonicalise" the JSON or XML of the original and returned resources and compare to see if these are equivalent. The client should not consider the server-assigned ID, versions or other metadata when it compares the original or returned transactions. 
  * If there is a difference between the sent and received resource then the client should not proceed with signing the returned resource, and should alert the client or operator that the server has offered a different resource to sign than the one expected.
  * If the returned resource is the same, then the client shall generate a signature for the entire returned resource including the ID, versions and metadata.  
The client then generates a Provenance resource containing the identity of the client, the thumbprint of the client's current identity certificate, date and timestamp information, and a reference to the Resource and resource representation that the Provenance resource refers to.
6. *Send Provenance resource with resource signature* As per 1 above, the client POSTs a copy of the newly created Provenance resource.
7. *Server saves Provenance resource & resource signature* As per 2 above, the Provenance resource is persisted in the FHIR server like any other POSTed resource.
8. *Store read-only action and Provenance in journal* Per 3 above, the persistence of the Provenance resource is recorded in the cryptographic journal.
9. *Server's fixed up copy of the Provenance resource* As per 4 above, the FHIR server returns a copy of the Provenance resource to the client. The FHIR client should check that the immutable content of the resource is identical to that which was sent and raise an error with the client or operator if the returned resource differs from the resource that was sent to the server.


#### Client Interactions

In a 21 CFR Part 11-compliant architecture the client would have responsibility for verifing the canonical portions of the “fixed up” resource that is returned from the server is identical to the provided resource information. If the canonical server resource information matches the client’s original, then the client creates a signature of the resource using the canonical version of the resource and the client’s identity certificate. The signature for the (canonical) resource containing PRO data is then enclosed in a Provenance resource and send to the server to be stored.

The level of provenance provided will depend on where the FHIR client sits in the system architecture. If the architecture is an app that is in the hands of a user, then the architecture supports signing that integrates with the authentication recommendations of the FHIR PRO Implementation Guide and FHIR resources created by a user’s app can be signed with a certificate that identifies that user.

For architectures that do not have a FHIR client that is in the control of a user, the “gateway” client that translates data into FHIR will have to make a provenance assertion of its own which guarantees provenance at the point that the gateway client has access to the data, but does not represent a provenance assertion from the original source of the data, just that point that the gateway has custody. 

When a FHIR client is a gateway for an external system it may be appropriate for the gateway client to provide a special resource, or additional metadata to describe the “upstream” source of the data and the kind of provenance assertions it can make, or a document describing how to audit the upstream provenance of the data.

#### Server Implementation Requirements

The FHIR server records all resource create, update and delete operations in a cryptographic journal where past transactions are immutable. This could be a blockchain-like cryptographically-assured data structure, but there is no mandatory requirement for a full blockchain implementation with distributed transactions and proof of work – just that the journal provide a immutability for past transaction records and a way to detect tampering in the journal or when the journal is correlated with the system state. For additional security the journal may be replicated to an external network.

### System Types

#### Conventional PRO Collection

The following diagram illustrates the conventional (traditional) approach to implementing a PRO Electronic Data Collection system and data warehouse with 21 CFR 11 compliance.

the study participants (1) submit data to an EDC/warehouse (2). The identity of each study participant is assuree with an enrolment (3) and identity management procedure (6). Authorized personnel store the participant’s contact information (4). All data changes are captured in an audit trail (5). 
Determining adherence to 21 CFR Part 11 (7) requires the Agency’s inspector to review governance policies and procedures (6) as well as audit information and personnel training and management necessary to deter data falsification (5).

This approach places trust in personnel and systems within the study. 21 CFR Part 11.10(j) prioritizes deterrence of record and identity falsification over protection from these classes of insider attack. Data entered by participants is stored in the study warehouse, but there is no means to determine that the data stored has not been tampered with, or otherwise corrupted, since it was provided by the participant as required by 21 CFR Part 11.10(a).


{% include img.html img="FHIR-Workflow-and-PRO-Resources-traditional-PRO-system-2.png" caption="Conventional non-FHIR PRO collection" %}

#### 21 CFR 11 PRO FHIR Warehousing

The figure below shows a PRO capture system that meets 21 CFR 11 requirements discussed above. 

Study participants (1) use their identity certificate obtained after proving their identity to a trusted certificate authority during study enrolment (3).  Study data is stored with a signature that can be used to attest to that the data has not been altered and that it originated from a valid study participant.

Because even signed data can be deleted, the audit trail is stored in a tamper-evident ledger (5) which can be used to identify if the ledger has been tampered with, or the study data.

{% include img.html img="FHIR-Workflow-and-PRO-Resources-FHIR_Provenance-PRO-system-2.png" caption="FHIR Workflow with an entirely FHIR system implementation" %}

### System Boundaries and Key Management

PRO EDC systems and warehouses come in both FHIR-based and hybrid FHIR/non-FHIR systems. Provenance and auditing requirements applicable to 21 CFR 11 compliance in this implementation guide are applicable to the edge of the FHIR subsystem within the entire system PRO pipeline, but the text below and the guidance in this implementation guide provide practical steps to create a 21 CFR 11 compliant system from both a purely FHIR-based architecture and a typical hybrid system.

The two main additional requirements for 21 CFR 11 compliance require digital signing, a key and certificate management system is required to ensure that PROs entering the FHIR subsystems are signed with certified keys and certificates are available for any systems or personnel who need to verify signatures.

FHIR clients participating in a 21 CFR 11 compliant system must make use of a private key and make a properly certified identity certificate available to system actors that will need to verify the provenance of client-submitetd data. 

FHIR servers, or the cryptographic journaling technology they use will also require a private key and identity certificate to maintain a cryptographic journal. 

Both clients and servers will have to participate in a Managed Public Key Infrastructure to ensure that their certificates and certificate status are:

  1. Revocable,
  2. Signed by an acceptably reputable certifying authority

Two design examples are discussed below.

#### 21 CFR 11 PRO FHIR Warehousing with a non-FHIR EDC

When PRO data is be collected into a non-FHIR EDC before being imported to a warehouse, 21 CFR 11 compliance must be assessed for both the entire system and the FHIR-based subsystems. The FHIR system boundary shown here is an external EDC like Medidata RAVE or EPIC that provides a mechanism for obtaining a journal of transactions that can be transformed into QuestionnareResponse resources and ingested into the PRO-storing FHIR Server.

{% include img.html img="FHIR-Workflow-and-PRO-Resources-Gateway-ed-FHIR.png" caption="External PRO collection to FHIR warehouse with gateway identity certificate" %}

The FHIR resources in this type of system are signed at the system boundary by the gateway software that is normally implemented as "back to back" EDC and FHIR clients. This gateway authenticates with the external EDC and transforms the PRO data into QuestionnaireResponses before HTTP POSTing them to the FHIR Server.

In this architecture transactions from the EDC have to be imported to FHIR in a repeatable order to allow a full audit of transactions and recreation of warehouse state for comparison.

Use of a gateway model will result in Provenance being asserted by only the number of actor identities as there are gateways to the FHIR server. The gateway can add the original EDC's view of the original PRO subject's identity, and the role it plays. 

#### 21 CFR 11 PRO FHIR EDC and Warehousing 

The diagram below shows a simplified context diagram for a PRO-collecting app that is a FHIR client that communicates securely with a FHIR server. In this entirely-FHIR system each patient who reports PRO data can have their own secret key and identity certificate. 

The subject's secret key need not even be known to the Subject if the process of enrolment and generation of secret keys and certified identity certificates is hidden from the user and protected by the app or the app's enrolment ecosystem.

{% include img.html img="FHIR-Workflow-and-PRO-Resources-Simple-end-to-end-fhir.png" caption="FHIR PRO system with patient identity certificates" %}

In a fully-FHIR based system each client provides Provenance for each of the QuestionnaireResponse resources they generate and send, and does not act as a gateway or proxy. The FHIR server will store and journal QuestionnaireResponses with data provided, and Provenance assured by, the originator of the PRO response.

### Public Key Infrastructure and Identity Verification Controls

Public Key Infrastructure (PKI) is required to provide a standards-based, resilient and manageable infrastructure for signing PRO data. This page discusses the use and limitations of PKI for assuring provenance and providing tamper-evidence for PROs.

In a perfect 21 CFR 11 implementation auditable provenance of PROs requires that the PRO data is signed off by the patient who generated the data. In systems where it is not possible for a strong assertion that the data is unchanged since it was entered by a patient, other manual or computational processes and non-cryptographic record-keeping are required to prove provenance. For the U24 project we propose a hybrid approach when a FHIR server is used as the storage mechanism.

  1. To establish the identity of a patient who is providing a Patient Reported Outcome a good practice is to have that patient/subject create a private key and to use that private key to generate an identity certificate which is shared with any other person or system who wants to be able to decrypt or verify information created by that subject.
  1. In cases where the software components of a system do not support a subject using their own certificate, the FHIR server portion can implement a "signing proxy" that takes PRO information from a system that does not use PKI, and sign those resources at the edge of the ecosystem boundary to add an assertion about the provenance of FHIR resources at the time they enter the FHIR ecosystem.


#### Establishing a key pair for the client

How a FHIR client assures its identity and creates a private key and CA-signed certificate is important to describe in a 21 CFR Part 11 PRO implementation guide.

To sign its provided resources, each FHIR client will require a private key and an ID certificate.

The ID certificate should be:

  * Only issued to verified patients
  * Signed by a (reputable) CA
  * Uniquely identified
  * Available to auditors when signatures need to be verified
  * Able to be revoked and revocation status made available to auditors
  * Able to expire and be reissued
  * Provide Detached Signatures for Resources in Provenance Resources

The ID certificate should also be made available to any system or person who needs to verify the authenticity of signed resources. The recommendation of this implementation Guide is for each client 
to POST any new identity certificate to the server embedded in a DocumentReference resource and provide a 
Provenance resource for that DocumentRefence with a reference to the client’s associated Patient resource as a context/sourcePatientInfo reference or Device resource context/related reference (depending on whether the client is a patient client or a gateway application for an EDC). The client adds a signature for the DocumentRefence in the accompanying Provenance resource.

#### Automated Client Identity Certificate 

To provide a flexible and standards-based mechanism for signing identity certificates a simple OAuth2-protected certificate-signing REST service (see [Guidance](guidance.html)), or an adaptation of the [ACME](https://tools.ietf.org/html/rfc8555) protocol that integrates with the FHIR security profile is recommended.

[ACME](https://tools.ietf.org/html/rfc8555) is a protocol designed largely for automating the signing of certificates for use to secure hosts in Internet domains and individual web hosts. There is no direct means in the standard to automate the signing request and proof provision required to issue an identity certificate, but the intent to be a general and extensible protocol is mentioned:

    Note that while ACME is defined with enough flexibility to handle different types of identifiers in principle, the primary use case addressed by this document is the case where domain names are used as identifiers.

The ACME API is a useful conceptual framework for a 21 CFR 11 system where FHIR clients need to perform certificate orders, but it has a couple of assumptions that can cause complexity:

  1. ACME assumes it will operate as a standalone server with its own registration and authentication mechanism that is OAuth-like in that each client request must provide a nonce. This parallels and duplicates functionality an OIDC or OAuth 2.0 implementation could provide.
  2.  The standard flow for ordering a certificate includes a requirement for a client to poll until a set of validation criteria is met, but these indeterminate duration tests can be omitted from the protocol for identity.

As a 21 CFR 11-compliant FHIR server adhering to the PRO Implementation Guide already has an OIDC
or OAuth 2.0 authentication and authorization interface, the account and authorization processes 
should be ignored and the OIDC or OAuth 2.0 equivalents used. 

An authorized client can order a signature for its self-created identity certificate and use the authorization token and claim and the challenge/proof 
step is unnecessary as the client ID is already known because of the authorization process. In this 
regard no challenge need be issued, or the challenge portion of the protocol can be ignored) and the 
order object can transition directly to the valid state. The ACME REST methods newOrder, revokeCert 
and keyChange are all available to an authorized client.

{% include img.html img="signing-flow.png" caption=" Sequence for authenticating, signing new identity certificates and storing them to the FHIR server" %}

When a server issues a certificate, the client stores a copy of that certificate with the FHIR server by POSTing it embedded in a DocumentReference  with a reference to the client’s associated Patient resource as a context/sourcePatientInfo reference or Device resource context/related reference (depending on whether the client is a Patient client or a gateway application for an EDC) and an accompanying signature in a Provenance resource.

The ACME server can either sign identity certificates as a system-wide Certificate Authority, or as a managed Certificate Authority delegated from a higher-level CA with a chain of trust that meets the approval of a regulatory body.

### Provenance and Resource Signing

Signed resources provide a mechanism for an auditor or audit process to validate that a resource was genuinely provided by a FHIR client with a known identity and has not been altered since it was signed.

A Provenance resource contains a pointer to the FHIR system's concept of who made a change to a resource, what version of the FHIR server's resource was changed, when it was changed, and optionally a signature to verify the resource. (Traditionally the signature is a hash of a DICOM file or PDF document resource)
For 21 CFR Part 11 compliance, a Provenance resource should provide:

  * A link to the server instance of the resource the signature is for
  * A signature
  * The identity or fingerprint of the currently-issued ID certificate that corresponds to the secret key used to sign the resource.

When a signature is provided it is important to note a known corresponding ID certificate. Because ID certificates can be reissued, and a single FHIR client could have multiple valid identity certificates published at once.

```
{
  "resourceType": "Provenance",
  "id": "42",
  "text": {
    "status": "generated",
    "div": "<div xmlns=\"http://www.w3.org/1999/xhtml\">Example provenance resource 42.</div>"
  },
  "target": [
    {
      "reference": "QuestionnaireResponse/42/_history/1"
    }
  ],
  "occurredPeriod": {
    "start": "2019-10-25",
    "end": "2019-10-25"
  },
  "recorded": "2015-06-27T08:39:24+10:00",
  "policy": [
    "http://acme.com/fhir/Consent/25"
  ],
  "location": {
    "reference": "Location/1"
  },
  "reason": [
    {
      "coding": [
        {
          "system": "http://terminology.hl7.org/CodeSystem/v3-ActReason",
          "code": "HCOMPL",
          "display": "Compliance"
        }
      ]
    }
  ],
  "agent": [
    {
      "type": {
        "coding": [
          {
            "system": "http://terminology.hl7.org/CodeSystem/v3-ParticipationType",
            "code": "AUT"
          }
        ]
      },
      "who": {
        "identifier": {
          "system": "urn:ietf:rfc:3986",
          "value": "mailto:odm-fhir-gw@chip.org"
        }
      }
    },
    {
      "id": "cert-04121321-4af5-424c-a0e1-ed3aab1c349d",
      "type": {
        "coding": [
          {
            "system": "http://terminology.hl7.org/CodeSystem/v3-ParticipationType",
            "code": "DEV"
          }
        ]
      },
      "who": {
        "reference": "Device/04121321-4af5-424c-a0e1-ed3aab1c349d"
      }
    }
  ],
  "signature": {
    "type": [
        {
          "system": "urn:iso-astm:E1762-95:2013",
          "code": "1.2.840.113549.1.1.1",
          "display": "Verification Signature"
        }
      ],
    "when": "2019-10-25T01:39:24+10:00",
    "who": {
        "reference": "Patient/1"
      },
    "targetFormat": "application/fhir+json",
    "sigFormat": "application/signature+json",
    "data": "9139be98f16cf53d22da63cb559bb06a93338da6a344e28a4285c2da33facb7080d26e7a09483779a016eebc207602fc3f90492c2f2fb8143f0fe30fd855593d"
  }
}
```

### Identity Certificate DocumentReference

```
{
  "resourceType": "DocumentReference",
  "subject": {
    "reference": "Patient/4952"
  },
  "description": "Certificate for Patient/4952/_history/1",
  "content": [
    {
      "attachment": {
        "data": "MIIC1KADAgECAgEBMA0GCSqGSIb3DQEBBQUAMIGsMR8wHQYJKoZIhvcNAQkBFhBjYS10ZXN0QGNoaXAub3JnMRkwFwYDVQQDDBBjYS10ZXN0QGNoaXAub3JnMQ8wDQYDVQQLDAZJSEwgQ0ExEDAOBgNVBAoMB0NISVAgQ0ExEjAQBgNVBAcMCUJyb29rbGluZTEqMCgGA1UECAwhVGhlIENvbW1vbndlYWx0aCBvZiBNYXNzYWNodXNldHRzMQswCQYDVQQGEwJVUzAeFw0yMDA2MDMwMzM4NTdaFw0zMDA2MDEwMzM4NTdaMIHFMQswCQYDVQQGEwJVUzEmMCQGA1UECAwdQ29tbW9ud2VhbHRoIG9mIE1hc3NhY2h1c2V0dHMxDzANBgNVBAcMBkJvc3RvbjEmMCQGA1UECgwdSW50ZWxsaWdlbnQgSGVhbHRoIExhYm9yYXRvcnkxDTALBgNVBAsMBENISVAxDTALBgNVBAMMBHRlc3QxNzA1BgkqhkiG9w0BCQEWKGNocmlzdG9waGVyLmdlbnRsZUBjaGlsZHJlbnMuaGFydmFyZC5lZHUwggEiMA0GCSqGSIb3DQEBAQUAA4IBDwAwggEKAoIBAQDgQwELuDuUD7SNfHgPJXVgjX1v2P1XCFK7MhRiUZeUZ7qAxTo3R5MckZgwE6d0Y5ToAAnIMHmtYM8AGFierV9JHPgjtGGVWpzeJx9m3Eys9o1S0NOqyP19LQCmhtjKGVSm9vgfIjWLbDtbDrZuxoF82nHzg6tqSyhceSFbuBa/gzOlPTZ3Ly3b8KpMijDPzarcvBUKZm4vyqXMmF+BS/HiAJ6EsUby+mdQTG84UYsULa970rXfaZHuBqRo99d9E2AfX2zKvOlMlSuSjbzmKdm4mJmkicnYRubZBnLYdytLBEePebNdRv+VpZPjKr0bnBq8jUNbBaAF7CrQiy2sF1BRAgMBAAE="
      }
    }
  ]
}```

### Cryptographic Journaling and Auditing

To audit every resource of interest the following process needs to be supported:

  1. The resource is retrieved
  2. The corresponding Provenance entry is retrieved
  3. The certificate referenced in the Provenance entry is retrieved
  4. The certificate (from 3) is used to validate that the signature (retreived in 2) corresponds to the canonical version of the resource (retrieved in 1)


<!-- {% raw %}>{% include link-list.md %} {% endraw %}-->

{% include link-list.md %}
