# Index mapping

Concept for programmatically generation of elasticsearch index mapping based on COLID Editor metadata ontology.

## Goal

Leverage available metadata from COLID Editor to determine how the elasticsearch index should look like. With the information which describes the data from COLID Editor and some mapping rules it is possible to generate automatically the elasticsearch index mapping.

The big advantage of automatization of this process is the time which can be saved. Changes in the data from COLID Editor does not require manually changes by a developer in the elasticsearch index mapping. Fields can be added, changed or removed. Index will be automatically generated based on the current metadata. The Data Marketplace application gets a high flexibility and changes in the COLID Editor data can be quickly applied in Data Marketplace without any manual changes.



## Explicit mapping

A schema is a description of one or more fields that describes how to handle different fields of a document. In elasticsearch the schema is a mapping that defines how a document, and the fields it contains, are stored and indexed. Because of this the schema is called “mapping” in elasticsearch.

Elasticsearch support two different concepts for mapping. The first one is dynamic mapping. Fields and mapping type do not to be defined before being used. Thanks to dynamic mapping, new field names will be added automatically, just by indexing a document. This concept is useful to get started, but for Data Marketplace rich features not sufficient. Fields have to be analyzed with special analyzers and stored in different datatypes to meet the requirements for the application. Instead the concept of specifying an explicit mapping is used. 

An explicit mapping is provided to elasticsearch as hierarchically structured JSON during index creation. This file contains a list of fields and properties pertinent to the document which should be indexed.



## Concept

Below are all steps listed for the creation of a new index. 

1. **Create Document Index**

   Create a new index on elasticsearch to store and search resources. Index will be created with static index settings. Index settings are described futher down on this page.

   After successful creation of the index the `DocumentUpdateAlias` will be updated to the just created index. 

   Rollback Mechanism: 

   - Delete Index
   - Undo update on `DocumentUpdateAlias`

2. **Create Index Mapping for search index**

   Based on metadata the mapping for the created index in step 1 is dynamically created.

   Rollback Mechanism:

   - Not needed, because in case of an exception the index will be deleted.

3. **Create Metadata Index**

   Creates a new index without a mapping to store the metadata.

   After successful creation of the index the `MetadataUpdateAlias` will be updated to the just created index. 

   Rollback Mechanism:

   - Delete Index
   - Undo update on `MetadataUpdateAlias`

4. **Index Metadata**

   Indexes the current metadata set to the update metadata index.

   Rollback Mechanism:

   - Not needed, because in case of an exception the index will be deleted.

5. **Update Aliases**

   Updates the `MetadataSearchAlias` and `ResourceSearchAlias` to currently created indices.

   Rollback Mechanism:

   - Undo update on `MetadataSearchAlias`
   - Undo update on `ResourceSearchAlias`

6. **Delete old indices**

   Deletes all old indices after successful creation of all indices and update of aliases.

   Rollback Mechanism:

   - It is ignored whether the deletion was successful or not.


   

## Implementation

This section contains relevant information and documents related to the implementation.

### Mapping rules

Rules for creating elasticsearch mapping based on metadata.

#### Disable dynamic mapping

First of all turn off dynamic mapping. Any unknown fields will not be added to the mapping and will not be searchable. Mapping explosion will be avoided.

```json
PUT <index>/_mapping/_doc
{
  "dynamic":false
}
```

#### Rules of index mapping

Based on the available metadata of COLID Editor resources the index mapping should be generated dynamically.  Therefore the metadata will be used as input and the following mapping rules will determine how the resources will be indexed as documents in elasticsearch.

##### Properties & Fields

###### Static Fields

1. Add resourceID as unique identifer of a document in the index.

   ```json
   "resourceId": {      
             "type": "keyword"
       }
   ```

2. Add fields for autocompletion functionality.

   ```json
   "autocomplete_filter": {
         "type": "text",
         "analyzer": "autocomplete_ngram",
         "search_analyzer": "search_autocomplete"
       },
       "autocomplete_terms": {
         "type": "text",
         "fielddata": true,
         "analyzer": "autocomplete_shingle"
       },
       "dym_trigram": {
         "type": "text",
         "analyzer": "dym_trigram"
       }
   ```

   

###### Fields based on metadata

