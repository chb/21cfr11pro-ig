---
title: Patient Reported Outcomes 21 CFR 11 Compliance FHIR Implementation Guide 
layout: default
active: home
---


<!-- TOC  the css styling for this is \pages\assets\css\project.css under 'markdown-toc'-->

* Do not remove this line (it will not be displayed)
{:toc}


<!-- end TOC -->



###  Introduction

This Patient Reported Outcomes 21 CFR 11 Compliance FHIR Implementation Guide complements the Patient Reported Outcomes (PRO) FHIR Implementation Guide (IG) 
to provide additional information on how to implement a FHIR-based PRO collection and storage system that
adheres to the United States of America's Code of Federal Regulations Title 21, Part 11 on electronic records and signatures ([21 CFR Part 11](https://www.accessdata.fda.gov/scripts/cdrh/cfdocs/cfCFR/CFRSearch.cfm?CFRPart=11&showFR=1)).

While the PRO implementation guide focuses on capturing and exchanging patient reported outcome data electronically using the FHIR standard, this implementation guide addresses many of the technical security
and auditing requirements in the Federal regulations. 

PRO FHIR IG references other implementation guide dependencies that are not re-listed here for 
maintainabililty. 

###  Guidance to the readers

The following table introduces the content of this implementation guide.

| Topic to Read  | What it Contains and its relationship to PRO IG | Where can I find the content ? |
|:---------------|:------------------------------------------------|-------------------------------:|
| Basic Definitions | The set of definitions applicable to the PRO FHIR IG. (Definition of“Supported or MUST Support”, Usage of Code Bindings in US Core Profiles.).| [US-Core Definitions]({{site.data.fhir.uscoreR4}}index.html)|
| PRO Overview | The artifact provides background on Patient Reported Outcomes, Patient Reported Outcome Measures and other PRO related topics.| [PRO Overview](pro-overview.html)|
| Profiles | The artifact defines the various profiles, extensions and resources that make up the PRO FHIR IG.| [Profiles](profiles.html)|
| Capability Statements | The artifact defines the various capability statements (implementation requirements) for each PRO actor that make up the PRO FHIR IG.| [Capability Statements](capstatements.html)|
| Implementation Guidance | The artifact contains guidance and examples that will help implementers of PRO FHIR IG.| [Implementation Guidance](guidance.html)|




<!-- {% raw %}>{% include link-list.md %} {% endraw %}-->

{% include link-list.md %}
