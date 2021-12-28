
### Background  

Modern clinical information systems are regulated by various laws, including (but not limited to) HIPAA, GDPR, and 21st Century Cures.  These laws outline expectations for patient privacy and access that clinicians are bound to.  In practice, these laws specify a default expectation of privacy, with the patient having the right to grant or deny access to their information to others.  When viewed through the lens of computer information systems, these laws detail a combination of data collection forms that need to be implemented and respected, and which inform the access control lists of who may log into which systems, and what information they are allowed to access or modify.    

### Purpose  

The purpose of this implementation guide is to help developers implement systems that exchange consent and advance directives data to be used in generating access control lists.  These systems may range widely from Electronic Medical Records (EHR) running in data warehouses, to Personal Health Records (PHR) running on personal mobile devices, to dedicated Consent Engine services, to graph databases, to blockchains, and many other services or products yet to be imagined.  

### Prior Art References  

- [PACIO Implemention Guide](https://www.hl7.org/fhir/implementationguide.html)    
- [SDOH Clinical Implementation Guide (Gravity)](https://build.fhir.org/ig/HL7/sdoh-cc/)  
- [ONC LEAP - Consent Management and Enforcement - Connectathon Track](https://confluence.hl7.org/display/FHIR/2021-01+Consent+Management+and+Enforcement+Services+Track)    


<!-- ### Domain Analysis 

This implementation guide was developed after performing a meta-analysis of three related implementation guides - SDOH Clinical Care (Gravity), PACIO Advanced Directives, and the LEAP Connectathon Track.  For those not familiar with this prior art, the Gravity IG has a particular emphasis on social determinates of health and translating the health conditions/goals of patients into consent records; while the PACIO IG has an emphasis on the advanced directive documents themselves, and the ONC Leap IG has an emphasis on clinician workflows within the hospital and clinical research environments.  

![MeHi-RosettaStone-DomainAnalysis](MeHi-RosettaStone-DomainAnalysis.png){:width="100%"}  

The common denominator between these three implementation guides was the Consent, Questionnaire, Patient, Practitioner, RelatedPerson, and Organization resources.

This implementation guide seeks to start with these resources as a base, add learnings from multiple implementations of advanced directives websites in the Node/Javascript and blockchain community, with a focus on security and access control lists.   -->

### Actors, Roles, and Relationships  

The following types of actors are recognized within the FHIR API, and external systems should map user accounts to these data resources accordingly.  Please reference the API documention for schemas and implementation details.  

- [Patient](https://www.hl7.org/fhir/patient.html)  
- [Practitioner](https://www.hl7.org/fhir/practitioner.html)  
- [Organization](https://www.hl7.org/fhir/organization.html)  
- [RelatedPerson](https://www.hl7.org/fhir/relatedperson.html)  
- [CareTeam](https://www.hl7.org/fhir/careteam.html)  

Additionally, the following two abstract group roles are valid for modeling relationships between individuals within FHIR. 

- [PractitionerRole](https://www.hl7.org/fhir/practitionerrole.html)  
- [OrganizationAffiliation](https://www.hl7.org/fhir/organizationaffiliation.html)  

These actors, roles, and relations eventually get used to populate more complex resources, primarily the Consent resource.  
- [Consent](https://www.hl7.org/fhir/consent.html)  


### Terminology  

When modeling relationships between entities, you will want to avail yourself of some of the existing terminologies, code systems, and value sets that are already in use.  Please refer to the following docummentation for inspiration and guidance on existing data models in use by the FHIR spec.

- [PatientRelationshipType](http://hl7.org/fhir/ValueSet/relatedperson-relationshiptype)    
- [PractitionerRole](https://www.hl7.org/fhir/valueset-practitioner-role.html)    
- [PracticeSettingCodeValueSet](https://www.hl7.org/fhir/valueset-c80-practice-codes.html)  
- [OrganizationType](http://hl7.org/fhir/ValueSet/organization-type)    
- [OrganizationAffiliationRole](https://www.hl7.org/fhir/valueset-organization-role.html)  
- [ParticipantRoles](https://www.hl7.org/fhir/valueset-participant-role.html)    
- [ConsentPolicyRuleCodes](http://hl7.org/fhir/ValueSet/consent-policy)    
- [ConsentProvisionType](http://hl7.org/fhir/ValueSet/consent-provision-type)  
- [SecurityRoleType](http://hl7.org/fhir/ValueSet/security-role-type)  
- [ConsentActionCodes](http://hl7.org/fhir/ValueSet/consent-action)  
- [SecurityLabels](https://www.hl7.org/fhir/valueset-security-labels.html)  
- [ConsentContentClass](http://hl7.org/fhir/ValueSet/consent-content-class)  
- [ConsentContentCodes](http://hl7.org/fhir/ValueSet/consent-content-code)  


Patient/Provider relationships are encoded by the `Patient.generalPractitioner` and `Patient.managingOrganization` fields.  Patient/ReleatedPerson relationships are encoded by `RelatedPerson.patient` and `RelatedPerson.relationship` fields.  For basic familial relationships, you will want to familiarize yourself with the `relatedperson-relationshiptype` value set.


### Use Case Scenarios    

*Scenario 1 - Interstate Pediatric Continuity of Care*  
The family lives in Alabama, where Jennifer visits Dr. Shuman for contraceptives.  Mrs. Smith sees details of her daughter's vist in her _Together Health_ app.  The family moves to MA, where Jennifer sees Dr. Yin for contraceptives.  By default, Mrs. Smith does not see details of the visit in her _Together Health_ app.  To maintain her engagement with her family and her health, Jennifer may consent to share with her monther, but in MA this is her choice and technology should support that...  

*Scenario 2 - Healthservice Price Transparency Shopping*  
When the move to MA, Mr. Smith looks for a new pharmacy.  He consents to allow _PlentyRX_ to assert his ID not only to fill prescriptions, but also to suggest & discount supplements based on what he already takes.  Over time, Mr. Smith decides that another service can better guide his regime with specific access to his health records.  He revokes consent for _PlentyRX_, and grants consent to his new vendor _PillBox_.  

*Scenario 3 - Patient Chooses a New Insurance Plan*  
When John Smith moves to MA, he changes insurance from ACME Insurance (Payer 1) to Massachusetts Community Insurance (Payor 2.  John consent to allow some or all of his data to be sent to the new plan. In one form of this exchange, the Payer 2 will get consent from John & send it to the Payer 1. The consent provided must identify the subscriber as the key actor, and their relationship with both payers. In TQD parlance, the Subject has a Consent Relationship with Payer 2, starting on a known date, and possibly scoped to a subset of data, that will allow the old Payer to send data.  

When John logs into PillBox using the Massachusetts Community Insurance ID, personal data is shared from Payor 1 to the App containing standardized meta-data to ensure a consistent engagement experience. John has attributes like DoB and State of Residence, and some number of familial relationships (to Jennifer Smith, Jane Smith, etc) and general relationships (to Boston Pharmacy, Together Health App).  The familial relationships are "just" stated relationships (and as we've discussed, common healthcare relationships map in the same way). The general relationships are non-HIPAA consents to share some scoped set of data, over some time-frame (which may still be active).

*Scenario 4 - Payor to Payor Data Exchange*
When John Smith moves from ACME Insurance to Massachusetts Community Insurance, John consents to allow some or all of his data to be sent to Massachusetts Community Insurance. In one form of this exchange, Massachusetts Community Insurance will get consent from John & send it to ACME Insurance. The consent provided must identify the subscriber as the key actor, and their relationship with both payers.


<!-- *Scenario 3 - Provider to Payor Data Exchange*  

Johnny Appleseed is a 34 year old male, living in Pittsburgh PA, and who has insurance through (Payor1).  His primary care physician is Dr. Susan Smith, who works at (Provider1).  Johnny visits Dr. Smith for a follow up exam, wherein he is prescribed Lisinopril, a blood pressure medication.  When complete, (Provider1) send an electronic bundle containing the medication prescription, patient, and provenance to (Payor 1).  A copy of the HIPAA Privacy Consent is included as a contained resource in the bundle.  

*Scenario 4 - Healthcare Agents (PACIO)*  

Betsy Smith-Johnson is a 71 yr old female, who is using a website MyDirectives.com to create a Living Will.  MyDirectives.com refers her to a partner organization, AdVault, which provides a patient portal and responsible for assembling her will.  Betsy logs into the AdVault portal, and specifies her children, Charles and Debra Johnson, as her healthcare agents; along with various personal intervention preferences and care experience preferences.  The AdVaultPortal then generates a Living Will document, which is sent to MyDirectives.com for long term custodial archiving.  

Later, Betsy becomes ill, and is sent to a nursing home (Provider).  Charles and Debra wish to view Betsy's medical history during her treatments, and go to the nursing home intake coordinator, where they present the Living Will from MyDirectives.com.  Provider queries MyDirectives.com for consent records to allow Charles and Debra access to the Betsy's medical records.  

*Scenario 5 - Advanced Directive Data Collection (Gravity)*  

Pieter van de Heuvel is a 77yr old male, living in Amsterdam.  The netherlands have a national food security program, and all citizens are regularly screened during annual physicals that they are receiving enough food and sustenance.  During a routine visit in July, 2019; Dr. Anne Langeveld determines that Pieter has not been receiving enough food the past 2 months, and assists him with applying for a food aid program (i.e food stamps).  She records Pieter as having a condition of food insecurity in the medical record, and a health care goal of food security.  3 months later, during a follow up visit, Pieter is seen by Dr. Simone Heps, who follows up with Pieter, and determines that the food aid program is allowing Pieter to eat 3 full meals a day.  

Dr. Langevevld and Helps use an EHR that supports patient lists and tracking of Hunger Vital Signs questionnaires.  The Food Insecurity (Hunger Vital Sign) Questionnaire Patient List currently contains two patients of concern, Jeremy Bates and Alice Newman.  Jeremy logs in via the patient portal, and fills out a Food Insecurity (Hunger Vital Sign) questionnaire.  While doing so, he signs a consent to collect information consent form.

*Scenario 6 - ONC LEAP - Provider to Provider ADI Exchange*  

Green Valley Hospital and Mariposea Community Health Center are business associates under HIPAA regulations, and both located in Arizona.  As busines associates, they have entered legal agreements to share legal documents between the two organizations, including HIPAA Patient Privacy consent forms, Living Wills, Powers of Attorney, and other advanced care directives.  Various patients enter either of the organizations, specify HIPAA patient privacy documents, and physicians at the other organization have access to patient medical charts based on the detials present in the ADI documents and consent forms.   -->


### Data Model  

From these usecases and clinical scenarios, the following data model was extracted. 

![./MeHIConsentExchange.jpg](./MeHIConsentExchange.jpg){:width="100%"}

The core consent engine data model center around the FHIR Consent resource, and the resources pertaining to human and organizational actors (in so far as legal jurisprudice concerns itself with human actors).   

Secondary workflows include the structured data capture workflows, in which Questionnaires are used to create forms, which then collect Responses, which are then mapped into Consent records.  Additionally, workflow for security and advanced network topologies such as blockchain are provided by inclusion of the FHIR Provenance record.

Finally, exploratory consideration is given to CareTeam resource for use in modeling clinical care workflows; but which creates complex taxonomies requiring recursive parsing algoritms.  

### Endpoints    

Endpoint query patterns vary depending on business usecase and workflow, but are generally organized around the perspective of each of the actors or organizations.  Typical queries will be around the patient accessing his or her own consent records; an organization querying which records a practitioner has access to; an organization determining if a specific related person is authorized to access a patient's records, and so forth.  You will want to review the following endpoints and queries, and select the ones most relevant to your  use case. 

<!-- 
*Data Collection*  
```
GET /Questionnaire/{questionnaireId}
POST /QuestionnaireResponse
GET /QuestionnaireResponse/{questionnaireId}
GET /QuestionnaireResponse?patient={patientId}
PUT /QuestionnaireResponse/{questionnaireId}
DELETE /QuestionnaireResponse/{questionnaireResponseId}
``` 
*Relationship Modeling*  
-->


```
GET /Patient/{patientId}
GET /Patient?_id={patientId}
GET /Patient?organization={managingOrganizationId}
GET /Organization/{organizationId}
GET /Organization?_id={organizationId}
GET /Practitioner/{practitionerId}
GET /Practitioner?_id={practitionerId}
GET /PractitionerRole/{practitionerRoleId}
GET /PractitionerRole?_id={practitionerRoleId}
GET /RelatedPerson/{relatedPersonId}
GET /RelatedPerson?_id={relatedPersonId}
GET /RelatedPerson?patient={patientId}
POST /Consent/{patientId}  
GET /Consent/{consentId}
GET /Consent?actor={actorReference}
GET /Consent?consentor={consentorReference}
GET /Consent?subject=Patient/{patientId}
GET /Consent?organization=Organization/{organizationId}
GET /Consent?performer=Practitioner/{practitionerId}
GET /Consent?performer=RelatedPerson/{relatedPersonId}
GET /Consent?performer=PractitionerRole/{practitionerRoleId}
DELETE /Consent/{patientId}  
```

<!-- *Consent Engine & Access Control List Generation*  

```
PUT /Consent/{patientId}/$parse
GET /Consent/$parseToBundle?patient=Patient/{patientId}
GET /Consent/$parseToBundle?organization=Organization/{organizationId}
GET /Consent/$parseToBundle?consentor=RelatedPerson/{relatedPersonId}
POST /Consent/{consentId}/$equals
POST /Consent/{consentId}/$diff
GET /Consent/$toAcl?patient=Patient/{patientId}
GET /Consent/$toAcl?organization=Organization/{organizationId}
GET /Consent/$toAcl?consentor=RelatedPerson/{relatedPersonId}
GET /Consent/$oauthScopes?patient=Patient/{patientId}
GET /Consent/$oauthScopes?organization=Organization/{organizationId}
GET /Consent/$oauthScopes?consentor=RelatedPerson/{relatedPersonId}
``` -->


### Sponsors  

This implementation guide is generously sponsored by Tranquil Data and the Massachusetts eHealth Institute (MeHI). For more information on the background of this project, please see the [Mass. eHealth Institute Supports Digital Health Innovation Statewide](https://masstech.org/press-releases/mass-ehealth-institute-supports-digital-health-innovation-statewide) press release.  


### Learning Resources & Background Information  

[Getting Started with FHIR](http://hl7.org/fhir/modules.html)  
[FHIR Resource List](https://www.hl7.org/fhir/resourcelist.html)  
[HL7 Education and Certification Course List](http://www.hl7.org/implement/courseList.cfm?ref=nav)  

# Contact Info  

For project maintenance, please contact:    

Seth Proctor <stp@tranquildata.com>  
Shawn Flaherty <sf@tranquildata.com>    
Reece Adamson <radamson@mitre.org>  
Russ Graves <digger@mitre.org>

For community help, please post questions on Zulip:  
[https://chat.fhir.org/](https://chat.fhir.org/)  

### Copyright Notice  

This (software/technical data) was produced under Contract Number 6521MT01-01, and is subject to Federal Acquisition Regulation Clause 52.227-14, Rights in Data-General.

No other use other than that granted to the U. S. Government, or to those acting on behalf of the U. S. Government under that Clause is authorized without the express written permission of The MITRE Corporation.

For further information, please contact The MITRE Corporation, Contracts Management Office, 7515 Colshire Drive, McLean, VA 22102-7539, (703) 983-6000.

&copy; 2022 The MITRE Corporation.