1. Mapping of URI fields

   If field is of type URI, add it as keyword.

   Check metadata:

   | Key                                        | Value                                   |
   | ------------------------------------------ | --------------------------------------- |
   | http://www.w3.org/2000/01/rdf-schema#range | http://www.w3.org/2001/XMLSchema#anyURI |

   Definition:

   ```json
   "properties": {
           "value": {
             "type": "keyword",      
   		  "normalizer": "lowercase",		  
             "fields": {
               "text": {
                 "type": "text",
   			  "analyzer": "dmp_standard_english"			  
               },
   			"ngrams":{
   				"type": "text",
   				"analyzer": "dmp_ngram_analyzer",
   				"search_analyzer": "dmp_standard_english"
   			}              
             }
           }
         }
   ```

   

2. Mapping of dates

   Check metadata

   | Key                                          | Value                                     |
   | -------------------------------------------- | ----------------------------------------- |
   | <http://www.w3.org/2000/01/rdf-schema#range> | http://www.w3.org/2001/XMLSchema#dateTime |

   ```json
   "properties": {
           "value": {
             "type": "date"
           }
   }
   ```

   

3. Mapping of version

   Currently only for the version field.

   Check metadata:

   | Key                                          | Value                                  |
   | -------------------------------------------- | -------------------------------------- |
   | <http://www.w3.org/2000/01/rdf-schema#range> | http://www.w3.org/2001/XMLSchema#float |

   ```json
   "properties": {
           "value": {
             "type": "keyword"
           }
   }
   ```

   

4. Mapping of boolean

   Check metadata:

   | Key                                          | Value                                    |
   | -------------------------------------------- | ---------------------------------------- |
   | <http://www.w3.org/2000/01/rdf-schema#range> | http://www.w3.org/2001/XMLSchema#boolean |

   ```json
   "properties": {
           "value": {
             "type": "keyword"
           }
         }
   ```

   

5. Mapping of persons

   Check metadata

   | Key                                          | Value                                          |
   | -------------------------------------------- | ---------------------------------------------- |
   | <http://www.w3.org/2000/01/rdf-schema#range> | http://COLID Editor.bayer.com/kos/19014/Person |

   ```json
   "properties": {
       "value": {
           "type": "keyword",      
           "normalizer": "lowercase",	
           "copy_to": ["dym_trigram"],
           "fields": {
               "text": {
                   "type": "text",
                   "analyzer": "dmp_standard_english"			  
               },
               "ngrams":{
                   "type": "text",
                   "analyzer": "dmp_ngram_analyzer",
                   "search_analyzer": "dmp_standard_english"
               }              
           }
       }
    }
   ```

   

6. Mapping of literals/text

   Check Metadata:

   | Key                                   | Value                                |
   | ------------------------------------- | ------------------------------------ |
   | "http://www.w3.org/ns/shacl#nodeKind" | <http://www.w3.org/ns/shacl#Literal> |

   AND NOT

   | Key                                        | Value                                            |
   | ------------------------------------------ | ------------------------------------------------ |
   | http://www.w3.org/2000/01/rdf-schema#range | <http://COLID Editor.bayer.com/kos/19014/Person> |

   ```json
   "properties": {
           "value": {
             "type": "keyword",          
             "copy_to": ["dym_trigram",
               "autocomplete_filter",
               "autocomplete_terms"],
             "fields": {
               "text": {
                 "type": "text",
   			  "analyzer": "dmp_standard_english"			  
               },
   			"ngrams":{
   				"type": "text",
   				"analyzer": "dmp_ngram_analyzer",
   				"search_analyzer": "dmp_standard_english"
   			}              
             }
           }
         }
   ```

   

7. Mapping of IRIs

   Check metadata:

   | Key                                 | Value                            |
   | ----------------------------------- | -------------------------------- |
   | http://www.w3.org/ns/shacl#nodeKind | <http://www.w3.org/ns/shacl#IRI> |

   ```json
    "properties": {
           "uri": {
             "type": "keyword"
           },
           "value": {
             "type": "keyword",          
             "copy_to": ["dym_trigram"],
             "fields": {
               "text": {
                 "type": "text",
   			  "analyzer": "dmp_standard_english"			  
               },			
   			"ngrams":{
   				"type": "text",
   				"analyzer": "dmp_ngram_analyzer",
   				"search_analyzer": "dmp_standard_english"
   			}  
             }
           }
         }
   ```

   

