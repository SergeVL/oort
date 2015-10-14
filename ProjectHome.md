# Out of RDF Transmogrifying #

Oort is a collection of formats and libraries/tools aimed to facilitate use of RDF. It is geared towards instrumental mechanisms for reading and manipulating RDF data.

The project itself aims to be an umbrella for methods of using RDF in settings where code-centric objects, such as JSON (and "instrumental" xml) are easier to work with than full-fledged RDF.

It is not about "hiding" RDF, just "encapsulating it with simplified lenses".

## SparqlTree ##

Digests SPARQL results for your convenience. SparqlTree is a mechanism for turning the SPARQL select tabular form into tree structures by using special conventions on the naming of variables used in a query.

See the SparqlTree page.

## [Gluon](Gluon.md) ##

Gluon is a JSON format for RDF. It has a full syntax covering properties, resource references, bnodes and literals with optional datatype or language. With profiles, more succinct forms are possible.

See the [Gluon](Gluon.md) page.

## [Grit](Grit.md) ##

Grit (Grokkable RDF Is Transformable) is an XML format intended to solve the shortcomings of the cumbersome and barely usable RDF/XML format. Primarily designed for use in XSLT, it has potential to be a generally versatile XML-format for RDF.

See the [Grit](Grit.md) page.

## Just for Python? ##

The mechanisms above are intended to be implemented in more than Python (see their respective pages for more info). However, initially Oort was the set of Python-implementations. These are described at [The Oort home page](http://oort.to/).

### Oort "rdfview" ###

The main **oort** python package contains an **rdfview** module; basically a kind of O/R-mapper for RDF.

### OortPub ###

The OortPub package contains a WSGI-based toolkit for creating RDF-driven web apps. Views of RDF Graphs are made by compositions of queries and templates using mainly declarative python code.