---
title: Store profiles in the FHIR service in Azure Healthcare APIs
description: This article describes how to store profiles in the FHIR service
author: ginalee-dotcom
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: reference
ms.date: 12/22/2021
ms.author: cavoeg
---

# Store profiles in the FHIR service

> [!IMPORTANT]
> Azure Healthcare APIs is currently in PREVIEW. The [Supplemental Terms of Use for Microsoft Azure Previews](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) include additional legal terms that apply to Azure features that are in beta, preview, or otherwise not yet released into general availability.

HL7 FHIR defines a standard and interoperable way to store and exchange healthcare data. Even within the base FHIR specification, it can be helpful to define other rules or extensions based on the context that FHIR is being used. For such context-specific uses of FHIR, **FHIR profiles** are used for the extra layer of specifications.
[FHIR profile](https://www.hl7.org/fhir/profiling.html) allows you to narrow down and customize resource definitions using constraints and extensions.

The FHIR service in the Azure Healthcare APIs (hereby called the FHIR service) allows validating resources against profiles to see if the resources conform to the profiles. This article guides you through the basics of FHIR profiles and how to store them. For more information about FHIR profiles outside of this article, visit [HL7.org](https://www.hl7.org/fhir/profiling.html).

## FHIR profile: the basics

A profile sets additional context on the resource that's represented as a `StructureDefinition` resource. A `StructureDefinition` defines a set of rules on the content of a resource or a data type, such as what elements a resource has and what values these elements can take.

Below are some examples of how profiles can modify the base resource:

- Restrict cardinality: For example, you can set the maximum cardinality on an element to 0, which means that the element is ruled out in the specific context.
- Restrict the contents of an element to a single fixed value.
- Define required extensions for the resource. 
 

A `StructureDefinition` is identified by its canonical URL: `http://hl7.org/fhir/StructureDefinition/{profile}` 

For example:

- `http://hl7.org/fhir/StructureDefinition/patient-birthPlace` is a base profile that requires information on the registered address of birth of the patient.
- `http://hl7.org/fhir/StructureDefinition/bmi` is another base profile that defines how to represent Body Mass Index (BMI) observations.
- `http://hl7.org/fhir/us/core/StructureDefinition/us-core-allergyintolerance` is a US Core profile that sets minimum expectations for `AllergyIntolerance` resource associated with a patient, and it identifies mandatory fields such as extensions and value sets.

When a resource conforms to a profile, the profile is specified inside the `profile` element of the resource.

```json
{
  "resourceType" : "Patient",
  "id" : "ExamplePatient1",
  "meta" : {
    "lastUpdated" : "2020-10-30T09:48:01.8512764-04:00",
    "source" : "Organization/PayerOrganizationExample1",
    "profile" : [
      "http://hl7.org/fhir/us/carin-bb/StructureDefinition/C4BB-Patient"
    ]
  },
```

> [!NOTE]
> Profiles must build on top of the base resource and cannot conflict with the base resource. For example, if an element has a cardinality of 1..1, the profile cannot make it optional.

Profiles are also specified by various Implementation Guides. Some common Implementation Guides are:

|Name |URL
|---- |----
Us Core |<https://www.hl7.org/fhir/us/core/>
CARIN Blue Button |<http://hl7.org/fhir/us/carin-bb/>
Da Vinci Payer Data Exchange |<http://hl7.org/fhir/us/davinci-pdex/>
Argonaut |<http://www.fhir.org/guides/argonaut/pd/>

## Accessing profiles and storing profiles

### Storing profiles

To store profiles to the FHIR server, you can `POST` the `StructureDefinition` with the profile content in the body of the request.


`POST http://<your FHIR service base URL>/StructureDefinition`

```
{ 
"resourceType" : "StructureDefinition",
"id" : "profile-id",
	…
}
```

For example, if you'd like to store the `us-core-allergyintolerance` profile, you'd do the following:

```rest
POST https://myworkspace-myfhirserver.fhir.azurehealthcareapis.com/StructureDefinition?url=http://hl7.org/fhir/us/core/StructureDefinition/us-core-allergyintolerance
```

```json
{
    "resourceType" : "StructureDefinition",
    "id" : "us-core-allergyintolerance",
    "text" : {
        "status" : "extensions"
    },
    "url" : "http://hl7.org/fhir/us/core/StructureDefinition/us-core-allergyintolerance",
    "version" : "3.1.1",
    "name" : "USCoreAllergyIntolerance",
    "title" : "US  Core AllergyIntolerance Profile",
    "status" : "active",
    "experimental" : false,
    "date" : "2020-06-29",
        "publisher" : "HL7 US Realm Steering Committee",
    "contact" : [
    {
      "telecom" : [
        {
          "system" : "url",
          "value" : "http://www.healthit.gov"
        }
      ]
    }
  ],
    "description" : "Defines constraints and extensions on the AllergyIntolerance resource for the minimal set of data to query and retrieve allergy information.",
```
For more examples, see the [US Core sample REST file](https://github.com/microsoft/fhir-server/blob/main/docs/rest/PayerDataExchange/USCore.http) on the open-source site that walks through storing US Core profiles. 

### Viewing profiles

You can access your existing custom profiles using a `GET` request, ``GET http://<your FHIR service base URL>/StructureDefinition?url={canonicalUrl}``, where `{canonicalUrl}` is the canonical URL of your profile.

For example, if you want to view US Core Goal resource profile:

`GET https://myworkspace-myfhirserver.fhir.azurehealthcareapis.com/StructureDefinition?url=http://hl7.org/fhir/us/core/StructureDefinition/us-core-goal`

This will return the `StructureDefinition` resource for US Core Goal profile, that will start like this:

```json
{
  "resourceType" : "StructureDefinition",
  "id" : "us-core-goal",
  "url" : "http://hl7.org/fhir/us/core/StructureDefinition/us-core-goal",
  "version" : "3.1.1",
  "name" : "USCoreGoalProfile",
  "title" : "US Core Goal Profile",
  "status" : "active",
  "experimental" : false,
  "date" : "2020-07-21",
  "publisher" : "HL7 US Realm Steering Committee",
  "contact" : [
    {
      "telecom" : [
        {
          "system" : "url",
          "value" : "http://www.healthit.gov"
        }
      ]
    }
  ],
  "description" : "Defines constraints and extensions on the Goal resource for the minimal set of data to query and retrieve a patient's goal(s).",

}
```
> [!NOTE]
> You'll only see the profiles that you've loaded into the FHIR service.


The FHIR service does not return `StructureDefinition` instances for the base profiles, but they can be found easily on the HL7 website, such as:

- `http://hl7.org/fhir/Observation.profile.json.html`
- `http://hl7.org/fhir/Patient.profile.json.html`


### Profiles in the capability statement

The `Capability Statement` lists all possible behaviors of your FHIR service. The FHIR service updates the capability statement with details of the stored profiles in the forms of:

- `CapabilityStatement.rest.resource.profile`
- `CapabilityStatement.rest.resource.supportedProfile`

For example, if you `POST` a US Core Patient profile, which starts like this:

```json
{
  "resourceType": "StructureDefinition",
  "id": "us-core-patient",
  "url": "http://hl7.org/fhir/us/core/StructureDefinition/us-core-patient",
  "version": "3.1.1",
  "name": "USCorePatientProfile",
  "title": "US Core Patient Profile",
  "status": "active",
  "experimental": false,
  "date": "2020-06-27",
  "publisher": "HL7 US Realm Steering Committee",
```

And send a `GET` request for your `metadata`:

`GET http://<your FHIR service base URL>/metadata`

You'll be returned with a `CapabilityStatement` that includes the following information on the US Core Patient profile you uploaded to your FHIR server:

```json
...
{
    "type": "Patient",
    "profile": "http://hl7.org/fhir/StructureDefinition/Patient",
    "supportedProfile":[
        "http://hl7.org/fhir/us/core/StructureDefinition/us-core-patient"
    ],
```
## Next steps

In this article, you've learned about FHIR profiles. Next, you'll learn how you can use $validate to ensure that resources conform to these profiles.

>[!div class="nextstepaction"]
>[Validate FHIR resources against profiles](validation-against-profiles.md)
