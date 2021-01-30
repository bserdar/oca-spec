# Base Schema and Overlay Format
- Authors: Burak Serdar (bserdar@computer.org)
- Status: [PROPOSED]
- Status Note: (explanation of current status)
- Supersedes: 
- Start Date: 2021-01-29

## Summary

This RFC does three things:

  * It limits the usage of normalized attribute names to the attributes of the
    schema instance,
  * It extends the base schema structure to deal with nested data objects, and
  * It adds an optional selectorDialect attribute to overlays that
    select the attributes of the base schema using JSONPath, JSON Pointer, XPath, etc.

## Motivation

The extensions proposed in this RFC were required to decompose the
FHIR schema into OCA layers. It is not feasible to flatten the FHIR
specification to enable OCA support, however by the addition of these
features, it becomes possible to generate OCA-compliant base schemas
and overlays, which then enables overlay-based processing of FHIR
messages.

Existing OCA specification uses normalized attribute names for all
attributes of the base schema and overlays. This requires that all
normalized attribute names must be predefined and registered. This
approach restricts schema/overlay extensions, and custom overlay types
that can be used in cases other than data capture (for instance,
privacy classification of data, or ETL-like processing of existing
message types such as FHIR). This RFC recommends using the normalized
attribute names only for the attributes of the data object being
defined by the schema.

Existing OCA specification has flat base schemas. This does not
support nested objects such as JSON. This RFC adds direct support for
nested objects. 

Currently overlays specify metadata or processing rules using
normalized attribute names. To support overlay-based processing of
nested data (such as FHIR objects), selecting attributes by name is
not sufficient. This RFC suggests using other established ways of 
selecting attributes, such as JONPath, XPath, FHIRPath, etc.

## Tutorial

### Base Schema

The suggested base-schema format is as follows:

```
{
  "@context": <url>,
  "id" : <schema id>,
  "classification": "",
  "issuedBy": "",
  "issuerRole": "",
  "purpose": "",
  "objectType": "",
  "attributes": {
    "normalized_id_value": { 
      "type": "bytes"
    },
    "normalized_id_object": {
      "type": "object",
      "attributes: {
        ...
      }
    },
    "normalized_id_array_of_values": {
      "type": "array",
      "items": {
        "type": "bytes" 
      }
    },
    "normalized_id_array_of_objects": {
      "type": "array",
      "items": {
        "type": "object",
        "attributes": {
          ...
        }
      }
    },
    "normalized_id_reference": {
      "type": "reference",
      "ref": <schema id>
    },
    "normalized_id_allof": {
      "allOf": [
         {
           "type": "reference",
           "ref": <schema id>
         },
         {
            "type": "object",
            "attributes": {...}
         },
         ...
       ]
     },
    "normalized_id_oneof": {
      "oneOf": [
         {
           "type": "reference",
           "ref": <schema id>
         },
         {
            "type": "object",
            "attributes": {...}
         },
         ...
       ]
     }
  },
  "attributeFlagging": [ "normalized_id",...]
}
  
```

The schema header is the same as the existing OCA specification with
the difference that it uses English keys instead of normalized
names. Because of this, additional information (such as metadata about
the schema base) can be included without a need to register these keys
with a central registry, or defining them in an index overlay.

`attributes` is a JSON object instead of a JSON array. 

Existing OCA base schema:
```
"_attributes_id": [
  "_id1",
  "_id2",
  ...
]
```

Proposed OCA base schema:
```
"attributes": {
  "_id1": { 
    "type": "bytes"
  },
  "_id2": {
    "type": "bytes"
  },
  ...
}
```

The `type: bytes` simply marks the attribute as a sequence of bytes
that should be interpreted based on the encoding and type information
provided in other overlays.

The proposed structure also supports defining nested objects, arrays,
references to other schemas, combinations of entities (using `allOf`)
and polymorphism (using `oneOf`). These features match their JSON 
equivalents.

#### Nested Objects

A nested object is defined by listing its attributes:

```
"_id": {
  "type": "object",
  "attributes": {
    "normalized_id": <type defn>,
    ...
  }
}
```

#### Arrays

An array is defined by specifying its item type:

```
{
  "type": "array",
  "items": {
    <type defn> 
  }
}
```

Above, the `type defn` can be a value (`bytes)`, object, array, reference, etc.

#### References

A reference defines a reference to another base schema:

```
{
  "type": "reference",
  "ref": <schema id>
}
```

#### Composition

An object can extend other objects using composition:

```
{
  "allOf": [
     <type defn>.
     <type defn>,
     ...
  ]
}
```

The resulting object contains the attributes of all its components.

#### Polymorphism

An object can be one of the given objects:

```
{
  "oneOf": [
     <type defn>,
     <type defn>
  ]
}
```

For example, a FHIR bundle contains an array of entries, where each
entry can be a Patient, Encounter, etc.

A JSON schema representation of this proposal is at: 

https://github.com/bserdar/ocaschemas/blob/main/json/base.schema.json


### Overlays

The suggested overlay format is as follows:

```
{
  "@context": <url>,
  "overlayType": <type>,
  <overlay specific attributes>
  selectorDialect: <dialect>
  <overlay spec>
```

where `overlay spec` is:

```
{ 
   <selector> : <rule>
   ...
}
```

The `selectorDialect` is an optional property specifying how the base
schema attributes are selected. If it is not specified, the base
schema attributes are selected using their normalized ids. For example:

```
{
  "@context": <url>
  "overlayType": "encoding",
  "attributes": {
    "_id1": "utf8",
    "_id2": "utf8"
    ...
   }
}
```

The `selectorDialect` can specify another method to select
attributes. The following example uses an XPath to select fields of
the base schema and applies overlay specific rules:

```
{
  "@context": <url>
  "overlayType": "projection",
  "selectorDialect": "xpath",
  "attributes": {
     "/_id1/_id2": <rule1>,
     "//_id3": <rule2>,
     ...
  }
}
```



## Reference

Provide guidance for implementers, procedures to inform testing,
interface definitions, formal function prototypes, error codes,
diagrams, and other technical details that might be looked up.  Strive
to guarantee that:

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
