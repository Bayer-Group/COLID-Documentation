# Metadata Graph Configuration

Metadata graph configuration basically is the integral named graph which provides the current metadata overview being used in the COLID Editor.

This named graph is created with the unique identifier - https://pid.bayer.com/kos/19050/367403 - reference [here](https://github.com/Bayer-Group/COLID-Setup/blob/master/fuseki-staging/graphs/metadata/metadata_graph_configuration.ttl).

Below is the overview of sample metadata graph configuration in COLID. 
![metadata graph configuration](./../assets/ontology/mcg.jpg)

## Metadata Graphs

The main COLID Ontology - https://pid.bayer.com/pid_ontology/12/OpenSource where the entire metadata is defined is part of this section.
This may also contain the resource specific ontologies and taxonomies defined in the earlier sections.

## Instance Graphs

As explained in the previous section, the large controlled vocabularies can be saved as instances in instance graphs. This include user specific information such as countries names, model names etc.

## Resource Graphs

As seen in the earlier sections, every COLID entry has two entry lifecycle statuses viz. Draft and Published. We have two separate graphs to store each draft and published resources. These are defined in *Resource Graphs* and *Resource Draft Graph* section of the configuration respectively.

## Link History Graph

This graph stores the link history between entries. Every time user adds or removes a link between two resources, the link type and user information is stored in this graph.

## Creation of Metadata and Reindexing

Once the graphs have been uploaded to the triplestore and metadata configuration has been created, the *create metadata* button should be clicked to transfer the updated metadata and resources to the Data Marketplace and Elasticsearch.