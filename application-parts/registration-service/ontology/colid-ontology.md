# COLID Ontology

The COLID Ontology also known as *PID Ontology* is combined with several standard Ontologies as form of collection.
- PID Ontology
- DASH Data Shapes Vocabulary
- ISO Country Codes
- SKOS Core

The PID Ontology is the application specific COLID Ontology. For the Open Source version currently the [pid_ontology_12_OpenSource.ttl](https://github.com/Bayer-Group/COLID-Setup/blob/master/fuseki-staging/graphs/metadata/pid_ontology_12__OpenSource.ttl) is being used.<br>
Various other standard classes such as Person, Concepts and SHACL validations etc. are inherited from the above-mentioned Ontologies which are part of the collection.

All the classes related to COLID are part of a super class **PID Concepts**. Everything related to COLID would be defined under this class.<br>

Every individual resource in COLID has its own class as well. Every resource type class belong to the parent *COLID Entry* Superclass. *COLID Entry* is the starting node for all resources that are registered in COLID.  



<pre>
PID CONCEPTS
│  
└───COLID Entry <mark>(new resource type declared here)</mark>
│   └───Core Entities
│       │   Column
│       │   Table
│       │   ...
│   └───Resources
│       └───Dataset
│           │   Generic Dataset
│           │   ...
│       │Mathematical **Model**
│       │   ...
│       
└───Consumer Group Class
└───Mathematical Model Concepts <mark>(Custom Classes for every resource)</mark>
│       └───(Sub classes for resource metadata)
└───...
</pre>

The Ontology primarily comprises of two sections. The resources are defined under the *COLID Entry* class. For example we offer Columns and Tables as core entities resources in our Open source version and Datasets such as Generic Dataset. As we know not only every resource is an owl class *(a: owl:Class)* but also is defined as a NodeShape *(a sh:NodeShape)* to specify validation constraints that need to be met as part of SHACL validations.<br>

One Important point to note that every resource class is defined as non abstract class as per DASH Data Shapes Vocabulary. In other words every resource type is identified as non abstract concept ***dash:abstract false***. This is important because every resource has direct instances in the system and cannot be marked as abstract.
To find more information about Data Shapes Vocabulary please visit [here](https://datashapes.org/dash#abstract). 

Thus below is the example of how the initial structure of any new resource class is defined.

<pre>
  https://pid.bayer.com/kos/19050/GenericDataset
  a owl:Class ;
  a sh:NodeShape ;
  <mark>dash:abstract false ;</mark>
  rdfs:comment "Any dataset that is not an RDF dataset, nor one of the more specific types of non-RDF datasets. It can be a file like xml, xls, csv or data stored in systems like sql, MongoDB, Neo4J etc." ;
  rdfs:label "Generic Dataset"@en ;
  rdfs:subClassOf https://pid.bayer.com/kos/19050/NonRDFDataset  .
</pre>

The information about metadata and instances for every resource is explained in the next section.