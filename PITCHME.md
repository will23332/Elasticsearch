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
La risposta:
```
{
  "_index" : "customer",
  "_type" : "_doc",
  "_id" : "1",
  "_version" : 1,
  "result" : "created",
  "_shards" : {
    "total" : 2,
    "successful" : 1,
    "failed" : 0
  },
  "_seq_no" : 0,
  "_primary_term" : 1
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
La risposta sarà vuota
```
health status index uuid pri rep docs.count docs.deleted store.size pri.store.size
```

---

## Installazione Cerebro
[Link](https://github.com/lmenezes/cerebro)

Cerebro è una GUI che permette di interagire con Elasticsearch.

---

## Lucene

+Document, Field, Token
+Analyzer, Filter, TokenFilter, CharFilter
+Inverted Index, DF/ITF, Vector Space Model
+Similarity
+Search Boolean, Fuzzy

---

# THE END

