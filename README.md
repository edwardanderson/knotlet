# Knotlet

- [Knotlet](#knotlet)
  - [Example](#example)
  - [Specification](#specification)
    - [Syntax](#syntax)
    - [Predicate Operators](#predicate-operators)
      - [Inverse](#inverse)
      - [Symetrical](#symetrical)
      - [Union](#union)
    - [JSON-LD Considerations](#json-ld-considerations)

Knotlet is a tiny language for writing RDF graphs. It's designed to be easy to write, in a similar way to how [KRML](https://github.com/edwardanderson/krml) is intended to be easy to read.

Knotlet supports RDF conventions for CURIEs, literal reification, languages, datatypes and sequences.

## Example

```text
John
  a
    schema:Person
  date of birth
    > 1940-10-09 |xsd:date
  foaf:knows |union
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
  "@id": "_:John",
  "_label": "John",
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
      "_label": "Paul",
      "foaf:knows": [
        {
            "@id": "_:John"
        },
        {
            "@id": "_:George"
        },
        {
            "@id": "_:Ringo"
        }
      ]
    },
    {
      "@id": "_:George",
      "_label": "George",
      "foaf:knows": [
        {
            "@id": "_:John"
        },
        {
            "@id": "_:Paul"
        },
        {
            "@id": "_:Ringo"
        }
      ]
    },
    {
      "@id": "_:Ringo",
      "_label": "Ringo",
      "foaf:knows": [
        {
            "@id": "_:John"
        },
        {
            "@id": "_:Paul"
        },
        {
            "@id": "_:George"
        }
      ]
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

### Syntax

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

### Predicate Operators

Predicate operators are syntactic sugar for materialising inverse, symetrical and product relationships.

#### Inverse

The `|inverse` token materialises a relationship between object and subject resources.

```
The Beatles
  memberOf |inverse
    John
    Paul
    George
    Ringo
```

```turtle
@prefix : <http://example.org/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

_:The_Beatles rdfs:label "The Beatles" .

[] rdfs:label "John" ;
  :memberOf _:The_Beatles .

[] rdfs:label "Paul" ;
  :memberOf _:The_Beatles .

[] rdfs:label "George" ;
  :memberOf _:The_Beatles .

[] rdfs:label "Ringo" ;
  :memberOf _:The_Beatles .
```

#### Symetrical

The `|symetrical` token materialises a relationship between both the subject and object, and the object and subject.

```
John
  knows |symetrical
    Paul
    George
    Ringo
```

```turtle
@prefix : <http://example.org/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

_:John rdfs:label "John" ;
  :knows _:Paul , _:George , _:Ringo .

_:Paul rdfs:label "Paul" ;
  :knows _:John .

_:George rdfs:label "George" ;
  :knows _:John .

_:Ringo rdfs:label "Ringo" ;
  :knows _:John .
```

#### Union

The `|union` token materialises irreflexive symetrical relationships between members of the union of subject and object resources.

```
John
  knows |union
    Paul
    George
    Ringo
```

```turtle
@prefix : <http://example.org/> .
@prefix rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
@prefix rdfs: <http://www.w3.org/2000/01/rdf-schema#> .

_:John rdfs:label "John" ;
  :knows _:Paul , _:George , _:Ringo .

_:Paul rdfs:label "Paul" ;
  :knows _:John , _:George , _:Ringo .

_:George rdfs:label "George" ;
  :knows _:John , _:Paul , _:Ringo .

_:Ringo rdfs:label "Ringo" ;
  :knows _:John , _:Paul , _:George .
```

### JSON-LD Considerations

- Predicate values are always arrays
- Predicate terms are santised
- Identifiers are encoded
- `rdfs:label` is aliased as `_label`