8. Mapping of distribution endpoints

   Check Metadata

   | Key                                                       | Value                                                   |
   | --------------------------------------------------------- | ------------------------------------------------------- |
   | <http://COLID Editor.bayer.com/kos/19014/hasCOLID Editor> | <https://COLID Editor.bayer.com/kos/19050/distribution> |

   
   ```json
   "properties": {
   			"uri": {
   			  "type": "keyword"
   			},
   			"value": {
   			  "type": "keyword",          
   			  "fields": {
   				"text": {
   				  "type": "text",
   				  "analyzer": "dmp_standard_english"			  
   				},
   				"ngrams":{
   					"type": "text",
   					"analyzer": "dmp_ngram_analyzer",
   					"search_analyzer": "dmp_standard_english"
   				}				
   			  }
   			},
   			"nested": {
   				"type": "nested",
   				"include_in_parent": true,
   				"properties": {
   					"http://www.w3.org/1999/02/22-rdf-syntax-ns#type":{
   						"properties": {
   							"uri": {
   							  "type": "keyword"
   							},
   							"value": {
   							  "type": "keyword",          					  
   							  "fields": {
   								"text": {
   								  "type": "text",
   								  "analyzer": "dmp_standard_english"			  
   								},
   								"ngrams":{
   									"type": "text",
   									"analyzer": "dmp_ngram_analyzer",
   									"search_analyzer": "dmp_standard_english"
   								}								
   							  }
   							}	
   						}
   						
   					},
   					"https://COLID Editor.bayer.com/kos/19050/hasNetworkedResourceLabel":{
   						"properties": {
   							"uri": {
   							  "type": "keyword"
   							},
   							"value": {
   							  "type": "keyword",          					  
   							  "fields": {
   								"text": {
   								  "type": "text",
   								  "analyzer": "dmp_standard_english"			  
   								},								
   								"ngrams":{
   									"type": "text",
   									"analyzer": "dmp_ngram_analyzer",
   									"search_analyzer": "dmp_standard_english"
   								}
   							  }
   							}
   						}	
   					},
   					"http://COLID Editor.bayer.com/kos/19014/hasNetworkAddress":{
   						"properties": {
   							"uri": {
   							  "type": "keyword"
   							},
   							"value": {
   							  "type": "keyword",          					  
   							  "fields": {
   								"text": {
   								  "type": "text",
   								  "analyzer": "dmp_standard_english"			  
   								},
   								"ngrams":{
   									"type": "text",
   									"analyzer": "dmp_ngram_analyzer",
   									"search_analyzer": "dmp_standard_english"
   								}           
   							  }
   							}
   						}	
   					},
   					"http://COLID Editor.bayer.com/kos/19014/hasCOLID Editor":{
   						"properties": {
   							"uri": {
   							  "type": "keyword"
   							},
   							"value": {
   							  "type": "keyword",          					  
   							  "fields": {
   								"text": {
   								  "type": "text",
   								  "analyzer": "dmp_standard_english"			  
   								},
   								"ngrams":{
   									"type": "text",
   									"analyzer": "dmp_ngram_analyzer",
   									"search_analyzer": "dmp_standard_english"
   								}            
   							  }
   							}
   						}	
   					},
   					"https://COLID Editor.bayer.com/kos/19050/hasContactPerson":{
   						"properties": {
   							"uri": {
   							  "type": "keyword"
   							},
   							"value": {
   							  "type": "keyword",
   								"normalizer": "lowercase",	
   							  "copy_to": ["dym_trigram"],
   							  "fields": {
   								"text": {
   								  "type": "text",
   								  "analyzer": "dmp_standard_english"			  
   								},
   								"ngrams":{
   									"type": "text",
   									"analyzer": "dmp_ngram_analyzer",
   									"search_analyzer": "dmp_standard_english"
   								}            
   							  }
   							}
   						}	
   					}
   
   				}
   			}
         }
   ```



### Index settings

Index settings will determine the behaviour of the index. For example custom analyzers will be defined in the settings. Index settings are static and can read from a file.

Current index settings:

filter

​    filter_autocomplete_ngram

​    filter_autocomplete_shingle

​    english_stopwords

normalizer

​    lowercase

analyzer

​    autocomplete_ngram

​		Tokenizers: lowercase

​		Filters: lowercase, asciifolding, filter_autocomplete_ngram

​    search_autocomplete

​		Tokenizers: lowercase

​		Filters: lowercase, asciifolding

​    dym_trigram

​		Tokenizers: lowercase

​		Filters: standard, lowercase, asciifolding, filter_autocomplete_shingle

​    autocomplete_shingle

​		Tokenizers: lowercase

​		Filters: lowercase, asciifolding, filter_autocomplete_shingle

 dmp_ngram_analyzer    

​		Tokenizer: dmp_ngram_tokenizer

​		Filters: lowercase, asciifolding

 dmp_standard_english

​		Tokenizers: standard

​		Filters:  lowercase, english_stopword, asciifolding

tokenizer

​    dmp_ngram_tokenizer



