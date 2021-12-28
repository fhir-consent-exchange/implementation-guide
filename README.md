# mehi-consent-exchange

Consent Exchange - Payor to Payor data exchange, with implementation guide and reference implementation.  

### Compiling Documentation  

```bash 
# install the sushi tool
git clone https://github.com/FHIR/sushi
npm install -g fsh-sushi
sushi --help

# compile the documentation  
cd output
./_genonce.sh

# open the documentation (assuming Mac + Chrome)
cd output
open -a "Google Chrome" index.html
```


#### Reference Implementation    

```bash
# install meteor  
npm install -g meteor

# clone the node-on-fhir base template
git clone https://github.com/symptomatic/node-on-fhir

# clone the reference implementation repository
git clone https://gitlab.mitre.org/mehi/mehi-consent-exchange  

# copy the reference implementation package into the base template
cp -R mehi-consent-exchange/reference-implementation/pkg-consent-exchange node-on-fhir/packages/consent-exchange

# install dependencies
cd node-on-fhir
npm install

# install reference implementation package
meteor add mitre:mehi-consent-engine clinical:vault-server-freemium

# install data import/export packages (optional)  
meteor add symptomatic:data-importer symptomatic:data-relay 

# run the application
meteor run --settings packages/consent-exchange/configs/settings.mehi.json

# run the application in debugging mode
DEBUG=true TRACE=true meteor run --settings packages/consent-exchange/configs/settings.mehi.json

# open the application in a browser
open -a "Google Chrome" http://localhost:3000 
```

Once running, you can use the data importer functionality to load in the example data from the `data` directory.  


#### API Usage  

Use a utility like `curl` or [Postman](https://www.postman.com/) to query the Reference Implementation at the API endpoints defined in the Implementation Guide.  

```bash
# server metadata and capability statement
curl http://localhost:3000/baseR4/metadata

# consent exchange endpoints
curl http://localhost:3000/baseR4/Organization
curl http://localhost:3000/baseR4/Patient
curl http://localhost:3000/baseR4/Consent
curl http://localhost:3000/baseR4/RelatedPerson
curl http://localhost:3000/baseR4/CareTeam

```



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
