# Elasticsearch
![](elasticsearch/assets/elastic-search-logo-color-horizontal.svg)

Elasticsearch is a distributed search and analytics engine. Elasticsearch provides real-time search and analytics for all types of data. Whether for structured or unstructured text, numerical data, or geospatial data, Elasticsearch can efficiently store and index it in a way that supports fast searches.

Data Marketplace is using Elasticsearch to store the COLID entries in an index and to provide a full text search over all entries to make them findable.


## Data in: documents and indices

Elasticsearch is a distributed document store. Instead of storing information as rows of columnar data, Elasticsearch stores complex data structures that have been serialized as JSON documents. The used data structure called an inverted index supports very fast full-text searches. An index is an optimzed collection of documents. A document is a collection of fields, which key value pairs that contain the data.


## Information out: search and analyze

While you can use Elasticsearch as a document store and retrieve documents and their metadata, the real power comes from being able to easily access the full suite of search capabilities built on the Apache Lucene search engine library. A wide range of queries allows to perform different kind of searches like full-text, phrase, similarity and prefix to find documents that match the query and return them sorted by relevance. In addition to searching Elasticsearch aggregations enable you to build complex summaries of your data and gain insights into key metrics, patterns nad trends.

For more details about Elasticsearch visit the offical [documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/elasticsearch-intro.html) by elastic.

## Amazon Elasticsearch Service

More information for managed Elasticsearch service by Amazon can be found [here](elasticsearch/amazon_elasticsearch_service.md).