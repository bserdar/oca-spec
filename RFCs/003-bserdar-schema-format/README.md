# JSON-LD Processing of Schema Layers to Enable Semantic Processing of Nested Data
- Authors: Burak Serdar (bserdar@computer.org)
- Status: [PROPOSED]
- Status Note: (explanation of current status)
- Supersedes: 
- Start Date: 2021-01-29

## Summary

This RFC defines JSON-LD processing for schema layers to enable nested
data structure support. With these extensions, OCA stacks can use
existing schemas in different domains (such as FHIR in healthcare) to
process data and/or to generate compliant data capture forms.


## Motivation

The main purpose of this RFC is to extend OCA to handle data
structures that are not necessarily flat. Such data structures are
common for data exchange scenarios, in particular, FHIR for health
data exchange. FHIR is particularly challenging because of its deeply
nested and cyclic structure. With these extensions, creating schema
layers for FHIR-like standard schemas is possible.

Existing OCA specification uses schema bases and overlays without a
well-defined JSON-LD context. This RFC defines an ontology and the
associated JSON-LD context for schemas and schema layers. This allows
precise semantic definitions for schema and layer terms and
functionality associated with those terms. It also removes the
unnecesary distinction between schema bases and different types of
overlays, because a well-defined ontology and the schema that combines
layers already provide that information.

## Tutorial

### Schema Layer

The proposed schema layer is as follows:

```
{
  "@context": "http://schemas.cloudprivacylabs.com/layers.jsonld",
  "@type": "Layer",
  "@id": "http://example.org/someEntity/schemaBase",
  "objectType": "someEntity",
  "attributes": {
    "key1": {},
    "key2": {
      "attributes": {
         "key2_1": {}
      }
    },
    "key3": {
      "privacyClassifications": ["https://someOntology/PII"]
    },
    ...
  ],
  
}

```

  * `@type`: This is a Layer document
  * `@id`: The ID for the layer
  * `objectType`: The object described by this layer.

The proposed layer is a JSON-LD document that can be expanded and
processed using existing JSON-LD tools. It can refer to other schemas,
and optional extensions can be added using additional contexts.

In this RFC, there is no functional distinction between a schema base
and an overlay. A schema base is simply the first layer in a schema
layer stack.

### @context

The @context defines several types and terms. Each term has specific
semantics and algorithms associated with it.

`Layer` type is used to describe a schema layer. 

 * objectType: This is the entity type defined by the schema base
 
```
        "Layer": {
            "@id": "http://schemas.cloudprivacylabs.com/Layer",
            "@context": {
                "@version": 1.1,
                "objectType": {
                    "@id": "http://schemas.cloudprivacylabs.com/Schema/objectType",
                    "@type": "@id"
                }
            }
        },

```

`attributes` term is used in schema layers. It defines a nested
attribute structure:

```
"attributes": {
    "@id": "http://schemas.cloudprivacylabs.com/attributes",
    "@container": "@id",
    "@context": {
        "@version": 1.1,
        "attributeName": {  
            "@id": "http://schemas.cloudprivacylabs.com/attribute/name"
        },
        "reference": {
            "@id": "http://schemas.cloudprivacylabs.com/attribute/reference",
            "@type": "@id"
        },  
        "arrayItems": {
            "@id": "http://schemas.cloudprivacylabs.com/attribute/arrayItems",
            "@context": {
                "@version": 1.1,
                "reference": {
                    "@id": "http://schemas.cloudprivacylabs.com/attribute/reference",
                    "@type": "@id"
                },
                "allOf": {
                    "@id": "http://schemas.cloudprivacylabs.com/attribute/allOf",
                    "@container": "@list",
                    "@context": {
                        "@version": 1.1
                    }
                },
                "oneOf": {
                    "@id": "http://schemas.cloudprivacylabs.com/attribute/oneOf",
                    "@container": "@list",
                    "@context": {
                        "@version": 1.1
                    }
                }
            }
        },
        "allOf": {
            "@id": "http://schemas.cloudprivacylabs.com/attribute/allOf",
            "@container": "@list",
            "@context": {
                "@version": 1.1,
                "reference": {
                    "@id": "http://schemas.cloudprivacylabs.com/attribute/reference",
                    "@type": "@id"
                }
            }
        },
        "oneOf": {
            "@id": "http://schemas.cloudprivacylabs.com/attribute/oneOf",
            "@container": "@list",
            "@context": {
                "@version": 1.1,
                "reference": {
                    "@id": "http://schemas.cloudprivacylabs.com/attribute/reference",
                    "@type": "@id"
                }
            }
        }
    }
},
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
  * `attributeName`: Name of this attribute as it appears in data.
  
```
"privacyClassification": {
    "@id": "http://schemas.cloudprivacylabs.com/attribute/privacyClassification",
    "@container": "@set"
},
"information": {
    "@id": "http://schemas.cloudprivacylabs.com/attribute/information"
},
"encoding": {
    "@id": "http://schemas.cloudprivacylabs.com/attribute/encoding"
},
"type": {
    "@id": "http://schemas.cloudprivacylabs.com/attribute/type"
},
"format": {
    "@id": "http://schemas.cloudprivacylabs.com/attribute/format"
},
"pattern": {
    "@id": "http://schemas.cloudprivacylabs.com/attribute/pattern"
},
"label": {
    "@id": "http://schemas.cloudprivacylabs.com/attribute/label"
},
"enumeration": {
    "@id": "http://schemas.cloudprivacylabs.com/attribute/enumeration",
    "@container": "@list"
},

