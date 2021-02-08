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

The purpose of the projection overlay is to provide a method for a
governing entity to publish a prescriptive method for constructing
instances of a base schema from source data. The governing entity can
publish multiple projection overlays to support different variations
of the input (for instance, different FHIR profiles).

The need for projection overlay arose in the context of generating a
vaccination certificate from FHIR resources. The projection overlay
can prescribe how to build most, if not all of the fields necessary to
produce a VC from a FHIR input.

Projection overlay can also be used in data exchange use-cases where
the input data schema is different from the exchange data schema, or
in ETL use cases to perform data transformations.

The current OCA specification already contains a subset overlay and a
mapping overlay that work similarly. The addition of projection
overlay would make the subset and mapping overlays obsolete.

## Tutorial


A projection overlay selects attributes from the source and projects
them into the target:

```
{
  "@context": [
    "http://schemas.cloudprivacylabs.com/BaseSchema",
    "http://schemas.cloudprivacylabs.com/Overlay",
    "http://schemas.cloudprivacylabs.com/ProjectionOverlay"],
  "sourceSchema": "http://sourceSchema",
  "targetSchema": "http://targetSchema",
  "attributes": [
  {
     "sourcePath": "source_id",
     "key": "target_id"
  },
  {
     "const": <value>,
     "key": "target_id"
  },
  ...
  ]
}

```

The `attributes` (from `BaseSchema`) specifies a nested attribute tree
that matches the target schema, so this structure desribes how to build
each attribute of the target schema. The term `sourcePath` selects an
attribute from the source schema using dot-notation. The term `const`
inserts a constant value to the target.

It is possible to specify other attribute selection methods such as
JSONPath, XPath, etc. An overlay processor capable of processing such
methods should publish a context . For
example, an overlay processor with JSONPath capabilities can support
an overlay:

```
{
  "@context": [
    "http://schemas.cloudprivacylabs.com/BaseSchema",
    "http://schemas.cloudprivacylabs.com/Overlay",
    "http://schemas.cloudprivacylabs.com/ProjectionOverlay",
    "http://myContext/JSONPathSupport"],
  "sourceSchema": "http://sourceSchema",
  "targetSchema": "http://targetSchema",
  "attributes": [
  {
     "jsonPath": <jsonpath expression>,
     "key": "target_id"
  },
  ...
  ]
}

```

Possible options are:

  * JSON Pointer: https://tools.ietf.org/html/rfc6901
  * XPath (for XML documents): https://www.w3.org/TR/xpath/
  * JSON Path (XPath for json): https://goessner.net/articles/JsonPath/
  * FHIRPath: http://hl7.org/fhirpath/N1/


Suppose the target schema is a vaccination certificate of the form
given below, and the purpose is to build this certificate from a FHIR
bundle obtained from an EHR.

```
{
  recip_id
  recip_first_name
  recip_middle_name
  recip_last_name
  recip_dob
  ...
  cvx
  ndc
  mvx
  vax_expiration
  ...
}
```

Using the following sample input:

