# Schema Base and Overlay Format
- Authors: Burak Serdar (bserdar@computer.org)
- Status: [PROPOSED]
- Status Note: (explanation of current status)
- Supersedes: 
- Start Date: 2021-01-29

## Summary

This RFC does three things:

  * It provides a JSON-LD context and structure for schema base,
  * It extends the schema base structure to deal with nested data
    objects, and
  * It specifies a common high-level JSON-LD structure for overlays.

## Motivation

The purpose of this RFC is to extend OCA to handle privacy flagging
and annotation of data that is not necessarily flat. Such data
structures are common for data exchange scenarios, in particular, FHIR
for health data exchange. FHIR is particularly challenging because of
its deeply nested and cyclic structure. With these extensions,
creating schema bases and overlays for FHIR-like standard schemas is
possible.

Existing OCA specification uses normalized attribute names for all
attributes of the schema base and overlays. Two problems with this
approach are: 1) A schema base is not a JSON-LD document, and 2) all
normalized attribute names must be predefined and registered.  This
approach restricts schema/overlay extensions, and custom overlay types
that can be used in cases other than data capture (for instance,
privacy classification of data, or ETL-like processing of existing
message types such as FHIR). This RFC defines a JSON-LD context and a
JSON schema for schema bases. Using these two, it is possible to
validate a schema base as a JSON document, and expand and interpret it
as a JSON-LD document.

Existing OCA overlays specify metadata or processing rules using
normalized attribute names. To support overlay-based processing of
nested data (such as FHIR objects), this RFC suggests supporting other
established ways of selecting attributes, such as JONPath, XPath,
FHIRPath, by using JSON-LD documents and multiple contexts for
overlays.

## Tutorial

### Schema Base

The existing schema base looks like this:

```
{
  "@context": "https://oca.tech/v1",
  "_100": "spec/schema_base/1.0",
  "_106": "GICS:35202010",
  "_107": "did:example:ebfeb1f712ebc6f1c276e12ec21",
  "_200": [
    "efxnizr39ifc4",
    "nfijh9i38ceSa",
    "Mceo097d72bi1",
    ...
  ],
  "_201": [
    "nfijh9i38ceSa",
     ...
  ]
}
```
This schema base can be written using the proposed format as follows:

```
{
  "@context": "http://schemas.cloudprivacylabs.com/SchemaBase",
  "@id": "http://someorg/someEntitySchema",
  "classification":"GICS:35202010",
  "issuedBy":"did:example:ebfeb1f712ebc6f1c276e12ec2",
  "issuerRole": "...",
  "purpose": "...",
  "attributes": [
    {
      "key": "efxnizr39ifc4"
    },
    {
      "key": "nfijh9i38ceSa",
      "flag": "https://someOntology/PII"
    },
    ...
  ],
  
}

```

The proposed schema base is a proper JSON-LD document that can be
expanded and processed using existing JSON-LD tools. It can refer to
other schema bases, and optional extensions can be added using
additional contexts. Here are the differences:

### @context

The @context identifies a JSON-LD context that defines the nested
structure of attributes. The context is defined as:

```
{
    "@context": {
        "classification": {
            "@id": "http://schemas.cloudprivacylabs.com/SchemaBase/classification"
        },
        "issuedBy": {
            "@id": "http://schemas.cloudprivacylabs.com/SchemaBase/issuedBy"
        },
        "issuerRole": {
            "@id": "http://schemas.cloudprivacylabs.com/SchemaBase/issuerRole"
        },
        "purpose": {
            "@id": "http://schemas.cloudprivacylabs.com/SchemaBase/purpose"
        },
        "attributes": {
            "@id":"http://schemas.cloudprivacylabs.com/SchemaBase/attributes",
            "@context": {
                "key": "http://schemas.cloudprivacylabs.com/SchemaBase/attributeKey",
                "flag": "@id",
                "reference": "@id",
                "items":{
                    "@id": "http://schemas.cloudprivacylabs.com/SchemaBase/items"
                },
                "allOf": {
                    "@id": "http://schemas.cloudprivacylabs.com/SchemaBase/allOf",
                    "@container": "@list"
                },
                "oneOf": {
                    "@id": "http://schemas.cloudprivacylabs.com/SchemaBase/oneOf",
                    "@container": "@list"
                }
            }
        }
    }
}
```

### @id

This defines the id for the schema base. Other schema bases can use
this @id to refer to this schema base.

### classification, issuedBy, issuerRole, purpose

Schema metadata fields.

