# RO-Crate ISA Context

This is a public repository for context files for the [ISA](https://github.com/ISA-tools/isa-api/tree/master/isatools/resources/schemas/isa_model_version_1_0_schemas/core) data JSON-LD file such as "ro-crate-metadata.json". 
The public urls for these files can be added into the "@context" section of JSON-LD file to expand the terms
for the ISA data. Without any changes of the data in "@graph" section, the result is that more ISA data will be linked and more triples can be extracted by the most RDF tools. 

## Background Information
The ro-crate is a great way to package all the data in a single deliverable json file. However, when ISA data
is packaged into the "ro-crate-metadata.json", schema information is missing and the context information for 
these ISA data terms do not exist. RDF parsing tools can not figure out data type and end up throw away these linkable data.

Here is an example from [W3C JSON-LD Specification](https://www.w3.org/TR/json-ld11/)
```
{
  "@context": {
    "name": "http://schema.org/name"
  },
  "name": "Manu Sporny",
  "status": "trollin'"
}
The "name" field has context definition, and is parsable. However, "status" does not expand to an IRI. It
is not Linked Data and thus ignored when processed.
```

## Simple illustrattion
To illustrate, we can use a simple "ro-crate-metadata.json" file, with and without our ISA context, 
plug it into the [JSON-LD Playground](https://json-ld.org/playground/) to see the differences of number of triples
extracted.

Here is a example ISA JSON-LD file without ISA context. To make it simple, we deleted some ro-crate nodes.
```
{
    "@context": [
        "https://w3id.org/ro/crate/1.1/context"
    ],
    "@graph": [
        {
            "@id": "https://bioconnect-ui-sqa.azurewebsites.net/search/metadata-search-browse/detail-view/investigation/269",
            "@type": "Investigation",
            "name": "JAXIN00000B",
            "title": "Transcriptome profiling of non-islet metabolic tissues in DO founder mice fed high-fat, high-sugar diet. "
        },
        {
            "@id": "https://bioconnect-ui-sqa.azurewebsites.net/search/metadata-search-browse/detail-view/study/444",
            "@type": "Study",
            "investigation": {
                "@id": "https://bioconnect-ui-sqa.azurewebsites.net/search/metadata-search-browse/detail-view/investigation/269"
            },
            "name": "JAXST00000J",
            "publicReleaseDate": "2024-01-21T13:53:08+00:00",
            "submissionDate": "2024-03-26T14:54:09+00:00",
            "title": "RNA-seq expression analysis of liver, adipose, and pancreatic islets from DO founder strain mice."
        }
    ]
}
```

Paste the file content into the [JSON-LD Playground](https://json-ld.org/playground/), we can see there are 4 triples in the graph, two for investigation and two for study.
![No ISA Context](img/no_isa_context.png "No ISA Context")


Now add our ISA context URL:
```
"https://raw.githubusercontent.com/TheJacksonLaboratory/ro-crate-isa-context/init/isa/isa_context_1_0.json"
```
to the "@context" section and everything else is the same as the previous one. Here is the entire file:

```
{
    "@context": [
        "https://w3id.org/ro/crate/1.1/context",
        "https://raw.githubusercontent.com/TheJacksonLaboratory/ro-crate-isa-context/init/isa/isa_context_1_0.json"
    ],
    "@graph": [
        {
            "@id": "https://bioconnect-ui-sqa.azurewebsites.net/search/metadata-search-browse/detail-view/investigation/269",
            "@type": "Investigation",
            "name": "JAXIN00000B",
            "title": "Transcriptome profiling of non-islet metabolic tissues in DO founder mice fed high-fat, high-sugar diet. "
        },
        {
            "@id": "https://bioconnect-ui-sqa.azurewebsites.net/search/metadata-search-browse/detail-view/study/444",
            "@type": "Study",
            "investigation": {
                "@id": "https://bioconnect-ui-sqa.azurewebsites.net/search/metadata-search-browse/detail-view/investigation/269"
            },
            "name": "JAXST00000J",
            "publicReleaseDate": "2024-01-21T13:53:08+00:00",
            "submissionDate": "2024-03-26T14:54:09+00:00",
            "title": "RNA-seq expression analysis of liver, adipose, and pancreatic islets from DO founder strain mice."
        }
    ]
}
```

Paste the file content into [JSON-LD Playground](https://json-ld.org/playground/) again, and now can see there are 9 triples
![ISA Context](img/isa_context.png "ISA Context")

## How the ISA context increases the linkable data
When a parser extracts the triples from JSON-LD, it needs to map terms to its definition, which is the context. 
Contexts can either be directly embedded into the document (an embedded context) or be a remote URL.
When I add the ISA context URL into the "@context" section of  "ro-crate-metadata.json" JSON-LD file, We added the 
schema information for these ISA data so more data will be understandable by the parser. 

Our ISA context helps ISA data parsing in three ways:
- Use uppercase ISA model objects to match the entry has the "@type" defined, such as "@type": "Study"
- Use lowercase ISA model objects to match the fields which use the lower case, such as the "investigation":"uri" in the study object
- Define context terms based ISA schema definition for the properties in ISA data object such as "submissionDate", "annotationValue"

## Experiment
To see how much more information we can get, we take 240 "ro-crate-metadata.json" files and use 
[python rdflib](https://pypi.org/project/rdflib/). We compare the triples extracted with and without our ISA context.
Here is the result: 

Number of Triples Read from rdflib

| File Name | # of Triples <br/>No ISA Context    |  # of Triples <br/>with ISA Conect    | Difference | Increase<br/>Percentage (%) |
| :----- | ---: | ---: | ---: | :---: |
|20230324_195803_574.zip|18212|23704|5492|30.2|
|20230324_195915_575.zip|26981|35289|8308|30.8|
|20230324_202404_578.zip|15221|21997|6776|44.5|
|20230324_202405_579.zip|1005|1469|464|46.2|
|20230324_202459_580.zip|105479|118085|12606|12.0|
|20230324_202505_581.zip|348|512|164|47.1|
|20230324_202526_582.zip|38098|46996|8898|23.4|
|20230324_202552_583.zip|348|512|164|47.1|
|20230324_202627_584.zip|6085|7930|1845|30.3|
|20230324_203059_585.zip|284|464|180|63.4|

Note: only first 10 rows list out here, see [entire file](data/isa_context.csv) for all rows

On average, there is 99.7% increase in the number of triples we can get from the json files, which
mean we can get double amount of triples with our ISA context

## How to use it
The only thing need to do is just to insert the ISA context URL into the "@context" section, the same level as "https://w3id.org/ro/crate/1.1/context".

For [python rocrate ](https://pypi.org/project/rocrate/) to generate the ro-crate json, just need to add the extra terms in the metadata like this
```
crate.metadata.extra_terms = "https://raw.githubusercontent.com/TheJacksonLaboratory/ro-crate-isa-context/init/isa/isa_context_1_0.json"
```
the rocrate lib will add the context in the generated json file

## Expanded the Context
The "@context" section in "ro-crate-metadata.json" file could be a array of context.  The parser will aggregate all of them and apply to the data.
So you can define any custom term you need and add additional URLs to point the context
