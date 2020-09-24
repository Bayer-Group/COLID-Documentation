# Handling of Taxonomies

To provide a semantic search in Data Marketplace taxonomies will be added. This will extend the the Data Marketplace search to find relating concepts and ideas in the documents and queries. The search will not only find a hit for strict matching terms, it will also find a hit even though they look different. 

## Why to use taxonomies?

Taxonomies will help modeling broader/narrower relationships and to increase the recall and precision of the search results. Lets have a look on a simple example to better understand the impact on the search. The following simple taxonomy is given:

```
Computer
|___ Laptop
	|___ Mini
```

To build a semantic search we want to get away from only strict matching of terms. If a user types in "Computer" we want to find all documents that contain the term computer. But using the taxonomy from above we also want to get results which contain the terms "Laptop" and "Mini", even if the terms are never mentioned in the query. The search has to be able to understand that "Laptop" and "Mini" are narrower concepts of a computer.

A general pattern in how users searches is that they often start broad to explore a data set. For example searching with the term "Computer". The user does not know the terminology but is also interested in the results of "Laptop". This is the reason why users should have an option to get an overview of the data and refine his search with broader/narrower terms to get more or less specific with their terminology. 

## Requirements

The following requirements should be meet by adding taxonomies to the Data Marketplace search:

- Allow users to search with broader term of a taxonomy and get also hits for narrower concepts of the search term.
- Hits with the exact search term should be ranked higher than the narrower concepts.
- Allow user to investigate the taxonomy and use the entries of the taxonomy as filter for further searches. 

## Implementation

The following chapter will describe how to use taxonomies in Elasticsearch to improve the search relevance.

### Data Ingestion

Taxonomies will be included in the metadata which will be send to Data Marketplace. For each field in the metadata it will be checked if a taxonomy is available. If the check is positive, the taxonomy will be added to this field. Taxonomies are stored in the field: `taxonomy`

This task happens in the *COLID Index Crawler Service*. In this service metadata will be extracted, transformed and transferred to Data Marketplace elasticsearch. 

### Add taxonomies to Index Mapping

Taxonomies are stored in the metadata for Data Marketplace. Metadata is provided by the *COLID Index Crawler Service*. The next step is to include the taxonomy in the search index to enable a semantic search. Metadata is already used to created the Data Marketplace index mapping dynamically. This process will be expanded with taxonomies.

As an example the taxonomy from above will be used.

#### Broader concept expansion of taxonomy

To allow users to find also narrower concepts for a search term, the taxonomy has to be stored in appropriate format. Therefore for each element in the taxonomy broader concept expansion will be applied.

Broader concept expansion widens the meaning of a term to be more generic. Take the following rules as example:

```
"computer => computer,",
"laptop => laptop, computer",
"mini => mini, laptop, computer"

```

If the user searches for "laptop" he will get results which contain the terms "laptop" and as well "computer".  For the term "mini" he will get additionally results for "laptop" and "computer".

#### Index mapping target structure

1. **Handling of phrases**

   Check phrases (more than one term) in taxonomies and transform them into one term. Using keepwords to only keep keyphrases into a separate taxonomical field. 

   Example: The phrase "Neuronal Network" has to be transformed into a single term, like "neuronal_network".

   *Solution:* Add Tokenfilter to handle phrases. Replace whitespace with underscore and save it as one term.

   ```
   "autophrase_syn": {
             "type": "synonym",
             "synonyms": [ 
               "pc portable => pc_portable",
               "mini latop => mini_laptop"
               ]
     }
   ```

   

2. **Broader concept expansion of taxonomy**

   Check for each element in the taxonomy if it has a broader element and if positive, add all broader elements to it. All elements will be stored in the field of the document in index.

   Example: mini => mini, laptop, computer

   *Solution:* Add Tokenfilter to apply broader concept expansion of taxonomy. All elements of taxonomy has to be add to the synonyms list.

   ```
   "vocab_taxonomy": {
             "type": "synonym",
             "tokenizer": "keyword",         
             "synonyms": [             
               "laptop => laptop, computer",            
               "mini => mini, laptop, computer"            
               ]
           }
   ```

   

3. **Add field for taxonomy to index mapping** 

   Finally, the taxonomy settings have to be applied to the index mapping.

   *Solution:* Index analyzers will be created which will use the created token filters from above and some other filters. Notice, broader concept expansion will be applied only at indexing. For search no broader concept expansions allowed. Therefore a dedicated search analyzer with only the token filter for handling of phrases is needed. 

   ```
   "analyzer": {
           "taxonomy_text": {
             "tokenizer": "standard",
             "filter": ["lowercase", "my_stop", "autophrase_syn", "vocab_taxonomy"]
           },
           "taxonomy_text_search": {
             "tokenizer": "standard",
             "filter": ["lowercase", "my_stop", "autophrase_syn"]
           }
         }
   ```

   Add an additional field to the index mapping which will store the terms of broader concept expansion. 

   ```
   "mappings": {
       "tech": {
         "properties": {
           "description": {
             "type": "text",
             "fields": {
               "synonyms": {
                 "type": "text",
                 "analyzer": "taxonomy_text",
                 "search_analyzer": "taxonomy_text_search"
               },
               "keyword": {
                "type": "keyword",
                 "ignore_above": 256
               }
             }
           }
         }
       }
     }
   ```

### Search with taxonomies

The search index with the described mapping from above is the foundation to enable search with taxonomies. Search with the taxonomy effect as described above is really easy. With a `query match` query on the synonyms field created in the mapping. 

```
"query":{
	"match": {
            "description.synonyms": "laptop"
     }
}
```

All the logic is happening during the indexing of the documents with the specified analyzers for the taxonomy field. The ranks on the search results will be based on taxonomic similarity. Top level terms in the taxonomy will become more common, like the term *computer*. Their document frequency increases tremendously, which makes them lower value terms when scoring. The more specific terms, like *laptop* which occur rarely, will have a higher score. 


