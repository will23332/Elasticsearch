# Elasticsearch Crash Course

![LOGO](https://cdn.freebiesupply.com/logos/large/2x/elasticsearch-logo-png-transparent.png)

+++

A cura di Guglielmo Piacentini e Alfredo Serafini

---

## Che cos'è Elasticsearch?

Un motore di ricerca e analisi full-text, open-source.

---

## Come lo uso?

@ul
- Hai un negozio online e vuoi permettere ai tuoi clienti di ricercare nel catalogo, utilizzando dei suggerimenti e il completamento automatico. Il catalogo e i tuoi prodotti verranno salvati in Elastic.
- Vuoi analizzare file di log e statistiche per estrarne conoscenza. Una volta portati i dati su Elastic puoi aggregare e ricercare per estrarre informazioni di interesse.
@ulend
+++
@ul
- Hai una lista di indirizzi scritti nei modi più disparati, utilizzi Elastic come un dizionario di controllo per costruire una lista di indirizzi normalizzati.
- Hai un prodotto commerciale che si basa su analisi full-text/semantiche (i.e. Pupilla). Elastic è la spina dorsale su cui vengono sia salvati i documenti che analizzati.
@ulend

---

## Com'è fatto?

Elasticsearch gira in cluster. 
Un cluster può essere formato da uno o più server.
Ogni server nel cluster è un nodo. 
ES stocca i documenti in indici. 
Un indice può essere "scomposto" in shard.

+++

ES si basa su [Apache Lucene](http://lucene.apache.org/) per l'indicizzazione dei documenti.
Ogni shard è un indice Lucene.

---

## Let's get our hands dirty.
[DEMO](https://demo.elastic.co/app/kibana)

---

## Installazione ES
[Link](https://www.elastic.co/downloads/elasticsearch)

+++
Creiamo un indice 
```
PUT /customer?pretty
```
Chiediamo a ES la lista degli indici
```
GET /_cat/indices?v
```
La risposta:
```
health status index    uuid                   pri rep docs.count docs.deleted store.size pri.store.size
yellow open   customer 95SQ4TSUT7mWBT7VNHH67A   5   1          0            0       260b           260b
```
+++
Inseriamo un documento semplice
```
PUT /customer/_doc/1?pretty
{
  "name": "John Doe"
}
```

+++

La risposta:
```
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created"
  [...]
}
```
+++

Ricerchiamo il documento appena inserito
```
GET /customer/_doc/1?pretty
```
La risposta:
```
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "found" : true,
  "_source" : { "name": "John Doe" }
}
```
+++
Eliminiamo l'indice creato
```
DELETE /customer?pretty
```
Richiamiamo la lista degli indici
```
GET /_cat/indices?v
```
La risposta sarà @color[#DC143C](vuota)

---

## Installazione Cerebro
[Link](https://github.com/lmenezes/cerebro)

Cerebro è una GUI che permette di interagire con Elasticsearch.

---

## REST API
Elastic espone delle API molto potenti che permettono di:

@ul
- Controllare lo stato di salute e le statistiche del cluster
- Amministrare il cluster, i nodi, gli indici, i dati e i metadati
- Performare CRUD (Create, Read, Update, and Delete) e operazioni di ricerca sugli indici
- Eseguire operazioni avanzate come paging, sorting, filtering, aggregazioni...
@ulend

---

## Hands On pt. II
Dataset [accounts.json](https://github.com/elastic/elasticsearch/blob/master/docs/src/test/resources/accounts.json?raw=true)

Bulk insert (anche tramite Cerebro):
```
curl -H "Content-Type: application/json" 
-XPOST "localhost:9200/bank/_doc/_bulk?pretty&refresh" 
--data-binary "@accounts.json"
```
+++
## La ricerca tramite URI
```
GET /bank/_search?q=*&sort=account_number:asc&pretty
```
+++
## Query DSL
La più semplice:
```
GET /bank/_search
{
  "query": { "match_all": {} }
}
```
+++
Sorting:
```
GET /bank/_search
{
  "query": { "match_all": {} },
  "sort": { "balance": { "order": "desc" } }
}
```
+++
Match Query:
```
GET /bank/_search
{
  "query": { "match": { "address": "mill lane" } }
}
```
+++
Filtri:
```
GET /bank/_search
{
  "query": {
    "bool": {
      "must": { "match_all": {} },
      "filter": {
        "range": {
          "balance": {
            "gte": 20000,
            "lte": 30000 }}}}}}
```

+++

Aggregazioni:
```
GET /bank/_search
{
  "size": 0,
  "aggs": {
    "group_by_state": {
      "terms": {
        "field": "state.keyword"
      }
    }
  }
}
```
+++

Documentazione [Search API](https://www.elastic.co/guide/en/elasticsearch/reference/current/search.html)

Documentazione [Query DSL](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl.html)

---

## Use Case

Regione Veneto:

```
 "settings": {
  "number_of_shards": 3,
  "number_of_replicas": 2,
  "analysis": {
   "analyzer": {
    "simple_rebuilt_comuni": {
     "tokenizer": "lowercase",
     "filter": [
      "lowercase",
      "asciifolding",
      "synonym_comuni"
     ]
    },
	"simple_rebuilt_province": {
     "tokenizer": "lowercase",
     "filter": [
      "synonym_province"
     ]
    }
   },
   "filter": {
    "synonym_comuni": {
     "type": "synonym",
     "synonyms_path": "analysis/synonym_comuni.txt"
    },
	"synonym_province": {
     "type": "synonym",
     "synonyms_path": "analysis/synonym_province.txt"
    }
   }
  }
 },
 "mappings": {
  "luogo": {
   "properties": {
    "nome_comune": {
     "type": "text",
     "analyzer": "simple_rebuilt_comuni"
    },
    "nome_provincia": {
     "type": "text",
     "analyzer": "simple_rebuilt_province"
    },
    "sigla_provincia": {
     "type": "text"
   },
    "nome_regione": {
     "type": "text"
    },
    "citta_metropolitana": {
     "type": "text"
    }
   }
  }
 }
}
```
@[2-4](Settaggi del Cluster)
@[5-11](Dichiarazione Analyzer Custom)
@[14-17](Settaggi Analyzer)
@[21-29](Definizione del filtro custom dei sinonimi)
@[24](Path dove risiede il file dei sinonimi)
@[35-38](Mapping campo comuni)
@[40-42](Mapping campo province)

+++
Documenti contenuti:
```
{"index":{"_index":"luoghi-istat", "_type":"luogo"}}
{"nome_comune" : "Agliè","nome_provincia" : "","sigla_provincia" : "TO","nome_regione" : "Piemonte","citta_metropolitana" : "Torino","codice_comune" : 1001,"codice_provincia" : 1,"codice_regione" : 1,"codice_citta_metropolitana" : 201}
{"index":{"_index":"luoghi-istat", "_type":"luogo"}}
{"nome_comune" : "Airasca","nome_provincia" : "","sigla_provincia" : "TO","nome_regione" : "Piemonte","citta_metropolitana" : "Torino","codice_comune" : 1002,"codice_provincia" : 1,"codice_regione" : 1,"codice_citta_metropolitana" : 201}
{"index":{"_index":"luoghi-istat", "_type":"luogo"}}
{"nome_comune" : "Ala di Stura","nome_provincia" : "","sigla_provincia" : "TO","nome_regione" : "Piemonte","citta_metropolitana" : "Torino","codice_comune" : 1003,"codice_provincia" : 1,"codice_regione" : 1,"codice_citta_metropolitana" : 201}
```

---
## Index Mapping
Elastic si fonda sul concetto di indice inverso:
- Un indice inverso consiste nella lista di tutte le parole non ripetute che appaiono in un documento e, per ognuna di queste, una lista dei documenti nelle quali appaiono.
- Questo permette una ricerca full-text particolarmente rapida.

+++

+ Analyzer

ES permette l'uso di diversi [analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-analyzers.html) out of the box, oltre che la possibilità di costruirne di custom.

+++
```
GET _analyze
{
  "analyzer" : "standard",
  "text" : "this is a test"
}
```
Altri casi di [test](https://www.elastic.co/guide/en/elasticsearch/reference/current/indices-analyze.html)

+++

## N.B. 
C'è una differenza sostianziale nella analisi a livello indice e l'analisi durante la ricerca.

Term Query VS Full-Text Query

More [here](https://madewithlove.be/basic-understanding-of-text-search/)

+++

+ Tokenizer

I [tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-tokenizers.html) ricevono uno stream di caratteri e li dividono in token individuali a seconda del tokenizer (di solito singole parole)

+++

+ Token Filter (Synonym)

I [token filter](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-tokenfilter.html) accettano e modificano gli stream di token provenienti dai tokenizer. Nello specifico il filtro [synonym](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-synonym-tokenfilter.html) permette la trasformazione dei token secondo delle tabelle di sinonimi determinate dall'utente.

---

## Use Case #2
CRAIM:
Il KM si appoggia ad ES per le funzionalità di ricerca full-text.

Il mapping di un unico indice dei documenti sul KM di CRAIM vede @color[#DC143C](più di 5300 righe di codice).

+++
```
"push": {
				"_all": {
					"enabled": false
				},
				"_source": {
					"excludes": ["formattedtext"]
				},
				"dynamic_templates": [{
					"suggestion": {
						"mapping": {
							"analyzer": "suggestion_analyzer",
							"type": "completion"
						},
						"match": "*_suggest_*"
					}
				}, {
					"entity_field": {
						"mapping": {
							"index": "analyzed",
							"analyzer": "chain_analyzer",
							"boost": 2.0,
							"type": "string"
						},
						"match": "named_entity_*"
					}
				}, {
					"crmtuple_analyzed": {
						"mapping": {
							"index": "analyzed",
							"analyzer": "crmtuple_analyzer",
							"type": "string"
						},
						"match": "TUPLE*"
					}
				}, {
					"string_not_analyzed": {
						"mapping": {
							"index": "not_analyzed",
							"type": "string"
						},
						"match": "*",
						"match_mapping_type": "string"
					}
				}, {
					"date_not_analyzed": {
						"mapping": {
							"index": "not_analyzed",
							"type": "string"
						},
						"match": "*",
						"match_mapping_type": "date"
					}
				}],
				"properties": {
					"ACCESS_PERMISSION:ASSEMBLE_DOCUMENT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"ACCESS_PERMISSION:CAN_MODIFY": {
						"type": "string",
						"index": "not_analyzed"
					},
					"ANOTHER_FILE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"APPLICATION-NAME": {
						"type": "string",
						"index": "not_analyzed"
					},
					"APPLICATION-VERSION": {
						"type": "string",
						"index": "not_analyzed"
					},
					"AUTHOR": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CHANNEL_ID": {
						"type": "long"
					},
					"CHANNEL_NAME": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CHANNEL_TYPE": {
						"type": "string",
						"index": "no"
					},
					"CHARACTER COUNT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CHARACTER-COUNT-WITH-SPACES": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CLASSIFICATION_LOGIC_NAME_": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CONFIDENTIAL_GROUP_NAME": {
						"type": "string",
						"analyzer": "whitespace"
					},
					"CONFIDENTIAL_LEVEL_LABEL": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CONFIDENTIAL_LEVEL_NUM": {
						"type": "integer"
					},
					"CONTACT_ID": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CONTENT-ENCODING": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CONTENT-TYPE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CP:REVISION": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CREATED": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CREATION-DATE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"CREATOR": {
						"type": "string",
						"index": "not_analyzed"
					},
					"DATE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"DC:CREATOR": {
						"type": "string",
						"index": "not_analyzed"
					},
					"DC:FORMAT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"DC:TITLE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"DCTERMS:CREATED": {
						"type": "string",
						"index": "not_analyzed"
					},
					"DCTERMS:MODIFIED": {
						"type": "string",
						"index": "not_analyzed"
					},
					"DialogAct": {
						"type": "string",
						"analyzer": "whitespace"
					},
					"EXTENDED-PROPERTIES:APPLICATION": {
						"type": "string",
						"index": "not_analyzed"
					},
					"EXTENDED-PROPERTIES:APPVERSION": {
						"type": "string",
						"index": "not_analyzed"
					},
					"EXTENDED-PROPERTIES:TEMPLATE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"EXTENDED-PROPERTIES:TOTALTIME": {
						"type": "string",
						"index": "not_analyzed"
					},
					"EXTERNAL_CREATE_DATE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"EXTERNAL_DOCUMENT_ID": {
						"type": "string",
						"index": "not_analyzed"
					},
					"EXTERNAL_UPDATE_DATE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"LAST-AUTHOR": {
						"type": "string",
						"index": "not_analyzed"
					},
					"LAST-MODIFIED": {
						"type": "string",
						"index": "not_analyzed"
					},
					"LAST-SAVE-DATE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"LINE-COUNT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"META:AUTHOR": {
						"type": "string",
						"index": "not_analyzed"
					},
					"META:CHARACTER-COUNT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"META:CHARACTER-COUNT-WITH-SPACES": {
						"type": "string",
						"index": "not_analyzed"
					},
					"META:CREATION-DATE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"META:LAST-AUTHOR": {
						"type": "string",
						"index": "not_analyzed"
					},
					"META:LINE-COUNT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"META:PAGE-COUNT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"META:PARAGRAPH-COUNT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"META:SAVE-DATE": {
						"type": "string",
						"index": "not_analyzed"
					},
					"META:WORD-COUNT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"MODIFIED": {
						"type": "string",
						"index": "not_analyzed"
					},
					"ONTOLOGY_FILENAME": {
						"type": "string",
						"analyzer": "whitespace"
					},
					"PAGE-COUNT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"PARAGRAPH-COUNT": {
						"type": "string",
						"index": "not_analyzed"
					},
					"PDF:DOCINFO:CREATED": {
						"type": "string",
						"index": "not_analyzed"
					},
					"PDF:DOCINFO:MODIFIED": {
						"type": "string",
						"index": "not_analyzed"
					},
					"PDF:DOCINFO:PRODUCER": {
						"type": "string",
						"index": "not_analyzed"
					},
					"PDF:ENCRYPTED": {
						"type": "string",
						"index": "not_analyzed"
					},
					"PDF:PDFEXTENSIONVERSION": {
						"type": "string",
						"index": "not_analyzed"
					},
					"PDF:PDFVERSION": {
						"type": "string",
						"index": "not_analyzed"
					},
					"PRODUCER": {
						"type": "string",
						"index": "not_analyzed"
					},
					"REVISION-NUMBER": {
						"type": "string",
						"index": "not_analyzed"
					},
					"SESSION_DURATION": {
						"type": "string",
						"index": "not_analyzed"
					},
					"answer": {
						"type": "semantic",
						"store": true,
						"term_vector": "with_positions_offsets",
						"fields": {
							"sem": {
								"type": "semantic",
								"term_vector": "with_positions_offsets",
								"ignore_malformed": true
							}
						},
						"ignore_malformed": true,
						"ignore_tokens": true
					},
					"boosting": {
						"type": "long"
					},
					"checksum": {
						"type": "string",
						"index": "not_analyzed"
					},
					"conceptualmap": {
						"type": "string",
						"index": "no"
					},
					"deleteDate": {
						"type": "date",
						"format": "strict_date_optional_time||epoch_millis"
					},
					"document_raw_storage": {
						"type": "string",
						"index": "not_analyzed"
					},
					"enable_speaker_separation": {
						"type": "string",
						"index": "not_analyzed"
					},
					"entity_suggest_Address": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_BadWord": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Colori": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Comuni": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Date": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Email": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Lingue": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Location": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Money": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Number": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Organization": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Percent": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Percentage": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Person": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Province": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entity_suggest_Regioni": {
						"type": "completion",
						"analyzer": "suggestion_analyzer",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50
					},
					"entitymap": {
						"type": "string",
						"index": "no"
					},
					"formattedtext": {
						"type": "binary",
						"store": true
					},
					"insertDate": {
						"type": "date",
						"format": "strict_date_optional_time||epoch_millis"
					},
					"language": {
						"type": "string",
						"index": "not_analyzed",
						"store": true
					},
					"mimetype": {
						"type": "string",
						"index": "not_analyzed"
					},
					"named_entity_Address": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_BadWord": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Colori": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Comuni": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Date": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Email": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Lingue": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Location": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Money": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Number": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Organization": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Percent": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Percentage": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Person": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Province": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"named_entity_Regioni": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"parentURI": {
						"type": "string",
						"analyzer": "chain_analyzer"
					},
					"plaintext": {
						"type": "semantic",
						"store": true,
						"term_vector": "with_positions_offsets",
						"fields": {
							"sem": {
								"type": "semantic",
								"term_vector": "with_positions_offsets",
								"ignore_malformed": true
							}
						},
						"ignore_malformed": true,
						"ignore_tokens": true
					},
					"propertyUriExactTriple": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"propertyUrisPermutation": {
						"type": "string",
						"analyzer": "chain_analyzer"
					},
					"query_suggestion": {
						"type": "completion",
						"analyzer": "simple",
						"payloads": false,
						"preserve_separators": true,
						"preserve_position_increments": true,
						"max_input_length": 50,
						"context": {
							"ontology": {
								"type": "category",
								"path": "_type",
								"default": []
							}
						}
					},
					"resourceURI": {
						"type": "string",
						"boost": 2.0,
						"analyzer": "chain_analyzer"
					},
					"resourceURI_answer": {
						"type": "string",
						"analyzer": "chain_analyzer"
					},
					"size": {
						"type": "long"
					},
					"sponsoring": {
						"type": "long"
					},
					"tuple_answer": {
						"type": "string",
						"analyzer": "crmtuple_analyzer"
					},
					"tuple_certified": {
						"type": "string",
						"analyzer": "crmtuple_analyzer"
					},
					"updateDate": {
						"type": "date",
						"format": "strict_date_optional_time||epoch_millis"
					},
					"uri": {
						"type": "string",
						"index": "not_analyzed"
					}
				}
			},
			".percolator": {
				"properties": {
					"query": {
						"enabled": false,
						"properties": {
							"query_string": {
								"properties": {
									"query": {
										"type": "string"
									}
								}
							}
						}
					},
					"textToPercolate": {
						"type": "string"
					}
				}
			}
```
@[8-15](Analyzer di suggerimento)
@[17-25](Analyzer con boosting delle entità)
@[446-453](Campo mappato per il suggerimento delle entità)
+++

```
{
	"document_raw_storage": "yes",
	"CHANNEL_ID": "69737",
	"entitymap": "rO0ABXNyABFqYXZhLnV0aWwuSGFzaE1hcAUH2sHDFmHhwc3EAfgAMdwwAAAAAP0AAAAAAAAB4eHEAfgDPcHQALmh0dHA6Ly9kYnBlZGlhLm9yZy9vbnRvbG9neS9QZXJzb24jVGhlcmVzYSBNYXlzcQB+AAUAAAAAAAAAAHB0AAlHZW50aWxvbml0AB9QZXJzb25fUjJWdWRHbHNiMjVwXy0xNTgyNTk0NjUzc3EAfgAMdwwAAAAAP0AAAAAAAAB4c3EAfgAOcHcEAAAAAXNxAH4AEAAAC7kAABHDAAAAAAAAC7Bwc3EAfgAMdwwAAAAAP0AAAAAAAAB4cHNxAH4ADHcMAAAAAD9AAAAAAAAAeHhxAH4Az3B0ACxodHRwOi8vZGJwZWRpYS5vcmcvb250b2xvZ3kvUGVyc29uI0dlbnRpbG9uaXh4",
	"named_entity_Comuni": "Force[:]Mattinata[:]Romana[:]Paese",
	"insertDate": "2017-07-11T08:07:19.296Z",
	"checksum": "4f2c990e29132757db0d59a7eb0aee29",
	"named_entity_Location": "Italia[:]Paese[:]Londra[:]Bretagna[:]Londra[:]Campidoglio[:]Italia[:]Londra[:]Tamigi[:]Westminster[:]Londra[:]Tokyo[:]Italia[:]Westminster Bridge[:]Italia[:]ponte di Westminster[:]Londra",
	"CONTACT_ID": "FILESYSTEM_VOICE_71b1db82-7271-4780-b074-c3aea3872079",
	"CHANNEL_NAME": "CanalePVT",
	"mimetype": "text/plain",
	"ONTOLOGY_FILENAME": "CraimInt_6",
	"CHANNEL_TYPE": "PUSH",
	"parentURI": "http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Oggetti[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Luoghi_Geografici[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Luoghi_Geografici[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Airport[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Luoghi_Geografici[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Polizia_di_Stato[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Luoghi_Geografici[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Fenome_Sociale[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Luoghi_Geografici[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Luoghi_Geografici[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Luoghi_Geografici[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Azioni[;]Thing[:]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Luoghi_Geografici[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Luoghi_Geografici[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Airport[;]Thing[:]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Luoghi_Geografici[;]Thing[:]",
	"named_entity_Province": "Roma[:]Roma[:]Roma",
	"entity_suggest_Organization": ["Partito Democratico", "Parlamento"],
	"CONFIDENTIAL_LEVEL_NUM": "00",
	"CONTENT-TYPE": "text/plain; charset=UTF-8",
	"SESSION_DURATION": "null",
	"SOURCE_LANGUAGE": "it",
	"TUPLE": "",
	"entity_suggest_Person": ["Theresa May", "Gentiloni", "Paolo Trombi"],
	"SOURCE_CHANNEL": "ChannelCRM-PVT-ITA",
	"plaintext": "{\"v\":\"1\",\"str\":\"7 30 buongiorno di nuovo benvenuti a prima pagina le immagini che vi stiamo per mostrare mostrano Londra il giorno dopo Londra ferita prima auto sulla folla poi il blitz con coltello per verso il Parlamento una Londra colpita al cuore in uno dei simboli della democrazia sono 4 le persone uccise dall' attentatore è entrato in azione ieri pomeriggio lo vedete lo scenario è ancora spettrale agenti presidiano è stato il teatro di questo agguato e che noi ripercorriamo nel prossimo servizio 3 feriti nell' attentato a Londra ci sono anche due italiani si tratta di una giovane bolognese che vive da 6 anni nel Paese ricoverati in ospedale o curate subito dimessa ed una ragazza romana in gita assieme al fidanzato quest' ultima è stata operata in serata le sue condizioni fortunatamente non sono gravi sono le ultimissime notizie sull' attacco davanti al Parlamento inglese avvenuto ieri pomeriggio poco dopo le 14 un Suv ha imboccato il ponte di Westminster è piombato sulla folla seminando morti panico l' attentatore quarantenne dai tratti asiatici ancora non identificato armato di coltello è sceso dall' auto ha tentato di entrare nell' edificio del Parlamento è riuscito ad accoltellare un agente è stato subito ucciso dagli altri poliziotti pesante il bilancio delle vittime 5 morti l' attentatore l' agente 3 civili e 40 feriti 3 gravi e lievi un attacco terroristico odioso disgustoso ha definito la premier britannica Theresa May che si trova a Westminster al momento intanto la premier ha convocato il comitato di emergenza per la sicurezza nazionale si osserva che le immagini dei momenti in cui il Suv a folle velocità imbocca il Westminster Bridge lo vedete nella quadrato invece nel cerchio rosso il momento in cui per sfuggire alla folle corsa dell' automobile una donna si butta nel Tamigi sappiamo verrà recuperata ferita ma non in gravi condizioni si alza il livello di allerta in tutte le capitali europee in particolare devo vedere come si prepara la macchina della sicurezza a Roma in vista del vertice europeo di sabato dopo l' attacco terroristico di Londra sale la tensione anche nella capitale italiana dove nel fine settimana sono previste le celebrazioni per i 60 anni dei Trattati di Roma le misure di sicurezza non sono mai state così elevate è tutto pronto per la task force delle forze dell' ordine che lavorerà ininterrottamente per 48 ore a partire da venerdì alle 8 fino a sabato notte per garantire la sicurezza in città ottomila gli uomini schierati che garantiranno i controlli non solo nella zona tra piazza di Spagna il Campidoglio dove si svolgerà la cerimonia ufficiale ma anche lungo i percorsi delle 4 manifestazioni previste sabato i controlli inizieranno già fuori Roma ai caselli autostradali dove transiteranno una cinquantina di pullman dei centri sociali provenienti da tutta Italia e lungo le vie consolari in centro invece alle telecamere fisse ne sono state aggiunte un centinaio alcuni mobili lungo i percorsi dei manifestanti il premier Gentiloni ha espresso tutta la vicinanza dell' Italia alla gran Bretagna per l' attacco di ieri a Londra siamo a fianco a fianco nella condanna afferma nella risposta contro ogni forma di terrorismo ha aggiunto il premier nel corso dell' assemblea del Partito Democratico ha poi toccato anche altri temi liquidando le parole da esse non come volgari e inaccettabili ribadendo che l' Italia proseguirà nel suo impegno europeista senza farsi condizionare quanto alle riforme Gentiloni ha sottolineato l' Italia non ha nessuna intenzione di ammainare la bandiera anzi dice abbiamo la maggioranza in Parlamento e davanti a noi c'è un periodo di tempo congruo per fare alcune cose non basta invece la bozza di decreto messa appunto dall' esecutivo di Gentiloni per evitare il nuovo sciopero dei taxi oggi dalle 8 alle 22 si prevedono disagi per lo sciopero indetto dai tassisti ieri alla fine la bozza di decreto proposto dal Governo ha scontentato tutti titolari di taxi e unione nazionale consumatori anche l' ultimo tentativo non è servito a sbloccare la vertenza e nelle prossime ore sarà di nuovo sciopero la proposta presentata dal viceministro dei trasporti vicinissimi con una regolamentazione suo Ncc auto bianche è stata apprezzata dell' approccio ma non è bastato anche se qualche sigla come i taxi gli aderenti a Legacoop ha ribadito che non scioperano utero tesa al fronte economico dai mercati con Paolo Trombi si partiamo subito del nulla della mattinata Tokyo ha chiuso in rialzo dello 0,23% sono torno alla stabilità al all' invarianza mercati\\n\",\"tokens\":[{\"t\":\"7\",\"s\":0,\"e\":1},{\"t\":\"30\",\"s\":2,\"e\":4},{\"t\":\"buongiorno\",\"s\":5,\"e\":15},{\"t\":\"nuovo\",\"s\":19,\"e\":24},{\"t\":\"benvenuto\",\"s\":25,\"e\":34},{\"t\":\"primo\",\"s\":37,\"e\":42},{\"t\":\"pagina\",\"s\":43,\"e\":49},{\"t\":\"immagine\",\"s\":53,\"e\":61},{\"t\":\"mostrare\",\"s\":80,\"e\":88},{\"t\":\"mostrare\",\"s\":89,\"e\":97},{\"t\":\"Londra\",\"s\":98,\"e\":104},{\"t\":\"giorno\",\"s\":108,\"e\":114},{\"t\":\"Londra\",\"s\":120,\"e\":126},{\"t\":\"ferire\",\"s\":127,\"e\":133},{\"t\":\"prima\",\"s\":134,\"e\":139},{\"t\":\"auto\",\"s\":140,\"e\":144},{\"t\":\"folla\",\"s\":151,\"e\":156},{\"t\":\"poi\",\"s\":157,\"e\":160},{\"t\":\"blitz\",\"s\":164,\"e\":169},{\"t\":\"coltello\",\"s\":174,\"e\":182},{\"t\":\"parlamento\",\"s\":196,\"e\":206},{\"t\":\"Londra\",\"s\":211,\"e\":217},{\"t\":\"colpire\",\"s\":218,\"e\":225},{\"t\":\"cuore\",\"s\":229,\"e\":234},{\"t\":\"simbolo\",\"s\":246,\"e\":253},{\"t\":\"democrazia\",\"s\":260,\"e\":270},{\"t\":\"4\",\"s\":276,\"e\":277},{\"t\":\"persona\",\"s\":281,\"e\":288},{\"t\":\"ucciso\",\"s\":289,\"e\":295},{\"t\":\"attentatore\",\"s\":302,\"e\":313},{\"t\":\"entrare\",\"s\":316,\"e\":323},{\"t\":\"azione\",\"s\":327,\"e\":333},{\"t\":\"ieri\",\"s\":334,\"e\":338},{\"t\":\"pomeriggio\",\"s\":339,\"e\":349},{\"t\":\"vedere\",\"s\":353,\"e\":359},{\"t\":\"scenario\",\"s\":363,\"e\":371},{\"t\":\"ancora\",\"s\":374,\"e\":380},{\"t\":\"spettrale\",\"s\":381,\"e\":390},{\"t\":\"agente\",\"s\":391,\"e\":397},{\"t\":\"presidiare\",\"s\":398,\"e\":408},{\"t\":\"essere|stare\",\"s\":411,\"e\":416},{\"t\":\"teatro\",\"s\":420,\"e\":426},{\"t\":\"agguato\",\"s\":437,\"e\":444},{\"t\":\"ripercorrere\",\"s\":455,\"e\":468},{\"t\":\"prossimo\",\"s\":473,\"e\":481},{\"t\":\"servizio\",\"s\":482,\"e\":490},{\"t\":\"3\",\"s\":491,\"e\":492},{\"t\":\"ferito\",\"s\":493,\"e\":499},{\"t\":\"attentato\",\"s\":506,\"e\":515},{\"t\":\"Londra\",\"s\":518,\"e\":524},{\"t\":\"italiano\",\"s\":543,\"e\":551},{\"t\":\"trattare\",\"s\":555,\"e\":561},{\"t\":\"giovane\",\"s\":569,\"e\":576},{\"t\":\"bolognese\",\"s\":577,\"e\":586},{\"t\":\"vivere\",\"s\":591,\"e\":595},{\"t\":\"6\",\"s\":599,\"e\":600},{\"t\":\"anno\",\"s\":601,\"e\":605},{\"t\":\"paese\",\"s\":610,\"e\":615},{\"t\":\"ricoverare\",\"s\":616,\"e\":626},{\"t\":\"ospedale\",\"s\":630,\"e\":638},{\"t\":\"curare\",\"s\":641,\"e\":647},{\"t\":\"subito\",\"s\":648,\"e\":654},{\"t\":\"dimesso\",\"s\":655,\"e\":662},{\"t\":\"ragazza\",\"s\":670,\"e\":677},{\"t\":\"romano\",\"s\":678,\"e\":684},{\"t\":\"gita\",\"s\":688,\"e\":692},{\"t\":\"assieme\",\"s\":693,\"e\":700},{\"t\":\"fidanzato\",\"s\":704,\"e\":713},{\"t\":\"ultimo\",\"s\":721,\"e\":727},{\"t\":\"essere\",\"s\":730,\"e\":735},{\"t\":\"operare\",\"s\":736,\"e\":743},{\"t\":\"serata\",\"s\":747,\"e\":753},{\"t\":\"condizione\",\"s\":761,\"e\":771},{\"t\":\"fortunatamente\",\"s\":772,\"e\":786},{\"t\":\"grave\",\"s\":796,\"e\":801},{\"t\":\"ultimo\",\"s\":810,\"e\":821},{\"t\":\"notizia\",\"s\":822,\"e\":829},{\"t\":\"attacco\",\"s\":836,\"e\":843},{\"t\":\"parlamento\",\"s\":855,\"e\":865},{\"t\":\"inglese\",\"s\":866,\"e\":873},{\"t\":\"avvenire\",\"s\":874,\"e\":882},{\"t\":\"ieri\",\"s\":883,\"e\":887},{\"t\":\"pomeriggio\",\"s\":888,\"e\":898},{\"t\":\"poco\",\"s\":899,\"e\":903},{\"t\":\"14\",\"s\":912,\"e\":914},{\"t\":\"Suv\",\"s\":918,\"e\":921},{\"t\":\"imboccare\",\"s\":925,\"e\":934},{\"t\":\"ponte\",\"s\":938,\"e\":943},{\"t\":\"Westminster\",\"s\":947,\"e\":958},{\"t\":\"piombare\",\"s\":961,\"e\":969},{\"t\":\"folla\",\"s\":976,\"e\":981},{\"t\":\"seminare\",\"s\":982,\"e\":991},{\"t\":\"morte|morto\",\"s\":992,\"e\":997},{\"t\":\"panico\",\"s\":998,\"e\":1004},{\"t\":\"attentatore\",\"s\":1008,\"e\":1019},{\"t\":\"quarantenne\",\"s\":1020,\"e\":1031},{\"t\":\"tratto\",\"s\":1036,\"e\":1042},{\"t\":\"asiatico\",\"s\":1043,\"e\":1051},{\"t\":\"ancora\",\"s\":1052,\"e\":1058},{\"t\":\"identificare\",\"s\":1063,\"e\":1075},{\"t\":\"armato\",\"s\":1076,\"e\":1082},{\"t\":\"coltello\",\"s\":1086,\"e\":1094},{\"t\":\"scendere\",\"s\":1097,\"e\":1102},{\"t\":\"auto\",\"s\":1109,\"e\":1113},{\"t\":\"tentare\",\"s\":1117,\"e\":1124},{\"t\":\"entrare\",\"s\":1128,\"e\":1135},{\"t\":\"edificio\",\"s\":1142,\"e\":1150},{\"t\":\"Parlamento\",\"s\":1155,\"e\":1165},{\"t\":\"riuscire\",\"s\":1168,\"e\":1176},{\"t\":\"accoltellare\",\"s\":1180,\"e\":1192},{\"t\":\"agente\",\"s\":1196,\"e\":1202},{\"t\":\"essere\",\"s\":1205,\"e\":1210},{\"t\":\"subito\",\"s\":1211,\"e\":1217},{\"t\":\"uccidere\",\"s\":1218,\"e\":1224},{\"t\":\"poliziotto\",\"s\":1237,\"e\":1247},{\"t\":\"pesante\",\"s\":1248,\"e\":1255},{\"t\":\"bilancio\",\"s\":1259,\"e\":1267},{\"t\":\"vittima\",\"s\":1274,\"e\":1281},{\"t\":\"5\",\"s\":1282,\"e\":1283},{\"t\":\"morte|morto\",\"s\":1284,\"e\":1289},{\"t\":\"attentatore\",\"s\":1293,\"e\":1304},{\"t\":\"agente\",\"s\":1308,\"e\":1314},{\"t\":\"3\",\"s\":1315,\"e\":1316},{\"t\":\"civile\",\"s\":1317,\"e\":1323},{\"t\":\"40\",\"s\":1326,\"e\":1328},{\"t\":\"ferito\",\"s\":1329,\"e\":1335},{\"t\":\"3\",\"s\":1336,\"e\":1337},{\"t\":\"grave\",\"s\":1338,\"e\":1343},{\"t\":\"lieve\",\"s\":1346,\"e\":1351},{\"t\":\"attacco\",\"s\":1355,\"e\":1362},{\"t\":\"terroristico\",\"s\":1363,\"e\":1375},{\"t\":\"odioso\",\"s\":1376,\"e\":1382},{\"t\":\"disgustoso\",\"s\":1383,\"e\":1393},{\"t\":\"definire\",\"s\":1397,\"e\":1405},{\"t\":\"premier\",\"s\":1409,\"e\":1416},{\"t\":\"britannico\",\"s\":1417,\"e\":1427},{\"t\":\"Theresa\",\"s\":1428,\"e\":1435},{\"t\":\"May\",\"s\":1436,\"e\":1439},{\"t\":\"trovare\",\"s\":1447,\"e\":1452},{\"t\":\"Westminster\",\"s\":1455,\"e\":1466},{\"t\":\"momento\",\"s\":1470,\"e\":1477},{\"t\":\"intanto\",\"s\":1478,\"e\":1485},{\"t\":\"premier\",\"s\":1489,\"e\":1496},{\"t\":\"convocare\",\"s\":1500,\"e\":1509},{\"t\":\"comitato\",\"s\":1513,\"e\":1521},{\"t\":\"emergenza\",\"s\":1525,\"e\":1534},{\"t\":\"sicurezza\",\"s\":1542,\"e\":1551},{\"t\":\"nazionale\",\"s\":1552,\"e\":1561},{\"t\":\"osservare\",\"s\":1565,\"e\":1572},{\"t\":\"immagine\",\"s\":1580,\"e\":1588},{\"t\":\"momento\",\"s\":1593,\"e\":1600},{\"t\":\"Suv\",\"s\":1611,\"e\":1614},{\"t\":\"folle\",\"s\":1617,\"e\":1622},{\"t\":\"velocità\",\"s\":1623,\"e\":1631},{\"t\":\"imboccare\",\"s\":1632,\"e\":1639},{\"t\":\"Westminster\",\"s\":1643,\"e\":1654},{\"t\":\"Bridge\",\"s\":1655,\"e\":1661},{\"t\":\"vedere\",\"s\":1665,\"e\":1671},{\"t\":\"quadrato\",\"s\":1678,\"e\":1686},{\"t\":\"invece\",\"s\":1687,\"e\":1693},{\"t\":\"cerchio\",\"s\":1698,\"e\":1705},{\"t\":\"rosso\",\"s\":1706,\"e\":1711},{\"t\":\"momento\",\"s\":1715,\"e\":1722},{\"t\":\"sfuggire\",\"s\":1734,\"e\":1742},{\"t\":\"folle\",\"s\":1748,\"e\":1753},{\"t\":\"corsa\",\"s\":1754,\"e\":1759},{\"t\":\"automobile\",\"s\":1766,\"e\":1776},{\"t\":\"donna\",\"s\":1781,\"e\":1786},{\"t\":\"buttare\",\"s\":1790,\"e\":1795},{\"t\":\"Tamigi\",\"s\":1800,\"e\":1806},{\"t\":\"sapere\",\"s\":1807,\"e\":1815},{\"t\":\"venire\",\"s\":1816,\"e\":1821},{\"t\":\"recuperare\",\"s\":1822,\"e\":1832},{\"t\":\"ferito\",\"s\":1833,\"e\":1839},{\"t\":\"grave\",\"s\":1850,\"e\":1855},{\"t\":\"condizione\",\"s\":1856,\"e\":1866},{\"t\":\"alzare\",\"s\":1870,\"e\":1874},{\"t\":\"livello\",\"s\":1878,\"e\":1885},{\"t\":\"allerta\",\"s\":1889,\"e\":1896},{\"t\":\"capitale\",\"s\":1909,\"e\":1917},{\"t\":\"europeo\",\"s\":1918,\"e\":1925},{\"t\":\"particolare\",\"s\":1929,\"e\":1940},{\"t\":\"dovere\",\"s\":1941,\"e\":1945},{\"t\":\"vedere\",\"s\":1946,\"e\":1952},{\"t\":\"preparare\",\"s\":1961,\"e\":1968},{\"t\":\"macchina\",\"s\":1972,\"e\":1980},{\"t\":\"sicurezza\",\"s\":1987,\"e\":1996},{\"t\":\"Roma\",\"s\":1999,\"e\":2003},{\"t\":\"vista\",\"s\":2007,\"e\":2012},{\"t\":\"vertice\",\"s\":2017,\"e\":2024},{\"t\":\"europeo\",\"s\":2025,\"e\":2032},{\"t\":\"sabato\",\"s\":2036,\"e\":2042},{\"t\":\"attacco\",\"s\":2051,\"e\":2058},{\"t\":\"terroristico\",\"s\":2059,\"e\":2071},{\"t\":\"Londra\",\"s\":2075,\"e\":2081},{\"t\":\"salire\",\"s\":2082,\"e\":2086},{\"t\":\"tensione\",\"s\":2090,\"e\":2098},{\"t\":\"capitale\",\"s\":2111,\"e\":2119},{\"t\":\"italiano\",\"s\":2120,\"e\":2128},{\"t\":\"fine\",\"s\":2138,\"e\":2142},{\"t\":\"settimana\",\"s\":2143,\"e\":2152},{\"t\":\"previsto\",\"s\":2158,\"e\":2166},{\"t\":\"celebrazione\",\"s\":2170,\"e\":2182},{\"t\":\"60\",\"s\":2189,\"e\":2191},{\"t\":\"anno\",\"s\":2192,\"e\":2196},{\"t\":\"Trattati\",\"s\":2201,\"e\":2209},{\"t\":\"Roma\",\"s\":2213,\"e\":2217},{\"t\":\"misura\",\"s\":2221,\"e\":2227},{\"t\":\"sicurezza\",\"s\":2231,\"e\":2240},{\"t\":\"mai\",\"s\":2250,\"e\":2253},{\"t\":\"essere\",\"s\":2254,\"e\":2259},{\"t\":\"così\",\"s\":2260,\"e\":2264},{\"t\":\"elevato\",\"s\":2265,\"e\":2272},{\"t\":\"pronto\",\"s\":2281,\"e\":2287},{\"t\":\"task\",\"s\":2295,\"e\":2299},{\"t\":\"force\",\"s\":2300,\"e\":2305},{\"t\":\"forza\",\"s\":2312,\"e\":2317},{\"t\":\"ordine\",\"s\":2324,\"e\":2330},{\"t\":\"lavorare\",\"s\":2335,\"e\":2343},{\"t\":\"ininterrottamente\",\"s\":2344,\"e\":2361},{\"t\":\"48\",\"s\":2366,\"e\":2368},{\"t\":\"ora\",\"s\":2369,\"e\":2372},{\"t\":\"partire\",\"s\":2375,\"e\":2382},{\"t\":\"venerdì\",\"s\":2386,\"e\":2393},{\"t\":\"8\",\"s\":2399,\"e\":2400},{\"t\":\"sabato\",\"s\":2408,\"e\":2414},{\"t\":\"notte\",\"s\":2415,\"e\":2420},{\"t\":\"garantire\",\"s\":2425,\"e\":2434},{\"t\":\"sicurezza\",\"s\":2438,\"e\":2447},{\"t\":\"città\",\"s\":2451,\"e\":2456},{\"t\":\"uomo\",\"s\":2470,\"e\":2476},{\"t\":\"schierare\",\"s\":2477,\"e\":2486},{\"t\":\"garantire\",\"s\":2491,\"e\":2503},{\"t\":\"controllo\",\"s\":2506,\"e\":2515},{\"t\":\"solo\",\"s\":2520,\"e\":2524},{\"t\":\"zona\",\"s\":2531,\"e\":2535},{\"t\":\"piazza\",\"s\":2540,\"e\":2546},{\"t\":\"Spagna\",\"s\":2550,\"e\":2556},{\"t\":\"Campidoglio\",\"s\":2560,\"e\":2571},{\"t\":\"svolgere\",\"s\":2580,\"e\":2588},{\"t\":\"cerimonia\",\"s\":2592,\"e\":2601},{\"t\":\"ufficiale\",\"s\":2602,\"e\":2611},{\"t\":\"percorso\",\"s\":2629,\"e\":2637},{\"t\":\"4\",\"s\":2644,\"e\":2645},{\"t\":\"manifestazione\",\"s\":2646,\"e\":2660},{\"t\":\"previsto\",\"s\":2661,\"e\":2669},{\"t\":\"sabato\",\"s\":2670,\"e\":2676},{\"t\":\"controllo\",\"s\":2679,\"e\":2688},{\"t\":\"iniziare\",\"s\":2689,\"e\":2700},{\"t\":\"già\",\"s\":2701,\"e\":2704},{\"t\":\"Roma\",\"s\":2711,\"e\":2715},{\"t\":\"casello\",\"s\":2719,\"e\":2726},{\"t\":\"autostradale\",\"s\":2727,\"e\":2739},{\"t\":\"transitare\",\"s\":2745,\"e\":2758},{\"t\":\"cinquantina\",\"s\":2763,\"e\":2774},{\"t\":\"pullman\",\"s\":2778,\"e\":2785},{\"t\":\"centro\",\"s\":2790,\"e\":2796},{\"t\":\"sociale\",\"s\":2797,\"e\":2804},{\"t\":\"provenire\",\"s\":2805,\"e\":2816},{\"t\":\"Italia\",\"s\":2826,\"e\":2832},{\"t\":\"via\",\"s\":2844,\"e\":2847},{\"t\":\"consolare\",\"s\":2848,\"e\":2857},{\"t\":\"centro\",\"s\":2861,\"e\":2867},{\"t\":\"invece\",\"s\":2868,\"e\":2874},{\"t\":\"telecamera\",\"s\":2880,\"e\":2890},{\"t\":\"fisso\",\"s\":2891,\"e\":2896},{\"t\":\"essere\",\"s\":2905,\"e\":2910},{\"t\":\"aggiungere\",\"s\":2911,\"e\":2919},{\"t\":\"centinaio\",\"s\":2923,\"e\":2932},{\"t\":\"mobile\",\"s\":2940,\"e\":2946},{\"t\":\"percorso\",\"s\":2955,\"e\":2963},{\"t\":\"manifestante\",\"s\":2968,\"e\":2980},{\"t\":\"premier\",\"s\":2984,\"e\":2991},{\"t\":\"Gentiloni\",\"s\":2992,\"e\":3001},{\"t\":\"esprimere\",\"s\":3005,\"e\":3013},{\"t\":\"vicinanza\",\"s\":3023,\"e\":3032},{\"t\":\"Italia\",\"s\":3039,\"e\":3045},{\"t\":\"gran\",\"s\":3051,\"e\":3055},{\"t\":\"Bretagna\",\"s\":3056,\"e\":3064},{\"t\":\"attacco\",\"s\":3072,\"e\":3079},{\"t\":\"ieri\",\"s\":3083,\"e\":3087},{\"t\":\"Londra\",\"s\":3090,\"e\":3096},{\"t\":\"fianco\",\"s\":3105,\"e\":3111},{\"t\":\"fianco\",\"s\":3114,\"e\":3120},{\"t\":\"condanna\",\"s\":3127,\"e\":3135},{\"t\":\"affermare\",\"s\":3136,\"e\":3143},{\"t\":\"risposta\",\"s\":3150,\"e\":3158},{\"t\":\"forma\",\"s\":3171,\"e\":3176},{\"t\":\"terrorismo\",\"s\":3180,\"e\":3190},{\"t\":\"aggiungere\",\"s\":3194,\"e\":3202},{\"t\":\"premier\",\"s\":3206,\"e\":3213},{\"t\":\"corso\",\"s\":3218,\"e\":3223},{\"t\":\"assemblea\",\"s\":3230,\"e\":3239},{\"t\":\"Partito\",\"s\":3244,\"e\":3251},{\"t\":\"Democratico\",\"s\":3252,\"e\":3263},{\"t\":\"poi\",\"s\":3267,\"e\":3270},{\"t\":\"toccare\",\"s\":3271,\"e\":3278},{\"t\":\"tema\",\"s\":3291,\"e\":3295},{\"t\":\"liquidare\",\"s\":3296,\"e\":3306},{\"t\":\"parola\",\"s\":3310,\"e\":3316},{\"t\":\"volgare\",\"s\":3334,\"e\":3341},{\"t\":\"inaccettabile\",\"s\":3344,\"e\":3357},{\"t\":\"ribadire\",\"s\":3358,\"e\":3367},{\"t\":\"Italia\",\"s\":3375,\"e\":3381},{\"t\":\"proseguire\",\"s\":3382,\"e\":3392},{\"t\":\"impegno\",\"s\":3401,\"e\":3408},{\"t\":\"europeista\",\"s\":3409,\"e\":3419},{\"t\":\"fare\",\"s\":3426,\"e\":3431},{\"t\":\"condizionare\",\"s\":3432,\"e\":3444},{\"t\":\"riforma\",\"s\":3457,\"e\":3464},{\"t\":\"Gentiloni\",\"s\":3465,\"e\":3474},{\"t\":\"sottolineare\",\"s\":3478,\"e\":3490},{\"t\":\"Italia\",\"s\":3494,\"e\":3500},{\"t\":\"intenzione\",\"s\":3516,\"e\":3526},{\"t\":\"ammainare\",\"s\":3530,\"e\":3539},{\"t\":\"bandiera\",\"s\":3543,\"e\":3551},{\"t\":\"anzi\",\"s\":3552,\"e\":3556},{\"t\":\"dire\",\"s\":3557,\"e\":3561},{\"t\":\"maggioranza\",\"s\":3573,\"e\":3584},{\"t\":\"Parlamento\",\"s\":3588,\"e\":3598},{\"t\":\"periodo\",\"s\":3622,\"e\":3629},{\"t\":\"tempo\",\"s\":3633,\"e\":3638},{\"t\":\"congruo\",\"s\":3639,\"e\":3646},{\"t\":\"fare\",\"s\":3651,\"e\":3655},{\"t\":\"cosa\",\"s\":3663,\"e\":3667},{\"t\":\"bastare\",\"s\":3672,\"e\":3677},{\"t\":\"invece\",\"s\":3678,\"e\":3684},{\"t\":\"bozza\",\"s\":3688,\"e\":3693},{\"t\":\"decreto\",\"s\":3697,\"e\":3704},{\"t\":\"mettere\",\"s\":3705,\"e\":3710},{\"t\":\"appunto\",\"s\":3711,\"e\":3718},{\"t\":\"esecutivo\",\"s\":3725,\"e\":3734},{\"t\":\"Gentiloni\",\"s\":3738,\"e\":3747},{\"t\":\"evitare\",\"s\":3752,\"e\":3759},{\"t\":\"nuovo\",\"s\":3763,\"e\":3768},{\"t\":\"sciopero\",\"s\":3769,\"e\":3777},{\"t\":\"taxi\",\"s\":3782,\"e\":3786},{\"t\":\"oggi\",\"s\":3787,\"e\":3791},{\"t\":\"8\",\"s\":3798,\"e\":3799},{\"t\":\"22\",\"s\":3805,\"e\":3807},{\"t\":\"prevedere\",\"s\":3811,\"e\":3820},{\"t\":\"disagio\",\"s\":3821,\"e\":3827},{\"t\":\"sciopero\",\"s\":3835,\"e\":3843},{\"t\":\"indire\",\"s\":3844,\"e\":3851},{\"t\":\"tassista\",\"s\":3856,\"e\":3864},{\"t\":\"ieri\",\"s\":3865,\"e\":3869},{\"t\":\"fine\",\"s\":3875,\"e\":3879},{\"t\":\"bozza\",\"s\":3883,\"e\":3888},{\"t\":\"decreto\",\"s\":3892,\"e\":3899},{\"t\":\"proporre\",\"s\":3900,\"e\":3908},{\"t\":\"Governo\",\"s\":3913,\"e\":3920},{\"t\":\"scontentare\",\"s\":3924,\"e\":3935},{\"t\":\"titolare\",\"s\":3942,\"e\":3950},{\"t\":\"taxi\",\"s\":3954,\"e\":3958},{\"t\":\"unione\",\"s\":3961,\"e\":3967},{\"t\":\"nazionale\",\"s\":3968,\"e\":3977},{\"t\":\"consumatore\",\"s\":3978,\"e\":3989},{\"t\":\"ultimo\",\"s\":3999,\"e\":4005},{\"t\":\"tentativo\",\"s\":4006,\"e\":4015},{\"t\":\"servire\",\"s\":4022,\"e\":4029},{\"t\":\"sbloccare\",\"s\":4032,\"e\":4041},{\"t\":\"vertenza\",\"s\":4045,\"e\":4053},{\"t\":\"prossimo\",\"s\":4062,\"e\":4070},{\"t\":\"ora\",\"s\":4071,\"e\":4074},{\"t\":\"nuovo\",\"s\":4083,\"e\":4088},{\"t\":\"sciopero\",\"s\":4089,\"e\":4097},{\"t\":\"proposta\",\"s\":4101,\"e\":4109},{\"t\":\"presentare\",\"s\":4110,\"e\":4120},{\"t\":\"viceministro\",\"s\":4125,\"e\":4137},{\"t\":\"trasporto\",\"s\":4142,\"e\":4151},{\"t\":\"vicino\",\"s\":4152,\"e\":4163},{\"t\":\"regolamentazione\",\"s\":4172,\"e\":4188},{\"t\":\"Ncc\",\"s\":4193,\"e\":4196},{\"t\":\"auto\",\"s\":4197,\"e\":4201},{\"t\":\"bianco\",\"s\":4202,\"e\":4209},{\"t\":\"essere\",\"s\":4212,\"e\":4217},{\"t\":\"apprezzare\",\"s\":4218,\"e\":4228},{\"t\":\"approccio\",\"s\":4235,\"e\":4244},{\"t\":\"bastare\",\"s\":4254,\"e\":4261},{\"t\":\"sigla\",\"s\":4279,\"e\":4284},{\"t\":\"taxi\",\"s\":4292,\"e\":4296},{\"t\":\"aderente\",\"s\":4301,\"e\":4309},{\"t\":\"Legacoop\",\"s\":4312,\"e\":4320},{\"t\":\"ribadire\",\"s\":4324,\"e\":4332},{\"t\":\"scioperare\",\"s\":4341,\"e\":4351},{\"t\":\"utero\",\"s\":4352,\"e\":4357},{\"t\":\"teso\",\"s\":4358,\"e\":4362},{\"t\":\"fronte\",\"s\":4366,\"e\":4372},{\"t\":\"economico\",\"s\":4373,\"e\":4382},{\"t\":\"mercato\",\"s\":4387,\"e\":4394},{\"t\":\"Paolo\",\"s\":4399,\"e\":4404},{\"t\":\"Trombi\",\"s\":4405,\"e\":4411},{\"t\":\"partire\",\"s\":4415,\"e\":4423},{\"t\":\"subito\",\"s\":4424,\"e\":4430},{\"t\":\"mattinata\",\"s\":4447,\"e\":4456},{\"t\":\"Tokyo\",\"s\":4457,\"e\":4462},{\"t\":\"chiudere\",\"s\":4466,\"e\":4472},{\"t\":\"rialzo\",\"s\":4476,\"e\":4482},{\"t\":\"0,23%\",\"s\":4489,\"e\":4494},{\"t\":\"torno\",\"s\":4500,\"e\":4505},{\"t\":\"stabilità\",\"s\":4511,\"e\":4520},{\"t\":\"invarianza\",\"s\":4529,\"e\":4539},{\"t\":\"mercato\",\"s\":4540,\"e\":4547}]}",
	"entity_suggest_Province": ["Roma"],
	"entity_suggest_Location": ["Bretagna", "Italia", "Londra", "Tokyo", "Tamigi", "Campidoglio", "Westminster Bridge", "ponte di Westminster", "Paese", "Westminster"],
	"conceptualmap": "rO0ABXNyAEJpdC5hbG1hd2F2Gh0dHA6Ly9hbG1hd2F2ZS5pdC9vbnRvbG9naWVzLzIwMTcvMDIvMDcvQ3JhaW1JbnQub3dsI1RlbGVjYW1lcmF4c3EAfgANAHNxAH4AEnB3BAAAAAB4cHNxAH4AGT9AAAAAAAAMdwgAAAAQAAAABnEAfgAbdAAHT2dnZXR0aXEAfgAddAAHT2dnZXR0aXEAfgAfdAAHT2dnZXR0aXEAfgAhdAAHT2dnZXR0aXEAfgAjdAAHT2dnZXR0aXEAfgAldAAHT2dnZXR0aXhzcQB+ABk/QAAAAAAAEHcIAAAAEAAAAAB4cQB+AChxAH4AKXQAPWh0dHA6Ly9hbG1hd2F2ZS5pdC9vbnRvbG9naWVzLzIwMTcvMDIvMDcvQ3JhaW1JbnQub3dsI09nZ2V0dGl4c3EAfgANAHNxAH4AEnB3BAAAAAB4cHNxAH4AGT9AAAAAAAAQdwgAAAAQAAAAAHhzcQB+ABk/QAAAAAAAEHcIAAAAEAAAAAB4cHB0AAVUaGluZw==",
	"named_entity_Organization": "Partito Democratico[:]Parlamento[:]Parlamento[:]Parlamento[:]Parlamento",
	"X-PARSED-BY": "org.apache.tika.parser.DefaultParser",
	"uri": "file%3A%2F%2F%2FNFS_IRIDE%2Fcraim-svil%2Foutput%2Fpvt_ITA%2Fita_Canale5_20170323_00006.txt",
	"entity_suggest_Comuni": ["Force", "Mattinata", "Romana", "Paese"],
	"size": 4571,
	"CONTENT-ENCODING": "UTF-8",
	"named_entity_Person": "Paolo Trombi[:]Theresa May[:]Gentiloni",
	"resourceURI": "http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Telecamera[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Londra[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Roma[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Londra[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Agente_di_Polizia[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Londra[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Terrorismo[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Italia[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Italia[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Italia[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Celebrare[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Manifestazione[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Londra[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Roma[;]http://almawave.it/ontologies/2017/02/07/CraimInt.owl#Londra[;]",
	"ANOTHER_FILE": "/NFS_IRIDE/craim-svil/output/pvt_ITA/ita_Canale5_20170323_00006.txt",
	"language": "it",
	"CONFIDENTIAL_LEVEL_LABEL": "UNCLASSIFIED"
}		
```
@[15](Campo analizzato con boosting delle entità)
@[22](Campo mappato per il suggerimento delle entità)

---

# THE END

