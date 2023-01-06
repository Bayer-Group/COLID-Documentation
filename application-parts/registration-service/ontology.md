# Ontology

This section provides an overview of the concepts comprising the COLID Ontology.

One should be familiar with the concepts of linked data and semantic web in order to understand Ontology.
If this is not the case [Wikipedia (Linked Data)](https://en.wikipedia.org/wiki/Linked_data)/[Wikipedia (Semantic Web)](https://en.wikipedia.org/wiki/Semantic_Web), [Shapes Constraint Language (SHACL)](https://www.w3.org/TR/shacl/) and the [W3C (Linked Data)](https://www.w3.org/wiki/LinkedData)/[W3C (Semantic Web)](https://www.w3.org/standards/semanticweb/) are good places to get an overview about it.

The role of Ontologies on the Semantic Web are to help data integration when, for example, ambiguities may exist on the terms used in the different data sets, or when a bit of extra knowledge may lead to the discovery of new relationships.<br>

Every COLID Concept which may include different Resource types - its properties and their relationships is stored in form of Ontology.
COLID uses the [RDFS Schema](https://en.wikipedia.org/wiki/RDF_Schema) for defining ontologies.<br> The individual resource types classes are then controlled and validated for data integrity using with the help of Use Shapes Constraint Language (SHACL).

## RDF 
The **Resource Description Framework (RDF)** is a World Wide Web Consortium (W3C) standard originally designed as a data model for metadata. It has come to be used as a general method for description and exchange of graph data. RDF provides a variety of syntax notations and data serialization formats with **Turtle (Terse RDF Triple Language)** currently being the most widely used notation.<br>

- Think about data as a directed graph, and that all things has a
relation to other things.
- RDF is a directed graph composed of triple statements.
- Data described as ***triples***.
- A triple is also called a fact or a statement.
- The elements of a triple are also called ***resources***.
  - subject predicate object
- Use of *Uniform Resource Identifiers* **(URI)** to tell resources
from another.
  -   Example - http://bayer.com/colid/GenericDataset

### RDF Syntax
There a several ways of writing RDF.
- Turtle, N-triples, RDF/XML, JSON-LD (see easyrdf.org/converter)

#### RDF Turtle
```
<http://bayer.com/colid/GenericDataset>
<http://bayer.com/colid/hasInformationClassification>
<http://bayer.com/colid/Public> .
```

#### RDF Turtle, prefix
```
@prefix colid: <http://bayer.com/colid/> .

colid:GenericDataset
  colid:hasInformationClassification colid:Public .
```

#### RDF Turtle, literals
```
@prefix colid: <http://bayer.com/colid/> .

colid:GenericDataset
  rdfs:label "My Test Resource"@en;
  colid:hasVersion "1.00"^^xsd:double;
  colid:hasPersonalData true;
  colid:hasInformationClassification colid:Public .
```

### Vocabularies
- Groups of related terms as seen above are gathered in vocabularies.
- Usually under one *namespace*.
- Important, and well known namespaces:
  - RDF
  - RDFS
  - DCTerms
  - FOAF
  - XSD

## SHACL
The SHACL Shapes Constraint Language, is a language for validating RDF graphs against a set of conditions. These conditions are provided as shapes and other constructs expressed in the form of an RDF graph. Prior to SHACL; there was no W3C standard for validating RDF. SHACL has become W3C recommendation as per July 2017. 
<br>
The integrity and quality of the users inputs are validated using standard SHACL constraints.<br>

### SHACL Core Constraint Components
- Value type
  - sh:class Each value node is an instance of a given type.
  - sh:datatype Datatype of each value node.
  - sh:nodeKind Node kind (IRI, blank node etc.) of each value node.

    ```
    @prefix colid: <http://bayer.com/colid/> .

    colid:GenericDataset
      a sh:NodeShape ;
      sh:property [
        sh:path :author ; 
        sh:class :Person ;
      ] .
      
    [above example explains that every colid resource must have an author property which should belong to the standard OWL person class]
    ```


- Cardinality
  - sh:minCount  Minimum cardinality as xsd:integer.
  - sh:maxCount  Maximum cardinality as xsd:integer.
- Value range
  - sh:minExclusive x < $value
  - sh:minInclusive x <= $value
  - sh:maxExclusive x > $value
  - sh:maxInclusive x >= $value

    ```
    @prefix colid: <http://bayer.com/colid/> .

    colid:GenericDataset
      a sh:NodeShape ;
      sh:property [
        sh:path :hasVersion ; 
        sh:maxInclusive 10 ;
      ] .
      
    [above example explains that every colid resource must have no more than 10 versions]
    ```
- String-based
  - sh:minLength Minimum length as xsd:integer.
  - sh:maxLength Maximum length as xsd:integer.
  - sh:pattern Regular expression.
  - sh:languageIn A list of languages as per RFC5646.
  - sh:uniqueLang One unique tag per language.
    ```
    @prefix colid: <http://bayer.com/colid/> .

    colid:GenericDataset
      a sh:NodeShape ;
      sh:property [
        sh:path :authorEmail ; 
        sh:pattern "/^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/" ;
      ] .
      
    [above example explains that every author email must be a valid address]
    ```

Further details regarding the COLID Ontology Concepts can be found in the sub-sections.