```
{
    "resourceType": "Bundle",
    "id": "DHIR",
    "meta": {
        "versionId": "1.1.5",
        "lastUpdated": "2020-12-07"
    },
    "type": "collection",
    "entry": [
        {
            "resourceType": "Immunization",
            "id": "Immunization01",
            "meta":{
                "lastUpdated":"2017-07-25T15:43:54.271-05:00"
            },
            "extension": [
                {
                    "url": "[base-structure]/ca-on-immunizations-extension-public-health-unit",
                    "valueString": "Toronto PHU"
                }
            ],
            "status": "completed",
            "vaccineCode": {
                "coding": [
                    {
                        "system": "http://snomed.info/sct",
                        "code": "61153008",
                        "display": "MMR"
                    },
                    {
                        "system": "http://snomed.info/sct",
                        "code": "7171000087106",
                        "display": "MMR Priorix GSK"
                    }
                ]
            },
            "patient": {
                "reference": "Patient/Patient1234"
            },
            "occurrenceDateTime": "2016-02-14T10:22:00-05:00",
            "primarySource": true,
            "lotNumber": "Some Lot",
            "performer": [
                {
                    "function": {
                        "coding": [
                            {
                                "system": "http://terminology.hl7.org/CodeSystem/v2-0443",
                                "code": "AP",
                                "display": "Administering Provider"
                            }
                        ]
                    },
                    "actor": {
                        "reference": "Practitioner/Practitioner1234"
                    }
                }]
        },
        {
            "resourceType": "Patient",
            "id": "Patient1234",
            "identifier": [
                {
                    "system": "[id-system-local-base]/ca-on-panorama-immunization-id",
                    "value": "95ZWBKWTC5"
                },
                {
                    "system": "[id-system-global-base]/ca-on-patient-hcn",
                    "value": "9393881587"
                }
            ],
            "name": [
                {
                    "family": "Doe",
                    "given": [
                        "John",
                        "W."
                    ]
                }
            ],
            "gender": "male",
            "birthDate": "2012-02-14"
        },
        {
            "resourceType": "Practitioner",
            "id": "Practitioner1234",
            "name": [
                {
                    "family": "Nurse",
                    "given": [
                        "Best"
                    ]
                }
            ],
            "qualification": [
                {
                    "code": {
                        "coding": [
                            {
                                "system": "[code-system-local-base]/ca-on-immunizations-practitionerdesignation",
                                "code": "RN",
                                "display": "Registered Nurse"
                            }
                        ]
                    }
                }
            ]
        },
        {
            "resourceType": "ImmunizationRecommendation",
            "id": "ImmunizationRecommendation01",
            "patient": {
                "reference": "Patient/Patient1234"
            },
            "date": "2016-07-28T11:04:15.817-05:00",
            "recommendation": [
                {
                    "targetDisease": {
                        "coding": [
                            {
                                "system": "http://snomed.info/sct",
                                "code": "36989005",
                                "display": "Mumps"
                            }
                        ]
                    },
                    "forecastStatus": {
                        "coding": [
                            {
                                "system": " http://snomed.info/sct",
                                "code": "8191000087109",
                                "display": "Overdue"
                            }
                        ]
                    },
                    "dateCriterion": [
                        {
                            "code": {
                                "coding": [
                                    {
                                        "system": " http://loinc.org ",
                                        "code": "30980-7",
                                        "display": "Date vaccine due"
                                    }
                                ]
                            },
                            "value": "2016-06-01"
                        }
                    ]
                },
                {
                    "targetDisease": {
                        "coding": [
                            {
                                "system": "http://snomed.info/sct",
                                "code": "14189004",
                                "display": "Measles"
                            }
                        ]
                    },
                    "forecastStatus": {
                        "coding": [
                            {
                                "system": "http://snomed.info/sct",
                                "code": "8191000087109",
                                "display": "Overdue"
                            }
                        ]
                    },
                    "dateCriterion": [
                        {
                            "code": {
                                "coding": [
                                    {
                                        "system": " http://loinc.org ",
                                        "code": "30980-7",
                                        "display": "Date vaccine due"
                                    }
                                ]
                            },
                            "value": "2016-06-01"
                        }
                    ]
                }
            ]
        } 
    ]
}

```

The projection overlay specifies a field-by-field mapping to construct
the target data object

```
{
  "@context": [
    "http://schemas.cloudprivacylabs.com/BaseSchema",
    "http://schemas.cloudprivacylabs.com/Overlay",
    "http://schemas.cloudprivacylabs.com/ProjectionOverlay"],
  "sourceSchema": "http://fhirSchema",
  "targetSchema": "http://targetSchema",
  "attributes": [
        {
            "const": "http://projection-test",
            "key": "@vocab"
        },
        {
            "jsonPath": "$.entry[?(@.resourceType==\"Immunization\")].id",
            "key": "vax_event_id"
        },
        {
            "jsonPath": "$.entry[?(@.resourceType==\"Patient\")].name[0].given[0]",
            "key": "recip_first_name"
        },
        {
            "jsonPath": "$.entry[?(@.resourceType==\"Patient\")].name[0].given[1]",
            "key": "recip_middle_name"
        },
     ...
}
```

The output would be:

``` 

  "@vocab": "http://projection-test",
  "admin_cvx": [
    "61153008",
    "7171000087106"
  ],
  "admin_date": "2016-02-14T10:22:00-05:00",
  "admin_name": [
    {
      "family": "Nurse",
      "given": [
        "Best"
      ]
    }
  ],
  "lot_number": "Some Lot",
  "recip_dob": "2012-02-14",
  "recip_first_name": "John",
  "recip_last_name": "Doe",
  "recip_middle_name": "W.",
  "recip_sex": "male",
  "vax_event_id": "Immunization01"
}
```

A prototype implementation for this transformation is available here: 

https://github.com/bserdar/oca-projection-prototype


Explain the proposal as if it were already implemented and you were
teaching it to another OCA consumer. That generally means:

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