### attributes

This part defines the attributes of the schema, and optionally flags
them using additional term references. A simple value is represented as:

```
{
  "key": "<key_value>"
}
```

The `key_value` is the value assigned to this attribute by the schema
author. Localized names can be given to this key using an index overlay.

Example:

The following defines a "name" attribute:
```
{
  "key": "name"
}
```
Using an index overlay, a separate value can be assigned to the key "name".

All key values are unique within the `attributes` section they are
defined. Same key values can be used to refer to different attributes
in nested fields. That is:

```
"attributes": [
  {
    "key": "name"
  },
  {
    "key": "obj",
    "attributes": [
       {
         "key": "name"
       }
    ]
  }
]
```

The above schema defines the following JSON document:

```
{
  "name": "...",
  "obj": {
    "name": "..."
  }
}
```

The `name` and `obj.name` refer to two different attributes.

An attribute can be flagged:

```
    {
      "key": "nfijh9i38ceSa",
      "flag": "https://someOntology/PII"
    }
```

For instance, this can be used to flag PII information based on BIT.

An attribute can be a reference to another schema base:

```
{
   "key": "patient",
   "reference": "http://someSite/Patient"
}
```

The above defines the field "patient" to be a "Patient" object, whose
schema base is given in the `reference` value.

An attribute can be a nested object:

```
{
   "key": "nestedObject",
   "attributes": [
      {
        "key": "nestedAttribute"
      },
      ...
   ]
}
```

An attribute can be an array whose items can be described in the schema base:

```
{
  "key": "valueArray",
  "items": {}
}
```

Instance:

```
"valueArray": [ "value1", "value2", ... ]
```

```
{
  "key": "objectArray",
  "items": {
     "attributes": [
        {
          "key": "key1"
        }
     ]
  }
}
```

Instance:

```
"objectArray": [ 
  { "key1": "value1" },
  { "key2": "value2" }
]
```

An attribute can be the composition of multiple objects:

```
{
  "key": "p1",
  "allOf": [
    {
      "reference": "http://someObject"
    },
    {
      "attributes": [
        {
           "key": "attr"
        }
      ]
    }
  ]
}
```

The above construct creates the `p1` attribute by using all
attributes of `http://someobject` and `attr`.

An attribute can be a polymorphic value:

```
{
   "key": "p2",
   "oneOf": [
     {
      "reference": "http://obj1"
     },
     {
      "reference": "http"//obj2"
     }
    ]
}
```

This describes the `p2` attribute as either `http://obj1` or
`http://obj2`.

### Overlays

The suggested overlay format is as follows:

```
{
  "@context": [ <schema_base_context>, <base_overlay_context>, <specific_overlay_context> ],
  "@id": "http://overlayId",
  <overlay specific attributes>
  <attributes specification>
```

This format uses JSON-LD contexts and terminology to refer to
capabilities, algorithms, metadata, etc. Because of this, there is no
need to include an overlay type. 

As an example, the following index overlay uses `attributes` to define
name for schema base keys following the same structure are the schema base:

``` 
{
  "@context": [
    "http://schemas.cloudprivacylabs.com/SchemaBase",
    "http://schemas.cloudprivacylabs.com/Overlay",
    "http://schemas.cloudprivacylabs.com/IndexOverlay"
  ],
  "base": "<schema base  id>,
  "attributes": [
    {
      "key": "id_key1",
      "attributeName": "name_Key1"
    },
    {
      "attributes": [
        {
          "key": "id_key2",
          "attributeName": "name_Key2"
        }
      ]
    }
  ]
}
```

The `attributes` term comes from the `SchemaBase` context, and thus
defines a nested structure matching the schema base. The `attributeName` term
comes from the `IndexOverlay` context and defines the name for the key.

The same index overlay can be defined as follows:

```
{
  "@context": [
    "http://schemas.cloudprivacylabs.com/Overlay",
    "http://schemas.cloudprivacylabs.com/IndexOverlay"
  ],
  "base": "<schema base id>",
  "paths": [
    {
      "key": "id_key1",
      "attributeName": "name_Key1"
    },
    {
      "key": "id_key1.id_key2",
      "attributeName": "name_Key2"
    }
  ]
}

```

This overlay uses `paths` from the `Overlay` context that uses
dot-notation to address schema base attributes.

The use of multiple contexts allow defining composite overlays. For
example, using the `IndexOverlay` and `EncodingOverlay` together a
single overlay can specify both the names and encodings for the
attributes of the schema base.


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
