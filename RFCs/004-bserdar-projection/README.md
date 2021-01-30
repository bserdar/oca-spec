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

The need for projection overlay arose in the context of generating a
vaccination certificate from FHIR messages. The projection overlay can
specify how to build most, if not all of the fields necessary to build
such a certificate that later can be signed to produce a VC. By
publishing a set of projection overlays from different source schemas
to a target schema, it becomes possible to formally prescribe how to
translate different formats used in the field to the required input.

Projection overlay is also needed in data exchange use-cases
where the input data schema is different from the exchange data
schema, which can also be different from the storage schema. 

The projection overlay defines a semantic association between the
fields of two schemas as well as a method for unidirectional
transformations.  It may be possible that some use cases may require
additional application-specific processing to implement a usable
mapping. 

The current OCA specification already contains a subset overlay and a
mapping overlay that work similarly. The addition of projection
overlay would make the subset and mapping overlays obsolete.

## Tutorial

A simple projection overlay simply selects attributes from the source and
projects them into the target:

```
projection: {
  "_target_id": "_source_id",
  "_target_id": "_source_id..."
  ...
}

```

If the target object is an array, of values, the source object should
also select an array of values. This RFC does not handle creating an
array of objects at this point.

It should be possible to use a richer selection method instead of
attribute ids. For example, processing JSON messages (FHIR, etc.) may
require selecting certain parts of the input based on criteria. To
support such cases, a `dialect` attribute can be used:

``` 
"dialect": "FHIRPath",
"projection" : {
  "_target_id": <fhirPath>,
  "_target_id": <fhirPath>,
  ...
}
```

Possible dialects are 
  
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
               "status": "completed",
               "vaccineCode": {
               "coding": [
                  {
                   "system": ""
                   "code": "
                  }
              ],
           },
          "occurrenceDateTime": "2013-01-10",
          "primarySource": true,
          "location": {
             "reference": "Location/1"
           },
          "manufacturer": {
             "reference": "Organization/hl7"
          },
          "lotNumber": "AAJN11K",
          "expirationDate": "2015-02-15",
          "site": {
            "coding": [
               {
                "system": "http://terminology.hl7.org/CodeSystem/v3-ActSite",
                "code": "LA",
                "display": "left arm"
               }
            ]
         },
        "route": {
           "coding": [
              {
               "system": "http://terminology.hl7.org/CodeSystem/v3-RouteOfAdministration",
               "code": "IM",
               "display": "Injection, intramuscular"
             }
           ]
        },
       "doseQuantity": {
            "value": 5,
            "system": "http://unitsofmeasure.org",
            "code": "mg"
       },
        ...
      }
   ],
    "resourceType": "Bundle",
    "type": "collection"
}
```

The projection overlay specifies a field-by-field mapping to construct
the target data object

```
sourceSchema: <link to FHIR Bundle schema>
targetSchema: <link to VC schema>
dialect: jsonpath
projection: {
 recip_id : /entry/[?(@.resource.resourceType='Patient')]/id,
 recip_first_name_id: /entry/[?(@.resource.resourceType='Patient')]/name[0]/given[0]
 recip_last_name_id: /entry/[?(@.resource.resourceType='Patient')]/name[0]/family
 recip_dob_id: /entry/[?(@.resource.resourceType='Patient')]/birthDate
 vax_route_id: /entry/[?(@.resource.resourceType='Immunization' and @.resource.vaccineCode.coding.code='<vaccinde code>')]/route/coding[?(@.system="http://terminology.hl7.org/CodeSystem/v3-RouteOfAdministration")]/display
 cvx_id: /entry/[?(@.resource.resourceType='Immunization' and @.resource.vaccineCode.coding.code='<vaccinde code>')]/vaccineCode/coding/[?(@.system='http://hl7.org/fhir/sid/cvx')]/code
 ndc_id: /entry/[?(@.resource.resourceType='Immunization' and @.resource.vaccineCode.coding.code='<vaccinde code>')]/vaccineCode/coding/[?(@.system='http://hl7.org/fhir/sid/ndc)]/code
     ...
}
```

The output would be:

``` 
{
  "recip_id": "83a77de9-dba0-4b41-be47-50e26e89d849",
  "recip_first_name_id": "Benito349",
  "recip_last_name_id": "Reilly95",
  "recip_dob_id": "1946-12-03",
  "vax_route_id": "Injection, intramuscular",
  "cvx_id": null,
  "ndc_id": null
  ...
}
```

The output of the projection overlay contains all the attributes that
can be populated from the source. The last two fields are null,
because the input document does not contain that information. Other
processing such as user entry may be required to fill the missing
information. However if the input contains all the necessary data, the
output can be used without further processing. 



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
