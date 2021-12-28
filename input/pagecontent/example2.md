
### Tutorial and Walkthrough

This tutorial extends on the previous tutoral, to consider organizational relationships.   

#### Step 1 - Define the Use Case

We continue with the following use case:  

> When personal data is shared it should contain meta-data to ensure a consistent engagement experience. I will have attributes like DoB or State of Residence, and some number of relationships. For this example, I have at least 2 family relationships (which should name the other party but divulge no other details about them), and at least two general (not Provider or Payer) relationships. These might be to a pharmacy, to an online service, to an app on my phone, a friend, etc. The familial relationships are "just" stated relationships (and as we've discussed, common healthcare relationships map in the same way). The general relationships are non-HIPAA consents to share some scoped set of data, over some time-frame (which may still be active).  

#### Step 2 - Grammar Analysis  

Again, we do a grammar analysis to identify the actors, organizations, and relationships defined in the example.  For example:

> `username` will have attributes like `birthDate` or `address.state`, and some number of relationships. For this example, `username` have at least 2 family `related persons` relationships, and at least two general `organizational` relationships. These might be to `pharmacy1`, to an `app1` on my phone, etc. The general relationships share some scoped set of data according to the `CMS Interoperability and Patient Access final rule`, over some `period of time`.  

We then use the [FHIR Resourcelist](https://www.hl7.org/fhir/resourcelist.html) to identify the relevant FHIR resources and their schemas.  When identified, we create a FHIR-ized version of the use case, like so:  

> `Patient2` will have attributes like `birthDate` and `address.state`, and some number of `RelatedPerson` records. For this example, `Patient1` will be referenced as a familial relationship by `RelatedPerson1` and `RelatedPerson2`, and at least two general organizational relationships by `Organization1` (pharmacy) and `Organization2` (app). The general relationships share some scoped set of data according to the `CMS Interoperability and Patient Access final rule`, over some `period of time`.  

#### Step 3.1 - Convert to Named Instances  

Using the resource schemas, we now want to create instances for each of the actors and entities in the use case, like so:
 
> `John Smith` was born on April 14th, 1980 in Alabama.  He moved back to his home state in 2005, where he resided until June 2020, when he and his family moved to Massachusettes.  He has a `wife Jane Smith`, and a `daughter Jennifer Smith`.  He has been using an online pharmacy that provides an app titled `PlentyRX`, and uses a patient portal provided by `Together Health`.  The general relationships share some scoped set of data according to the `CMS Interoperability and Patient Access final rule`, for the period between 2005 and 2020 when he was living in Alabama.  

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
  }],
  "gender" : "male",
  "birthDate" : "1980-04-14"
}

// Releated Person - Jane Smith
{
  "id": "related-person-jane-smith-wife",
  "resourceType": "RelatedPerson",
  "name": [{
      "family": "SMITH",
      "given": "JANE"
  }],
  "gender" : "female",
  "birthDate" : "1981-01-10"
}

// Releated Person - Jennifer Smith
{
  "id": "related-person-jennifer-smith-daughter",
  "resourceType": "RelatedPerson",
  "name": [{
      "family": "SMITH",
      "given": "JENNIFER"
  }],
  "gender" : "female",
  "birthDate" : "2006-01-10"
}

// Plenty RX - Online Pharmacy
{
  "id": "org-plentyrx",
  "resourceType": "Organization",
  "name" : "Plenty Rx"
}

// Together Health - Patient Portal
{
  "id": "org-together-health",
  "resourceType": "Organization",
  "name" : "Together Health"
}
```


#### Step 3.3 - Create Records in Database via HTTP

Once the consent is constructed, it can be served up by an endpoint URL.

```bash
# create the necessary supporting records
PUT http://localhost:3000/baseR4/Patient/patient-john-smith  
PUT http://localhost:3000/baseR4/RelatedPerson/related-person-jane-smith-wife   
PUT http://localhost:3000/baseR4/RelatedPerson/related-person-jennifer-smith-daughter    
PUT http://localhost:3000/baseR4/Organization/org-plentyrx  
PUT http://localhost:3000/baseR4/Organization/org-together-health  

