**_Important_**

> This design has served as input to [JSON-LD](http://json-ld.org/). The work on Gluon has stopped, and JSON-LD is to be considered its successor.

> This page is left for informational purposes, but should be considered deprecated.



## Introduction ##

Gluon is a JSON format for RDF. It has a full syntax covering explicit (CURIE) properties, resource references and literals with optional datatype or language.

With profiles, gluon can be made more succinct and "naturally JSONic", by binding properties to names with control of their value types.

## Overview ##

Gluon supports a full, "raw" format, where RDF-specific details such as the subject URI, language, datatype or referenced URI for the object are expressed using special properties of a JSON-object.

Example of **Raw Gluon**:

```
{
    "profile": {
        "prefix": {
            "dct": "http://purl.org/dc/terms/",
            "xsd": "http://www.w3.org/2001/XMLSchema#"
        }
    },
    "resources": [
        {
            "$uri": "http://example.org/item/1",

            "rdf:type": [
                {"$ref": "http://www.w3.org/2000/01/rdf-schema#Resource"},
            ]
            "rdfs:label": "item_1",

            "dct:title": {"@en": "Example", "@sv": "Exempel"},

            "dct:created": {
                "$value": "2010-01-04T01:35:14Z",
                "$datatype": "xsd:dateTime"
            }

            "rdf:value": [1, 1.0, true]

            "rdf:value": [
                {
                    "$list": ["a", "b", "c"]
                }
            ],
            "dct:source": [
                {
                    "$list": [
                        {"$ref": "http://example.org/source/1"},
                        {"dct:title": "Source 2"}
                    ]
                }
            ]
        },
        {
            "$uri": "http://example.org/source/1",
            "dct:title": "Source 1"
        }
    ]
}
```

Notice that prefixes are defined in a "profile", which are used in the object properties (and datatype references) to form "qnames" (CURIEs). This is "as far" as Gluon goes. If you want direct triples with full URI:s where applicable, use one of the existing JSON formats instead (see [Plain Triple JSON](#Plain_Triple_JSON.md) below).

To support a natural notation both easier to write, read (by humans) and work with using direct access (commonly via javascript), Gluon allows to define plain property names and bind these with the property URI, datatype, language (combined with a global lang) and base URI resolution.

Example of **Compact Gluon**:

```
{
    "profile": {
        "prefix": {
            "dct": "http://purl.org/dc/terms/",
            "owl": "http://www.w3.org/2002/07/owl#",
            "rdf": "http://www.w3.org/1999/02/22-rdf-syntax-ns#",
            "rdfs": "http://www.w3.org/2000/01/rdf-schema#",
            "xsd": "http://www.w3.org/2001/XMLSchema#"
        },
        "default": "dct",
        "define": {
            "a": {"term": "rdf:type", "symbol": true},
            "created": {"datatype": "xsd:dateTime"},
            "title": {"localized": true},
            "alternative": {"many": true},
            "source": {"reference": true, "many": false},
            "Resource": {"from": "rdfs"},
            "Thing": {"from": "owl"}
        }
    },
    "lang": "en",
    "linked": {
        "http://example.org/item/1": {
            "a": ["Resource", "Thing"],
            "created": "2010-01-04T01:35:14Z",
            "source": "http://example.org/source",
            "title": "Example",
            "alternative": ["ex."]
        },
        "http://example.org/source": {
            "title": "Source"
        }
    }
}
```


## The Format ##

### Wrapper ###

The "root" object of a Gluon document is a wrapping "envelope", where definitions for interpreting the data are put, along with a collection of one or more described resources.

A skeleton for Gluon looks like:

```
{
    "profile": {

        "prefix": {
            ...
        },

        // optional (sets prefix to use if left out)
        "default": ...,

        // optional
        "define": {
            ...
        }

    },

    // optional
    "lang": ...,

    // optional
    "base": ...,

    // either:
    "resources": [
        ...
    ],

    // or:
    "linked": {
    },
    "nodes": [
    ]

}
```


### The Data ###

The object carrying the data is provided as a value in the wrapper object.

In raw form:

  * `resources`: gives a list of objects, who each will have the key `$uri` to give their URI:s (if it is known).

In compact form:

  * `linked`: an object, giving more compact form where each key represents the URI of the referenced object (which thus should not contain a "$uri" key for this).
  * If `linked` is used and there are top-level bnodes in the data (i.e. bnodes not being objects of any triples, in which case they would be nested within another object), `nodes` is a list of these.


### Raw Form ###

In this form, "profile" only contains the "prefix" object (no "define", "default" or "lang"). These prefixes are used just as in N3 and RDFa, for declaring prefixes used to interpret properties given as CURIE:s.


### Resource Description Objects ###

An object describing a resource has:

  * a URI: given with the key "$uri",
  * Properties: given as CURIE:s in raw form, and regular tokens in compact form.

The different value types are:

  * Multi-valued properties: given in JSON lists
  * List-valued properties: given as an object with the key "$list"


#### Resource References ####

By default, any properties for which there is _one or more_ URI:s as objects, _JSON lists_ are used as values.

Resource references are used for referencing any resource for which a URI is known.

##### BNodes #####

BNodes are directly inlined as a nested _resource description_ object (without URI). Thus, bnode id:s are _not_ supported.

#### Literals ####

By default, values are expected to have a cardinality of one.

##### Language Literals #####

Literal with a given language tag are represented as JSON objects with language as keys of the form "@LANG", e.g. "@en".

##### Typed literals #####

  * For XSD types which have native JSON value counterparts, pure values are used.
  * Any other types (including non-XSD types) are written as {"$datatype": ..., "$value": ...}


### Compact Profiles ###

In the `define` object you can define mechanisms for using compact expressions of literals and references.

The `default` can be used to provide a default prefix used for prefix-less CURIE:s. In these examples, this is used:

```
    "default": "dct",
```

A definition is bound to a URI via either `term`, giving a CURIE, or `from`, which takes the key and prepends the value of `from` plus ":" to calculate the CURIE. The CURIEs are resolved from the "prefix" data as expected.


#### symbol ####

This rdf:type shortcut definition says that the value is a "symbol" (CURIE or prefix-less token used with `default`):

```
    "a": {"term": "rdf:type", "symbol": true}
    "Resource": {"from": "rdfs"}

    ...

    {
        "a": ["Resource", "Thing"], ...
    }
```

#### datatype ####

By defining a `datatype` you can provide simple strings to represent datatyped literals:

```
    "created": {"datatype": "xsd:dateTime"}

    ...

    {
        "created": "2010-01-04T01:35:14Z", ...
    }
```

#### localized ####

If the wrapper object provides a `lang` to set the default language, this can be used to use plain strings as language literals using this default:

```
    "title": {"localized": true}

    ...

    {
        "title": "Example", ...
    }
```


#### reference ####

By setting `reference` to `true`, plain strings are interpreted as resource references (links):

```
    "source": {"reference": true},

    ...

    {
        "source": ["http://example.org/source", ...],
        ...
    }

```

#### many ####

The single/multiple distinction is based on the rule (from above) stating that literals are singular by default and references are multiple (given in lists). The `many` key can be used to change that, as in:

```
    "source": {"reference": true, "many": false},
    "alternative": {"many": true}

    ...

    {
        "source": "http://example.org/source", ...
        "alternative": ["ex."],
    }

```

#### Still Undefined ####

It is currently _undefined_ if the use of "define" to represent RDF as Compact Gluon should _exclude_ triples whose properties have not been "defined" or not. If not omitted, such data can be provided as raw Gluon of course, but this might be detrimental to the ideal of compact Gluon working as a _simple_, constricted data format for "casual use".

Also, it is feasible to imagine profiles being more or less automatically created (or even implied). Such an algorithm remains to be defined, but the Linked Data API mechanisms for this is a feasible way to go (see the [Linked Data API JSON format](http://code.google.com/p/linked-data-api/wiki/API_Formatting_Graphs)).


## See Also ##

### Source ###

If you just want to dive in, check out the source in the Oort repo, which contains imlementations with runnable tests, and especially a set of [specs and examples](http://code.google.com/p/oort/source/browse/#hg/etc/gluon) consisting of N3 and JSON combos illustrating the possibilities.

(The Python test runner tests both parsing and serialization, comparing all results to ensure full conversion coverage (this requires RDFLib 3.0).)

### Other JSON-based RDF Formats ###

For a full coverage, see [the linked-data-api collection of JSON Formats](http://code.google.com/p/linked-data-api/wiki/JSONFormats).

#### Plain Triple JSON ####

  * [SPARQL Results JSON format](http://www.w3.org/TR/rdf-sparql-json-res/)
  * [Talis RDF/JSON](http://n2.talis.com/wiki/RDF_JSON_Specification)

#### Terse RDF-JSON ####

Lots of stuff here, hopefully to converge!

  * [Linked Data API JSON format](http://code.google.com/p/linked-data-api/wiki/API_Formatting_Graphs)
  * [JRON](http://decentralyze.com/2010/06/04/from-json-to-rdf-in-six-easy-steps-with-jron/)
  * [JSON-LD](http://rdfa.digitalbazaar.com/specs/source/json-ld/)
  * [JSN3](http://webr3.org/apps/specs/jsn3/)
  * [IrON irJSON](http://openstructs.org/iron/iron-specification#mozTocId462570)
  * [RDFj](http://code.google.com/p/backplanejs/wiki/Rdfj)

### Other JSON-based Formats ###

  * [GData-JSON](http://code.google.com/intl/sv-SE/apis/gdata/docs/json.html)


---


Thanks for reading!