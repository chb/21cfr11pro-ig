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


### Implementation Guidance for tamper-proof transactions journal






#### PROM Instrument and Meta data Repository Implementation






