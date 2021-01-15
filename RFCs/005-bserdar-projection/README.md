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

The projection overlay specifies a source schema, an optional target
schema, and how target data objects are constructed using the fields
of the source objects:

{
   source: <schema reference>
   target: <schema reference>
   
   projection: { object_definition }
   
}




Object definition describes how a new object can be constructed from
the elements of the source object. It does this by first selecting the
parts of the source document into the current context, and then
assigning them to new data elements using rules:

object_definition: {
   context: { context_definition }
   rules: [ rule_definition, ...]
}

The `context_definition` part defines the data elements to be used in
the current and nested objects:

context_definition: {
   name: selector,
   ...
}

Here, `selector` selects an element of the source document using the
current context, and exposes that selected element under the new name
`name`.

As an example, consider the following source object segment:

```
{
  "entry": [
     {
        "resource": {
          ...
        }
     }
  ]
}

```

The following context selects the `resource` of the first entry and exposes that as
`rsc`:

```
context: {
   "rsc": "/entry/0/resource"
}
```

A richer, XPath-like selector format is possible:

```
context: {
  "rsc": "/entry/*/resource[resourceType='Patient']"
}
```
which will select the resource with `resourceType=Patient`.


Field selectors in rules and nested object_definitions can use "@rsc" to
refer to the selected object.


Rules section is a section of field definitions:

```
rules: [ rule_definition,...]
```

Each `rule_definition` can be used to define a mapped field, object field, or an
array field:

```
rule_definition : map_definition | object_definition | array_definition
```

A `map_definition` simply selects an element from the source document.

```
map_definition: {
  target: name,
  source: selector
}
```


{
  context: {
     name : selector
     ...
  },
  rules: [
    {
      target: name,
      source: context name
    },
    {
      target: name,
      object: <object definition>
    },
    {
      target: name,
      source: context name,
      items: <object definition>
    }
  ]
}


Consider the following example FHIR bundle containing a patient record:

{
    "entry": [
        {
            "fullUrl": "urn:uuid:83a77de9-dba0-4b41-be47-50e26e89d849",
            "resource": {
                "resourceType": "Patient",
                "address": [
                    {
                        "city": "Longmeadow",
                        "country": "US",
                         "line": [
                            "221 Schimmel Rapid",
                            "Suite 633"
                        ],
                        "postalCode": "01116",
                        "state": "MA"
                    }
                ],
                "birthDate": "1946-12-03",
                "communication": [
                    {
                        "language": {
                            "coding": [
                                {
                                    "code": "en-US",
                                    "display": "English (United States)",
                                    "system": "http://hl7.org/fhir/ValueSet/languages"
                                }
                            ]
                        }
                    }
                ],
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
        }
    ],
    "resourceType": "Bundle",
    "type": "collection"
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
