# Databases

Within the application the appropriate database is used for the different application scenarios. In total COLID uses three databases:
* [Graph Database](/infrastructure/databases/graph-database)
* [ElasticSearch](/infrastructure/databases/elasticsearch)
* [Relational Database](/infrastructure/databases/relational-database)

While the primary data of the linked COLID entries are stored in a graph database, secondary user settings of the AppData Service as well as working data of the Scheduler Service are stored in a relational database.
For the use case of the full text search over the COLID entries the document database of Elasticsearch, a search engine based on Lucene, is used.
