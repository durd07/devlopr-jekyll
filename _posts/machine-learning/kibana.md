---
title: Kibana & Elasticsearch
---

# Kibana & Elasticsearch

## Install

docker-compose.yml

```
version: "3.2"

services:
  elasticsearch:
     image: elasticsearch:7.12.1
     restart: always
     environment:
       discovery.type: single-node
     ports:
       - "9200:9200"
       - "9300:9300"
  kibana:
    image: kibana:7.12.1
    restart: always
    environment:
      ELASTICSEARCH_URL: "127.0.0.1:9200"
      I18N_LOCALE: "en-US"
    ports:
       - "5601:5601"
```

Kibana: `http://127.0.0.1:5601` 
Elasticsearch: `http://127.0.0.1:9200`

## Commands

```sh
# Query
GET logbot/_search
{
  "track_total_hits": true,  # query all items, otherwise max is 10000
  "query":{
    "match": {
      "type": "callload"
    }
  }
}

curl -X GET 'http://10.99.133.65:9200/logbot/_search' -H "Content-Type: application/json" -d '{"track_total_hits": true,"query": {"bool": {"must": {"match":{"type":"pm"}}}}}' | python -m json.tool

# Delete
POST logbot/_delete_by_query?conflicts=proceed
{
  "query":{
    "match": {
      "type": "callload"
    }
  }
}

curl -X GET 'http://10.99.133.65:9200/logbot/_search' -H "Content-Type: application/json" -d '{"query": {"bool": {"must": {"match":{"type":"pm"}}}}}' | python -m json.tool

curl -X POST 'http://10.99.133.65:9200/logbot/_delete_by_query?conflicts=proceed&timeout=5m' -H "Content-Type: application/json" -d '{"query": {"bool": {"must": {"match":{"type":"pm"}}}}}' | python -m json.tool

# Count
curl -s -X GET 'http://10.99.133.65:9200/logbot/_count?q=type:pm' -H "Content-Type: application/json" | python -m json.tool
```





[Vega | Kibana Guide [7.15\] | Elastic](https://www.elastic.co/guide/en/kibana/current/vega.html)
