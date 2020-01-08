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

** TBD: The implementation guidance outlined in this page provides useful information for developers implementing the various PRO actors and their capabilities. All resources and parameters used in this part of the IG are examples and are expected to conform to the resources, profiles and other constraints outlined in the [CapabilityStatements](capstatements.html).

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

The ID certificate should also be made available to any system or person who needs to verify the authenticity of signed resources.

### Provenance and Resource Signing

Signing resources provides a mechanism to validate that a resource was genuinely provided by a FHIR client with a known identity and has not been altered since it was signed.

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


### Cryptographic Journaling and Auditing


To audit every resource of interest the following process needs to be supported:
  1. The resource is retrieved
  2. The corresponding Provenance entry is retrieved
  3. The certificate referenced in the Provenance entry is retrieved
  4. The certificate (from 3) is used to validate that the signature (retreived in 2) corresponds to the canonical version of the resource (retrieved in 1)


***

### Implementation Guidance for administering Basic Questionnaires

The section outlines the implementation guidance for the various actors involved in the administration of Basic Questionnaires.

#### PROM Instrument and Meta data Repository Implementation

​The capabilities and requirements that need to be implemented by the PROM Instrument and Meta data Repository are outlined at [CapabilityStatements](capstatements.html)

Developers can follow these steps to implement the above capabilities.

**Step 1:** Implement a FHIR Server. (Choose Release 4)

