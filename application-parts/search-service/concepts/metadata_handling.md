# Metadata handling

The most important data for Data Marketplace are the registered resources in COLID Editor. This data is transferred to Data Marketplace backend and stored in a search index to make them findable. But to make this happen it is essential to have the corresponding metadata available.

The metadata provides information about the content of certain resources from COLID Editor. It describes the data coming from COLID Editor. The particular metadata consist of the types descriptive, structural and administrative metadata. The metadata provides information such as a description of the contents, types, standards used, labels, limitations and much more.

## Requirements

COLID Editor metadata are crucial for the Data Marketplace application. They are needed within the whole application like setting up the search index or displaying information in the frontend. But the data is subject to change as part of the software evolution process. This means the metadata will change as well. Therefore a flexible concept of metadata handling in Data Marketplace is necessary. Changes in the metadata should not need any manually modifications in the application. 

The following requirements exist for the metadata handling in Data Marketplace:

- Metadata handling has to be flexible
- Changes can be applied at any time
- Avoid manually changes in the application
- Ensure integrity between current metadata and COLID Editor resource data

## Concept

COLID Editor metadata has to become available and stored in Data Marketplace in a way that it will meet the requirements from above. The following concept for the metadata handling is implemented.

### Transfer of metadata

Metadata has to be transferred from COLID Editor to Data Marketplace. For this reason COLID Editor has an API endpoint `GET /api/v3/metadata/mapping` which provides current metadata. Metadata has to be transferred at initial build of Data Marketplace and in case of any changes on the metadata. 

### Storage of metadata in Data Marketplace

Metadata are stored in a dedicated elasticsearch index. This index has only the purpose to store the data and will not be used for full text search. The index is called `dmp_metadata` and contains only one document. This document will be overwritten always to ensure the usage of the current metadata. 

Below you can find the command to create the metadata index in elasticsearch with special settings: Apply the enabled setting for the mapping type `_doc`. This causes Elasticsearch to skip parsing of the contents of the field entirely. The JSON can still be retrieved from the `_source` field, but it is not searchable or stored in any other way. On metadata no need to query or run aggregations on the data itself. 

```JSON
PUT dmp_metadata
{
  "mappings": {
    "doc": { 
      "enabled": false
    }
  }
}	
```

For performance reasons metadata will be cached in Data Marketplace backend so that only needs to be read once from elasticsearch on startup. 

Any change in the COLID Editor metadata requires a complete transfer of all current metadata to Data Marketplace. Document in in `dmp_metadata` index will be always overwritten with new metadata.  Any update on the `dmp_metadata` index requires a complete reindex of the COLID Editor resources data.

### Trigger reindex of Data Marketplace search index

To ensure integrity between the metadata und COLID Editor resources data a complete reindex has to be triggered. It is essential to avoid any inconsistency. Data Marketplace backend uses the metadata for several purposes as for example to build the elasticsearch index. If the format of the resources which are send from COLID Editor to Data Marketplace will not fit to current search index they cannot serve full search functionality and experience to the users.  Any inconsistency will lead to misbehave of Data Marketplace application.

Reindex process includes the following steps:

- Build index mapping based on current metadata
- Create new index with created index mapping
- Index all resources from COLID Editor
- Delete old index

More details on the implementation of the reindexing process can be found at the documentation for the [Indexing Crawler Service](application-parts/indexing-crawler-service.md).

