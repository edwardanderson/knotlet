# Knotlet

- [Knotlet](#knotlet)
  - [Example](#example)
  - [Specification](#specification)
    - [JSON-LD Considerations](#json-ld-considerations)

Knotlet is a tiny language for writing RDF graphs. It's designed to be easy to write, in a similar way to how [KRML](https://github.com/edwardanderson/krml) is intended to be easy to read.

Knotlet supports RDF conventions for CURIEs, literal refification, languages, datatypes and sequences.

## Example

This example demonstrates almost all the features of the language.

```text
.
  name
    > John Lennon
  a
    schema:Person
  date of birth
    > 1940-10-09 |xsd:date
  foaf:knows
    Paul
    George
    Ringo
  mother
    .Julia
  half-sister
    .Julia
  father
    .
      foaf:nick
        > Freddie
  schema:spouse
    - http://www.wikidata.org/entity/Q253192
      name
        > Cynthia
    - :Yoko
      name
        > 小野 洋子 |jp
```

This is equivalent to the following JSON-LD:

```json
{
  "@context": [
    "https://prefix.cc/context",
    {
      "@base": "http://example.org/",
      "@vocab": "http://example.org/",
      "_content": "rdf:value",
      "_label": "rdfs:label"
    }
  ],
  "name": [
    {
      "@value": "John Lennon"
    }
  ],
  "@type": "schema:Person",
  "date_of_birth": [
    {
      "@value": "1940-10-09",
      "@type": "xsd:date"
    }
  ],
  "foaf:knows": [
    {
      "@id": "_:Paul",
      "_label": "Paul"
    },
    {
      "@id": "_:George",
      "_label": "George"
    },
    {
      "@id": "_:Ringo",
      "_label": "Ringo"
    }
  ],
  "mother": [
    {
      "_label": "Julia"
    }
  ],
  "father": [
    {
      "foaf:nick": "Freddie"
    }
  ],
  "half-sister": [
    {
      "_label": "Julia"
    }
  ],
  "schema:spouse": {
    "@list": [
      {
        "@id": "http://www.wikidata.org/entity/Q253192",
        "name": [
          {
            "@value": "Cynthia"
          }
        ]
      },
      {
        "@id": "Yoko",
        "_label": "Yoko",
        "name": [
          {
            "@value": "小野 洋子",
            "@language": "jp"
          }
        ]
      }
    ]
  }
}
```

## Specification

| Knotlet           | Turtle                 | Description                               |
|-                  |-                       |-                                          |
| `.`               | `[]`                   | Anonymous resource                        |
| `.abc`            | `[ rdfs:label "abc" ]` | Anonymous labelled resource               |
| `abc`             | `_:abc`                | Anonymous named resource                  |
| `:abc`            | `:abc` `<abc>`         | Base resource                             |
| `http://...`      | `<http://...>`         | Globally identified resource              |
| `ex:abc`          | `ex:abc`               | Globally recognised resource              |
| `a`               | `a` `rdf:type`         | Resource is an instance of class          |
| `> abc`           | `"abc"`                | Literal                                   |
| `> abc \|en`      | `"abc"@en`             | Literal with language tag                 |
| `> 123 \|xsd:int` | `"123"^^xsd:int`       | Literal with globally recognised datatype |
| `- abc`           | `( _:abc )`            | Sequence                                  |

- Significant whitespace indentation relates resources vertically
- One resource per line
- A globally recognised resource is a [CURIE](https://en.wikipedia.org/wiki/CURIE) which uses a [prefix.cc](https://prefix.cc/) key
- Literals with outgoing relationships are automatically reified

  ```
  > Foo
    relationship
      Bar
  ```

  ```turtle
  @prefix : <http://example.org/> .
  @prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
  @prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

  [] rdf:value "Foo" ;
    :relationship _:Bar .

  _:Bar rdfs:label "Bar" .
  ```

### JSON-LD Considerations

- Predicate values are always arrays
- Predicate terms are santised
- Identifiers are encoded
- `rdfs:label` is aliased as `_label`
