## sqlqs - sql query strings

[![Build Status](https://travis-ci.org/kissmygritts/sqlqs.svg?branch=master)](https://travis-ci.org/kissmygritts/sqlqs)

[![Coverage Status](https://coveralls.io/repos/github/kissmygritts/sqlqs/badge.svg?branch=master)](https://coveralls.io/github/kissmygritts/sqlqs?branch=master)

A very simple and rudimentary query string to SQL predicate parser.

Heavily influenced by the query string methods from the PostgREST API server, this module will create SQL WHERE clause predicates from the query string. With PostgREST the query string handles almost all of the database filtering. For instance, to filter the database the query string may look like the following, `?x=eq.10`. Where x is the column to filter, `eq` is the operator to use, and `10` is the criteria to filter. I call these a predicate object. I found this query string structure very flexible and decided to try and recreate it with Node.

## Use

``` javascript
const { parse, sqlize, where } = require('sqlqs')

let qs = {
  class: 'in.Mammal,Bird',
  genus: 'eq.Neotoma',
  id: 'gt.1000'
}

// parse query string into query object
const QueryObj = parse(qs)

// returns
[
  {
    column:"class",
    operator:"IN",
    criteria:["Mammal","Bird"]
  }, {
    column:"genus",
    operator:"=",
    criteria:"Neotoma"
  }, {
    column:"id",
    operator:">",
    criteria:"1000"
  }
]

// parse query object into a WHERE clause
sqlize(QueryObj)

// returns
"class IN ('Mammal','Bird') AND genus = 'Neotoma' AND id > 1000"

// where pipes these two methods together
where(qs)

// returns
"class IN ('Mammal','Bird') AND genus = 'Neotoma' AND id > 1000"
```

[Try it on RunKit](https://runkit.com/kissmygritts/sqlqs)

## Caveats

This will be helpful for writing an API using database tools like pg-promise, pg, etc. This will not be useful for an ORM.

A query string like this `?species=eq.arctos&species=eq.americanus` is attempting to return all records where species is arctos or americanus. However it creats this query `SELECT * FROM animals where species = 'arctos' AND species = 'americanus'`. While this is a valid SQL string, it will not return any records. The query string should be `?species=in.arctos,americanus` which will create this SQL query `SELECT * FROM animals WHERE species IN ('arctos','americanus')`.

sqlqs assumes that numbers provided in the query string map to fields with number datatypes in the database. I can for see this being an issue if numbers with leading zeros are being stored in a database as a text datatype.