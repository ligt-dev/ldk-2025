# Experiments for the LDK-2025 paper "Ligt: Towards an Ecosystem for Managing Interlinear Glossed Texts with Linguistic Linked Data"

## Introduction

The paper contains very brief information on the SPARQL queries in different configurations, i.e.:
 1. Querying a Fuseki endpoint
 2. Querying via Comunica SPARQL engine
 3. Converting data on the fly and querying via Comunica SPARQL engine

In each scenario, the following CLDF datasets are queried:
 * [Atlas of Pidgin and Creole Languages](https://apics-online.info/)
 * [Cross-linguistic differential and optional marking database](https://github.com/cldf-datasets/levshinadifferentialmarking)
 * Examples extracted from the feature descriptions of [GramBank](https://grambank.clld.org/), extracted with `engligten`, see below.

 Total amount of utterances: 18,983; triples: 994,138.

### Querying a Fuseki endpoint

 * The endpoint is installed on a [`t2.small` Amazon EC2 instance](https://aws.amazon.com/ec2/instance-types/)
 * Ubuntu 20.04.6
 * Server is available in the EU-Central-1 zone (Germany)
 * Apache Fuseki version is 3.17.0
 * Accessed with SPARQLWrapper

### Querying via Communica SPARQL engine

 * Launched on a local machine, 12th Gen Intel® Core™ i7-1255U × 12, 16GB RAM
 * Ubuntu 24.02
 * Comunica version is 4.1.0
 * Accessed via command line

### Converting data on the fly and querying via Comunica SPARQL engine

 * The setup is identical to [[Querying via Comunica SPARQL engine]] but instead of specifying a local file, Ligt is being generated on the fly using `enligten` — a light-weight server working over _ligttools_ — a converter for Ligt.

### `enligten`

`enligten`: a light-weight server that is capable (in a very limited way) of scraping IGT from web sites. Currently, it is tuned to extracting information from CLDF typological databases like [GramBank](https://grambank.clld.org/parameters/GB020#2/21.0/151.7).
In the near future the tool will be published as part of the _ligttools_ suite.

## Queries

In the paper, we provided the results for 3 queries of different complexity:
 
 1. A simple query looking for all labels with a certain gloss ("woman"@en)
 2. A slightly more complex query with a regular expression, looking for all possible surface realisations of a causative grammatical category
 3. Finally, an even more complex query with property paths and links to external datasets

## Query 1

```sparql
PREFIX ligt: <http://purl.org/ligt/ligt-0.3#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT (group_concat(DISTINCT ?tr; separator=" | ") as ?tr) ?lang ?doc
{
	?s a ligt:Morph ;
	   rdfs:label ?tr ;
	   ligt:gloss "woman"@en .

	   BIND(LANG(?tr) AS ?lang)
} GROUP BY ?lang ?doc LIMIT 100
```

## Query 2
```sparql
PREFIX ligt: <http://purl.org/ligt/ligt-0.3#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>

SELECT DISTINCT (group_concat(?gram; separator=" | ") as ?grams) ?morph_val ?lang ?doc
{
	?doc a ligt:Document ;
	     ligt:hasUtterances/ligt:utterance/ligt:hasMorphs/ligt:item ?s .
	?s a ligt:Morph ;
	   rdfs:label ?gram ;
	   ligt:gloss ?morph_val .

	   BIND(LANG(?gram) AS ?lang)
	   FILTER(REGEX(STR(?morph_val), "CAUS"))
} GROUP BY ?lang ?doc ?morph_val LIMIT 100
```

## Query 3
```sparql
PREFIX ligt: <http://purl.org/ligt/ligt-0.2#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX apics: <http://purl.org/liodi/ligt/apics/>
PREFIX rdf: <http://www.w3.org/1999/02/22-rdf-syntax-ns#>
PREFIX unimorph: <http://purl.org/olia/unimorph.owl#>

SELECT (GROUP_CONCAT(DISTINCT ?case; SEPARATOR=", ") AS ?cases) ?lang
WHERE {

?morph ligt:gloss ?case ;
rdfs:label ?label .

?tag owl:sameAs ?case .

?tag rdfs:subClassOf+ unimorph:Case .

BIND(LANG(?label) as ?lang)
FILTER(?lang != '')
} GROUP BY ?lang
```
