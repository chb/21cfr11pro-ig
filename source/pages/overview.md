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

#### 11.10 Controls for closed systems.

```
Persons who use closed systems to create, modify, maintain, or transmit electronic records shall employ procedures and controls designed to ensure the authenticity, integrity, and, when appropriate, the confidentiality of electronic records, and to ensure that the signer cannot readily repudiate the signed record as not genuine.[...]
```

This section is a summary of hte following requirements and notably mentions the signing of the electronic records and non-repudiation requirements. Also notible is that there is a stated requirement for a system to ensure the _integrity_ of electronic records.


Non-repudiation can be implemented with a managed public key infrastructure and identity verification controls. 
 
The integrity requirement can be met with a capability to independently verify the totality of the state of the stored records (cryptographic journaling) and the individual records themselves (client digital signing).





#### 11.10(a): 


### The PRO Implementation Guide and FHIR implementation patterns

A “pure FHIR” PRO collection mechanism adheres to the PRO Working Group’s Implementation Guide with PROs being captured using questionnaires delivered based on Questionnaire resources and the PRO data stored and exchanged using QuestionnaireResponse resources. These two resources are two of the three foundational resource types in the FHIR standard’s workflow pattern. 

{% include img.html img="pro-ig-scope.png" caption="PRO Implementation adherence to the FHIR workflow pattern " %}

21 CFR 11 compliance does not change this pattern, but it introduces some complementary Resources and 
constrains operations to provide a secure, auditable store with non-repudiable and tamper-evident transactions.

The main 

###  Background and Threat Model

When designing a system to meet the requirements of 21 CFR Part 11 more requirements than are described in FHIR Security and FHIR PRO Implementation Guides are required. FHIR standards generally assume that all attackers are outside the FHIR system’s perimeter trying to penetrate the defences and exfiltrate that data. 

System perimeter security and general security hygiene is described, but no method of ensuring that an internal attacker’s actions can be detected or curtailed.
The two main architectural approaches proposed in U24 are:

  1.  Digital signing of Provenance-sensitive resources by FHIR clients. While FHIR doesn’t have a fully transactional method for achieving this the FHIR system provides a mechanism to ensure that the provenance of data can be digitally verified when a FHIR client implements an additional operation to verify, sign and provide a Provenance resource for critical data.

  2.  Write-only, cryptographically-signed journaling. A 21 CFR Part 11-compliant FHIR server can implement a write-only, cumulatively signed journal of system changes to allow system tampering and corruption to be detected. The combination of resource provenance for patient-provided resources and the system journal satisfy 21 CFR Part 11 requirements for auditability, protection of records and secure, computer-generated audit records for all create, modify and delete activities for electronic records.

A flow diagram for a general implementation is provided below:

{% include img.html img="signing-and-journaling-flow.png" caption="Cryptographic provenance and transaction journaling " %}


The figure above shows a FHIR client which could be embedded in an app or a gateway to an Electronic Data Capture system like Medidata RAVE or EPIC. The client sends PRO and any other data as FHIR resources to a FHIR server which stores the resource and provides the “fixed up” FHIR resource back to the client according to the FHIR standard. 

In a 21 CFR Part 11-compliant architecture the client would verify the canonical portions of the “fixed up” resource that is returned from the server is identical to the provided resource information. If the canonical server resource information matches the client’s original, then the client creates a signature of the resource using the canonical version of the resource and the client’s identity certificate. 
The signature for the resource containing PRO data is enclosed in a Provenance resource and send to the server to be stored.

The level of provenance provided will depend on where the FHIR client sits in the system architecture. If the architecture is an app that is in the hands of a user, then the architecture supports signing that integrates with the authentication recommendations of the FHIR PRO Implementation Guide and FHIR resources created by a user’s app can be signed with a certificate that identifies that user.
For architectures that do not have a FHIR client that is in the control of a user, the “gateway” client that translates data into FHIR will have to make a provenance assertion of its own which guarantees provenance at the point that the gateway client has access to the data, but does not represent a provenance assertion from the original source of the data, just that point that the gateway has custody. 
When a FHIR client is a gateway for an external system it may be appropriate for the gateway client to provide a special resource, or additional metadata to describe the “upstream” source of the data and the kind of provenance assertions it can make, or a document describing how to audit the upstream provenance of the data.

The FHIR server records all resource create, update and delete operations in a cryptographic journal where past transactions are immutable. This could be a blockchain-like cryptographically-assured data structure, but there is no mandatory requirement for a full blockchain implementation with distributed transactions and proof of work – just that the journal provide a immutability for past transaction records and a way to detect tampering in the journal or when the journal is correlated with the system state. For additional security the journal may be replicated to an external network.

Amazon offers a service suitable for a persistence and journaling layer with immutable history designed for document that is called the Quantum Ledger Database (QLDB). A FHIR server implemented using QLDB to store FHIR resources as documents would generate an immutable journal with controlled and audited access meeting the auditability and tamper-evidence requirements of 21 CFR Part 11. Alternative implementation technologies like HyperLedger Fabric are more complex to set up, and can be expensive to operate privately.
QLDB is expected to be available as a HIPAA-eligible service, but has not been added to the HIPAA-eligible service catalog yet. 

#### RAVE  

The CARRA Medidata RAVE ODM to FHIR gateway is a FHIR client that produces QuestionnaireResponses, Patient, Organization and Encounter resources. The ODM-to-FHIR gateway adheres to the PRO Implementation Guide in that it generates QuestionnaireResponse resource, but Medidata RAVE takes care of the definition of questionnaires internally and this is not made available for translation to an external FHIR resource.
To adhere to the recommendations of this architecture the minimum requirement for the ODM-to-FHIR gateway would be to provide a Provenance resource with a signature for each of the added, removed and changed server resources.

#### EPIC 

Similarly to Medidata RAVE, EPIC has an internal proprietary Questionnaire resource equivalent. EPIC also makes use of LOINC to automatically define question structure, and the LOINC coding is available in the FHIR resources available from the EPIC FHIR Server API.

Unlike RAVE, EPIC has a simple FHIR server API that can be queries by an external FHIR client application to retrieve PRO data. The data is provided as FHIR DSTU3 Observation resources, which, while not what the PRO Implementation Guide accommodates, could be translated into a QuestionnaireResponse resource similarly to the RAVE approach. 

There are practical problems with using this interface as a FHIR mirror of data stored in EPIC, as the server interface does not provide access to an entire journal of state changes, just current state for the queried resources. For a 21 CFR Part 11-compliant adjunct were to be engineered, the EPIC portion of the system would have to be architected and audited as 21 CFR 11 compliant.

For providing 21 CFR Part 11 security measures, a gateway that used a DSTU3 client to retrieve the data from EPIC, then a FHIR R4 client to send Observation or QuestionnaireResponse resources to a server could implement the same Provenance protocol as the RAVE approach above. 




<!-- {% raw %}>{% include link-list.md %} {% endraw %}-->

{% include link-list.md %}
