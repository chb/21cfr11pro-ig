---
title: CapabilityStatements defined for this Guide
layout: default
active: capstatements
---
<!-- TOC  the css styling for this is \pages\assets\css\project.css under 'markdown-toc'-->

* Do not remove this line (it will not be displayed)
{:toc}

<!-- end TOC -->

The section outlines conformance requirements for each of the PRO 21 CFR 11 actors which include specific profiles, operations, security mechanisms and search parameters that need to be supported.

### Conformance Requirements for Actors supporting Provenance 

This section outlines conformance requirements for actors supporting the Provenance resource.

#### 21 CFR 11-Compliant PRO EDC or Warehousing IT System Conformance Requirements

The following are the conformance requirements for the 21 CFR 11-Compliant PRO EDC or Warehousing IT System.

##### Behavior and Formats

The 21 CFR 11-Compliant PRO EDC or Warehousing IT System **SHALL**: 
* Implement the RESTful behavior according to the [FHIR HTTP Specification](http://hl7.org/fhir/http.html).
* Support **json** resource formats for all PRO interactions.
* Declare a CapabilityStatement identifying the list of profiles, operations and search parameters supported.
* Respond to requests with the `Prefer: return=representation` header set with the full JSON of the server resource
* Implement a tamper-proof or tamper-evident cryptographic journal for recording at least Provenance and QuestionnaireResponse resource state and changes.

---
**Implementation Note**
The FHIR HTTP specification outlines specific requirements on how to deal with successful operations, invalid parameters, unauthorized request, insufficient scopes, unknown resources etc.

---
##### Profiles and Interactions

The 21 CFR 11-Compliant PRO EDC or Warehousing IT System needs to implement the profile specific interactions as specified table below.

---
**Implementation Note**
The way to read the table (for example the first row) is as follows

The 21 CFR 11-Compliant PRO EDC or Warehousing IT System **SHALL** support the CREATE, READ, SEARCH operations, SHOULD support the vREAD and HISTORY operations for the SDC QuestionnaireResponse profile.

Role based access control should be supported for each operation on each resource.

--- 

| Profile/Resource Name          | create     | read       | search (type level)      | vread          |  history (instance level) |
|:----------------------|-----------:|-----------:|-------------------------:|---------------:|--------------------------:|
|[SDC QuestionnaireResponse](http://build.fhir.org/ig/HL7/sdc/sdc-questionnaireresponse.html)|SHALL|SHALL|SHALL|SHOULD|SHOULD|
|[Provenance](http://hl7.org/fhir/us/core/StructureDefinition-us-core-provenance.html)|SHALL|MAY|MAY|MAY|MAY|
|[DocumentReference](http://hl7.org/fhir/us/core/StructureDefinition-us-core-documentreference.html)|SHALL|MAY|MAY|MAY|MAY|


--- 

##### Search and Other Parameters

The 21 CFR 11-Compliant PRO EDC or Warehousing IT System needs to implement the search parameters as specified below.

---
**Implementation Note**
The way to read the table (for example the first row) is as follows

The 21 CFR 11-Compliant PRO EDC or Warehousing IT System **SHALL** support _id, identifier, combination of identifier & version search parameters and _summary search result parameters for the SDC Questionnaire profile.

Role based access control should be supported for search on each resource.

--- 

| Profile Name          | SEARCH and Search Result parameters and combinations     |   
|:----------------------|---------------------------------------------------------:|
|[SDC QuestionnaireResponse](http://build.fhir.org/ig/HL7/sdc/sdc-questionnaireresponse.html)|_id (SHALL)|
||identifier (SHALL)|
||patient (SHALL)|
||questionnaire (SHALL)|
||author (SHALL)|
||authored (SHALL)|
|[Provenance](http://hl7.org/fhir/us/core/StructureDefinition-us-core-provenance.html)|target (SHALL)|
||patient (SHALL)|
||patient & date (SHALL)|
||Target referring to QuestionnaireResponse (SHALL)|
|[DocumentReference](http://hl7.org/fhir/us/core/StructureDefinition-us-core-documentreference.html)|target (SHALL)|
||patient (SHALL)|
||patient & date (SHALL)|
||Target referring to Patient (SHALL)|



##### Security Requirements

The 21 CFR 11-Compliant PRO EDC or Warehousing IT System **SHALL** support the Communication security mechanisms outlined in [FHIR Security Specification](http://hl7.org/fhir/security.html#http)

The 21 CFR 11-Compliant PRO EDC or Warehousing IT System **SHALL** support the Authentication security mechanisms outlined in [FHIR Security Specification](http://hl7.org/fhir/security.html#authentication)

The 21 CFR 11-Compliant PRO EDC or Warehousing IT System **SHOULD** support other security recommendations outlined in FHIR Security as appropriate.



<br />
