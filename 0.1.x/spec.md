# Profile/Module : RO-Crate Extensibility Profile

AKA: Convention to reference schemas and metadata outside `ro-crate-metadata.json`

## Index
- [Terms](#terms)
- [Motivation](#motivation)
- [Goals](#goals)
- [Specification](#specification)
- [Reference Example](#reference-example)
    - [Reference Example as json-rdf](#reference-example-as-json-rdf)
    - [Reference Example as csvw](#reference-example-as-csvw)
- [People](#people)

## Terms
* Schema: A description of the shape of our metadata. This proposal doesn't cover the shape of the data itself.
* Metadata: Entries following the shape indicated by the schema.

## Motivation

We are working on multiple integration projects between systems, including ELNs, LIMSs, analysis software and repositories.
Of particular interest is the exchange object graphs extracted from one system to another. 
Each existing system works natively with its own schema and metadata format.

The naive approach of integrating every system with every other system would lead to a large number of required integrations, growing with `n^2`
. To address this, we want a general interchange format so every system only has to write a single
adapter and is then able to interoperate with all other compliant systems.

The current Ro-Crate `ro-crate-metadata.json` does not have a way to specify withing itself dynamic schemas with features like: 
* [Class](https://www.w3.org/TR/rdf-schema/#ch_classes)
* [Semantic Class](https://www.w3.org/TR/owl-ref/#equivalentClass-def)
* [Property](https://www.w3.org/TR/rdf-schema/#ch_property)
* [Semantic Property](https://www.w3.org/TR/owl-ref/#equivalentProperty-def)
* [Data Type](https://www.w3.org/TR/rdf-schema/#ch_datatype)

The conclusion is that to use RO-Crate to both package datasets and communicate between systems the current `ro-crate-metadata.json` needs to be extended.

## Goals

The main aim is to provide a rich RO-Crate that is machine actionable.
To this end, the schema of the metadata in the RO-Crate plus the ontologies used have to be included in a standardized way.
Ontological information allows mapping of objects with different syntax but matching semantics. 
This should be both close to existing ways of using RO-Crates and built on established concepts. 

The most important points are:
1. including machine actionable schema for the metadata
2. including machine actionable metadata
3. including machine actionable ontological annotations on the schema
4. staying close to established use RO-Crate
5. bundling referenced ontologies to prevent link rot, ideally


## Specification

This profile focuses on how to indicate that potential schema or metadata file is referenced from the `ro-crate-metadata.json` but not included.


RO-Crate indicates it references on the context part of the `ro-crate-metadata.json`.

```json
{
  "@context": ["https://w3id.org/ro/crate/1.1/context"],
  "@graph": [
    {
      "@id": "./",
      "@type": "Dataset",
      ...
    }
  ]
}
```

Is possible to include local files is uris following [File URI scheme](https://en.wikipedia.org/wiki/File_URI_scheme).

```json
{
  "@context": [
    "https://w3id.org/ro/crate/1.1/context",
    "file:///metadata/my-rdf-schema.json",
    "file:///metadata/my-rdf-metadata.json",
  ], "@graph": [
    {
      "@id": "./",
      "@type": "Dataset",
      ...
    }
  ]
}
```

By just looking at the `ro-crate-metadata.json` context is not possible to know the format of the referenced files.

Ideally included json files have a `@context` themselves.

The `@context` can be used to detect special formats.

The absence of `@context` could indicate that there is no further extension to the one indicated in `ro-crate-metadata.json`. 

## Reference example

This example specifies two types:
- Person
- Experiment

These show what the metadata schema looks like and how references are handled in the formats.
To illustrate this in a widespread format, here's the example in SQL

```sql
CREATE TABLE person(
    personid VARCHAR(255),
    givenname VARCHAR(255),
    familyname VARCHAR(255),
    identifier VARCHAR (255),
    PRIMARY KEY(personid)


);

CREATE TABLE experiment(
    experimentid VARCHAR(255),
    date DATETIME,
    name VARCHAR(255),
    creatorid VARCHAR(255)
    PRIMARY_KEY(experimentid),
    FOREIGN KEY(creatorid) REFERENCES person(personid)
);

```
The actual metadata looks like this:

**TABLE person**

| personid                           | firstname | lastname | identifier                            |
|------------------------------------|-----------|----------|---------------------------------------|
| /ANDREAS/RO-CRATE-EXAMPLE/PERSON14 | Andreas   | Meier    | https://orcid.org/0009-0002-6541-4637 |

**TABLE experiment**

| experimentid                           | date                       | name          | creatorid                                                            |
|----------------------------------------|----------------------------|---------------|----------------------------------------------------------------------|
| /ANDREAS/RO-CRATE-EXAMPLE/EXPERIMENT13 | 2023-04-01T05:00:30.001000 | My Experiment | /ANDREAS/RO-CRATE-EXAMPLE/PERSON14/ANDREAS/RO-CRATE-EXAMPLE/PERSON14 |

### Reference example as json-rdf

On this example two additional json files referenced on the context.

Since these files follow [json-rdf format](https://github.com/ethz-sis/ro-crate-interoperability-profile) conventions, systems supporting such conventions should be able to process these files as part of the `ro-crate-metadata.json`.

`ro-crate-metadata.json`
```json
{
  "@context": [
    "https://w3id.org/ro/crate/1.1/context",
    "file:///metadata/my-rdf-schema.json",
    "file:///metadata/my-rdf-metadata.json",
  ], "@graph": [
    {
      "@id": "./",
      "@type": "Dataset",
      ...
    }
  ]
}
```

`file:///metadata/my-rdf-schema.json`

```json
{
  "@context": "https://github.com/ethz-sis/ro-crate-interoperability-profile",
  "@graph":
  [
    {
      "@type": "rdfs:Class",
      "@id": "EXPERIMENT",
      "owl:restriction": [
        {
          "@id": "a6390bfc-3c78-4e5d-a0cf-2c905afc3cc7"
        },
        {
          "@id": "e79a5c51-ce58-4a62-80ea-97cc177f11a5"
        },
        {
          "@id": "17183bf6-c583-400a-9950-c9936ad0de8a"
        }
      ],
      "rdfs:subClassOf": {
        "@id": ":Object"
      }
    },
    {
      "@type": "rdfs:Class",
      "@id": "PERSON",
      "owl:restriction": [
        {
          "@id": "8605eb32-65ab-449e-a43d-db66981c8a26"
        },
        {
          "@id": "6b30978b-f8f5-4bd1-8cba-399969e13ae6"
        },
        {
          "@id": "a26c3f82-8e5c-4e23-bbab-128579df601f"
        }
      ],
      "owl:equivalentClass": {
        "@id": "https://schema.org/Person"
      }
    },
    {
      "@type": "rdfs:Property",
      "@id": "openBIS:hasPERSON.IDENTIFIER",
      "rdfs:label": "PersonIdentifier",
      "rdfs:comment": "PersonIdentifier",
      "schema:rangeIncludes": {
        "@id": "xsd:string"
      },
      "schema:domainIncludes": {
        "@id": "PERSON"
      },
      "owl:equivalentProperty": {
        "@id": "https://schema.org/Identifier"
      }
    },
    {
      "@type": "rdfs:Property",
      "@id": "openBIS:hasPERSON.GIVENNAME",
      "rdfs:label": "Given name",
      "rdfs:comment": "Given name",
      "schema:rangeIncludes": {
        "@id": "xsd:string"
      },
      "schema:domainIncludes": {
        "@id": "PERSON"
      },
      "owl:equivalentProperty": {
        "@id": "https://schema.org/GivenName"
      }
    },
      {
      "@type": "rdfs:Property",
      "@id": "openBIS:hasEXPERIMENT.NAME",
      "rdfs:label": "Name",
      "rdfs:comment": "Name",
      "schema:rangeIncludes": {
        "@id": "xsd:string"
      },
      "schema:domainIncludes": {
        "@id": "EXPERIMENT"
      }
    },
      {
      "@type": "rdfs:Property",
      "@id": "openBIS:hasPERSON.FAMILYNAME",
      "rdfs:label": "Family Name",
      "rdfs:comment": "Family Name",
      "schema:rangeIncludes": {
        "@id": "xsd:string"
      },
      "schema:domainIncludes": {
        "@id": "PERSON"
      },
      "owl:equivalentProperty": {
        "@id": "https://schema.org/FamilyName"
      }
    },
    {
      "@type": "rdfs:Property",
      "@id": "openBIS:hasEXPERIMENT.CREATOR",
      "rdfs:label": "Creator",
      "rdfs:comment": "Creator",
      "schema:rangeIncludes": {
        "@id": "PERSON"
      }
    },
    {
      "@type": "owl:restriction",
      "@id": "8605eb32-65ab-449e-a43d-db66981c8a26",
      "owl:onProperty": {
        "@id": "openBIS:hasPERSON.GIVENNAME"
      },
      "owl:minCardinality": 0,
      "owl:maxCardinality": 1
    },
    {
      "@type": "owl:restriction",
      "@id": "6b30978b-f8f5-4bd1-8cba-399969e13ae6",
      "owl:onProperty": {
        "@id": "openBIS:hasPERSON.FAMILYNAME"
      },
      "owl:minCardinality": 0,
      "owl:maxCardinality": 1
    },
    {
      "@type": "owl:restriction",
      "@id": "a26c3f82-8e5c-4e23-bbab-128579df601f",
      "owl:onProperty": {
        "@id": "openBIS:hasPERSON.IDENTIFIER"
      },
      "owl:minCardinality": 0,
      "owl:maxCardinality": 1
    },
    {
      "@type": "owl:restriction",
      "@id": "a6390bfc-3c78-4e5d-a0cf-2c905afc3cc7",
      "owl:onProperty": {
        "@id": "openBIS:hasEXPERIMENT.NAME"
      },
      "owl:minCardinality": 0,
      "owl:maxCardinality": 1
    },
    {
      "@type": "owl:restriction",
      "@id": "e79a5c51-ce58-4a62-80ea-97cc177f11a5",
      "owl:onProperty": {
        "@id": "openBIS:hasEXPERIMENT.DATE"
      },
      "owl:minCardinality": 0,
      "owl:maxCardinality": 1
    },
    {
      "@type": "owl:restriction",
      "@id": "17183bf6-c583-400a-9950-c9936ad0de8a",
      "owl:onProperty": {
        "@id": "openBIS:hasEXPERIMENT.CREATOR"
      },
      "owl:minCardinality": 0,
      "owl:maxCardinality": 1
    }
  ]
}
```

`file:///metadata/my-rdf-metadata.json`

```json
{
  "@context": "https://github.com/ethz-sis/ro-crate-interoperability-profile",
  "@graph":
  [
    {
      "@type": "PERSON",
      "@id": "/ANDREAS/RO-CRATE-EXAMPLE/PERSON14",
      "openBIS:hasPERSON.IDENTIFIER": "https://orcid.org/0009-0002-6541-4637",
      "openBIS:hasPERSON.FAMILYNAME": "Meier",
      "openBIS:hasPERSON.GIVENNAME": "Andreas",
      "openBIS:hasSPACE": {
        "@id": "ANDREAS"
      },
      "openBIS:hasPROJECT": {
        "@id": "/ANDREAS/RO-CRATE-EXAMPLE"
      },
      "openBIS:hasCOLLECTION": {
        "@id": "/ANDREAS/RO-CRATE-EXAMPLE/RO-CRATE-EXAMPLE_EXP_2"
      }
    },
    {
      "@type": "EXPERIMENT",
      "@id": "/ANDREAS/RO-CRATE-EXAMPLE/EXPERIMENT15",
      "openBIS:hasEXPERIMENT.NAME": "My Experiment",
      "openBIS:hasEXPERIMENT.DATE": "2025-04-16 08:57:39 +0000",
      "openBIS:hasSPACE": {
        "@id": "ANDREAS"
      },
      "openBIS:hasEXPERIMENT.CREATOR": {
        "@id": "/ANDREAS/RO-CRATE-EXAMPLE/PERSON14"
      }
    }
  ]
}
```

### Reference Example as csvw

On this example two additional json files referenced on the context.

Since these files follow [CSVW](https://csvw.org/) conventions, systems supporting such conventions should be able to process these files as part of the `ro-crate-metadata.json`, systems that doesn't could just skip them.

`ro-crate-metadata.json`
```json
{
  "@context": [
    "https://w3id.org/ro/crate/1.1/context",
    "person-schema.json",
    "experiment-schema.json",
  ], "@graph": [
    {
      "@id": "./",
      "@type": "Dataset",
      ...
    }
  ]
}
```

##### Described Metadata

CSVW enriches CSV files with additional metadata that describe their schema and ontologies. 
The contents of the CSV files are human-readable. 
Metadata and schema may be referenced together, e.g.

`person.schema.json`
```json
{
    "@context": "http://www.w3.org/ns/csvw",
    "rdfs:label": "Person Table",
    "propertyUrl": "https://schema.org/Person",
    "tableSchema": {
        "columns": [
            {
                "datatype": "string",
                "propertyUrl": "https://schema.org/givenName",
                "name": "PERSON.GIVENNAME"
            },
            {
                "datatype": "string",
                "propertyUrl": "https://schema.org/familyName",
                "name": "PERSON.FAMILYNAME"
            },
            {
                "datatype": "string",
                "propertyUrl": "https://schema.org/identifier",
                "name": "PERSON.IDENTIFIER"
            },
            {
                "datatype": "string",
                "name": "PRIMARYKEY"
            }
        ]
    },
    "url": "person-data.csv"
}

```

`person-data.csv`
```csv
PERSON.GIVENNAME,PERSON.FAMILYNAME,PERSON.IDENTIFIER,PRIMARYKEY
Andreas,Meier,https://orcid.org/0009-0002-6541-4637,/ANDREAS/RO-CRATE-EXAMPLE/PERSON14
```

`experiment-schema.json`
```json
{
    "@context": "http://www.w3.org/ns/csvw",
    "rdfs:label": "Experiment Table",
    "tableSchema": {
        "columns": [
            {
                "datatype": "string",
                "name": "PRIMARYKEY"
            },
            {
                "datatype": "dateTime",
                "name": "EXPERIMENT.DATE"
            },
            {
                "datatype": "string",
                "name": "CREATOR"
            }
        ],
        "foreignKeys": [
            {
                "columnReference": [
                    "CREATOR"
                ],
                "reference": {
                    "resource": "person.csv",
                    "columnReference": [
                        "PRIMARYKEY"
                    ]
                }
            }
        ]
    },
    "url": "experiment-data.csv"
}
```

`experiment-data.csv`
``` csv
PRIMARYKEY,EXPERIMENT.DATE,EXPERIMENT.CREATOR
/ANDREAS/RO-CRATE-EXAMPLE/EXPERIMENT13,2023-04-01T05:00:30.001000,/ANDREAS/RO-CRATE-EXAMPLE/PERSON14
```

## People

- Andreas Meier (andreas.meier@ethz.ch)
- Juan Fuentes (juan.fuentes@id.ethz.ch)
