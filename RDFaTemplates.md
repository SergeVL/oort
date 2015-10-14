

# Introduction #

The following is a definition of a simple method of expanding RDF data into templates using RDFa attributes as placeholders. It is not the first, and probably not the last. For other good works on this topic, see [#References](#References.md).

There is a working implementation (in XSLT of all things) which handles most of the common RDFa patterns. It also supports some (experimental) additional templating constructs for hidden matches, conditionals, sorting and defining reusable selections.

# Example #

## RDFa Template ##
```
   <html xmlns="http://www.w3.org/1999/xhtml"
         xmlns:dct="http://purl.org/dc/terms/"
         xmlns:foaf="http://xmlns.com/foaf/0.1/">
     <head>
       <title/>
     </head>
     <body>
       <div typeof="foaf:Person">
         <dl>
           <dt>Name</dt>
           <dd property="foaf:name"/>
           <dt>Homepage</dt>
           <dd>
             <a rel="foaf:homepage" href="?">
               <span property="dct:title"/>
             </a>
           </dd>
         </dl>
       </div>
     </body>
   </html>
```

## Input RDF ##
```
   <?xml version="1.0" encoding="utf-8"?>
   <rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
            xmlns:dct="http://purl.org/dc/terms/"
            xmlns:foaf="http://xmlns.com/foaf/0.1/"
            xmlns:sioc="http://rdfs.org/sioc/ns#">

     <foaf:Person rdf:about="http://example.net/me#">
       <foaf:name>Some One</foaf:name>
       <foaf:homepage>
         <sioc:Site rdf:about="http://example.net/">
           <dct:title xml:lang="en">Example.NET</dct:title>
         </sioc:Site>
       </foaf:homepage>
     </foaf:Person>

     <foaf:Person rdf:about="http://example.org/me#">
       <foaf:name>Some Other</foaf:name>
       <foaf:homepage>
         <sioc:Site rdf:about="http://example.org/">
           <dct:title xml:lang="en">Example.ORG</dct:title>
         </sioc:Site>
       </foaf:homepage>
     </foaf:Person>

   </rdf:RDF>
```

## Resulting HTML ##
```
   <html xmlns="http://www.w3.org/1999/xhtml"
         xmlns:foaf="http://xmlns.com/foaf/0.1/"
         xmlns:dct="http://purl.org/dc/terms/">
     <head>
       <title/>
     </head>
     <body>
       <div typeof="foaf:Person">
         <dl>
           <dt>Name</dt>
           <dd property="foaf:name">Some One</dd>
           <dt>Homepage</dt>
           <dd>
             <a rel="foaf:homepage" href="http://example.net/">
               <span property="dct:title">Example.NET</span>
             </a>
           </dd>
         </dl>
       </div>
       <div typeof="foaf:Person">
         <dl>
           <dt>Name</dt>
           <dd property="foaf:name">Some Other</dd>
           <dt>Homepage</dt>
           <dd>
             <a rel="foaf:homepage" href="http://example.org/">
               <span property="dct:title">Example.ORG</span>
             </a>
           </dd>
         </dl>
       </div>
     </body>
   </html>

```


# Basics #

  * Attribute values beginning with "?" are used as wildcards.
  * Wildcards are iterated on to produce multiple elements for multiple objects/values.
  * For hanging rels only the inner markup is repeated for multiple objects.
  * **@typeof** and **rel="rdf:type"** can be used as for matching _(currently only these...)_.

...

# Advanced #

...

# Implementation #

There is an XSLT for transforming RDFa Templates to XSLT at <http://purl.org/oort/impl/xslt/templating/rdfat-to-xslt.xslt>.

This in turn will take [Grit](Grit.md) as input, so pipe RDF/XML via the  [Grit XSLT](http://purl.org/oort/impl/xslt/grit/rdfxml-grit.xslt) and then into the RDFa Template output XSLT.

Got lost in the previous paragraphs? This is the gist of it, in a full example with no prerequisites but `xsltproc`:
```
    # Get the implementation:
    curl -LO http://purl.org/oort/impl/xslt/templating/rdfat-to-xslt.xslt
    curl -LO http://purl.org/oort/impl/xslt/grit/rdfxml-grit.xslt

    # Get example data and template:
    curl -LO http://purl.org/oort/impl/xslt/templating/test/personpages.rdf
    curl -LO http://purl.org/oort/impl/xslt/templating/test/personpages-rdfat.xhtml

    # Define variables for clarity  (use your own stuff here when going further):
    INPUT_RDF=personpages.rdf
    RDFA_TEMPLATE=personpages-rdfat.xhtml
    RESULT=personpages.xhtml

    # Main:
    xsltproc rdfat-to-xslt.xslt $RDFA_TEMPLATE > _tmp_rdfat.xslt
    xsltproc rdfxml-grit.xslt $INPUT_RDF | xsltproc _tmp_rdfat.xslt - > $RESULT
```

For more details and examples (and some code to show how you can use this in your project), see the [repository](http://code.google.com/p/oort/source/browse/#hg%2Fimpl%2Fxslt%2Ftemplating).

# References #

Other templating proposals/mechanisms using RDFa:

  * [A RDFa-based Templating Language proposal](http://www.kjetil.kjernsmo.net/software/rat/)
  * [Callimachus](http://code.google.com/p/callimachus/wiki/WebPatterns)