

## Introduction ##

Grit (Grokkable RDF Is Transformable) is an XML format intended to solve the shortcomings of the often cumbersome and varied [RDF/XML](http://www.w3.org/TR/REC-rdf-syntax/) format.

The design of Grit has been geared towards applying [XSLT](http://www.w3.org/TR/xslt) to generate output markup (e.g. html), but it's intended to be a generally versatile XML-format for RDF.


## Principles ##

Grit is a new format. This makes it clear that any XSLT (or other code) which uses the output isn't intended to transform RDF/XML. There is no formal way to indicate if an RDF/XML document has been normalized in any way, which makes it hard to impossible to use XML-tools with that serialization efficiently. Grit is designed to make that possible.

The core points of Grit are:

  * Enforce normalization such as keeping every triple about the same subject enclosed in a single `<resource>` element.
  * Make URI:s and references uniform so lookups will be dead simple (e.g. via xsl:key in XSLT).
  * Represent properties and types (metadata) as XML elements (qnames) in a completely uniform manner.


## Design ##

Grit uses _non-namespaced_ elements and attributes to represent the structure skeleton of a resource tree, where all resources have a `@uri` attribute. All properties use namespaced elements (as in RDF/XML).

A document consists of a `<graph>` root element containing (usually) one or more `<resource>` elements. These `<resource>` elements normally have a `@uri` attribute to set the subject of the described resource.

Example:

```
<graph
       xmlns:foaf="http://xmlns.com/foaf/0.1/"
       xmlns:dct="http://purl.org/dc/terms/"
       xmlns:xsd="http://www.w3.org/2001/XMLSchema#">

    <resource uri="http://example.org/somebody#">
        <a><foaf:Person/></a>
        <foaf:name>Some Body</foaf:name>
        <foaf:homepage ref="http://example.org/somebody"/>
    </resource>

    <resource uri="http://example.org/somebody">
        <dct:title xml:lang="en">The homepage of Some Body</dct:title>
        <dct:created fmt="datatype"><xsd:dateTime>2010-01-17T17:00:00Z</xsd:dateTime></dct:created>
    </resource>

</graph>
```

### Normalization ###

When representing a set of RDF statements as Grit XML, each described resource must be fully enclosed in a `<resource>` element, if it has a URI. If it is anonymous (i.e. a bnode), all statements either go within a property referencing it, or in a top-level `<resource>` element _without_ a `@uri`.

(In the rare cases where bnodes are used as _object_ in more than one statement, the bnode ID:s are given in the @ref and @uri attributes respecively, _prefixed by_ `_:`.  See [#BNodes](#BNodes.md) below for more details.)

### Types ###

Type references are treated specially, to facilitate XPath matching. They are represented as elements wrapped in an `<a>` element, as in:

```
    <resource uri="...">
        <a><foaf:Agent/></a>
        <a><foaf:Person/></a>
        ....
    </resource>
```

### URI references ###

Properties referencing resources with a `@ref` attribute containing the URI:

```
    <foaf:homepage ref="http://example.org/somebody"/>
```

### BNodes ###

Properties referencing bnodes carry the descriptions inline (similar to @parseType='Literal' in RDF/XML).

Example:
```
    <foaf:knows>
        <foaf:nick>otherone</foaf:nick>
    </foaf:knows>
```

In the rare cases where bnodes are used as _object_ in more than one statement, Grit allows giving the needed bnode id in the regular `@ref` and `@uri`, as a special value prefixed by `_:`.

Example:
```
    <resource uri="_:p1">
        <a><foaf:Person/></a>
        <foaf:knows ref="_:p2"/>
    </resource>
    <resource uri="_:p2">
        <a><foaf:Person/></a>
        <foaf:knows ref="_:p1"/>
    </resource>
```

This form isn't allowed in the URI formal syntax, but in order to make processing of Grit XML easier, the value space in these attributes is overloaded to be both real URI:s and this bnode id syntax.

The opinion of Grit is that this is rarely a feature, as it makes the particulars of bnodes more difficult to work with. But since it _may_ have it's usage (and since such statements exist in the wild), Grit allows it.

(There is also a usability trade-off in nesting property elements for bnodes in the referencing property element, as using code must check for the presence of a `@ref` in order to determine whether to use the contents of the property element, or look up a possible description via `/graph/resource/@uri` on it.)

### Literals ###

Literals are given as inlined strings, as expected.

#### Language Literals ####

A literal with language looks just as in RDF/XML:

```
    <dct:title xml:lang="en">The homepage of Some Body</dct:title>
```

#### Typed Literals ####

_This is a preliminary design which must be evaluated for usability._

Typed literals are marked with `@fmt`, which may have a value of either `datatype`, for which the value is wrapped in an element whose qname is resolvable to the datatype URI:

```
    <dct:created fmt="datatype"><xsd:dateTime>2010-01-17T17:00:00Z</xsd:dateTime></dct:created>
```

or `xml` (for an inlined XML literal):

```
    <dct:description fmt="xml">
        <p xmlns="http://www.w3.org/1999/xhtml" xml:lang="en">Details in the <a href="...">XHTML</a> format.</p>
    </dct:description>
```

### RDF Lists ###

Lists use null-namespaced `<li>` elements, corresponding to the content of regular `<resource>` elements (either carrying @ref attributes or representing nested bnodes). (This, of course, is equivalent to the @parseType="Collection" of RDF/XML.)

Example:

```
    <bibo:editorList>
      <li ref="http://example.org/editor/1"/>
      <li ref="http://example.org/editor/2"/>
      <li ref="http://example.net/editor/3"/>
      <li>
        <rdfs:label>editor 4<rdfs:label>
      </li>
      <li>
        <rdfs:label>editor 5<rdfs:label>
      </li>
    </bibo:editorList>
```


## Implementations ##

There is an XSLT for transforming RDF/XML to Grit at http://purl.org/oort/impl/xslt/grit/rdfxml-grit.xslt. There's also an XSLT for [GRDDL](http://www.w3.org/TR/grddl/):ing Grit back to RDF/XML at http://purl.org/oort/impl/xslt/grit/grit-grddl.xslt.

(Also, checkout the [source repository](http://code.google.com/p/oort/source/browse/trunk/), especially the [examples](http://code.google.com/p/oort/source/browse/trunk/impl/etc/grit/examples) and the aforementioned [implementations](http://code.google.com/p/oort/source/browse/trunk/impl/#impl/xslt/grit).)


## Relationship to Other XML-based RDF formats ##

There are several XML-based formats for expressing RDF, with different design principles and intended use.

Only RDF/XML and RDFa are W3C standards, and XHTML+RDFa isn't intended for the same things as Grit (representing normalized data), so it is excluded in these comparisons.

### RDF/XML ###

When implementing an XSLT for instrumental use of RDF/XML, it became apparent that "just" normalizing RDF/XML (as has been done by many people) may not be enough to make the result readily usable in XSLT. Matching types, looking up things by XPath etc. should be as simple as possible. This is what Grit is intended to solve.

Allowing for compact xpath expressions is the primary reason for not using a namespace (a.k.a. the "blank" or "null" namespace) for the "skeleton" (the `graph` and `resource` elements and the `@uri`, `@ref` and `@fmt`attributes).

Another motivation for deviating from the RDF/XML format is to attempt to provide a more readable format in general, designed have a _much_ more normalized form (limiting many variations of expression). This should also make it much more straightforward to work with using regular XML tools such as XSLT+XPath, XQuery and many existing XML API:s available.

### TriX, RXR ###

Using namespaces to represent properties and types with qnames is at the heart of Grit, just as in RDF/XML. This sets it apart from e.g. [TRiX](http://sw.nokia.com/trix/TriX.html) and [RXR](http://www.wasab.dk/morten/blog/archives/2004/05/30/transforming-rdfxml-with-xslt). These formats express triples directly, and use full URI:s for properties.

### Atom ###

The design of Grit is to some degree reminiscent of the [Atom](http://tools.ietf.org/html/rfc4287) XML syntax. In terms of resource description, it is both more versatile, by supporting rich datatypes and also a uniform way of representing both literals and "links", i.e. resource references.

In comparison, Atom, the `@rel`-attribute of the `<atom:link>` element is used to name relations, and for anything else either the builtin properties of atom (title, updated etc.) or invent extensions who lack any shared formal semantics. Still it is _important_ to understand that the Atom semantics are about capturing entries (slightly similar to a "unit of description") and _temporality_, that is the `updated` element determines which is the most recent (and thus currently "active") description. It also defines little in terms of an open world, graphs of statements etc.

Working on integrating these aspects is (at least currently) beyond the scope of Grit, interesting as it may be. The Grit serialization of RDF has a form which may be suitable for embedding in entries (along with rules for how they relate). And something could be made of multiple named graphs/contexts with timestamps..

## Tribute ##

All of the other formats mentioned here (and more), as well as a lot of other existing (often XSLT-leveraging) tools for normalizing RDF/XML have been extensive sources of inspiration for Grit. If anything Grit should be considered as homage to all that work. That said, it does strive, to some extent, to be their alternative.

Here are some interesting articles on the topic of using/normalizing XML-serializations of RDF in random order:

  * [XSLT-based Data Grounding for RDF](http://www.wsmo.org/TR/d24/d24.2/v0.1/20070412/rdfxslt.html)
  * [SimpleRdfXml](http://esw.w3.org/topic/SimpleRdfXml)
  * [TreeTriples: Syntax Comparison](http://djpowell.net/schemas/treetriples/1/SyntaxComparison.html)
  * [RDF syntax normalization using XML validation](http://www.aifb.kit.edu/images/7/72/SemRUs2009SyntaxNorm.pdf)

## Tips & Tricks ##

Leverage [EXSLT](http://exslt.org/) whenever you can when working with XSLT on this format (and other XML) .

_Some useful examples of using Grit are in the making but have yet to be published._