# fetch the supporting records
GET http://localhost:3000/baseR4/Patient/patient-john-smith  
GET http://localhost:3000/baseR4/RelatedPerson/related-person-jane-smith-wife   
GET http://localhost:3000/baseR4/RelatedPerson/related-person-jennifer-smith-daughter    
GET http://localhost:3000/baseR4/Organization/org-plentyrx  
GET http://localhost:3000/baseR4/Organization/org-together-health  
```


#### Step 4 - Define Relationships  

We continue by using the [ConsentExchangeRelations](./ConsentExchangeRelations.html) value set to define the relationship types.  

*Jane Smith - Wife*
```json
{
  "id": "related-person-jane-smith-wife",
  "resourceType": "RelatedPerson",
  "name": [{
      "family": "SMITH",
      "given": "JANE"
  }],
  "gender" : "female",
  "birthDate" : "1981-01-10",
  "patient" : {
    "reference" : "Patient/john-smith",
    "display" : "John Smith"
  },
  "relationship" : [{
    "coding" : [
      {
        "system" : "http://hl7.org/fhir/CodeSystem/relatedperson-relationshiptype",
        "code" : "WIFE",
        "display" : "wife"
      }
    ],
    "text" : "wife"
  },
  {
    "coding" : [
      {
        "system" : "http://hl7.org/fhir/CodeSystem/consent-exchange-relationships",
        "code" : "Spouse",
        "display" : "Spouse"
      }
    ],
    "text" : "Spouse"
  }]
}
```

*Jane Smith - Daughter*
```json
{
  "id": "related-person-jennifer-smith-daughter",
  "resourceType": "RelatedPerson",
  "name": [{
      "family": "SMITH",
      "given": "JENNIFER"
  }],
  "gender" : "female",
  "birthDate" : "2006-01-10",
  "patient" : {
    "reference" : "Patient/john-smith",
    "display" : "John Smith"
  },
  "relationship" : [
    {
      "coding" : [
        {
          "system" : "http://hl7.org/fhir/CodeSystem/relatedperson-relationshiptype",
          "code" : "DAU",
          "display" : "daughter"
        }
      ],
      "text" : "daughter"
    },
    {
      "coding" : [
        {
          "system" : "http://hl7.org/fhir/CodeSystem/consent-exchange-relationships",
          "code" : "Dependant",
          "display" : "Dependant"
        }
      ],
      "text" : "Dependant"
    }
  ]
}
```

#### Step 5 - Create Consent Resource  

Once created, these actors and organizations can be used to build consent resources. 

```json
{
  "resourceType" : "Consent",
  "id" : "12345",
  "status" : "active",
  "patient" : {
    "reference" : "Patient/patient-john-smith",
    "display" : "SMITH, JOHN"
  },
  "dateTime" : "2021-12-15",
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
      "start" : "2005-02-20",
      "end": "2020-06-01"
    },
    "provision": [{
      "type" : "permit",
      "period" : {
        "start" : "2005-02-20",
        "end": "2020-06-01"
      },
      "actor" : [
        {
          "role" : {
            "coding" : [
              {
                "system" : "http://hl7.org/fhir/CodeSystem/relatedperson-relationshiptype",
                "code" : "WIFE",
                "display" : "wife"
              }
            ],
            "text" : "wife"
          },
          "reference" : {
            "reference" : "RelatedPerson/related-person-jane-smith-wife",
            "display" : "Jane Smith"
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
    }, {
      "type" : "permit",
      "period" : {
        "start" : "2005-02-20",
        "end": "2020-06-01"
      },
      "actor" : [
        {
          "role" : {
            "coding" : [
              {
                "system" : "http://hl7.org/fhir/CodeSystem/relatedperson-relationshiptype",
                "code" : "DAU",
                "display" : "daughter"
              }
            ],
            "text" : "daughter"
          },
          "reference" : {
            "reference" : "RelatedPerson/related-person-jennifer-smith-daughter",
            "display" : "Jennifer Smith"
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
    }, {
      "type" : "permit",
      "period" : {
        "start" : "2005-02-20",
        "end": "2020-06-01"
      },
      "actor" : [
        {
          "role" : {
            "coding" : [
              {
                "system" : "http://terminology.hl7.org/CodeSystem/organization-type",
                "code" : "prov",
                "display" : "Healthcare Provider"
              }
            ],
            "text" : "Healthcare Provider"
          },
          "reference" : {
            "reference" : "Organization/org-plentyrx",
            "display" : "Plenty RX"
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
    }, {
      "type" : "permit",
      "period" : {
        "start" : "2005-02-20",
        "end": "2020-06-01"
      },
      "actor" : [
        {
          "role" : {
            "coding" : [
              {
                "system" : "http://terminology.hl7.org/CodeSystem/organization-type",
                "code" : "prov",
                "display" : "Healthcare Provider"
              }, {
                "system" : "http://dicom.nema.org/resources/ontology/DCM",
                "code" : "110150",
                "display" : "Application"
              }
            ],
            "text" : "Healthcare Provider"
          },
          "reference" : {
            "reference" : "Organization/org-together-health",
            "display" : "Together Health"
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
      ],
      "class": [{
          "system" : "http://hl7.org/fhir/resource-types",
          "code" : "MedicationRequest",
          "display" : "MedicationRequest"
        }, {
          "system" : "http://hl7.org/fhir/resource-types",
          "code" : "MedicationAdministration",
          "display" : "MedicationAdministration"
        }, {
          "system" : "http://hl7.org/fhir/resource-types",
          "code" : "MedicationDispense",
          "display" : "MedicationDispense"
        }, {
          "system" : "http://hl7.org/fhir/resource-types",
          "code" : "MedicationStatement",
          "display" : "MedicationStatement"
      }]
    }]
  }
}
```



#### Step 6 - Post Over HTTP   

Alternatively, you may wish to post the consent record elsewhere.  

```bash
# if the Organizations already exist on the FHIR server
# and you fill out the Consent resource correctly
# you can use POST to ask the server to assign a Consent.id value
POST http://localhost:3000/baseR4/Consent

# if you would like to specify the Organization ids explicitly
# you will need to use the PUT operation to register the resource with the desired id
# then fill out the Consent resource accordingly and PUT it to the server
PUT http://localhost:3000/baseR4/Organization/1
PUT http://localhost:3000/baseR4/Organization/2
PUT http://localhost:3000/baseR4/Consent/12345
```
