# JSON-LD Processing of Schema Bases and Overlays
- Authors: Burak Serdar (bserdar@computer.org)
- Status: [PROPOSED]
- Status Note: (explanation of current status)
- Supersedes: 
- Start Date: 2021-01-29

## Summary

This RFC defines JSON-LD processing for schema bases and overlays to
enable nested data structure support. With these extensions, OCA
stacks can use existing schemas in different domains (such as FHIR in
healthcare) to process data and/or to generate compliant data capture
forms.


## Motivation

The main purpose of this RFC is to extend OCA to handle data
structures that are not necessarily flat. Such data structures are
common for data exchange scenarios, in particular, FHIR for health
data exchange. FHIR is particularly challenging because of its deeply
nested and cyclic structure. With these extensions, creating schema
bases and overlays for FHIR-like standard schemas is possible.

Existing OCA specification uses schema bases and overlays without a
well-defined JSON-LD context. This RFC defines an ontology and the
associated JSON-LD context for schema bases and overlays. This allows
precise semantic definitions for schema base and overlay terms and
functionality associated with those terms. It also removes unnecesary
information such as overlay types, because the well-defined ontology
already provides that information.

## Tutorial

### Schema Base

The proposed schema base is as follows:

```
{
  "@context": "<baseurl>/layered.jsonld",
  "@type": "SchemaBase",
  "objectType": "someEntity",
  "attributes": [
    {
      "key": "efxnizr39ifc4"
    },
    {
      "key": "nfijh9i38ceSa",
      "flags": ["https://someOntology/PII"]
    },
    ...
  ],
  
}

```

The proposed schema base is a proper JSON-LD document that can be
expanded and processed using existing JSON-LD tools. It can refer to
other schema bases, and optional extensions can be added using
additional contexts. 

### @context

The @context defines several types and terms. Each term has specific
semantics and algorithms associated with it.

`SchemaBase` type is used to describe a schema base. 

 * objectType: This is the entity type defined by the schema base
 
```
        "SchemaBase": {
            "@id": "<base>/SchemaBase",
            "@context": {
                "@version": 1.1,
                "objectType": {
                    "@id": "<base>/Schema/objectType",
                    "@type": "@id"
                }
            }
        },

```

`attributes` term is used in schema bases and overlays. It defines a
nested attribute structure with some of the terms describing
properties of the attributes. 

```
"attributes": {
    "@id": "<base>/attributes",
    "@container": "@list",
    "@context": {
        "@version": 1.1,
        "@vocab": "</base>/attribute/",
        "key": "</base>/attribute/key",
        "reference": {
            "@id": "</base>/attribute/reference",
            "@type": "@id"
        },
        "arrayItems": {
            "@id": "</base>/attribute/arrayItems"
        },
        "allOf": {
            "@id": "</base>/attribute/allOf",
            "@container": "@list"
        },
        "oneOf": {
            "@id": "</base>/attribute/oneOf",
            "@container": "@list"
        },
        "flags": {
            "@id": "</base>/attribute/flags",
            "@type": "@id",
            "@container": "@set"
        },
        "attributeName": {
            "@id": "</base>/attribute/name"
        },
        "encoding": {
            "@id": "</base>/attribute/encoding"
        },
        "type": {
            "@id": "</base>/attribute/type"
        },
        "format": {
            "@id": "</base>/attribute/format"
        },
        "pattern": {
            "@id": "</base>/attribute/pattern"
        },
        "label": {
            "@id": "</base>/attribute/label"
        },
        "information": {
            "@id": "</base>/attribute/information"
        },
        "enumeration": {
            "@id": "</base>/attribute/enumeration"
        }
    }
}
```

  * `key`: Specifies the key for the attribute. It must be unique in
    the context it is defined. Keys are similar to the object keys of
    a JSON document, with the difference that array elements or
    references may have keys as well. These are used to match overlay
    components specific to a key.
  * `reference`: Specifies another object referenced by this object
  * `arrayItems`: If the defined attributes is an array, `arrayItems`
    specifies the structure of one element.
  * `allOf`: Specifies composition. The resulting element is the
    combination of the contents of the elements of this term.
  * `oneOf`: Specifies polymorphism. The resulting element is one of
    the elements of this term.
  * `flags`: A set of flags associated with the term. Each flag can
    belong to an ontology that flags this attributes. This can be
    privacy/risk classifications, blinding identity, etc.
  * `attributeName`: Name of this attribute as it appears in data.
  * `encoding`: Character encoding for the attribute.
  * `type`: Data type
  * `format`: Expected data format
  * `pattern`: Expected data pattern
  * `label`: Prompt label when constructing a form for this object
  * `information`: Comments
  * `enumeration`: Enumerated options for the attribute

#### Examples

A simple value is represented as:

```
{
  "key": "<key_value>"
}
```

The `key_value` is the value assigned to this attribute by the schema
author. Localized names can be given to this key using an overlay with term:

```
{
   "key": "<key_value>",
   "attributeName": "name"
}
```

All key values are unique within the container they are defined
in. Same key values can be used to refer to different attributes in
nested fields. That is:

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
      "flags": ["https://someOntology/PII"]
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
  "@context": "<base>/layered.jsonld",
  "schemaBase": "baseSchemaRef",
  "attributes": {
    ...
  },
  "schemaPaths": {
    ...
  }
```

There is no need to specify an overlay type. Each term has associated
semantics and algorithms that imply the type of operation and metadata
that is added to the underlying attribute.

As an example, the following index overlay uses `attributes` to define
name for schema base keys following the same structure are the schema base:

``` 
{
  "@context": "<base>/layered.jsonld",
  "schemaBase": "<schema base  id>,
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

The same overlay can also be defined using `schemaPaths` term:

```
{
  "@context": "<base>/layered.jsonld",
  "schemaBase": "<schema base id>",
  "schemaPaths": [
    {
      "schemaPath": "id_key1",
      "attributeName": "name_Key1"
    },
    {
      "schemaPath": "id_key1.id_key2",
      "attributeName": "name_Key2"
    }
  ]
}

```

This overlay uses `schemaPath` term that used dot-notation to address
schema base attributes.



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
