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
  "@context": "https://layeredschemas.org/layered.jsonld",
  "@type": "SchemaBase",
  "objectType": "someEntity",
  "attributes": {
    "key1": {},
    "key2": {
      "attributes": {
         "key2_1": {}
      }
    },
    "key3": {
      "flags": ["https://someOntology/PII"]
    },
    ...
  ],
  
}

```

The proposed schema base is a JSON-LD document that can be expanded
and processed using existing JSON-LD tools. It can refer to other
schema bases, and optional extensions can be added using additional
contexts.

### @context

The @context defines several types and terms. Each term has specific
semantics and algorithms associated with it.

`SchemaBase` type is used to describe a schema base. 

 * objectType: This is the entity type defined by the schema base
 
```
        "SchemaBase": {
            "@id": "https://layeredschemas.org/SchemaBase",
            "@context": {
                "@version": 1.1,
                "objectType": {
                    "@id": "https://layeredschemas.org/Schema/objectType",
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
    "@id": "https://layeredschemas.org/attributes",
    "@container": "@id",
    "@context": {
        "@version": 1.1,
        "@vocab": "https://layeredschemas.org/attribute/",
        "key": "https://layeredschemas.org/attribute/key",
        "reference": {
            "@id": "https://layeredschemas.org/attribute/reference",
            "@type": "@id"
        },
        "arrayItems": {
            "@id": "https://layeredschemas.org/attribute/arrayItems"
        },
        "allOf": {
            "@id": "https://layeredschemas.org/attribute/allOf",
            "@container": "@list"
        },
        "oneOf": {
            "@id": "https://layeredschemas.org/attribute/oneOf",
            "@container": "@list"
        },
        "flags": {
            "@id": "https://layeredschemas.org/attribute/flags",
            "@type": "@id",
            "@container": "@set"
        },
        "attributeName": {
            "@id": "https://layeredschemas.org/attribute/name"
        },
        "encoding": {
            "@id": "https://layeredschemas.org/attribute/encoding"
        },
        "type": {
            "@id": "https://layeredschemas.org/attribute/type"
        },
        "format": {
            "@id": "https://layeredschemas.org/attribute/format"
        },
        "pattern": {
            "@id": "https://layeredschemas.org/attribute/pattern"
        },
        "label": {
            "@id": "https://layeredschemas.org/attribute/label"
        },
        "information": {
            "@id": "https://layeredschemas.org/attribute/information"
        },
        "enumeration": {
            "@id": "https://layeredschemas.org/attribute/enumeration"
        }
    }
}
```

  * `@id`: Specifies the id for the attribute. It must be unique in
    the schema it is defined in. 
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

A simple key/value pair is represented as:

```
"attributes": {
   "<key>": {},
   ...
}
```
or

```
"attributes": [
  {
    "@id": "<key>"
  },
  ...
]
```

The `key` is the value assigned to this attribute by the schema
author. Localized names can be given to this key using an overlay with term:

```
{
   "@id": "<key>",
   "attributeName": "name"
}
```

Nested objects can be defined for keys:

```
"attributes": [
  {
    "@id": "name1",
    "attributeName": "name"
  },
  {
    "@id": "obj",
    "attributes": [
       {
         "@id": "name2",
         "attributeName": "name"
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

The `name` and `obj.name` refer to two different attributes with ids
"name1" and "name2" respectively.

An attribute can be flagged:

```
    {
      "@id": "nfijh9i38ceSa",
      "flags": ["https://someOntology/PII"]
    }
```

For instance, this can be used to flag PII information based on BIT.

An attribute can be a reference to another schema base:

```
{
   "@id": "patient",
   "reference": "http://someSite/Patient"
}
```

The above defines the field "patient" to be a "Patient" object, whose
schema base is given in the `reference` value.

An attribute can be a nested object:

```
{
   "@id": "nestedObject",
   "attributes": [
      {
        "@id": "nestedAttribute"
      },
      ...
   ]
}
```

An attribute can be an array whose items can be described in the schema base:

```
{
  "@id": "valueArray",
  "arrayItems": {}
}
```

Instance:

```
"valueArray": [ "value1", "value2", ... ]
```

```
{
  "@id": "objectArray",
  "arrayItems": {
     "attributes": [
        {
          "@id": "key1"
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
  "@id": "p1",
  "allOf": [
    {
      "reference": "http://someObject"
    },
    {
      "attributes": [
        {
           "@id": "attr"
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
   "@id": "p2",
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
  "@context": "https://layeredschemas.org/layered.jsonld",
  "@type": "Overlay",
  "schemaBase": "baseSchemaRef",
  "attributes": {
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
  "@context": "https://layeredschemas.org/layered.jsonld",
  "@type": "Overlay",
  "schemaBase": "<schema base  id>,
  "attributes": [
    {
      "@id": "id1",
      "attributeName": "name_Key1"
    },
    {
      "@id": "id2",
      "attributeName": "name_Key2"
    }
  ]
}
```



## Semantics 

Each term has well-defined semantics that include the meaning and
operations defined for that term. 

### Term: attributes

The term `attributes` is a container where each node with an @id
defines a new attribute. An attribute can be one of:

  * Object: An  `attributes` term defines the nested object structure.
  * Array: An `arrayItems` term defines the structure of each array element.
  * Reference: A `reference` term links to another object. This can be
    a pointer to a schema manifest, or a pointer to an object whose
    schema can be derived based on the current processing context.
  * Composition: An `allOf` term lists the parts of the object
  * Polymorphism: A `oneOf` term lists the possible types.
  * A simple value: If none of the above exists, the value is a simple
    value.
    
The term `attributes` defines a `merge` algorithm that receives two
`attributes` and combines the contents of matching attributes:

Input 1 :
```
"attributes": {
  "k1": {
    "flags": ["flag1"]
  },
  "k2": {
    "attributes": {
      "k3":{}
    }
  }
}
```

Input 2 :
```
"attributes": {
  "k1": {
    "flags": [ "flag2" ]
  },
  "k3": {
    "attributeName": "attr3"
  }
}
```
Result:

```
"attributes": {
  "k1": {
    "flags": [ "flag1", "flag2" ]
  },
  "k2": {
    "attributes": {
      "k3": {
        "attributeName": "attr3"
      }
    }
  }
}
```

During the merge operation:
  * If a term is defined as `@set`, their contents are combined
  * If a term is defined as `@list`, contents of the second input is appended to
    the first
  * For non-container terms, terms of input1 and input2 are merged,
    with input2 terms overwriting matching input1 terms

### Schemas

A schema is a schema base and zero or more overlays. A schema is
constructed using a "schema manifest" that combines the schema layers:

```
{
  "@context": "https://layeredschemas.org/layered.jsonld",
  "@type": "Schema",
  "@id": "schema Id",
  "issuedBy": "...",
  "issuerRole": "...",
  "issuedAt": "...",
  "purpose": "...",
  "classification": "...",
  "objectType": "...",
  "objectVersion": "...",
  "schemaBase": "link to schema base",
  "overlays": [
     "overlay1",
     "overlay2",
     ...
   ]
}
```

The schema manifest links a schema base and overlays to create a
schema that is localized, adopted to a particular
context/jurisdiction, and versioned. It defines the entity type
specified by the schema (objectType), the version of the specification
(objectVersion), and optionally, adds a signature by the schema
publisher for the schema users to validate.

#### Referencing Schemas

A schema base may include references to other schemas. The overlay
structure also makes it possible to extend a base schema to include
references. These references can be

  * A link to a schema manifest using its @id. This link will resolve
  to a unique schema representing a fixed version of an object.
  * A link to a schema base. There can be many schemas based on the
  selected schema base. The selection of the actual schema should be
  done based on the processing context and metadata associated with
  schemas. For example, suppose a schema base for object A refers to
  the schema base for object B. When processing an instance of A in
  English language schema, only the schemas of B that are in English
  would be considered.

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
