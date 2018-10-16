TOC
============

 + ES istallazione [~10 min]
	- note generali sul cluster e sugli elementi necessari ad una configurazione di base
	- approccio "hands on" su [demo online](https://demo.elastic.co/app/kibana):
		- mostriamo come sono fatti di solito i documenti velocemente, serve solo a dare una idea di cosa guardare dopo con attenzione
		- mostriamo che tutto può essere visto in SQL velocemente, serve solo ad introdurre il tema del riuso su tool noti o con logiche note
 + ES base [~30 min]
	- aggiunta documenti da esempi (ex: books.json)
	- query semplici di esempio via http
	- review mapping generato 
	- review settings generato
 + GUI
	 + Cerebro setup
	 + Cluster Health + Overwiev
	 + Query via REST
 + bonus: ES SQL (kibana + XPack)
 + lucene
	- concetti: Document, Field, Token
	- concetti: Analyzer, Filter, TokenFilter, CharFilter
	- concetti: Inverted Index, DF/ITF, Vector Space Model
	- concetti: Similarity, etc
	- tipologie di ricerca
		- boolean
		- fuzzy
	- analyzer in fase di indexing o di query
 + ES semplice
	- creazione indice ad hoc
		- settings ad-hoc
		- mapping ad-hoc
			- senza analyzer, e/o con Keyword
			- analyzer simile a quello di default, ma creato esplicitamente concatenando i componenti di base
			- con sinonimi
			- con stopwords
	- indicizzazione esempi (ex: books.json o altri con parte geografica) -> 	  Visualizzazione tramite Kibana?
	- review vantaggi in termini di query
 + Esempi Pratici
	 + Indice Regione Veneto
	 + Indice CRAIM
 + ES più complicato
	- stemming
	- language detect
	- etc....
 + altro...
	
 + * *

## references: 

+ analysis
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis.html
+ analyzer
https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-custom-analyzer.html
+ SQL
https://www.elastic.co/guide/en/elasticsearch/reference/current/sql-getting-started.html
	- https://www.elastic.co/blog/an-introduction-to-elasticsearch-sql-with-practical-examples-part-1
	- https://www.elastic.co/blog/an-introduction-to-elasticsearch-sql-with-practical-examples-part-2
+ term vectors - Eviterei
https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-termvectors.html
+ multitermvectors - Eviterei
https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-multi-termvectors.html



* * *

# TODO: esempi base presi dalle guide, da espandere e personalizzare


### example: bulk indexing of simple books data

```bash
curl -X PUT "localhost:9200/library/book/_bulk?refresh" -H 'Content-Type: application/json' -d'
{"index":{"_id": "Leviathan Wakes"}}
{"title": "Leviathan Wakes", "author": "James S.A. Corey", "release_date": "2011-06-02", "page_count": 561}
{"index":{"_id": "Hyperion"}}
{"title": "Hyperion", "author": "Dan Simmons", "release_date": "1989-05-26", "page_count": 482}
{"index":{"_id": "Dune"}}
{"title": "Dune", "author": "Frank Herbert", "release_date": "1965-06-01", "page_count": 604}
'
```

### example: SQL query from default example data
```bash
curl -X POST "localhost:9200/_xpack/sql?format=txt" -H 'Content-Type: application/json' -d'
{
    "query": "SELECT * FROM library WHERE release_date < \u00272000-01-01\u0027"
}
'
```

NOTA: possiamo usare i formati csv, json, txt

### example: another SQL query
```bash
curl -XPOST "https://632af92a492e459ea456f9acb6cf0d83.us-central1.gcp.cloud.es.io:9243/_xpack/sql/translate" -H 'Content-Type: application/json' -d'
{
  "query":"SELECT OriginCityName, DestCityName FROM flights WHERE FlightTimeHour > 5 AND OriginCountry=\"US\" ORDER BY FlightTimeHour DESC LIMIT 10"
}'
```

### TODO: further explaination using these ones:

SEE: http://localhost:9200/library/book/Dune
SEE: http://localhost:9200/library/book/Dune/_termvectors

### example: review of settings and mappings automatically generated from documents
SEE: http://localhost:9200/library/book/_mapping
SEE: http://localhost:9200/library/_settings


### example: termverctors of a single document
```bash
curl -X GET "localhost:9200/library/book/_termvectors" -H 'Content-Type: application/json' -d'
{
  "doc" : {
    "title" : "Il Nome della Rosa",
    "author" : "Umberto Eco"
  },
  "fields": ["fullname"],
  "per_field_analyzer" : {
    "author": "keyword",
	"title": "hello_analyzer"
  }
}
'
```
### example: how to change the analyzer of an existing index?

```bash
curl -X POST "localhost:9200/library/_close"
curl -X PUT "localhost:9200/library/_settings" -H 'Content-Type: application/json' -d'
{
  "analysis" : {
    "analyzer":{
      "hello_analyzer":{
        "type":"custom",
        "tokenizer":"whitespace",
		"filter": [
			"lowercase"
		]
      }
    }
  }
}
'
curl -X POST "localhost:9200/library/_open"
```
NOTA: possiamo cambiare impostazioni solo se non impattano su documenti già indicizzati. Come creare indice da zero?


### example: delete an index
```
curl -X DELETE "http://localhost:9200/library/"
```

### example: create an index
```
curl -X PUT 'http://localhost:9200/library'
```

### example: create an index with specific settings
TODO: settings

### example: get settings ofr an index
```
curl -X GET "http://localhost:9200/library/_settings" -H 'Content-Type: application/json'
```

### example: various common search
```
curl -X GET 'http://localhost:9200/library/_search?pretty=true&q=*'
curl -X GET 'http://localhost:9200/library/_search?pretty=true&q=Dune'
curl -X GET 'http://localhost:9200/library/_search?pretty=true&q=*her*'
curl -X GET 'http://localhost:9200/library/_search?pretty=true&q=*bert*'
```

### example: explaination of simple query

```
http://localhost:9200/twitter/_search?explain=true&q=twitter
```

### example: assign a mapping to an index
```
curl -X PUT 'http://localhost:9200/library/book/_mapping' -H 'Content-Type: application/json' -d'
{
	"properties": {
		"author": {
			"type": "text",
			"fields": {
				"keyword": {
					"type": "keyword",
					"ignore_above": 256
				}
			}
		},
		"page_count": {
			"type": "long"
		},
		"release_date": {
			"type": "date"
		},
		"title": {
			"type": "text",
			"analyzer": "hello_analyzer"
		}
	}
}
'
```

----

## example: explicit term vectors

```
curl -X PUT "localhost:9200/twitter/" -H 'Content-Type: application/json' -d'
{ "mappings": {
    "_doc": {
      "properties": {
        "text": {
          "type": "text",
          "term_vector": "with_positions_offsets_payloads",
          "store" : true,
          "analyzer" : "fulltext_analyzer"
         },
         "fullname": {
          "type": "text",
          "term_vector": "with_positions_offsets_payloads",
          "analyzer" : "fulltext_analyzer"
        }
      }
    }
  },
  "settings" : {
    "index" : {
      "number_of_shards" : 1,
      "number_of_replicas" : 0
    },
    "analysis": {
      "analyzer": {
        "fulltext_analyzer": {
          "type": "custom",
          "tokenizer": "whitespace",
          "filter": [
            "lowercase"
          ]
        }
      }
    }
  }
}
'

curl -X PUT "localhost:9200/twitter/_doc/1" -H 'Content-Type: application/json' -d'
{
  "fullname" : "John Doe",
  "text" : "twitter test test test "
}
'
curl -X PUT "localhost:9200/twitter/_doc/2" -H 'Content-Type: application/json' -d'
{
  "fullname" : "Jane Doe",
  "text" : "Another twitter test ..."
}
'
curl -X PUT "localhost:9200/twitter/_doc/3" -H 'Content-Type: application/json' -d'
{
  "fullname" : "Mario Rossi",
  "text" : "ma come si usa twitter?"
}
'
curl -X GET "localhost:9200/twitter/_doc/1/_termvectors?pretty=true" -H 'Content-Type: application/json' -d'
{
  "fields" : ["text"],
  "offsets" : true,
  "payloads" : true,
  "positions" : true,
  "term_statistics" : true,
  "field_statistics" : true
}
'

curl -X POST "localhost:9200/twitter/_doc/_mtermvectors?pretty=true" -H 'Content-Type: application/json' -d'
{
    "ids" : ["1", "2", "3"]
}
'
```

* * *

# TODO: da guardare per esempi

## datasets

+ https://appbaseio-apps.github.io/booksearch-onboarding/
+ https://github.com/appbaseio-apps/booksearch
+ https://github.com/zygmuntz/goodbooks-10k
+ https://github.com/uchidalab/book-dataset
+ http://www2.informatik.uni-freiburg.de/~cziegler/BX/

## lucene + spark

+ https://github.com/zouzias/spark-lucenerdd
+ https://github.com/zouzias/spark-lucenerdd/wiki/Record-Linkage

## Cerebro

 - https://github.com/lmenezes/cerebro/releases