```
  
  * `privacyClassification`: A set of flags associated with the
    term. Each flag can belong to an ontology that flags this
    attributes. This can be privacy/risk classifications, blinding
    identity, etc.
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

An attribute has privacy classification:

```
    {
      "@id": "nfijh9i38ceSa",
      "privacyClassification": ["https://someOntology/PII"]
    }
```

For instance, this can be used to flag PII information based on BIT.

An attribute can be a reference to another schema. References are
open-ended, and they can be a
  
   * Reference using a DRI
   * Reference using target object type
   * Reference using a specific variant of a schema

```
{
   "@id": "patient",
   "reference": "http://someSite/Patient"
}
```

The above defines the field "patient" to be a "Patient" object, whose
schema is given in the `reference` value. This is a reference using
the object type, which does not specify a definite schema, thus an
application specific schema selection must be done. A reference using
a DRI would specify a definite schema.

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

An attribute can be an array:

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



### Layers as Overlays

Overlays are simply layers. There is no functional or structural
difference between layers. Each term used in a layer has associated
semantics and algorithms that imply the type of operation and metadata
that is added to the underlying attribute.

As an example, the following index layer uses `attributes` to define
name for schema keys:

``` 
{
  "@context": "http://schemas.cloudprivacylabs.com/layers.jsonld",
  "@type": "Layer",
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
    "privacyClassification": ["flag1"]
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
    "privacyClassification": [ "flag2" ]
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
    "privacyClassification": [ "flag1", "flag2" ]
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
  * Attributes that are defined as `@set` in the context
    (`privacyClassification`) are combined
  * If a term is defined as `@list`, contents of the second input is
    appended to the first
  * For non-container terms, terms of input1 and input2 are merged,
    with input2 terms overwriting matching input1 terms

### Schemas

A schema specifies one of more schema layers:

```
{
  "@context": "http://schemas.cloudprivacylabs.com/schema.jsonld",
  "@type": "Schema",
  "@id": "schema Id",
  "issuedBy": "...",
  "issuerRole": "...",
  "issuedAt": "...",
  "purpose": "...",
  "classification": "...",
  "objectType": "...",
  "objectVersion": "...",
  "layers": [
     "layer1",
     "layer2",
     ...
   ]
}
```

The schema combined schema layers to create a schema that is
localized, adopted to a particular context/jurisdiction, and
versioned. It defines the entity type specified by the schema
(objectType), the version of the specification (objectVersion), and
optionally, adds a signature by the schema publisher for the schema
users to validate.

#### Referencing Schemas

A schema layer may include references to other schemas. These
references can be

  * A link to a schema using its @id. This link will resolve to a
  unique schema representing a fixed version of an object.
  * A link to a schema using a DRI. This link will resolve to a
  unique schema representing a fixed version of an object.
  * A link to a defined object type. There can be many schemas based
  on the selected object type. The selection of the actual schema
  should be done based on the processing context and metadata
  associated with schemas. For example, suppose a schema for object A
  refers to the schema for object B. When processing an instance of A
  in English language schema, only the schemas of B that are in
  English would be considered.

## Implementations

The layered schemas implementation is here:


https://github.com/cloudprivacylabs/lsa
