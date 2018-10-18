# Elasticsearch Crash Course

![LOGO](https://cdn.freebiesupply.com/logos/large/2x/elasticsearch-logo-png-transparent.png)

+++

A cura di Guglielmo Piacentini e Alfredo Serafini

---

### Che cos'è Elasticsearch?

Un motore di ricerca e analisi full-text, open-source.

---

# Come lo uso?

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

# Com'è fatto?

Elasticsearch gira in cluster. 
Un cluster può essere formato da uno o più server.
Ogni server nel cluster è un nodo. 
ES stocca i documenti in indici. 
Un indice può essere "scomposto" in shard.

---

### THE END

