# Introduction #

The SPARQL Tree tools digest SPARQL results for your convenience.

A "SPARQL tree" is a processed SPARQL query result which combines bound variables into trees of data. It processes regular results from queries using variables named according to a specific convention (designating "tree structure").

Example:

```
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX : <http://xmlns.com/foaf/0.1/>

SELECT * WHERE {
    ?person a :Person;
        :name ?person__1_name .
    OPTIONAL {
        ?person :givenName ?person__1_givenName;
            :surname ?person__1_surname .
    }
    OPTIONAL { ?person rdfs:comment ?person__1_comment }
    OPTIONAL {
        ?person :interest ?person__interest .
        ?person__interest dct:title ?person__interest__1_title .
    }
}
```

Results for the above are automatically turned into something like the following JSON:

```
{
  "person": [
    {
      "$uri": "http://purl.org/NET/dust/foaf#self",
      "name": "Niklas Lindstr\u00f6m",
      "givenName": null, "surname": null,
      "comment": {"@en": "Does stuff with RDF."},
      "interest": [
        {
          "$uri": "http://en.wikipedia.org/wiki/Resource_Description_Framework",
          "title": {"@en": "Resource Description Framework"}
        },
        {
          "$uri": "http://python.org/",
          "title": {"@en": "Python Programming Language"}
        }
      ]
    }
  ]
}
```


# Mechanics #

Implementations use the naming convention to inspect the query variable names (in the result head). Variables containing `'__'` as "key separators" are interpreted as "tree" variables. This tells SparqlTree to combine the results for these as parent-child keys (splitting on `'__'`).

The values for these keys become lists of all bound combinations (at the given "tree level"), including:

  * plain strings
  * other "native json types" (boolean, int and float)
  * non-coerced "typed literals" as dictionaries with the special keys `"$datatype"` and `"$value"`
  * "language literals" as dictionaries with language tag keys (using the locale, prefixed by '`'@'`)
  * dictionaries of "nested objects" (who, along with "regular node keys" may contain the special `"$uri"` or `"$id"` keys if the SPARQL result binding is of type URI or BNode).

Furthermore, if a "child" key starts with `'1_'`, SparqlTree instead expects the result to hold either one value, or that any bound values are language literals. The effect of this is to produce single values (that is, instead of lists of values). It also merges lists of language literals into a single dictionary with language tag keys.

_Note: these conventions are currently idiomatic for "SparqlTree-JSON", but the ideal is to make them compliant with the ongoing work for representing RDF in JSON._


# Implementations #

Some usage examples can be found [here](http://code.google.com/p/oort/source/browse?repo=default#hg/etc/sparqltee/examples).

In the Oort JS repository, you can find [an example webpage](http://js.oort.googlecode.com/hg/examples/dbpedia_nations.html) using the the javascript implementation.

The initial implementation in Python is in the [main (python) repository](http://code.google.com/p/oort/source/browse/?repo=python#hg/oort/sparqltree).


## Installation ##

**Important**: _These instructions are a bit out of date, but the code references above should give you the needed tools to try it out._

If you have `distribute` (includes the `pip` cmdline tool) installed (fairly common), you can install **from the latest trunk** by invoking:
```
$ sudo pip install hg+http://python.oort.googlecode.com/hg#egg=Oort-dev
```

Or manually by [checking out the source code](http://code.google.com/p/oort/source/checkout?repo=python) (you only need the Oort python package) and do either:
```
$ python setup.py install
```
or (to be able to do `svn update` and automatically having the latest code importable):
```
$ sudo python setup.py develop
```


## Usage ##

The python implementation can be used both programmatically and from the cmdline. Use it to run sparqltrees against any sparql endpoints (currently any that support the (optional) SPARQL result JSON serialization format).

The `oort.sparqltree.autotree` module uses regular SPARQL queries as detailed above.

### POSIX-compliant command-line example ###

Example of using a regular query, as per the example above. To do this, you need to:

  * install `oort` (see above)
  * have a copy of the `examples/` from the repository locally (e.g. by checking it out)
  * set up a SPARQL endpoint and load it with the example data.

One way to set up a SPARQL endpoint is to use Joseki. The examples include a configuration for loading a simple example. To get this going:

  1. Download Joseki from <http://www.joseki.org/>.
  1. Unpack the distribution and define a JOSEKIROOT environment variable pointing to it.
  1. Go to the unpacked directory and run:
```
   $ chmod u+x:a bin/*
   $ bin/rdfserver <path-to-SparqlTree-examples>/data/joseki-config.ttl
```

Go to the `examples` directory and run:
```
    $ python -m oort.sparqltree.run http://localhost:2020/sparql person.rq
```