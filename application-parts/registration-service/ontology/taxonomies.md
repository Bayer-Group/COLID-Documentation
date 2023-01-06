# Taxonomies in COLID


A taxonomy is a system of hierarchical types that can be used to describe entities. The types are expressed in a class and subclass system.
So for example, a basic taxonomy might consist of a class called "Mathematical Model" which might have subclasses "Time Series Model" and "Classification Model". Then, "Classification Model" might in turn have subclasses "Neural Networks" and "Convolutional Networks". This hierarchy means that a "Neural Networks" is a type of "Classification Model", and is also a type of "Mathematical Model".

The below image exhibits the same example of all available categories for the mathematical models. 
![taxonomy example in COLID](./../assets/ontology/taxonomy.jpg)

## Creating Taxonomy

Taxonomies should generally follow a logical hierarchical structure. The terms representing the broadest concepts should be at the top level, and terms representing more specific concepts within those concepts must be placed at a deeper level. This can be represented well by **SKOS** Simple Knowledge Organization System Primer broader and narrower terms - *skos:broader* and *skos:narrower*. 
- a broader term which is generic and a narrower term which is a more specific type of the generic broader term,
- a broader term which is generic and the narrower term is a named instance (proper noun) of the generic broader term,
- a broader term which is a whole entity and a narrower term which is an integral part.

<pre>
Mathematical Model Category Taxonomy
│  <font size=1>every broad/narrow concept must be an instance of the corresponding COLID metadata field.</font>
│  <font size=1>rdf:type https://pid.bayer.com/kos/19050/MathematicalModelCategory </font>
└───Mathematical Model Category <font size=1>(Concept scheme for Mathematical Model Category)</font>
│   └───Regression Model
│   └───Time Series Model 
│   └───Classification Model 
│       └───Deep Learning Model <font size=1>skos:narrower concepts below</font>
│           │   Convolutional Networks <font size=1>skos:broader concept of Deep Learning Model</font>
│           │   Neural Networks <font size=1>skos:broader concept of Deep Learning Model</font>
│           │   LTSM Networks <font size=1>skos:broader concept of Deep Learning Model</font>
│       
│   └───Clustering Model
└───...
</pre>

Thus COLID follows the standard semantics process for defining taxonomies. The only point to be noted is that every concept should in-turn be instantiated to the respective COLID Metadata field.
<center>rdf:type https://pid.bayer.com/kos/19050/MathematicalModelCategory</center>

Once the taxonomy .ttl file is generated it can be attached to the metadata configuration which is explained in the next section.
