
### Tutorial and Walkthrough

This tutorial is intended for the beginner FHIR user who is implementing a FHIR interface for a legacy system to exchange user relationship data with other systems.  To illustrate the process, we will be taking one of the use cases (Payor to Payor Data Exchange), and walk through how it gets translated from use case into structured data that is sent over the wire.  

#### Step 1 - Define the Use Case

We begin with the following predefined use case.  When implementing this guide in your environment, you will want to collect the use case from your business sponsor, client, director, or other business executive. 

> When I move from one payer to another, I consent to allow some or all of my data to be sent to the new plan. In one form of this exchange, the new payer will get consent from me & send it to the previous payer. The consent provided must identify the subscriber as the key actor, and their relationship with both payers.  

#### Step 2 - Grammar Analysis  

Once obtained, you will want to do a grammar analysis, and identify the actors, organizations, and relationships defined in the example.  For example:

> When `username` moves from `payer1` to `payer2`, `username` consents `consent1` to allow some or all of their data to be sent to `payer2`. In one form of this exchange, `payer2` will get consent from `username` & send it to `payer1`. The consent provided must identify the subscriber as the key actor, and their relationship with both payers.

We then use the [FHIR Resourcelist](https://www.hl7.org/fhir/resourcelist.html) to identify the relevant FHIR resources and their schemas.  When identified, we create a FHIR-ized version of the use case, like so:  

> When [Patient1](https://www.hl7.org/fhir/patient.html) moves from [Organization1](https://www.hl7.org/fhir/organization.html) to [Organization2](https://www.hl7.org/fhir/organization.html), [Patient1](https://www.hl7.org/fhir/patient.html) consents [Consent1](https://www.hl7.org/fhir/consent.html) to allow some or all of their data to be sent to [Organization2](https://www.hl7.org/fhir/organization.html). In one form of this exchange, a [Practitioner2](https://www.hl7.org/fhir/practitioner.html) at [Organization2](https://www.hl7.org/fhir/organization.html) will get consent from [Patient1](https://www.hl7.org/fhir/patient.html) & send it to [Practitioner1](https://www.hl7.org/fhir/practitioner.html) at [Organization1](https://www.hl7.org/fhir/organization.html). The consent provided must identify the subscriber as the key actor, and their relationship with both payers.

#### Step 3.1 - Convert to Named Instances  

Using the resource schemas, you will now want to create instances for each of the actors and entities in the use case.  It does not matter which order steps 3.1 and 3.2 are done in; and will likely depend on whether you already have resources created for some of the actors; or if you are creating them from scratch.  
 
If creating from scratch, you will probably want to start by naming each of the actors and organizations.  Simply use the FHIR-ized version of the usecase to play [mad libs](https://www.madlibs.com/), and replace the resources with names of proper nouns, like so:  
 
> When `John Smith` moves from `ACME Insurance` to `Massachusetts Community Insurance`, `John Smith` consents to allow some or all of his data to be sent to `Massachusetts Community Insurance`. In one form of this exchange, `Alice Yin` at `Massachusetts Community Insurance` will get consent from `John Smith` & send it to `Bernard Shuman` at `ACME Insurance`. The consent provided must identify the subscriber as the key actor, and their relationship with both payers.

#### Step 3.2 - Define Actor and Organization Instances  

As you do this, you will want to begin creating structured JSON documents for each of the resources.  In many instances, you will already have these resources available, from either the examples in the Artifacts page of this implementation guide, or from a live FHIR server, or perhaps provided from a database.   

```js
// John Smith
{
    "id": "patient-john-smith",
    "resourceType": "Patient",
    "name": [{
        "family": "SMITH",
        "given": "JOHN"
    }]
}

// ACME Insurance
{
    "id": "org-acme-insurance",
    "resourceType": "Organization",
    "name": "ACME Insurance"
}

// Massachusetts Community Insurance
{
    "id": "org-mass-community-insurance",
    "resourceType": "Organization",
    "name": "Massachusetts Community Insurance"
}
```

You may find it useful to either define your usecase from existing resources in your data warehouse; or, conversely, you may wish to take examples from this implementation guide and load them in and hydrate a database.  The process tends to be iterative, and go back-and-forth; so don't get hung up on doing one or the other.  Just know that you're going to be developing these schemas in accordance to the usecase; or vice versa, depending on the needs of your business.  

#### Step 4 - Define Relationships  

Once you have stubbed out the resources for each of the actors, you will want to review the [ConsentExchangeRelations](./ConsentExchangeRelations.html) value set to get the code systems, code, and display values for the relationships.  

Begin encoding the relationships by identifying the managing organization that the patient belongs to, and the general practitioners that attend to the patient.

*John Smith*
```json
{
    "id": "patient-john-smith",
    "resourceType": "Patient",
    "name": [{
        "family": "SMITH",
        "given": "JOHN"
    }],
    "managingOrganization": {
        "display": "ACME Insurance",
        "reference": "Organization/org-acme-insurance"
    }
}
```
#### Step 5 - Create Consent Resource  

Once you have created resources for the actors and organizations in the scenario, you can then proceed to encoding the consent resource itself.  Specify the date, patient, organization, and the default policy rule.  Then you will want to begin adding provisions.  Consent records act as a filter mechanism, and are defined as restrictive by default with opt-in 'allow' provisions; or permissive by default, with opt-out 'deny' provisions.  You will need to specify a provision for each actor, and the associated action or function that provision covers.  You will again want to use the code systems, codes, and display values from the [ConsentExchangeRelations](./ConsentExchangeRelations.html) value set for use in the `provision.actor.role` field.  

```json
{
  "resourceType" : "Consent",
  "id" : "12345",
  "status" : "active",
  "patient" : {
    "reference" : "Patient/patient-john-smith",
    "display" : "SMITH, JOHN"
  },
  "dateTime" : "2021-06-01",
  "organization" : [{
    "reference" : "Organization/org-mass-community-insurance",
    "display" : "Massachusetts Community Insurance"
  }],
  "policyRule" : {
    "coding" : [
      {
        "system" : "http://mehi-ig.tranquildata.com/CodeSystem/CmsInteroperabilityPolicyCodes",
        "code" : "CMS-9115-F.V",
        "display" : "Health Information Exchange and Care Coordination Across Payers"
      }
    ]
  },
  "provision" : {
    "type" : "permit",
    "period" : {
      "start" : "2021-06-01",
      "end": "2022-06-01"
    },
    "actor" : [
      {
        "role" : {
          "coding" : [
            {
              "system" : "http://terminology.hl7.org/CodeSystem/v3-RoleClass",
              "code" : "PROV",
              "display" : "healthcare provider"
            }
          ]
        },
        "reference" : {
          "reference" : "Organization/org-acme-insurance",
          "display" : "ACME Insurance"
        }
      }
    ],
    "action" : [
      {
        "coding" : [
          {
            "system" : "http://terminology.hl7.org/CodeSystem/consentaction",
            "code" : "access",
            "display" : "Access"
          }
        ],
        "text" : "Access"
      }
    ]
  }
}
```


#### Step 6 - Fetch Over HTTP   

Once the consent is constructed, it can be served up by an endpoint URL.

```bash
# fetch a specific record, and only that record
# this is the default 
GET http://localhost:3000/baseR4/Consent/12345

# the _include operation will fetch the primary consent record
# as well as the Organization records which it references
# these records will be added to the Consent.contains array
GET http://localhost:3000/baseR4/Consent/12345?_include=Organization.*
```

The above example with `_include` should return a record that looks like the following:  


```json
{
  "resourceType" : "Consent",
  "id" : "12345",
  "status" : "active",
  "patient" : {
    "reference" : "Patient/patient-john-smith",
    "display" : "SMITH, JOHN"
  },
  "dateTime" : "2021-06-01",
  "organization" : [{
      "reference" : "Organization/org-acme-insurance",
      "display" : "ACME Insurance"
  }],
  "policyRule" : {
    "coding" : [
      {
        "system" : "http://mehi-ig.tranquildata.com/ValueSet/ConsentExchangePolicies",
        "code" : "CMS-9115-F",
        "display" : "CMS Interoperability and Patient Access final rule"
      }
    ]
  },
  "provision" : {
    "type" : "permit",
    "period" : {
      "start" : "2021-06-01",
      "end": "2022-06-01",
    },
    "actor" : [
      {
        "role" : {
          "coding" : [
            {
              "system" : "http://terminology.hl7.org/CodeSystem/v3-RoleClass",
              "code" : "PROV",
              "display" : "healthcare provider"
            }
          ]
        },
        "reference" : {
          "reference" : "Organization/org-mass-community-insurance",
          "display" : "Massachusetts Community Insurance"
        }
      }
    ],
    "action" : [
      {
        "coding" : [
          {
            "system" : "http://terminology.hl7.org/CodeSystem/consentaction",
            "code" : "access",
            "display" : "Access"
          }
        ],
        "text" : "Access"
      }
    ]
  },
  "contained": [{
    "id": "1",
    "resourceType": "Organization",
    "name": "ACME Insurance"
  }, {
    "id": "2",
    "resourceType": "Organization",
    "name": "Massachusetts Community Insurance"
  }]
}
```
#### Step 7 - Post Over HTTP   

Alternatively, you may wish to post the consent record elsewhere.  

```bash
# if the Organizations already exist on the FHIR server
# and you fill out the Consent resource correctly
# you can use POST to ask the server to assign a Consent.id value
POST http://localhost:3000/baseR4/Consent

# if you would like to specify the Organization ids explicitly
# you will need to use the PUT operation to register the resource with the desired id
# then fill out the Consent resource accordingly and PUT it to the server
PUT http://localhost:3000/baseR4/Organization/org-acme-insurance
PUT http://localhost:3000/baseR4/Organization/org-mass-community-insurance
PUT http://localhost:3000/baseR4/Consent/12345
```
