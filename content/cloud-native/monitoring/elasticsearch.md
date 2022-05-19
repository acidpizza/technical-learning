---
title: "Elasticsearch"
weight: 1
---

## Resources

- https://thoughts.t37.net/designing-the-perfect-elasticsearch-cluster-the-almost-definitive-guide-e614eabc1a87

### Kibana

- https://www.elastic.co/blog/visualizing-observability-with-kibana-event-rates-and-rate-of-change-in-tsvb
- https://www.elastic.co/blog/timelion-tutorial-from-zero-to-hero
- https://coralogix.com/log-analytics-blog/advanced-guide-to-kibana-timelion-functions/

## REST API

```bash
# curl
curl -X PUT localhost:9200/_cluster/settings -H 'Content-Type: application/json' -d '{ json }'

# Remove replicas
# Replace * with index name for specific index
PUT localhost:9200/*/_settings 
{ 
  "index" : { 
    "number_of_replicas" : 0 
  } 
}

# Drain node
PUT localhost:9200/_cluster/settings 
{
  "transient" : {
    "cluster.routing.allocation.exclude._ip": "x.x.x.x,y.y.y.y",
    "cluster.routing.allocation.node_concurrent_recoveries": "10",
    "cluster.routing.allocation.cluster_concurrent_rebalance": "10"
  }
}

# Remove voting
POST localhost:9200/_cluster/voting_config_exclusions?node_names=xx,yy 

# View cluster status
GET localhost:9200/_cluster/health?pretty

```

## Downgrading Cluster to Single Node

1. Remove replicas
2. Drain nodes
3. Remove master nodes by adding to voting exclusion
4. Ensure es01 is the master (down other masters and bring up again if necessary)

> Hack: remove _state/ folder and lock file in data folder but keep the indices/ folder if nodes were deleted without properly removing voting rights.


## Reindex to Change Field Mapping Type

```bash
# Get index mapping
curl -XGET localhost:9200/my-index-01/_mapping

# Create new index
curl -XPUT localhost:9200/my-index-01-reindex -H 'Content-Type: application/json' -d '{
  "mappings": {
    XXX
  }
}'

# Create ingest pipeline
curl -X PUT "localhost:9200/_ingest/pipeline/my-pipeline-id?pretty" -H 'Content-Type: application/json' -d '{
  "description" : "describe pipeline",
  "processors" : [
    {
      "convert" : {
        "field": "_source.field_name",
        "type": "integer"
      }
    }
  ]
}'

# Reindex
curl -XPOST localhost:9200/_reindex?pretty -H Content-Type:application/json -d '{
  "source": {
    "index": "my-index-01"
  },
  "dest": {
    "index": "my-index-01-reindex"
  }
  "script": {
    "lang": "painless",
    "source": "if (ctx._source.request_content_length == \"-\" || ctx._source.request_content_length == \"UNKNOWN\") {ctx._source.request_content_length = 0} if (ctx._source.response_content_length == \"-\" || ctx._source.response_content_length == \"UNKNOWN\") {ctx._source.response_content_length = 0}"
  }
}'
```