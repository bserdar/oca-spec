# Projection Overlays
- Authors: Burak Serdar (bserdar@computer.org)
- Status: PROPOSED
- Status Note: (explanation of current status)
- Supersedes: (link to anything this RFC supersedes)
- Start Date: 2021/01/13

## Summary

This is a proposal for a Projection Overlay that defines a mapping
between a source schema and a target schema. This is a structural
mapping associating each field in the source to zero or more fields in
the target.

## Motivation

Projection overlay is especially needed in data exchange use-cases
where the input data schema is different from the exchange data
schema, which can also be different from the storage schema. The
projection overlay defines a semantic association between the fields
of two schemas as well as a method for unidirectional transformations.
It may be possible that some use cases may require additional
application-specific processing to implement a usable
mapping. Projection overlay is also useful in data procesing pipelines
to build data objects by selecting parts of the input, for instance,
for generating cryptographic hashes, or VCs.

The current OCA specification already contains a subset overlay and a
mapping overlay that work similarly. The addition of projection
overlay would make the subset and mapping overlays obsolete.


## Tutorial

We will work through an example where the input to the system is a
FHIR bundle containing patient information and immunization records,
and the output is a document that will be used as claims for VC
generation. Suppose the target schema is:

```
{
  holderID
  holderGivenName
  holderFamilyName
  holderBirthDate
  vaccinationEvents: [
    {
      vaccineCode
      vaccineManufacturer
      vaccineLotNumber
      totalDosesRequired
      vaccinationDate
      doseNumber
      practitionerID
      facilityID
    },
    ...
  ]
}
```


The sample input is the following entry from Synthea synthetic patient
record (edited):

```
{
    "entry": [
        {
            "resource": {
                "resourceType": "Patient",
                "birthDate": "1946-12-03",
                "gender": "male",
                "id": "83a77de9-dba0-4b41-be47-50e26e89d849",
                "name": [
                    {
                        "family": "Reilly95",
                        "given": [
                            "Benito349"
                        ],
                        "prefix": [
                            "Mr."
                        ],
                        "use": "official"
                    }
                ]
            }
        },
        {
           "resource": {
             "resourceType": "Immunization",
              "occurenceDatetime": "2008-07-09T12:31:52-04:00",
              "encounter": {
               "reference": "urn:uuid:95845705-f125-40d5-ada8-baf4d94a10ea"
              },
             "status": "completed",
             "vaccineCode": {
               "coding": [
                 {
                   "code": "140",
                   "display": "Influenza, seasonal, injectable, preservative free",
                   "system": "http://hl7.org/fhir/sid/cvx"
                 }
              ],
            }
          }
       },
    ],
    "resourceType": "Bundle",
    "type": "collection"
}
```

The projection overlay specifies a field-by-field mapping to construct
the target data object

```
sourceSchema: <link to FHIR schema>
targetSchema: <link to VC schema>
selectorDialect: jsonpath

object: {
  scope: {
     patient: /entry/[?(@.resource.resourceType='patient')]
  },
  properties: {
     holderID : {
          context: patient
          source: /id  (83a77de9-dba0-4b41-be47-50e26e89d849)
     },
     holderGivenName: {
         context: patient
         source: /name[0]/given[0]   (Benito349)
     },
     holderFamilyName: {
         context: patient
         source: /name[0]/family   (Reilly95)
     },
     holderBirthDate: {
         context: patient
         source: /birthDate   (1946-12-03)
     },
     vaccinationEvents: {
       array: {
          scope: {
            immunization: /entry/[?(@.resource.resourceType='Immunization' and @.resource.vaccineCode='code')]
          },
          items: {
        item: {
          context: immunization
          items: @
        },
        object: {
          properties: {
            vaccinationDate: {
               source: /occurenceDateTime  (2008-07-09T12:31:52-04:00)
            },
            doseNumber: {
               source: /
      },
      ...
    }
  }
}

```

The output would be:
holder: {
  ID: 83a77de9-dba0-4b41-be47-50e26e89d849
  GivenName: Benito34
  FamilyName: Reilly95
  BirthDate: 1946-12-03
}
  vaccine: {
     object: {
       scope: {
         immunization: /entry/[?(@.resource.resourceType='Immunization' and @.resource.vaccineCode='code')][0]
       },
       properties: {
         vaccineCode: {
           context: immunization
           source: /vaccineCode/coding[?(@.code='code' and @.system='system')] (140)
         },
         totalDosesRequired: {
           context: immunization,
           source: /protocolApplied/seriesDoses/seriesDosesPositiveInt (null)
         }
       }
    }
 },
 vaccinationEvent: {
    array: {
      scope: {
         immunization: /entry/[?(@.resource.resourceType='Immunization' and @.resource.vaccineCode='code')]
      },
      items: {
        item: {
          context: immunization
          items: @
        },
        object: {
          properties: {
            vaccinationDate: {
               source: /occurenceDateTime  (2008-07-09T12:31:52-04:00)
            },
            doseNumber: {
               source: /
      },
      ...
    }
  }
}




Explain the proposal as if it were already implemented and you
were teaching it to another OCA consumer. That generally
means:

- Introducing new named concepts.
- Explaining the feature largely in terms of examples.
- Explaining how OCA consumers should *think* about the
feature, and how it should impact the way they use the ecosystem.
- If applicable, provide sample error messages, deprecation warnings, or
migration guidance.

Some enhancement proposals may be more aimed at contributors (e.g. for
consensus internals); others may be more aimed at consumers.

## Reference

Provide guidance for implementers, procedures to inform testing,
interface definitions, formal function prototypes, error codes,
diagrams, and other technical details that might be looked up.
Strive to guarantee that:

- Interactions with other features are clear.
- Implementation trajectory is well defined.
- Corner cases are dissected by example.

## Drawbacks

Why should we *not* do this?

## Rationale and alternatives

- Why is this design the best in the space of possible designs?
- What other designs have been considered and what is the rationale for not
choosing them?
- What is the impact of not doing this?

## Prior art

Discuss prior art, both the good and the bad, in relation to this proposal.
A few examples of what this can include are:

- Does this feature exist in other ecosystems and what experience have
their community had?
- For other teams: What lessons can we learn from other attempts?
- Papers: Are there any published papers or great posts that discuss this?
If you have some relevant papers to refer to, this can serve as a more detailed
theoretical background.

This section is intended to encourage you as an author to think about the
lessons from other implementers, provide readers of your proposal with a
fuller picture. If there is no prior art, that is fine - your ideas are
interesting to us whether they are brand new or if they are an adaptation
from other communities.

Note that while precedent set by other communities is some motivation, it
does not on its own motivate an enhancement proposal here.

## Unresolved questions

- What parts of the design do you expect to resolve through the
enhancement proposal process before this gets merged?
- What parts of the design do you expect to resolve through the
implementation of this feature before stabilization?
- What related issues do you consider out of scope for this
proposal that could be addressed in the future independently of the
solution that comes out of this doc?

## Implementations

The following lists the implementations (if any) of this RFC. Please do a pull request to add your implementation. If the implementation is open source, include a link to the repo or to the implementation within the repo.

*Implementation Notes* [may need to include a link to test results](README.md#accepted).

Name / Link | Implementation Notes
--- | ---
 |
