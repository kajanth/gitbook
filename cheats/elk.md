# ELK

## Referentes

Cheat Sheet: [https://lzone.de/cheat-sheet/ElasticSearch](https://lzone.de/cheat-sheet/ElasticSearch)

## Troubleshoot

```text
curl -XGET 'http://localhost:9200/_cluster/state?pretty=true'
curl -XGET 'http://localhost:9200/_cluster/health?pretty'
curl -XGET 'http://localhost:9200/_cat/indices'
curl -XGET localhost:9200/_cluster/allocation/explain?pretty
curl -XGET http://localhost:9200/_cluster/allocation/explain?include_disk_info=true&pretty
curl -XGET localhost:9200/_cat/shards?h=index,shard,prirep,state,unassigned.reason| grep UNASSIGNED
```

```text
curl -XPOST 'localhost:9200/_cluster/reroute?retry_failed'
curl -XDELETE 'localhost:9200/my-index-name/'
```



