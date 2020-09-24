# Create an index

This section explains how to create new index in Elasticsearch based on a current set of metadata. 

## API Usage

```shell
POST "https://localhost:51800/api/index/create" 
{
	"URI-of-property" : {
      "key" : "some-URI",
      "properties" : {
        "key1": "value1",
        ...
      },
      "nestedMetadata" : [ ]
    },
}
```

## Description

An index is an optimized collection of documents which will be used as data basis by Elasticsearch. A document represents a COLID entry. Before a COLID entry becomes findable in Data Marketplace it has to be stored in the search index. But first the index has to be existent.

Calling this API endpoint it will create a new index with an automatically generated explicit index mapping based on the metadata in the body of the request. More details on the explicit index mapping can be found in [Index Mapping](concepts/index.md).

It is recommend not to call this single API endpoint. The creation of a new index should always happen as part of the so called reindexing process. A reindexing process is a chain of operations which will ensure that the search service will be based on the latest and consistent data. The [Indexing Crawler Service](docs/application-parts/indexing-crawler-service.md) implements the whole reindexing process and calls also this API endpoint to create a new index.