There are many open source FHIR Servers that are available to be downloaded and used instead of starting from scratch. These can be found at [FHIR Open Source Implementations](http://wiki.hl7.org/index.php?title=Open_Source_FHIR_implementations).

Publish the FHIR Server Capability Statement for the Server.

**Step 2:** Implement the APIs for Questionnaires

* POST API : Support Creation of a PRO Measure using Questionnaire Resource and store it in a repository. 

`POST [BaseURL]/Questionnaire/` - Submit payload (body) of Questionnaire in JSON format.


* GET API : Will be used by an EHR or an External PRO Administration App to download a specific questionnaire.

`GET [BaseURL]/Questionnaire/1234` - Receive a payload (body) of Questionnaire in JSON format.

`GET [BaseURL]/Questionnaire/?_summary=true` - Receive all Questionnaires present in the system in summary format.

* Search API - Find Specific questionnaires.

`GET [BaseURL]/Questionnaire?_identifier=phq-9` - Receive a phq-9 Questionnaire.


**Step 3:** Implement appropriate security protocols for the Server.

At a minimum, HTTPS protocols must be used for the implementation using the right certificate.

#### EHR or Care Delivery Health IT System

​The capabilities and requirements that need to be implemented by the EHR or Care Delivery Health IT System to support PRO requirements are outlined at [CapabilityStatements](capstatements.html)

Developers can follow these steps to implement the above capabilities.

**Step 1:** Implement a FHIR Server. (Choose Release 4)

There are many open source FHIR Servers that are available to be downloaded and used instead of starting from scratch. These can be found at [FHIR Open Source Implementations](http://wiki.hl7.org/index.php?title=Open_Source_FHIR_implementations).

Map EHR data (schemas,tables,lookup tables) to FHIR Resources and interfaces as needed. (Some common FHIR resources that will be required are Patient, Observation, and Procedure)

Publish the FHIR Server Capability Statement for the Server.

**Step 2a:** Implement the APIs for Questionnaires

* POST API : Support creation of PRO Measure using Questionnaire Resource and store it internally for tracking administrations. 

`POST [BaseURL]/Questionnaire/` - Submit payload (body) of Questionnaire in JSON format.


* GET API : Will be used to download a specific Questionnaire by authorized systems and personnel.

`GET [BaseURL]/Questionnaire/1234` - Receive a payload (body) of Questionnaire in JSON format.

`GET [BaseURL]/Questionnaire/?_summary=true` - Receive all Questionnaires present in the system in summary format.

* Search API - Find Specific questionnaires.

`GET [BaseURL]/Questionnaire?_identifier=phq-9` - Receive a phq-9 Questionnaire.

**Step 2b:** Implement the APIs for QuestionnaireResponse

* POST API : Support collection of PRO Responses using QuestionnaireResponse Resource and store it internally for tracking administrations. 

`POST [BaseURL]/QuestionnaireResponse/` - Submit payload (body) of QuestionnaireResponse in JSON format.


* GET API : Will be used to download a specific QuestionnaireResponse (PRO Responses) by authorized systems and personnel.

`GET [BaseURL]/QuestionnaireResponse/1234` - Receive a payload (body) of QuestionnaireResponse in JSON format.

* Search API - Find Specific QuestionnaireResponses.

`GET [BaseURL]/QuestionnaireResponse?patient=1234` - Receive all QuestionnaireResponses for a patient

`GET [BaseURL]/QuestionnaireResponse?questionnaire=1234&authored=gt2017-01-15` - Receive QuestionnaireResponses for a specific questionnaire administered after Jan 15 2017.

**Step 3:** Implement appropriate security protocols for the Server.

At a minimum, HTTPS protocols must be used for the implementation using the right certificate.

**Step 4:** Implement SMART on FHIR Authorization Server (Only if the EHR supports External PRO Administration System)

When the EHR supports integration with External PRO Administration System, the EHR system has to be complemented with a SMART on FHIR Authorization Server that can be used to authorize the External App. The External App would be launched with Patient Context and after the data is collected the External App will submit the data to the EHR using the POST APIs.

To implement the SMART on FHIR Authorization Server and other SMART on FHIR capabilities, refer to [SMART on FHIR Sandbox](https://sandbox.smarthealthit.org).

#### External PRO Administration System

​The capabilities and requirements that need to be implemented by the External PRO Administration System to support PRO requirements are outlined at [CapabilityStatements](capstatements.html)

Developers can follow these steps to implement the above capabilities.

**Step 1:** Implement a FHIR Server. (Choose Release 4)

There are many open source FHIR Servers that are available to be downloaded and used instead of starting from scratch. These can be found at [FHIR Open Source Implementations](http://wiki.hl7.org/index.php?title=Open_Source_FHIR_implementations).

Publish the FHIR Server Capability Statement for the Server.

**Step 2a:** Implement the APIs for Questionnaires

* POST API : Support creation of PRO Measure using Questionnaire Resource and store it internally for tracking administrations. 

`POST [BaseURL]/Questionnaire/` - Submit payload (body) of Questionnaire in JSON format.

* GET API : Will be used to download a specific Questionnaire by authorized systems and personnel.

`GET [BaseURL]/Questionnaire/1234` - Receive a payload (body) of Questionnaire in JSON format.

`GET [BaseURL]/Questionnaire/?_summary=true` - Receive all Questionnaires present in the system in summary format.

* Search API - Find Specific questionnaires.

`GET [BaseURL]/Questionnaire?_identifier=phq-9` - Receive a phq-9 Questionnaire.

**Step 2b:** Implement the APIs for QuestionnaireResponse

* POST API : Support collection of PRO Responses using QuestionnaireResponse Resource and store it internally for tracking administrations. 

`POST [BaseURL]/QuestionnaireResponse/` - Submit payload (body) of QuestionnaireResponse in JSON format.


* GET API : Will be used to download a specific QuestionnaireResponse (PRO Responses) by authorized systems and personnel.

`GET [BaseURL]/QuestionnaireResponse/1234` - Receive a payload (body) of QuestionnaireResponse in JSON format.

* Search API - Find Specific QuestionnaireResponses.

`GET [BaseURL]/QuestionnaireResponse?patient=1234` - Receive all QuestionnaireResponses for a patient

`GET [BaseURL]/QuestionnaireResponse?questionnaire=1234&authored=gt2017-01-15` - Receive QuestionnaireResponses for a specific questionnaire administered after Jan 15 2017.

**Step 3:** Implement appropriate security protocols for the Server.

At a minimum, HTTPS protocols must be used for the implementation using the right certificate.

**Step 4:** Implement SMART on FHIR Authorization for clients (Only if the External PRO Administration System interacts with an EHR)

When the External PRO Administration System interacts with an EHR, the External PRO Administration System has to be registered with the EHRs Authorization Server to enable launching from within the EHR using SMART on FHIR Authorization protocols. After launch, the External PRO Administration System must support the query of appropriate resources for context. 

To implement the SMART on FHIR Authorization Server and other SMART on FHIR capabilities, refer to [SMART on FHIR Sandbox](https://sandbox.smarthealthit.org).

Once the data is collected, the External PRO Administration System may store the PRO responses in the EHR using the POST API creating the QuestionnaireResponse.

### Implementation Guidance for administering Adaptive Questionnaires

The section outlines the implementation guidance for the various actors involved in the administration of Adaptive Questionnaires.

#### Patient Facing Administration App

​The capabilities and requirements that need to be implemented by the Patient Facing Administration App to support PRO requirements are outlined at [CapabilityStatements](capstatements.html)

Developers can follow these steps to implement the above capabilities.

**Step 1:** Implement a FHIR Server. (Choose Release 4)

There are many open source FHIR Servers that are available to be downloaded and used instead of starting from scratch. These can be found at [FHIR Open Source Implementations](http://wiki.hl7.org/index.php?title=Open_Source_FHIR_implementations).

Publish the FHIR Server Capability Statement for the Server.

**Step 2a:** Implement the APIs for Questionnaires

* POST API : Support creation of PRO Measure using Questionnaire Resource and store it internally for tracking administrations. 

`POST [BaseURL]/Questionnaire/` - Submit payload (body) of Questionnaire in JSON format.


* GET API : Will be used to download a specific Questionnaire by authorized systems and personnel.

`GET [BaseURL]/Questionnaire/1234` - Receive a payload (body) of Questionnaire in JSON format.

`GET [BaseURL]/Questionnaire/?_summary=true` - Receive all Questionnaires present in the system in summary format.

* Search API - Find Specific questionnaires.

`GET [BaseURL]/Questionnaire?_identifier=phq-9` - Receive a phq-9 Questionnaire.

**Step 2b:** Implement the APIs for QuestionnaireResponse

* POST API : Support collection of PRO Responses using QuestionnaireResponse Resource and store it internally for tracking administrations. 

`POST [BaseURL]/QuestionnaireResponse/` - Submit payload (body) of QuestionnaireResponse in JSON format.


* GET API : Will be used to download a specific QuestionnaireResponse (PRO Responses) by authorized systems and personnel.

`GET [BaseURL]/QuestionnaireResponse/1234` - Receive a payload (body) of QuestionnaireResponse in JSON format.

* Search API - Find Specific QuestionnaireResponses.

`GET [BaseURL]/QuestionnaireResponse?patient=1234` - Receive all QuestionnaireResponses for a patient

`GET [BaseURL]/QuestionnaireResponse?questionnaire=1234&authored=gt2017-01-15` - Receive QuestionnaireResponses for a specific questionnaire administered after Jan 15 2017.

**Step 3:** Implement appropriate security protocols for the Server.

At a minimum, HTTPS protocols must be used for the implementation using the right certificate.

**Step 4:** Implement the Adaptive Questionnaire Administration following the guidance provided at 
[Adaptive Question Administration](http://build.fhir.org/ig/HL7/sdc/adaptive.html) using the [Questionnaire Next Question operation](http://build.fhir.org/ig/HL7/sdc/Questionnaire-next-question.html).

**Step 5:** Convert results from Adaptive Questionnaire Administration to Questionnaire and QuestionnaireResponse for access using APIs outlined above.

#### External Assessment Center

​The capabilities and requirements that need to be implemented by the External Assessment Center to support PRO requirements are outlined at [CapabilityStatements](capstatements.html)

Developers can follow these steps to implement the above capabilities.

**Step 1:** Implement a FHIR Server. (Choose Release 4)

There are many open source FHIR Servers that are available to be downloaded and used instead of starting from scratch. These can be found at [FHIR Open Source Implementations](http://wiki.hl7.org/index.php?title=Open_Source_FHIR_implementations).

Publish the FHIR Server Capability Statement for the Server.

**Step 2a:** Implement the APIs for Questionnaires

* POST API : Support creation of PRO Measure using Questionnaire Resource and store it internally for administrations. 

`POST [BaseURL]/Questionnaire/` - Submit payload (body) of Questionnaire in JSON format.


* GET API : Will be used to download a specific Questionnaire by authorized systems and personnel.

`GET [BaseURL]/Questionnaire/1234` - Receive a payload (body) of Questionnaire in JSON format.

`GET [BaseURL]/Questionnaire/?_summary=true` - Receive all Questionnaires present in the system in summary format.


**Step 2b:** Implement the APIs for QuestionnaireResponse

* GET API : Will be used to download a specific QuestionnaireResponse (PRO Responses) by authorized systems and personnel.

`GET [BaseURL]/QuestionnaireResponse/1234` - Receive a payload (body) of QuestionnaireResponse in JSON format.

**Step 3:** Implement appropriate security protocols for the Server.

At a minimum, HTTPS protocols must be used for the implementation using the right certificate.

**Step 4:** Implement the Adaptive Questionnaire Administration following the guidance provided at 
[Adaptive Question Administration](http://build.fhir.org/ig/HL7/sdc/adaptive.html) using the [Questionnaire Next Question operation](http://build.fhir.org/ig/HL7/sdc/Questionnaire-next-question.html).



