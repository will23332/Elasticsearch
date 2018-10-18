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
@[1-3](Init Spark cluster data source)
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

##Use Case
Regione Veneto:

---
## Index Mapping
+ Analyzer
+ Similarity

---

# THE END

